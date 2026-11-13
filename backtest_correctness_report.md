# Backtest correctness report

## V6-informed update, 2026-11-13

The simulator enforces a configurable aggregate planned-risk cap and does not
round minimum volume above the risk budget. A stop-entry gap legitimately can
increase risk after the pending order was placed; a new test verifies that its
size is not changed using future knowledge. The deep run restricts itself to
market entries and one position per strategy while the other execution cases
remain less thoroughly verified.

The Release suite passed 70 tests, including bid/ask, short-entry commission,
slippage and holding-time cases, a 60-path independent single-trade market
reference comparison, and truncated-history catalog checks on M1 and M15. This
does not establish full simulator parity or EUR currency conversion. Broker
cost assumptions remain provisional.

## Existing implementation evidence

The current simulator replays tick bid/ask quotes. The implementation documents
long entries at ask and exits at bid, and short entries at bid and exits at ask.
It checks stop-loss before take-profit on each tick, includes configured
commission and slippage, and calculates position size from configured risk.
Existing `BacktesterTests` cover long ask entry / bid take-profit, stop-loss,
and risk-based position sizing.

## Validation status

`dotnet test tests\\Core.Tests -c Release` passed: 54 passed, 0 failed,
including the new family-budget allocation regression test.

This is not a complete verification of the V6 backtest checklist. The current
test suite does not establish accelerated/reference parity, EUR conversion,
margin enforcement, partial fills, or every stop/target ambiguity case. The
V5 benchmark includes configured historical spread, commission, and slippage,
but broker specifications and cost sensitivity remain configuration-dependent.
