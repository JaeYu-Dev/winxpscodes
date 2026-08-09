# STATUS — Public-Key-Only Track

Version: v0.1.4
Date: 2026-08-09

## One-line status
No public-key-only break below 2^128 is proven. The early RSAENH provider state is now pruned more aggressively: its pre-init 160-bit state is a fixed self-test constant, and the first two provider RNG consumptions use consecutive segments of one ADVAPI entry1 RC4 stream. Therefore KSecDD rekeys #2..#8, although chronologically interposed, are not ancestors of the RSAENH state after init + acquire bridge.

## Strongest confirmed reductions
- OpenSSL 0.9.8h Windows RAND_poll requests 64 bytes from CryptGenRandom.
- XP SP3 RSAENH serves the relevant request as 40+24 from two 40-byte generator blocks.
- RSAENH final block `H(state20,aux20)->out40` is bit-exact replayed.
- One out40 block projects nominal `(state20,aux20)` through an effective 160-bit XVAL, so `|Image(out40)| <= 2^160`.
- RSAENH state immediately before the provider-initialization transition is the fixed 20-byte AlgorithmCheck/self-test expected vector, not an independent hidden 160-bit root.
- V17 first KSecDD rekey input is bit-exact `MD4(L1)||0^240`.
- V17 C2..C8 are repeated embeddings of one `MD4(L2)` root at successive +7 byte positions.
- V17 PRGA #1 and #9 use the same ADVAPI entry1 RC4 state and continue indices `0->20->40`, matching SystemFunction036 outputs #1 and #2.
- Therefore KSecDD rekeys #2..#8 do not influence SystemFunction036 output #2 and do not enter the RSAENH init→bridge state ancestry.

## Current early-state equation
Let `M1` be the KSecDD-derived material that keys ADVAPI entry1; `L1,L2` the two SystemFunction036 pre-call in/out buffers; and `B1,B2` the rsaenh caller-buffer prefixes mixed after SystemFunction036. With fixed initial RSAENH state S0:

`R1 = L1 XOR KS(M1)[0:20]`

`R2 = L2 XOR KS(M1)[20:40]`

`A1 = R1 XOR B1`

`A2 = R2 XOR B2`

`S2 = T_state(T_state(S0,A1),A2)`.

Hence `S2 = F(M1,L1,L2,B1,B2)` and contains no dependency on KSecDD rekey roots M2..M8.

## Structured prehistory evidence
V17 raw SystemFunction036 pre-call buffers are highly structured rather than random-looking:

L1 = `98 19 03 68 00 00 00 00 00 00 00 68 00 ae 25 00 00 00 00 00`

L2 = `10 00 00 00 00 00 00 00 70 4b 25 00 40 fb 23 00 08 00 00 00`

L2 parses as 32-bit words `0x10, 0, 0x00254b70, 0x0023fb40, 0x8`. This is promising stale-frame structure, but no support bound is claimed until exact writers are recovered.

## Public-key-only blockers
1. KSecDD material M1 still depends on persistent kernel state/pool data.
2. Exact first-writer provenance of L1,L2 and rsaenh prefixes B1,B2 remains open.
3. Later cached ADVAPI entries actually consumed before the first Bitcoin key still introduce KSecDD roots that must be mapped by data-flow ancestry, not chronology.
4. OpenSSL's remaining RAND_poll inputs must ultimately be composed into the first-private-scalar bound.

## Immediate next actions
P0. Recover exact previous writers of all bytes in L1 and L2; test whether their support is determined by fixed DLL addresses, stack-frame geometry, heap pointers, small counters, or genuinely irreducible data.
P1. Recover B1/B2 rsaenh caller-prefix provenance for init and acquire-bridge transitions.
P2. Map SystemFunction036 outputs #3 onward to actual cached ADVAPI entries and only retain KSecDD rekeys that feed the first-key dependency graph.
P3. Bound M1 by tracing only the first KSecDD rekey's persistent-state and pool ancestry.
P4. Push confirmed reductions through both OpenSSL RAND_poll executions and the first BN_rand_range draw.

## Break criterion
Call it a Public-Key-Only break only when the first private scalar is restricted to an enumerable/structured search provably below generic secp256k1 ECDLP (~2^128), validated on synthetic or researcher-owned keys only.
