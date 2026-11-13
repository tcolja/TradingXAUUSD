# Local refinement report

## V6-informed update, 2026-11-13

The coordinate refiner now counts plateau probes against its explicit total
budget, reserves part of that budget for those probes, and bounds stop/target
trials by the configured ranges. The full run config selects at most 15
distinct eligible candidates per periodic round, with 20 calls per candidate.
It still does not report all requested cross-fold stable-region statistics.

## Measured V5 baseline

The completed V5 run performed 80 refinement starts over four optimization
rounds, with a configured budget of 60 evaluations per start. The artifact
records 72 refinements with at least one accepted move. The run also reports
4,873 refinement backtests across the pipeline, including repeated/cache-backed
work as counted by the backtester.

Refinement operates on the same Dataset-A evaluator used for evolution. The
implementation searches one coordinate at a time, evaluates nearby values, and
reports a plateau fraction based on neighboring configurations.

## Limitations

The current refinement is randomized coordinate neighborhood search, not a
documented top-5%-of-eligible policy. Its plateau measure is not a complete
stable-region assessment across folds, execution costs, trade count, and
drawdown. The V6 refinement requirements remain partially unmet.

No Dataset B or C evaluations are used by the refiner.
