# BTC Momentum Bot — make.fun btc-race-24h-20261118-1444

A bot that trades Coinbase Wrapped BTC (cbBTC) against USDC, one minute at a
time, on the make.fun `agent-fun/bars-replay/v1` protocol. It starts with
$1,000 of paper money and races only if it made money on the frozen dev
window; it is ranked by net PnL on the 24-hour race window.

## Strategy: 60-minute momentum breakout + 0.8% trailing stop

The dev window is a choppy, slowly falling market (BTC $65,083 → $62,261,
−2.80%) where fast trend-following whipsaws itself to death (the kit's
EMA-cross baseline loses −14.8% in 80 trades) and dip-buying fights the
drift (Bollinger mean reversion loses at every window). So this bot is
patient:

- **Entry** — the close is at least **+0.30% above the close 60 bars ago**,
  and the bot holds nothing. One open position at most, full size (the whole
  cash balance, capped by the weight cap). A 60-minute return above +0.30%
  is a genuine move, not 1-minute noise (1-minute returns have a ~6.7 bps
  standard deviation), so the bot trades rarely: 2 round trips on the dev
  window.
- **Exit** — the close falls more than **0.80% below the highest close since
  the position opened** (a trailing stop that only ever moves up). A sell
  the market refuses is sent again on the next bar while the close stays
  under the stop.
- Each order caps slippage at 100 bps (1%) past the close it was decided
  on, so a fill into a liquidity gap is refused rather than filled for
  almost nothing.

No clock, no environment, no randomness: the same bars give the same orders,
run after run.

## Dev-window result

Graded with the kit's own grader (`node cli/makefun.mjs grade-local --kit .
--bot . --driver process`):

| Metric | Value |
|---|---|
| Net PnL | **+$8.80** (dev gate: above $0 — passes) |
| Trades (fills) | 4 (2 round trips, 1 win / 1 loss) |
| Maximum drawdown | 1.41% (guardrail: ≤ 25%) |
| Fees paid | $1.61 |

The parameter neighbourhood is positive, not a single lucky spike: K = 45–120
bars, entry +0.25–0.50%, trailing stop 0.8% all end the dev window in
profit ($3.18–$8.81).

## Files

- `main.ts` — the bot (Node 22, erasable TypeScript, standard library only).
- `vendor/makefun-bars.ts` — a byte-identical copy of the kit's
  `sdk/makefun-bars.ts` shim, which speaks the protocol for the bot.
- `README.md`, `LICENSE` (MIT), `.makefun/owner.json` (claim file).

## Grade it locally

From the unpacked kit directory:

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repository> --driver process
```
