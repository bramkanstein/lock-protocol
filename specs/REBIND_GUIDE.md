# REBIND_GUIDE.md

## Rebinding Vault Ownership in the LOCK Protocol

Rebinding is the act of transferring access to a sealed vault from one Bitcoin wallet to another.  
This is a **core feature** of the LOCK protocol and is enforced via a cryptographically signed Bitcoin transaction.

📎 Vault Rebinding Note

Vault rebinding alters the vault’s metadata and PoA verification conditions but does not change the cryptographic key used to encrypt or decrypt the SEAL. The SEAL remains accessible to any authorized wallet per the original ECDH key derivation.

---

## 🔁 What Rebinding Does

Rebinding:

- Updates the `authorized_wallet` to a new address
- Binds the vault to a **new transaction ID (TXID)**
- Changes the `vault_id` (which includes the new TXID)
- Preserves or resets metadata (depending on client design)

---

## ✅ Required Protocol Behavior

To rebind a vault, clients must:

1. Require a new transaction from the **new wallet** (this becomes the new TXID)
2. Verify a **signature from the current (old) wallet**, proving authorization
3. Append the new TXID to the vault metadata
4. Recompute the vault ID:
vault_id = hash(SEAL + metadata + new_txid)
5. Optionally reset or preserve unlock counters

---

## 🔐 Signature Verification

Clients must enforce:

```python
verify_signature(
message = vault.txid,
signature = old_wallet_sig,
pubkey = vault.bound_wallet_pubkey
)
```
If this check fails, the rebind must be rejected.

---

## ⚖️ Client Responsibilities

The LOCK protocol defines how rebinding is validated, but clients must decide:

- Whether to expose rebinding via UI
- Whether to monetize rebinding (e.g. fees)
- Whether to preserve or reset unlock counters
- How to show rebinding in audit logs

---

## 🧩 Rebinding Use Cases

| Scenario         | Description                                           |
|------------------|-------------------------------------------------------|
| Key Rotation     | Periodic hygiene — move vault to a new wallet         |
| Loss Recovery    | Still have the old wallet → can reclaim access        |
| Delegation       | Transfer vault to a partner or successor              |
| Inheritance      | Transfer to a new wallet posthumously                 |
| Access Expiry    | Rebind to a time-locked wallet with delayed use       |

---

## 🛑 Rebinding Must Be Explicit

Rebinding **must not** be automatic.  
It must be:

- Requested by the user  
- Signed by the old wallet  
- Bound to a new on-chain transaction

## 💰 Amount Conditions During Rebinding

When rebinding a vault, the amount conditions must be preserved:

- For fixed amounts: The same amount requirement is maintained
- For range amounts: The range parameters (min/max) are preserved, but a new random amount is selected during the rebinding PSBT generation
- The selected amount from the original binding is not carried over to the rebinding

This ensures that:
1. The security properties of the amount condition are maintained
2. Each rebinding creates a new, independent amount requirement
3. The range parameters remain consistent with the vault's original configuration

---

## 💬 Example Flow (Client Perspective)

1. User selects vault to transfer  
2. Client generates a PSBT for the new wallet  
3. Client prompts old wallet to sign the TXID  
4. Vault is re-bound and `vault_id` is updated  
5. Vault can now be unsealed by the new wallet
