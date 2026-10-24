# Arbitrum Sepolia (421614) deployment — 2026-10-16

The complete PoPo.meme stack — one EIP-2535 Diamond holding all 19 facets, the
ERC-6909 multi-asset vault, the v4-style PoolManager AMM, prediction markets and
paid-episode buybacks — is live on **Arbitrum Sepolia (chainId 421614)**, with
`$P` (`USDPApp`) registered as vault quote token `id = 1`.

- RPC: `https://sepolia-rollup.arbitrum.io/rpc`
- Explorer: <https://sepolia.arbiscan.io>

## 1. Addresses

| Contract | Address | Notes |
| --- | --- | --- |
| **PoPoApp Diamond** | [`0xa57ba665456d23aAC98Ebf688d2Bf32fd58810F7`](https://sepolia.arbiscan.io/address/0xa57ba665456d23aAC98Ebf688d2Bf32fd58810F7) | 19 facets / 209 selectors; CREATE at block `319627533` |
| **$P token (USDPApp)** | [`0x0e872d6867323921489a23FF4572a7a749178A99`](https://sepolia.arbiscan.io/address/0x0e872d6867323921489a23FF4572a7a749178A99) | `USDP` / `$P` / 18 decimals, quote id 1 |
| PPTApp (prediction platform token) | [`0x86aE1829C2a2b072fADDBEbEF6bBbB83461e338D`](https://sepolia.arbiscan.io/address/0x86aE1829C2a2b072fADDBEbEF6bBbB83461e338D) | `Pred Platform Token` / 18 |
| PFTApp (failed-share SBT) | [`0x586c9619d38d5B1E66FB235A3a887136990e7913`](https://sepolia.arbiscan.io/address/0x586c9619d38d5B1E66FB235A3a887136990e7913) | 0 decimals, whitelisted on PoPoApp |
| Create2Deployer | [`0x00675E9aA5B05980ea74c35da555A4E0dC903648`](https://sepolia.arbiscan.io/address/0x00675E9aA5B05980ea74c35da555A4E0dC903648) | used by the graduation-hook deploy step |
| CreatorFeeHookDefault | [`0x471e4575249A2711b61D46d78D4C891936F70044`](https://sepolia.arbiscan.io/address/0x471e4575249A2711b61D46d78D4C891936F70044) | flags `0x44`, `defaultGraduationHook` intentionally left `address(0)` |
| EpisodeBuybackRouter | [`0x97a422279d0B9F28f26625D2953908B48C965C51`](https://sepolia.arbiscan.io/address/0x97a422279d0B9F28f26625D2953908B48C965C51) | wired into `EpisodeFacet` |

> **Why these are the same addresses as Base Sepolia.** Both deployments used one
> deployer EOA with the same nonce sequence, and `CREATE` addresses are
> `keccak256(sender, nonce)`. The instances are independent — chainId `421614`
> here, `84532` on Base Sepolia. Never reuse an address across chains without
> checking the chainId.

## 2. On-chain verification (reproduce over public RPC)

```bash
RPC=https://sepolia-rollup.arbitrum.io/rpc
D=0xa57ba665456d23aAC98Ebf688d2Bf32fd58810F7

# 19 facets
cast call $D "facets()((address,bytes4[])[])" --rpc-url $RPC | wc -l

# $P metadata
cast call 0x0e872d6867323921489a23FF4572a7a749178A99 "name()(string)"   --rpc-url $RPC  # USDP
cast call 0x0e872d6867323921489a23FF4572a7a749178A99 "symbol()(string)" --rpc-url $RPC  # $P
cast call 0x0e872d6867323921489a23FF4572a7a749178A99 "decimals()(uint8)" --rpc-url $RPC # 18
```

Last verified result set:

| Check | Value |
| --- | --- |
| `facets()` | **19 facets**, **209 selectors** |
| `$P` | `USDP` / `$P` / `18` |
| `getConfig()` graduation threshold | `14,000e18` ($P) |
| `getConfig()` create-token fee | `2e18` ($P) |
| Curve platform fee / creator fee / AMM fee | `100` / `50` / `25` bps |
| Quote token id | `1` |
| Deployer roles | `ROLE_90` + `ROLE_4` |
| Ops wallet `0x85A6…D598` | `ROLE_4` |
| EIP-712 domain | `PoPo.meme`, version `1`, chainId `0x66eee` |

## 3. Reproduce the deployment

```bash
cd smart-contract

# 1) $P / PPT / PFT diamonds (quote-token prerequisites)
DEBUG='deploy:*' npx hardhat deploy \
  --tags deploy:PPAppTokens:00_DeployToken --network arbitrumSepolia

# 2) PoPoApp: 19 facets, then config steps 02..09
npm run deploy:PoPoApp -- --network arbitrumSepolia
```

Data files consumed by the deploy scripts:

- `deployments/arbitrumSepolia-data/config.json` — `marketName = PoPo.meme`, EIP-712 domain, token URI prefix
- `deployments/arbitrumSepolia-data/admin-role-b.json` — deployer + ops `0x85A6…D598`, both granted `ROLE_4`

Steps `02..09` end state:

- `02` bulk `ROLE_4` grant (deployer + ops)
- `03` USDP registered as quote `id = 1`
- `04` `setConfig`: 14,000 $P graduation threshold, 2 $P create fee, 100 bps curve platform fee, 50 bps creator fee, 25 bps AMM fee
- `05` CreatorFeeHook CREATE2 deploy; `defaultGraduationHook` stays `address(0)`
- `06` Market facet pointed at PFTApp + PoPoApp whitelisted on the PFT SBT
- `07` EpisodeBuybackRouter deployed and wired
- `08` funding setup — skipped (no payment-stablecoin mock on this chain)
- `09` `endMarket` restricted to `ROLE_5`, `ROLE_4` retained

## 4. What was exercised end to end

- token create on the curve, buy/sell, and **atomic graduation into the AMM**
- LP add/remove through the router facet and the PoolManager facet
- prediction markets created, traded and resolved inside the same vault
- paid episodes: unlock with `$P`, creator earnings, buyback-and-burn through `EpisodeBuybackRouter`
- same bytecode, no fork: the identical artifact set also runs on Robinhood
  Chain testnet, Base Sepolia, Monad, Arc, HyperEVM, Tempo and Ethereum Sepolia

## 5. Known deviations from mainnet intent

- `defaultGraduationHook` is left unset on testnet (`WIRED_DEFAULT_HOOK` not enabled), so graduation does not auto-deploy a hook.
- The ops wallet carries `ROLE_4` but not `ROLE_90`; the deployer EOA is the only super admin on this deployment.
- Testnet `$P` is minted by the faucet flow (`testnet-faucet-claim`), not sold.
