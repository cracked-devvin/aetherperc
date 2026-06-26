# AetherPerc

A film-noir-styled perpetual-futures venue on Solana, built on Anatoly
Yakovenko's open-source **Percolator** risk engine. Oracle-priced, coin-margined,
with no central order book to hunt and no public mempool queue to sandwich.

> **Preview status — read this first.** This repository is the public **frontend**.
> It runs as a **simulated paper-trading terminal**: prices are live (pulled from
> public oracles/DEX APIs), but **trades are not real** and no funds move on-chain.
> Real-money mainnet is gated behind an external audit. **$AETH is a utility token
> — not an investment or a security.** Nothing here is financial advice.

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
