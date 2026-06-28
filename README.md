# BetKarma Provably Fair Anchors

This repository is an append-only audit log for BetKarma provably fair seedsets and reveals, **and** the public trust anchor for the exchange Letter of Guarantee (all swap directions: internal BTC→XMR pool + SHKeeper multi-pair).

Supported games: `coinflip38`, `karmapoker`, `karmaladder`, `dice`.

## Structure

- `anchors/<gameId>/seedsets.jsonl`  
  One line per seedset commit (serverSeedHashHex, rulesVersion, rngVersion).

- `anchors/<gameId>/reveals.jsonl`  
  One line per seedset reveal (serverSeedHex + serverSeedHashHex).  
  Appears only after seedset lifecycle is `REVEALED`.

- `rules/<rulesVersion>.json`  
  Human-readable rules snapshot for each game version.

- `exchange/signing-address.json`  
  Trust anchor for exchange Letters of Guarantee. Publishes the public BTC signing address (`bc1q6etkarmarvej0874ghw396eacejlsy5ygew5yk`), the algorithm (`bitcoin-message-bip137-p2wpkh`), and a link back to the issuer page. The key is dedicated, holds no funds, and never rotates silently — any rotation is committed here first.

- `letter.html`  
  Self-contained HTML/JS verifier for letter-of-guarantee artifacts. Paste the **full** letter (canonical `-----BEGIN BITCOIN SIGNED MESSAGE-----` … `-----END BITCOIN SIGNATURE-----` armor, or the legacy `-----BEGIN BETKARMA LETTER OF GUARANTEE-----` envelope for older downloads) or upload the `.txt` from the order page. Recovers the BIP-137 P2WPKH signer and checks it against `exchange/signing-address.json`. Because the letter is now a standard Bitcoin Signed Message, it also verifies in Electrum / Sparrow / Bitcoin Core and on third-party checkers like bitref.com and bitlist.co. Crypto loads lazily from esm.sh (`@noble/secp256k1@2.1.0`, `@noble/hashes@1.5.0/sha2`, `@noble/hashes@1.5.0/ripemd160`, `@scure/base@1.1.7`). **Do not** import `@noble/hashes@1.5.0/legacy` — that path 404s and breaks all verifications (fixed 2026-05-31, commit `966994a`). No data leaves the page.

- `verify.html`  
  Casino bet receipt verifier (recompute outcomes from anchored seedsets).

## Format

Each file is JSONL (one JSON object per line).  
Lines are stable-serialized (key order is deterministic).

Example seedset commit line:
```
{"ts":"2026-02-21T00:00:00.000Z","gameId":"coinflip38","seedSetId":"<uuid>","serverSeedHashHex":"<64hex>","rulesVersion":"coinflip38-american-v1","rngVersion":"hmac-sha256-v1"}
```

Example reveal line:
```
{"ts":"2026-02-22T00:00:00.000Z","gameId":"coinflip38","seedSetId":"<uuid>","serverSeedHex":"<hex>","serverSeedHashHex":"<64hex>"}
```

## Verification checklist

1) Find `seedSetId` in `seedsets.jsonl` and confirm the hash commit exists.  
2) When the seedset is revealed, verify `sha256(serverSeedHex) == serverSeedHashHex`.  
3) Use the app's verify endpoints and receipts to recompute RNG and confirm outcomes.  

## Append-only guarantee

Files are never edited retroactively.  
Any change to existing lines invalidates the audit log.
