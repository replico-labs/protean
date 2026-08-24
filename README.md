# Protean

A governance-agnostic DAO framework, made native to Telegram. Deploy a DAO,
get a wallet, stake, propose, and vote — all from inside a group chat, on any EVM Chain.

This is the umbrella repository. It holds no code of its own — just three
git submodules, each an independently deployable project, tied together
here for convenience.

## The three pieces

| Submodule | What it is | Status |
|---|---|---|
| [`contracts`](https://github.com/MarvinSunday/Spaces) | The on-chain governance protocol: token, staking wrapper, treasury, governance, factory, welcome distributor | ✅ Fully built, tested, currently deployed to Monad Testnet |
| [`bot`](https://github.com/MarvinSunday/protean-bot) | The Telegram bot — every user-facing command, wallet generation, and on-chain interaction | ✅ Live, write-capable (stake/propose/vote), custodial wallet model |
| [`connect`](https://github.com/MarvinSunday/protean-connect) | A wallet-linking website for a non-custodial alternative via Privy | ⏸️ Paused mid-debugging, not currently used by the bot |

Each has its own README with full detail — this file is just the map.

## How it fits together

```
Telegram Group
      │
      ▼
  bot/  ───────────────────────────────┐
  - all commands (/dao, /stake,        │
    /propose, /vote, /createdao, etc)   │
  - generates a wallet for every        │
    Telegram user on the spot           │
  - signs and submits transactions       │
      │                                  │
      ▼                                  ▼
  contracts/                        connect/
  - Governance, Treasury,           - Telegram OAuth via Privy
    GovernanceToken,                - would create a genuinely
    StakedGovernanceToken,            non-custodial wallet instead
    DAOFactory,                     - not currently wired in;
    WelcomeDistributor               bot uses its own wallet
                                      generation instead (see below)
```

## Why this exists

Most EVM chains handle DAO governance one of two ways: **Snapshot**
(off-chain — votes aren't binding, someone still has to manually execute
what was decided), or a **custom-built governance system** that's hard to
get right and, once deployed, is usually locked in — changing the rules
later means migrating the entire treasury.

Protean's contracts solve both: everything a DAO votes on executes
automatically, fully on-chain, and governance can evolve over time
(the treasury and the governance rules are deliberately kept as separate,
swappable pieces) without ever moving the treasury.

On top of that, most DAO tooling assumes you're already deep in Web3 — a
separate dashboard, a wallet extension, gas in hand before you can do
anything. Protean puts all of that inside the chat people are already in.

## Current wallet model — read this before assuming how it works

The bot currently uses **custodial, deterministically-derived wallets**
(`bot/src/wallet.js`) — every Telegram user's wallet is computed on the
spot from their Telegram ID and one server-side secret. No third-party
wallet provider, no OAuth step, works instantly. The tradeoff: the bot's
backend can, in principle, regenerate any user's private key.

The `connect/` submodule is the intended path to a genuinely
**non-custodial** alternative (Privy-based embedded wallets), but it's
currently blocked mid-debugging and not wired into the bot. See
`connect/README.md` for exactly where that stands and what's needed to
finish it.

## Getting started

```bash
git clone --recurse-submodules https://github.com/replico-labs/protean.git
cd protean
```

If you already cloned without `--recurse-submodules`:
```bash
git submodule update --init --recursive
```

Each submodule has its own setup instructions — see:
- `contracts/README.md` — Foundry setup, testing, deployment
- `bot/README.md` — env vars, running locally, deploying to Railway
- `connect/README.md` — Privy setup, current debugging status

## Keeping submodules in sync

This repo only tracks *pointers* to specific commits in each of the three
real repos. After pushing changes to any of them individually, update the
umbrella's pointers:

```bash
git submodule update --remote --merge
git add contracts bot connect
git commit -m "Sync submodules"
git push origin main
```

Deployment platforms (Railway, Vercel) are configured against each
individual repo directly, not this umbrella — see each submodule's own
README for deployment instructions.

## Try it live

[**@proteandao_bot**](https://t.me/proteandao_bot) on Telegram — assuming
the bot backend is currently deployed and running (see `bot/README.md`).

## License

MIT