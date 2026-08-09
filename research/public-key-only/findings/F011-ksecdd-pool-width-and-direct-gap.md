# F011 — Validated KSecDD/VLH input width is 600 bytes; direct pre-QSI provenance gap is 24 bytes

Status: CONFIRMED RECOVERED ANALYSIS + TARGET REPLAY
Version: v0.1.8
Date: 2026-08-09

## Statement
The target long-gather path must not be modeled as hashing the entire `0xE00 = 3584` byte temporary allocation. The validated direct VLH pool is exactly:

`0x258 = 600 bytes = 4 * 0x96`.

Recovered pinned-campaign analysis further narrows the direct allocation-history region before the first QSI contribution to 24 bytes.

The 24-byte figure is a **provenance width**, not an entropy estimate.

## Evidence
The public target replay defines a 600-byte `pool.bin`, splits it into four 150-byte segments, and reproduces the VLH/seedbase transition from those segments. This independently falsifies the earlier shortcut `allocation capacity == hashed-input width`.

The reference semantics are consistent: `GatherRandomKey` allocates a larger working buffer but calls `VeryLargeHashUpdate` with the actual used length, not automatically with the allocation capacity.

Recovered target-specific work then audited the long-gather cursor before the first QSI contribution and reduced the directly inherited allocation-history prefix to 24 bytes for the pinned campaign.

## Reachability consequence
Do not assign fresh degrees of freedom from 3584 bytes. The actual object to classify is the used 600-byte pool, and within it only bytes whose last writers are genuinely independent `FOREIGN` values count as new roots.

The remaining 24-byte direct pre-QSI region must be classified byte-by-byte as:

`CONST | SELF | DERIVED | DELTA | FOREIGN`.

No inference such as `24 bytes = 192 bits entropy` is allowed.

## Relation to F009
F009's repeated `...008` allocator-address candidate remains useful for last-writer provenance, but allocator capacity and address reuse are logically separate from the validated 600-byte VLH input width.

## Scope / falsifier
The 600-byte claim is target-replay anchored. The 24-byte direct-gap claim is tied to the pinned long-gather campaign and must be revised if a different reachable production path writes/positions the first QSI contribution differently.

## Next
Recover exact last writers of the 24 bytes and determine whether any survive as irreducible `FOREIGN` roots in the first-key dependency graph.