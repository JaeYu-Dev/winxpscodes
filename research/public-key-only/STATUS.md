# STATUS — Public-Key-Only Track

Version: v0.1.0
Date: 2026-08-09

## One-line status
Provider-side RSAENH final generation is largely closed; no provider-layer public-key-only collapse below 128 bits is proven. The highest-value remaining target is the effective state dimension of the ADVAPI `SystemFunction036` / `rc4_safe` / KSecDD chain.

## Closed facts
- OpenSSL 0.9.8h Windows `RAND_poll()` requests 64 bytes from `CryptGenRandom` and mixes them with entropy credit 0.
- XP SP3 RSAENH runtime handles >40-byte requests by generating a new 40-byte block per iteration; 64 bytes therefore use 40+24, not the SP1 source's 20-byte loop model.
- RSAENH checks `SystemFunction036` success; stale-buffer-on-failure hypothesis is falsified.
- Caller-buffer contents are auxiliary XOR material, not the sole randomness source.
- RSAENH final block `H(state20, aux20) -> out40` has been replayed bit-exact in controlled traces.
- A real machine-algebra collapse exists: nominal 320-bit `(state20, aux20)` maps through a 160-bit effective `XVAL`, so one `out40` image is at most 2^160.
- SP1 `SHA_mod_q` source contains obvious defects, but they do not imply a 120-bit fresh-randomness loss; the 15 untouched bytes in the 5-byte-copy branch retain the just-generated `rgdwNewSeed` suffix.

## Public-key-only blockers still alive
1. Persistent KSecDD/registry-derived hidden state has not been bounded below 128 bits.
2. Fresh system-pool contributions have not been proven deterministic functions of public build facts.
3. ADVAPI's 8-entry RC4-safe manager and its first-use/rekey history need an exact shipped-binary quotient-state analysis.
4. OpenSSL's other RAND_poll inputs must ultimately be composed into the first-key search bound; variation must not be confused with independent entropy.

## Priority queue
P0. Exact ADVAPI32 SP3 machine code: `SystemFunction036 -> RandomFillBuffer -> rc4_safe_select -> rekey -> rc4_safe`.
P1. Prove the number of *effective* independent RC4 states reachable before the first Bitcoin key, rather than counting eight structs nominally.
P2. Exact KSecDD rekey input/output dependency graph; look for width loss, overwrite-before-use, state aliasing, constant initialization, or partial rekey.
P3. Compose consecutive `SystemFunction036` outputs into a joint image bound.
P4. Push the bound through RSAENH's XVAL projection and OpenSSL first-key draw.

## Break criterion
A result is only called a Public-Key-Only break if either:
- an enumerable candidate set for the private scalar is provably < 2^128 in the relevant execution class, or
- machine structure yields an algorithm asymptotically/practically below generic secp256k1 ECDLP,
with validation only on synthetic/researcher-owned keys.
