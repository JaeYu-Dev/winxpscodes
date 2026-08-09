# Research Changelog

This file is the fastest handoff path for a new researcher. Read newest version first, then follow referenced findings.

## v0.1.4 — Early RSAENH ancestry pruning
- Added F006: composed fixed pre-init RSAENH state + F005 same-stream reuse through provider initialization and CryptAcquireContext bridge.
- Formal early-state dependency is now `S2 = F(M1,L1,L2,B1,B2)` for fixed shipped constants.
- KSecDD rekey roots M2..M8 are proven absent from the data-flow ancestry of S2 even though their rekeys occur chronologically between the first two useful SystemFunction036 outputs.
- Promoted the fixed post-AlgorithmCheck RSAENH state into the active PK-only model rather than counting it as a hidden 160-bit root.
- Next target is exact last-writer provenance of L1/L2 and rsaenh caller-prefixes B1/B2. V17 raw values are strongly structured but are not yet assigned a support bound.

## v0.1.3 — NT threshold mutation + same-stream reuse
- Directly read V17 raw SP3 trace and all eight 256-byte IOCTL input blobs.
- Resolved first/second call asymmetry: `g_dwRC4RekeyParam` changes from 512 to 16384 **inside the first NT rekey**.
- First SystemFunction036: entry1 rekey + immediate 20-byte output.
- Second SystemFunction036: seven zero-byte first-use rekeys for entries2..0, then returns to entry1 and emits the next 20 bytes from the same RC4 stream.
- Added F005: first two SystemFunction036 outputs share one cached ADVAPI RC4-key root; intervening KSecDD transitions #2..#8 are not ancestors of the second output.
- Corrected F004: C1 uses MD4 root D1; C2..C8 repeatedly embed a second digest D2 at 7-byte offsets. Raw blobs confirm the pattern exactly.
- v0.1.2's claim that all eight rekey inputs share one D is **superseded/falsified**.

## v0.1.2 — Superseded exploratory model
- Detected that zero-byte first-use rekey passes exist in source semantics.
- Proposed an eight-rekey-before-first-byte model and one-D C1..C8 family.
- Raw V17 validation in v0.1.3 showed the first call is exceptional because the global NT rekey threshold mutates mid-call. Preserve this version as an audit trail; do not reuse its one-D/eight-before-first-byte conclusions.

## v0.1.1 — ADVAPI first-use reductions
- Added F001: eight rc4_safe structs are not eight hidden initial keys under reference semantics; first use forces rekey.
- Added F002: 20-byte caller-buffer contribution is projected through MD4 to at most 128 bits before rekey.
- Recorded selector order `1,2,3,4,5,6,7,0,...` for the reference user-mode path.

## v0.1.0 — PK-only baseline
- Froze Public-Key-Only invariant and proof ladder.
- Corrected SP1/SP3 FIPS path confusion.
- Fixed SP3 OpenSSL/CryptoAPI request shape at 64 bytes -> RSAENH 40+24.
- Retained positive RSAENH machine-algebra result `|Image(out40)| <= 2^160`.
- Moved main bottleneck to ADVAPI/SystemFunction036/KSecDD effective hidden-state dimension.

## Handoff rule
Every future milestone must:
1. create `versions/vX.Y.Z.md`;
2. update `STATUS.md`;
3. update `CLAIMS.md` when a proposition changes status;
4. create a separate `findings/FNNN-*.md` for any nontrivial machine-level result;
5. explicitly list which older assumption is invalidated, if any.
