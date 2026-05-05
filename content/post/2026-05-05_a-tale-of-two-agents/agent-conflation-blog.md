---
title: “A Tale of Two \"Agents\""
date: 2026-05-04T09:00:00+08:00
subtitle: "A robotics researcher's plea for precision"
tags: ["robotics", "LLM", "AI", "MARL"]
draft: true
---

# Two Things Called "Agent"

*A robotics researcher's plea for precision*

---

If you go to an embodied AI seminar this month, you will probably hear the phrase "multi-agent system" used twice within the same hour — once to describe a team of quadrupeds learning a soccer policy with QMIX, and once to describe a swarm of GPT-4 instances spawned by AutoGen to write a software project.

The two settings share almost nothing beyond the noun. And yet the noun is doing all the work. Conferences, grant calls, internal roadmaps, and breathless tech-press articles routinely flatten them into a single object of study. The result is a steady leak of claims, benchmarks, and engineering arguments across a category boundary that should not be crossed.

I want to make that boundary explicit, because the conflation is causing real damage to how we frame robotics research.

---

## The core distinction: sub-process vs. body

Here is the argument in one sentence:

> **An LLM agent is a sub-process. A robot agent is a body.**

That's the whole thing. Everything else falls out of it.

An LLM agent — the kind you build with ReAct, AutoGen, CrewAI, MetaGPT — is a function call wrapped in a control loop. It has a role, a context window, and maybe some tools. You can spawn a hundred of them in a `for` loop. You can kill them when the script ends. They have no body, no dynamics, no real-time clock pressing on them. The environment they inhabit is text, code, or an API. When they "fail," they hallucinate or get stuck in a chatter loop; nothing breaks.

A robot agent — the kind we mean when we say MARL, Dec-POMDP, or just "a quadruped" — is bound to a physical body. Its action space is torques and joint velocities at hundreds of Hz. Its observations are sensor streams under hard latency budgets. The world keeps moving whether or not the policy outputs anything. When it fails, it falls over.

You cannot `range(n)` your way to ten quadrupeds. Each one is capital expenditure, weight, power, calibration, a safety case, and a sim-to-real cycle.

This is not a minor difference of scale or implementation. It is an ontological difference. One kind of agent is a *process*; the other is an *entity*. They share a noun by historical accident — both lineages trace back to Russell and Norvig's perceive-decide-act abstraction — but the LLM community kept the abstraction and dropped the situatedness, while the robotics community kept the situatedness and dropped the assumption that the agent is just software.

---

## Where the difference shows up

Once you see the distinction, you start noticing it everywhere. A few of the axes that matter most in practice:

| | **LLM agent** | **Robot agent** |
|---|---|---|
| Substrate | Tokens | Physics |
| Action space | Tool calls, text emission | Continuous torques/velocities |
| Time | Turn-based, seconds | Real-time, 50–1000 Hz |
| Identity | Ephemeral, spawnable | Persistent, tied to hardware |
| Failure | Hallucination, infinite loops | Collision, damage, instability |
| Communication | Natural language over a bus | Implicit (shared environment) or low-bandwidth comms |
| Scaling bound | API cost, context length | Hardware cost, weight, power |

The most consequential axis, in my view, is **time**. An LLM agent that pauses for 60 seconds is slow. A robot agent that pauses for 60 seconds has fallen over. You cannot paper over this gap with engineering. It dictates which architectures are even admissible.

The second-most consequential axis is **scaling**. When a paper claims to "scale multi-agent emergent coordination," ask what kind of system. Scaling LLM-MAS from 5 to 50 agents is a software-engineering problem. Scaling MARL to 50 quadrupeds is a sim-to-real, comms-bandwidth, and procurement problem. The lessons do not transfer.

---

## The "skill" problem

Of all the places this conflation does damage, the worst offender is the word **skill**.

In reinforcement learning, a skill is a precise object. Sutton, Precup, and Singh formalized it in 1999 as an *option*: a triple of an initiation set, an internal policy, and a termination condition. A temporally extended action that the agent commits to executing for many primitive timesteps. Eysenbach's DIAYN, Sharma's DADS, and the whole unsupervised skill discovery literature build on this — a skill is a learned controller fragment π(a | s, z), conditioned on a latent z, judged by what it produces in the environment.

A skill, in this sense, has four defining properties:
1. It is **temporally extended** — it spans many control steps.
2. It is **learned from interaction** — shaped by reward, intrinsic or extrinsic.
3. It has a **termination condition** — also typically learned.
4. It is **closed-loop on physics** — its meaning is what it does to the world.

In LLM agent frameworks, "skill" means something completely different. In Anthropic's Claude Skills, in OpenAI's custom GPTs, in most agentic-workflow tools, a skill is a **prompt template** or an `.md` instruction file. You retrieve it by embedding similarity over its description and paste it into a context window.

I want to say this clearly: a markdown file is not a skill. It is an instruction. It has no termination condition. It is not learned from interaction. It does not close a loop with physics. Calling it a skill borrows the prestige of a precise RL concept to dress up what is, fundamentally, a glorified prompt.

This is not pedantry. When someone says "our system has a library of 200 skills," that claim could mean either (a) 200 learned options with empirically validated initiation sets, or (b) 200 markdown files in a folder. These are not in the same league of difficulty, generality, or research contribution. Conflating them flattens a genuine technical advance into vibes.

The interesting case is the genuine middle ground — Voyager's library of GPT-4-emitted JavaScript that calls Mineflayer APIs, or Code-as-Policies in an accumulating mode. There the LLM emits *executable code* that closes a loop with a (simulated) environment. That's closer to an options library than a prompt library, and the term "skill" is more defensible. But the bar should be: **does this thing close a sensorimotor loop, or doesn't it?** If it doesn't, call it a tool, a procedure, or a prompt. Not a skill.

---

## The hybrid middle ground

I want to be fair: most interesting current work in robotics + foundation models lives in the middle, not at either pole. The conflation problem isn't that hybrids exist — it's that researchers fail to specify *which layer* the LLM is operating at and *which layer* is doing the embodied work.

A quick taxonomy of the hybrids worth being careful about:

**VLA models (RT-2, OpenVLA-OFT, RDT-1B, π0).** These are unambiguously embodied agents. They output torques or end-effector deltas. They close a real-time loop. The fact that they have a transformer backbone and consume language as input does not make them "LLM agents" any more than a CNN-based policy is a "vision agent." Language is an *input modality*, not the action substrate. What makes a VLA work in the real world is the action representation (flow matching, chunked diffusion, discretized tokens) and the careful 50 Hz engineering — not the language pretraining alone.

**LLM-as-planner with embodied executors (SayCan, Code-as-Policies, VoxPoser, PaLM-E).** This is the dominant architecture for long-horizon embodied tasks today. The LLM operates at the seconds-to-minutes timescale, emitting symbolic or geometric outputs. A learned or classical low-level controller executes in real time. Calling the *whole pipeline* "an agent" is fine as system-level shorthand. Calling the LLM and the robot "the same kind of agent" erases the architectural decisions that make it work.

**LLM-driven multi-robot coordination (RoCo, CoELA, SMART-LLM).** Here the conflation pressure is at its peak, because the LLMs are doing inter-agent communication and the robots are physical. Two things are happening at once: LLMs are roles in a software pipeline (planner, communicator, validator), and robots are bodies executing the resulting plans. When these systems report success rates, those rates measure the *combined system*. They are not evidence that "LLM agents have solved multi-robot coordination." The embodied execution layer is doing the heavy lifting on contact, dynamics, and real-time control.

**LLM-as-reward-designer (Eureka, Text2Reward).** This is one of the cleanest places for the two communities to meet. The LLM emits candidate reward functions in code; an RL training process optimizes them. The agent that runs on the robot is unambiguously RL. The LLM is an offline tool for the engineer. No conflation, no confusion.

**LLM-as-policy directly.** This is the one that fails — and fails for predictable reasons. Inference latency of seconds is incompatible with control rates above 1 Hz. The action space is poorly matched to continuous control. The lack of a learned dynamics model means LLM-emitted joint commands are ungrounded. Where it appears to work, it is for very slow, highly constrained tasks — and VLAs decisively outperform precisely because they add proper action heads and regression objectives.

---

## Strategy and orchestration

This is where my own work sits, so I'll be brief but pointed.

In MARL, *strategy* has a game-theoretic backbone — pure and mixed strategies, best response, population-based training (PSRO, AlphaStar's League). Hierarchical strategy is the natural composition of strategy with skills: a meta-policy selects which option to execute; the option governs primitive actions for some duration. Recent diffusion-based MARL work (MADIFF, graph diffusion methods, projected diffusion for MAPF) generates *coordinated trajectories* by jointly modeling agent behavior in state space.

In LLM-MAS, "strategy" is rarely formalized. A planner LLM emits a natural-language plan; worker LLMs parse and execute it. There is no equilibrium notion, no best-response computation, no population. The "meta-policy" is a system prompt.

Both can be called "orchestration," but they are structurally different objects. An LLM orchestrator runs at 0.1–1 Hz, is human-readable, relies on pretraining + few-shot prompts, and fails by hallucinating plans that violate affordances. An RL meta-policy runs at 1–10 Hz, is a black-box neural network, requires population rollouts under non-stationary partners, and fails by overfitting to its training distribution.

Both have legitimate use cases. An LLM orchestrator makes sense when latency is permissive, interpretability is valuable, and the search space is combinatorially large but linguistically structured. An RL meta-policy makes sense when control rates exceed a few Hz, the task is poorly described in language, and there is abundant interaction data.

The key is to be explicit about which one you're proposing — and not slip silently between them mid-paragraph.

---

## Recommended language

If you only take one thing from this piece, take this set of distinctions:

- **LLM agent** — an LLM call wrapped in a control loop. Disembodied, ephemeral, spawnable. Use this rather than the unqualified "agent."
- **Embodied agent** or **robot agent** — an agent with a body, sensors, and actuators. Persistent, tied to hardware.
- **LLM-MAS** or **agentic LLM system** — multiple LLM agents (AutoGen, MetaGPT, ChatDev). Not the same as "multi-agent system" without qualification.
- **MARL system** or **embodied MAS** — multiple embodied agents in a stochastic game or Dec-POMDP.
- **LLM skill** or **prompt skill** or **tool definition** — a prompt template or function spec invoked via context insertion. Not an option.
- **RL skill** or **option** — a learned policy π(a | s, z) with initiation and termination conditions, judged by environmental return.
- **VLA policy** — a vision-language-action model used as a robot policy. Embodied, not an "LLM agent."
- **LLM orchestrator** vs. **RL meta-policy** — a planner LLM directing modules vs. a learned controller selecting options.

When in doubt, ask the diagnostic question: **does this thing have a body, dynamics, and a real-time control loop?** If yes, it is an embodied agent. If no, it is a process, role, or module.

---

## Why this matters

A naive reading of the current literature would conclude that "multi-agent systems" are a single rapidly-progressing field and that LLM-MAS frameworks like AutoGen are evidence of progress on multi-robot coordination. They are not. They are progress on a different problem that happens to use the same words.

Robotics groups that take the framing too literally will design architectures that assume cheap agent spawning (because LLMs are cheap to spawn) and discover too late that each "agent" is a $50,000 hardware platform with a 90-day lead time. Reviewers will misjudge claims because the noun has done their evaluation work for them. Students will spend years on what they think is the same problem and discover, mid-PhD, that the literature they have been reading is in a different ontological category.

The genuinely productive hybrid systems — VLAs, LLM-planner-plus-controller stacks, Voyager-style executable skill libraries, LLM-designed rewards, dialectic multi-robot coordinators — succeed precisely because they *respect* the distinction. They put LLMs at the layers where language, abstraction, and combinatorial search are valuable, and they put embodied policies at the layers where dynamics, real-time control, and sensorimotor coupling are non-negotiable. The mistake is not in building hybrids. The mistake is in pretending the layers are the same kind of thing.

So: keep using "agent." It's too useful a word to give up. But when you do, expect the follow-up question — and ask it of others.

*What kind of agent?*
