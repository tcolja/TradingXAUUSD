# Search rules (V5 engine) — short-hold, two-direction search on 3 years of XAUUSD ticks (2023-01-06 .. 2026-01-06)

Only overrides are listed; every other key keeps its built-in default (see `search_rules.md` for the full list).
The engine never forces a number of strategies or trades. 0 passed strategies is a valid outcome.

## Data split
- Dataset A (evolution and internal chronological folds): 2023-01-06 .. 2025-01-06
- Dataset B (frozen-parameter development validation): 2025-01-06 .. 2025-07-06
- Dataset C (final evaluation, only after Dataset A/B selection): 2025-07-06 .. end of file
- Dataset C overlaps prior research using the final 3 months; it is not an untouched holdout.

dataset_a_start_utc: 2023-01-06
dataset_a_end_utc: 2025-01-06
dataset_b_end_utc: 2025-07-06
final_holdout_history: previously_exposed
day_start_hour_utc: 22

## Account and execution
initial_balance: 10000
risk_per_trade_pct: 2.0
max_positions_per_strategy: 2
latency_ms: 250
slippage: 0.05

## Short holding times: <= 10 min earns fitness points, <= 2 min is excellent (30 min hard cap)
max_hold_cap_minutes: 30
max_hold_minutes: 1, 2, 3, 5, 8, 10, 15, 30
target_r: 0.5..3
short_hold_bonus: 0.25
min_conditions: 2
max_conditions: 6
complexity_bias: balanced
fitness.complexity_penalty: 0.01

## Both directions: every strategy must trade long AND short
- Only "Both" genomes are generated, mutated and seeded (EMA study and baseline seeds included).
- The smaller side must take at least 30% of trades (fitness penalty in the GA, hard reject in validation).
- Long trades and short trades must each be net profitable in-sample.
- Extra fitness when both sides are profitable and balanced.

directions: Both
min_side_share_pct: 30
side_must_profit: true
two_sided_bonus: 0.15

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
finalists: 24
validation_mode: classic
checkpoint_minutes: 5
checkpoint_every_generation: true
seed: 20261113

## V5 search engine (families A-K incl. K = Heikin-Ashi, Pareto archive, operator adaptation, A-only refinement, audited L1 screen)
search_engine: v5
screen_l1: on
screen_l1_fraction: 0.25
screen_audit_rate: 0.05
family_warmup_generations: 10
family_min_share: 0.01
family_max_share: 0.40
pareto_archive_size: 500
refine_top: 20
refine_budget: 60

validate_all_stages: on

## Warm start (optional) - seed part of the population with earlier genomes
# Sources: a results.json, a run folder, or a cBot folder (Xau_G######.cs ids are resolved
# against results\**\results.json). cBot-sourced seeds are forced to market entries.
# NOTE: those genomes were picked after seeing Dataset B/C, so B/C of their descendants is
# no longer independent evidence - confirm on newer data or a forward/demo test.
seed_from: Bots\13_11_2026_Export_1; cbots
# seed_ids: G097795
seed_fraction: 0.2
