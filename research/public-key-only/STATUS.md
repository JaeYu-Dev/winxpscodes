# STATUS — Reachable Private-Key Image Track

Version: v0.1.7
Date: 2026-08-09

## One-line status
No public-key-only break below 2^128 is proven. The Windows-side model has now collapsed from several apparent KSecDD rekey roots to one persistent 80-byte ratchet `X_{t+1}=VLH(X_t,P_t)`. The main problem is the fresh/irreducible provenance of `X0` and sequential gather inputs `P1..P6`. V24 additionally shows a non-local repeated page+8 kernel-pointer candidate between cycles 2 and 7, but its identity as the 0xE00 KSecDD working allocation and preservation of residual bytes are still OPEN.

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
- Therefore early useful-output ancestry is `[M1,M1,M2,M3]`; under the modeled seven-SystemFunction first-key sequence it is `[M1,M1,M2,M3,M4,M5,M6]`.
- V24 proves `M1..M6` must not be modeled as six independent kernel roots: the full 80-byte `seedbase_after` is handed across adjacent long-path cycles and reused as RC4 KSA key material. The kernel side is one persistent state trajectory plus gather differentials.
- Only the actually used 0x258-byte gather prefix is hashed by the validated long path; the 0xE00 allocation capacity is not itself an entropy/input width.

## New allocator-provenance observation (F009)
Eight V24 post-gather frame captures contain the candidate pointer sequence:

`e1337008, e1379008, e137f008, e138e008, e12cc008, e134d008, e1379008, e137e008`.

Cycle 2 and cycle 7 repeat `0xe1379008` exactly. Every value ends in `0x008`, structurally consistent with an XP x86 page+8 pool payload after an 8-byte pool header. This is a **strong candidate**, not yet a semantic proof that the slot equals `pbWorkingBuffer`.

Do not infer preserved contents or zero fresh entropy from address equality alone.

## Primary research object
For fixed implementation/build class `C`:

`d_first = Phi_C(omega)`

`R_C = Image(Phi_C)`.

The goal is to bound/characterize `R_C` and its search structure. Public-key-only recovery is a corollary only when candidates from `R_C` can be searched below generic secp256k1 ECDLP (~2^128) using only `Q=dG` as verifier.

## Current Windows-side state model
The useful KSecDD/rekey sequence is represented by:

`X_{t+1} = VLH(X_t, P_t)`

`M_t = C_t XOR KS_{X_{t+1}}[0:256]`.

The first-key Windows-side candidate family is therefore controlled by one starting kernel state `X0`, sequential gather inputs `P1..P6`, caller-side startup roots already compressed through MD4/circular-hash structure, and RSAENH caller/local provenance.

Never multiply nominal widths of `M1..M6` or `P1..P6` without a distinct-root proof.

## Current blockers
1. Reachable image/provenance of initial KSecDD state `X0`.
2. Fresh differential dimension of the sequential SP3 gather inputs `P1..P6` over the actually hashed `[0,0x258)` region.
3. Exact semantic identity and last-writer history of V24 allocator/pool pointer candidates; classify residual bytes as `ZERO`, `SELF`, `DERIVED/EXPLICIT`, or `FOREIGN`.
4. Exact first-writer/last-writer provenance of RSAENH/SystemFunction036 pre-call locals `L_t` and caller-prefixes `B_t`.
5. Exact D640 `D681..D693` audit for within-CGR64 second-round local reuse.
6. Full composition of both OpenSSL RAND_poll executions and remaining Win32 sources into the pre-key OpenSSL state.
7. First BN_rand_range draw/rejection-path image.

## Immediate next actions
P0. Prove or falsify that the V24 `...008` frame values are the shipped-SP3 0xE00 `GatherRandomKey` working-buffer/free-argument addresses.
P1. Build the SP3-native write mask over `[0,0x258)` from machine/trace evidence; do not transplant SP1 offsets where SP3 layout differs.
P2. If cycle-2/cycle-7 snapshots can be recovered, compare only unwritten hashed offsets and establish exact last writers.
P3. Bound the surviving `FOREIGN` degrees plus `X0`; SELF/DERIVED repeats contribute no new root.
P4. Close `D681..D693` and L/B last-writer provenance in parallel.
P5. Compose the reduced Windows image into OpenSSL and finally `BN_rand_range`.

## Break criterion
Call it a Public-Key-Only break only when the first private scalar is restricted to an enumerable/structured search provably below generic secp256k1 ECDLP (~2^128), validated on synthetic or researcher-owned keys only.