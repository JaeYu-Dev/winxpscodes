# Research Changelog

This file is the fastest handoff path for a new researcher. Read newest version first, then follow referenced findings.

## v0.1.2 — First-use rekey burst correction
- Corrected the early call-count model: reference source can perform eight zero-byte first-use rekeys inside one initial GenRandom request before returning bytes.
- Added F003: zero-byte first-use rekey control flow.
- Added F004: conditional theorem that the first eight user-side KSecDD rekey-input snapshots are functions of one 128-bit `D=MD4(B)` root.
- First SP3 snapshot `C1=MD4(B)||0^240` already bit-exact confirmed; C2..C8 and API-call nesting remain open.
- Earlier notes equating one early SystemFunction036 call with one KSecDD rekey must not be reused without revalidation.

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
