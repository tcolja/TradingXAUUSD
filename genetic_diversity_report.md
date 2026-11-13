# Genetic diversity report

## V6-informed update, 2026-11-13

An optional bounded novelty archive now compares trade frequency, direction,
holding duration, exposure and session distribution for close positive-fitness
tournament ties. The staged validation funnel also clusters by entry times and
daily P&L. This does not implement every requested behavioral descriptor or a
full island model. The Dataset-A-only smoke run ended with four novelty archive
entries and clustered 20 pool candidates into 10 groups. Its results are not an
apples-to-apples comparison with the earlier benchmark below.

## Measured V5 baseline

The completed V5 benchmark used family niches, adaptive family budgets,
per-family archives, a Pareto archive, adaptive operator selection, immigrants,
and stagnation responses. The run logged 200 optimization generations with 300
offspring per generation.

| Metric | V4 | V5 |
|---|---:|---:|
| Distinct evaluated structures | 16,847 | 20,119 |
| Positive distinct structures | 7,377 | 8,340 |
| Positive-genome family entropy | 0.382 | 0.920 |
| Positive-genome trigger entropy | 0.287 | 0.673 |
| Pareto front | 323 | 506 |
| Mean V5 duplicate rate | n/a | 0.005 |
| Mean V5 population diversity | n/a | 0.294 |

V5 logged 14 stagnation diagnoses: 10 low-diversity, 2 family-dominance, and 2
fitness-plateau responses. Family K was added after this benchmark and is not
represented in these figures.

## Limitation

These are structural and categorical diversity measures. The engine does not
compare aligned trade timestamps, exposure, or return streams, so it cannot
establish that two candidates with different signatures have distinct
behaviors. It also does not implement the V6 four-island population model or
behavioral novelty archive.

## Regression fix

Family-budget allocation now redistributes slots lost to a family cap so the
sum of family budgets remains the configured population when floor/cap
constraints are feasible. The targeted regression and full Core.Tests suite
pass.
