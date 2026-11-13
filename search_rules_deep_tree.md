# Search rules

This file is read by `xau-research search` (pass `--rules <file>` to use another file).
Every line of the form `key: value` (or `key = value`, optionally as a bullet or in back-ticks) overrides a default.
Text after ` # ` on a rule line is a comment.
Headings, prose, `#` comments and code fences are ignored. Unknown keys are reported as warnings in the report.
All keys below are listed with their **default** value, so this file reproduces the built-in behaviour unchanged.
Edit the values you want to change.

> The engine never forces a number of strategies or trades. 0 passed strategies is a valid outcome
> and means nothing robust was found with the current data and rules.

## Data and splits

- `holdout_months: 2`  # The most recent N months are never seen by the search. They are only used for the final unseen-data check. 0 disables the check (not recommended).
- `day_start_hour_utc: 22`  # D1/H4 bars start at this UTC hour (the cTrader/NY-close convention).
- `timeframes: M1, M5, M15, M30, H1, H4, D1`  # Timeframes the conditions may use. M1 is always included.

## Strategy shape (genome)

- `min_conditions: 2`  # Minimum conditions per strategy (trigger + filters). Alias: `min_indicators`.
- `max_conditions: 6`  # Maximum conditions per strategy. Alias: `max_indicators`.
- `max_hold_cap_minutes: 360`  # Hard absolute holding-time cap (six hours).
- At most six distinct indicator families are allowed, counting ATR used by the execution plan; strategies exceeding this are repaired before evaluation.
- `directions: Long, Short, Both`
- `entry_types: Market, Stop, Limit`  # Stop and Limit place a pending order offset by ATR, with an expiry.
- `stop_modes: Atr, Fixed, Structure`
- `max_hold_minutes: 2, 3, 5, 10, 15, 30, 60, 120, 240, 360`  # Allowed maximum holding times.
- `target_r: 0.5..6`  # Take-profit range in R (multiples of the stop distance).
- `stop_atr: 0.5..5`  # Stop range in ATR multiples (Atr mode).
- `stop_usd: 0.5..15`  # Stop range as a price distance in USD per ounce (Fixed mode).
- `forbidden_indicators: none`  # Comma list of indicator families, e.g. `Ichimoku, PSAR`.
- `required_indicators: none`  # At least one of these families must appear in every strategy.
- `forbidden_conditions: none`  # Comma list of condition kinds (see `xau-research catalog`).
- `required_conditions: none`
- `archetypes: none`  # Restrict triggers to archetypes, e.g. `Breakout, MeanReversion` (empty = all).
- `range.<Condition>.<param>: min..max step s` — Override one parameter range. Example: `range.RsiReversal.period: 7..21 step 1`.

## Genetic algorithm

- `discovery_population: 250`
- `discovery_generations: 250`
- `population: 250`
- `generations: 200`
- `crossover_rate: 0.7`
- `mutation_rate: 0.2`  # Adaptive. It rises up to `max_mutation_rate` while the search stagnates.
- `max_mutation_rate: 0.6`
- `elite_pct: 0.05`
- `immigrant_pct: 0.03`  # Fraction of fresh random genomes per generation.
- `stagnation_generations: 25`
- `finalists: 40`  # Candidates passed to validation.
- `niche_max_share: 1.0`  # Max population share of one trigger kind (e.g. 0.08). 1.0 = no cap.
- `distinct_finalists: off`  # off | conditions (finalists share no trigger/filter kind) | families (share no indicator family).
- `seed: 12345`  # The same data, rules and seed give the same result.
- `cache_budget_mb: 3000`  # Memory budget for cached indicator/condition arrays.
- `time_limit_hours: 0`  # 0 = no limit. When the limit is reached the search stops and reports partial results.
- Thread count defaults to CPU count - 1; add a `threads: N` line to change it.

## Fitness (final score weights)

- `fitness.drawdown: 0.25`
- `fitness.robustness: 0.20`
- `fitness.walk_forward: 0.15`
- `fitness.monte_carlo: 0.15`
- `fitness.profit_factor: 0.10`
- `fitness.stability: 0.10`
- `fitness.net_profit: 0.05`
- `fitness.frequency_bonus: 0.05`  # Small bonus for more trades. It is never a quota.
- `fitness.complexity_penalty: 0.02`  # Per condition above the minimum.

Custom objectives and hard constraints on any metric are also supported:

- `objective.<Metric>: weight` — For example `objective.Sortino: 0.1`.
- `constraint.<Metric>: >= value` or `<= value` — For example `constraint.WorstMonthPct: >= -5`.

Available metrics: AnnualReturnPct, AvgHoldMinutes, AvgLossR, AvgWinR, DailyKurtosis, DailySharpeRaw, DailySkew, EquityR2,
ExpectancyR, ExpectancyUsd, ExposurePct, GrossLoss, GrossProfit, LongNet, Longs, MaxDrawdownPct, MaxDrawdownUsd,
MaxLosingStreak, MedianHoldMinutes, Months, NetProfit, PositiveMonthsPct, ProfitFactor, ReturnPct, ReturnToDrawdown,
RTStat, Sharpe, ShortNet, Shorts, Sortino, TotalCommission, TotalSwap, Trades, TradesPerDay, TradingDays, WinRate, WorstMonthPct.

## Acceptance gates (a strategy must pass all of them)

- `min_profit_factor: 1.30`
- `preferred_profit_factor: 1.50`  # Used in scoring, not a gate.
- `max_drawdown_pct: 25`
- `min_trades: 30`  # Minimum for statistical validity. It is not a target.
- `min_walk_forward_efficiency: 0`  # Out-of-sample/in-sample performance ratio must be above this. Alias: `min_wfe`.
- `min_wf_profitable_windows_pct: 50`
- `holdout_min_profit_factor: 1.0`
- `holdout_min_trades: 3`
- `min_robustness_score: 0.5`  # Neighbour-parameter and noise stability (0..1).
- `min_deflated_sharpe: 0.5`  # Deflated Sharpe probability. It corrects for the thousands of strategies tried during the search. On a pure random walk the best "strategies" score about 0.1, so they are rejected. 0 makes it informational only.
- `min_mc_profit_probability: 0.90`  # Trade-order Monte Carlo probability of ending in profit.
- `min_execution_profitable_pct: 80`  # % of randomized-execution runs (latency, slippage, spread, partial fills) that stay profitable.

## Validation

- `wf_train_months: 6`  # Reduced automatically when there is not enough history (a warning is reported).
- `wf_test_months: 1`
- `wf_reoptimize: true`  # Re-optimize parameters in each training window (true walk-forward).
- `wf_population: 24`
- `wf_generations: 10`
- `mc_runs: 1000`
- `mc_execution_runs: 50`
- `robustness_neighbours: 12`
- `noise_runs: 2`
- `noise_atr_fraction: 0.1`  # Price-noise test amplitude as a fraction of ATR.
- `stress_latency_ms: 1000`
- `stress_slippage: 0.15`
- `checkpoint_minutes: 5`  # GA checkpoint interval; phase boundaries are also saved.
- `fitness_profile: classic`  # classic | portfolio
- `validation_mode: classic`  # classic | staged
- `pool_size: 300`
- `cluster_count: 100`
- `cluster_similarity: 0.6`
- `light_folds: 3`
- `light_keep: 40`
- `wf_folds: 5`
- `perturb_count: 30`
- `mc_count: 20`
- `mc_top_count: 5`
- `mc_top_runs: 1000`
- `top_n: 20`

## Portfolio

- `max_strategies: 10`  # An upper limit, not a target.
- `max_correlation: 0.6`  # Maximum daily-PnL correlation between portfolio members.
- `max_portfolio_open_risk_pct: 6`

## Execution and account (backtester and exported cBots)

- `initial_balance: 10000`
- `risk_per_trade_pct: 2.0`
- `max_positions_per_strategy: 2`
- `latency_ms: 250`
- `slippage: 0.05`  # USD per ounce, applied adversely to market and stop fills.
- `extra_spread: 0`  # Added on top of the recorded tick spread.
- `friday_no_entry_utc: 19:00`
- `friday_close_utc: 19:45`


# ---------------------------------------------------------------------------
# Deep seed-tree search (2026-11-12): combine as many indicators as possible,
# grow promising genomes into new seed trees each round. 500 population x 500 generations.
# ---------------------------------------------------------------------------
holdout_months: 3
initial_balance: 10000
risk_per_trade_pct: 2.0
max_hold_cap_minutes: 360
max_hold_minutes: 5, 10, 15, 30, 60, 120, 240, 360

# Explicit user override: the baseline screen is still run and reported, but does not block the GA.
baseline_gate: report

min_conditions: 3
max_conditions: 6
complexity_bias: rich
fitness.complexity_penalty: 0.005

discovery_population: 300
discovery_generations: 60
population: 500
generations: 500
seed_rounds: 5
seed_tree_parents: 30
stagnation_generations: 20
max_diversity_bursts: 6
niche_max_share: 0.08

distinct_finalists: off
finalists: 20
validation_mode: classic
min_trades: 100
min_profit_factor: 1.30
max_drawdown_pct: 10
min_deflated_sharpe: 0.50
checkpoint_minutes: 5
seed: 20261112

