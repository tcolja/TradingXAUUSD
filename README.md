# XAUUSD autonomous strategy research (cTrader)

This tool searches XAUUSD bid/ask **tick data** for rule-based M1 strategies and validates them. Strategies that survive are exported as cTrader cBots.

- **Source of truth:** raw ticks (time, bid, ask).
- **Bars:** M1 and higher-timeframe bars are built from the ticks, so they are consistent by construction.
- **Execution:** every backtest replays the real ticks. Longs fill at ask and exit at bid; shorts do the opposite. The real spread, latency, slippage, commission and swap are applied.
- **No quotas:** the engine is never asked for a number of strategies or trades. **0 strategies is a valid result.** It means nothing robust was found with the given data and rules.
- **Baseline before GA:** each search first screens 12 established strategy families with two fixed configurations each, using six months for training and three for development. The final three months of a year-long dataset are not touched by this baseline stage; the GA starts only if at least one baseline clears preliminary gates.
- **Complexity cap:** at most six distinct indicator families per strategy, including ATR used by execution; preferred strategies remain simpler.
- **Risk assumption:** research starts at a 10,000 account balance and 2% risk per trade. XAUUSD P&L is currently calculated in the symbol's quote currency; EUR account-currency conversion is not modeled without a matching FX history.

## 1. Get the data (Dukascopy ticks)

```powershell
npx dukascopy-node -i xauusd -from 2021-01-01 -to 2026-01-01 -t tick -f csv -v true -dir data\raw
```

- Columns: `timestamp,askPrice,bidPrice,askVolume,bidVolume` (Unix ms, UTC).
- Other CSVs work too, as long as the header contains a time column (`timestamp|time|datetime|date`) plus `bid` and `ask`.
- Download in yearly chunks if it is slow. Several files or a whole folder can be passed to `--input`.
- Overlapping ranges are de-duplicated.
- **More history = more trust.** Aim for at least 2-3 years. With less, walk-forward windows shrink and the report says so.

Check the data before searching:

```powershell
dotnet run --project src\Research.Cli -c Release -- prepare --input data\raw
```

This parses the files and writes `data\cache` (a fast binary cache) plus a data-quality report covering gaps, spikes, crossed quotes, duplicates and spread statistics.

- Suspicious ticks are **flagged or excluded, never invented or interpolated**.
- If the report says **BLOCKING**, the files are corrupt or incomplete. Re-download them.

## 2. Configure

- `config\symbol.xauusd.json`: broker contract details: lot size, volume step, commission and swaps.
  - **Replace the example swap values with your broker's.**
  - Commission is USD per lot per side.
- `search_rules.md`: every search, fitness, acceptance-gate, validation and execution setting, listed with its default.
  - Edit values there. Unknown or invalid keys are reported as warnings.

## 3. Search

```powershell
dotnet run --project src\Research.Cli -c Release -- search --input data\raw --out results\run1
```

For the currently available year of data, `search_rules_initial.md` uses a 6-month training / 3-month development / 3-month untouched split and an initial 100 × 25 GA budget. The project data currently ends on 2026-01-06; do not treat it as data through 2026-11.

- `--from 2022-01-01 --to 2025-12-31` restricts the data range.
- `--quick` runs a fast smoke test with small populations.
- `--population <n>` and `--generations <n>` override the GA size for both discovery and optimisation.
- `--diverse` keeps many indicator combinations alive (one trigger kind may hold at most 8% of the population) and makes every finalist use completely different indicator families from all other finalists.
- The console (and `search.log`) shows overall % done + phase every 30 s and the best candidate every 60 s.
- Ctrl+C stops gracefully and still writes partial reports.

### Background run with a live monitor window

```powershell
powershell -ExecutionPolicy Bypass -File tools\start_search.ps1 `
    -Input data\xauusd-tick-2023-01-06-2026-01-06.csv -Rules search_rules_ema3y.md -Out results\ema3y_p300_g200
```

- `start_search.ps1` builds the solution and refuses to start if the PID in `<Out>\run.json` is still alive. It then starts the search hidden (`stdout.log` / `stderr.log`), records the run ID, PID and command in `run.json`, and opens a second PowerShell window running `tools\monitor.ps1`.
- Optional arguments:
  - `-ExtraArgs '--population 20 --generations 3'` passes extra CLI arguments.
  - `-NoBuild` and `-NoMonitor` skip the build and the monitor window.
- The engine writes `<Out>\progress.json` atomically every 10 s. It contains:
  - phase and phase %, overall %, ETA, and generation/population;
  - genomes processed in the current generation;
  - main GA evaluations completed vs planned (population × (generations + 1));
  - promising, validated and passed counts;
  - the best candidate, the best validated candidate, a heartbeat, and errors.
- The monitor prints:
  - every 30 s, the status in cyan (promising counts in green; stalls, crashes and errors in red);
  - every 60 s, a yellow **BEST FOUND CANDIDATE** block, or "NO VALIDATED CANDIDATE FOUND YET", followed by the best not-yet-validated candidate. The block shows PF, max DD, trades/day, net per trading day and per calendar day after costs, and average hold.
- To reopen the monitor: `powershell -ExecutionPolicy Bypass -File tools\monitor.ps1 -Dir results\ema3y_p300_g200`.

### EMA behaviour conditions and genome protection

- **EMA conditions:**
  - `EmaTrendSide` (filter): price on one side of the EMA.
  - `EmaStretch` (filter): distance from the EMA, in ATR.
  - `EmaRejection` (trigger): a trend pulls back, touches the EMA and is rejected.
  - `EmaBreakReversal` (trigger): a trend breaks through the EMA, signalling a reversal.
  - `EmaStretchSnap` (trigger): price snaps back after stretching far from the EMA.
- `ema_study: true` measures win rates of these conditions for EMA periods 9–200 on M1/M5/M15/H1 before the GA, writes `ema_study.md`, and seeds the best EMA setups into discovery and optimisation.
- `protect_viable: true` with `kill_fitness_threshold: -0.6`: immigrants and diversity bursts only replace genomes whose fitness is below the threshold (about PF < 0.8 or no trades), so promising genomes are not killed.

### Short holds and both directions (`search_rules_scalp_both.md`)

These keys are off by default. Bonuses apply only to profitable strategies and scale with profit factor (full at PF ≥ 1.3, none at PF ≤ 1), so fast losers earn nothing.

- `short_hold_bonus: 0.25`:
  - adds fitness points for trades held ≤ 10 min (0.5 point each) and ≤ 2 min (1 point each, "excellent");
  - points are averaged over all trades and multiplied by the weight;
  - if short trades lose money in aggregate, they earn only a quarter of the points.
- `two_sided_bonus: 0.15`: extra fitness when both the long and the short side are profitable, scaled by how balanced their trade counts are.
- `directions: Both` (used by `search_rules_scalp_both.md`): every genome — random, mutated, crossed, EMA-study and baseline seeds — trades both long and short.
- `min_side_share_pct: 30`: the less-used side must take at least 30 % of trades. Below that, fitness is scaled down (to 0.2× for a one-sided strategy), and validation rejects the candidate.
- `side_must_profit: true`: long and short trades must each be net profitable in-sample, on Dataset B and (for selected finalists) Dataset C. Failures are penalized during search and rejected in validation.
- `dataset_a_start_utc`, `dataset_a_end_utc`, `dataset_b_end_utc`: optionally define chronological A/B/C intervals. Evolution, EMA study and internal folds use A only; B is frozen-parameter development validation; C is evaluated once for the frozen portfolio shortlist.
- `final_holdout_history`: set to `untouched` only if Dataset C has not informed previous research. `unknown` and `previously_exposed` are reported as such, never labeled unseen.
- `checkpoint_every_generation: true`: atomically save GA progress after each generation instead of using only the interval.
- `direction_min_share: 0.20` / `finalists_per_direction: 4`: when several directions are allowed (e.g. `directions: Long, Short, Both`), each keeps at least 20 % of the GA population and the best 4 of each direction get a guaranteed finalist slot.
- New metrics in reports, `progress.json` and the monitor:
  - `PctHoldLe10Min`, `PctHoldLe2Min`, `NetHoldLe10Min`;
  - long and short trade counts and net profit, `MinSideSharePct`.
  - They can also be used as `constraint.` or `objective.` keys.

```powershell
powershell -ExecutionPolicy Bypass -File tools\start_search.ps1 -Input data\xauusd-tick-2023-01-06-2026-01-06.csv -Rules search_rules_scalp_both.md -Out results\scalp_both_p300_g200
```

### Pipeline

1. Validate tick data and build M1 / higher-timeframe bars.
2. Screen fixed strategy-family baselines on Dataset A and Dataset B; Dataset C is excluded. Without fixed boundaries, the legacy relative split is used. If no family shows a preliminary repeatable edge, the GA is skipped and reported as such (set `baseline_gate: report` to run the GA anyway while still reporting the baseline).
   - Seed-tree search: `seed_rounds: N` splits optimisation into N rounds. After each round the best `seed_tree_parents` distinct structures survive as roots, pairs of roots are grafted (filter chains merged up to `max_conditions`), roots are branched by mutation, and these seed the next round. `complexity_bias: simple|balanced|rich` controls how many filters random and crossover genomes carry. See `search_rules_deep_tree.md`.
3. Discovery GA, EMA study and evolutionary search use Dataset A only when fixed boundaries are configured.
4. Validation of each finalist:
   - Neighbour-parameter robustness and price noise.
   - Execution stress and randomized execution.
   - Internal chronological walk-forward on Dataset A and frozen-parameter development checks on Dataset B.
   - Trade-order Monte Carlo.
   - Deflated Sharpe, correcting for the number of strategies tried.
   - Dataset C is withheld until A/B selection is frozen; only selected portfolio candidates are evaluated on C.
5. Portfolio selection by daily-PnL correlation.

### Outputs in `results\run1`

| File | Content |
|---|---|
| `report.md` | Data quality, warnings, every validated strategy with metrics, rejection reasons and the portfolio |
| `results.json` | Machine-readable results (used by `export`) |
| `candidates.csv` | All finalists with key metrics |
| `trades\<id>.csv` | Every trade of each passed strategy |
| `portfolio_correlation.csv` | Correlation matrix of the passed strategies |
| `cbots\Xau_<id>.cs / .cbotset / .md` | cTrader cBot source, parameter preset and strategy documentation |
| `effective_settings.json`, `search.log` | Exact settings used, and the run log |
| `baseline.json`, `baseline.csv` | Fixed baseline-family configurations with separate training/development results and rejection reasons |

Regenerate a single cBot later:

```powershell
dotnet run --project src\Research.Cli -c Release -- export --results results\run1\results.json --id G000425 --out exported
```

## 4. Import into cTrader

1. In cTrader Algo, create a **new cBot** (one cTrader project per strategy) and name it like the class, e.g. `Xau_G000425`.
2. Replace the code with the content of `Xau_G000425.cs`.
3. Build. It targets the standard `cAlgo.API` and needs no extra access rights.
4. Attach it to an **XAUUSD m1** chart. You can either:
   - load `Xau_G000425.cbotset` via *Parameters → Load*, or
   - leave the defaults, which are already the optimized values.
5. Backtest in cTrader with **tick data** first, then demo trade before any live use.

**`.cbotset` decimal separator:** cTrader reads preset numbers using the Windows regional setting.

- The tool writes presets in this PC's format by default (for example `1,5` on a Slovenian locale).
- If cTrader runs on a PC with different settings, regenerate the preset with `--preset-decimal .` or `--preset-decimal ,`.

### Parity notes (research backtest vs. cTrader)

- **Signal timing:**
  - Signals use closed bid bars only. The decision is made on the first tick after the bar closes, like cTrader's `OnBar`.
  - Higher timeframes use only bars that have **fully closed**.
- **Indicators:** they are computed by the same shared source code that is embedded in the cBot. They do not depend on cTrader's built-in indicators.
- **Expected differences** from a cTrader backtest:
  - Broker feed versus Dukascopy prices and spreads.
  - Real commission and swap schedules.
  - The broker's stop-level and volume rules.
  - Weekend and holiday sessions.
  - Results will be close, not identical.
- **Treat the holdout and walk-forward numbers as the realistic estimate**, not the in-sample ones.

### Visual mode (default): TradingView-style signal boxes, no orders

Exported cBots **do not trade** by default. `LiveExecutionEnabled = false`, and the call `// LiveTradeExecution(dir, atr);` in `OnStrategySignal` is commented out.

For every signal the bot draws:
- a green take-profit rectangle and a red stop-loss rectangle, from the entry for `ProjectionBars` (default 10) candles into the future;
- an entry line (dashed while a Stop/Limit order is pending), an arrow, and a label with entry, TP, SL, R:R and ID;
- the result once the virtual trade closes: TP, SL, time or Friday exit, with R and net after `VirtualCost`.

On start it also does the following:
- replays the last `ReplayBars` M1 bars and draws the last `HistoricalSignalLimit` historical signals;
- draws the strategy's EMA line(s) in gold;
- shows a statistics panel with historical and live virtual win rate, PF and net.

To test in cTrader:
1. Attach the bot to an XAUUSD m1 chart, or run a **visual backtest** with tick data.
2. Watch whether the boxes appear where the pattern really occurs.

To enable real orders, set `LiveExecutionEnabled = true` and uncomment the `LiveTradeExecution` call. Only do this after demo testing.

## 5. Other commands

```powershell
# M1 feature dataset (bar data, spread, time and all condition values on closed bars) for ML experiments
dotnet run --project src\Research.Cli -c Release -- features --input data\raw --out results\features.csv
# List every condition and its parameter ranges (for search_rules.md range./required_conditions keys)
dotnet run --project src\Research.Cli -c Release -- catalog
# Synthetic ticks for pipeline smoke tests (NOT for real research)
dotnet run --project src\Research.Cli -c Release -- synth --out data\synth\synth.csv --days 150
dotnet run --project src\Research.Cli -c Release -- synth --out data\rw\rw.csv --days 150 --random-walk
```

**Sanity check:**

- A pure random walk (`--random-walk`) should produce **0** passed strategies. Strategies found there would be overfitting.
- The default synthetic data contains a real momentum edge, so the search should find something.

## Development

```powershell
dotnet build XauusdResearch.slnx -c Release
dotnet test tests\Core.Tests
```

### Projects

| Project | Contents |
|---|---|
| `src\Core.Data` | CSV reader, tick validation, binary cache, bar builder |
| `src\Core.Indicators` | Indicators; the shared code is also embedded in the cBots |
| `src\Core.Backtester` | Tick-replay backtester and metrics |
| `src\Core.SearchEngine` | Conditions, genetic algorithm, validation, portfolio |
| `src\Core.Reporting` | Reports |
| `src\Core.Export` | cBot generator |
| `src\Core.ML` | Feature dataset and the ML extension point |
| `src\Research.Cli` | The `xau-research` command-line tool |

The tests compile generated cBots against the real `cTrader.Automate` API (cAlgo.API) to make sure exports always build.
