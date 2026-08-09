# Methodology

## Public-Key-Only invariant
The predictor receives only the public key `Q=dG` plus fixed, non-secret implementation/build facts. Runtime secrets are never attack inputs.

## Per-finding template
Every hypothesis must be written as:

1. **Machine fact** — exact instruction/data-flow property.
2. **Reachability** — prove the historical production path reaches it.
3. **Algebra** — normalize instructions into a state transition.
4. **Cardinality/search claim** — state exactly how many effective degrees of freedom disappear, or why search complexity improves.
5. **Composition** — show the reduction survives downstream layers.
6. **Falsifier** — name the observation that would kill the hypothesis.
7. **Verdict** — CONFIRMED/FALSIFIED/OPEN.

## Forbidden shortcuts
- byte variation != entropy
- race != predictability
- short seed != small support
- deterministic != enumerable
- 160-bit state width != 160 bits of min-entropy
- source bug != shipped-binary bug
- correlation != dimensional collapse

## Machine-code flaw scan
For each RNG layer, inspect normal-success paths for:
- dead inputs / optimized-away contributions
- sub-register or width truncation
- signed/unsigned length errors
- aliasing between nominally independent buffers/states
- overwrite-before-use and write-after-check errors
- partial state/key updates
- fixed or repeated initialization
- stale threshold/counter reads that alter rekey cadence
- selector collapse / unreachable entries
- failure paths incorrectly reported as success
- state update ordering that causes reachable-state convergence
- projection into a substantially smaller equivalence class

## Baseline
A simple candidate enumeration must beat generic secp256k1 ECDLP (~2^128 group operations) to qualify as a PK-only computational break. A local dimensional reduction above that threshold remains a useful finding but not a break.

## Experimental ethics
End-to-end recovery is limited to synthetic or researcher-owned keys. Historical public-chain data may be used for aggregate measurement, not target selection or unauthorized recovery.