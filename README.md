# BetKarma Provably Fair Anchors

This repository is an append-only audit log for BetKarma provably fair seedsets and reveals, **and** the public trust anchor for the BTC -> XMR exchange Letter of Guarantee.

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
  Trust anchor for the BTC -> XMR exchange Letter of Guarantee. Publishes the public BTC signing address, the algorithm (`bitcoin-message-bip137-p2wpkh`), and a link back to the issuer page. The key is dedicated, holds no funds, and never rotates silently — any rotation is committed here first.

- `letter.html`  
  Self-contained HTML/JS verifier for letter-of-guarantee artifacts. Drop in the `.txt` letter (or paste the text), and the page checks the SHA-256 + Bitcoin message signature in your browser. No data leaves the page.

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
