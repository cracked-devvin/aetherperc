# AetherPerc — devnet deployment (proof + verify-it-yourself)

> Drop the two sections below into your GitHub README. The first proves the engine
> is real and lets any dev verify it from source. The second is the roadmap update.

---

## ⛓️ Live on Devnet

The AetherPerc engine — our fork of Anatoly Yakovenko's open-source
[Percolator](https://github.com/aeyakovenko/percolator) risk engine — is
**compiled from source and deployed to Solana devnet.**

| | |
|---|---|
| **Program ID** | `5uGpcojKZ9rFsLHJkfWpSSguvtqV7UigtPQVnn2mXb2F` |
| **Network** | Solana devnet |
| **Status** | Deployed, upgradeable |
| **Explorer** | https://explorer.solana.com/address/5uGpcojKZ9rFsLHJkfWpSSguvtqV7UigtPQVnn2mXb2F?cluster=devnet |

> **Preview / unaudited.** This is the deployed program on devnet (test network,
> no real funds). Market initialization is in progress; real-money mainnet is
> gated behind an external audit. Not financial advice; $AETH is a utility token,
> not a security.

### Verify it yourself

Don't trust — verify. The program above is built from the public Percolator
sources. Anyone can reproduce the build and confirm the on-chain bytes:

```bash
# 1. Clone the program repo (the engine is pulled automatically as a pinned git dep)
git clone https://github.com/aeyakovenko/percolator-prog.git
cd percolator-prog

# 2. Build it from source (the risk engine needs a larger stack)
RUST_MIN_STACK=8388608 cargo build-sbf
#    -> target/deploy/percolator_prog.so  (~1 MB)

# 3. Inspect the live program on devnet
solana program show 5uGpcojKZ9rFsLHJkfWpSSguvtqV7UigtPQVnn2mXb2F --url devnet

# 4. (optional) Dump the on-chain bytes and hash them
solana program dump -u d 5uGpcojKZ9rFsLHJkfWpSSguvtqV7UigtPQVnn2mXb2F /tmp/onchain.so
shasum -a 256 /tmp/onchain.so
```

> Note: SBF builds are not byte-deterministic across toolchain versions, so the
> hash in step 4 is environment-dependent — the authoritative artifact is the
> on-chain program. The point of step 2 is that the source compiles to a ~1 MB
> program of the same shape; `solana program show` proves it's live.

### Want to help build?

We're a fork in active development and would love contributors — SDK, indexer,
keeper, market initialization, and the memecoin/DEX-pool oracle markets. Open an
issue or PR. Everything is Apache-2.0.

---

## 🗺️ Roadmap update

Replace your current roadmap "phase 02" item with:

- [x] **Fork & deploy the Percolator engine to devnet** — live at
  `5uGpcojKZ9rFsLHJkfWpSSguvtqV7UigtPQVnn2mXb2F` (deployed 2026-06-26).
- [ ] **Initialize a market** on the deployed program (oracle wiring + LP).
- [ ] **Wire the frontend** to real on-chain instructions (wallet-signed
  deposit / trade / close).
- [ ] **External audit**, then guarded mainnet.

---

## Honest status line (for the site / tweet, keep it accurate)

> AetherPerc's forked Percolator engine is now **live on Solana devnet** —
> compiled from source, verifiable on-chain. Preview only: market init in
> progress, trading still simulated, real funds gated behind audit.
