# Deep XAUUSD research run — V6-informed, with documented implementation limits

# Three fixed chronological intervals. Dataset C is historically exposed.
dataset_a_start_utc: 2023-01-06
dataset_a_end_utc: 2025-01-06
dataset_b_end_utc: 2025-07-06
final_holdout_history: previously_exposed
day_start_hour_utc: 22

# The account is simulated in XAUUSD quote currency; EUR conversion requires
# matching historical FX data, which was not provided. Broker costs are examples.
initial_balance: 10000
risk_per_trade_pct: 2
max_aggregate_risk_pct: 2
max_positions_per_strategy: 1
latency_ms: 250
slippage: 0.05
entry_types: Market

# Broad intraday search; filters, stop/target, holding time and sessions evolve.
directions: Long, Short, Both
min_conditions: 2
max_conditions: 5
complexity_bias: simple
max_hold_cap_minutes: 240
max_hold_minutes: 2, 5, 10, 15, 30, 60, 120, 240
enable_rule_logic: on
behavioral_novelty: on
novelty_archive_size: 256
novelty_min_distance: 0.12

# Initial population and discovery are additional to the 60,000 optimisation
# offspring slots. No seeded prior finalists and no time-based stop.
search_engine: v5
discovery_population: 300
discovery_generations: 20
population: 300
generations: 200
seed_rounds: 4
seed_tree_parents: 30
elite_pct: 0.10
immigrant_pct: 0.10
stagnation_generations: 10
max_diversity_bursts: 10000
family_warmup_generations: 10
family_min_share: 0.03
family_max_share: 0.30
pareto_archive_size: 500
baseline_gate: report
ema_study: false
seed: 26111307
threads: 8
cache_budget_mb: 2000
checkpoint_every_generation: true

# A-only signal-count screen, with a stable-hash sample of rejected candidates
# fully replayed and recorded per family. This is not a regime-stratified screen.
screen_l1: on
screen_l1_fraction: 0.25
screen_audit_rate: 0.10
refine_top: 15
refine_budget: 20

# Frozen-parameter chronological validation and behavioral clustering.
validation_mode: staged
pool_size: 300
cluster_count: 100
light_folds: 3
light_keep: 40
wf_folds: 5
perturb_count: 30
mc_count: 20
top_n: 20
min_trades: 200
min_profit_factor: 1.30
max_drawdown_pct: 10
min_deflated_sharpe: 0.50
holdout_min_trades: 20
holdout_min_profit_factor: 1.0
