# 09 ROSplat and 3DGS: Why I Still Want a Gaussian Scene Layer

This chapter is a side quest, and a short one.

It is not part of the main line for now.  
My real main line is still `SLAM + Nav2 + semantic navigation + grasping`.

But I still wired in the `ROSplat` direction, because the further I went, the more obvious it became that `RViz`, while engineering-friendly and reliable, is not actually very good at answering one simple question:

**what does the scene itself really look like**

So what I am doing here is not using 3DGS to replace the main navigation stack, and not using a Gaussian scene layer to directly control the robot.
I put it in a much more restrained place:

**a parallel representation layer**

Even in the code, this line is very well-behaved.
I kept a dedicated `rosplat_visualization.launch.py`, off by default and only enabled when needed.
It runs in its own namespace, and it can either subscribe to Gaussian topics automatically or directly load a `.ply`.

In other words, from the start it was never meant to steal control authority from the main system.
It sits off to the side as a parallel visualization extension.

That said, it may still become a big hint toward what I want to study later, because with the hardware and compute I have right now, I simply cannot go deep on 3DGS itself.

![alt text](assets/09_3dgs_rosplat_visualization_demo_01.gif)

## The Most Realistic Value of This Line Right Now Is Not Control

It Is Making Spatial Memory Visible

I think that part matters quite a lot.

Earlier in the project, I already built object maps, scene memory, and 3D relocalization.
All of those things already imply that the robot is no longer just "something that can move."
It is gradually accumulating a richer kind of spatial memory.

But those memories are often still too abstract:

- the 2D map is flat
- markers are symbolic
- JSON is cold
- logs are discrete

What makes `ROSplat / 3DGS` attractive is that it has a chance to turn those abstract spatial memories back into something more continuous, and closer to what the scene actually looked like.

This is also a good place to compare it with the `RTAB-Map` line I already have running.
In my system, `RTAB-Map` is already a working thing, and its value is very clear:

- it can build a 3D memory database
- it can support kidnapped recovery as a global relocalization fallback
- it can give the system some basis for saying "I think I have seen this scene here before"

But `RTAB-Map` is more functional.
It feels more like a reliable 3D geometry memory and relocalization tool.
Its strength is in "can this be used for localization, recovery, and navigation semantics."

`ROSplat / 3DGS`, on the other hand, is trying to fill another gap:

**not first asking whether the system can compute correctly, but whether it can express the scene more like the scene itself**

So if I put them side by side, I do not see them as replacements:

- `RTAB-Map` is more like the 3D working layer that already exists today
- `ROSplat / 3DGS` is more like a potentially stronger 3D representation layer for later

The former has already entered my navigation, recovery, and memory chain.
The latter is still mostly sitting in the visualization and expression layer.
But once hardware, data, and compute conditions improve, it may push forward the question of **what kind of world the robot is actually remembering** by a very large margin.

If I ever keep pushing this line further, I think it has at least a few very practical directions:

- give semantic navigation and scene memory a stronger 3D playback interface
- give kidnapped recovery a more intuitive relocalization visualization
- give post-exploration map inspection a viewing window that is closer to the scene itself
- support better demo replay than just staring at lines and boxes in RViz
- move further toward digital twins, offline analysis, and remote-operation understanding

To put it bluntly, its first value is not "make the robot smarter."
Its first value is making the spatial knowledge the robot has already accumulated finally look like **a scene**, instead of just a pile of topics.

## Why I Did Not Keep Going Further Right Now

The reason is simple.
It is not that the direction is uninteresting.
It is that it is genuinely expensive.

High-quality Gaussian scene layers do not just require enthusiasm.
They require a whole stack of more expensive conditions:

- better data-collection hardware
- more stable multi-view capture
- stronger GPU resources
- longer offline reconstruction time

Right now, the conditions I have are still centered on the mobile-robot main line:
first get the system running, first hold together navigation, exploration, semantics, grasping, and real-hardware deployment.

Under that constraint, I really do not have the conditions to push deep into high-quality Gaussian scene reconstruction yet.

That said, a lot of the things I am currently doing on the `RTAB-Map` side are already moving toward functions that 3DGS could eventually provide.
In a sense, I am using more primitive tools to approximate some of the things a richer 3D scene representation would naturally give me.

So this chapter is more like an interface I am deliberately keeping alive.
It says: I did not miss this direction. I know why it matters, and I know why it is still paused here for now.

## If This Line Ever Gets Properly Filled In, Its Value Could Be Very High

My judgment about it has never really changed:
in the short term it looks like a visualization extension, but in the long term it may become much more than that.

Once the hardware and data conditions are good enough, representations like 3DGS may take on a much heavier role inside robot systems:

- finer scene memory
- more natural human-facing explanation interfaces
- stronger task replay and error analysis
- maybe even one of the long-term visual world models inside an embodied system

So I wrote this as a side quest not because it is unimportant.
Quite the opposite.
It is because I have not really pushed it to the end yet, but I can already see what it might grow into later.
