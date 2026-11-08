# Propose-and-check on a real game: first results (08.11.2026, early)

Game: Second Tail, a two-rat co-op platformer (one player, swap with Q). Question from the author: "Sal's controls feel
right, Ben has no advantage. During the run both should be comfortable but each should have an edge; at bosses, strength
should come only from mixing their skills." Step 1 (Prolog, clpfd) is done; step 2 (TLA+ on the top layouts) is running.

## Inputs

- Movement numbers taken from the game code: Sal double jump (second jump 152 px), run 364 px/s; Ben dash
  176 px / 0.85 s cooldown, single jump 164.5 px vs Sal 128.1 px, run 316 px/s.
- 18 real platform transitions on the route, scored by an exact Python port of the game's own reach check
  (MoveReachCheck.cs, 0 mismatches against the C# output): the score is the width of the take-off window that still lands.
- Gates the code makes rat-specific: 5 dash-only (ram) gates, 2 double-jump-only buttons; 3 jumps that need both rats.
- Enemies in three kinds (ground, flyer, armoured) and crates. Their COUNTS are an assumption, so three route profiles
  were run (a: 39 obstacles, b: 50, c: 61). Scores for non-jump obstacles (4-8) are designer judgment, not measured.
- 19 boss phases across the game with their shortest vulnerable window from
  the code; 1 of them already needs both rats.

## Design space

8 candidate new abilities (Ben: slam, brace, wall claws, gnaw; Sal: tail glide, tail whip, squeeze; pair: Q relay),
at most 2 per rat + 1 shared, at most 2 new inputs: 112 ability sets x 6 retunings of Ben's numbers = 672 combinations.

## Rules (the author's words) and what they did

- (a) Comfort: the pair passes everything with one Q; each rat alone passes >= 80 % of ordinary obstacles.
- (b) Edge: each rat is clearly better (score gap >= 3) on 35-65 % of obstacles.
- (c) Bosses: no phase winnable by one rat; every phase winnable by a mix (one opens, the other exploits; window >= 0.8 s,
  1.0 s with a Q relay).
- (d) Learnable: <= 2 new inputs; each new ability is taught by one obstacle.
- Cost: obstacles redesigned (budget 20), then boss novelty, then implementation cost.

## Numbers

- Today (profile b, 50 obstacles): Sal clearly better on 5, Ben on 7, 35 neutral, and Ben's 7 are all "can / cannot"
  gates and armoured enemies, not comfort. The model confirms the author's feeling.
- Retuning Ben's numbers alone (dash 176/240/300 px, run 316/340) changes almost nothing: Sal's double jump is never worse
  than Ben's dash on any measured transition.
- Feasible layouts within the budget of 20 redesigned obstacles: 672 / 408 / 384 (profiles a / b / c); 56 at budget 14.
- Best (profile b): Ben slam + Ben gnaw + pair Q relay, 13 obstacles redesigned, 3 boss patterns, cost 8; Sal clearly
  better on 18/50, Ben on 23/50. Same 13 redesigns with slam + gnaw only (cost 5) or brace + gnaw (cost 4).
- With no new ability at all the rules need 24 redesigned obstacles: new abilities for Ben roughly halve the level work.
- Profile a Pareto front: 11 layouts nobody beats on all three costs.

## Next

TLC on the top 2 layouts over 2 boss phases (the tank's hatch window and one more): is there a
single-rat win, is a mixed win reachable. Every counterexample will be replayed on the game code. Numbers will be added
here.

## Step 2: TLA+ (TLC) on the top layouts (added 08.11, 02:00)

Spec: one boss phase with both rats, Q swaps, the phase's windows and the proposed abilities as constants. Three phases
modelled from the code: the tank's valve window, a helicopter phase (Q is blocked during its grabs), a duel.

- TODAY's abilities: TLC finds a one-rat win with zero Q on all three phases (the tank in 9 steps). So the bosses do not
  need the mix today, which is exactly the author's complaint about Ben.
- Top 5 Prolog layouts (slam + gnaw + relay; slam + gnaw; brace + gnaw; gnaw + whip + relay; slam only):
  "no single-rat win" holds on all three phases, and a mixed win is reachable on all three
  (tank 18-21 steps with 4-5 Q; helicopter 40-47 steps with 6-12 Q; duel 24-29 steps with 4-7 Q;
  1.1k-7.4k distinct states per run).
- Controls: when the same rat owns both the opener and the exploiter, TLC finds a one-rat win; the relay without the
  pair ability gives no win at all. Both behave as expected, so the property can fail.
- Parameter sweeps the author has to decide on: the mix needs a vulnerable window >= 1.0 s (duel) and >= 1.5 s (tank);
  the helicopter phase needs >= 3 hearts, because Q is blocked inside the grab windows.

Still running: a per-rule funnel (how many layouts pass a, b, c, d separately and together) and a second, fully
reified clpfd implementation that must agree with the first on all 672 results.

## Final (08.11, 02:05): two independent models, and they corrected the early numbers

The early run above is superseded. A second, fully reified clpfd implementation (every ability a 0/1 bit, "ability
covers category" via #<==>) had to agree with the first on every layout of all three profiles. It did not at first:
the cross-check found a real bug in the first model, `forall(..., T = 0)` never bound the clpfd variable, so an ability
with no teaching obstacle was counted as taught for free (exactly the pitfall the skill warns about). After the fix the
two agree on all results (self-test 12/12).

- Funnel (profile b, budget 20): 1536 raw layouts -> 672 after learnability (d) -> 392 after the redesign budget.
  Rules (a) comfort and (c) bosses-can-be-a-mix removed nothing: they hold for every layout. No layout reaches the
  advantage band without redesigning obstacles (0 of 1536).
- Today: of 21 measured jumps Sal is clearly better on 15, Ben on 0. Retuning Ben's numbers does not change that.
- Best: Ben gnaw (hold F) + Sal tail lash (Shift): 14 of 50 obstacles redesigned (vs 24 with no new ability), edges
  18 and 18, 3 boss patterns. Cheapest meaningful: Ben gnaw only, 16 redesigns, one new key.
- TLC on 3 boss phases: today a one-rat win with zero Q exists on all three (9 / 18 / 12 steps); under every proposal
  there is none, and a mixed win exists (18-45 steps, 3-11 Q). Surprise from TLC: the mix itself needs no new ability
  ("Ben's dash opens, Sal's double jump finishes" already works once the windows are >= 1.0 s); new abilities only add
  more mix patterns. Sweeps: the duel needs a window >= 1.0 s, the tank >= 1.5 s; the helicopter needs >= 3 hearts.
