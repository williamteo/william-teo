---
title: A Tale of Two Agents
date: 2026-05-04T09:00:00+08:00
subtitle: "A robotics researcher's plea for precision"
tags: ["robotics", "LLM", "AI", "MARL"]
draft: true
citable: true
---

If you go to an embodied AI seminar this month, you will probably hear the phrase "multi-agent system" used twice within the same hour — once to describe a team of humanoids learning a soccer policy with QMIX[^1], and once to describe a swarm of GPT-5.5 instances spawned by AutoGen[^2] to write a software project.

The two settings share almost nothing beyond the noun. And yet the noun is doing all the work. Conferences, grant calls, internal roadmaps, and breathless tech-press articles routinely flatten them into a single object of study. The result is a steady leak of claims, benchmarks, and engineering arguments across a category boundary that should not be crossed.

I want to make that boundary explicit, because the conflation is causing real damage to how we frame robotics research.

---

## What this argument is, and what it isn't

Let me be careful here, because the easy version of this argument is wrong and I don't want to make it.

The easy version says: *LLM agents aren't really agents — only embodied robots are.* That's gatekeeping, and it's indefensible. Both LLM agents and embodied MARL agents satisfy the classical Wooldridge[^3] / Russell–Norvig criteria for agency: autonomy, reactivity, pro-activeness, social ability. A Reflexion[^4] agent that revises its own strategy based on self-critique is autonomous and goal-directed. A Generative Agent[^5] that plans over simulated weeks has persistence and pro-activeness. A trading agent that moves $10M through an API has consequential, real-world action. Agency does not require carbon, silicon, or servos.

So the argument is **not** that LLM agents are "merely" anything.

My argument is this: **agents in different media impose different engineering constraints, and the constraints don't transfer.** The robotics community and the LLM-agent community are both doing legitimate work on legitimate agents — but the engineering, the failure modes, the scaling bounds, the meaning of "skill," the meaning of "strategy," and the meaning of "emergent coordination" are *substantively different across media*. Sharing a noun makes us forget this.

That's the conflation I think is worth fighting against.

---

## The medium is the argument

The clean way to state the distinction:

> [!NOTE]
> **An LLM agent acts through tokens, code, and APIs.**\
> **A robot agent acts through physics.**

Both are agents. Both have goals, perception, and action. What differs is the *medium of action* and everything that medium dictates.

Tokens, code, and APIs are a forgiving medium. They are turn-based. They tolerate latency on the order of seconds. They can be undone, retried, sandboxed, and parallelized. You can spawn a hundred LLM agents in a `for` loop. You can kill them when the script ends. The world they act on is software, and software is patient.

Physics is unforgiving. It runs at hundreds of hertz whether you've decided what to do or not. Actions are torques, joint velocities, and end-effector commands; consequences are momentum, contact forces, and falling over. You cannot retry a collision. You cannot `range(n)` your way to ten quadrupeds — each one is capital expense, weight, power, calibration, a safety case, and a sim-to-real cycle.

This isn't an ontological hierarchy. A robot policy is "just" a function $\pi$(a|o), and from the right angle so is an LLM agent. The body isn't what makes the robot an agent; the body is what makes the *engineering* unforgiving. And unforgiving engineering is what the robotics community has spent fifty years learning to handle. That hard-won knowledge doesn't survive being flattened into the same noun as a Python script that calls GPT-5.5.

---

## Where the medium difference shows up

| | **LLM agent** | **Robot agent** |
|---|---|---|
| Medium of action | Tokens, code, API calls | Physics |
| Action space | Tool calls, text emission | Continuous torques/velocities |
| Time | Turn-based, seconds | Real-time, 50–1000 Hz |
| Reversibility | Often reversible | Irreversible |
| Failure | Hallucination, infinite loops | Collision, damage, instability |
| Communication | Natural language | Implicit (shared environment) or low-bandwidth comms |
| Scaling bound | API cost, context length | Hardware cost, weight, power |

The most consequential row is **time**. An LLM agent that pauses for 60 seconds is slow. A robot agent that pauses for 60 seconds has fallen over. You can't paper over this with engineering — it dictates which architectures are even admissible. If your "agent" needs five seconds to think, it cannot be the inner loop of a quadruped controller. Period.

The second-most consequential row is **scaling**. When a paper claims to "scale multi-agent emergent coordination," ask which medium. Scaling LLM-MAS from 5 to 50 agents is a software-engineering problem — config files, API quotas, orchestration overhead. Scaling MARL to 50 quadrupeds is a sim-to-real, comms-bandwidth, and procurement problem. The *lessons do not transfer*. A claim about emergent coordination among LLM roles is not evidence about emergent coordination among robots, and vice versa.

The third row that matters more than people admit: **reversibility**. Most LLM agent failures are recoverable — restart the loop, edit the prompt, roll back the file. Most robot agent failures aren't. This single asymmetry shapes everything from how you train (online RL is dangerous; sim-to-real exists for a reason) to how you deploy (safety cases, certified controllers, reachable-set analysis).

These aren't gradient differences. They're discontinuities in the engineering problem.

---

## The skill problem

The medium distinction is mostly polite. Here's where I get less polite.

In reinforcement learning, a **skill** is a precise object. Sutton, Precup, and Singh[^6] formalized it in 1999 as an *option*: a triple of an initiation set, an internal policy, and a termination condition — a temporally extended action that the agent commits to executing for many primitive timesteps. The unsupervised skill discovery literature — DIAYN[^7], DADS[^8], VALOR[^9], HMASD[^10] — builds on this. A skill is a learned controller fragment π(a | s, z), conditioned on a latent z, judged by what it produces in the environment.

Four defining properties:
1. **Temporally extended** — spans many control steps.
2. **Learned from interaction** — shaped by reward, intrinsic or extrinsic.
3. **Has a termination condition** — typically also learned.
4. **Closed-loop on its medium** — its meaning is what it does to the world.

In LLM agent frameworks — Anthropic's Claude Skills, OpenAI's custom GPTs, basically every agentic-workflow tool on the market — a "skill" is a **prompt template** or an `.md` instruction file. You retrieve it by embedding similarity over its description and paste it into a context window.

A markdown file is not a skill. It is an instruction.

It has no termination condition. It is not learned from interaction. It does not close a loop with anything. Calling it a skill borrows the prestige of a precise RL concept — built up over twenty-five years of hierarchical RL theory — to dress up what is, fundamentally, a glorified prompt.

This isn't pedantry. When someone says "our system has a library of 200 skills," that claim could mean either (a) 200 learned options with empirically validated initiation sets and termination conditions, with hierarchical-RL guarantees, or (b) 200 markdown files in a folder. These are not in the same league of difficulty, generality, or research contribution. Conflating them flattens a genuine technical advance into vibes.

The interesting case is the genuine middle — Voyager[^11]'s library of GPT-4-emitted JavaScript that calls Mineflayer APIs, or Code-as-Policies[^12] in an accumulating mode. There the LLM emits *executable code* that closes a loop with a (simulated) environment. That's closer to an options library than a prompt library, and "skill" is more defensible. But the bar should be: **does this thing close a loop on its medium, or doesn't it?** A skill in any meaningful sense is something whose semantics live in the *consequences of execution*, not in the words used to describe it. If the only thing your skill does is get pasted into a prompt, you have a procedure, a tool, or a template. Not a skill.

I am willing to die on this hill.

---

## The hybrid middle ground

To be fair: most interesting current work in robotics + foundation models lives in the middle, not at either pole. The conflation problem isn't that hybrids exist — it's that researchers fail to specify *which layer* the LLM is operating at and *which layer* is doing the embodied work.

A quick taxonomy:

**VLA models (RT-2[^13], OpenVLA-OFT[^14], RDT-1B[^15], π0[^16]).** These are unambiguously embodied agents. They output torques or end-effector deltas. They close a real-time loop. The fact that they have a transformer backbone and consume language as input does not make them "LLM agents" any more than a CNN-based policy is a "vision agent." Language is an *input modality*, not the action medium. What makes a VLA work in the real world is the action representation (flow matching, chunked diffusion, discretized tokens) and the careful 50 Hz engineering — not the language pretraining alone. Don't be fooled by the architecture diagram.

**LLM-as-planner with embodied executors (SayCan[^17], Code-as-Policies[^12], VoxPoser[^18], PaLM-E[^19]).** This is the dominant architecture for long-horizon embodied tasks today. The LLM operates at the seconds-to-minutes timescale, emitting symbolic or geometric outputs. A learned or classical low-level controller executes in real time. Calling the *whole pipeline* "an agent" is fine as system-level shorthand. Calling the LLM and the robot "the same kind of agent" erases the architectural decisions that make it work.

**LLM-driven multi-robot coordination (RoCo[^20], CoELA[^21], SMART-LLM[^22]).** Here the conflation pressure peaks, because the LLMs are doing inter-agent communication and the robots are physical. Two things are happening at once: LLMs are roles in a software pipeline (planner, communicator, validator), and robots are bodies executing the resulting plans. When these systems report success rates, those rates measure the *combined system*. They are not evidence that "LLM agents have solved multi-robot coordination." The embodied execution layer is doing the heavy lifting on contact, dynamics, and real-time control. Read the experiments before you read the abstract.

**LLM-as-reward-designer (Eureka[^23], Text2Reward[^24]).** This is one of the cleanest places for the two communities to meet. The LLM emits candidate reward functions in code; an RL training process optimizes them. The agent that runs on the robot is unambiguously RL. The LLM is an offline tool for the engineer. No conflation, no confusion.

**LLM-as-policy directly.** This one fails — and fails for predictable reasons. Inference latency of seconds is incompatible with control rates above 1 Hz. The action space is poorly matched to continuous control. The lack of a learned dynamics model means LLM-emitted joint commands are ungrounded. Where it appears to work, it is for very slow, highly constrained tasks — and VLAs decisively outperform precisely because they add proper action heads and regression objectives. If your architecture has GPT-4 in the inner control loop, you are doing it wrong.

---

## Strategy and orchestration

This is where my own research sits, so I'll be brief but pointed.

In MARL, *strategy* has a game-theoretic backbone — pure and mixed strategies, best response, population-based training (PSRO[^25], AlphaStar[^26]'s League). Hierarchical strategy is the natural composition of strategy with skills: a meta-policy selects which option to execute; the option governs primitive actions for some duration. Recent diffusion-based MARL work (MADIFF[^27], graph diffusion methods, projected diffusion for MAPF) generates *coordinated trajectories* by jointly modeling agent behavior in state space. There's mathematical content here — convergence guarantees, equilibrium notions, sample complexity bounds.

In LLM-MAS, "strategy" is rarely formalized. A planner LLM emits a natural-language plan; worker LLMs parse and execute it (MetaGPT[^28], ChatDev[^29], AutoGen[^2]). There is no equilibrium notion, no best-response computation, no population. The "meta-policy" is a system prompt. This is fine — sometimes a system prompt is exactly what the problem needs — but it should not be mistaken for the same kind of object.

Both can be called "orchestration," but they are structurally different. An LLM orchestrator runs at 0.1–1 Hz, is human-readable, relies on pretraining + few-shot prompts, and fails by hallucinating plans that violate affordances. An RL meta-policy runs at 1–10 Hz, is a black-box neural network, requires population rollouts under non-stationary partners, and fails by overfitting to its training distribution.

Both have legitimate use cases. An LLM orchestrator makes sense when latency is permissive, interpretability is valuable, and the search space is combinatorially large but linguistically structured. An RL meta-policy makes sense when control rates exceed a few Hz, the task is poorly described in language, and there is abundant interaction data.

The key is to be explicit about which one you're proposing — and not slip silently between them mid-paragraph.

---

## Recommended language

If you only take one thing from this piece, take this set of distinctions:

- **LLM agent** — an LLM call wrapped in a control loop (ReAct[^30]-style). Acts through tokens, code, and APIs. Use this rather than the unqualified "agent."
- **Embodied agent** or **robot agent** — an agent that acts through physics, with sensors and actuators.
- **LLM-MAS** or **agentic LLM system** — multiple LLM agents (AutoGen, MetaGPT, ChatDev). Not the same as "multi-agent system" without qualification.
- **MARL system** or **embodied MAS** — multiple embodied agents in a stochastic game or Dec-POMDP[^31].
- **LLM skill** or **prompt skill** or **tool definition** — a prompt template or function spec invoked via context insertion. Not an option.
- **RL skill** or **option** — a learned policy π(a | s, z) with initiation and termination conditions, judged by environmental return.
- **VLA policy** — a vision-language-action model used as a robot policy. Embodied, not an "LLM agent."
- **LLM orchestrator** vs. **RL meta-policy** — a planner LLM directing modules vs. a learned controller selecting options.

When in doubt, ask the diagnostic question: **what medium does this agent act through, and what does that medium cost?** Tokens are cheap, reversible, and patient. Physics is none of those. The answer determines almost everything that follows.

---

## Why this matters

A naive reading of the current literature would conclude that "multi-agent systems" are a single rapidly-progressing field and that LLM-MAS frameworks like AutoGen are evidence of progress on multi-robot coordination. They are not. They are progress on a different problem in a different medium. The progress is real; the transfer is not.

Robotics groups that take the framing too literally will design architectures that assume cheap agent spawning (because LLMs are cheap to spawn) and discover too late that each "agent" is a $50,000 hardware platform with a 90-day lead time. Reviewers will misjudge claims because the noun has done their evaluation work for them. Students will spend years on what they think is the same problem and discover, mid-PhD, that the literature they have been reading is solving a different one.

The genuinely productive hybrid systems — VLAs, LLM-planner-plus-controller stacks, Voyager-style executable skill libraries, LLM-designed rewards, dialectic multi-robot coordinators — succeed precisely because they *respect the difference in medium*. They put LLMs at the layers where language, abstraction, and combinatorial search are valuable, and they put embodied policies at the layers where dynamics, real-time control, and sensorimotor coupling are non-negotiable. The mistake is not in building hybrids. The mistake is in pretending the layers are the same kind of thing.

So: keep using "agent." It's too useful a word to give up, and both communities are entitled to it. But when you do, expect the follow-up question — and ask it of others.

> [!IMPORTANT]
> **What medium does it act through?**

[^1]: Rashid, T. et al. ["QMIX: Monotonic Value Function Factorisation for Deep Multi-Agent Reinforcement Learning."](https://arxiv.org/abs/1803.11485) ICML 2018.
[^2]: Wu, Q. et al. ["AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation."](https://arxiv.org/abs/2308.08155) 2023.
[^3]: Wooldridge, M. ["An Introduction to MultiAgent Systems."](https://www.wiley.com/en-us/An+Introduction+to+MultiAgent+Systems%2C+2nd+Edition-p-9780470519462) 2nd ed. Wiley, 2009.
[^4]: Shinn, N. et al. ["Reflexion: Language Agents with Verbal Reinforcement Learning."](https://arxiv.org/abs/2303.11366) NeurIPS 2023.
[^5]: Park, J.S. et al. ["Generative Agents: Interactive Simulacra of Human Behavior."](https://arxiv.org/abs/2304.03442) UIST 2023.
[^6]: Sutton, R.S., Precup, D., Singh, S. ["Between MDPs and Semi-MDPs: A Framework for Temporal Abstraction in Reinforcement Learning."](https://www.sciencedirect.com/science/article/pii/S0004370299000521) Artificial Intelligence, 1999.
[^7]: Eysenbach, B. et al. ["Diversity is All You Need: Learning Skills without a Reward Function."](https://arxiv.org/abs/1802.06070) ICLR 2019.
[^8]: Sharma, A. et al. ["Dynamics-Aware Unsupervised Discovery of Skills."](https://arxiv.org/abs/1907.01657) ICLR 2020.
[^9]: Achiam, J. et al. ["Variational Option Discovery Algorithms."](https://arxiv.org/abs/1807.10299) 2018.
[^10]: Zhao, R. et al. ["Hierarchical Multi-Agent Skill Discovery."](https://papers.neurips.cc/paper_files/paper/2023/hash/c276c3303c0723c83a43b95a44a1fcbf-Abstract-Conference.html) NeurIPS 2023.
[^11]: Wang, G. et al. ["Voyager: An Open-Ended Embodied Agent with Large Language Models."](https://arxiv.org/abs/2305.16291) 2023.
[^12]: Liang, J. et al. ["Code as Policies: Language Model Programs for Embodied Control."](https://arxiv.org/abs/2209.07753) ICRA 2023.
[^13]: Brohan, A. et al. ["RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control."](https://arxiv.org/abs/2307.15818) CoRL 2023.
[^14]: Kim, M.J. et al. ["OpenVLA-OFT: A Unified Framework for Fine-Tuning Open-Source Vision-Language-Action Models."](https://arxiv.org/abs/2502.19645) 2025.
[^15]: Liu, S. et al. ["RDT-1B: A Diffusion Foundation Model for Bimanual Manipulation."](https://arxiv.org/abs/2410.07864) 2024.
[^16]: Black, K. et al. ["π0: A Vision-Language-Action Flow Model for General Robot Control."](https://www.physicalintelligence.company/blog/pi0) Physical Intelligence, 2024.
[^17]: Ahn, M. et al. ["Do As I Can, Not As I Say: Grounding Language in Robotic Affordances."](https://arxiv.org/abs/2204.01691) CoRL 2022.
[^18]: Huang, W. et al. ["VoxPoser: Composable 3D Value Maps for Robotic Manipulation with Language Models."](https://arxiv.org/abs/2307.05973) CoRL 2023.
[^19]: Driess, D. et al. ["PaLM-E: An Embodied Multimodal Language Model."](https://arxiv.org/abs/2303.03378) ICML 2023.
[^20]: Mandi, Z. et al. ["RoCo: Dialectic Multi-Robot Collaboration with Large Language Models."](https://arxiv.org/abs/2307.04738) ICRA 2024.
[^21]: Zhang, H. et al. ["Building Cooperative Embodied Agents Modularly with Large Language Models."](https://arxiv.org/abs/2307.02485) ICLR 2024.
[^22]: Kannan, S. et al. ["SMART-LLM: Smart Multi-Agent Robot Task Planning using Large Language Models."](https://arxiv.org/abs/2309.10062) 2023.
[^23]: Ma, Y.J. et al. ["Eureka: Human-Level Reward Design via Coding Large Language Models."](https://arxiv.org/abs/2310.12931) ICLR 2024.
[^24]: Xie, T. et al. ["Text2Reward: Reward Shaping with Language Models for Reinforcement Learning."](https://arxiv.org/abs/2309.11489) ICLR 2024.
[^25]: Lanctot, M. et al. ["A Unified Game-Theoretic Approach to Multiagent Reinforcement Learning."](https://arxiv.org/abs/1711.00832) NeurIPS 2017.
[^26]: Vinyals, O. et al. ["Grandmaster Level in StarCraft II using Multi-Agent Reinforcement Learning."](https://www.nature.com/articles/s41586-019-1724-z) Nature, 2019.
[^27]: Zhu, Z. et al. ["MADIFF: Offline Multi-agent Learning with Diffusion Models."](https://arxiv.org/abs/2305.17330) 2023.
[^28]: Hong, S. et al. ["MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework."](https://arxiv.org/abs/2308.00352) ICLR 2024.
[^29]: Qian, C. et al. ["ChatDev: Communicative Agents for Software Development."](https://arxiv.org/abs/2307.07924) ACL 2024.
[^30]: Yao, S. et al. ["ReAct: Synergizing Reasoning and Acting in Language Models."](https://arxiv.org/abs/2210.03629) ICLR 2023.
[^31]: Oliehoek, F.A. & Amato, C. ["A Concise Introduction to Decentralized POMDPs."](https://link.springer.com/book/10.1007/978-3-319-28929-8) Springer, 2016.
