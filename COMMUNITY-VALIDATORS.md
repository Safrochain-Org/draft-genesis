<div align="center">

# Safrochain — Community Validator gentx Repair

**A complete step-by-step guide for the 7 community validators who need to re-sign their gentx for `safrochain-1`.**

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

The two SF Foundation gentxs are already valid and will form the block-1 set. The seven below are rejected and need a fresh signature. **Click your moniker** to download your original gentx.

| Moniker | Issues | Status |
| :------ | :----- | :----- |
| safro-validator-1 | — | ✅ Accepted (block-1 validator) |
| safro-validator-2 | — | ✅ Accepted (block-1 validator) |
| [**Winnode**](./othersGenesis/gentx-0ac3a0672fb9a91b5651dbea198e5209d57d45ce.json) | `delegator_address` + `commission.max_change_rate` (0.50 → 0.05) | ❌ Awaiting re-sign |
| [**NodeStake**](./othersGenesis/gentx-NodeStake.json) | `delegator_address` | ❌ Awaiting re-sign |
| [**VALIDARIOS**](./othersGenesis/gentx-1adee57b2f70d1759b8b05330b757ed31c94655a.json) | `delegator_address` | ❌ Awaiting re-sign |
| [**catsmile**](./othersGenesis/gentx-fbe6e37cec0acbf9c310bcb5a753c5bc6064a94a.json) | `delegator_address` | ❌ Awaiting re-sign |
| [**HusoNode**](./othersGenesis/gentx-35b4137ae4011298e46629b5f820ca26257410ba.json) | `delegator_address` | ❌ Awaiting re-sign |
| [**Vinjan.Inc**](./othersGenesis/gentx-b7a2c40d3e24f859649f995889904c7bb23a5b5c.json) | `delegator_address` | ❌ Awaiting re-sign |
| [**lehuukhoa**](./othersGenesis/gentx-ab6de7a06dbb514d22a070db111c6fb4751b9710.json) | `delegator_address` | ❌ Awaiting re-sign |

---

## Why your gentx was rejected (the short version)

Your file has `delegator_address: ""` (an empty string) in the staking message. When CometBFT validates the gentx at block 0, it re-encodes the body from JSON back to protobuf to recompute the bytes your signature is supposed to cover — and an **empty string** for that field round-trips into a different byte layout than what your signing tool actually signed. The signatures don't match the recomputed bytes, so InitGenesis rejects the tx with `unauthorized: signature verification failed`.

**Winnode** additionally has `commission.max_change_rate = 0.50`. The chain hard-caps that value at `0.05` (= 5 %), and the staking handler rejects the message before the signature even matters.

The fix is the same for everyone: populate `delegator_address` with a real, non-empty value (your own operator account address), then re-sign over the new body. Once the field has actual bytes, both encoders agree on the byte layout, the round-trip is lossless, and the signature checks out.

---

## Chain parameters

| Field              | Value                              |
| :----------------- | :--------------------------------- |
| Chain ID           | `safrochain-1`                     |
| Bond denom         | `usaf` (1 SAF = 1 000 000 usaf)    |
| Binary             | `safrochaind v0.2.2`               |
| Self-stake amount  | `10000000000usaf` (10 000 SAF)     |
| Self-stake source  | Your **operator account** (the `addr_safro1…` derived from your `addr_safrovaloper1…`). It is already pre-funded with 10 000 SAF in the draft genesis. |
| Account number     | `0` (mandatory for genesis tx)     |
| Sequence           | `0` (mandatory for genesis tx)     |
| Min commission rate | `0.05` (5 %)                      |
| Max `max_change_rate` | `0.05` (5 %) — **hard cap**     |

> **Delegator = operator.** In your re-signed gentx, `delegator_address` must be your own operator account address (the `addr_safro1…` form of your `addr_safrovaloper1…`). The signer of the tx must be the same key that controls that account. Each operator account has been pre-funded with 10 000 SAF in the draft genesis, which becomes your initial self-delegation at block 1. The DAO Reserve will delegate additional SAF to all 9 validators **after** launch via a signed multisig tx — that is intentionally not in the genesis state.

---

## Before you start — checklist

1. **The same machine** you used the first time, ideally — it already has your operator key and your consensus key (`priv_validator_key.json`). If you re-sign from a different machine, you must import the same operator key into the new keyring **and** make sure the consensus pubkey in your `priv_validator_key.json` matches the one already embedded in your original gentx (extract it with `jq -r '.body.messages[0].pubkey.key' gentx-original.json`).
2. **`safrochaind v0.2.2`** installed:
   ```bash
   safrochaind version
   # Expected: v0.2.2
   ```
   Older versions are exactly what produced the bug, so this matters.
3. **`jq`** installed (for the patch step). Most distros: `sudo apt install jq` / `brew install jq`.
4. **Your operator key name & keyring backend** ready (e.g. `--from winnode --keyring-backend os`). Confirm with:
   ```bash
   safrochaind keys list --keyring-backend <BACKEND>
   ```
   You should see your key. To make sure it is the **same** key you used the first time, compare its pubkey to the one already embedded in your original gentx:
   ```bash
   safrochaind keys show <KEY_NAME> -p --keyring-backend <BACKEND>
   # Expected: a JSON line whose "key" field equals
   #   .auth_info.signer_infos[0].public_key.key  in your original gentx
   jq -r '.auth_info.signer_infos[0].public_key.key' gentx-original.json
   ```
   The two base64 strings must match.

---

## Two ways to produce the re-signed gentx

You only need to do **one** of these.

| Path | When to use it | Effort |
| :--- | :------------- | :----- |
| **A — Patch your existing gentx** (recommended) | You still have your original gentx and your operator key | 4 short shell commands |
| **B — Regenerate from scratch** | You lost your original file, or you suspect your earlier signing tool was buggy | A bit more setup |

Both paths produce a functionally identical signed JSON. We accept either one.

---

## Path A — Patch your existing gentx and re-sign (recommended)

This is the fastest route. We keep everything from your original gentx (moniker, identity, consensus pubkey, P2P memo, commission settings) and only change the broken field(s), then sign with your operator key.

### Step A.1 — Get your original gentx

If you do not already have it locally, download it from this repo:

```bash
# Replace <YOUR_FILE> with the filename from the Status table above
curl -fsSL -o gentx-original.json \
  https://raw.githubusercontent.com/Safrochain-Org/draft-genesis/main/othersGenesis/<YOUR_FILE>
```

### Step A.2 — Derive your operator account address

This is the value that must go into `delegator_address`. Pull it directly from your keyring so there is zero risk of typing the wrong address:

```bash
# Replace <KEY_NAME> and <BACKEND> with your values
OPERATOR_DELEG=$(safrochaind keys show <KEY_NAME> -a --keyring-backend <BACKEND>)
echo "Operator account: $OPERATOR_DELEG"
```

**Sanity check.** `$OPERATOR_DELEG` must be the `addr_safro1…` form of your `addr_safrovaloper1…`. Same 20-byte hash, different bech32 prefix — both are derivable from the same operator key. If your key returns a totally unrelated address, you're using the wrong key.

### Step A.3 — Patch the body

This is one `jq` call. It sets `delegator_address`, ensures the genesis sequence (`0`) is in place, and clears the stale signature so `tx sign` writes a fresh one over the new bytes.

```bash
jq --arg d "$OPERATOR_DELEG" '
  .body.messages[0].delegator_address = $d
  | .auth_info.signer_infos[0].sequence = "0"
  | .signatures = [""]
' gentx-original.json > unsigned.json
```

**Winnode only — also fix the commission cap:**

```bash
jq '.body.messages[0].commission.max_change_rate = "0.050000000000000000"' \
  unsigned.json > unsigned.tmp && mv unsigned.tmp unsigned.json
```

(This brings `max_change_rate` from `0.50` down to `0.05`, the chain's hard cap. You can keep `max_rate = 0.50` if you want a high commission ceiling — only the per-epoch change rate is capped.)

### Step A.4 — Sign offline

`--offline --account-number 0 --sequence 0` are mandatory for genesis transactions (your operator account does not exist on chain yet, so the signer cannot query for an account number — we tell it to assume `0`, which is what InitGenesis will use).

```bash
safrochaind tx sign unsigned.json \
  --from <KEY_NAME> --keyring-backend <BACKEND> \
  --chain-id safrochain-1 --offline \
  --account-number 0 --sequence 0 \
  --output-document gentx-fixed-<YOUR_MONIKER>.json
```

Replace `<YOUR_MONIKER>` with your moniker in lowercase, e.g. `winnode`, `nodestake`, `catsmile`. The output filename pattern is `gentx-fixed-<moniker>.json`.

✅ **You now have the signed file.** Skip to [Verify and send](#verify-and-send).

---

## Path B — Regenerate the gentx from scratch

Use this if you do not have your original file, or you want to start clean. The procedure mirrors what we do internally for our own SF Foundation validators (see [`provisioning/genesis/remote-validator-gentx.sh`](../provisioning/genesis/remote-validator-gentx.sh) in our node repo).

### Step B.1 — Initialize a clean signing home

A "home" here is just a directory `safrochaind` uses for its keyring, config, and consensus key. We make a fresh one so we do not disturb your running node.

```bash
export HOME_GEN="$HOME/safro-gentx-home"
rm -rf "$HOME_GEN"
safrochaind init "<YOUR_MONIKER>" --chain-id safrochain-1 --home "$HOME_GEN"
```

### Step B.2 — Bring in your operator key

You must use the **same** operator key whose pubkey already appears in your original gentx — i.e. the key whose secp256k1 pubkey equals `auth_info.signer_infos[0].public_key.key` in your downloaded gentx. Otherwise we will not be able to merge your file (the operator account in our genesis is bound to that specific public key).

```bash
# If your key already exists in another keyring, import it into HOME_GEN's keyring:
safrochaind keys add <KEY_NAME> --recover --home "$HOME_GEN" --keyring-backend <BACKEND>
# (Paste your mnemonic when prompted. Use the same key you used the first time.)
```

### Step B.3 — Bring in your existing consensus key

This is critical. Your **consensus** key is the ed25519 key your running validator node uses to sign blocks (it lives at `<your-node-home>/config/priv_validator_key.json`). The pubkey is what appears in `body.messages[0].pubkey.key` of your original gentx — that exact value is what we have registered for your validator.

If you re-init a fresh home, `safrochaind` will generate a **new** ed25519 key by default — that is the wrong key, and the chain will reject your gentx because the consensus pubkey will not match what we already know about your validator. So copy your existing one in **before** generating the gentx:

```bash
# Replace <PATH_TO_YOUR_RUNNING_NODE_HOME> — usually /var/lib/safrochain or ~/.safrochain
install -m 600 \
  <PATH_TO_YOUR_RUNNING_NODE_HOME>/config/priv_validator_key.json \
  "$HOME_GEN/config/priv_validator_key.json"
```

You can confirm the right pubkey is in place by comparing it to your downloaded original gentx:

```bash
jq -r '.pub_key.value' "$HOME_GEN/config/priv_validator_key.json"
jq -r '.body.messages[0].pubkey.key'   gentx-original.json
# These two base64 strings MUST match. If they differ, your priv_validator_key.json
# is from a different machine — locate the one used to produce your original gentx.
```

### Step B.4 — Generate the unsigned gentx

```bash
# === EDIT THESE ===
KEY_NAME="<your_key_name>"
KEYRING="<your_backend>"      # os | file | test | pass
MONIKER="<your_moniker>"      # e.g. Winnode, NodeStake, catsmile…
PUBLIC_IP="<your_public_ip>"  # from your row's P2P memo
P2P_PORT="26656"              # from your row's P2P memo (usually 26656)

# === COMMISSION (edit if you want different values within policy) ===
COMMISSION_RATE="0.10"           # >= 0.05 (chain minimum)
COMMISSION_MAX_RATE="0.20"       # any value >= COMMISSION_RATE
COMMISSION_MAX_CHANGE="0.05"     # <= 0.05 (chain hard cap)

safrochaind genesis gentx "$KEY_NAME" "10000000000usaf" \
  --home "$HOME_GEN" --chain-id safrochain-1 --keyring-backend "$KEYRING" \
  --moniker "$MONIKER" \
  --commission-rate "$COMMISSION_RATE" \
  --commission-max-rate "$COMMISSION_MAX_RATE" \
  --commission-max-change-rate "$COMMISSION_MAX_CHANGE" \
  --min-self-delegation 1 \
  --ip "$PUBLIC_IP" --p2p-port "$P2P_PORT" \
  --generate-only --output-document unsigned.json
```

### Step B.5 — Patch + sign

This patches in `delegator_address` and signs. Same logic as Path A.3 and A.4:

```bash
OPERATOR_DELEG=$(safrochaind keys show "$KEY_NAME" -a --home "$HOME_GEN" --keyring-backend "$KEYRING")
NODE_ID=$(safrochaind comet show-node-id --home "$HOME_GEN")
MEMO="${NODE_ID}@${PUBLIC_IP}:${P2P_PORT}"

jq --arg d "$OPERATOR_DELEG" --arg m "$MEMO" '
  .body.messages[0].delegator_address = $d
  | .body.memo = $m
  | .auth_info.signer_infos[0].sequence = "0"
  | .signatures = [""]
' unsigned.json > unsigned-patched.json

safrochaind tx sign unsigned-patched.json \
  --from "$KEY_NAME" --home "$HOME_GEN" --keyring-backend "$KEYRING" \
  --chain-id safrochain-1 --offline \
  --account-number 0 --sequence 0 \
  --output-document "gentx-fixed-${MONIKER,,}.json"
# (the ${MONIKER,,} lowercases the moniker for the filename — bash 4+)
```

✅ **You now have the signed file.**

---

## Verify and send

Whichever path you took, run these two checks before sending:

```bash
# 1. Eyeball the important fields. They should match your original gentx
#    (with delegator_address now populated, and max_change_rate corrected if you're Winnode).
jq '.body.messages[0] | {
  moniker: .description.moniker,
  delegator_address,
  validator_address,
  consensus_pubkey: .pubkey.key,
  commission,
  amount: .value.amount
}' gentx-fixed-<YOUR_MONIKER>.json

# 2. Compute the SHA — include this in your message to us.
shasum -a 256 gentx-fixed-<YOUR_MONIKER>.json
```

Then send:

| Channel | How |
| :------ | :-- |
| 💬 **Discord (preferred)** | DM **`@BDan`** with the file attached and the SHA-256 in the message |
| ✉️ Email | `dev@safrochain.com` with the file attached and the SHA-256 in the body |

We will run `safrochaind genesis collect-gentxs` + a full InitGenesis simulation against the draft genesis and reply with a ✅ within ~24 hours.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
| :------ | :----------- | :-- |
| `safrochaind keys show <KEY> -p` returns a pubkey that does not match `auth_info.signer_infos[0].public_key.key` in your original gentx | You're using the wrong key, or wrong keyring backend | Try other backends (`--keyring-backend file/os/test`), or import the right mnemonic. Your operator key is the secp256k1 one whose pubkey matches your original gentx's signer pubkey. |
| `tx sign` fails with `account sequence mismatch` | You forgot `--offline` | Re-run with `--offline --account-number 0 --sequence 0`. Genesis txs cannot query account state — they must use these fixed values. |
| `tx sign` writes a file but the signature still does not validate | You signed with a different key than the one you registered originally | The consensus pubkey is bound to the operator account in our genesis. Use the same secp256k1 key you used the first time. |
| You generated the gentx on a fresh machine and the consensus pubkey changed | A new `priv_validator_key.json` was auto-generated | Copy your real `priv_validator_key.json` from your running node into the signing home **before** `genesis gentx` — see Path B Step B.3. |
| `commission rate cannot be less than min commission rate` | Your `commission.rate` is below `0.05` | Set `--commission-rate` to at least `0.05`. |
| `max change rate too high` | Your `max_change_rate` exceeds `0.05` | Set `--commission-max-change-rate 0.05` or lower. |

If you hit anything else, **just ping `@BDan` on Discord with the error** — we will get back to you quickly.

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
