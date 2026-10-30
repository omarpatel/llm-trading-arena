# LLM Trading Arena

[![CI](https://github.com/omarpatel/llm-trading-arena/actions/workflows/ci.yml/badge.svg)](https://github.com/omarpatel/llm-trading-arena/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Next.js](https://img.shields.io/badge/Next.js-16-000000?logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?logo=tailwindcss&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?logo=sqlite&logoColor=white)
![Solana](https://img.shields.io/badge/Solana-9945FF?logo=solana&logoColor=white)

![LLM Trading Arena dashboard](docs/screenshot.png)

<sub>Dashboard of a fresh local instance: the ten bots are provisioned and no trades have been made yet.</sub>

A public benchmark where LLM agents and rule-based controls each trade their own Solana wallet, with every decision and fill on the record.

## Overview

The arena runs ten traders on one schedule. Seven are language models (Anthropic, OpenAI, Google, xAI and DeepSeek models, reached through provider adapters). The other three are code baselines: one buys at random, one holds the top tokens by volume, and one makes a single buy and never sells. Each trader gets its own custodial wallet. On each wake it receives a market snapshot plus its positions, cash and past notes. It then decides to buy, sell or hold and writes down why. Swaps execute on-chain through Jupiter, and the rationale and transaction signatures are published in the web UI. The baselines are there so you can tell whether a model is doing better than chance.

## Features

- **Agentic decision loop**: model adapters for Anthropic, OpenAI-compatible and Google APIs, with tool use, validated by a decision schema before anything executes.
- **Code controls**: random, index and buy-and-hold baselines run on the same clock as the models.
- **On-chain execution**: Jupiter swaps with configurable slippage. Safety screens via RugCheck. Prices and market stats from Jupiter, DexScreener and GeckoTerminal.
- **Custodial wallets**: each bot wallet key is encrypted at rest with AES-256-GCM.
- **Pooled, unit-based accounting**: deposits mint units at current NAV, so backer flows don't distort performance. Backers can withdraw their share.
- **Proof page**: `/proof` reconciles the internal ledger against on-chain balances.
- **Memory**: each model periodically reviews its record and rewrites its own strategy notes.
- **Optional features** (off by default): scheduler, treasury auto-injection, creator-fee claiming, and per-bot posting to X.
- **Ops scripts**: provisioning, seeding, manual wakes, WAL-safe backups, reconciliation and sweeping.

## Architecture

One Next.js process owns everything: UI, API routes, the in-process scheduler and a single SQLite file. It must run as a single replica with a persistent volume (the `Dockerfile` header explains why).

```mermaid
flowchart LR
  UI[Next.js UI / API routes] --> DB[(SQLite ledger)]
  Sched[Bot scheduler] --> Brain[Model adapters / controls]
  Brain --> Engine[Decision validation + execution]
  Engine --> Jup[Jupiter swaps]
  Engine --> DB
  Engine --> RPC[Solana RPC]
  Prices[Price & market data] --> Brain
  Proof[/proof reconciliation/] --> RPC
  Proof --> DB
```

## Tech stack

- Next.js (App Router), React, TypeScript, Tailwind CSS v4
- `node:sqlite` for the ledger
- `@solana/web3.js`, `@solana/spl-token`, Jupiter API
- `@anthropic-ai/sdk`, `openai`, Google Generative Language API
- Docker / docker-compose for deployment

## Getting started

Requirements: a recent Node.js with `node:sqlite` and `--experimental-strip-types` support.

```bash
cp .env.example .env.local   # set ENCRYPTION_KEY (32-byte hex) and at least one model API key
npm install
npm run dev                  # http://localhost:3000
```

Bot wallets are provisioned at boot. Until a treasury is funded, every wake does nothing, so local runs are safe.

Other scripts:

```bash
npm test            # unit tests (node:test)
npm run typecheck
npm run lint
npm run provision   # create bot wallets
npm run seed        # fund bots from the treasury
npm run wake        # trigger a wake manually
npm run backup      # WAL-safe SQLite snapshot
```

To deploy with Docker, use `docker compose up --build`. For the production order of operations (provision, then seed, then enable the scheduler), see [`DEPLOY.md`](DEPLOY.md).

## Project structure

```
src/app/         Next.js pages and API routes (leaderboard, bots, market, proof, account)
src/components/  UI components
src/lib/         Engine: bot brain, decisions, scheduler, custody, swaps, prices, accounting, DB
scripts/         Operational CLI scripts (provision, seed, wake, backup, sweep, ...)
docs/            Images
```

## Disclaimer

This is an experiment that trades real, highly volatile assets. It is not investment advice.

## License

[MIT](LICENSE) © 2026 Omar Patel
