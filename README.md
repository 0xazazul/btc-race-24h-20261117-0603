# btc-race-24h-20261117-0603

[![make.fun · btc-race-24h-20261117-0603](https://make.fun/badge/btc-race-24h-20261117-0603.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![rank](https://make.fun/badge/btc-race-24h-20261117-0603/0xazazul/btc-race-24h-20261117-0603/rank.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![reward](https://make.fun/badge/btc-race-24h-20261117-0603/0xazazul/btc-race-24h-20261117-0603/reward.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)

A long-only cbBTC trading bot for make.fun's "Race a BTC trading bot live for 24 hours" bounty ($250 in USDC). It trades Coinbase Wrapped BTC against USDC minute by minute on paper money, starting with $1,000, and races only if it made money on the published dev window.

## Strategy

Fills land at the next minute's open and pay 4 bps fee + 2 bps slippage + constant-product price impact, so a full-size round trip costs roughly 26–40 bps: a trade has to catch a real move, not noise. The bot has three legs, one position at a time, checked in this order:

1. **TURN — buy the capitulation rebound.** After the price has fallen at least 3.00% over 240 minutes *and* the last 15 minutes have turned up at least 0.30%, buy with all cash. A 4-hour washout that is already turning is the reliable intraday edge: on the previous race's dev window (a −4% day) this leg alone made +$35.64. Exit on take-profit +4.00%, stop-loss −1.00%, a trailing stop 1.50% below the peak since entry, or 480 bars.
2. **DIP — buy the shallow dip, but only when the longer trend is not falling.** After a 60-minute fall of at least 0.80% with the last 15 minutes turning up at least 0.15%, buy with all cash — but only if the 240-minute return is better than −1.00%. The trend filter is what keeps this leg from catching falling knives: without it the same rule loses money on a falling day; with it, it is positive on both observed dev windows. Exit on take-profit +2.00%, stop-loss −1.00%, a trailing stop 1.50% below the peak, or 360 bars.
3. **PARTICIPATION — one small late trade** so the bot always meets the race guardrail of at least one trade. Only if no round trip has completed and the window is past bar 1200: after a 30-minute fall of at least 0.50% with the last 15 minutes turning up at least 0.20%, buy a quarter of the cash; exit on take-profit +1.00%, stop-loss −1.00%, or 120 bars.

All comparisons are exact integer arithmetic on the bars' raw prices — no floats, no clock, no network, no files, no randomness — so two runs on the same data produce the same orders byte for byte.

## Local measurement (dev window, exact replay)

Graded on the kit's frozen 24-hour dev window (1,440 one-minute bars, 2026-11-16T06:03Z–2026-11-17T06:03Z, BTC $64,461 → $64,776) with a replay that reproduces the official grader's numbers for the house baselines to 6 decimals (verified with `grade-local --driver process`):

| Bot | Net PnL | Trades | Max drawdown |
|---|---|---|---|
| **this bot** | **+$29.25** | **4 (2 round trips)** | **0.7%** |
| momentum | +$2.45 | 1 | 2.1% |
| buy-and-hold | +$1.07 | 1 | 2.1% |
| hold-cash | $0.00 | 0 | 0.0% |
| rsi-bollinger | −$24.25 | 11 | 2.8% |
| adx-momentum | −$32.66 | 26 | 3.6% |
| ema-cross | −$106.21 | 68 | 11.0% |

The same bot on the previous race's dev window (2026-11-14–2026-11-15, BTC −4.0%) makes **+$33.30** (3 trades, max drawdown 1.4%), so it is positive on both observed 24-hour windows. The dev window is two 24-hour samples, so these numbers say the bot clears the dev gate with margin on them; they are not a promise about the unseen race window. Block-bootstrap copies of the dev windows average near zero (mean −$2.00 on 50 copies of this window, 24% positive), which is the honest spread: the edge is the dip-rebound structure both real days showed, not a universal money machine.
