# Artifact / Reproducibility Index

This file tells a successor **what evidence set each claim refers to** and prevents accidental mixing of SP1 source, SP3 shipped binaries, public replay artifacts, and later recovered research logs.

## 1. Canonical code/source repository

Current research branch:

```text
JaeYu-Dev/winxpscodes
branch: research/public-key-only
```

The large `Source/XPSP1/...` tree is a **reference/source-hypothesis corpus only**. Never promote SP3 behavior from it without binary/trace/replay confirmation.

## 2. Public SP3 replay/campaign corpus

External public replay project used throughout the reconstruction:

```text
Melik159/xp-cgr-replay
```

Campaign names appearing in findings/logs include V17, V20/V20.1, V22, V23, V24, V26, V28/V29 and related `seed2state_*`, `vlh/camp01`, QSI, KSA/PRGA and provider replay artifacts.

This repository does **not** currently vendor all raw campaign blobs. When reproducing a claim, record the exact upstream commit/path used in a new finding or evidence manifest.

## 3. Historical empirical target set

The empirical XP SP3 campaign/paper records these file identifiers for its experimental environment:

```text
advapi32.dll  MD5 bab489a5fe26f2d0c910cf7af7e4cf92
rsaenh.dll    MD5 54dae3ea34802b4ed9ae1c6b1209fa56
libeay32.dll  MD5 192ef960bd269b499097fb154ef2b6f8
bitcoin.exe   MD5 307ad86c412c02abeb821afbb900355b
```

Use MD5 here only as an artifact identifier, not as a security primitive.

Important: earlier notes contain more than one `ksecdd.sys` identity from different artifact sets. **Do not silently mix them.** For a new KSecDD machine-level claim, pin the exact binary hash/version in that finding.

## 4. RSAENH target

Primary shipped provider target:

```text
Windows XP SP3
rsaenh.dll 5.1.2600.5507
```

Important addresses used by the current model include:

```text
rsaenh+0x0D640   provider helper / loop
rsaenh+0x0D693   SystemFunction036(aux20,20) setup
rsaenh+0x0D7D5   runtime call into D640
rsaenh+0x27101   FIPS provider block
rsaenh+0x31958   process-global 20-byte provider state
```

All absolute/RVA observations are artifact-specific; rebase/address changes across builds must not be treated as semantic changes without CFG comparison.

## 5. Exact XP SP3 kernel pinned for class-05 continuation

Recovered later-stage work pinned:

```text
ntoskrnl.exe
size   = 2,188,928 bytes
SHA256 = 6005a1a1db35481612ef76856a609954990f058bfc42dfa7b0d43402bf3f3dcf
RSDS   = 47A5AC97-343A-4A7A-BF14-EFD9E9933772, age 2
```

Use this target for the pending `ExpGetProcessInformation` hot/cold-block audit associated with F012/C-040.

## 6. Important captured values

### Class-05 projection sample

```text
QSI05 length = 0xDC8 = 3528 bytes
SHA1(QSI05[0:3528]) = be1f012d701e54af2b0b722d3a32d59cd5fd5375
```

### First parsed Idle-process record candidate

```text
NextEntryOffset = 0x1b8
NumberOfThreads = 4
PID = 0
base record 0xb8 + 4 * thread 0x40 = 0x1b8
```

The claim that all bytes after `0x1b8` are untouched tail is OPEN.

## 7. Evidence provenance labels

Use these labels in new findings:

- `VENDORED-SOURCE`: content present in this repository.
- `EXTERNAL-REPLAY`: public replay/campaign artifact outside this repository.
- `EXACT-BINARY`: exact shipped binary / disassembly with hash pinned.
- `TRACE`: controlled runtime capture.
- `REPLAY`: bit-exact local reproduction.
- `RECOVERED-LOG`: result recovered from prior research logs; promote only when sufficient target identifiers / reproducible facts are preserved.

Any `RECOVERED-LOG` result that cannot be independently revalidated must be downgraded to OPEN rather than silently treated as proof.

## 8. Successor checklist before making a new claim

1. Pin build + file hash.
2. Record exact external replay commit/path if used.
3. Separate SP1 source semantics from SP3 binary behavior.
4. Record raw input/output hashes or relevant bytes where practical.
5. State the last-writer / state-ancestry consequence.
6. State the reachable-image consequence.
7. State the falsifier and what is still OPEN.
8. Update `CLAIMS.md`, `STATUS.md`, `CHANGELOG.md`, and `HANDOFF.md` if priorities changed.
