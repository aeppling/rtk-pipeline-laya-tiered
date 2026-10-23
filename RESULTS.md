# laya-tiered — benchmark results

Two benchmarks, both on SkillsBench through Harbor with Claude Opus 5.5 (subscription billing, 1h prompt cache), 4 trials per arm per task, stock Claude Code (OFF) always in the same batch as the pipeline:

- **Bench B, 2026-10-22/23: 11 tasks, OFF vs laya-tiered.** The broad run: the 6 tasks of bench A plus 5 more software tasks, 44 trials per arm.
- **Bench A, 2026-10-21/22: 6 tasks, OFF vs laya-delegate vs laya-tiered.** The three-arm run that compares this pipeline with [laya-delegate](https://github.com/aeppling/rtk-pipeline-laya-delegate) in the same batch.

Per-trial rows for both are in `results.tsv` (column `bench`).

## Best gains

- **-51% cost on parallel-tfidf-search** ($0.885 → $0.432 per trial in bench B; -62% in bench A).
- **-31% on fix-visual-stability with the pass rate up** (0.00 → 0.50), **-28% on python-scala-translation** (-46% with the timed-out-but-passing trials counted), **-24% on fix-build-google-auto with the pass rate up** (0.75 → 1.00).
- **-20.2% on the 11-task suite** ($4.312 → $3.440 summed over the tasks; -24.0% with timed-out-but-passing trials counted), reward 0.66 → 0.64. On the 9 tasks any model can score (two score 0 for every model): -23.5% / -27.5%, reward 0.81 → 0.78.
- **-26.7% on the 6-task suite with reward 0.72 → 0.85** (-32.3% and 0.68 → 0.85 with timed-out-but-passing trials counted). laya-delegate in the same batch: -13.0%.
- **27 implementation hand-offs to a Sonnet sub-agent and 26 investigation hand-offs to a Haiku sub-agent** across the 42 usable trials of bench B (bench A: 18 and 20 across 27), decided by Laya and enforced by RTK, with no task-specific rules.
- The price: **1.8× the wall-clock** of stock Claude Code on bench B (sub-agent chains run one after the other), and a small loss of implementation sharpness on one task (see below).

## Bench B — 11 tasks, OFF vs laya-tiered (2026-10-22/23)

Mean cost per trial and mean agent minutes per trial:

| task | OFF | laya-tiered | change | reward OFF → tiered | wall min OFF → tiered |
|---|---|---|---|---|---|
| parallel-tfidf-search | $0.885 | $0.432 | -51% | 1.00 → 0.75 | 13.1 → 23.5 |
| python-scala-translation | $0.891 | $0.641 (n=2) | -28% | 1.00 → 1.00 | 8.2 → 10.9 |
| fix-visual-stability | $0.620 | $0.427 | -31% | 0.00 → 0.50 | 6.8 → 11.7 |
| fix-build-google-auto | $0.298 | $0.227 (n=3) | -24% | 0.75 → 1.00 | 10.4 → 12.0 |
| react-performance-debugging | $0.476 | $0.431 | -9% | 1.00 → 0.25 | 4.6 → 12.2 |
| debug-trl-grpo | $0.274 | $0.253 | -8% | 0.60 → 0.60 | 1.9 → 7.0 |
| fix-erlang-ssh-cve | $0.233 | $0.250 | +7% | 1.00 → 1.00 | 8.5 → 11.1 |
| dialogue-parser | $0.234 | $0.260 | +11% | 0.92 → 0.92 | 1.5 → 5.0 |
| jax-computing-basics | $0.125 | $0.167 | +34% | 1.00 → 1.00 | 1.6 → 4.6 |
| azure-bgp-oscillation-route-leak | $0.134 | $0.120 | -11% | 0.00 → 0.00 | 1.4 → 5.2 |
| llm-prefix-cache-replay | $0.141 | $0.232 | +65% | 0.00 → 0.00 | 1.4 → 4.6 |
| **total (11 tasks)** | **$4.312** | **$3.440** | **-20.2%** | **0.66 → 0.64** | |

Trials that errored or ran past the task time limit are excluded.

| task | OFF | laya-tiered | change | reward OFF → tiered | wall min OFF → tiered |
|---|---|---|---|---|---|
| parallel-tfidf-search | $0.885 | $0.432 | -51% | 1.00 → 0.75 | 13.1 → 23.5 |
| python-scala-translation | $0.891 | $0.478 (n=3) | -46% | 1.00 → 1.00 | 8.2 → 12.4 |
| fix-visual-stability | $0.620 | $0.427 | -31% | 0.00 → 0.50 | 6.8 → 11.7 |
| fix-build-google-auto | $0.298 | $0.227 (n=3) | -24% | 0.75 → 1.00 | 10.4 → 12.0 |
| react-performance-debugging | $0.476 | $0.431 | -9% | 1.00 → 0.25 | 4.6 → 12.2 |
| debug-trl-grpo | $0.274 | $0.253 | -8% | 0.60 → 0.60 | 1.9 → 7.0 |
| fix-erlang-ssh-cve | $0.233 | $0.250 | +7% | 1.00 → 1.00 | 8.5 → 11.1 |
| dialogue-parser | $0.234 | $0.260 | +11% | 0.92 → 0.92 | 1.5 → 5.0 |
| jax-computing-basics | $0.125 | $0.167 | +34% | 1.00 → 1.00 | 1.6 → 4.6 |
| azure-bgp-oscillation-route-leak | $0.134 | $0.120 | -11% | 0.00 → 0.00 | 1.4 → 5.2 |
| llm-prefix-cache-replay | $0.141 | $0.232 | +65% | 0.00 → 0.00 | 1.4 → 4.6 |
| **total (11 tasks)** | **$4.312** | **$3.277** | **-24.0%** | **0.66 → 0.64** | |

Same, but trials that ran past the task time limit and still scored are counted (they were graded; the cost is what they spent).

## Bench A — 6 tasks, OFF vs laya-delegate vs laya-tiered (2026-10-21/22)

| task | OFF | laya-delegate | laya-tiered | tiered vs OFF | reward OFF → delegate → tiered |
|---|---|---|---|---|---|
| parallel-tfidf-search | $1.336 (n=2) | $0.685 (n=4) | $0.501 (n=1) | -62% | 1.00 → 1.00 → 1.00 |
| python-scala-translation | $0.888 (n=3) | $0.912 (n=4) | $0.678 (n=5) | -24% | 1.00 → 1.00 → 1.00 |
| fix-visual-stability | $0.545 (n=4) | $0.536 (n=4) | $0.446 (n=4) | -18% | 0.25 → 0.25 → 0.50 |
| fix-build-google-auto | $0.315 (n=4) | $0.363 (n=4) | $0.257 (n=4) | -18% | 0.50 → 1.00 → 1.00 |
| fix-erlang-ssh-cve | $0.173 (n=4) | $0.204 (n=4) | $0.235 (n=4) | +36% | 1.00 → 1.00 → 1.00 |
| debug-trl-grpo | $0.166 (n=4) | $0.277 (n=4) | $0.392 (n=4) | +135% | 0.60 → 0.60 → 0.60 |
| **total (6 tasks)** | **$3.423** | **$2.979** (-13.0%) | **$2.509** (-26.7%) | **-26.7%** | **0.72 → 0.81 → 0.85** |

Trials that errored or ran past the task time limit are excluded.

| task | OFF | laya-delegate | laya-tiered | tiered vs OFF | reward OFF → delegate → tiered |
|---|---|---|---|---|---|
| parallel-tfidf-search | $1.606 (n=4) | $0.685 (n=4) | $0.514 (n=4) | -68% | 0.75 → 1.00 → 1.00 |
| python-scala-translation | $0.888 (n=3) | $0.912 (n=4) | $0.657 (n=7) | -26% | 1.00 → 1.00 → 1.00 |
| fix-visual-stability | $0.545 (n=4) | $0.536 (n=4) | $0.446 (n=4) | -18% | 0.25 → 0.25 → 0.50 |
| fix-build-google-auto | $0.315 (n=4) | $0.363 (n=4) | $0.257 (n=4) | -18% | 0.50 → 1.00 → 1.00 |
| fix-erlang-ssh-cve | $0.173 (n=4) | $0.204 (n=4) | $0.235 (n=4) | +36% | 1.00 → 1.00 → 1.00 |
| debug-trl-grpo | $0.166 (n=4) | $0.277 (n=4) | $0.392 (n=4) | +135% | 0.60 → 0.60 → 0.60 |
| **total (6 tasks)** | **$3.694** | **$2.979** (-19.4%) | **$2.502** (-32.3%) | **-32.3%** | **0.68 → 0.81 → 0.85** |

Same, but trials that ran past the task time limit and still scored are counted (they were graded; the cost is what they spent).

Mean minutes per trial (agent phase, timed-out trials counted):

| task | OFF | laya-delegate | laya-tiered |
|---|---|---|---|
| parallel-tfidf-search | 27.8 | 14.3 | 28.7 |
| python-scala-translation | 8.3 | 10.9 | 12.7 |
| fix-visual-stability | 5.4 | 10.8 | 14.9 |
| fix-build-google-auto | 10.7 | 17.1 | 13.9 |
| fix-erlang-ssh-cve | 7.4 | 8.9 | 10.5 |
| debug-trl-grpo | 2.5 | 4.7 | 4.3 |

## What the pipeline triggered

| action | bench B (42 trials) | bench A (27 trials) |
|---|---|---|
| RTK refusals pointing to the sub-agents | 32 | 22 |
| Haiku Explore sub-agent hand-offs | 26 | 20 |
| Sonnet general-purpose sub-agent hand-offs (the implementation) | 27 (in 27 trials) | 18 (in 18 trials) |
| Tokens removed by RTK output filtering | 60K | 85K |

Sub-agent counts come from the main transcript; 3 trials of bench B and 3 of bench A have no main transcript in the collected artifacts (a harness artifact-download miss; their cost is the one Claude Code itself reported, which matches harbor's figure exactly where both exist).

The hand-offs happen on the write and data tasks: every trial there delegated. On the bug tasks (debug-trl-grpo, fix-erlang-ssh-cve) Laya labels the work "bug" and the pipeline stays out of the way; they land within a few cents of OFF.

## Reading the numbers

- **4 trials per arm:** a per-task difference under about 20% is within run-to-run noise. The suite totals are the numbers to quote. Stock Claude Code itself cost $1.336 on parallel-tfidf-search in bench A and $0.885 the next day, so only same-batch comparisons are used here.
- **Where the saving comes from:** the main Opus agent stops writing the code. Its own output (code plus thinking, billed at $20/M) was half the bill on write tasks; the Sonnet sub-agent writes it at $10/M after a Haiku ($0.50/M) investigation, and Opus keeps the brief, the review and the final test run.
- **The saving is concentrated on tasks that cost stock Opus about $0.40 or more.** On short tasks that stock Opus finishes in about 90 seconds for under $0.25 (dialogue-parser, jax-computing-basics, llm-prefix-cache-replay) the investigate-implement-review chain adds a fixed 3 to 9 cents that nothing offsets, with quality unchanged. A size gate before delegating is the obvious next improvement.
- **Quality:** up on fix-visual-stability and fix-build-google-auto, equal on the bug tasks and the short tasks, down on the two tasks whose verifier asserts wall-clock performance. parallel-tfidf-search: one bench-B trial measured a 1.49× indexing speedup against a 1.5× threshold (stock Opus failed the same test in bench A at 1.37×). react-performance-debugging: 3 of 4 tiered trials failed the checkout latency test (921 ms against a limit of 800 ms) because the Sonnet implementer left one fetch waiting on another it did not depend on, which stock Opus reordered in 4 of 4 trials. That is a real loss of implementation sharpness from delegating, and the Opus review step did not catch it. Bench B reward: 0.66 → 0.64 over 11 tasks.
- **Wall-clock is the other cost.** Small tasks go from 1.5 to 5 minutes; on parallel-tfidf-search 3 of 4 bench-A tiered trials ran past the 30-minute agent limit and still passed; on python-scala-translation 2 of 4 bench-B tiered trials ran past the limit and still passed (included in the inclusive tables).
