<div align="center">

# Safrochain — Community Validator gentx Repair

**All 9 validators are baked into the launch genesis — the 2 SF Foundation validators and the 7 community validators (NodeStake, catsmile, Winnode, Vinjan.Inc, lehuukhoa, HusoNode, VALIDARIOS). This document remains as the reference for the re-sign procedure that was used.**

[![Chain ID](https://img.shields.io/badge/chain--id-safrochain--1-1f6feb?style=for-the-badge)](#chain-parameters)
[![Binary](https://img.shields.io/badge/safrochaind-v0.2.2-8957e5?style=for-the-badge)](#chain-parameters)
[![Self-stake](https://img.shields.io/badge/self--stake-10%2C000_SAF-2da44e?style=for-the-badge)](#chain-parameters)

</div>

---

## TL;DR — what we need from you

**Send us one file:** a re-signed gentx JSON for your validator. We will drop it into the launch `genesis.json` and reply when it is verified.

**Send it to either of:**

| Channel | Address |
| :------ | :------ |
| 💬 **Discord** (preferred) | DM **`@BDan`** |
| ✉️ Email | `dev@safrochain.com` |

Please also paste the file's **SHA-256** in the message so we can confirm we received the exact same bytes you produced.

---

## Status — find your validator

All 9 gentxs are baked into the launch genesis (sha256 `964423b6008f6c3f0f0410455336372d74bb2f5cda0a8da52dfa152347850ec8`). Each row links to the **accepted** signed gentx.

| Moniker | Issues | Status |
| :------ | :----- | :----- |
| safro-validator-1 | — | ✅ Accepted (block-1 validator) |
| safro-validator-2 | — | ✅ Accepted (block-1 validator) |
| [**NodeStake**](./othersGenesis/gentx-fixed-NodeStake.json) | re-signed with `delegator_address` populated | ✅ **Accepted** (block-1 validator) — `addr_safrovaloper1sdlfp8n5fcfa7qw7770ngqs02k876gf6m749ly` |
| [**catsmile**](./othersGenesis/gentx-fixed-catsmile.json) | re-signed with `delegator_address` populated (new operator + consensus key) | ✅ **Accepted** (block-1 validator) — `addr_safrovaloper13njz6aqmtwtu7vl4w0c6j7dvt7qj6t77vg6r9s` |
| [**Winnode**](./othersGenesis/gentx-fixed-Winnode.json) | re-signed with `delegator_address` populated and `commission.max_change_rate` lowered (0.50 → 0.05); consensus key rotated | ✅ **Accepted** (block-1 validator) — `addr_safrovaloper1a6ve2escz8h4ws3ttelfp54av2wwvty6f4xq8z` |
| [**VALIDARIOS**](./othersGenesis/gentx-fixed-VALIDARIOS.json) | re-signed with `delegator_address` populated; consensus key rotated | ✅ **Accepted** (block-1 validator) — `addr_safrovaloper1yftmqycaa4td0x6zzgpwcpqg8ze988tdyvgcpc` |
| [**HusoNode**](./othersGenesis/gentx-fixed-HusoNode.json) | re-signed with `delegator_address` populated (new operator + consensus key) | ✅ **Accepted** (block-1 validator) — `addr_safrovaloper1t5smj0hxatf05gqw3y75an02lhed7xqewjl3lq` |
| [**Vinjan.Inc**](./othersGenesis/gentx-fixed-Vinjan.Inc.json) | re-signed with `delegator_address` populated (new operator + consensus key) | ✅ **Accepted** (block-1 validator) — `addr_safrovaloper1t0aw2zvghsdr7avfksgtsu090w8nvqpckefdsq` |
| [**lehuukhoa**](./othersGenesis/gentx-fixed-lehuukhoa.json) | re-signed with `delegator_address` populated (new operator + consensus key) | ✅ **Accepted** (block-1 validator) — `addr_safrovaloper1d2qnc709usexrg9uyjxgfet6xy0vt8wv8jj6m4` |

---

## Why your gentx was rejected (the short version)

Your file has `delegator_address: ""` (an empty string) in the staking message. When CometBFT validates the gentx at block 0, it re-encodes the body from JSON back to protobuf to recompute the bytes your signature is supposed to cover — and an **empty string** for that field round-trips into a different byte layout than what your signing tool actually signed. The signatures don't match the recomputed bytes, so InitGenesis rejects the tx with `unauthorized: signature verification failed`.

**Winnode** additionally has `commission.max_change_rate = 0.50`. The chain hard-caps that value at `0.05` (= 5 %), and the staking handler rejects the message before the signature even matters.

The fix for everyone is the same: produce a fresh signed gentx where `delegator_address` is populated with a real, non-empty value. The procedure below does this from scratch in 13 short steps.

---

## Chain parameters

| Field              | Value                              |
| :----------------- | :--------------------------------- |
| Chain ID           | `safrochain-1`                     |
| Bond denom         | `usaf` (1 SAF = 1 000 000 usaf)    |
| Binary             | `safrochaind v0.2.2`               |
| Self-stake amount  | `10000000000usaf` (10 000 SAF)     |
| Account number     | `0` (mandatory for genesis tx)     |
| Sequence           | `0` (mandatory for genesis tx)     |
| Min commission rate | `0.05` (5 %)                      |
| Max `max_change_rate` | `0.05` (5 %) — **hard cap**     |

> **About your self-stake.** Each of the 7 community operator accounts is pre-funded with 10 000 SAF in our draft genesis. The 10 000 SAF you "spend" in the gentx is just the message body — the actual debit happens at block 0 against your operator account on the live chain. The DAO Reserve will additionally delegate ~21 M SAF to each validator **after** launch via a signed multisig tx — that's intentionally not in genesis.

---

## How to produce your signed gentx

This is the exact procedure we ran and verified end-to-end on `safrochaind v0.2.2`. Every step is one copy-paste block. Run them top to bottom, check the expected output between each.

> **⚠️ Important.** This procedure creates **fresh** operator + consensus keys for your validator. Whatever keys it produces become your **permanent** keys for `safrochain-1`. **Back up the mnemonic** printed in Step 4 and **back up `$HOME_GEN/config/priv_validator_key.json`** at the end — losing either is unrecoverable.

### Prerequisites

- `safrochaind v0.2.2` installed (`safrochaind version` must print `v0.2.2`)
- `jq` installed (`brew install jq` / `sudo apt install jq`)

### Step 1 — Set your variables

Edit only the four placeholders (`<...>`). Everything else is fixed by the chain.

```bash
export CHAIN_ID="safrochain-1"
export MONIKER="<your-moniker>"          # e.g. Winnode, NodeStake, catsmile…
export KEY_NAME="<your-key-name>"        # local name for your operator key (any string)
export PUBLIC_IP="<your-public-ip>"      # your validator's public IPv4
export P2P_PORT="26656"
export KEYRING="test"
export HOME_GEN="$HOME/.safro-gentx"
export STAKE_AMOUNT="10000000000usaf"
export COMMISSION_RATE="0.10"
export COMMISSION_MAX_RATE="0.20"
export COMMISSION_MAX_CHANGE="0.05"
```

### Step 2 — Verify your tools

```bash
safrochaind version
jq --version
```

Expected: `v0.2.2` and a `jq` version.

### Step 3 — Initialize a fresh signing home

```bash
rm -rf "$HOME_GEN"
safrochaind init "$MONIKER" --chain-id "$CHAIN_ID" --home "$HOME_GEN"
```

Expected: a JSON dump ending in `"moniker":"<your-moniker>"`. A fresh chain home now exists at `~/.safro-gentx/`.

### Step 4 — Create your operator wallet

```bash
safrochaind keys add "$KEY_NAME" \
  --home "$HOME_GEN" --keyring-backend "$KEYRING"
```

Expected: a JSON line with `address`, `pubkey`, and a `mnemonic` field containing 24 words.

> **🔐 BACK UP THE MNEMONIC NOW.** Write it down on paper or save it in your password manager **before** moving on. This is the only time it is printed. If you lose it, you lose control of your validator's operator account forever.

### Step 5 — Capture your operator address

```bash
export OPERATOR_DELEG=$(safrochaind keys show "$KEY_NAME" -a \
  --home "$HOME_GEN" --keyring-backend "$KEYRING")
echo "Operator (delegator) address: $OPERATOR_DELEG"
```

Expected: `Operator (delegator) address: addr_safro1…` (~43 chars). Send this address to us via Discord/email so we can add it to the pre-funded list in the draft genesis.

### Step 6 — Show your consensus pubkey

```bash
jq '.pub_key' "$HOME_GEN/config/priv_validator_key.json"
```

Expected: a JSON object `{ "type": "tendermint/PubKeyEd25519", "value": "..." }`. This is the **consensus key** your live validator node will need. Back up the `priv_validator_key.json` file at the very end of this procedure.

### Step 7 — Add your account to the local genesis

Required so the next step's balance check passes.

```bash
safrochaind genesis add-genesis-account "$OPERATOR_DELEG" "$STAKE_AMOUNT" \
  --home "$HOME_GEN"
```

Expected: silent success. Verify with:

```bash
jq --arg a "$OPERATOR_DELEG" '.app_state.bank.balances[] | select(.address == $a)' \
  "$HOME_GEN/config/genesis.json"
```

You should see your address with `10000000000 usaf`.

### Step 8 — Generate the unsigned gentx

```bash
safrochaind genesis gentx "$KEY_NAME" "$STAKE_AMOUNT" \
  --home "$HOME_GEN" --chain-id "$CHAIN_ID" --keyring-backend "$KEYRING" \
  --moniker "$MONIKER" \
  --commission-rate "$COMMISSION_RATE" \
  --commission-max-rate "$COMMISSION_MAX_RATE" \
  --commission-max-change-rate "$COMMISSION_MAX_CHANGE" \
  --min-self-delegation 1 \
  --ip "$PUBLIC_IP" --p2p-port "$P2P_PORT" \
  --generate-only --output-document unsigned.json
```

Expected: `Genesis transaction written to "unsigned.json"`.

### Step 9 — Build the P2P memo

```bash
export NODE_ID=$(safrochaind comet show-node-id --home "$HOME_GEN")
export MEMO="${NODE_ID}@${PUBLIC_IP}:${P2P_PORT}"
echo "Memo: $MEMO"
```

Expected: `Memo: <40-hex-chars>@<your-ip>:26656`.

### Step 10 — Patch the body

Sets `delegator_address`, writes the P2P memo, and clears the signature placeholder so Step 11 writes a fresh signature.

```bash
jq --arg d "$OPERATOR_DELEG" --arg m "$MEMO" '
  .body.messages[0].delegator_address = $d
  | .body.memo                        = $m
  | .auth_info.signer_infos[0].sequence = "0"
  | .signatures = [""]
' unsigned.json > unsigned-patched.json
```

Expected: no output. Verify with:

```bash
jq '.body.messages[0].delegator_address, .body.memo' unsigned-patched.json
```

You should see your `addr_safro1…` and your memo.

### Step 11 — Sign offline

```bash
SIGNED="gentx-fixed-${MONIKER}.json"

safrochaind tx sign unsigned-patched.json \
  --from "$KEY_NAME" --home "$HOME_GEN" --keyring-backend "$KEYRING" \
  --chain-id "$CHAIN_ID" --offline \
  --account-number 0 --sequence 0 --overwrite \
  --output-document "$SIGNED"
```

Expected: no output. A new file `gentx-fixed-<your-moniker>.json` appears in your current directory.

> `--overwrite` is required: the patch in Step 10 leaves an empty signature placeholder, and without `--overwrite` the CLI appends instead of replacing, which trips the "max 1 DIRECT signer" guard.

### Step 12 — Inspect and capture the SHA-256

```bash
jq '.body.messages[0] | {
  moniker:           .description.moniker,
  delegator_address,
  validator_address,
  consensus_pubkey:  .pubkey.key,
  commission,
  amount:            .value.amount
}' "$SIGNED"

shasum -a 256 "$SIGNED"
```

Expected: a clean JSON object with your moniker, your `addr_safro1…` as `delegator_address`, your `addr_safrovaloper1…`, your consensus pubkey, your commission, and `amount: "10000000000"` — followed by one 64-hex-char SHA-256 line.

### Step 13 — Back up everything you need to keep, then send the file

**Permanent backups (do these before deleting `$HOME_GEN`):**

```bash
# Your consensus key — your live validator node will need this file.
cp "$HOME_GEN/config/priv_validator_key.json" \
   ~/safro-validator-backup-priv_validator_key.json

# Your operator mnemonic — already shown in Step 4. Save it to a password manager.
```

**Then send us the signed gentx:**

| Channel | How |
| :------ | :-- |
| 💬 **Discord** (preferred) | DM **`@BDan`** with `gentx-fixed-<your-moniker>.json` attached, plus the SHA-256 from Step 12, plus your `OPERATOR_DELEG` address from Step 5 |
| ✉️ Email | `dev@safrochain.com` with the same three items |

We will validate the file against the draft genesis and reply within ~24 hours.

### (Optional) Tear-down

Once you have backed up the mnemonic and `priv_validator_key.json`:

```bash
rm -rf "$HOME_GEN" unsigned.json unsigned-patched.json
# Keep "$SIGNED" until we confirm receipt.
```

---

## Troubleshooting

| Symptom | Likely cause | Fix |
| :------ | :----------- | :-- |
| Step 8 fails with `account ... does not have a balance in the genesis state` | You skipped Step 7 | Run Step 7 (`genesis add-genesis-account`), then retry Step 8 |
| Step 11 fails with `txs signed with CLI can have maximum 1 DIRECT signer` | You forgot `--overwrite` | Re-run Step 11 with `--overwrite` (already shown above) |
| Step 11 fails with `account sequence mismatch` | You forgot `--offline` | Re-run Step 11 with `--offline --account-number 0 --sequence 0` (already shown above) |
| `commission rate cannot be less than min commission rate` | Your `COMMISSION_RATE` is below `0.05` | Set `COMMISSION_RATE="0.05"` (or higher) in Step 1 |
| `commission max change rate too high` | `COMMISSION_MAX_CHANGE` exceeds `0.05` | Set `COMMISSION_MAX_CHANGE="0.05"` (or lower) in Step 1 |
| Step 12 shows the wrong `validator_address` | `MONIKER` or `KEY_NAME` was wrong in Step 1 | Restart from Step 3 (`rm -rf "$HOME_GEN"` resets everything) |

If you hit anything else, **ping `@BDan` on Discord with the error** — we will get back to you quickly.

---

## Help & contact

- 💬 **Discord** — DM **`@BDan`** (preferred for fast back-and-forth)
- ✉️ **Email** — `dev@safrochain.com`
- 🐛 **Issue tracker** — open an issue on this repository
- 🌐 **Website** — [safrochain.network](https://safrochain.network)
- 📦 **Binary source** — [Safrochain-Org/safrochain-node](https://github.com/Safrochain-Org/safrochain-node)

---

<div align="center">

**Safrochain — Sovereign infrastructure for African on-chain finance**

</div>
