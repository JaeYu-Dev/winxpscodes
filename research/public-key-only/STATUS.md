# STATUS — Reachable Private-Key Image Track

Version: v0.1.5
Date: 2026-08-09

## One-line status
No public-key-only break below 2^128 is proven. The project is now framed around the reachable first-private-scalar image `R_C = Image(Phi_C)`. A new SP3 trace/KSA result maps the first four SystemFunction036 outputs to KSecDD-derived ADVAPI RC4 roots `[M1,M1,M2,M3]`; composing the standard seven-call first-key path yields only six consumed KSecDD roots `[M1,M1,M2,M3,M4,M5,M6]`, not eight chronological rekeys.

## Strongest confirmed reductions
- OpenSSL 0.9.8h Windows RAND_poll requests 64 bytes from CryptGenRandom.
- XP SP3 RSAENH serves the relevant request as 40+24 from two 40-byte generator blocks.
- RSAENH final block `H(state20,aux20)->out40` is bit-exact replayed.
- One out40 block projects nominal `(state20,aux20)` through an effective 160-bit XVAL, so `|Image(out40)| <= 2^160`.
- RSAENH state immediately before provider initialization is the fixed AlgorithmCheck/self-test 20-byte expected vector.
- V17 first KSecDD rekey input is bit-exact `MD4(L1)||0^240`.
- V17 C2..C8 are repeated embeddings of one `MD4(L2)` root at successive +7 byte positions.
- V17 PRGA #1 and #9 use the same ADVAPI entry1 state and continue indices `0->20->40`, matching SystemFunction036 outputs #1 and #2.
- V17 KSA replay maps PRGA #10 to KSecDD/IOCTL #2 and PRGA #11 to #3, matching SystemFunction036 outputs #3 and #4.
- Therefore early output-root ancestry is `[M1,M1,M2,M3]`.
- Under the standard first-key seven-SystemFunction call sequence, selector continuation yields `[M1,M1,M2,M3,M4,M5,M6]`; `M7,M8` are created but are not first-key output ancestors on this path.

## Primary research object
For fixed implementation/build class `C`:

`d_first = Phi_C(omega)`

`R_C = Image(Phi_C)`.

The goal is to bound/characterize `R_C` and its search structure. Public-key-only recovery is a corollary when candidates from `R_C` can be searched below generic secp256k1 ECDLP (~2^128) using only `Q=dG` as verifier.

## Current Windows-side ancestry model
The early provider state begins from fixed `S0`.

First two SystemFunction outputs share one cached RC4-key root `M1`:

`R1 = L1 XOR KS(M1)[0:20]`

`R2 = L2 XOR KS(M1)[20:40]`.

Later useful outputs advance through already-keyed entries:

`R3 <- M2`, `R4 <- M3`, and under the modeled first-key sequence `R5 <- M4`, `R6 <- M5`, `R7 <- M6`.

Do **not** treat `M1..M6` as six independent secrets. Their joint ancestry passes through the same KSecDD persistent-state ratchet and may collapse further.

## Current blockers
1. Joint reachable image/provenance of KSecDD-derived roots `M1..M6`.
2. Exact first-writer/last-writer provenance of RSAENH/SystemFunction036 pre-call locals `L_t` and caller-prefixes `B_t`.
3. Exact D640 `D681..D693` audit for within-CGR64 second-round local reuse.
4. Full composition of both OpenSSL RAND_poll executions and remaining Win32 sources into the pre-key OpenSSL state.
5. First BN_rand_range draw/rejection-path image.

## Immediate next actions
P0. Treat `M1..M6` as outputs of one KSecDD state machine and derive a joint-image bound; never multiply six nominal key widths.
P1. Close `D681..D693`: prove or falsify that the second RSAENH CGR64 round adds zero fresh stack-prehistory freedom.
P2. Recover byte-level last writers for V17 `L1/L2` and the RSAENH caller prefixes mixed into aux20.
P3. Compose the resulting Windows output image into OpenSSL RAND_poll state transitions.
P4. Evaluate `log2 |R_C|` or a stronger structured-search bound at the first private scalar.

## Break criterion
Call it a Public-Key-Only break only when the first private scalar is restricted to an enumerable/structured search provably below generic secp256k1 ECDLP (~2^128), validated on synthetic or researcher-owned keys only.
