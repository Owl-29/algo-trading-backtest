# 📈 Algo Trading Backtest
### Mean-Reversion Strategy Research, Freshman Quant Project

A backtesting framework I built to test three mean-reversion trading strategies against buy-and-hold across different asset types, market conditions, and parameter settings. Built with Python, `yfinance`, `pandas`, `numpy`, and `matplotlib`.

---

## The Question I Was Trying to Answer

Do simple mean-reversion rules actually work in practice, and does the answer change depending on what kind of asset you're trading or what the market is doing?

---

## Strategies I Tested

| Strategy | Logic |
|---|---|
| **Buy & Hold** | Benchmark, buy at the start and hold the whole time |
| **Red Days** | Buy after N consecutive down days, hold for X days |
| **Drop / Rise** | Buy on a single-day drop ≥ X%, sell when price rises Y% from entry |
| **Z-Score Mean Reversion** | Buy when the rolling Z-score is very negative (price is statistically cheap, sell when it normalizes |

---

## Tickers

| Ticker | Type |
|---|---|
| SPY | Stable growth ETF |
| MSFT | Large-cap growth |
| GOOGL | Large-cap growth |
| NVDA | High-risk growth |
| HGRAF | Speculative penny stock |

I also ran a separate test on an alternative universe: `GLD, TLT, BTC-USD, XOM, AMZN`

---

## How It Works

1. Pull historical daily close prices via `yfinance`
2. Compute daily returns and cumulative portfolio growth
3. Apply each strategy's buy/sell logic
4. Simulate trades with configurable transaction costs
5. Compare everything to buy-and-hold using both return and risk metrics
6. Run across multiple tickers, parameter settings, and date ranges

I used the same parameters across all tickers intentionally, no per-stock tuning, so the comparison is fair and the results aren't just overfitted to one asset.

---

## Metrics I'm Tracking

- Final portfolio value and profit/loss
- Total return
- Sharpe ratio (annualised, risk-free rate = 0)
- Average daily return + daily volatility
- Max drawdown
- Signal count, completed trades, win rate

---

## Runs

| Run | What I was testing | Date Range |
|---|---|---|
| `baseline` | Standard params, main tickers | 2020–today |
| `long_term` | Patient params (3 red days, drop 3%/rise 5%) | 2020–today |
| `short_term` | Aggressive params (1 red day, drop 1%/rise 1.5%) | 2020–today |
| `bull_market` | Pre-COVID calm bull run | 2017–2020 |
| `covid_era` | COVID crash + recovery | 2020–2021 |
| `inflation_bear` | Rate hikes + bear market | 2022–2023 |
| `alt_assets` | Gold, bonds, BTC, oil, Amazon | 2020–today |
| `no_cost` | 0% transaction cost | 2020–today |
| `high_cost` | 0.5% per-trade cost | 2020–today |

---

## What I Found

### Red Days never worked. Not once.
This was probably the clearest result, the consecutive red days strategy lost money in literally every single test. Every ticker, every run, every parameter variation. Average total return was −80.6%. My guess is it gets crushed in trending markets because it keeps buying during downtrends, and in volatile markets it gets whipsawed constantly. I went in thinking this might work as a short-term bounce strategy. It doesn't.

### Drop / Rise was the best active strategy
93.7% win rate across all completed trades. It consistently made money by buying sharp single-day drops and waiting for a specific recovery target before selling. It worked best on volatile assets, NVDA and HGRAF, where panic selling is more common and reversals tend to follow. On the aggressive short_term run, it returned +3,773% on HGRAF, though HGRAF is a penny stock with unreliable data so I'd take that number with a grain of salt.

### Z-Score actually shone during the bear market
During the inflation/bear era (2022–2023), Z-Score outperformed buy-and-hold on a risk-adjusted basis. Sharpe of 1.12 on MSFT and 0.77 on GOOGL, compared to buy-and-hold Sharpes of 0.37 and 0.16 for those same tickers in that period. Makes sense in hindsight, mean reversion needs prices to move in both directions to generate signals, and choppy bear markets provide exactly that.

### Buy & Hold still won overall, but not in every regime
In the full baseline run (2020–today), buy-and-hold dominated because of NVDA (+2,850% over the period) and GOOGL (+368%). Hard to beat that. But in the inflation/bear era, buy-and-hold averaged only +8% across tickers while Drop/Rise averaged +17% and Z-Score averaged +22%. So the active strategies didn't fail, they just need the right environment.

### Transaction costs barely mattered
Comparing 0%, 0.1%, and 0.5% per-trade costs: Drop/Rise returned 7.23%, 7.61%, and 7.28% respectively. Almost no difference. The strategies don't trade often enough for costs to compound badly, which is a good sign for robustness.

### The alt asset run was interesting
Drop/Rise averaged +155% on the alternative universe, mostly driven by BTC and XOM. XOM was the one ticker where Drop/Rise consistently beat buy-and-hold, probably because oil stocks have more predictable cyclical volatility than tech.

---

## Limitations

- No position sizing, strategies are all-in or all-out, which isn't realistic
- Parameters were chosen manually, not optimised, so there's no guarantee the best settings were found
- HGRAF data from Yahoo Finance may not be reliable, penny stock results should be treated carefully
- No short selling, strategies only go long
- All tickers are known successful companies, a random stock universe would probably show weaker results

---

## How to Run

```bash
git clone https://github.com/Owl-29/algo-trading-backtest.git
cd algo-trading-backtest
pip install yfinance pandas numpy matplotlib
jupyter lab Algo_Trading_Backtest.ipynb
```

The notebook will prompt you for all inputs, tickers, dates, parameters, transaction cost. Results append to `master_strategy_summary.csv` so runs accumulate over time.

---

## Output Files

| File | What's in it |
|---|---|
| `master_strategy_summary.csv` | All results, one row per ticker per strategy per run |
| `master_strategy2_trade_log.csv` | Individual trade log for Drop/Rise |
| `master_strategy3_trade_log.csv` | Individual trade log for Z-Score |
| `charts/` | PNG charts for every ticker and run |

---

*Freshman quant research project, Lehigh University*
