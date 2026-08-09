# Canonical Handoff — Early Bitcoin / Windows XP Reachable Private-Key Research

This file is the **single entry point for a successor researcher**. Do not rely on prior chat transcripts. Read this file first, then `STATUS.md`, `CLAIMS.md`, `ARTIFACTS.md`, the numbered findings, and finally the version notes.

## 0. Research objective

Primary object of study:

```text
Bitcoin early key generation
  -> OpenSSL 0.9.8h RAND state
  -> CryptoAPI / RSAENH
  -> ADVAPI SystemFunction036 / randlib
  -> KSecDD persistent RNG state
  -> first secp256k1 private scalar d_first
```

The research target is **not** vague entropy estimation. It is the reachable image

```text
R_XP = { d_first produced by the exact historical implementation over all reachable hidden states }
```

and its searchability. A public-key-only recovery result follows only if the reachable image (or a structured search over it) beats generic secp256k1 discrete-log work. Local <=2^160 projections are important reductions, but are not themselves a key break.

## 1. Evidence rule

Every claim must be one of:

- `CONFIRMED`: target-SP3 binary/trace/replay or exact composition proof.
- `FALSIFIED`: contradicted by target evidence.
- `SOURCE-ONLY`: historical source fact, not target-SP3 proof.
- `OPEN`: unresolved proof obligation.
- `SUPERSEDED`: an earlier model retained for auditability but replaced by stronger evidence.

Never infer SP3 machine behavior from the leaked SP1 source alone.

For every new finding record:

1. exact artifact/build,
2. machine/source observation,
3. use-def / state ancestry,
4. algebraic reduction,
5. reachable-image consequence,
6. falsifier / remaining proof obligation.

`ARTIFACTS.md` is the reproducibility index. Pin exact replay commit/path or binary hash there or in the finding before promoting a new machine-level claim.

## 2. Current highest-confidence model

### RSAENH

- Runtime provider state is a process-global 20-byte state.
- The initial provider state before the first relevant runtime transition is a fixed self-test/AlgorithmCheck-derived vector, not a free 160-bit root.
- A provider 40-byte output block is determined through a 160-bit `XVAL`; therefore the image of one `out40` block is <=2^160.
- A 64-byte CryptoAPI request consumes two 40-byte provider transitions (40 + 24 bytes copied).
- The second transition in the same `D640` invocation reuses the same local `[ebp-0x18]`; it does **not** introduce a second independent 160-bit stack-prehistory root. Treat this as SELF/history of the first round, not fresh entropy.

### ADVAPI / SystemFunction036

- `rc4_safe` has eight entries, but they are not eight arbitrary initial secret keys; first use forces rekey.
- Startup threshold mutation 512 -> 16384 creates an asymmetric first-use schedule.
- First two observed SystemFunction036 outputs are consecutive 20-byte segments of the same cached RC4 stream.
- In the validated early sequence the first four outputs have ancestry `[M1, M1, M2, M3]`; for the seven-call first-key model, consumed KSecDD materials are path-conditionally `M1..M6`, while later created roots are not yet output ancestors.

### KSecDD

Do **not** model `M1..M6` as independent secrets. They descend from one persistent 80-byte ratchet:

```text
X_{t+1} = VLH(X_t, P_t)
M_t     = deterministic RC4 transform keyed by X_{t+1}
```

The target-SP3 validated VLH input is a **0x258 = 600-byte pool**, not the full 0xE00 allocation capacity.

Recovered later-stage evidence narrows the pre-QSI direct allocation-history region in the long gather to **24 bytes in the pinned campaign**. This is a provenance bound, not an entropy-bit claim: last writers still matter.

`SystemProcessInformation` class 05 contributes a 3528-byte captured region that is hashed to one SHA-1 value before downstream use, so that boundary has image <=2^160. The stronger statement that everything after the first 0x1b8-byte Idle-process record is untouched pool tail remains OPEN until the exact XP SP3 kernel cold blocks are closed.

## 3. Current decisive blocker

The main blocker is no longer “there are many entropy sources.” It is:

```text
What is the reachable/fresh dimension of the persistent KSecDD state trajectory X_t,
including the exact provenance of P_1...P_n and the boot/persistent Seed ancestry?
```

The registry Seed read bug hypothesis was falsified in later exact-SP3 work: if the 80-byte Seed exists, it is read; zero state is only the missing-Seed branch, not a universal fact.

A simple public-key-only enumeration route dies if an irreducible >=128-bit hidden root survives in the first-key dependency graph. Conversely, if the remaining `P_t` and `X_0` ancestry collapses to a small enumerable state family, the project moves directly into end-to-end key-image computation.

## 4. Immediate next proof obligations — execute in this order

### P1 — close class-05 partial-write semantics

Pinned XP SP3 kernel evidence already shows the captured class-05 buffer begins with a valid 0x1b8-byte Idle-process record:

```text
NextEntryOffset = 0x1b8
NumberOfThreads = 4
PID = 0
0xb8 + 4*0x40 = 0x1b8
```

The bytes immediately after 0x1b8 do not parse as the next normal SPI header. Complete the exact `ExpGetProcessInformation` hot + compiler-split cold blocks and determine, byte-for-byte, what is written on buffer-too-small / partial-enumeration paths. Do not promote “~3 KB untouched tail” until this is proved.

### P2 — 24-byte direct allocation-history region

Recover exact last writers of the 24-byte region preceding the first QSI contribution in the pinned long-gather trace. Classify each byte as:

```text
CONST | SELF | DERIVED | DELTA | FOREIGN
```

Do not convert byte width directly into entropy bits.

### P3 — persistent Seed / X0 reachable family

For the exact first-key path, determine whether `X0` is:

- arbitrary 80-byte persistent history,
- a restricted family induced by setup/boot transitions,
- shared/repeated under image/clone scenarios,
- or reducible by the writeback ratchet.

This is the largest remaining universal public-key-only blocker.

### P4 — compose into OpenSSL first scalar

Only after P1–P3 are bounded, propagate the surviving variables through exact OpenSSL 0.9.8h pool/state-index behavior and `BN_rand_range()` to obtain an upper bound or structured search for `d_first`.

## 5. Important no-go results — do not repeat

- SP1 `SHA_mod_q` 5-byte copy does not imply target-SP3 120-bit collapse.
- `SystemFunction036` failure is checked; stale-local-on-failure attack is false for the validated SP3 provider path.
- Caller output buffer is mixing material, not the sole entropy source.
- Eight ADVAPI entries are not eight arbitrary independent keys.
- Chronologically observed KSecDD rekeys are not automatically independent first-key ancestors.
- 0xE00 allocation capacity is not the VLH input width; validated direct pool width is 0x258.
- OpenSSL does not universally compress the entire Windows history into one final 160-bit digest before the first scalar; the state pool preserves multiple chunk contributions.
- Do not use “stack garbage is constant because XP has no ASLR” as a theorem. Use exact last-writer analysis.

## 6. Recommended reading order

1. `HANDOFF.md` — this file.
2. `STATUS.md` — current branch state.
3. `CLAIMS.md` — claim ledger; this is authoritative for verdicts.
4. `ARTIFACTS.md` — binary/replay identities, external corpus, reproducibility caveats.
5. `findings/` — machine-level evidence and reductions.
6. `versions/` — historical evolution and superseded models.
7. `CHANGELOG.md` — version transition index.
8. XP SP1 source only for hypotheses; never as final SP3 proof.

## 7. Branch policy

The canonical research branch is:

```text
research/public-key-only
```

There is no newer GitHub research branch at the time this handoff was created. Some later results had existed only in prior research logs; the validated subset is now migrated here. Future work must land here immediately so chat history is never required for continuity.

## 8. Publication / safety boundary

Reproduction and proof should use owned/synthetic historical-equivalent keys and systems. The scientific target is implementation/state-space characterization and controlled public-key verification, not recovery or movement of third-party funds.
