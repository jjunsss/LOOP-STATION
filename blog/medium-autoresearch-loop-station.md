# LOOP-STATION: Turning Frontier Models Into Long-Running Research Collaborators

![LOOP-STATION workflow](../assets/loop-station-flow.png)

I have been thinking seriously about one question recently:

> If frontier models are already good enough to read code, inspect logs, compare outputs, and argue about next steps, why are we still using them mostly as one-shot assistants?

The thought became sharper after reading and thinking about **AutoResearch**-style systems. The interesting part was not just "AI writes research code." That is already impressive, but also slightly easy to misunderstand.

The more important idea, at least for my own work, was this:

> Research is not a single prompt. Research is a loop.

You run something.  
It fails in a weird way.  
You inspect metrics.  
You look at images.  
You compare with previous runs.  
You suspect one direction, then realize the real issue is somewhere else.  
You change code, configs, or the experimental frame.  
Then you do it again.

That loop is where most of the work happens.

And honestly, that loop is also where a lot of fatigue happens.

## The Problem With Normal Automation

In machine learning research, we already have plenty of automation tools.

Hydra configs, grid search, sweep scripts, bash launchers, experiment trackers, and so on.

They are useful. I use them too.

But they usually operate inside a space that I define in advance.

```text
try lr = [1e-3, 5e-4, 1e-4]
try lambda = [0.1, 0.3, 0.5]
try densify_until = [5k, 10k, 15k]
```

This is powerful when I already know the right axis to search.

But research often does not feel like that.

Sometimes the issue is not a hyperparameter. Sometimes the problem is hidden in a script, a metric, a rendering artifact, a dataset assumption, a logging bug, or a misleading visual trend.

In that situation, a preset sweep can keep running faithfully while missing the real point.

That was the gap I wanted to attack.

## What I Wanted Frontier Models To Do

I did not want an agent that only runs commands.

I wanted something closer to a research collaborator that can stay inside the loop:

- run experiments
- inspect outputs
- compare metrics and images
- read code changes
- summarize trends
- question weak explanations
- ask for review from another model
- decide the next experiment from evidence

Not perfectly, of course.

But meaningfully enough that the loop keeps moving while I sleep, eat, or do other work.

That became the motivation behind **LOOP-STATION**.

## The Core Idea

LOOP-STATION is built around a simple pattern:

```text
Executor runs
=> Executor writes results and self-review
=> Reviewer waits until the session is ready
=> Reviewer reviews the artifacts
=> Executor consumes the review
=> Executor plans the next session
=> repeat
```

In my setup, **Codex** usually acts as the executor/supervisor.

It runs the experiment, writes reports, checks metrics, inspects logs, prepares next-session candidates, and makes the final decision for the next run.

**Claude Code** acts as an external reviewer.

It does not need to run the main experiment. Instead, it waits, reads the artifacts, challenges the interpretation, and writes a review. If needed, it can focus on visual quality, metric trends, code/config audit, or even literature/method checks.

The important part is that they do not talk through vibes.

They coordinate through files.

LOOP-STATION creates a `loop_station/` workspace where agents leave reports, reviews, decisions, summaries, and flags. The flags are not meant to be fancy. They are just a practical way to say:

```text
I am running.
I am done.
It is your turn.
I read your review.
Here is the next session.
```

That small amount of structure matters a lot.

Without it, agents easily interrupt each other, review incomplete results, or start writing conclusions before the experiment is actually finished.

I learned this the annoying way.

## What Happened In Practice

The first serious use case was a research-quality improvement loop.

I used LOOP-STATION for a 50-session run, focused on improving the rendering quality of one subject. The metric I tracked most clearly was **PSNR**, an explicit rendering quality metric.

In practice, the run improved one subject from roughly **PSNR 24 to ~29 within about 10 hours**.

![LOOP-STATION runtime example](../assets/loop-station-runtime-example.svg)

This was not a magical "press one button and publish a paper" moment.

It was much more grounded than that.

The agents tried directions, wrote summaries, identified weak trends, retired bad axes, and shifted to new ones. Some sessions were useful. Some were not. Some looked promising in a metric but failed visually. Some visual improvements were not reflected cleanly by the first metric.

That is exactly why I wanted a loop.

If a direction gets worse, the system should not simply stop.

It should understand that the current axis is weak, return to the best known candidate, and try a different axis while budget remains.

That sounds obvious when written down, but it is easy for agents to get this wrong unless the workflow says it clearly.

## The Token Story

One unexpected part was how much more work I could push through by letting the loop run continuously.

With a Pro subscription, I was able to use far more tokens than I normally spend in ordinary interactive usage. Codex kept running, and Claude stayed in review standby through Monitor.

![LOOP-STATION token usage example](../assets/loop-station-token-usage-example.svg)

That changed the way I thought about agent usage.

Instead of treating the model as a short chat window, I started treating it more like a long-running research process.

The key is not just token volume.

The key is making sure the tokens are spent on durable artifacts:

- session reports
- metric summaries
- review notes
- decisions
- compact handoffs
- next-session plans

If the agent spends 10 hours working but leaves no usable memory, the loop is not useful.

If it leaves a clean research trail, the loop becomes much easier to resume, audit, and redirect.

## Why I Made It As A Skill

After running this manually for a while, one thing became very clear:

Coordinating agents is more exhausting than it looks.

Codex may finish a session before Claude notices.  
Claude may start reviewing too early.  
One agent may write a decision before the other review is complete.  
A monitor may be duplicated.  
The context may compact before the important summary is written.  
The user may need to intervene and redirect the axis.

None of these are conceptually hard.

But they are annoying enough that they should not be re-explained every time.

So I packaged the workflow as a reusable skill:

**LOOP-STATION**  
<https://github.com/jjunsss/LOOP-STATION>

It is designed to be used from Codex and Claude Code. The goal is not to provide one optimizer that works for every project. The goal is to provide a practical protocol for long-running agent loops:

- who runs
- who reviews
- where artifacts go
- how agents wait for each other
- how summaries survive compaction
- how the next session is planned from evidence

You still need to give project-specific goals, metrics, paths, and permission rules.

But once those are clear, the agents can keep the loop alive much more reliably.

## Closing Thought

I do not think this replaces research judgment.

If anything, it makes research judgment more important.

The user still needs to define the goal, inspect the direction, decide what matters, and intervene when the loop starts optimizing the wrong thing.

But frontier models are now strong enough that I do not want to use them only as autocomplete.

I want them inside the research loop:

running, reading, reviewing, doubting, summarizing, and trying again.

That is the spirit behind LOOP-STATION.

And this is the direction I want to keep exploring.
