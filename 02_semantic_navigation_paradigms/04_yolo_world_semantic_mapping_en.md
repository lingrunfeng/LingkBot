# 04 Route A: Object-Level Online Semantic Navigation with YOLO-World

## Volume Two Really Starts Here, Because This Is Where Semantics First Lands on the Map

If the previous two articles were about whether the robot could survive stably and whether it could chain movement, vision, and grasping into one task loop, then from this article onward the question starts changing.

This post is no longer just about whether the robot can move, or whether it can grab.
It starts caring about something else:

**can the robot know that some place on the map is not just "reachable," but "there is a specific thing there"?**

That is why I put this route at the beginning of Volume Two.

Because from this point on, semantics stops being just a detection box on the screen, or a label floating inside an image.
It starts getting compressed into a stable spatial frame.

In other words, the robot no longer only knows:

- where the walls are
- where the free space is
- where navigation is possible

It starts trying to know:

- where the green trash can is
- where the brown wooden cabinet is
- where the dining table is
- which observations belong to the same object, and which ones do not

At first glance this can look like "just adding one more label layer."
At the system level, it is really a qualitative shift.
Because once semantics can actually land on the map, navigation is no longer just about going to a geometric goal point. It can go to a goal point that has a name, a category, and a spatial meaning.

![alt text](assets/04_yolo_world_semantic_mapping_demo_01.gif)

## This Route Has a Very Different Temperament from the Later RAG Route

I already said in `00` that Volume Two is really about comparing several very different paradigms of semantic navigation.

This article corresponds to the hardest and most explicit route:

- detect concrete objects first
- project those objects into map coordinates
- then let the navigation stack go directly to those coordinates

Its whole vibe is like a handshake between classical robotics and modern vision models.
The vision model answers, "what is this?"
The classical robot stack answers, "where exactly is it on the map?"

What I like about this route is that it does not try to be mystical.
It is not saying "I kind of remember a place that looks like this."
It is not saying "the model feels like the goal might be on the left."
It is more like building an actual object distribution map:

- this trash can is at `(x, y, z)`
- that cabinet is at `(x, y, z)`
- this table is not the same table as the previous one

So if I had to summarize Route A in one sentence, it would be:

**turn open-vocabulary vision into an object-level world model you can keep, query, and actually navigate to**

## The Real Key Is Not YOLO-World Itself, But How It Gets Pulled into the Full Loop

If all I did was run a standalone `YOLO-World` detection demo, that would not be very interesting.
At most it would prove that the model can recognize some words inside an image.

What I actually wanted was not "make the robot able to see."
I wanted to plug "seeing" into the whole system.

The workflow I ended up with looks roughly like this:

### State 1: First Build a Stable 2D Spatial Closed Loop

This part still follows the old rule:
let `SLAM + Nav2` stand firmly first.
Without that, semantics has no reliable place to land.

This matters a lot, because here I did not let 3D visual mapping steal authority from the 2D navigation loop.
I kept a fairly restrained design:

- the 2D map and main navigation loop still come from the already stable workflow
- `RTAB-Map` mainly handles the 3D visual mapping and visualization layer
- in hybrid mapping, I explicitly keep `RTAB-Map` from publishing the `map -> odom` TF

To put it bluntly, I did not want one extra 3D semantic layer to mess up a 2D action loop that was already working.

That is a pretty typical engineering philosophy for this route:

**the semantic layer should stack on top of a stable spatial layer, not break it from underneath**

There is also one thing people often misunderstand here:
if I already have 3D mapping and 3D semantics, why am I still insisting on a 2D main navigation stack?

The answer is not complicated.
At the current stage of this project, the robot is still mostly doing the classic ground robot jobs:

- localization on a plane
- obstacle avoidance on a plane
- path planning on a plane
- continuous forward progress in an unknown environment

For that task shape, the 2D stack is not "outdated."
It is simply more mature, more stable, and much cheaper to debug.

It is not that I never thought about letting 3D become the main navigation stack.
I did think about it.
I just realized very quickly that at this stage it would blow up system complexity without buying me enough in return. That is not laziness. That is minimum-cost implementation.

So the conclusion I eventually accepted was:

- let 2D keep the main action loop
- let 3D fill in the parts 2D is naturally bad at expressing

Things like object height, scene appearance, semantic anchors, and scene-level memory are all awkward for a pure 2D map.
But if the goal today is to let the robot move, avoid obstacles, and explore stably in a planar environment, 2D is still the more practical choice.

So 3D is absolutely useful here.
Its value just does not have to show up as "replace 2D navigation immediately."

I gradually became more comfortable with a pragmatic division of labor:

**2D keeps the robot alive, 3D helps the robot start understanding more**

And this becomes even clearer later.
For example, when 2D localization gets badly damaged and the robot falls into a classic kidnapped-robot situation, the value of the 3D map and visual relocalization suddenly shoots up.
At that point it is no longer just a pretty semantic layer. It becomes a key clue for restoring global consistency.

So in this article, I do not let 3D steal the main navigation role.
But that does not mean 3D is just decoration.
More accurately, it enters the system in a restrained way first, and later, when I get into robustness and kidnapped recovery, its importance becomes much more obvious.

### State 2: Start the 3D Visual Memory Layer in Parallel

In the hybrid route, I launch `RTAB-Map` in parallel so it can do 3D visual mapping, point cloud accumulation, and database building.

One thing I really like here is that this branch is not reckless.
I do not let `RTAB-Map` take over all authority over how the map is interpreted.
I keep it fairly well-behaved under the assumptions of a ground robot:

- `Reg/Force3DoF = true`
- `publish_tf = false` during mapping
- `Mem/IncrementalMemory = true`
- `Grid/3D = true`

The meaning behind those settings is very straightforward:
I want the 3D semantic layer, but I do not allow it to drag the whole system off the ground.

### State 3: YOLO-World Takes Over "What I Want the Robot to Look For"

The thing that makes this route different from an ordinary YOLO detection demo is that in the launch setup I kept the ability to pass the vocabulary at runtime.

That means the model is not frozen into a fixed training label set.
Instead, "what I want the robot to look for right now" becomes an open-vocabulary input that can change at runtime.

For example, I used word lists like:

- `brown wooden cabinet`
- `window`
- `green trash can`
- `dining table`
- `large grey table`
- `bed`
- `bookshelf`

That is already interesting by itself.
Because from this point on, the robot is no longer just running an industrial vision task with frozen labels. It starts to feel a little more like "look at the world according to what I care about right now."

That said, there is still a tiny bit of cheating here.
The vocabulary still needs to be based on the things that actually exist in the map I am testing in. Otherwise YOLO-World either recognizes nothing or confidently recognizes everything. You can think of it as giving the robot a little warning in advance: "these are the kinds of things you are likely to encounter, pay attention."

### State 4: `semantic_mapper` Starts Compressing 2D Detections into 3D Objects

This is where the real core of the route lives.

The `semantic_mapper.py` I wrote is not some little helper that casually logs detections when they arrive.
It is basically the heart of the whole Route A pipeline.

What it actually does is very concrete:

- subscribe to `/yolo/prediction/item_dict`
- read the depth image and camera intrinsics at the same time
- convert a 2D center point into camera-frame 3D coordinates
- **then transform that 3D point into the `map` frame with `TF2`**
- filter out obviously unreliable points
- deduplicate and smooth the result
- finally persist everything into a JSON semantic map

There is no flashy "look at this AI moment" here.
But I honestly think this is the most valuable part of the whole article.
Because from here onward, YOLO-World stops being just an image-understanding model and starts participating in **spatial world modeling**.

## I Am Not Just Throwing Bounding Box Centers into the Map

A lot of people imagine this step as:

detection -> take the center point -> convert the coordinate -> done

If I actually did that, the semantic map would turn into garbage very quickly.

Inside `semantic_mapper.py`, I added quite a few constraints that are not flashy at all, but matter a lot.

### 1. Confidence First, Semantics Second

I put a confidence threshold in front of the detections first. The default is `min_confidence = 0.4`.

That is not some beautiful number. It just answers one very plain question:
if the model itself is not that sure what it saw, why rush to write it into the map?

And in my case the simulation world is genuinely abstract, almost Minecraft-like, not a realistic scene.
Imagine asking the model to recognize a cat-shaped cookie as a real cat. That sounds ridiculous, but it is honestly not that far from the kind of thing YOLO-World ends up dealing with in simulation.

### 2. Depth Comes from a Small Patch Median, Not One Lonely Pixel

I did not grab one center pixel depth value and just trust it.
I take a small patch around the center, keep only valid depth values, and then use the median.

This sounds like a tiny detail, but it decides whether the system is "remembering a map" or "remembering noise."

Anyone who has actually done RGB-D projection knows the drill:
edges, reflections, missing values, and weird geometry can all make a single-pixel depth estimate look like a joke.

### 3. I Added a Second Filtering Layer in 3D Space

Even if the 2D detection passes, that still does not mean the depth is trustworthy.
So after projection into the map frame, I apply another spatial filter:

- too close, do not keep it
- too far, do not keep it
- too low, like fake ground-near points, do not keep it
- too high, like ceiling junk or distant wall hits, do not keep it

This layer is basically answering a simple question:
the robot is not building a philosophical dictionary of all things in the universe.
It is building an object map that is useful for action.

### 4. I Do Not Treat Every New Detection as a New Object

This is another design choice that I specifically like.

If the robot passes the same cabinet three times and every pass gets written as a new object into the JSON file, then that semantic map becomes untrustworthy very quickly.
So I added a straightforward instance-level deduplication logic:

- only compare objects of the same class
- if a new 3D point is within 2 meters of an existing instance of the same class, first assume they are the same object
- then use a simple EMA to smooth the object position

None of this is very "academic."
But it feels exactly like what a real system should do.
You do not need perfect object tracking first.
You just need to stop the map from becoming more and more chaotic.

### 5. I Keep Multiple Instances Instead of Smearing Everything Together

On the other hand, if it is not the same object, I do not lazily merge every cabinet into one mega-cabinet either.

I add suffixes for new instances:

- `brown wooden cabinet_1`
- `brown wooden cabinet_2`
- `green trash can_1`

That is the point where the map finally starts looking a little like an object database instead of a flat label list.

### 6. The Semantic Map Is Not a Temporary Effect, It Gets Saved

This part matters too.

I did not build this semantic branch as something that "looks lively while the node is open and disappears when you close it."
`semantic_mapper` saves its output every few seconds and writes it into `semantic_map.json`.

That means the goal of this route was never "make a visual show."
It was:

**build a semantic map that stays behind**

![alt text](assets/04_yolo_world_semantic_mapping_figure_01.png)

## The Strongest Part of This Route Is Determinism

The more I worked on this, the more I felt that the biggest charm of Route A is not that it is the smartest route.
It is that it is the most deterministic.

It is especially good at answering instructions like:

- go to the green trash can
- go near the dining table
- go beside that cabinet

Because in this route, those targets are all pushed, as much as possible, into a clear coordinate.

That is very different from the later RAG route.
RAG is more like saying, "I remember a place that looks like this."
This route is more like saying, "that object is right there."

That kind of determinism is powerful.
Once it works, the whole navigation system becomes very clean:

- find the object
- get the object coordinate
- send that coordinate to Nav2

This is about as close as semantic navigation gets to classical robotics intuition.

## But This Route Has Very Clear Boundaries Too

I do not want to write this as some unbeatable route.
It is strong, but its limits are also pretty obvious.

### 1. It Depends on the Detection Vocabulary and Object Visibility

If the target itself is hard to detect, or the vocabulary does not cover it, or the robot never sees it from a sensible angle, then the object map is not going to magically grow that object on its own.

### 2. At the End of the Day It Is Still "Object-Center Navigation"

That means it is better at things like:

- cabinets
- trash cans
- tables
- windows

But if your instruction is more like:

- go to that messy place from earlier
- go to the corner that feels like a dining area
- go to the spot near the bed but not right next to the bed

then this route becomes much less natural.

### 3. It Is More Sensitive to Geometry Quality than Language Ability

The fragile parts of this route are usually not "the model did not understand the word."
They are things like:

- is the depth stable
- is the TF tree stable
- is the 3D projection trustworthy
- is the deduplication too coarse or too sensitive

So even though this route looks like an AI route, a large part of its upper limit is still decided by classical spatial geometry.

## What the Full System Looks Like When This Route Is Running

I think this has to be stated clearly, because this route is not "one node."
It is an entire workflow.

When fully running, it looks roughly like this:

1. first, a stable 2D mapping and navigation loop already exists underneath
2. hybrid mapping launches the `RTAB-Map` 3D layer in parallel, but without stealing the main TF role
3. `YOLO-World` performs online detection based on the current open vocabulary
4. `semantic_mapper` continuously projects the detections into the `map` frame
5. the system publishes semantic markers in RViz
6. semantic objects are continuously deduplicated, smoothed, and saved into `semantic_map.json`
7. once mapping is done, this object map can be consumed directly by later navigation
8. `semantic_navigator` loads the map, matches the instruction to a target, and then calls `NavigateToPose`

So even though this route is already semantic, the control flow is still very plain.

It does not pretend to be some general brain that can reason about everything.
It simply upgrades an ordinary map into a map with object anchors.

That is why I call it Route A of Volume Two.
It is the classical school of semantic navigation:

- first there is space
- then there are objects
- then the objects become navigation anchors

![alt text](assets/04_yolo_world_semantic_mapping_figure_02.png)

## A Final Note on Route A

I like this route a lot, not because it is the most cutting-edge one, but because it is honest.

It does not try to immediately give the robot fuzzy memory, open-ended reasoning, or complicated common sense.
It just does one very hard thing first:

**compress "what exists in the world" into "what exists where on the map"**

Once that works, the robot's understanding of the world grows for the first time from a geometric map into an object map.

I think that is a very important step.
Because it means the robot no longer only knows where it can go. It starts knowing what it is going there to find.

And in the next article, I move to a completely different route.
That one stops insisting on pinning every object into a precise coordinate. Instead, it lets the robot start remembering scenes, descriptions, and that faint feeling of "I think I have been here before."

That is the move from an object-level world to scene-level memory.

One more honest detail:
YOLO-World is still not going to be perfectly accurate in a simulation world like this.
But the nice part is that you can always open the map JSON and edit the markers by hand while looking at the 3D map. Honestly, that is not even a bad solution. It is direct, and sometimes it is the precise solution.
