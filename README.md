# PoPo.meme

> **AI-native IP incubation & trading platform.**
> Anyone can launch a meme/IP coin on a bonding curve; when it graduates the
> liquidity migrates into a Uniswap-v4-style concentrated-liquidity pool.
> Creators monetize continuously through tiered swap fees, paid video episodes
> with an automatic buyback-and-burn, and on-chain referral rewards.
>
> 🌐 Product brand: **PoPo.meme**

**📂 How to read:** every section is collapsed by default — only the title and a one-line summary are shown. Click a title to expand its full contents.
Judges can jump straight to: **Smart Contract Deployments (top)**, Technical highlights (§5), Feature checklist (§6), Monad bounty artifacts (§7), Demo flow (§8).

---

<details open>
<summary><strong>🔗 Smart Contract Deployment (Testnets)</strong> — live PoPoApp Diamond + $P (USDPApp) addresses on every deployed testnet, each linked to its chain explorer</summary>

All product trading — the bonding-curve **Hatch** launcher, the **ERC-6909**
multi-asset vault, YES/NO **prediction markets**, and the graduation /
concentrated-liquidity **AMM router** facets — runs on **one EIP-2535 Diamond
per chain** (`PoPoApp`). `$P` is the ERC-20 testnet token (`USDPApp`),
registered as quote token **`id = 1`** in the vault and minted by the faucet.
Click any address to open it on that chain's explorer.

| Chain | Chain ID | PoPoApp Diamond (all facets) | $P token (USDPApp) |
|---|---|---|---|
| Robinhood Chain Testnet | 46630 | [`0xCA1E…53C2`](https://explorer.testnet.chain.robinhood.com/address/0xCA1E405289335507782ad322d67916e7517953C2) | [`0x7423…C2AE`](https://explorer.testnet.chain.robinhood.com/address/0x74231829420B9A65BD83F6efD10FD266DB84C2AE) |
| Arbitrum Sepolia | 421614 | [`0xa57b…10F7`](https://sepolia.arbiscan.io/address/0xa57ba665456d23aAC98Ebf688d2Bf32fd58810F7) | [`0x0e87…8A99`](https://sepolia.arbiscan.io/address/0x0e872d6867323921489a23FF4572a7a749178A99) |
| Monad Testnet | 10143 | [`0xa08e…4c07`](https://testnet.monadexplorer.com/address/0xa08e66FD7f19C22Fe44928518DFe6A867B0E4c07) | [`0xf8D4…C662`](https://testnet.monadexplorer.com/address/0xf8D44b7682fe1f2D3d5089DCA3510D830675C662) |
| Arc Testnet | 5042002 | [`0x34B4…D627`](https://explorer.testnet.arc.io/address/0x34B4E1647ca6A049ED5876c5c71A8874b97bD627) | [`0x4dB1…4aB3`](https://explorer.testnet.arc.io/address/0x4dB1Ac5457E26db7530358378E576994d2574aB3) |
| Base Sepolia | 84532 | [`0xa57b…10F7`](https://sepolia.basescan.org/address/0xa57ba665456d23aAC98Ebf688d2Bf32fd58810F7) | [`0x0e87…8A99`](https://sepolia.basescan.org/address/0x0e872d6867323921489a23FF4572a7a749178A99) |
| HyperEVM Testnet | 998 | [`0x1CEB…d7Bd`](https://app.hyperliquid-testnet.xnode.l4n.xyz/address/0x1CEB0c21B999D55A89828713258f788EA649d7Bd) | [`0x586c…7913`](https://app.hyperliquid-testnet.xnode.l4n.xyz/address/0x586c9619d38d5B1E66FB235A3a887136990e7913) |
| Ethereum Sepolia | 11155111 | [`0x240B…f2CB`](https://sepolia.etherscan.io/address/0x240B7D86F6D3B89B6f6C5f5D34Cff4dE0255f2CB) | [`0x0067…3648`](https://sepolia.etherscan.io/address/0x00675E9aA5B05980ea74c35da555A4E0dC903648) |
| Tempo Moderato | 42431 | [`0x0067…3648`](https://explore.testnet.tempo.xyz/address/0x00675E9aA5B05980ea74c35da555A4E0dC903648) | [`0x35Db…f0C4`](https://explore.testnet.tempo.xyz/address/0x35Db7408b922CAd9f158cE0155008Af95aBaf0C4) |

**Notes**

- The **same unforked** EIP-2535 Diamond / ERC-6909 vault / v4-style PoolManager code ships unchanged on every chain above. On chains with a low per-tx gas ceiling (Arc, Base Sepolia, HyperEVM, Ethereum Sepolia, Tempo) the Diamond is assembled with a small constructor core plus additive `diamondCut` batches rather than one giant deploy.
- Arc Testnet additionally runs a separate `PPTSaleApp` sale/AMM diamond at [`0xBaaB…a089`](https://explorer.testnet.arc.io/address/0xBaaBb1197329D7d9db5d332e6E0308EB46c2a089).
- Arbitrum Sepolia shows the **same addresses as Base Sepolia** on purpose: one deployer EOA with an identical nonce sequence, and `CREATE` addresses are `keccak256(sender, nonce)`. They are independent contract instances on chainId `421614` and `84532` — do not cross-wire them.
- Addresses are the canonical single source of truth in `app/supabase/functions/_shared/chains-registry.ts` (frontend mirror `app/src/lib/contracts-registry.ts`). Localhost (`31337`) uses ephemeral addresses and is intentionally omitted.

</details>

<details>
<summary><strong>1. What this folder is</strong> — what the judge pack contains and the repo layout</summary>

A **judge-facing excerpt pack**: it explains the architecture and shows the
hard/interesting parts of the code as curated snippets. It is **not** the full
repository and deliberately omits:

- complete contract / application source trees,
- deployment scripts, deployed addresses, private keys and `.env` files,
- internal admin tooling and operational runbooks.

Each snippet is a short, self-contained excerpt of production code (comments
kept, surrounding boilerplate removed) plus a "why it matters" note.

```
.
├── README.md                     # this file — pitch + architecture + demo script
├── ARCHITECTURE.md               # deep dive of every on/off-chain component
├── TECHNICAL-HIGHLIGHTS.md       # the 7 hardest engineering problems, solved
├── DEMO-SCRIPT.md                # 5-minute live demo flow
├── snippets/                     # curated production-code excerpts
│   ├── contracts/                # Solidity excerpts (real code, trimmed)
│   └── app/                      # TypeScript / React excerpts
├── diagrams/architecture.md      # mermaid diagrams (render on GitHub/markdown)
├── bounty-submissions/           # Monad Metropolis bounty writeups (Privy/Envio/Alchemy/MetaMask)
├── envio-indexer/                # Envio HyperIndex v2 indexer for Monad testnet (real, runnable)
├── metamask-agent-plugin/        # `mm popo …` Agent Wallet plugin (8 commands, real on-chain calls)
└── docs/                         # Monad deployment facts + bounty execution plan
```

</details>

<details>
<summary><strong>2. The product in 30 seconds</strong> — problem / solution / audience / moat + creator-economy flywheel</summary>

| | |
|---|---|
| **Problem** | Creators have no way to launch a tradeable IP coin and earn *ongoing* value from it; launchpads dump liquidity into dead AMMs and give creators nothing after launch. |
| **Solution** | One contract system covers the full lifecycle: **bonding-curve launch → graduation → concentrated-liquidity market**, with creator fees, token-gated video episodes, daily algorithmic buyback-and-burn, and two-level referrals — all settled in one gas-efficient multi-token vault. |
| **Audience** | Meme/IP creators and their communities; prediction-market users. |
| **Moat** | Deep on-chain integration (v4-style custom PoolManager + hooks, not a fork wrapper), an off-chain content/social layer secured by EIP-712 signatures, and a creator-economy flywheel (episode revenue → buyback → burn → holders). |

### The creator-economy flywheel

```
fan buys episode access ($P)
        │
        ├─► creator revenue (claimable)
        ├─► platform share
        └─► buyback treasury
                 │  once per UTC day, permissionless trigger
                 ▼
        AMM market buy of the creator's own coin ──► BURN
                 │
                 ▼
      reduced supply / price support ──► holders & creators win
```

</details>

<details>
<summary><strong>3. Tech stack</strong> — Solidity/Diamond · React 19/viem/Privy · Supabase · multi-chain (incl. Monad testnet)</summary>

**Smart contracts** — Solidity `0.8.20`, Hardhat, EIP-2535 Diamond pattern.

**Frontend** — React 19, TypeScript, Vite, Tailwind v4, shadcn/ui, viem 2,
Privy embedded wallets, Three.js / Framer Motion.

**Off-chain** — Supabase Postgres + Deno Edge Functions; all user writes are
authorized by an EIP-712 signature the function verifies with `ecrecover`
(Supabase Auth disabled, service-role writes only).

**Media** — Supabase Storage + Service-Worker/WebRTC P2P CDN offload.

**Networks** — Multi-chain by design (URL-routed): the same unforked codebase is
deployed across X Layer mainnet, Sei mainnet/testnet, Mantle Sepolia,
Robinhood Chain testnet, **Arbitrum Sepolia (421614)**, Arc testnet and
**Monad Testnet (chainId 10143)**, plus a local Hardhat node. The read path and write path are explicitly de-synchronized to
survive wallet chain-switch lag.

> **Arbitrum Sepolia deployment (2026-10-16):** the full PoPoApp Diamond — all 19
> facets, 209 selectors, `0xa57ba665456d23aAC98Ebf688d2Bf32fd58810F7` — plus
> USDP/PPT/PFT, Create2Deployer, CreatorFeeHookDefault and EpisodeBuybackRouter
> are live on chainId `421614` and verified over public RPC (`facets()` = 19,
> 14,000-$P graduation threshold, quoteId = 1, ops wallet ROLE_4). Address table,
> verification commands and reproduce steps:
> [`docs/2026-10-16-arbitrum-sepolia-deployment.md`](./docs/2026-10-16-arbitrum-sepolia-deployment.md).

> **Monad Testnet deployment (2026-12-08):** the full PoPoApp Diamond — all 19
> facets, `0xa08e66FD7f19C22Fe44928518DFe6A867B0E4c07` — plus USDP/PPT/PFT,
> Create2Deployer, CreatorFeeHookDefault and EpisodeBuybackRouter are live and
> verified over public RPC (`facets()` = 19, 14,000-$P graduation threshold,
> quoteId = 1, ops wallet ROLE_4/90). Address table, verification commands and
> reproduce steps: [`docs/2026-12-08-monad-testnet-deployment.md`](./docs/2026-12-08-monad-testnet-deployment.md).

</details>

<details>
<summary><strong>4. Architecture at a glance</strong> — end-to-end diagram: browser / edge / Diamond</summary>

```
┌──────────────────────────── Browser (React 19 + viem 2) ───────────────────────────┐
│                                                                                     │
│  Privy embedded wallet  ◄── ALL business signatures & txs (external wallets         │
│        │                                  are funding-only)                          │
│        │ eth_signTypedData_v4 (PoPoAuth)                  sendTransaction            │
│        ▼                                              ▼                             │
│  Supabase Edge Functions (Deno)                Diamond Proxy (EIP-2535)             │
│  ─ ecrecover verify, replay window            ┌──────────┴───────────┐               │
│  ─ service-role Postgres writes               │ Facets (upgradable)  │               │
│        │                                      │  · HatchSwap curve   │               │
│        ▼                                      │  · PoolManager (v4)  │               │
│  Postgres: threads, likes, replies,           │  · Vault ERC-6909    │               │
│  trades index, holders, episodes meta,        │  · Migration         │               │
│  applications (admin review)                  │  · Referral          │               │
│                                               │  · Episodes+Burn     │               │
│  P2P media layer (SW + WebRTC)                │  · Prediction Market │               │
│  Supabase Storage ◄─ event-driven fallback ──►│  · AccessControl     │               │
│                                  getLogs      └──────────────────────┘               │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

Full component breakdown: [`ARCHITECTURE.md`](./ARCHITECTURE.md).
Diagrams: [`diagrams/architecture.md`](./diagrams/architecture.md).

</details>

<details>
<summary><strong>5. Technical highlights (judges: start here) ⭐</strong> — the 7 hardest engineering problems, solved</summary>

1. **Bonding-curve → CLM migration in one contract.** After the curve
   completes, the same Diamond initializes a v4-style pool, derives the starting
   sqrt-price from the curve reserves with 512-bit math, computes the maximal
   full-range liquidity, and locks it in a Diamond-owned position — no LP
   tokens, no external router. → `snippets/contracts/02-graduation-migration.md`

2. **Custom Uniswap-v4-style PoolManager + Hooks, from scratch.** Ticks,
   tick-bitmap, `SwapMath`, flash-accounting `unlock`/`settle`/`take`, and an
   address-flag-gated hook dispatcher — written against our ERC-6909 multi-token
   ledger, not forked. → `snippets/contracts/03-poolmanager-hooks.md`

3. **Creator fee hook with market-cap-tiered pricing.** A CREATE2-mined hook
   address (`0x44` permission bits) charges fees in the afterSwap delta; the fee
   tier is evaluated at the *pre-trade* price so a trade can't manipulate its
   own fee class. → `snippets/contracts/03-poolmanager-hooks.md`

4. **ERC-6909 single-vault settlement.** Every asset — $P quote, all launched
   coins — is an internal id on one ledger with strict 1:1 ERC20 backing,
   eliminating hop-by-hop token transfers across curve, pool, referrals and
   buybacks. → `snippets/contracts/04-vault-erc6909.md`

5. **Permissionless daily buyback-and-burn.** Episode revenue funds a per-coin
   treasury; anyone can trigger one swap per UTC day through a stateless router,
   with a sqrt-price-limit slippage guard; bought coins are burned on the spot.
   → `snippets/contracts/06-episodes-buyback.md`

6. **Passwordless authz for off-chain writes.** Supabase Auth is off: every
   write Edge Function reconstructs an EIP-712 digest and runs `ecrecover`,
   with a 10-min freshness window, canonical-JSON payloads and realm separation.
   → `snippets/app/01-eip712-wallet-auth.md`

7. **Trustless client in a multi-chain world.** On-chain `canCall(address,
   selector)` is the only admin source of truth (no client allowlist); a
   getLogs-based fallback indexer reconstructs trades/holders when the server
   can't reach the user's RPC (e.g. localhost). → `snippets/app/04-onchain-indexer.md`

More: [`TECHNICAL-HIGHLIGHTS.md`](./TECHNICAL-HIGHLIGHTS.md).

</details>

<details>
<summary><strong>6. Feature checklist</strong> — launch/trading · AMM · creator economy · growth · prediction · infra</summary>

**Launch & trade (PPHatchSwap)**
- [x] Permissionless coin creation with on-chain creation fee, metadata, 18-decimals
- [x] xy=k virtual-reserve bonding curve, platform + creator fees on buy/sell
- [x] Slippage protection (`minTokensOut` / `minQuoteOut`)
- [x] Creator vesting: cliff + linear daily unlock starting at graduation
- [x] Creator fee recipient transferability
- [x] Graduation threshold → one-tx migration into concentrated liquidity

**AMM (v4-style PoolManager)**
- [x] Full-range / tick-range liquidity, tick bitmap crossing, protocol fees
- [x] Flash accounting: `unlock` callback session + `settle` / `take`
- [x] Hook flags via low address bits (`beforeSwap` / `afterSwap` + delta-return)
- [x] Market-cap-tiered creator fees post-graduation

**Content & creator economy**
- [x] Token-gated / paid video episodes per coin (on-chain access mapping)
- [x] Revenue split: creator / platform / buyback treasury (dust-free)
- [x] Daily permissionless buyback-and-burn with slippage cap
- [x] P2P-assisted video delivery to cut storage egress

**Growth**
- [x] Two-level referrals, fees carved from the platform fee
- [x] Optional "gold" referrer tier, rate frozen at binding time
- [x] Cycle/self/duplicate binding guards; binding closes after first trade

**Prediction market (PPApp)**
- [x] ERC-1155 YES/NO shares, `price = k·√x + c` bonding curve
- [x] Admin-reviewed on-chain enablement, finalization & pro-rata settlement
- [x] Losing-side compensation token

**Infra**
- [x] EIP-2535 Diamond: upgradeable, role-gated per selector
- [x] Privy embedded wallet mandatory for business actions
- [x] EIP-712-secured Supabase write layer (service-role only)
- [x] Multi-chain routing + on-chain fallback indexer

</details>

<details>
<summary><strong>7. Monad Metropolis bounty artifacts</strong> — Envio indexer · MetaMask Agent plugin · bounty writeups · live on-chain market</summary>

Built specifically for the Monad hackathon (execution status:
[`bounty-submissions/README.md`](./bounty-submissions/README.md), full plan:
[`docs/2026-10-11-monad-metropolis-bounty-plan.md`](./docs/2026-10-11-monad-metropolis-bounty-plan.md)):

| Artifact | What it is |
| --- | --- |
| [`envio-indexer/`](./envio-indexer/) | **Envio HyperIndex v2** real-time indexer for PoPo.meme on Monad testnet: 8 events (Hatch `TokenCreated`/`Buy`/`Sell`/`BondingCurveComplete` + prediction `Buy`/`Sell`/`BuyBatch`/`SellBatch`) → 7 typed entities, GraphQL/REST API. Both facets share one Diamond address. Runnable (`npm i && envio dev`). |
| [`metamask-agent-plugin/`](./metamask-agent-plugin/) | **MetaMask Agent Wallet** plugin `mm-plugin-popo` with 8 `mm popo …` commands — quote/trade/markets/market/position/predict/vault/deposit — wired to the Monad Diamond with real bonding-curve math and typed contract calls. |
| [`bounty-submissions/`](./bounty-submissions/) | Submission writeups: Privy (P0, embedded-wallet-only signing), Envio, Alchemy Supernode/Enhanced APIs, MetaMask Agent Wallet. |

A live prediction market was also opened and smoke-tested on-chain during the
event (market digest `0x7fd5…3e52`, YES/NO 52/53).

</details>

<details>
<summary><strong>8. 5-minute demo</strong> — login → create coin → trade → graduate → paid episode/buyback → prediction market</summary>

See [`DEMO-SCRIPT.md`](./DEMO-SCRIPT.md). Short path:

1. **0:30** — Email login → Privy creates an embedded wallet; fund it.
2. **1:00** — Create a coin (name, art, reserve %) → instant bonding-curve market.
3. **2:00** — Buy/sell live; show price curve, fees accruing to creator, referral link.
4. **3:00** — Graduate → pool auto-seeded in the v4-style AMM; hook fee keeps paying creator.
5. **4:00** — Upload a paid episode → buy access as a fan → trigger daily buyback-and-burn.
6. **4:45** — Open the prediction market and show YES/NO share purchase & settlement.

</details>

<details>
<summary><strong>9. Security considerations</strong> — on-chain role gating · EIP-712 verification · callback auth · slippage · fund integrity · vesting</summary>

- **Diamond + per-selector roles**: privileged functions are gated on chain via
  `AccessControl.canCall(account, selector)`; the client never decides.
- **Signature layer**: EIP-712 with timestamp freshness (replay protection),
  canonical JSON, domain (realm) separation; service-role key never leaves the
  Edge Function.
- **Migration callback auth**: `unlockCallback` accepts only self-Diamond calls
  inside an open unlock session, preventing hook/external replay of seeded
  positions.
- **Slippage**: curve trades take explicit minima; buyback enforces a sqrt-price
  limit; hook fees are int128-clamped.
- **Fund integrity**: referral carve-outs and pending totals are deducted from
  admin-withdrawable platform fees, so fee money can never be double-spent.
- **Vesting**: creator reserve tokens are non-transferable through the curve
  until cliff/vest conditions after graduation.

</details>

<details>
<summary><strong>10. License note</strong> — excerpt usage terms and provenance of v3/v4 math libraries</summary>

Snippets are excerpts of proprietary PoPo.meme code provided for hackathon
review only. V4-style math libraries (`FullMath`, `SqrtPriceMath`, `SwapMath`,
`Tick*`, `TickMath`) follow established Uniswap v3/v4 public design.

</details>
