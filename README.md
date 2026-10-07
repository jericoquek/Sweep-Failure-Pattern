[README (4).md](https://github.com/user-attachments/files/33139834/README.4.md)
# Swing Failure Pattern

A Pine Script v6 indicator for TradingView. It marks swing failure patterns at structural highs and lows on the 15-minute chart, then tracks whether price follows through with a short-term and an intermediate-term market structure shift. The intermediate-term shift is what flips the MMXM program between BUY and SELL.

I built this for MNQ. I read structure on the 15m and execute on the 1m, so the indicator works the same way: every signal comes from the 15m, and lower timeframes only display it. I also trade a NAS100 perpetual on the same CME session timing, and that's what the **CME hours only** option is for.

## What I count as an SFP

Price trades through a high or low that has been standing as liquidity, can't hold beyond it, and closes back inside. That's the pattern. Where it happens matters more than the pattern itself. An SFP at the extreme of a move is worth watching. An SFP in the middle of a trend, on some higher low inside a rally, is noise, and this indicator is built to leave those out.

## How it works

**Liquidity levels.** Swing highs and lows on the 15m become standing liquidity. A swing has to dominate a few bars to its left and needs one bar on its right to confirm. Levels within a few ticks of each other are treated as one level, and the extreme of the cluster is kept. A level stays live until price deals with it. There is no clock on it.

**The sweep.** A wick trades beyond a standing level. The sweep has to make a new extreme for the current program. If price sweeps a secondary low while the real low of the move is still standing below it, that's an internal sweep and it doesn't count. When one wick takes out several levels, the deepest one becomes the reference.

**The reclaim.** A later 15m candle closes back inside the swept level with a real body in the signal direction. The sweep candle can't confirm itself; that's the rule I was taught and it stays on by default. If price holds beyond the level for too long instead, it's treated as acceptance and the raid is dropped.

**ST MSS.** After the SFP, a close through the most recent short-term swing on the other side prints ST MSS. This is the early confirmation, the point where I start looking for an entry on the lower timeframe. It doesn't change the program on its own.

**IT MSS.** A close through the intermediate-term swing confirms the shift and flips the MMXM program to BUY or SELL. Intermediate-term swings are built from the short-term ones.

**Staying out of the trend.** Two rules keep the indicator quiet inside a delivery. In a BUY program, a bullish SFP only counts if it takes out the low the program started from, and the reverse applies in a SELL program. And a reversal against a program is only allowed once that program has been running for 16 hours. Right after a transition the program's range is a few bars wide, and the first pullback would otherwise look like a new extreme.

## Why 15m only

The whole model runs on the 15m and nowhere else. On a 15m chart the engine runs directly. On a 1m, 3m or 5m chart, only closed 15m candles are projected down, so a 15m SFP shows up on the 1m at the first 1m bar after the 15m candle closes. Nothing appears mid-candle and nothing repaints.

Charts above 15m aren't supported. On a 30m or 1h chart nothing is drawn and the state badge says `15m MODEL · Use 15m or lower`.

## CME hours on a 24/7 symbol

The perp I trade prints bars around the clock. MNQ doesn't. It stops from Friday 17:00 to Sunday 18:00 New York time, and for the 17:00 hour every weekday. On the perp, about a third of the 15m bars fall in those closed hours, and the defaults were never tuned on them.

Leaving them in doesn't mostly show up as signals printing on a Saturday. It's quieter than that. Weekend bars form swings, act as the neighbour bar of a swing, and run up the bar counts behind the timers, so the signals during normal hours drift away from what the same structure would give on MNQ.

Tick **CME hours only** and the engine reads CME-hours bars only, as if the rest don't exist:

- A closed-hours bar never registers a level, sweeps one, prints an SFP or MSS, or flips the program.
- The bar after Friday 16:45 is Sunday 18:00, and the bar after 16:45 on a weekday is 18:00.
- Every timer counts open bars only, so the 16-hour maturity gate means 16 hours of CME trading.
- State holds through the closure. The MMXM badge on Sunday at 18:00 shows what it showed on Friday at 17:00.
- A level that only weekend price traded through is still standing at Sunday 18:00. If Sunday opens beyond a level, that bar sweeps it, as it would on MNQ.

Off by default. Off, the indicator runs on every bar exactly as it always has. On MNQ or any other CME future it makes no difference either way, because those symbols have no closed-hours bars to skip.

Two limits. The gate works off the clock, so CME holidays and early closes aren't detected, about 11 days a year. And the perp isn't NQ: even with the hours matched, its prices form some swings MNQ wouldn't.

The Pine compiler will warn that the engine should be called on each calculation for consistency. That's expected. Skipping the closed bars is the whole point.

## Reading the chart

| Mark | Meaning |
|---|---|
| `SFP` | Sweep and reclaim of a standing level |
| `ST MSS` | Short-term structure broken after the SFP |
| `IT MSS · BUY` / `IT MSS · SELL` | Program flipped |
| `SFP ×`, `ST MSS ×` | The setup reached ST MSS and then failed |

Setups that are still developing are drawn faded and go to full colour when they confirm. A setup is invalidated if price closes back beyond the raid wick before IT MSS, or if IT MSS doesn't come within 12 hours. An SFP that fails before reaching ST MSS is deleted. One that fails after ST MSS stays on the chart marked with `×`, because an ST MSS may already have been traded. A timeout and a stop-out get the same mark.

Leave **Remove invalidated setups** off when you review performance. With it on, failures disappear and the chart only shows what worked.

## Settings

**Structure / MMXM**

| Setting | Default | Notes |
|---|---|---|
| CME hours only | Off | For 24/7 symbols traded on CME session timing. See above |
| SFP swing strength (bars) | 3 | Bars to the left a swing must dominate |
| Equal-level tolerance (ticks) | 4 | Levels this close are merged into one |
| Liquidity levels tracked per side | 100 | When full, the oldest level is dropped. Don't go much below 40 |
| Show MMXM state | Off | Badge showing BUY, SELL or UNKNOWN |
| MMXM state position | Top Center | Six positions available |
| Show active liquidity extremes | Off | Outermost standing high and low. Diagnostic only |

**Detection**

| Setting | Default | Notes |
|---|---|---|
| Max raid time (minutes) | 90 | Holding beyond the level longer than this is acceptance |
| Min sweep depth (ATR) | 0 | Off by default |
| Min level age (minutes) | 10 | How long a level must stand before it can be swept |
| Next-candle confirmation | On | The reclaim must come after the raid candle |
| Min reclaim body (ATR) | 0.2 | Reclaim candle body, in the signal direction |
| Max MMXM transition time (minutes) | 720 | Time allowed from SFP to IT MSS |
| Min program age before reversal (minutes) | 960 | 16 hours |
| Bullish SFP / Bearish SFP | On | |

**Style**

| Setting | Default | Notes |
|---|---|---|
| SFP liquidity line, Sweep wick, SFP label | On | |
| Short-Term MSS, Intermediate-Term MSS | On | |
| Sweep zone box | Off | Shades from the swept level to the raid extreme |
| Sweep zone minimum width (15m bars) | 8 | Only matters with the box on. Sets the width of a developing box. A confirmed box already runs out to its IT MSS, so on most setups this changes nothing |
| Fade developing setups | On | |
| Remove invalidated setups | Off | |
| Historical setups | 50 | |

## Alerts

- Bullish 15m SFP
- Bearish 15m SFP
- Bullish 15m Short-Term MSS
- Bearish 15m Short-Term MSS
- Bullish 15m Intermediate-Term MSS / BUY Program
- Bearish 15m Intermediate-Term MSS / SELL Program

Set them to once per bar close. On a lower chart they fire from closed 15m data only. With CME hours on, none of them fire during closed hours. When you update the script, delete and recreate your alerts, because each alert stays bound to the version it was created on.

## What it doesn't use

No session windows, no previous day or week highs and lows, no premium/discount filter, no volume. Levels come from price structure alone. CME hours only isn't a session filter in that sense. It doesn't decide where liquidity comes from. It only removes bars a CME contract never prints.

## Notes on the defaults

I set the defaults by checking the engine bar by bar against my own hand-marked MNQ charts: the tops and bottoms I'd take, and the in-trend signals I wouldn't. The 16-hour maturity gate gave the same result anywhere between 13 and 21 hours, which is why it sits at 16. That range comes from a limited stretch of MNQ data and I haven't recalibrated it for the perp, so treat it as a starting point.

An SFP on its own isn't a trade. The MMXM state tells you which side of the market the indicator thinks is being delivered. Where it sits in the bigger picture is still your call.

## License

No open-source license is included with this repository.

## Disclaimer

This is a market structure tool. It doesn't predict price and it doesn't guarantee any trade. Execution and risk are yours.
