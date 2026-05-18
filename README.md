<div align="center">

# Safrochain Mainnet — Draft Genesis

**The canonical draft `genesis.json` for `safrochain-1`**

[![Chain ID](https://img.shields.io/badge/chain--id-safrochain--1-1f6feb?style=for-the-badge)](#chain-parameters)
[![Denom](https://img.shields.io/badge/denom-usaf-8957e5?style=for-the-badge)](#chain-parameters)
[![Supply](https://img.shields.io/badge/supply-1%2C000%2C000%2C000_SAF-2da44e?style=for-the-badge)](#tokenomics)
[![Validators](https://img.shields.io/badge/genesis_validators-6-fb8500?style=for-the-badge)](#genesis-validators)
[![Status](https://img.shields.io/badge/status-draft-d29922?style=for-the-badge)](#status)

</div>

---

## Status

> **⚠️ DRAFT** — This file is published for community review of the initial state ahead of network launch. The `genesis_time`, validator set, and individual delegations may still change before the final launch genesis is cut. Treat the SHA below as a checkpoint, not the launch hash.

| | |
| --- | --- |
| **File** | [`genesis.json`](./genesis.json) |
| **Size** | 40,987 bytes (≈ 40 KB) |
| **SHA-256** | `da7c9be5ef09c32b3d23a18fe66bdbeb07fd7183d0f85f9cbbc5737c25457403` |
| **Genesis time** | `2026-04-18T15:44:32.803866Z` |
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
| **Min gas price** | `100 000 usaf` (global fee) |
| **Inflation** | min 3 % · max 14 % · target bonded 67 % |
| **Slash — downtime** | 0.01 % · jail 1 h · window 10 000 blocks @ 80 % |
| **Slash — double-sign** | 5 % |
| **Evidence max age** | 14 days / 241 920 blocks |
| **Max block bytes** | 22 020 096 |
| **Max block gas** | 100 000 000 |

---

## Tokenomics

**Total supply: 1 000 000 000 SAF**

| Allocation | Amount (SAF) | % of supply | Notes |
| :--- | ---: | ---: | :--- |
| **DAO Reserve** | 200 000 000 | 20 % | Governance & community-driven decisions. 170 K SAF earmarked to fund validator self-stake wallets at genesis; ~149 M SAF pre-delegated across the 9 genesis validators. |
| **Community** | 150 000 000 | 15 % | Airdrops, missions, community rewards |
| **Team** | 150 000 000 | 15 % | Team allocation (to be split into vesting schedules in a future genesis revision) |
| **Ecosystem Development** | 150 000 000 | 15 % | Ecosystem growth and developer programs |
| **Public Sale** | 100 000 000 | 10 % | Public sale participants |
| **Liquidity** | 100 000 000 | 10 % | DeFi liquidity pools & market making |
| **Treasury** | 100 000 000 | 10 % | Foundation treasury |
| **Partners** | 50 000 000 | 5 % | Strategic partnerships |
| **— Total** | **1 000 000 000** | **100 %** | |

### Initial bonded stake

| Source | Bonded at genesis |
| :--- | ---: |
| Community validator self-delegations (7 × 10 K) | 70 000 SAF |
| SF foundation validator self-delegations (2 × 50 K) | 100 000 SAF |
| DAO Reserve delegations to all 9 validators | 149 079 999 SAF |
| **Total bonded at block 1** | **≈ 149.25 M SAF (~14.9 % of supply)** |

---

## Genesis Validators

The validator set at block 1 consists of the **2 Safrochain Foundation validators + 4 community validators** that re-signed their gentxs after the InitGenesis compliance review.

| # | Moniker | Class | Self-stake | Commission | Max | Operator (valoper) |
| :-- | :--- | :--- | ---: | ---: | ---: | :--- |
| 1 | safro-validator-1 | Foundation | 50 000 SAF | 10 % | 20 % | `addr_safrovaloper1j9l9salsfwq83cf83zsg7wdhvc5p2f6xshlpzn` |
| 2 | safro-validator-2 | Foundation | 50 000 SAF | 10 % | 20 % | `addr_safrovaloper1h2jy3sghpwevgw3cjwqk055ejwxn5ls3ur7c5q` |
| 3 | **NodeStake** | Community | 10 000 SAF | 5 % | 20 % | `addr_safrovaloper1sdlfp8n5fcfa7qw7770ngqs02k876gf6m749ly` |
| 4 | **catsmile** | Community | 10 000 SAF | 10 % | 20 % | `addr_safrovaloper13njz6aqmtwtu7vl4w0c6j7dvt7qj6t77vg6r9s` |
| 5 | **Winnode** | Community | 10 000 SAF | 5 % | 20 % | `addr_safrovaloper1a6ve2escz8h4ws3ttelfp54av2wwvty6f4xq8z` |
| 6 | **Vinjan.Inc** | Community | 10 000 SAF | 10 % | 20 % | `addr_safrovaloper1t0aw2zvghsdr7avfksgtsu090w8nvqpckefdsq` |

### Community validators expected post-launch

The remaining 3 community operators are pre-funded with 10 000 SAF each in genesis, but their original gentxs failed compliance. They will join the network post-launch by submitting `MsgCreateValidator` after re-signing with the fixes below (or directly against the running chain):

| Moniker | Operator wallet (pre-funded, 10 000 SAF) | Issue with original gentx |
| :--- | :--- | :--- |
| VALIDARIOS | `addr_safro1yftmqycaa4td0x6zzgpwcpqg8ze988tdjjrqsg` | empty `delegator_address` field — re-sign with operator address set |
| HusoNode | `addr_safro1v7tgzrhgfhlfk07u24cy76pktq2qqxqyfu04gm` | empty `delegator_address` field — re-sign with operator address set |
| lehuukhoa | `addr_safro1p9nru77vj63uw25hx68hp4s9zrmr8cl4flcc2m` | empty `delegator_address` field — re-sign with operator address set |

All 4 baked gentxs pass `safrochaind genesis collect-gentxs`, `safrochaind genesis validate`, and a clean `InitGenesis` simulation locally. The consensus pubkeys of the foundation validators are the **production** ed25519 keys held by the running daemons and sharded across the 3 Horcrux cosigners (2-of-3 threshold, one cluster per validator).

---

## Verification

Reproduce the checksum and validate the file with the official binary:

```bash
# 1. Download
curl -fsSL -o genesis.json \
  https://raw.githubusercontent.com/Safrochain-Org/draft-genesis/main/genesis.json

# 2. Verify integrity
shasum -a 256 genesis.json
# Expected: da7c9be5ef09c32b3d23a18fe66bdbeb07fd7183d0f85f9cbbc5737c25457403

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
