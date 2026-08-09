# F012 — Class-05 3528B input is SHA-1 projected; exact partial-write tail remains open

Status: **CONFIRMED projection / OPEN tail provenance**

## Confirmed local projection

In the pinned XP SP3 KSecDD gather campaign, the captured `SystemProcessInformation` class-05 region has length:

```text
0xDC8 = 3528 bytes
```

The downstream KSecDD path hashes that captured region through SHA-1 and carries a 20-byte result forward. A recorded sample produced:

```text
SHA1(QSI05[0:3528]) = be1f012d701e54af2b0b722d3a32d59cd5fd5375
```

Therefore, regardless of the nominal 3528-byte raw width, this local boundary has:

```text
|Image(class05_projection)| <= 2^160
```

This is a codomain/image bound, not an entropy estimate.

## Strong partial-write candidate

Parsing the beginning of the captured class-05 buffer as 32-bit XP `SYSTEM_PROCESS_INFORMATION` yields a valid Idle-process record:

```text
NextEntryOffset = 0x1b8
NumberOfThreads = 4
ImageName       = empty
PID             = 0
record size     = 0xb8 + 4*0x40 = 0x1b8
```

The bytes immediately after offset `0x1b8` do not parse as the next ordinary process-information header in the captured sample.

This strongly suggests a buffer-too-small / partial-write path in which a valid first record is written while a large remainder may retain prior contents.

## What remains OPEN

Do **not** yet state that bytes `[0x1b8:0xdc8)` are all untouched tail.

The exact XP SP3 `ntoskrnl!ExpGetProcessInformation` implementation has compiler-split hot/cold blocks. The hot path shows size accumulation and `STATUS_INFO_LENGTH_MISMATCH` behavior, but the cold blocks must be closed before a byte-exact write boundary can be promoted.

The remaining theorem obligation is:

```text
Which exact offsets of QSI05[0:0xdc8) are written on the observed path,
and what are the last writers of every unwritten offset?
```

## Reachability relevance

If a large tail is proved to be SELF/current-call residue or deterministic allocator history, the raw process-information contribution may add far fewer independent roots than its nominal size suggests. If it is FOREIGN pool residue, it may instead preserve a large hidden-state blocker.

Either way, the 3528-byte raw width itself must never be counted as 3528 bytes of independent entropy because the downstream projection is only 160 bits.

## Target-kernel note

Recovered exact-kernel work pinned an XP SP3 `ntoskrnl.exe` sample with size 2,188,928 bytes and SHA-256:

```text
6005a1a1db35481612ef76856a609954990f058bfc42dfa7b0d43402bf3f3dcf
```

with RSDS GUID/age recorded as:

```text
47A5AC97-343A-4A7A-BF14-EFD9E9933772, age 2
```

Use this identity when resuming the `ExpGetProcessInformation` cold-block audit.
