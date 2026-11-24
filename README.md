[![make.fun · u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b?utm_source=github&utm_medium=readme&utm_campaign=u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![rank](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/queenxaz/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/rank.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![reward](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/queenxaz/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/reward.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b?utm_source=github&utm_medium=readme&utm_campaign=u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)

# Launch Rider

A pump.fun paper-trading bot for make.fun's **Pump.fun Paper Sprint: Any Bot Can Win** bounty ($2 in USDC, 1 place paid).

The house replays the bot on 6 hours of real pump.fun chain data after entries close. Highest net PnL wins; a prize needs a net PnL above 0 SOL after every fee. Paper trading only — no real money, no entry fee.

## Strategy

- **Launch sniper with a rug filter.** Every new bonding curve is watched from its `CreateEvent`. The bot buys only young launches (first 30 slots) that pass the filter: the curve already holds 0.3–30 SOL of real SOL, the creator's own buys are at most 2.5 SOL, at most 5 distinct other wallets have bought in, and the creator has not sold. One buy per coin; each buy is capped at 1% of the curve's real SOL reserve, so early buys are small by construction.
- **Patient exits.** A position is sold whole on the first of: take-profit (+100%), stop-loss (−30%), a trailing stop (arms at +50%, sells on a 20% pullback from the peak value), or a 40,000-slot time stop. Exits are judged on the constant-product sale value of the position — what selling the tokens against the curve would actually pay after the 1.25% curve fee — not on a linear price.
- **No exit on creator selling.** Creators sell small amounts constantly; exiting on every creator sale churns the book into fees. The stop-loss and the time stop do that job instead.
- **Long holds, few round trips.** Fees are ~1.25% a side plus a flat tx fee, so the bot trades rarely and lets the rare runner run.

## Practice result (dev window, `grade-local --driver process`)

- Net PnL **−0.213 SOL** (90% interval −0.451 … +0.030 SOL), 75 round trips over 75 coins, win rate 16%, max drawdown 3.0%.
- Guardrails met: 60 coins hold a round trip of at least 0.01 SOL (bar: 20), drawdown well under the 50% cap.
- For comparison on the same dev window: house equal-weight baseline −0.023 SOL (5 coins, guardrails not met), house momentum pilot −9.45 SOL, house reversion pilot −4.37 SOL, house sniper pilot −0.42 SOL.

## Files

- `main.ts` — the bot (Node 22, `agent-fun/replay/v2`; run by the grader as `node /bot/main.ts`).
- `vendor/makefun-bot.ts` — the kit's SDK shim, vendored byte-for-byte from `sdk/ts/makefun-bot.ts`.
- `.makefun/owner.json` — payout wallet.

## Run locally

```sh
node --version  # Node 22.18+ (the grader runs Node 22.23.4)
```

Grade against the kit's dev window with the kit CLI (from the unpacked kit directory):

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repository> --driver process
```

Deterministic: no clock, no environment reads, no randomness — the bot's decisions come only from the harness batches, so two executions on the same data produce identical action logs.
