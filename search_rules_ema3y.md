# Search rules — EMA intraday search on 3 years of XAUUSD ticks (2023-01-06 .. 2026-01-06)

Only overrides are listed; every other key keeps its built-in default (see `search_rules.md` for the full list).
The engine never forces a number of strategies or trades. 0 passed strategies is a valid outcome.

## Data split
- Search (train + development via anchored walk-forward folds): 2023-01-06 .. ~2025-10-05
- Final unseen test (evaluated once, after the search): the last 3 months, ~2025-10-06 .. 2026-01-06

holdout_months: 3
day_start_hour_utc: 22

## Account and execution (spec: 10k, 2 % risk, max 2 positions)
initial_balance: 10000
risk_per_trade_pct: 2.0
max_positions_per_strategy: 2
latency_ms: 250
slippage: 0.05

## Intraday shape (hold 1-5 min preferred, up to 30 min fine, 6 h hard cap; R:R 0.75-3)
max_hold_cap_minutes: 360
max_hold_minutes: 2, 3, 5, 10, 15, 30, 60, 120, 240, 360
target_r: 0.75..3
min_conditions: 2
max_conditions: 6
complexity_bias: balanced
fitness.complexity_penalty: 0.01

## EMA behaviour study (touch / rejection / break statistics) feeds EMA seeds into the GA
ema_study: true

## Genetic algorithm: 300 x 200, keep promising genomes alive, only kill clearly broken ones
discovery_population: 300
discovery_generations: 40
population: 300
generations: 200
seed_rounds: 4
seed_tree_parents: 30
protect_viable: true
kill_fitness_threshold: -0.6
stagnation_generations: 20
max_diversity_bursts: 1000
niche_max_share: 0.10
elite_pct: 0.10
immigrant_pct: 0.10
baseline_gate: report

## Acceptance gates (minimums for statistical validity, never targets)
min_trades: 200
min_profit_factor: 1.30
max_drawdown_pct: 10
min_deflated_sharpe: 0.50
holdout_min_trades: 20

## Validation
distinct_finalists: off
finalists: 20
validation_mode: classic
checkpoint_minutes: 5
seed: 20260106
