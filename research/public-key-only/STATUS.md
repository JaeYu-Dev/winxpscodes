# STATUS — Reachable Private-Key Image Track

Version: v0.1.8
Date: 2026-08-09

## One-line status
No public-key-only break below `2^128` is proven. The Windows-side model has collapsed from many apparent RNG roots to one persistent 80-byte KSecDD ratchet plus sequential gather provenance. Recovered later-stage work additionally removes the second RSAENH CGR64 stack-prehistory root, fixes the direct VLH input width at 600 bytes, narrows the pinned pre-QSI direct allocation-history region to 24 bytes, and confirms a class-05 3528-byte -> SHA-1 20-byte projection. The decisive blocker remains the reachable/fresh dimension of `X0` and the actual `P_t` ancestry.

## Canonical handoff
Read `HANDOFF.md` first. It is the authoritative successor entry point. Chat transcripts are not required to continue the project.

## Strongest confirmed reductions
- OpenSSL 0.9.8h Windows RAND_poll requests 64 bytes from CryptGenRandom.
- XP SP3 RSAENH serves the relevant request as 40+24 from two 40-byte generator blocks.
- RSAENH final block is bit-exact replayed; one `out40` block has `|Image(out40)| <= 2^160` through the 160-bit XVAL projection.
- RSAENH state immediately before provider initialization is the fixed AlgorithmCheck/self-test 20-byte expected vector.
- Within one 64-byte `D640` invocation, the second `SystemFunction036` call reuses the same `[ebp-0x18]` local; the first round has already determined that local, so the second round adds **no independent 160-bit stack-prehistory root**.
- V17 first KSecDD rekey input is bit-exact `MD4(L1)||0^240`.
- V17 C2..C8 are repeated embeddings of one `MD4(L2)` root at successive +7 byte positions.
- V17 PRGA #1 and #9 use the same ADVAPI entry1 state and continue indices `0->20->40`, matching SystemFunction036 outputs #1 and #2.
- Early useful-output ancestry is `[M1,M1,M2,M3]`; under the modeled seven-SystemFunction first-key sequence it is `[M1,M1,M2,M3,M4,M5,M6]`.
- V24 proves `M1..M6` must not be modeled as six independent kernel roots: they descend from one persistent KSecDD 80-byte state trajectory.
- Target replay validates the direct VLH pool as `0x258 = 600` bytes; the full `0xE00` allocation capacity is **not** the hashed-input width.
- In the pinned long-gather campaign, the direct allocation-history region before the first QSI contribution has been narrowed to **24 bytes**. This is an unresolved-provenance width, not an entropy estimate.
- Class-05 `SystemProcessInformation` contributes a captured `0xdc8 = 3528` byte region that is SHA-1 projected to one 20-byte value before downstream use; therefore that local boundary has image <= `2^160`.

## Important open refinements
- The captured class-05 buffer begins with a valid `0x1b8`-byte Idle-process record (`NextEntryOffset=0x1b8`, four threads, PID 0), and bytes after it do not parse as the next ordinary SPI header. However, the stronger claim that the rest of the 3528-byte region is untouched pool tail is **OPEN** pending exact XP SP3 `ExpGetProcessInformation` cold-block closure.
- The 24-byte direct allocation-history region is not yet classified byte-for-byte as `CONST | SELF | DERIVED | DELTA | FOREIGN`.
- The registry Seed exists/read path does not universally collapse to zero state; the missing-Seed branch is only one reachable branch. The reachable family of `X0` remains the largest universal blocker.

## Primary research object
For fixed implementation/build class `C`:

`d_first = Phi_C(omega)`

`R_C = Image(Phi_C)`.

The goal is to bound/characterize `R_C` and its search structure. Public-key-only recovery is a corollary only when candidates from `R_C` can be searched below generic secp256k1 ECDLP (~`2^128`) using only `Q=dG` as verifier.

## Current Windows-side state model
The useful KSecDD/rekey sequence is represented by:

`X_{t+1} = VLH(X_t, P_t)`

`M_t = deterministic RC4 transform keyed by X_{t+1}`.

The first-key Windows-side family is controlled by one starting kernel state `X0`, sequential gather inputs `P_t`, reduced caller-side MD4/circular-hash roots, and RSAENH caller/output-history variables. Never multiply nominal widths of `M_t` or chronological RNG events without a distinct-root proof.

## Current blockers — priority order
1. **P1: class-05 partial-write semantics.** Complete exact XP SP3 `ExpGetProcessInformation` hot + cold blocks and determine byte-for-byte which part of the 3528-byte buffer is actually written on the buffer-too-small path.
2. **P2: 24-byte provenance.** Recover last writers of the direct pre-QSI 24-byte region and classify each byte `CONST | SELF | DERIVED | DELTA | FOREIGN`.
3. **P3: persistent Seed / X0 image.** Bound the reachable family of the 80-byte initial KSecDD state under real setup/boot/persistence transitions.
4. **P4: OpenSSL composition.** Propagate only surviving independent variables through both RAND_poll paths and first `BN_rand_range()` draw; keep modal and all-reachable rejection paths distinct.

## No-go / superseded paths
- SP1 `SHA_mod_q` 5-byte copy => target-SP3 120-bit collapse: false.
- `SystemFunction036` failure => stale local bytes: false on validated provider path.
- Eight ADVAPI entries => eight arbitrary secret keys: false.
- `M1..M6` => six independent kernel roots: false.
- Full 0xE00 allocation => VLH entropy/input width: false; validated direct width is 0x258.
- “Stack garbage is constant because XP lacks ASLR”: not an acceptable theorem; use last-writer analysis.
- OpenSSL universally compresses the entire Windows history to one 160-bit digest before the scalar: false.

## Break criterion
Call it a Public-Key-Only break only when the first private scalar is restricted to an enumerable/structured search provably below generic secp256k1 ECDLP (~`2^128`), validated on synthetic or researcher-owned keys only.
