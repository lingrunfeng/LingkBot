# 05 Route B: Scene-Memory Navigation with LLM Keyframes and RAG

## If the Previous Route Was About Remembering Objects, This One Is Much More About Remembering Scenes

After finishing `03`, one thing became clearer and clearer to me:
semantic navigation does not have only one route.

Quick note first:
this route is still exploratory and still needs optimization.
But it already reached maybe 80 percent of what I had in mind, so I think it is enough to show that the direction is real.

Route A is very hard-edged and very classical.
It tries to compress the world into an object distribution map:

- where the trash can is
- where the cabinet is
- where the dining table is

Then the navigation system simply heads for those coordinates.

But later I started feeling that humans do not always remember places like that.
Very often, what we remember is not "the exact coordinate of some object."
We remember something more like:

- I have been to that place before
- there was a messy table there
- that corner felt like a dining area
- I remember the green trash can being near a wall

So Route A is more like an object-level map.
Route B is more like scene-level memory.

It no longer insists on first compressing every target into a stable long-term object anchor.
Instead, it takes another route:

- let the robot remember some key views during exploration
- let a multimodal model analyze those views offline
- store those views together with pose, depth, and semantic descriptions as a retrievable memory
- and during navigation, do not ask "where is the object coordinate" first, ask "have I seen the kind of place you are talking about"

That idea already feels much more like memory, and much less like a database.

![alt text](assets/05_llm_rag_scene_memory_navigation_demo_01.gif)

## The Really Charming Part of This Route Is That It Allows Ambiguity

Why did I start wanting to build this route at all?
The reason is actually simple.

Route A is neat, but it expects the target to be a clean object:
nameable, detectable, and stably projectable.

Real instructions are often not that clean.

For example:

- go to that messy table from before
- go near the bookshelf
- go to the corner with the green trash can
- go to the place that feels like a dining area

Sometimes these descriptions include objects.
Sometimes they include relations.
Sometimes they are more like impressions.
It is hard to compress them immediately into "one object label plus one fixed coordinate."

So the temperament of Route B is very different from Route A.

Route A is saying:
**"the target is right here."**

Route B is more like:
**"I remember seeing a place kind of like what you described. Let me find that frame first, then recover the target from inside that frame."**

It is less certain than Route A, but it has much more of a memory-and-association flavor.

## The Real Key Is Not That the Big Model Is Smart, But That I Did Not Let It Directly Take Over Navigation

I have become more and more restrained about this over time.

I did not build this system into one of those dramatic demos where the user says one sentence, the LLM directly spits out `(x, y, yaw)`, and the robot just drives there.
That kind of thing looks cool, sure.
But once you think about it seriously for two minutes, you realize it skips the most important layer:

**geometric grounding**

So the route I finally designed is very explicit:

- language understanding handles memory retrieval
- visual grounding finds the target again inside the retrieved memory frame
- depth and the camera model turn that pixel back into a 3D point
- the stored pose and transform matrix pull that 3D point back into `map`
- and the final execution still goes to `Nav2`

So even though this route contains `LLM / VLM / RAG`, it still does not bypass the hardest classical robotics ingredients:

- depth
- intrinsics
- TF / pose
- 2D navigation

That matters a lot.
Because it means this is not a pure language-hallucination system.
It is a system that pushes memory back down into geometry.

## State 1: Record Keyframes During Exploration, But Do Not Dump the Whole Video into Memory

I really do not like the approach of stuffing the entire video stream into a memory base.
Yes, that is easy.
No, that is not very disciplined engineering.
And later retrieval and memory building get messy very quickly.

So the `keyframe_recorder.py` I wrote was never meant to be a recorder in the ordinary sense.
It is a **sparse keyframe sampler**.

Of course, what counts as a keyframe still needs more tuning. Otherwise the "key" part is not very key.

Its logic is roughly:

- wait until RGB, Depth, and CameraInfo are all ready
- check whether RGB and Depth timestamps are actually synchronized
- get the current `map <- camera_frame` TF
- only save a frame if the robot has moved far enough or rotated enough relative to the previous saved frame

The thresholds are not decorative either.
I used something roughly like:

- translation around `1.0 m`
- heading change around `45 deg`
- RGB / Depth sync tolerance around `0.2 s`

The meaning behind that design is very clear:
I do not want every image.
I want **memory slices with spatial increment**.

If the robot just jitters in place and you save ten nearly identical frames, the memory base quickly turns into repetitive junk.
If it actually moves forward a meter, or really changes how it is looking at the world, then that frame becomes worth keeping.

And one detail I like a lot:
I do not save just the image.

For every frame, I save:

- the RGB image
- the depth `.npy`
- the pose JSON
- the camera intrinsics
- `transform_map_from_camera`

So it is not "I took a picture."
It is "I took a picture at a world pose from which I can later project things back into the map."

That becomes incredibly important later.
Without that geometric anchor, the so-called memory base would only become an image-text retrieval toy instead of a navigation system.

## State 2: During Offline Memory Building, I Am Not Asking the Model to Write Prose

Once the keyframes are recorded, I do not call the model online while the robot is still running around.
I split that part out into an offline pipeline.

And by offline here, I do not mean disconnected from the internet.
I mean disconnected from the robot's already overloaded little brain. Let the robot rest a bit and send the frames to some cheaper external brain for analysis.

`build_rag_memory_offline.py` is basically the memory organizer of this whole route.

It looks for the full keyframe bundle:

- `frame_xxxxxx.jpg`
- `frame_xxxxxx_depth.npy`
- `frame_xxxxxx_pose.json`

Then it sends each frame to a relatively cheap VLM for scene analysis.
Though to be fair, current VLMs are not even that expensive anymore.

But the important part is this:
I do not let the model output free-form prose.
In the prompt, I force it to return strict JSON, roughly containing:

- `objects`
- `scene_summary`
- `navigation_landmarks`

I think this matters a lot.
If you let the model freestyle, the memory base becomes hard to control very quickly.
Today it says "a cluttered dining corner," tomorrow it says "messy table area near wall," and the day after that it says something else again. Then your local retrieval starts suffering from wording drift for no good reason.

So I would rather make it behave like it is writing notes for a robot, not writing literature for a human.

Each memory entry eventually contains:

- the image path
- the depth path
- the pose path
- the camera pose
- the camera intrinsics
- the scene-description JSON

And all of that gets written into one `rag_knowledge_base.json`.

There is also one very real point here that I actually want to say out loud.
This memory-building script does not pretend the API is always online.
If the model interface is unavailable, it falls back to a placeholder description instead of letting the whole pipeline die.

That is very engineering-coded, and I mean that in a good way.
I am not building a paper figure.
I am building a workflow that should still be alive when you want to run it again and again.

APIs go weird.
Free-tier quotas wobble.
Networks misbehave.
Those are not rare exceptions. They are daily life.

So this route was never built on the fantasy that the cloud is always reliable.

![alt text](assets/05_llm_rag_scene_memory_navigation_figure_01.png)

## State 3: During Real Retrieval, I Do Not Let the Model Guess from Scratch

The interesting part inside `rag_navigator_node.py` is not just "it uses Gemini."
The interesting part is that I did not write retrieval in a naive way.

The whole logic is layered.

### First Layer: Do a Very Plain Local Lexical Ranking First

I first compress each frame into a searchable text package:

- `frame_id`
- `scene_summary`
- `navigation_landmarks`
- object names, attributes, and relative positions

Then I do one round of local scoring first.

That does not sound fancy at all, but it matters a lot.
Because it shrinks the candidate space before anything gets sent to a remote model.

And I also added one tiny thing that turns out to be surprisingly useful:
**relation-aware boost**

If the query contains words like:

- near
- next to
- beside
- nearby

and also mentions two entities, then the ranking will bias a little toward frames where both entities appear together.

That sounds minor, but it is exactly on theme.
Because the real charm of Route B is not "single-object hit."
It is "scene relation retrieval."

### Second Layer: Let the Model Rerank Only Inside the Compressed Candidate Set

After the local recall stage, I compress the top-k candidates into a tighter description and send that to Gemini for reranking.

Notice what I did not do:
I did not dump all the original images into the model and tell it to rediscover the world from scratch.

I give it a narrower job:

- what is the user asking for
- which of these candidate frames is the closest match
- return only one `frame_id` and a score

That is much more stable than "let the model handle everything."
It is only responsible for the last bit of semantic disambiguation after local retrieval already did the first narrowing pass.

To put it more bluntly, the model here is more like the person who makes the final call, not the person hallucinating the whole world from zero.

### Third Layer: Add Anti-Shake Logic, Or Even Terminal Input Can Wreck the System

This part is very code-flavored and very system-flavored.

I added several guards around query handling:

- `query_cooldown_sec`
- `repeat_query_block_sec`
- `api_min_interval_sec`

These things are not cool at all, but they are valuable.
The moment you connect terminal input to a system that can trigger remote APIs and also send navigation goals, you immediately discover that repeated input, nervous Enter presses, and the same sentence being asked again and again will stir the whole chain into a mess.
I am a broke student. I do not have the budget to let people DDOS my own robot through a keyboard.

So Route B is not "the user casually chats, and the robot naturally understands."
Inside, it is actually very restrained about debouncing repeated input and stopping API calls from firing too fast.

## State 4: Retrieving the Right Keyframe Is Still Not the End

I think this is the part people misunderstand most easily.

When people hear "RAG navigation," they often imagine:

user query -> retrieve the most relevant keyframe -> drive to the camera pose of that keyframe

But later I realized that is too rough.
Usually the place you want to go is not "where the camera was when the picture was taken."
It is **where the thing you care about inside that picture actually is**.

So I added one more very important step:

**do visual grounding again inside the retrieved memory frame**

That means:

1. pick the most relevant frame from the memory base
2. **then ask again on that RGB image: where in this image is the thing the user meant**
3. once I get `(u, v)` or a bounding box, read the target depth from the saved depth map
4. back-project that pixel into a 3D point in camera coordinates
5. then use the saved `transform_map_from_camera` to bring it back into `map`

Once this step exists, the nature of Route B changes completely.
It is no longer "navigate to a memory photo."
It becomes "recover the target location on the map through a memory photo."

I think this step is especially valuable.
Because it shows that even though the route looks like an AI memory demo, it still comes back seriously to the hard geometric layer of robotics.

## I Even Handled Depth Holes, Because I Did Not Want the Whole Route to Die on One Empty Pixel

This is another kind of system detail that I like a lot.

When grounding returns `(u, v)`, I do not naively assume that pixel definitely has valid depth.
In real life, usually:

- that point may land on an object edge
- the depth may be missing
- reflections or occlusions may have damaged it

So I added two fallback layers:

- first take a median depth over a small patch
- if there is still no valid depth there, search nearby for the nearest valid depth pixel

And if the grounding API fails completely, I even added a fallback:

- fall back to the image center

That is obviously not a "smart" fallback.
But it is an honest one.
Because a repeatable pipeline does not survive by never failing.
It survives by not collapsing completely when one part fails.

## State 5: In the End It Still Goes Back to Nav2, But Not Recklessly

Once the target point is recovered into `map`, the final step is still to send `NavigateToPose`.
I very deliberately did not change that.

The more this system grows, the more I think it should follow one principle:

**the freer the high-level semantics become, the more conservative the low-level execution should be**

So Route B does not directly navigate to the target point itself.
It keeps a `nav_standoff_m`.
The default is roughly `0.7 m`.

The reasoning is simple:

- the recovered target came from memory, not live measurement
- memory itself has error
- grounding has error
- depth recovery and pose transformation have error

So if the robot drives straight into the exact target center as if nothing can possibly be wrong, that is just way too confident.

So in the end I generate a more conservative goal in front of the target, based on the direction from the original camera pose toward the target point, and then make the robot face toward the target.

That cut is small, but I think it fits Route B very well.
Because this route carries the romance of memory, but it also has to accept the imprecision of memory.

![alt text](assets/05_llm_rag_scene_memory_navigation_figure_02.png)

## The Strongest Part of This Route Is Not Determinism, It Is That the Robot Finally Starts Having Something Like Recall

If I had to summarize the difference between Route B and Route A in one line now, I would say:

- Route A turns the world into an object map
- Route B turns the world into scene memory

The strongest part of Route A is determinism.
The strongest part of Route B is the sense of memory.

It is especially good for:

- relation-heavy descriptions
- scene-flavored descriptions
- descriptions that do not compress cleanly into a fixed object label

It moves the robot one step forward from "knowing where an object is" toward "remembering what kind of place it has seen."

That step feels very important to me.
Because from that moment on, the robot no longer faces the world only with immediate detection and immediate control. It gains one more temporal layer:

the past.

And that past is memory.

## But It Is Also More Fragile, and More Expensive

I do not want to write this route as something magical.

It really does have flavor, but its problems are also pretty clear.

### 1. It Is Less Deterministic than Route A

Once Route A has pushed an object into the map, it becomes a relatively stable anchor.
Route B, on the other hand, has to go through the whole chain again for every query:

- retrieval
- frame selection
- grounding
- depth recovery
- geometric reprojection

The chain is longer, and there are more sources of error.

### 2. It Depends Much More on Memory Quality

If the keyframes are sampled badly, or the relevant area was never covered at all, then no amount of smart retrieval will save it.
It does not understand the world from nowhere.
**It can only recover from what it saw in the past.**

### 3. It Depends More on APIs and Budget

Offline memory building calls models.
Online reranking and grounding also call models.
So this route is naturally more expensive than Route A, and naturally more sensitive to API status.

That is why I intentionally wrote in all the ugly but necessary things like placeholder fallback, cooldown, and minimum intervals.
If you do not account for those real constraints, what you built is not a system. It is a lottery. It is also a very efficient way to burn money.

## What the Full System Looks Like When This Route Actually Runs

I still want to compress the whole workflow once more, because if I only talk about the modules separately, the article can become misleading.

When it is fully running, the state chain looks roughly like this:

1. first there is still a stable `2D SLAM + Nav2` main loop underneath
2. during exploration or patrol, `keyframe_recorder` sparsely stores keyframes based on translation and rotation thresholds
3. each keyframe stores RGB, Depth, Pose, Intrinsics, and `transform_map_from_camera`
4. an offline script sends those keyframes to a cheap VLM, gets structured scene descriptions back, and writes them into `rag_knowledge_base.json`
5. optionally, the system can publish keyframe-memory markers in RViz as a 3D memory layer on top of the 3D map
6. the user enters a natural-language target in the terminal
7. `rag_navigator_node` does local retrieval first, then model reranking, and picks the most relevant keyframe
8. visual grounding runs again on that selected keyframe to locate the target in image pixels
9. using the stored depth and camera model, the target is back-projected into a 3D point
10. using the stored pose transform, that point is mapped back into `map`
11. finally, a standoff goal pose is generated and handed to `NavigateToPose`

After this whole chain works, the thing I am happiest about is not that "it looks like AI."
It is that it finally nails down something that is usually hard to describe:

**natural-language description -> scene memory -> geometric recovery -> classical navigation**

Once that bridge exists, the semantic layer of the whole project changes level completely.

![alt text](assets/05_llm_rag_scene_memory_navigation_figure_03.png)

## A Final Note on Route B

If Route A answers "where exactly is the object on the map," then Route B is answering another question:

**can the robot remember the world it has seen**

It is not as hard-edged as Route A.
It is not as stable as Route A.
And it is definitely not as cheap as Route A. Though to be fair, Route A is not as perfect in practice as many people imagine either.

But Route B is the first time this project really moves from object coordinates into scene memory.

And once the system starts having memory, the story changes again.
Because the next step is no longer just "remember where I have been."
It becomes "look at the current scene and start deciding online where to explore next."

At that point, memory may not even have to be fully prebuilt.
High-level semantic decision-making starts happening online.

Route A shows that the robot can pin named things onto the map.
Route B shows that the robot can recall past scenes a bit more like a human.

And that is exactly the more aggressive Route C I want to write next.
