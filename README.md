# BetKarma Provably Fair Anchors

This repository is an append-only audit log for BetKarma provably fair seedsets and reveals.

## Structure

- `anchors/<gameId>/seedsets.jsonl`  
  One line per seedset commit (serverSeedHashHex, rulesVersion, rngVersion).

- `anchors/<gameId>/reveals.jsonl`  
  One line per seedset reveal (serverSeedHex + serverSeedHashHex).  
  Appears only after seedset lifecycle is `REVEALED`.

- `rules/<rulesVersion>.json`  
  Human-readable rules snapshot for each game version.

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
