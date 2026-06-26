# AetherPerc — Devnet Runbook (Route A, beginner edition)

Goal: get a **real on-chain trade and a real liquidation on Solana devnet**, using the
already-deployed Percolator program and its open-source CLI. No Rust, no program
deploys — just Node/TypeScript and the Solana command-line tools.

**Read this first**
- This is **devnet** = a free test network. The "SOL" here is fake test SOL with no value. You cannot lose real money on devnet.
- The Percolator program is **experimental and unaudited**. That's fine for devnet testing; it is *not* fine for real funds. Real money waits for an external audit (your roadmap already says this).
- Take it one step at a time. If a step errors, stop and check the **Troubleshooting** section at the bottom before moving on.
- Commands are shown in code blocks. Copy one block at a time, paste into Terminal, press Enter.

---

## Phase 0 — Open Terminal and install the tools

Open **Terminal** (press `Cmd+Space`, type "Terminal", Enter).

**0.1 — Install Homebrew** (the Mac package manager). Skip if you already have it (`brew --version` prints a version).
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
When it finishes it may print 2 lines starting with `echo` / `eval` to "add Homebrew to your PATH." Copy-paste and run those lines, then continue.

**0.2 — Install Node, pnpm, and git**
```
brew install node pnpm git
```
Verify:
```
node --version
pnpm --version
git --version
```
You want Node v18 or higher. If `git` says it needs command-line tools, run `xcode-select --install`, click through the popup, then re-run.

**0.3 — Install the Solana CLI**
```
sh -c "$(curl -sSfL https://release.anza.xyz/stable/install)"
```
This is the official Anza/Agave installer. When it finishes it will likely print a line starting with `export PATH=`. **Copy that exact line, paste it, press Enter.** Then close and reopen Terminal so the change sticks.

Verify:
```
solana --version
```
If you get "command not found," see Troubleshooting → PATH.

---

## Phase 1 — Make a devnet wallet and get test SOL

**1.1 — Point the CLI at devnet**
```
solana config set --url devnet
```

**1.2 — Create a wallet keypair** (this is a *test* wallet — never put real money in it)
```
solana-keygen new
```
Press Enter to accept the default location (`~/.config/solana/id.json`). You can leave the passphrase empty for a test wallet (just press Enter). It will show a "pubkey" — that's your wallet address.

See your address any time:
```
solana address
```

**1.3 — Get free test SOL**
```
solana airdrop 2
solana balance
```
You should see ~2 SOL. If the airdrop fails or rate-limits, see Troubleshooting → Airdrop.

---

## Phase 2 — Get the Percolator CLI

**2.1 — Pick a folder and clone the CLI**
```
cd ~
mkdir percolator
cd percolator
git clone https://github.com/aeyakovenko/percolator-cli.git
cd percolator-cli
```

**2.2 — Install and build it**
```
pnpm install
pnpm build
```
This downloads the CLI's dependencies and compiles it. It may take a couple of minutes the first time.

**2.3 — Create the CLI config file**
```
cat > ~/.config/percolator-cli.json << 'EOF'
{
  "rpcUrl": "https://api.devnet.solana.com",
  "programId": "2SSnp35m7FQ7cRLNKGdW5UzjYFF6RBUNq7d3m5mqNByp",
  "walletPath": "~/.config/solana/id.json"
}
EOF
```
This tells the CLI: talk to **devnet**, use the **deployed Percolator program**, and sign with the **wallet you just made**.

> Tip: the public devnet RPC gets busy and may throw rate-limit errors. A free Helius devnet RPC key is much more reliable — sign up at helius.dev, then replace the `rpcUrl` above with your `https://devnet.helius-rpc.com/?api-key=YOUR_KEY` URL.

---

## Phase 3 — Sanity check: talk to the live program

Before trading, just *read* state. This proves your machine is connected.

The CLI's scripts are run with `npx tsx`. List a known devnet market (the public SOL/USD slab):
```
npx tsx scripts/dump-market.ts
```
or read raw slab state for a given market:
```
npx tsx node_modules/.bin/... # if a script needs a slab arg, it will tell you
```
If you see JSON describing a market (config, accounts, parameters) — connection works. If a script asks for a `--slab` argument, you'll supply that in the next phase with your own market.

> Every CLI command supports `--help`. When unsure of the exact flags, run e.g. `npx tsx scripts/setup-devnet-market.ts --help` or `node dist/index.js init-user --help`.

---

## Phase 4 — Create YOUR OWN devnet market

Why your own: on a market you create, **you are the admin** — you fund the LP (so your trades actually fill against liquidity) and you control the test oracle (so you can trigger a liquidation on purpose). This is the cleanest way to see the full loop.

This does **not** deploy a program — it just creates a new market (a "slab") on the already-deployed program.

**4.1 — Open the setup script and read its top section first**
```
open -e scripts/setup-devnet-market.ts
```
Near the top you'll see config (oracle, collateral mint, insurance/LP amounts). For a first run, the defaults (SOL/USD via Chainlink, wrapped-SOL collateral) are fine. Just confirm it points at devnet.

**4.2 — Run it**
```
npx tsx scripts/setup-devnet-market.ts
```
This creates the market, seeds an insurance fund, and adds a funded LP. **When it finishes, it prints a Slab address** (and usually a Matcher address). **Copy the Slab address** — you'll paste it into the commands below as `<SLAB>`. Save it somewhere; it's your market.

---

## Phase 5 — Start the crank (keep this running)

Percolator markets need a "keeper crank" running continuously, or trades get rejected and the market auto-pauses. Open a **second Terminal window** (`Cmd+N`), go back to the folder, and start the crank bot — then leave it running while you trade:
```
cd ~/percolator/percolator-cli
npx tsx scripts/crank-bot.ts
```
You'll see it tick every few seconds. **Leave this window open.** Do the next steps in your first window.

---

## Phase 6 — Make a real trade

In your **first** Terminal window (replace `<SLAB>` with your market's slab address):

**6.1 — Create your trading account in the market**
```
node dist/index.js init-user --slab <SLAB>
```

**6.2 — Find your account index** (the slot number you were assigned)
```
npx tsx scripts/find-user.ts <SLAB> $(solana address)
```
Note the index number it prints — call it `<IDX>`.

**6.3 — Deposit collateral** (amount is in lamports; 1 SOL = 1,000,000,000 lamports. Start with 0.2 SOL = 200000000)
```
node dist/index.js deposit --slab <SLAB> --user-idx <IDX> --amount 200000000
```
> If deposit complains about a token account: this market uses **wrapped SOL (wSOL)** as collateral. Run `node dist/index.js deposit --help` to see the wrap flag, or the setup script may have created the wSOL account for you already.

**6.4 — Place a trade.** You trade against your LP (the LP index from Phase 4 is usually `0`; `find-user.ts` lists all accounts so you can confirm). `--size` is signed: positive = long, negative = short.
```
node dist/index.js trade-nocpi --slab <SLAB> --user-idx <IDX> --lp-idx 0 --size 1000000 --oracle <ORACLE>
```
(`<ORACLE>` is the oracle address the setup script printed.) If it confirms, **you just made a real on-chain perpetual trade on devnet.**

**6.5 — Look at your position**
```
npx tsx scripts/dump-state.ts
```
You'll see your account's size, entry, margin, and PnL — read straight from the chain.

---

## Phase 7 — Trigger a real liquidation (optional but satisfying)

Because you're the admin, you can push a fake oracle price to simulate a crash and watch your position get liquidated by the crank.

**7.1 — Make yourself the oracle authority**
```
node dist/index.js set-oracle-authority --slab <SLAB> --authority $(solana address)
```

**7.2 — Push a price far enough to put your position underwater** (price is in USD; pick a value that blows past your liquidation level — check it with `dump-state` first)
```
node dist/index.js push-oracle-price --slab <SLAB> --price 50
```

**7.3 — Watch it happen.** The crank bot in your second window processes liquidations automatically. Confirm:
```
npx tsx scripts/check-liquidation.ts
npx tsx scripts/dump-state.ts
```
Your account should show as liquidated. **That's a real on-chain liquidation.**

**7.4 — Reset the oracle** back to the real feed when done
```
node dist/index.js set-oracle-authority --slab <SLAB> --authority 11111111111111111111111111111111
```

---

## What you just proved

You ran the actual Percolator engine on-chain: created a market, funded liquidity, opened a position, and liquidated it — all with real Solana transactions on devnet. That is the "swap the simulated layer for real on-chain instructions" milestone from your roadmap, end to end.

## What's next (when you're ready)

1. **Wire the AetherPerc terminal to this.** Replace the simulated `placeOrder` / `closePosition` in `trade.html` with the CLI's instruction builders + your browser wallet (Phantom) signing, and read positions from the slab instead of the sim. This is real frontend work — tell me when you're here and I'll build the adapter (I'll read the CLI's instruction code to get the byte layout exactly right).
2. **Your meme markets.** The public market is SOL/USD only. Your long-tail token thesis needs one market per token, with the DEX-pool oracle for coins that aren't on Chainlink/Pyth. That's the fork work (Route B): `cargo build-sbf`, deploy your own program, create markets. Needs Rust and is the version that goes through the audit before mainnet.

---

## Troubleshooting

**"command not found: solana"** — the installer didn't add it to PATH. Run:
```
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"
```
To make it permanent, add that same line to the end of `~/.zshrc`:
```
echo 'export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.zshrc
```
Then close and reopen Terminal.

**Airdrop fails / "rate limited"** — the faucet is busy. Wait a minute and try `solana airdrop 1` (smaller amount). Or use a Helius devnet RPC (Phase 2.3 tip) and retry. You can also use the web faucet at faucet.solana.com with your `solana address`.

**RPC errors / 429 / timeouts** — the public devnet RPC is overloaded. Switch `rpcUrl` in `~/.config/percolator-cli.json` to a free Helius devnet key. Many commands also accept `SOLANA_RPC_URL=...` as an environment variable in front of the command.

**Trade rejected / "stale" / market paused** — the crank isn't running or fell behind. Make sure the `crank-bot.ts` window (Phase 5) is still open and ticking. Trades need a recent crank (within ~40 seconds).

**"insufficient funds"** — you need more test SOL (`solana airdrop 1`), or you deposited more than you have. Remember amounts are in lamports (1 SOL = 1,000,000,000).

**A script wants an argument you don't have** — run it with `--help`, or open the file (`open -e scripts/<name>.ts`) and read the top; the addresses it needs (slab, oracle) are what the setup script printed in Phase 4.

**Totally stuck on a step** — copy the exact command you ran and the exact error text, and bring both back to me. Don't guess-and-retry destructive commands.

---

*Reference repos (all Apache-2.0): github.com/aeyakovenko/percolator-cli (the toolkit you're using), percolator-prog (the program), percolator (the engine). Unaudited, devnet/test use only.*
