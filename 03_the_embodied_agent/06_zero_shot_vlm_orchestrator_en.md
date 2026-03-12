# 06 A Hierarchical Embodied Agent with Online VLM Control Handoff

## This Is the Chapter Where I Finally Want to Talk About Embodied AI

In the previous two routes, I was still being fairly restrained.

Route A was about object maps.  
Route B was about scene memory.

Both already had semantics in them, and both had clearly moved beyond traditional navigation.  
But at the end of the day, they still carried that flavor of "store things first, come back and use them later."

This chapter is different.  
Here I want to touch a much bigger phrase directly:

**embodied AI.**

Not because I built some world-class robot foundation model. Of course not.
I still ask myself sometimes whether embodiment is even the point, or whether good old automation would already solve most of the practical problems. Maybe it would. Maybe not.

But by the time I got here, this system was finally starting to answer a very current and very central question seriously:

**can multimodal understanding enter the control loop online and directly change what the robot does next**

If the answer is yes, then this is no longer just "a robot feature pack with some AI inside."
It starts entering the discussion space of embodied AI.

That is what this chapter is really about.

Sixty-point version:
![alt text](assets/06_zero_shot_vlm_orchestrator_demo_01.gif)

Eighty-point version:
![alt text](assets/06_zero_shot_vlm_orchestrator_demo_02.gif)

## Let Me Be Precise First: Embodied AI Is Not Just One Thing Right Now

Before writing this chapter, I went back and looked again at the routes publicized by the big companies.
The more I looked, the more one thing became obvious:

today's embodied AI is not one single shape.

If you look at Google DeepMind's recent public work around `Gemini Robotics` and `Gemini Robotics-ER`, even they are already separating two rather different directions:

- one route that looks more like direct vision-language-action execution
- one route that looks more like embodied reasoning, meaning stronger spatial understanding and action reasoning over the current scene

If you look at Figure's `Helix`, that route is even more aggressive. It leans much closer to the dream of a unified end-to-end embodied system controlling the whole body. To be honest, like most people, I also hoped at first that AI could just perfectly command the robot body from top to bottom.

Then if you look at NVIDIA's `GR00T`, that feels more like a cross-embodiment robot foundation-model substrate.

So the actual frontier discussion today is no longer just a slogan like:

- "are you embodied AI or not"

It is much more like:

- what kind of embodied AI are you
- how much of your high-level understanding has really entered the control loop
- is your system one unified policy, or is it hierarchical orchestration

That matters a lot.
Because it determines how I should describe this chapter.

If I tried to oversell my own system as:

- **pure end-to-end**
- one single unified brain
- a full-body control foundation model

that would obviously be fake.

But if I describe it as:

**a hierarchical embodied agent where a VLM enters online control handoff**

then I think that claim can stand.

## So the Real Point of This Chapter Is Not "I Built End-to-End"

It is:

**I connected online embodied reasoning into a running robot system.**

That is the right way to understand `vlm_orchestrator`.

It is not Route A style object anchoring.  
It is not Route B style offline memory retrieval either.

What it does is more aggressive:

- look at the current scene
- combine that with the current task
- judge which direction is more worth exploring next
- judge when exploration should stop and locking should begin
- judge when control should be handed off to narrower and more precise modules

In other words, this route no longer depends mainly on:

- a prebuilt object-level semantic map
- a preorganized scene-memory database

It starts pushing the job of "understand the current scene and decide what to do next" into the online loop.

To me, that already looks very close to one clear direction inside today's embodied-AI discussion.
Not the most idealistic route. Not the most neural-network-myth route.
But definitely a very real route:

**orchestration-style embodied AI, or more plainly, hierarchical embodied orchestration**

## I Gradually Realized the Hard Part Is Not Making a VLM Look at an Image

It Is Making the VLM Sit in the Right Place Inside the System

That was probably the strongest feeling I got after looking back at my own code.

If all you want is a demo, you can absolutely write something like:

- grab one image
- call a VLM once
- let it say left / right / forward
- make the robot move a bit

That already looks kind of like "AI is controlling a robot."
To be honest, I started there too. I also tried that direction first.
The result, unsurprisingly, was a confused idiot that wandered around like a headless fly.

The moment you actually let the robot run, you immediately discover that the hard part is not "can it look at an image."
The hard part is:

- model outputs jitter
- the robot blindly obeys and spins in place
- task efficiency becomes terrible
- API latency is real
- one frame is not stable enough
- continuous control cannot wait for the cloud
- near-field approach errors get amplified very quickly
- once the target is already clear, keeping the VLM in charge actually makes the robot dumber

So in the end, I did not treat the VLM as an all-purpose brain.
I treated it as a **high-level control handoff module**.

And that, to me, is the part that really gives this chapter the right to lean into the embodied-AI conversation.
Because embodied intelligence here stops being just "it recognized something."
It starts showing up as:

**who gets to take over the robot, and when**

## My `vlm_orchestrator` Is Basically an Embodied State Machine

I think this point deserves extra emphasis.

The valuable part is not whether the prompt looks pretty.
The valuable part is that it is a fairly complete hierarchical state machine.

The main states look roughly like this:

- `IDLE`
- `QUICK_CHECK`
- `ASYNC_EXPLORE`
- `PRE_SCAN_360`
- `OPTION_EXECUTE`
- `VLM_CHECK`
- `ALIGNMENT`
- `BLIND_DRIVE`
- `YOLO_APPROACH`
- `GRASP_EXECUTE`

Once these states are connected together, the system is no longer a "one model node plus one action server" setup.
It starts feeling more like I hand-built a small brain.

It is much closer to an actual embodied control graph than to AI-flavored slot-machine robotics.

And I think that is the part that most deserves to be seen in this chapter:

**embodied AI does not have to begin as one giant unified policy network. It can also begin as a control system where high-level semantics have already entered the loop.**

## Later I Did Another Very Engineering-Like Thing That Still Felt Very Embodied to Me

I Took This Brain Apart Properly

At first, `vlm_orchestrator.py` was a strong and very real brain.  
It handled almost everything:

- looking at images
- choosing frontiers
- deciding when to switch state
- deciding when to use Nav2
- deciding when to let YOLO take over
- deciding when to trigger the grasp stage

That style has one very big advantage:
**you can get a live system running quickly.**

But it also has a cost that becomes clearer and clearer:

**as soon as one piece of logic gets complicated, the coupling of the whole system rises very fast**

You change exploration, and you may break approach.  
You change approach, and you may break state transitions.  
You patch one local corner case, and suddenly the personality of the entire control loop changes.

So later I made an upgrade that I am personally very happy with.
I did not throw away the original orchestrator. I kept it as the fallback working system.
At the same time, I added a fully decomposed architecture:

**ROS 2 Components + Composition + a lightweight BT mission flow**

That takes the old big brain and breaks it into modules with much clearer responsibilities:

- `Perception`
- `FrontierPlanner`
- `VLMReasoner`
- `NavigationExecutor`
- `Alignment`
- `Approach`
- `YoloApproach`
- `Grasp`
- `LifecycleSupervisor`

Then on top of that, a very light BT handles only the high-level stages:

- `EXPLORE`
- `ALIGN`
- `APPROACH`
- `YOLO_APPROACH`
- `GRASP`
- `DONE`

![alt text](assets/06_zero_shot_vlm_orchestrator_figure_01.png)

The more I thought about it, the more I felt that this step was also deeply embodied.
Not because it makes the AI look more godlike.
But because it makes the system look more like a real body.

A body is not one organ doing everything.
It is many subsystems cooperating:

- some are responsible for seeing
- some are responsible for thinking
- some are responsible for moving
- some are responsible for fine correction
- some are responsible for the final grasp

And those subsystems are not connected randomly either.
They communicate through standard ROS messages, services, and lifecycle states. Who should be active, who should sleep, and who should take over are all written down explicitly.

That matters a lot to me.
Because it pushes this route from "a very capable demo" toward an **embodied architecture that could actually be open-sourced, extended, and picked up by other people later**.

![alt text](assets/06_zero_shot_vlm_orchestrator_figure_02.png)

Though one small note: I still only plan to seriously think about open-sourcing after around September 2026.

In other words, the earlier `vlm_orchestrator` was more like the first working brain I hand-built.
The later decoupled version was the first time I seriously turned it into a **body with organs and division of labor**.
Of course, once you decouple everything, bugs also come rushing in. Getting that architecture truly stable still takes real work.

## State 1: When a Task Comes In, the System Does `QUICK_CHECK` First

It Does Not Start Performing Immediately

This is a tiny detail, but it really feels like something only people who have actually built systems bother adding.

When the user enters something like:

- `grab the blue cube`
- `find the red cube`
- `hey leo, grab the coke behind you`

the system does not immediately start spinning in place, and it does not immediately rush off.
It first does the most ordinary and most sensible thing:

**look at the current scene first**

If the target is already visible, then the whole large exploration routine is unnecessary.

There is also one detail here that I really like.
The system refreshes the target classes for the current task based on the task text itself.

If the user says blue, red, green, yellow, or box, the lower-level visual filtering updates accordingly.

What does that tell you?

It means the system is no longer a dead perception stack.
The high-level language task is already rewriting what lower-level perception pays attention to.

That already has a very embodied-AI flavor of **task-driven perception**.

## State 2: If One Look Is Not Enough, It Enters `ASYNC_EXPLORE`

But It Does Not Stop and Wait for the Cloud Brain

This is the first part of the chapter that really starts feeling agent-like to me.

Many people imagine a multimodal robot like this:

look at one frame -> let the model think -> output one action -> the robot does one move -> then look at the next frame

That is not what I built here.

What I actually built is an **asynchronous copilot**.

The robot keeps moving.  
The base keeps driving.  
At the same time, `copilot_loop` runs in another thread and keeps asking a lighter VLM about the current image:

- is this direction low-potential
- should we keep going forward, or bias left, or right, or u-turn
- is it possible that the target is actually already in view

And that result does not brutally overwrite the controller.
It behaves more like a high-level bias term that keeps nudging exploration.

To me, that feels very close to the truly valuable part of many frontier systems today.
Not "give every timestep to the large model."
But **let the large model become a high-level copilot while the robot is in motion**.

That said, this was also one of the parts where I stepped on the most landmines.
The earliest version was "asynchronous in name, blocking in spirit":
once the request went out, the control loop still got dragged down, which meant the robot was still moving while the next decision was based on a stale frame. Then you would see weird things in the terminal: the VLM says "right," but the base seems to be executing the ghost of the previous "left."

Later I made two changes that do not look very dramatic, but were absolutely life-saving:

- force a `full stop` before the VLM request, and add a stationary gate before image capture becomes valid
- once the system falls into physical fallback branches like `manual_drive`, immediately discard old async results so stale commands cannot leak across states

After those two cuts, the feeling of "robot lottery" dropped for the first time in a noticeable way.

## But the Copilot Does Not Get Veto Power

You Gave It a Lot of Inertia Constraints

This is where I think the line becomes much more mature.

If the VLM says once that "the left looks more promising," and the robot immediately jerks left, that system is not going to survive long.
So I added a set of thresholds in code:

- `semantic_low_score_threshold`
- `semantic_low_streak_trigger`
- `inertia_min_travel_m`

In plain language, that means:

- the model has to feel consistently bad about the current direction for multiple rounds
- the robot also has to have already traveled far enough in the current direction
- only then is a real switch allowed

This matters a lot.
Because a real embodied system is not just "the model has opinions."
It has to manage the tension between model opinions and physical inertia.

To me, this is already starting to look a little embodied in the strong sense.
It starts to look like the system **kind of has a way of thinking**, even if that thinking is still highly engineered.

## State 3: When Necessary, It Enters `PRE_SCAN_360`

It Actively Creates a More Complete Perception Round

If `ASYNC_EXPLORE` is the lightweight continuous mode, then `PRE_SCAN_360` is the more deliberate mode of this route.

The system:

- rotates in place
- samples multiple frames in a paced way
- records for each frame:
  - the current sector
  - front distance
  - scan openness
  - visual novelty
  - current YOLO brief
- then uses the full sweep to do sector-level planning

I think this step is beautiful.
Not because it is flashy, but because it reflects one very important embodied-AI habit:

**if the current observation is not enough, do not guess harder, look more first**

At that point it is not just perception anymore.
It is active perception.

And I still did not lock the whole system to the model here.
If the `VLM sector plan` is unavailable, the system drops back to code-based frontier fallback.

So the design is:

- the model can help with higher-level semantic judgment
- but the rule-based system can still stay alive on its own

To me, that is exactly what a realistic and deployable embodied route looks like today.
Not "nothing works without the model."
But "the model improves the system's judgment at the right moments."

![alt text](assets/06_zero_shot_vlm_orchestrator_figure_03.png)

## State 4: The Part That Really Made Me Feel This Route Entered the Embodied-AI Discussion

Was the `beacon` Logic

I will say this directly.
This is one of the most exciting parts of the chapter.

Because it is no longer just:

- left or right
- forward or turn

What it is really doing is something much closer to agent-style working memory.

The system maintains:

- `memory_buffer`
- `visited_pose_counts`
- `blocked_pose_counts`
- recently attempted frontiers
- frontiers that already failed
- the current preferred sweep direction

That means that even though this agent does not have a long-term scene memory bank like Route B, it already has **short-term working memory**.

And that memory is not narrative memory like "I remember this room."
It is action memory:

- which side has already been tried
- which side keeps getting blocked
- which places are getting repetitive
- which side still feels like new territory

That is why I think `beacon` is such a good name.
It is not an all-knowing brain.
It is more like a local action-oriented guidance system that keeps updating as exploration continues.

There was also a very real lesson from actual testing here.
If you rely on visual semantics alone, the model can misread floor highlights or near textured walls as either "promising explorable area" or "complete dead end." Both happen. The first makes the robot rush nonsense, the second makes it spin in place.

So later I turned `beacon` into a hybrid of **semantic suggestion plus physical veto**:

- candidate directions first go through LiDAR / Costmap safety filtering
- repeated `ROTATE` loops or repeated empty candidates trigger hard escape logic instead of endless AI queries
- rotation is no longer fixed to one side either; openness or random escape behavior is used to break local loops

If you put that back into today's embodied-AI discussion, it maps very cleanly to one important idea:

**an agent does not need a giant long-term world model first. It can already become interesting once it has strong online working memory and control handoff ability.**

## Frontier Selection Here Is Also Not "Let the VLM Decide Everything"

Geometry, Memory, and Semantics Are Negotiating

This is something I especially want to emphasize.

Because once people start writing about AI controlling robots, there is always a temptation to make the model the only protagonist.
But that is exactly what I did not do here.

I first build candidate frontiers.
Then I combine:

- geometric traversability
- distance
- openness
- novelty
- blockage
- revisit penalty
- inertia
- semantic bias

So the VLM is not coming in at the top and ruling over everything.
It only truly enters final decision-making when several directions are all still plausible and the system needs a higher-level semantic tie-breaker.

There is another important tradeoff behind this design too:
I did not keep extending the prompt forever.
I made the prompt shorter and more structured, and I pushed actual stability into the state-machine boundaries instead.

Because the side effect of very long prompts is brutally direct:
cloud timeout risk goes up, and once timeouts pile up, the control layer is forced to degrade more often, which makes the whole system feel more like gambling again.

So the final design became:

- the prompt keeps only the core candidate fields like distance, visited count, aversion, and the necessary embodied summary
- high-risk decisions like "is this traversable, when should we back off, when should we stop rotating" remain in the local rule layer

In other words, the robot's bodily feelings are being fed into its higher-level brain.
That looks a lot like the most practical path in many current frontier systems:

- low-level geometry and safety stay hard
- high-level semantics choose the more meaningful option among several valid ones

## So, Is This Embodied AI or Not

This is the point where I think there is no need to pretend.

If you define embodied AI very narrowly as:

- one giant end-to-end policy
- one unified brain
- one foundation model that directly controls the whole body

then no, this is not that.

There is no need to fake it.

But if you define embodied AI through a more essential question:

**has high-level multimodal understanding started deciding online what a physical robot does next, and has that decision entered real control flow instead of staying at the level of text**

Then the answer for this route is very clear:

**yes.**

It is obviously not the purest possible form of embodied AI.
But it is already a very clear embodied-agent prototype, and I think it is worth discussing exactly for that reason.

It belongs to a more realistic, more engineering-heavy, more hierarchical frontier route.
It is not a unified nervous system.
But it is real online embodied control orchestration.

Of course, arriving at that answer also required me to first convince myself.

## When the Whole Route Runs End to End, the System State Really Looks Like This

I still want to compress the full workflow once more, because the biggest risk in this chapter is turning it into concept marketing.

The full state chain looks roughly like this:

1. the user enters a natural-language task
2. the system refreshes the active target-class filters based on the task
3. it performs `QUICK_CHECK` first to see whether the current view already contains a usable target cue
4. if not, it enters `ASYNC_EXPLORE`, where the robot keeps moving while the asynchronous VLM copilot periodically produces high-level exploration bias
5. if a more formal decision round is needed, it enters `PRE_SCAN_360` and actively gathers multi-frame, multi-sector observations
6. the system combines frontier geometry, local working memory, failure history, and semantic bias to choose the next direction
7. when `Nav2` is stable enough, it is used to send the robot to local frontier staging points; when it is not, the system falls back to manual drive
8. once the target starts becoming clear either through current vision or VLM judgment, preemption triggers and the system switches into `ALIGNMENT`
9. in `ALIGNMENT`, the body is oriented properly toward the target
10. then `BLIND_DRIVE` eats up a chunk of near-range distance under LiDAR and progress-watchdog protection
11. once YOLO's stable 2D / 3D signal becomes strong enough, control switches into `YOLO_APPROACH`
12. YOLO visual servoing handles the final local approach and publishes the grasp target
13. the arm executes the grasp and the task is done

If you line up those thirteen steps, what this chapter actually built is not "one model is very smart."
It is:

**how an online embodied agent organizes current perception, local memory, rule-based safety, classical navigation, and local visual closed loops into one continuous execution chain**

To me, that is already enough to put it inside the embodied-AI conversation.

![alt text](assets/06_zero_shot_vlm_orchestrator_figure_05.png)

## A Final Note on This Chapter

If Route A was the first time the robot knew **what exists on the map**,  
and Route B was the first time the robot started **remembering what it had seen**,  
then Route C is the first time the robot starts doing this:

**look at the current scene and decide how the body should move next**

That is why I treat it as the red-carpet part of the whole series.

Not because it looks the most like science fiction.
But because it touches the hottest and hardest-to-ground layer in today's embodied-AI discussion:

- online multimodal understanding
- active perception
- high-level control handoff
- dynamic relay between modules
- continuous transition from reasoning to physical action

It is obviously not a world-class unified policy model.
And it obviously does not represent the final form of anything.

But it is standing very clearly on a direction that I think is both frontier-level and worth digging into further:

**embodied AI does not necessarily have to begin from pure end-to-end control. It can also begin from an embodied agent that can think online, take over online, and hand off online.**

And this route is my own serious and very engineering-heavy answer to that direction.

One last thing I want to say:
when you work on problems like this, you keep asking yourself the same question again and again. Does this really work? Is this route actually open?

And if I answer that honestly for this specific project right now, the answer is still no.
There are still lots of bugs. There are still moments where the system is basically rolling dice. There are still plenty of unreasonable decisions.

But those few successful runs already got it to the 80-point zone for this exploratory question in my own head.
And once it has succeeded even once, that already proves something important:

it has a real chance of succeeding again.
