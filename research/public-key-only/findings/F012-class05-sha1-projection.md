# F012 — Class-05 SystemProcessInformation is projected through one SHA-1 value

Status: CONFIRMED LOCAL PROJECTION; PARTIAL-WRITE TAIL REMAINS OPEN
Version: v0.1.8
Date: 2026-08-09

## Statement
In the pinned XP SP3 long-gather campaign, `SystemProcessInformation` (class 05) contributes a captured region of:

`0xDC8 = 3528 bytes`.

At the downstream boundary relevant to the reconstructed KSecDD gather path, that region is SHA-1 projected to one 20-byte value. Therefore the local class-05 projection has image at most:

`2^160`.

This does **not** imply that the whole KSecDD state or first Bitcoin private key has only 160 bits of support.

## Captured structure
The first `0x1B8` bytes of the pinned class-05 capture parse consistently as one XP 32-bit Idle-process record:

- `NextEntryOffset = 0x1B8`
- `NumberOfThreads = 4`
- empty process image name
- PID = 0
- record-size arithmetic: `0xB8 + 4 * 0x40 = 0x1B8`.

Bytes immediately after `0x1B8` do not parse as the next ordinary process-information record header.

## Confirmed vs open
### Confirmed
- class-05 request/captured width is 3528 bytes in the pinned campaign;
- the whole captured class-05 region is SHA-1 projected to a 20-byte downstream value;
- therefore raw byte width must not be counted as thousands of independent bits after that boundary.

### OPEN
The stronger hypothesis:

`only [0,0x1B8) was written by ExpGetProcessInformation and [0x1B8,0xDC8) is untouched prior pool residue`

is **not yet proved**. The exact XP SP3 `ntoskrnl!ExpGetProcessInformation` compiler-split cold/error blocks must be closed to determine whether a failed second-record attempt writes any bytes before returning a buffer-too-small / information-length status.

## Public-key-only relevance
If the tail is proved untouched and its allocator history is SELF/DERIVED, class-05 may contribute much less fresh freedom than its 3528-byte appearance suggests. If the tail is FOREIGN, it can remain a blocker even though SHA-1 reduces its local output image to 160 bits.

The next theorem must therefore be a **last-writer theorem**, not an entropy estimate from the observed bytes.

## Falsifier / next proof
Trace exact target `ExpGetProcessInformation` hot and cold blocks around the second-record size check and error return. Record every write to the output buffer before the failure branch. Only then promote or falsify the untouched-tail hypothesis (C-040).