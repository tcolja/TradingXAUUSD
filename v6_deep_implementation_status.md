# V6-informed deep search: implementation and evidence

## Scope and data

The new run uses `search_rules_v6_deep.md`: 20 discovery generations plus
200 optimisation generations of 300 offspring slots, with Dataset A ending
2025-01-06, Dataset B ending 2025-07-06, and Dataset C afterward. The source
cache records 164,204,647 cleaned quotes from 2023-01-06 00:00:00.641 UTC to
2026-01-05 23:59:58.686 UTC. The file contains six unexpected gaps totaling
about 95.84 minutes. Dataset C has been exposed in earlier research.

The engine uses example broker cost and contract settings. Its account balance,
position sizing and P&L are in USD; no matching historical EUR/USD series was
provided, so this is not a 10,000 EUR-account simulation. The run uses market
entries and at most one position per strategy. This avoids relying on the
remaining pending-order and concurrent-equity approximations in the broader
simulator. Neither slippage nor the assumed broker specifications are verified.

## Changes and measured checks

- Entry logic can vary between all/any filters and a causal trigger in the
  current or up to three prior M1 bars; structural mutation and crossover
  change these genes. This is a bounded extension of the existing trigger and
  filter model, not the full strongly typed expression-tree grammar in the
  request.
- Novelty uses observed trade frequency, direction, holding time, exposure and
  four session shares, and only breaks close positive-fitness ties. Staged
  finalist clustering also compares entry minutes and daily P&L streams.
  The novelty archive does not yet contain every requested regime, EMA-distance
  or volatility descriptor.
- Local refinement counts plateau probes within its explicit budget and keeps
  stop and target trials within configured bounds.
- The fast screen counts actual entry signals across its entire Dataset-A
  evaluation segment. Rejections are sampled by stable hash and reported by
  family with Wilson intervals. This is not a predeclared regime-stratified
  multi-fidelity screen and does not measure rejection by all requested regimes.
- Dataset-A-only spread and range statistics now set validation stress levels.
  Complete genomes and parameter variants, plus fixed baseline hypotheses, are
  counted in the multiple-testing correction. The separate EMA study is off.
- Atomic generation checkpoints include the GA population, cached evaluations,
  random state, adaptive family/operator state and data/settings/binary
  fingerprint. A regression test reproduced a four-generation continuation
  with one and four workers. Resume is deliberately refused after the GA stage,
  to prevent repeating final holdout evaluation. This is not a proof of exact
  resumption under every possible interruption or platform.
- Finalists and selection settings are frozen before Dataset C; a durable ledger
  marks each evaluation before it starts, and the selected portfolio is not
  re-optimised after C. C remains historically exposed, not unseen.
- The Release test suite passed 70 tests. Among them are gap behavior,
  aggregate risk, short/long execution, a single-position market-order reference
  comparison, and truncated-history checks across catalog conditions on M1
  and M15. These tests do not prove every execution or indicator scenario.

The integration smoke in `results/v6_verified_smoke_20261113` used Dataset A
only, 33 population slots and two optimisation generations. It recorded 66
optimisation offspring slots, 178 full evaluator backtests, 31 screen
rejections including five audited rejections, four novelty-archive entries at
the final snapshot, 10 behavioral clusters and no candidates passing validation.
These small-sample results are not evidence for or against a trading edge.

The existing benchmark reports describe earlier V4/V5 runs with different
settings. They are not an apples-to-apples performance comparison for this
upgrade. The requested general tree grammar, complete validation suite,
regime-stratified screening audit and a like-for-like performance benchmark
remain incomplete; the running experiment must not be described as a fully
implemented V6 engine.
