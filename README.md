<div align="center">

# Safrochain Mainnet — Draft Genesis

**The canonical draft `genesis.json` for `safrochain-1`**

[![Chain ID](https://img.shields.io/badge/chain--id-safrochain--1-1f6feb?style=for-the-badge)](#chain-parameters)
[![Denom](https://img.shields.io/badge/denom-usaf-8957e5?style=for-the-badge)](#chain-parameters)
[![Supply](https://img.shields.io/badge/supply-1%2C000%2C000%2C000_SAF-2da44e?style=for-the-badge)](#tokenomics)
[![Validators](https://img.shields.io/badge/genesis_validators-9-fb8500?style=for-the-badge)](#genesis-validators)
[![Status](https://img.shields.io/badge/status-draft-d29922?style=for-the-badge)](#status)

</div>

---

## Status

> **⚠️ DRAFT** — This file is published for community review of the initial state ahead of network launch. The `genesis_time`, validator set, and individual delegations may still change before the final launch genesis is cut. Treat the SHA below as a checkpoint, not the launch hash.

| | |
| --- | --- |
| **File** | [`genesis.json`](./genesis.json) |
| **Size** | 64,739 bytes (≈ 63 KB) |
| **SHA-256** | `f5b699aa9c6d76e18f4abf93c4a2833e09364e40789f62c6d3018a5f9fdf75a2` |
| **Genesis time** | `2026-05-30T12:00:00.000000Z` |
| **Initial height** | `1` |

---

## Chain Parameters

| Parameter | Value |
| :--- | :--- |
| **Chain ID** | `safrochain-1` |
| **Base denom** | `usaf` (1 SAF = 1 000 000 usaf) |
| **Bond denom** | `usaf` |
| **Consensus** | CometBFT, ed25519 validator keys |
| **Max validators** | 100 |
| **Min commission rate** | 5 % |
| **Unbonding period** | 14 days (`1209600s`) |
| **Voting period** | 7 days (`604800s`) — expedited 1 day |
| **Min gov deposit** | 5 000 SAF (`5 000 000 000 usaf`) |
| **Gov quorum** | 33.4 % |
| **Min gas price** | `0.05 usaf` (global fee) |
| **Inflation** | min 3 % · max 12 % · target bonded 55 % |
| **Slash — downtime** | 0.01 % · jail 10 min · window 10 000 blocks @ 50 % |
| **Slash — double-sign** | 5 % |
| **Evidence max age** | 14 days / 241 920 blocks |
| **Max block bytes** | 22 020 096 |
| **Max block gas** | 100 000 000 |

---

## Tokenomics

**Total supply: 1 000 000 000 SAF**

| Allocation | Amount (SAF) | % of supply | Notes |
| :--- | ---: | ---: | :--- |
| **DAO Reserve** | 210 000 000 | 21 % | Governance & community-driven decisions. ~135.76 M SAF delegated to the 8 manifested validators via post-launch `MsgDelegate` (see [Initial bonded stake](#initial-bonded-stake)). Includes a 10 M SAF Developer Reserves residual moved from the Team pool, held under DAO control until specific contributors are designated. |
| **Community** | 150 000 000 | 15 % | Airdrops, missions, community rewards |
| **Team** | 140 000 000 | 14 % | Cofounders (Dan Baruka, Gentil Samvura — 40 M each, 80 M total) hold `PeriodicVestingAccount`s (6 mo cliff + 24 mo monthly). All other 13 team members hold `ContinuousVestingAccount`s — linear release between `start_time` (= genesis + cliff) and `end_time`. Cliff/vest by group: Developers 3 mo + 18 mo, Management 6 mo + 18 mo, Advisors 3 mo + 12 mo. Gentil's Mgmt + Advisor allocations are merged into a single 10.5 M continuous account under the Management schedule. The 10 M Developer Reserves residual is parked on the DAO Reserve wallet. See [wallets-tokenomics.json](../wallets-tokenomics.json) for the per-member breakdown. |
| **Ecosystem Development** | 150 000 000 | 15 % | Ecosystem growth and developer programs |
| **Public Sale** | 100 000 000 | 10 % | Public sale participants |
| **Liquidity** | 100 000 000 | 10 % | DeFi liquidity pools & market making |
| **Treasury** | 100 000 000 | 10 % | Foundation treasury |
| **Partners** | 50 000 000 | 5 % | Strategic partnerships |
| **— Total** | **1 000 000 000** | **100 %** | |

### Initial bonded stake

| Source | Bonded |
| :--- | ---: |
| Community validator self-delegations (7 × 10 K) | 70 000 SAF |
| SF foundation validator self-delegations (2 × 50 K) | 100 000 SAF |
| **Total bonded at block 1** | **170 000 SAF** |
| DAO Reserve `MsgDelegate` txs (8 entries in [`dao-delegate-manifest.json`](./dao-delegate-manifest.json)), broadcast post-launch | 135 757 093 SAF |
| **Bonded once DAO delegations confirm (~block 5–10)** | **≈ 135.93 M SAF (~13.6 % of supply)** |

The DAO Reserve delegations are **applied as live transactions immediately after launch**, not baked into genesis state. The runbook is in [`bin/apply-dao-delegations-postgenesis.sh`](../bin/apply-dao-delegations-postgenesis.sh) — it imports the `dao-reserve` key, waits until the chain is producing blocks, and broadcasts the 8 `MsgDelegate` txs in sequence. (Earlier drafts attempted to embed these delegations directly in `app_state.staking.delegations`, but the chain's `InitGenesis` ordering runs `staking` before `genutil`, so delegations to validators that gen_txs would later create produce a `validator does not exist` panic.)

---

## Genesis Validators

The validator set at block 1 consists of the **2 Safrochain Foundation validators + 7 community validators** (all 9 gentxs baked into `app_state.genutil.gen_txs`).

| # | Moniker | Class | Self-stake | Commission | Max | Operator (valoper) |
| :-- | :--- | :--- | ---: | ---: | ---: | :--- |
| 1 | safro-validator-1 | Foundation | 50 000 SAF | 10 % | 20 % | `addr_safrovaloper1j9l9salsfwq83cf83zsg7wdhvc5p2f6xshlpzn` |
| 2 | safro-validator-2 | Foundation | 50 000 SAF | 10 % | 20 % | `addr_safrovaloper1h2jy3sghpwevgw3cjwqk055ejwxn5ls3ur7c5q` |
| 3 | **NodeStake** | Community | 10 000 SAF | 5 % | 20 % | `addr_safrovaloper1sdlfp8n5fcfa7qw7770ngqs02k876gf6m749ly` |
| 4 | **catsmile** | Community | 10 000 SAF | 10 % | 20 % | `addr_safrovaloper13njz6aqmtwtu7vl4w0c6j7dvt7qj6t77vg6r9s` |
| 5 | **Winnode** | Community | 10 000 SAF | 5 % | 20 % | `addr_safrovaloper1a6ve2escz8h4ws3ttelfp54av2wwvty6f4xq8z` |
| 6 | **Vinjan.Inc** | Community | 10 000 SAF | 10 % | 20 % | `addr_safrovaloper1t0aw2zvghsdr7avfksgtsu090w8nvqpckefdsq` |
| 7 | **lehuukhoa** | Community | 10 000 SAF | 10 % | 20 % | `addr_safrovaloper1d2qnc709usexrg9uyjxgfet6xy0vt8wv8jj6m4` |
| 8 | **HusoNode** | Community | 10 000 SAF | 10 % | 20 % | `addr_safrovaloper1t5smj0hxatf05gqw3y75an02lhed7xqewjl3lq` |
| 9 | **VALIDARIOS** | Community | 10 000 SAF | 5 % | 20 % | (operator: `addr_safro1yftmqycaa4td0x6zzgpwcpqg8ze988tdjjrqsg`) |

All 9 gentxs pass `safrochaind genesis collect-gentxs` and `safrochaind genesis validate`, and a clean `InitGenesis` simulation locally. The consensus pubkeys of the foundation validators are the **production** ed25519 keys held by the running daemons and sharded across the 3 Horcrux cosigners (2-of-3 threshold, one cluster per validator).

---

## Verification

Reproduce the checksum and validate the file with the official binary:

```bash
# 1. Download
curl -fsSL -o genesis.json \
  https://raw.githubusercontent.com/Safrochain-Org/draft-genesis/main/genesis.json

# 2. Verify integrity
shasum -a 256 genesis.json
# Expected: f5b699aa9c6d76e18f4abf93c4a2833e09364e40789f62c6d3018a5f9fdf75a2

# 3. Validate against safrochaind
mkdir -p ~/.safrochain/config
cp genesis.json ~/.safrochain/config/genesis.json
safrochaind genesis validate --home ~/.safrochain
# Expected: "File at ... is a valid genesis file"
```

Inspect the bundled gen-txs:

```bash
jq '.app_state.genutil.gen_txs
    | map(.body.messages[0]
          | {moniker: .description.moniker,
             valoper: .validator_address,
             self_stake_usaf: .value.amount,
             commission: .commission.rate})' genesis.json
```

Confirm supply & bonded plan:

```bash
# Total supply (must equal 1e15 usaf = 1B SAF)
jq -r '.app_state.bank.supply[] | "\(.denom): \(.amount)"' genesis.json

# Sum of bank balances (must equal supply)
jq '[.app_state.bank.balances[].coins[]
     | select(.denom == "usaf") | .amount | tonumber] | add' genesis.json

# DAO genesis delegations (totals ~149.08 M SAF)
jq '[.app_state.staking.delegations[] | .shares | tonumber] | add' genesis.json
```

---

## Resources

| | |
| :--- | :--- |
| 🌐 Website  | [safrochain.network](https://safrochain.network) |
| 📦 Node binary | [`safrochaind`](https://github.com/Safrochain-Org) |
| 📝 Issues & review | Open an issue on this repository to flag concerns about this draft |

---

<div align="center">

**Safrochain — Sovereign infrastructure for African on-chain finance**

</div>
