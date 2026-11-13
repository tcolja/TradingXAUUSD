# Screening reliability report

## V6-informed update, 2026-11-13

All rejected evaluations, including audited ones, are now counted; family
audit counts and Wilson confidence intervals are reported. In the Dataset-A
smoke test, 5 of 31 screen rejections were audited and none met the minimum
trade threshold. The 95% Wilson upper bound was 43.45%, reflecting the very
small sample. Sampling remains stable-hash-based, not explicitly stratified
by regime, duration or frequency.

## Measured V5 baseline

V5's L1 screen counts possible entry signals on the evaluator's segments and
skips the tick backtest when the count is below a fraction of `min_trades`.
The completed benchmark recorded:

- 6,224 L1 rejections.
- 313 full-backtest audits (5.03% of rejections).
- 91 audited rejections with positive fitness.
- 1 audited rejection with fitness greater than 0.05.
- 0 audited rejections meeting the configured minimum-trade threshold.

The result supports the narrow claim that no audited rejection met the
run's minimum-trade gate. It does not show that the screen is unbiased across
families, years, volatility regimes, or holding durations; the current audit
sample is stable-hash-based, not stratified.

## V6 gaps

The current L1 screen is not the proposed representative multi-regime
multi-fidelity screen. It does not evaluate predeclared year/session/volatility
blocks, maintain per-family false-rejection rates, or report confidence
intervals. `fitness > 0` is not an adequate promising-candidate criterion:
91 of 313 audited rejections had positive fitness, while none met the
minimum-trade threshold.
