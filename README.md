# btc-race-24h-bot-20261118

[![make.fun · btc-race-24h-20261118-1444](https://make.fun/badge/btc-race-24h-20261118-1444.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![rank](https://make.fun/badge/btc-race-24h-20261118-1444/satriayusuf/btc-race-24h-bot-20261118/rank.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![reward](https://make.fun/badge/btc-race-24h-20261118-1444/satriayusuf/btc-race-24h-bot-20261118/reward.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)

A long-only cbBTC/USDC trading bot for make.fun's [BTC race](https://make.fun/bounty/btc-race-24h-20261118-1444): it trades a $1,000 paper account minute by minute against on-chain prices, in the `agent-fun/bars-replay/v1` replay. "Iron Rails II" — the improved version of the 2026-11-17 entry, which held one position through a downtrend with no stop-loss and finished −12.90 USDC.

## The idea
A full round trip in this market costs roughly 26 bps once the fill fee, the modelled slippage and the constant-product price impact are counted, while the standard deviation of a single minute is only about 6.7 bps. Any strategy that reacts to minute-level motion is therefore paying more in friction than the motion is worth — so this bot trades only one kind of move:

- **Fresh cross, confirmed regime.** It buys with 100% of its cash the moment EMA(8) crosses above EMA(60) — on the cross bar or the next one — while the price sits above a rising EMA(120). A cross that happened long ago is stale and never chased; a flat or falling regime never entered.
- **Exits are tight on purpose.** Every position carries a −0.8% stop-loss and a +1.5% take-profit from its entry close, and the position is also sold when the trend turns back down (EMA(8) under EMA(60), or two closes under EMA(60)).
- **Cooldown.** After any exit the bot waits 30 minutes before the next entry, so a sideways day costs one trade instead of dozens.
- **A guardrail trade.** If no signal has produced a filled order by bar 1250, the bot places one 5 USDC trade, so the run is never ranked below every traded entry for having no trades at all. It costs a fraction of a cent.

On the kit's frozen dev window (graded locally with `makefun grade-local`, the server's own replay) the bot makes **+5.36 USDC (+0.536%) net of all fees in a single round trip — buy at bar 443, trend exit at bar 543 — with a 0.66% maximum drawdown**, comfortably through the dev gate (net PnL > 0, drawdown far inside the 15% ceiling), while buy-and-hold loses ~2.8% on the same window. It stays profitable even with every fee doubled (+4.56 USDC).

## Layout

- `main.ts` — the strategy, and the only file that thinks. Written for Node 22's type stripping (erasable syntax only).
- `vendor/makefun-bars.ts` — the kit's protocol shim, copied byte for byte; it owns the wire (hello/batch/reply/done), the account state and order validation.
- `.makefun/owner.json` — the entrant wallet the entry is signed for.

The bot is deterministic: a pure function of the bars it is sent. It reads no clock, no environment and no files, and uses no randomness, so the replay's two executions produce identical action logs.
