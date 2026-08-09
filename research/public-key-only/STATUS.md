# STATUS — Public-Key-Only Track

Version: v0.1.2
Date: 2026-08-09

## One-line status
No public-key-only break below 2^128 is proven. The most important new correction is that reference ADVAPI randlib can perform an **eight-entry KSecDD rekey initialization burst before the first returned RNG byte**, not one rekey per early SystemFunction036 call. Under that control flow, the entire user-side first-eight rekey-input family is determined by one 128-bit `D=MD4(B)` root; the first SP3 snapshot is already bit-exact confirmed, while C2..C8 and exact call nesting remain open.

## Closed / strongly anchored facts
- OpenSSL 0.9.8h Windows RAND_poll requests 64 bytes from CryptGenRandom.
- XP SP3 RSAENH serves the relevant 64-byte request as 40+24 from two 40-byte generator blocks.
- RSAENH checks SystemFunction036 success; stale-on-failure is falsified.
- RSAENH final block `H(state20,aux20)->out40` is bit-exact replayed for controlled traces.
- One out40 block is determined through a 160-bit effective XVAL, so its image is at most 2^160.
- Reference user-mode rc4_safe has eight entries, selector order `1,2,3,4,5,6,7,0,...`, and all entries start `BytesUsed=0xffffffff`.
- Reference RandomFillBuffer hashes the caller buffer through MD4 into a 256-byte circular state before rekey/output.
- V17 SP3 first rekey input is bit-exact `C1=MD4(B)||0^240` for the captured 20-byte pre-call buffer B.
- Existing V17/V20 notes report first-eight KSecDD outputs initializing the eight ADVAPI RC4 states and later PRGA reuse; this is consistent with the burst model.

## v0.1.2 model correction
Reference source does:
1. select a first-use entry (`BytesUsed=0xffffffff`),
2. rekey it,
3. set **local** `RC4BytesUsed` to the threshold,
4. compute max output as threshold minus that local value = 0,
5. emit zero bytes,
6. GenRandom loops and selects the next entry.

Therefore one initial non-empty GenRandom request can source-semantically execute eight zero-byte rekey passes before a ninth pass emits bytes. Previous notes that implicitly equated an early SystemFunction036 call with one KSecDD rekey are **suspended pending exact SP3 call-boundary validation**.

## Conditional first-use theorem
If the first-use burst holds in shipped SP3, the initial circular state is zero, and no concurrent update interleaves, then the unchanged caller buffer B is repeatedly MD4-hashed during the eight zero-byte passes. With `D=MD4(B)` and 7-byte circular increments:

`C_j = C_{j-1} XOR Embed_{7(j-1)}(D)`, j=1..8.

Thus `|(C1,...,C8) image| <= 2^128` for the **user-side ADVAPI contribution**. This does not bound the complete RNG output because KSecDD state/pools remain.

## Public-key-only blockers
1. Exact shipped-SP3 validation of the zero-byte first-use branch and API-call nesting.
2. V17 C2..C8 rekey-input replay from the single D root.
3. KSecDD recurrent state `X_j=F(X_{j-1},P_j)` and fresh pool innovations P1..P8 remain potentially large.
4. The joint image of the SystemFunction036 outputs consumed by RSAENH has not been bounded below 2^128.
5. OpenSSL's remaining RAND_poll inputs must eventually be composed into the first-private-scalar bound.

## Immediate next actions
P0. Recover V17 IOCTL input blobs C2..C8 and call nesting; compare with one-D circular replay.
P1. Validate exact SP3 ADVAPI assembly for post-rekey local BytesUsed assignment and zero-byte return behavior.
P2. Rebuild the true KSecDD transition count before first Bitcoin key using actual call boundaries.
P3. Analyze P1..P8 kernel-pool write provenance for repeated/derived fields, width loss, uninitialized gaps, overwrite-before-use, or machine-level aliasing—without counting mere variation as entropy.
P4. Compose confirmed reductions through RSAENH XVAL and OpenSSL.

## Break criterion
Call it a Public-Key-Only break only when the first private scalar is restricted to an enumerable/structured search provably below generic secp256k1 ECDLP (~2^128), validated on synthetic or researcher-owned keys only.
