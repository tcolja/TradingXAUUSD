# Initial XAUUSD research budget
# Based on the latest complete tick file (2025-01-06 through 2026-01-06).
# Preserve 6 months for training, 3 for development, and the final 3 months for unseen validation.

holdout_months: 3
initial_balance: 10000
risk_per_trade_pct: 2.0
min_conditions: 2
max_conditions: 6
max_hold_cap_minutes: 360
max_hold_minutes: 5, 10, 15, 30, 60, 120, 240, 360

discovery_population: 100
discovery_generations: 25
population: 100
generations: 25
finalists: 8
seed: 12345

min_trades: 30
min_profit_factor: 1.30
max_drawdown_pct: 10
min_deflated_sharpe: 0.50

checkpoint_minutes: 5
fitness_profile: classic
validation_mode: classic
