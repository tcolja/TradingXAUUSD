# Performance benchmark

## V6-informed update, 2026-11-13

The upgraded Dataset-A-only smoke run completed in approximately 14 seconds
including source fingerprinting and setup; it recorded 178 full evaluator
backtests. Its small budget and changed search settings make it unsuitable for
comparison with the earlier V4/V5 benchmark below. A like-for-like benchmark
and peak-memory/CPU trace remain outstanding.

## Measured deterministic V4/V5 comparison

Source: `results/bench_v5/benchmark.md`.

| Metric | V4 | V5 |
|---|---:|---:|
| Wall time | 226 s | 581 s |
| Candidate evaluations | 109,502 | 116,710 |
| Unique tick backtests | 90,957 | 108,875 |
| Summed compile time across workers | 1,365.1 s | 2,980.5 s |
| Summed tick-replay time across workers | 956.8 s | 3,003.7 s |
| Cache hit rate | 0.176 | 0.020 |
| Distinct structures | 16,847 | 20,119 |

Summed per-worker times can exceed wall time because evaluations run in
parallel. The V5 run processed 43,695,626,808 optimization bars and
3,447,569,532 refinement bars according to the search audit counters.

## Interpretation and missing measurements

V5 expanded unique backtests and structural coverage at higher wall-clock cost.
No peak-memory or CPU-utilization trace was collected, so neither is reported.
The benchmark does not establish equivalence against an independent reference
backtester or demonstrate the V6 grammar.
