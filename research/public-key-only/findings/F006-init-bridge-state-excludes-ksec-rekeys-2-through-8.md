# F006 — RSAENH init→acquire-bridge state excludes KSecDD rekeys #2..#8
Status: COMPOSED FROM TRACE-CONFIRMED COMPONENTS
PK-only effect: removes seven fresh-looking kernel transitions from the ancestry of the early provider state.

## Inputs already established
1. The RSAENH state immediately before the provider-initialization transition is a fixed 20-byte constant left by AlgorithmCheck/self-test:

`S0 = 24 c5 06 c0 44 5b 81 78 6d c9 1c 85 6e 0d f5 c0 7c 88 04 eb`.

2. V17 call stacks identify SystemFunction036 output #1 with the provider-initialization D640 transition (return site `rsaenh+0x120D4`) and output #2 with the subsequent CryptAcquireContext bridge D640 transition (return site `rsaenh+0x0D766`).

3. F005 proves outputs #1 and #2 consume consecutive 20-byte segments of the same ADVAPI entry1 RC4 stream keyed by one KSecDD-derived root `M1`.

4. KSecDD rekeys #2..#8 occur between those useful outputs but initialize other ADVAPI entries; the second useful output returns to entry1.

## Dependency algebra
Let:
- `L1,L2` be the 20-byte SystemFunction036 in/out buffers before RC4 processing;
- `K(M1)[0:20]`, `K(M1)[20:40]` be consecutive entry1 keystream segments;
- `B1,B2` be the rsaenh caller/output-buffer prefixes XORed into the returned SystemFunction bytes before each FIPS block;
- `T(S,A)` be the deterministic rsaenh FIPS transition on state S and finalized aux A.

Then:

`R1 = L1 XOR K(M1)[0:20]`

`R2 = L2 XOR K(M1)[20:40]`

`A1 = R1 XOR B1`

`A2 = R2 XOR B2`

`S1 = T_state(S0,A1)`

`S2 = T_state(S1,A2)`.

Therefore:

`S2 = F(M1,L1,L2,B1,B2)`

for fixed shipped code/constants.

There is **no dependency edge** from KSecDD rekey roots `M2,...,M8` into `S2`.

## Why this is a real state-space reduction
A timeline-based model would see seven kernel reseeds between the two RSAENH transitions and may count their fresh pool/state variables as additional uncertainty in the provider state. The actual data-flow graph proves they are irrelevant to the second transition because ADVAPI caching returns to entry1.

Thus chronology != ancestry. Only machine data-flow ancestors belong in the Public-Key-Only support bound.

## What remains unknown
F006 does not bound the root `M1`, nor the stack/caller prehistories `L1,L2,B1,B2`. Those are now the precise next targets. The seven excluded KSecDD rekeys may matter later when entry2..entry0 are eventually consumed, but not for the early state `S2`.

## Next proof obligation
Recover first-writer provenance for `L1,L2` and rsaenh `B1,B2`. V17 already shows highly structured pre-call L buffers rather than random-looking 160-bit blobs; this observation is not yet a support bound until their writers are identified.