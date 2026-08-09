# F011 — KSecDD validated VLH pool is 600B; direct pre-QSI provenance gap narrows to 24B

Status: **CONFIRMED for width/provenance bound; entropy value OPEN**

## Claim A — direct VLH pool width

The target-SP3 long-path replay uses a `pool.bin` of:

```text
POOL_LEN = 0x258 = 600 bytes
```

and the VLH update consumes that used pool, not the entire `0xE00 = 3584` byte allocation capacity.

Therefore the previous model

```text
0xE00 allocation == 3584 bytes of direct VLH input
```

is false.

## Claim B — 24-byte direct allocation-history region

Recovered later-stage analysis of the pinned long-gather path narrowed the direct allocation-history-dependent region **before the first QSI contribution** to 24 bytes.

This is a **provenance-width result**, not an entropy estimate.

The correct statement is:

```text
unresolved direct pre-QSI provenance width <= 24 bytes
```

not:

```text
fresh entropy = 192 bits
```

Each byte still requires an exact last-writer classification.

## Reachability relevance

This correction removes a major overcount. The KSecDD gather should be modeled as a 600-byte structured pool containing explicit QSI/system contributions, self-history, deterministic/derived fields, and a much smaller unresolved direct allocation-history region.

The remaining first-key problem is the joint reachable image of:

```text
X0, P1, P2, ...
```

not the nominal size of the temporary pool allocation.

## Required byte classes

For the remaining 24 bytes, classify every byte as one of:

- `CONST`
- `SELF`
- `DERIVED`
- `DELTA`
- `FOREIGN`

Only `FOREIGN` or genuinely independent `DELTA` dimensions can add a new root to the Public-Key-Only search model.

## What this finding does NOT claim

- It does not prove the 24 bytes are all uninitialized.
- It does not prove allocator reuse.
- It does not prove those 24 bytes are preserved across cycles.
- It does not assign min-entropy.
- It does not reduce the final key image below 2^128 by itself.

## Next proof obligation

Recover the exact writer mask for those 24 bytes in the pinned SP3 path and link each surviving byte to its last writer before VLH consumption.
