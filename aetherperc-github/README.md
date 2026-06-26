# AetherPerc

A film-noir-styled perpetual-futures venue on Solana, built on Anatoly
Yakovenko's open-source **Percolator** risk engine. Oracle-priced, coin-margined,
with no central order book to hunt and no public mempool queue to sandwich.

> **Preview status — read this first.** This repository is the public **frontend**.
> It runs as a **simulated paper-trading terminal**: prices are live (pulled from
> public oracles/DEX APIs), but **trades are not real** and no funds move on-chain.
> Real-money mainnet is gated behind an external audit. **$AETH is a utility token
> — not an investment or a security.** Nothing here is financial advice.


## ⛓️ Live on Devnet

The AetherPerc engine — our fork of Anatoly Yakovenko's open-source
[Percolator](https://github.com/aeyakovenko/percolator) risk engine — is
**compiled from source and deployed to Solana devnet.**

| | |
|---|---|
| **Program ID** | `5uGpcojKZ9rFsLHJkfWpSSguvtqV7UigtPQVnn2mXb2F` |
| **Network** | Solana devnet |
| **Status** | Deployed, upgradeable — market init in progress |
| **Explorer** | https://explorer.solana.com/address/5uGpcojKZ9rFsLHJkfWpSSguvtqV7UigtPQVnn2mXb2F?cluster=devnet |

### Verify it yourself

Don't trust — verify. Reproduce the build and confirm the live program:

```bash
git clone https://github.com/aeyakovenko/percolator-prog.git
cd percolator-prog
RUST_MIN_STACK=8388608 cargo build-sbf          # -> target/deploy/percolator_prog.so (~1 MB)
solana program show 5uGpcojKZ9rFsLHJkfWpSSguvtqV7UigtPQVnn2mXb2F --url devnet
```

> SBF builds aren't byte-deterministic across toolchains, so the authoritative
> artifact is the on-chain program. `solana program show` proves it's live and
> upgradeable.

### Want to help build?

We're a fork in active development and would love contributors — market
initialization, SDK, indexer, keeper, and the memecoin / DEX-pool oracle markets.
Open an issue or PR. Everything is Apache-2.0. Devnet/unaudited; real funds gated behind audit.


## What's in here

| File | Path when deployed | Purpose |
|------|--------------------|---------|
| `index.html` | `/` | Landing page |
| `trade.html` | `/trade` | Trading terminal (simulated) |
| `whitepaper.html` | `/whitepaper` | Whitepaper |
| `tokenomics.html` | `/tokenomics` | $AETH buyback & burn model |
| `vercel.json`, `netlify.toml` | — | One-drag deploy + clean URLs |

Every page is fully self-contained (logo and favicon are inlined as base64),
so there are no build steps and no external asset dependencies.

## Run locally

Just open `index.html` in a browser — that's it. No server, no install.

(If a browser blocks the live price/chart requests over `file://`, serve the
folder instead: `npx serve .` or `python3 -m http.server`, then visit the
printed URL.)

## Deploy

- **Netlify:** drag this folder onto https://app.netlify.com/drop
- **Vercel:** `npm i -g vercel` then `vercel --prod`, or import at https://vercel.com/new
- **GitHub Pages:** enable Pages on the repo root (links keep their `.html` suffix there)

## Data sources

Keyless public APIs only — no keys, no secrets in this repo:
Pyth (oracle majors), Jupiter (Solana token prices), Coinbase Exchange and
GeckoTerminal (candles). Charts embed TradingView (majors) and GeckoTerminal (DEX pools).

## Built on Percolator

The risk engine this venue is designed around is **Percolator** by
Anatoly Yakovenko, licensed under Apache-2.0:

- https://github.com/aeyakovenko/percolator-prog (Solana program)
- https://github.com/aeyakovenko/percolator (engine library)

The Kani formal-verification proofs referenced in the whitepaper cover the
**upstream engine**, not this fork. See `NOTICE`.

## License

Licensed under the **Apache License 2.0** — see [`LICENSE`](./LICENSE) and
[`NOTICE`](./NOTICE). Apache-2.0 keeps this repo compatible with the upstream
Percolator engine and includes an explicit patent grant. (If you'd rather use
MIT for the frontend, you can — just swap the `LICENSE` file and this section.)

## Contributing

PRs welcome. Please don't commit secrets (`.env`, keys, seeds — see `.gitignore`),
and keep the preview/simulated status accurate in any user-facing copy.
