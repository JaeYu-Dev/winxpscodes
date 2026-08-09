# F008 — KSecDD rekey materials are observations of one persistent 80-byte ratchet

Status: CONFIRMED SP3 TRACE + REPLAY + REFERENCE-SEMANTICS CONSISTENCY
Version: v0.1.6
Date: 2026-08-09

## Statement
The KSecDD-derived ADVAPI rekey materials consumed before the first modeled Bitcoin private key are **not independent random roots**. In the validated XP SP3 long-path cycles, one 80-byte live kernel state is updated and handed directly across adjacent cycles.

Let:
- `X_t` be the 80-byte live KSecDD/randlib seedbase before cycle `t`;
- `P_t` be the gathered long-path working input mixed by VeryLargeHashUpdate in cycle `t`;
- `C_t` be the METHOD_BUFFERED/caller-side 256-byte rekey buffer entering the final RC4 transform;
- `M_t` be the 256-byte rekey material returned toward ADVAPI.

The target-path state machine is:

`X_{t+1} = VLH(X_t, P_t)`

`M_t = C_t XOR KS_{X_{t+1}}[0:256]`.

Here `KS_K` denotes the RC4 keystream after KSA with the full 80-byte key `K`.

## Exact SP3 state handoff evidence
V24 binary artifacts show the full 80-byte `seedbase_after` of a cycle is reused verbatim as:
1. the cycle's second RC4 KSA key, and
2. the next cycle's first RC4 KSA key.

Representative exact checks:

- `seedbase_after #1 == cycle1 second RC4 keybuf[0:80] == cycle2 first RC4 keybuf[0:80]` (80/80 bytes exact);
- `seedbase_after #2 == cycle2 second RC4 keybuf[0:80] == cycle3 first RC4 keybuf[0:80]` (80/80 bytes exact).

The V24 validator independently reports 16/16 RC4 KSA matches, 16/16 PRGA XOR matches, and 8/8 KSecDD after/pre transport matches over eight cycles.

## Reference-source consistency
The XP reference randlib path performs the same logical recurrence:

1. copy current `g_VeryLargeHash` to local state;
2. build an old RC4 key from that state;
3. `VeryLargeHashUpdate(...)` to obtain the next 80-byte state;
4. publish the new state back to `g_VeryLargeHash`;
5. build a new RC4 key from the full new 80-byte state;
6. RC4-transform the returned random-key buffer.

The source is used only as semantic corroboration; the shipped-SP3 claim is anchored by V24 trace/replay artifacts.

## Composition with F007 and the caller circular state
F007 established that the seven modeled pre-first-key SystemFunction036 outputs consume ADVAPI roots:

`[M1, M1, M2, M3, M4, M5, M6]`.

V17 also established that the caller-side KSecDD rekey input buffers `C1..C8` are generated from at most two MD4 digest roots `D1,D2` in that startup sequence.

Therefore the Windows-side rekey vector may be represented as one deterministic transducer:

`(M1,...,M6) = Psi(X0, P1,...,P6, D1, D2)`.

This replaces the invalid model of six independent KSecDD secrets plus six independent 256-byte caller buffers.

## Reachability consequence
The relevant image is now controlled by:
- one initial persistent-state root `X0`;
- the **fresh differential content** of sequential gathers `P1..P6`;
- two caller-side 128-bit MD4 roots `D1,D2` (before further provenance reduction).

No entropy/search bound follows merely from `|X0|=80 bytes`; state width is not entropy. Likewise, `P1..P6` must not be counted as independent full-width buffers without a machine-level provenance proof.

## Falsifier / scope boundary
This finding would need revision if a target SP3 path is shown to replace `g_VeryLargeHash` with a separate unrelated state between validated long-path cycles, or if the modeled first-key path consumes rekey material outside this ratchet. The exact V24 cycle handoffs remain valid for the captured execution.

## Next proof obligation
Classify every byte/field entering `P1..P6` as one of:
- `INVARIANT`: same underlying value/root across cycles;
- `DERIVED`: deterministic function of prior state/counters;
- `DELTA`: changing with a bounded machine-level domain;
- `FOREIGN`: an irreducible external unknown.

The Public-Key-Only / reachable-image route succeeds only if the surviving joint root/delta image becomes searchable below generic secp256k1 ECDLP.
