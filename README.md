# BTC race bot v2 — pullback mean reversion on cbBTC (make.fun btc-race-24h-20261112-1439)

A bot that trades Coinbase Wrapped BTC (cbBTC) against USDC, minute by minute,
on 1-minute on-chain bars from the Orca Whirlpool, with $1,000 of paper money
(simulation only, no real money).

## Strategy

Pullback mean reversion against the 20-bar rolling high of closes:

- **Buy** the whole cash when the latest close is at least 60 bps below the
  highest close of the last 20 bars (a sharp pullback from the recent high).
- **Sell** the whole position when the latest close is at least 80 bps above
  the close of the first batch where the position became visible (the
  pullback has reverted and the take-profit is hit).
- **Trailing stop**: sell if the latest close falls 30 bps below the best
  close seen since the position opened.
- **Time stop**: sell after holding 120 bars, whatever the price.
- **No new entries** once 1,320 closes have been seen: with only ~120 bars
  of runway left before the window ends and positions are flattened, a new
  entry has no time for its take-profit (which takes 70–90 bars to hit).

On the published 24h dev window this makes **+$14.21 net PnL after all fees**
(2 round trips, $1.61 fees, -0.30% max drawdown), passing the dev gate (net
PnL above $0). The two round trips: bought bar 68 at $73,909 and sold bar
159 at $74,595 (+$8.47 net), then bought bar 515 at $73,522 and sold bar 587
at $74,000 (+$5.74 net). Neighbouring parameters in the same family
(lookback 15–20, entry -55 to -60 bps, take +70 to +90 bps, trailing 30–50
bps, hold 120–240 bars, no-entry cutoff anywhere from bar 1,250 to 1,320)
all stay profitable at +$10 to +$14, so the result is a cluster, not one
lucky parameter.

## Why this shape

The dev window is a falling market (BTC -2.3% over 24h): buy-and-hold fails
the dev gate (-$25.97) and fast trend crossovers and oscillators whipsaw
into fees (RSI -$30, Bollinger -$52, EMA cross -$21 to -$166 in the sweep).
The window's money is in two pullback-and-revert moves: a -50 bp dip that
rallies +176 bps, and a later -66 bp dip that recovers +92 bps. A short
20-bar lookback catches the first dip while a 120-bar average is still
warming up (the v1 bot's SMA-120 missed it entirely and made only +$4.27).
The fixed +80 bp take-profit locks the rally in before the chop gives it
back; a trailing-only exit gives back too much (-$1.06 in the same family).
Scale-in, dual-lookback entries, uptick confirmation and armed-trailing exits
were all swept and rejected (churn or never firing). In a downtrend the bot
spends most of its time in cash, which is the correct long-only position.

## Layout

- `main.py` — the bot (Python 3.13, standard library only).
- `vendor/makefun_bars.py` — byte-identical copy of the kit's
  `sdk/makefun_bars.py` protocol shim (agent-fun/bars-replay/v1).
- `LICENSE` — MIT.
- `.makefun/owner.json` — claim file naming the payout wallet.
