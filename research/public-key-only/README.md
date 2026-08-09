# Public-Key-Only Early Bitcoin / Windows XP RNG Research

## Objective
Determine whether the exact shipped 2009-era Windows Bitcoin key-generation stack admits a **public-key-only computational collapse**: given only the public key and fixed implementation/build facts, does the implementation restrict the private scalar to a candidate space or structured search with cost below generic secp256k1 ECDLP (~2^128 group operations)?

## Non-negotiable invariant
No private runtime state, registry dump, process memory, boot-time guess, PID guess, VM-image assumption, or wallet-side secret may be supplied to the predictor. Environmental observations may be used only to formulate/falsify hypotheses, not as attack inputs.

## Evidence ladder
1. FIELD — historical observations; hypothesis generation only.
2. SOURCE — leaked/reference source; semantics only, never shipped-binary proof.
3. BINARY — exact shipped machine-code fact.
4. TRACE — controlled execution reaches the binary path.
5. REPLAY — transformation reproduced bit-for-bit.
6. BOUND — image/support/search-complexity theorem.
7. PKO-PoC — synthetic/owned key recovery using public key only.

A claim may only move upward; a higher layer can falsify a lower-layer hypothesis.

## Current target stack
- Bitcoin 0.1.x / historical Windows build
- OpenSSL 0.9.8h
- Windows XP SP3 RSAENH 5.1.2600.5507
- ADVAPI32 `SystemFunction036` / `RtlGenRandom`
- KSecDD / randlib / persistent RNG state

## Current strongest positive result
The shipped RSAENH FIPS-186-style 40-byte generator is machine-algebraically lower-dimensional than its nominal `(state20, aux20)` 320-bit input: the observed block is determined through a 160-bit `XVAL` combination, hence `|Image(OUT40)| <= 2^160`. This is a real dimensional collapse but **not yet a public-key-only break** because 2^160 enumeration is worse than the ~2^128 generic ECDLP baseline.

## Current bottleneck
Reduce the effective hidden state of the `SystemFunction036 -> ADVAPI rc4_safe -> KSecDD persistent-state` chain, and then compose that bound through OpenSSL to the first Bitcoin private scalar.

## Repository map
- `STATUS.md` — one-page handoff state
- `CLAIMS.md` — CONFIRMED / FALSIFIED / OPEN ledger
- `METHODOLOGY.md` — proof rules and PK-only gate
- `versions/` — immutable milestone notes
- `findings/` — one machine-level finding per file

## Safety/reproducibility scope
All end-to-end recovery validation must use synthetic or researcher-owned keys. No live-wallet targeting, balance-driven candidate selection, or third-party fund movement belongs in this project.