# V6 preflight smoke test, not the requested full V6 search

# The launcher must use --from 2023-01-06 --to 2025-01-06. This bounds all
# loaded ticks to Dataset A, so this smoke test cannot evaluate B or C.
holdout_months: 0
final_holdout_history: previously_exposed
baseline_gate: report
ema_study: false
search_engine: v5
enable_rule_logic: on
behavioral_novelty: on
entry_types: Market
max_positions_per_strategy: 1
initial_balance: 10000
risk_per_trade_pct: 2
latency_ms: 250
slippage: 0.05
directions: Long, Short, Both
population: 33
generations: 2
discovery_population: 33
discovery_generations: 2
seed_rounds: 1
seed: 20261113
screen_l1: on
screen_audit_rate: 0.05
refine_top: 1
refine_budget: 2
finalists: 1
validation_mode: staged
pool_size: 20
cluster_count: 10
light_keep: 5
perturb_count: 3
mc_count: 2
top_n: 2
checkpoint_every_generation: true
