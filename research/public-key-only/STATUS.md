# STATUS — Public-Key-Only Track

Version: v0.1.1
Date: 2026-08-09

## One-line status
Provider-side RSAENH final generation is largely closed; no public-key-only collapse below 128 bits is proven. New source-level reductions show ADVAPI `rc4_safe` adds no hidden initial RC4-key roots on normal first use, and a 20-byte SystemFunction036 caller-buffer contribution is MD4-projected to at most 128 bits before rekey. The remaining dominant unknown is the shipped-SP3 ADVAPI/KSecDD recurrent state chain.

## Closed facts
- OpenSSL 0.9.8h Windows `RAND_poll()` requests 64 bytes from `CryptGenRandom` and mixes them with entropy credit 0.
- XP SP3 RSAENH runtime handles >40-byte requests by generating a new 40-byte block per iteration; 64 bytes therefore use 40+24, not the SP1 source's 20-byte loop model.
- RSAENH checks `SystemFunction036` success; stale-buffer-on-failure hypothesis is falsified.
- Caller-buffer contents are auxiliary XOR material, not the sole randomness source.
- RSAENH final block `H(state20, aux20) -> out40` has been replayed bit-exact in controlled traces.
- A real machine-algebra collapse exists: nominal 320-bit `(state20, aux20)` maps through a 160-bit effective `XVAL`, so one `out40` image is at most 2^160.
- SP1 `SHA_mod_q` defects do not imply a 120-bit fresh-randomness loss; the untouched suffix already contains current `NewGenRandom` bytes.

## New v0.1.1 reductions
- Reference user-mode `rc4_safe` has 8 entries, but every entry starts with `BytesUsed=0xffffffff`; normal first use forces rekey before RC4 output. Initial allocator residue in the RC4 key structs therefore contributes no independent hidden key under source semantics.
- The sequential user-mode selector order is `1,2,3,4,5,6,7,0,...`, so immediate same-entry reuse is not the default source behavior.
- `RandomFillBuffer()` feeds the caller buffer into a circular hash configured as MD4 before rekey/output. A 20-byte caller buffer therefore contributes through a 16-byte digest: fixed-prior-state image <=2^128 for that contribution.
- High-value OPEN hypothesis: within one RSAENH CGR64 loop, the second `SystemFunction036(aux20,20)` may receive the previous iteration's aux residue rather than a new independent stack-prehistory variable. Exact SP3 first-writer/last-reader SSA must confirm this.

## Public-key-only blockers still alive
1. Persistent KSecDD/registry-derived hidden state has not been bounded below 128 bits.
2. Fresh system-pool contributions have not been proven deterministic functions of public build facts.
3. ADVAPI reference-source reductions still require exact shipped-SP3 machine-code validation.
4. The joint image of consecutive `SystemFunction036` outputs remains unbounded below 128 bits.
5. OpenSSL's other RAND_poll inputs must ultimately be composed into the first-key search bound; variation must not be confused with independent entropy.

## Priority queue
P0. Exact SP3 ADVAPI machine code: circular-hash update -> selector -> threshold -> KSecDD rekey -> selected-entry key -> RC4 output.
P1. Exact RSAENH D640 loop-back SSA: identify the first writer of `aux20` before the second SystemFunction036 call.
P2. KSecDD rekey dependency graph: buffered-I/O input prefix, persistent state, pool, RC4 transform; search width loss/overwrite/alias/partial-update.
P3. Compose all pre-key `SystemFunction036` outputs into a joint image/search bound.
P4. Push the bound through RSAENH XVAL projection and OpenSSL first-key draw.

## Break criterion
A result is only called a Public-Key-Only break if either:
- an enumerable candidate set for the private scalar is provably < 2^128 in the relevant execution class, or
- machine structure yields an algorithm asymptotically/practically below generic secp256k1 ECDLP,
with validation only on synthetic/researcher-owned keys.
