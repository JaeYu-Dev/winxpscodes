# Early Bitcoin / Windows XP Private-Key Reachability Research

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
- The first two observed SystemFunction036 outputs are consecutive segments of the same cached ADVAPI entry1 RC4 stream.
- V17 KSA replay maps the next useful outputs to KSecDD rekey roots M2 and M3.
- Under the standard first-key seven-call sequence, the Windows output ancestry is `[M1,M1,M2,M3,M4,M5,M6]`: six KSecDD rekey roots, not eight chronological rekeys.

These are real dependency/image reductions but **not yet a key-recovery break**. The joint reachable set of `M1..M6` and the OpenSSL state composition remain open.

## Current bottleneck
Determine the smallest effective historical root that generates KSecDD-derived rekey materials `M1..M6`, then compose only the roots that actually survive the machine dependency graph through RSAENH, both OpenSSL RAND_poll executions, and the first BN_rand_range draw.

## Repository map
- `STATUS.md` — one-page handoff state
- `CLAIMS.md` — CONFIRMED / FALSIFIED / OPEN ledger
- `METHODOLOGY.md` — proof rules and PK-only gate
- `versions/` — immutable milestone notes
- `findings/` — one machine-level finding per file

## Safety/reproducibility scope
All end-to-end recovery validation must use synthetic or researcher-owned keys. No live-wallet targeting, balance-driven candidate selection, or third-party fund movement belongs in this project.
