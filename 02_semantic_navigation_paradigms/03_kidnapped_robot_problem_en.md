# 03 Kidnapped Robot Recovery: When 2D Navigation Forgets, 3D Memory Steps In

In the previous articles, I kept repeating one point pretty clearly:
for the main navigation stack, I still prefer to trust 2D. `SLAM + Nav2` is mature enough, and the engineering cost is lower too.

But that does not mean 3D has no value.
The moment 3D really starts earning its keep is often not during normal navigation. It is when the 2D stack suddenly loses the plot.

That is basically the classic kidnapped robot problem.
Maybe someone moved the robot by hand. Maybe odometry drifted hard. Maybe local localization quietly went off the rails. `AMCL` still thinks the robot is somewhere near the old pose, and `Nav2` is still planning as if that belief were still true. If you keep trusting 2D at that point, the whole system only gets more awkward and more wrong.

So the idea here was very simple:
let the 2D stack handle everyday navigation, and when things go sideways, let the `RTAB-Map` 3D memory / relocalization line step in as the fallback.

![alt text](assets/03_kidnapped_robot_problem_demo_01.gif)

## What I Actually Built Was Not "3D Replaces 2D"

It was much narrower than that.
I only wanted 3D to handle one very specific job:

**when the 2D side has already picked the wrong worldview, 3D should drag it back to the right one.**

The whole chain is basically just three steps:

1. Build a 3D map database with RGB-D memory using `rtabmap_hybrid.launch.py`
2. When kidnapped recovery is needed, switch to read-only localization mode with `rtabmap_localization.launch.py`
3. Use `rtabmap_pose_overwrite.py` to compare how far apart the `RTAB-Map` pose and the `AMCL` pose really are

The key point is not just that "a 3D map exists."
The real question is whether that 3D map can still give you a globally believable pose after the rest of the system has already gotten confused.

I wrote `rtabmap_pose_overwrite.py` in a pretty restrained way.
It does not see one `RTAB-Map` result and immediately shove it through like a hero in a hurry.
It checks several things first:

- compare `/rtabmap/localization_pose` with `/amcl_pose`
- check whether the translation gap exceeds a threshold
- check whether the yaw gap exceeds a threshold
- if the `AMCL` covariance is already looking ugly, lower the thresholds a bit
- enforce a minimum re-publish interval so `/initialpose` does not get spammed like crazy

Only when it really looks like "2D thinks it is in the wrong place" do I send the `RTAB-Map` global pose back into `/initialpose` and pull `Nav2 / AMCL` back into the correct frame.

And before that, I send a zero velocity command first.
It is a tiny move, but it matters a lot. When the robot's internal picture of the world is already wrong, the last thing it should be doing is confidently continuing to drive.

## The Recovery Workflow Is Actually Pretty Simple

This chapter does not need fake complexity.
The state chain is basically just this:

`normal 2D navigation -> kidnapped event / severe drift -> RTAB-Map performs global relocalization from the 3D map -> recovery bridge compares 3D pose with AMCL pose -> difference is large enough -> stop the robot -> resend /initialpose -> 2D navigation resumes`

So the point of this article is also very straightforward:

- I do not use 3D to dominate everyday navigation
- but I do keep a more expensive and more global 3D memory line alive in the system
- once 2D localization becomes seriously wrong, 3D stops being a nice extra and becomes the fallback layer

There is also one small detail here that I personally like a lot:
I added an extra `adaptive_tuner` layer on top of `RTAB-Map`.
When the robot is mostly still, it leans toward faster relocalization settings.
When the robot is moving, it tightens the matching conditions a bit more, mainly to reduce bad teleports.

That is not glamorous, but it is very real.
Because the scariest thing in kidnapped recovery is not "slow."
The scariest thing is **being wrong while sounding extremely sure of yourself.**

Of course it can still jump to nonsense sometimes.
That leads straight into the classic symmetric-room problem, which is not exactly new in robotics.

## So What Is 3D Really Doing Here

If Route A is about object maps, and Route B is about scene memory, then this small chapter is really about another very practical value of 3D:

**when the 2D world model collapses, who tells the robot where it actually is**

That is why I never felt 2D and 3D had to be some dramatic either-or choice.
In my system, 2D is responsible for living efficiently on normal days.
3D is responsible for pulling the robot back when the system suddenly forgets what world it is in.
