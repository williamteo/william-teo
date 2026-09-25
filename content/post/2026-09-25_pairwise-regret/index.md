---
title: When the Score Picks the Wrong Robot Plan
date: 2026-09-25T09:00:00+08:00
subtitle: "Pairwise scores, frozen replays, and plan-selection regret"
tags: ["robotics", "multi-robot", "exploration", "IROS"]
draft: true
---

Many multi-robot coordination methods score a joint plan by adding up what each robot does alone and then correcting for each pair of robots. Anything that involves three or more robots at once is left out. A pairwise score is compact and cheap to evaluate, which makes it attractive when a planner has many joint plans to compare. I wanted to know whether it picks the same plan as the full objective.

This post summarizes my paper for the IROS 2026 Workshop on Intelligent Information Gathering, [*Pairwise Approximation Can Select the Wrong Multi-Robot Plan*](https://arxiv.org/abs/2609.29929). The [project page](https://www.william-teo.com/pairwise-regret/) has a replay of the two plans discussed below, an interactive view of the subset values behind each score, and the code to recompute every score and selection.

## The setup

I used the [indoor exploration benchmark](https://github.com/BYU-FROST-Lab/indoor-exploration-competition) from the BYU FROST Lab, with four-robot teams on its seven maps. The objective is delivered coverage, the fraction of the map that the base station knows at the end of the episode. Robots share maps when they come within communication range of each other or of the base, so what reaches the base depends on who meets whom and when.

For each map I generated eight candidate joint plans in each of two candidate families, one built from nearest-frontier exploration and one from random-frontier exploration. I then replayed every plan with each of the 16 subsets of its four robots, with each remaining robot following its original trajectory. Those 16 values give the exact delivered-coverage set function F for that plan. Its Möbius decomposition splits F into singleton, pairwise, triple and four-robot terms, and keeping only the first two gives the pairwise score F₂.

## Same eight plans, different choice

On env1, in the nearest-frontier family at the 15 m range used to generate the plans, the plan ranked first by F delivers 79.4% of the map. The plan ranked first by F₂ delivers 45.6%, a loss of 33.7 percentage points. Both rankings use the same eight plans. Only the score changes.

At 15 m, ranking by F₂ instead of F changes the selected plan on six of seven maps in each family. The map where F₂ still picks the best plan is env3 for the nearest-frontier plans and env1 for the random-frontier plans.

## Refitting the pairwise terms

F₂ keeps the exact singleton and pairwise terms of F. A least-squares fit, G, instead chooses singleton and pair coefficients that match all 15 nonempty subset values of each plan as closely as possible. G reduces the regret, but it still changes the selection on three of seven maps in each family.

G is fitted to all 15 values, including the full-team value, so it is an in-sample comparison. It does not tell us what a pairwise model could predict from singleton and pair measurements alone.

## An additive baseline

I also compared the additive score F₁, which keeps only the singleton terms. At 15 m, F₁ selects the best plan on six of seven nearest-frontier maps and four of seven random-frontier maps, against one of seven for F₂ in each family. I added this comparison after the pairwise analysis, using the same recorded data.

## Reconstruction error and selection

The usual way to judge an approximation is by its reconstruction error. At 15 m in the nearest-frontier family, env3 has a mean normalized error E₂ of about 0.26, and F₂ still picks the best plan. Env5 has a lower mean E₂ of about 0.14, but the plan F₂ picks there delivers 24.0 percentage points less than the best one.

F₂ picks a worse plan when its full-team overestimate of that plan exceeds its overestimate of the best plan by more than the gap between their exact scores. An error that shifts every plan's score by the same amount leaves the ranking unchanged.

## Scope

These results cover four-robot teams on seven maps of one benchmark, with fixed start positions and eight candidate plans per family and map. The plans are frozen trajectories, and I did not test online replanning. The communication ranges on a map reuse the same trajectories. The results do not show that pairwise planners fail in general.

What I take from this is that when an approximate objective is used to choose between plans, it should be checked on the plans it chooses, as well as on how closely it reconstructs the objective.

*Paper: [arXiv:2609.29929](https://arxiv.org/abs/2609.29929). Project page, data and code: [william-teo.com/pairwise-regret](https://www.william-teo.com/pairwise-regret/).*
