# STATUS — Public-Key-Only Track

Version: v0.1.3
Date: 2026-08-09

## One-line status
No public-key-only break below 2^128 is proven. A new shipped-SP3 startup transient is now trace-grounded: NT detection changes ADVAPI's RC4 rekey threshold from 512 to 16384 during the first rekey, causing the first two observed SystemFunction036 outputs to consume consecutive 20-byte segments of the **same cached entry1 RC4 stream**. This removes one expected independent OS-RNG output root from the early rsaenh provider-state history.

## Strongest confirmed reductions
- OpenSSL 0.9.8h Windows RAND_poll requests 64 bytes from CryptGenRandom.
- XP SP3 RSAENH serves the relevant request as 40+24 from two 40-byte generator blocks.
- RSAENH final block `H(state20,aux20)->out40` is bit-exact replayed.
- One out40 block projects nominal `(state20,aux20)` through an effective 160-bit XVAL, so `|Image(out40)| <= 2^160`.
- ADVAPI has eight rc4_safe entries; hidden arbitrary initial RC4 keys are overwritten by first-use KSecDD-derived rekeys.
- V17 first KSecDD rekey input is bit-exact `MD4(B1)||0^240`.
- V17 raw C2..C8 blobs are successive XOR additions of one common `D2=MD4(B2)` at offsets 7,14,21,28,35,42,49.
- Most importantly, V17 PRGA #1 and #9 use the same RC4 state pointer and continue indices `0->20->40`, matching SystemFunction036 outputs #1 and #2 respectively.

## v0.1.3 startup mechanism
Initial global rekey threshold: 512.

First SystemFunction036:
1. select entry1 with `BytesUsed=0xffffffff`;
2. local bytes-used captures 512;
3. KSecDD rekey path reaches NT detection and changes global threshold to 16384;
4. post-rekey available bytes become `16384-512`, so 20 bytes are emitted from entry1.

Second SystemFunction036:
1. global threshold is already 16384;
2. entries2..0 each first-use rekey with local=16384, producing seven zero-byte PRGA passes;
3. loop returns to entry1;
4. entry1 emits its next 20 bytes, continuing the same stream used by call #1.

Therefore the seven intervening KSecDD rekeys initialize other entries but are not data-flow ancestors of SystemFunction036 output #2.

## Corrected circular-input model
- C1 depends on `D1=MD4(B1)`.
- C2..C8 depend on repeated embeddings of `D2=MD4(B2)`.
- Complete C1..C8 caller-side family uses at most two 128-bit digest roots in the V17 trace, not eight independent 256-byte values.
- The v0.1.2 one-D-for-all-eight hypothesis is superseded.

## Public-key-only blockers
1. KSecDD material M1 that keys entry1 still depends on persistent kernel state/pool data.
2. SystemFunction036 in/out prehistory buffers B1/B2 (and later buffers) still need exact machine-code provenance bounds.
3. Later cached ADVAPI entries (entry2, entry3, ...) depend on KSecDD rekeys #2,#3,...; their joint kernel-state ancestry remains potentially large.
4. Need compose the same-stream R1/R2 relation through rsaenh provider initialization + acquire bridge into the state used by runtime CryptGenRandom.
5. OpenSSL's other RAND_poll sources remain for final first-private-scalar composition.

## Immediate next actions
P0. Compose F005 through rsaenh init and acquire-bridge FIPS transitions; derive provider state after both as a function of one RC4 root M1 plus buffer prehistories.
P1. Trace first writers of the two SystemFunction036 20-byte input/output locals B1/B2 in exact RSAENH/ADVAPI machine code; test whether either is fixed, reused, or a deterministic function of known stack/global values.
P2. Map SystemFunction036 outputs #3 onward to cached entry2/entry3 KSecDD roots and exact Bitcoin/OpenSSL call sequence.
P3. Continue KSecDD pool provenance analysis only for kernel states that actually lie on the first-key dependency graph; discard transitions proven irrelevant by caching.
P4. Push all confirmed reductions through OpenSSL and evaluate the joint search/image bound against 2^128.

## Break criterion
Call it a Public-Key-Only break only when the first private scalar is restricted to an enumerable/structured search provably below generic secp256k1 ECDLP (~2^128), validated on synthetic or researcher-owned keys only.
