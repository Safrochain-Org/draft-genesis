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
| **Size** | 40,987 bytes (≈ 40 KB) |
| **SHA-256** | `d2b0951bce8e7804efecb8e2e36ae025016550fabd98f8c7471f762d4382b99d` |
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

The validator set at block 1 consists of 7 community validators and 2 Safrochain Foundation validators. All gen-txs are cryptographically verified and collected in `app_state.genutil.gen_txs`.

| # | Moniker | Self-stake | Commission | Max | Operator (valoper) |
| :-- | :--- | ---: | ---: | ---: | :--- |
| 1 | safro-validator-1 | 50 000 SAF | 10 % | 20 % | `addr_safrovaloper1j9l9salsfwq83cf83zsg7wdhvc5p2f6xshlpzn` |
| 2 | safro-validator-2 | 50 000 SAF | 10 % | 20 % | `addr_safrovaloper1h2jy3sghpwevgw3cjwqk055ejwxn5ls3ur7c5q` |
| 3 | NodeStake          | 10 000 SAF | 5 %  | 20 % | `addr_safrovaloper1sdlfp8n5fcfa7qw7770ngqs02k876gf6m749ly` |
| 4 | Winnode            | 10 000 SAF | 10 % | 50 % | `addr_safrovaloper1a6ve2escz8h4ws3ttelfp54av2wwvty6f4xq8z` |
| 5 | VALIDARIOS         | 10 000 SAF | 10 % | 20 % | `addr_safrovaloper1yftmqycaa4td0x6zzgpwcpqg8ze988tdyvgcpc` |
| 6 | catsmile           | 10 000 SAF | 10 % | 20 % | `addr_safrovaloper1n5wh99ntprx89656y3rtlarldxsztgpfwkr37e` |
| 7 | HusoNode           | 10 000 SAF | 5 %  | 20 % | `addr_safrovaloper1v7tgzrhgfhlfk07u24cy76pktq2qqxqylzydet` |
| 8 | Vinjan.Inc         | 10 000 SAF | 5 %  | 50 % | `addr_safrovaloper155sufte0atrxla4duvx94rh3s7u5k8cqyn9lm9` |
| 9 | lehuukhoa          | 10 000 SAF | 5 %  | 20 % | `addr_safrovaloper1p9nru77vj63uw25hx68hp4s9zrmr8cl4lpnqmt` |

> All 9 entries pass `safrochaind genesis collect-gentxs` and `safrochaind genesis validate`. Consensus pubkeys, operator addresses, and signers are unique across the set.

---

## Verification

Reproduce the checksum and validate the file with the official binary:

```bash
# 1. Download
curl -fsSL -o genesis.json \
  https://raw.githubusercontent.com/Safrochain-Org/draft-genesis/main/genesis.json

# 2. Verify integrity
shasum -a 256 genesis.json
# Expected: d2b0951bce8e7804efecb8e2e36ae025016550fabd98f8c7471f762d4382b99d

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
