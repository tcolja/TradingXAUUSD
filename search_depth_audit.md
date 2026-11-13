# Search depth audit

## V6-informed update, 2026-11-13

The subsequent engine changes and their current limitations are summarized in
`v6_deep_implementation_status.md`. The Dataset-A-only integration smoke at
`results/v6_verified_smoke_20261113` recorded 66 optimisation offspring slots,
178 full evaluator backtests, 17 cache hits, 31 screen rejections (five
audited), 10 behavioral clusters and no final candidate. These are smoke-test
measurements, not results of the 200-by-300 run and not a comparable benchmark.
The full run writes its own `generations.jsonl` and `search_audit.json`.

## Scope and provenance

This audit uses the completed `results/bench_v5/v5` run and its `benchmark.md`,
`search.log`, `search_audit.json`, `generations.jsonl`, `family_coverage.json`,
and `progress.json`. It is a pre-Heikin-Ashi V5 baseline, not a V6 run. The run
used Dataset A (2023-01-06 through 2025-01-06) for discovery and evolution,
Dataset B (2025-01-06 through 2025-07-06) for development validation, and
Dataset C (2025-07-06 through 2026-01-06) only after finalist selection.
Dataset C is labelled `previously_exposed`.

## Measured search depth

| Measurement | V5 result |
|---|---:|
| Wall time | 581 s |
| Optimization generations | 200 / 200 |
| Discovery generations | 12 |
| Logged offspring slots | 63,600 (212 generations × 300) |
| Main-GA evaluations completed / planned | 74,082 / 60,300 |
| Main-GA evaluations planned including immigrants | 66,300 |
| Candidate evaluation calls, all phases | 116,710 |
| Unique tick backtests, all phases | 108,875 |
| Unique structures in optimization archive | 20,119 |
| Positive-fitness distinct structures | 8,340 |
| M1 bars in loaded data | 1,061,859 |
| V5 optimization bars processed | 43,695,626,808 |
| V5 refinement bars processed | 3,447,569,532 |
| V5 validation bars processed | 7,167,417,138 |
| Cache hit rate | 2.03% |
| Median spread | 0.386 |
| L1 rejects / audited | 6,224 / 313 |
| Refinement candidates / refinements with improvement | 80 / 72 |
| Finalists evaluated / passed all validation | 8 / 0 |

The 63,600 offspring slots comprise 12 discovery generations and 200
optimization generations. The reported unique-backtest and bar counters include
multiple phases; bar counts are summed work across evaluations, not a count of
distinct bars in the source file. Evaluation and backtest counters have
different definitions: for example, the run's `eval.full_backtests` counter is
108,115, while the pipeline-wide `backtest.total` counter is 108,875.

The search did not stop early in optimization: the generation log contains 200
optimization rows at 300 offspring each. The 12-generation discovery phase was
configured separately. Therefore this run's duration is not evidence that the
requested 200-generation optimization was silently shortened.

## Benchmark comparison

| Metric | V4 | V5 |
|---|---:|---:|
| Wall time | 226 s | 581 s |
| GA candidate evaluations | 109,502 | 116,710 |
| Unique tick backtests | 90,957 | 108,875 |
| Distinct structures | 16,847 | 20,119 |
| Positive-fitness distinct structures | 7,377 | 8,340 |
| Family entropy | 0.382 | 0.920 |
| Trigger entropy | 0.287 | 0.673 |
| Pareto front size | 323 | 506 |
| Best Dataset-A fitness | 1.0760 | 1.0897 |
| Finalists with positive Dataset-B net | 0 / 8 | 0 / 8 |
| Passed all validation | 0 / 8 | 0 / 8 |

V5 explored more structures and had higher family/trigger entropy, but took
about 2.57× as long and did not produce a validated candidate. This is measured
coverage improvement, not evidence of a profitable edge.

## Screening audit

The V5 run audited 313 of 6,224 L1 rejections (5.03%). Of those audited:

- 91 had positive fitness on full evaluation.
- 1 had fitness above 0.05.
- 0 met the run's minimum-trade threshold.

Thus, this audit found no false rejection that met the configured
minimum-trade criterion. The 91 positive-fitness cases show that `fitness > 0`
alone is too permissive to define a promising candidate. This was a stable-hash
sample, not a regime/family-stratified sample, so it does not establish a
family-specific false-rejection confidence bound.

## Known gaps in this baseline

- Genomes are a trigger plus a conjunction of filters and separate exit genes,
  not the typed Boolean/event/temporal expression grammar in the V6 spec.
- The V5 benchmark predates the Heikin-Ashi family K implementation; it covered
  families A-J. A later short smoke run exercised K, but did not constitute a
  full V6 search.
- Behavioral diversity is not measured by aligned trade streams. Current
  generation diversity is primarily structural/signature-based.
- Checkpoints persist populations and archives, but not random-generator state
  or all adaptive/archive state; resumed execution is not proven bit-for-bit
  reproducible.
- The existing fast screen is an entry-signal count screen, not the
  representative multi-regime multi-fidelity funnel described by V6.
- The full V6 implementation and its required full search have not been
  completed. Do not treat this report as a V6 acceptance report.

## Code change from this audit

V5 family-budget allocation now redistributes all available slots after applying
family floors and caps. Previously, a capped family's removed share could leave
unallocated offspring slots. A regression test verifies the total population
is fully assigned and no family exceeds its cap. Core tests pass: 54 passed,
0 failed.
