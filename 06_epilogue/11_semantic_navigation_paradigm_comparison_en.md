# 11 This Is Not Just a Comparison of Three Technical Routes

It Is the Evolution of a Civilian-Scale Embodied System

By the time I got to the end, I actually stopped wanting to understand this whole volume as just a comparison between three semantic-navigation paradigms.

Because if I only describe it as:

- Route A builds an object map
- Route B builds scene memory
- Route C does online semantic decision-making

then the whole thing immediately collapses back into a standard feature list.

What I actually built feels much more like a strange little evolutionary history.

Not the history of algorithms.  
Not the history of papers.  
Not even fully the history of a product.

It is more like the history of how one person, under very ordinary conditions, with limited time, limited resources, and limited hardware, kept forcing classical robotics, visual perception, semantic maps, memory retrieval, embodied orchestration, and real-hardware coupling onto one single body.

So if I had to summarize this project again now, I would not call it "three routes."
I would call it:

**the evolution of a civilian-scale embodied system**

## Looking Back Across the Whole Series, This Was Really Four Thickening Steps

I think this point needs to be said clearly, otherwise this epilogue can easily look like it is only summarizing Volume Two.

If you spread the whole series out from the beginning, it was never talking about semantics right away, and it was definitely not touching embodiment right away.
There was a long, very honest, and frankly not very sexy period of just laying the foundation.

What Volume One solved was not really "intelligence" at all.
It was much closer to the most basic survival problem of a robot:

- where am I
- where around me is traversable
- how do I gradually explore unknown space
- how do I avoid getting lost inside my own closed loop

![alt text](assets/11_semantic_navigation_paradigm_comparison_demo_01.gif)

So Volume One was really building a **spatial closed loop**.
Without that layer, all the later things like object maps, scene memory, and VLM decision-making would just become words floating in the air.

Volume Two is where semantics really begins.
But even inside Volume Two, it is not one single problem.
It is two thickening steps:

- first pin the world down as an object map
- then start remembering the world as scene memory

At that point, the robot is no longer only asking "where can I go."
It is starting to ask "what is that," "where have I seen something like this," and "what kind of place should I be looking for."

![alt text](assets/11_semantic_navigation_paradigm_comparison_demo_02.gif)
![alt text](assets/11_semantic_navigation_paradigm_comparison_demo_03.gif)

Volume Three gets more aggressive.
In the first two volumes, semantics had already entered, but it was still mostly serving maps, memory, and retrieval.
By Volume Three, semantics starts entering control handoff itself.
That means the robot is no longer just using semantics to query memory.
It is using semantics to influence what action should happen right now.

That is the first real step from "semantic robot" toward "embodied agent."

![alt text](assets/11_semantic_navigation_paradigm_comparison_demo_04.gif)
![alt text](assets/11_semantic_navigation_paradigm_comparison_demo_05.gif)

Then Volume Four drags the whole story downward again.
It forced me to admit something:
no matter how complete the previous layers looked in simulation, the moment they touch real hardware, they have to answer everything again:

- is communication stable
- are the frames correct
- can the laser data be trusted
- should `cmd_vel` really run naked
- when the map jumps, will the system tear itself apart

![alt text](assets/11_semantic_navigation_paradigm_comparison_demo_06.gif)

So Volume Four did not add one more "higher-level capability."
What it really did was patch **real-world survivability** back into all the previous capabilities.

If you look at it that way, the whole series really feels like four rounds of thickening:

1. from not being able to survive, to surviving at all  
2. from merely surviving, to beginning to understand scenes  
3. from understanding scenes, to letting that understanding enter action  
4. from working in simulation, to becoming less easy to kill in reality

Once you see it like that, the main line of the series becomes much clearer.
It is not "I tried a bunch of fancy directions."
It is me slowly answering one question that keeps getting harder:

**how does a resource-limited robot grow from a spatial animal into something that has memory, semantics, action organization, and at least some tolerance for real-world friction**

## Why Does This Project Look Like a Frankenstein System

Because It Is One

And the more I worked on it, the less I felt that this needed to be hidden.

From start to finish, this project was never some pure one-line research effort:

- at the bottom there is `SLAM + Nav2 + exploration`
- on top of that sits `YOLO / YOLO-World`
- then `RTAB-Map`
- then another layer of `RAG`
- then another layer of `VLM orchestrator`
- then `MoveIt2`, `MTC`, `ROSplat` hanging off the side
- and on the real-hardware side, even more launch files, guards, watchdogs, and snapshots

Of course it is not elegant.
You could even say it has a very obvious "system-stitching" smell.

But to me, that is exactly the most real and most valuable part of it.
Because real robot systems rarely look like paper diagrams.
Much more often, they look like layers of history, layers of compromise, and layers of added capability somehow forced into one working whole.

So the project is not merely "like a stitched monster."
It **is** a stitched monster.

**It is just that, in the end, this stitched monster stayed alive.**

At least in simulation. At least in the eighty-point version. Let us not over-romanticize it.

## And Precisely Because It Is a Frankenstein System

You Actually Do Not See Many Personal Projects Like This Online

That is another thing I felt more and more strongly over time.

If I try to compress the outside landscape into a cleaner comparison frame, I now prefer to see it in three layers.

The first layer is **single-point open-source projects**.
These are the most common, and the easiest to find on GitHub.
They usually solve one very clear problem:

- navigation
- grasping
- `VLM / VLA`
- `RAG memory`
- 3D representation

Their advantage is that they are clean, pure, and easy to explain.
Their weakness is that they very rarely answer the ugliest question:

**how do these things stay alive together on the same robot**

The second layer is **personal-scale integration projects**.
This layer is the rarest and the most awkward.
There is no team-sized budget, no expensive platform, no long experimental window, but there is still the refusal to stop at one isolated module.
So the only option is to stitch the chains together by hand.

That is basically where my project has been struggling.

The advantage of this layer is that it is very close to real engineering.
Because it cannot avoid ugly things like interfaces, patches, recovery, compromise, and failure states.
Its weakness is also obvious:
it is not pure enough, not elegant enough, and not shaped like a beautiful academic problem.

The third layer is **team-scale embodied platforms**.
Something like `OK-Robot` already leans closer to that layer.
And above that, `Gemini Robotics` or `Helix` are obviously not within one person's budget to imitate.
Those efforts sit on much more complete platforms, more stable hardware, longer experiment cycles, and stronger compute and data conditions.

Once you place my own project inside that frame, its position actually becomes clearer.
It is not rare because it is somehow more advanced than the team-scale platforms.
It is rare because it sits exactly in that uncomfortable middle layer that very few people want to do, and very few people have the conditions to do.

If you search around GitHub, you will find a lot of related work, but most of it is more scattered:

- someone focuses on navigation
- someone focuses on grasping
- someone focuses on `VLM / VLA`
- someone focuses on `RAG memory`
- someone focuses on 3DGS visualization
- someone focuses on a benchmark

None of those projects are worthless.
Quite the opposite.
Most of them are clear, pure, and shaped like standard research questions.

But precisely because they are so pure, they usually only answer one local question.
The most painful part, which is **how to keep all these things alive together on one robot**, is often left to the reader's imagination.

And once you look at genuine integration projects, you immediately enter another world.
Something like `OK-Robot` is already much closer to team-scale investment:
platform, hardware, data, experiment time, reproduction chains, all of that is far beyond what an ordinary student can casually assemble on weekends.
And once you look higher, like Google DeepMind's `Gemini Robotics` or Figure's `Helix`, that is no longer "project inspiration." That is what happens when industrial and research frontiers are piled together.

So there are actually two kinds of things you see a lot of outside:

- single modules that are individually strong
- whole platforms backed by teams, funding, and hardware

What looks strangely empty is the awkward middle:

**can an ordinary robotics student, with limited resources, actually build an integrated system that spans navigation, perception, grasping, memory, language, and embodied control**

That is why I increasingly understand why there are not that many similar projects online.
It is not because nobody wants to do it.
It is because it sits in a very uncomfortable place:

- stronger people often do not want to do this kind of system-stitching labor
- ordinary students usually do not have the conditions to really run it through

So it turns into a very strange blank space.

## The Most Awkward and Most Valuable Thing About This Project

Is Exactly This Middle Layer

I think this point deserves to be said a bit cruelly, otherwise it stops being honest.

Why do top academic players usually not like this kind of project very much?
Because it is not pure.

It does not have one elegant core innovation point.
It does not have one clean contribution that looks like a paper title.
In many places it smells very obviously of engineering dirt:

- tweaking launch files
- patching remaps
- adding watchdogs
- building blacklists
- adding cooldowns
- pressing state machines into shape

In the eyes of very strong people, this kind of work is easy to dismiss as "system assembly," and therefore not noble enough.
Because they can go chase more frontier models, purer algorithms, and more futuristic unified policies directly.

But why do ordinary students usually not manage to finish it either?

**Because it is also too dirty, too long, and too time-consuming.**

What it needs is not one flash of inspiration.
It needs sustained engineering patience:

- understand ROS2 multi-package workspaces
- handle TF, topics, actions, and services
- tune simulation
- write state machines
- do the dirty work from perception to geometry
- connect the arm
- deal with real-hardware communication
- accept many moments where there is simply no shortcut
- **keep making mistakes, keep debugging, keep designing workflows to make the system real**

So in the end it lands in a very awkward but also very scarce place:

**top researchers often do not care for it, and ordinary students often cannot finish it**

And I think that explains quite well why it feels rare.

## From the Evolutionary View, Those Three Routes Were Never Really Replacements

They Were Three Layers of Cognitive Thickening

If you switch into an "evolutionary history" view, then the previous three routes should not be understood as a three-way choice.

They are much closer to the robot's cognitive ability getting thicker layer by layer.

At the first layer, the robot learns to pin the world down as objects.  
That is Route A.

At the second layer, it stops remembering only objects and starts remembering scenes, relations, and impressions.  
That is Route B.

At the third layer, it no longer only queries memory after the fact. It lets current semantics directly enter action decisions.  
That is Route C.

This is a rough but very real cognitive evolution:

- from "what is this, and where is it"
- to "I have seen this place before"
- to "which direction should I try right now"

So Route C does not mean Route A and Route B became obsolete.
Quite the opposite.
The further I went, the more I felt that the later capabilities are literally growing on top of the earlier ones.

Embodiment does not descend from the sky all at once.
It probably grows out of these old-fashioned, layered, compromise-heavy systems step by step.

## The More I Worked on This Project, the More Conservative My Judgment About the Future Became

Before doing the work, I was very easy to seduce by one narrative:

if the model becomes strong enough, then future robot systems should become more unified, more end-to-end, and eventually the whole stack will be swallowed by one bigger brain.

After doing the work, I believe that much less.

It is not that I do not believe in embodied AI.
Quite the opposite.
It is because actually doing the work made me much clearer that the hardest part is not "can the model say smart things."
It is:

**how does high-level understanding avoid killing the lower layers**

Of course stronger unified embodied systems will appear in the future.
That part feels almost inevitable.
Maybe in five years everyone starts yelling about Jarvis.
Maybe in ten years people want C-3PO.
Maybe in twenty years people accidentally summon Ultron if they are really unlucky.

But at least at my current stage, I believe something else more:

- the geometry layer will not disappear
- the memory layer will not disappear
- the control layer will not disappear
- high-level semantics will become stronger and stronger, but it will not erase everything underneath so easily

In other words, the future of embodiment may be less "one super-brain replaces everything" and more "stronger high-level layers keep pressing downward while classical layers remain the skeleton."

That is why I ended up preferring the phrase **hierarchical embodiment**.
It is less romantic, but right now it is the thing I can actually make stay alive.

## So What This Project Really Proved in the End Was Not That I Invented Something

It Was That I Walked Through a Road Most People Do Not Want to Walk

That is probably the calmest and also the harshest final evaluation I can give this project.

It is not an academic breakthrough.  
It is not a world-class platform.  
It is not a foundation model.

But it is also absolutely not just "a repo with a lot of demos."

What it really proves is something else:

**under personal-scale conditions, embodied systems do not have to remain trapped in PPTs and paper figures. They can first be made real in a very stitched, very engineering-heavy, very inelegant form.**

That is not especially glamorous.
But I actually think that is the hardest part of it.

And honestly, by the time this article goes out, I might really be one of the very few people on earth shameless enough to publish and showcase this kind of stitched-together thing in public.
There is a tiny bit of concept-god energy in that, I admit.

If the whole series had to leave behind one final sentence, I would probably stop here:

future embodied robots will definitely become more advanced.
But before that happens, someone still has to take all these scattered, expensive, academic capabilities and compress them into one robot that can actually move, see, remember, search, grasp, and still be repaired afterward.

And that may be the real meaning of why this stitched monster exists.
