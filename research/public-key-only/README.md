# Early Bitcoin / Windows XP Private-Key Reachability Research

## Start here

**Successor researchers must read `HANDOFF.md` first.** It is the canonical entry point and is written so that prior ChatGPT transcripts are not required to continue the work.

Recommended order:

1. `HANDOFF.md`
2. `STATUS.md`
3. `CLAIMS.md`
4. `findings/`
5. `versions/`
6. `CHANGELOG.md`
7. `METHODOLOGY.md`

## Primary objective
Characterize the **reachable private-key image** of the exact shipped historical key-generation stack.

For a fixed implementation/build class `C`, let `omega` denote all hidden execution state admitted by that class and define:

`d_first = Phi_C(omega)`.

The main research object is:

`R_C = Image(Phi_C)`.

We seek rigorous upper bounds and structure for `R_C`, plus the computational cost of enumerating/searching it. We do **not** estimate security by summing nominal buffer widths, entropy-credit counters, or chronological RNG events.

## Public-key-only corollary / attack model
Public-key-only recovery is the final success condition, not the primary object of study. Given only `Q=dG` and fixed public implementation/build facts, if the implementation restricts `d` to an efficiently searchable set/structure below generic secp256k1 ECDLP (~2^128 group operations), then `Q` acts only as a verifier for candidates generated from the reachable image.

No private runtime state, registry dump, process memory, boot-time guess, PID guess, VM-image assumption, or wallet-side secret may be supplied to the predictor. Environmental observations may be used only to formulate/falsify hypotheses, not as attack inputs.

## Evidence ladder
1. FIELD — historical observations; hypothesis generation only.
2. SOURCE — leaked/reference source; semantics only, never shipped-binary proof.
3. BINARY — exact shipped machine-code fact.
4. TRACE — controlled execution reaches the binary path.
5. REPLAY — transformation reproduced bit-for-bit.
6. BOUND — image/support/search-complexity theorem.
7. PKO-PoC — synthetic/owned key recovery using public key only.

A higher evidence layer may falsify a lower-layer hypothesis.

## Current target stack
- Bitcoin 0.1.x / historical Windows build
- OpenSSL 0.9.8h
- Windows XP SP3 RSAENH 5.1.2600.5507
- ADVAPI32 `SystemFunction036` / `RtlGenRandom`
- KSecDD / randlib / persistent RNG state

## Current strongest reductions
- RSAENH pre-initialization provider state is a fixed self-test constant, not an independent 160-bit root.
- One RSAENH 40-byte block has `|Image(OUT40)| <= 2^160` through its machine-level XVAL projection.
- A 64-byte RSAENH request uses two provider transitions, but the second transition reuses the same `[ebp-0x18]` local and does **not** add a second independent 160-bit stack-prehistory root.
- The first two observed SystemFunction036 outputs are consecutive segments of the same cached ADVAPI entry1 RC4 stream.
- Early useful-output ancestry is `[M1,M1,M2,M3]`; under the modeled seven-call first-key path, consumed KSecDD materials are `[M1,M1,M2,M3,M4,M5,M6]`.
- `M1..M6` are not six independent secrets; they descend from one persistent 80-byte KSecDD ratchet `X_{t+1}=VLH(X_t,P_t)`.
- The validated direct KSecDD/VLH input width is `0x258 = 600` bytes, not the full `0xE00` allocation capacity.
- Later recovered work narrows the direct pre-QSI allocation-history region in the pinned long-gather campaign to 24 bytes; this is a provenance bound, not an entropy-bit claim.
- Class-05 `SystemProcessInformation` contributes a 3528-byte captured region that is SHA-1 projected to one 20-byte value at the downstream boundary, so that local image is at most `2^160`.

These are real dependency/image reductions but **not yet a key-recovery break**.

## Current bottleneck
The decisive remaining question is the reachable/fresh dimension of the persistent KSecDD trajectory: the ancestry of initial state `X0` plus exact provenance of sequential gather inputs `P_t`. A simple Public-Key-Only enumeration route fails if an irreducible >=128-bit hidden root survives the actual first-key dependency graph.

## Repository map
- `HANDOFF.md` — canonical successor handoff; read first
- `STATUS.md` — one-page current state
- `CLAIMS.md` — CONFIRMED / FALSIFIED / OPEN ledger
- `METHODOLOGY.md` — proof rules and PK-only gate
- `versions/` — immutable milestone notes
- `findings/` — one machine-level finding per file
- `CHANGELOG.md` — version transition history

## Safety/reproducibility scope
All end-to-end recovery validation must use synthetic or researcher-owned keys. No live-wallet targeting, balance-driven candidate selection, or third-party fund movement belongs in this project.
