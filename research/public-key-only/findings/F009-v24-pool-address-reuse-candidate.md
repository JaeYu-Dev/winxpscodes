# F009 — V24 post-gather trace contains a non-local repeated page+8 kernel pointer

Status: OPEN — STRONG TRACE CANDIDATE, SEMANTIC SLOT NOT YET PROVED
Version: v0.1.7
Date: 2026-08-09

## Statement
The eight V24 `NewGenRandomEx` post-`GatherRandomKey` frame captures contain a changing kernel-looking pointer whose observed sequence is:

```text
cycle 1  0xe1337008
cycle 2  0xe1379008
cycle 3  0xe137f008
cycle 4  0xe138e008
cycle 5  0xe12cc008
cycle 6  0xe134d008
cycle 7  0xe1379008
cycle 8  0xe137e008
```

The exact value `0xe1379008` therefore recurs at cycles 2 and 7.

All eight values have low 12 bits `0x008`. This is structurally consistent with an XP x86 pool payload beginning immediately after an 8-byte pool header at a page boundary. For a 3584-byte (`0xE00`) KSecDD working-buffer request, the reference allocator geometry is compatible with a `0xE08` header+payload block.

This is **not yet a proof** that the observed frame value is `pbWorkingBuffer`.

## Why the candidate is important
The reference `GatherRandomKey` path:

1. allocates `cbWorkingBuffer = 3584` bytes through `ALLOC`;
2. uses the buffer to assemble the gather input;
3. hashes only the used prefix via `VeryLargeHashUpdate(pbWorkingBuffer, cbBufferSize, ...)`;
4. frees `pbWorkingBuffer` immediately before returning.

Therefore a pool-looking pointer surviving in a post-gather frame/register is a high-value candidate for the just-freed working-buffer address. The repeated cycle-2/cycle-7 value is particularly important because it would demonstrate non-local reuse of one allocation address within the same eight-cycle KSecDD campaign.

## What is confirmed vs open
### Confirmed observation
- The V24 post-gather frame captures contain the eight-address sequence above.
- Cycle 2 and cycle 7 contain the same candidate pointer `0xe1379008`.
- The candidate is not constant across all cycles; neither an "always same block" nor an "always fresh distinct address" model matches the observed sequence.

### Still open
- Exact shipped-SP3 instruction/stack-slot proof that this value equals `pbWorkingBuffer` or the argument passed to the pool free routine.
- Whether the same-address reuse preserved any byte in the hashed `[0,0x258)` prefix.
- Whether another allocator owner wrote the block between cycles 2 and 7.
- Therefore whether any residual byte can yet be reclassified from `FOREIGN` to `SELF`.

## Reachability consequence
No entropy/search reduction is claimed from address equality alone.

If later machine-code provenance establishes that the value is the `0xE00` working allocation address, the trace will falsify the model that every gather necessarily receives a never-reused allocation. However, the stronger Exact-Reuse condition needed for support reduction is:

```text
same working block
AND
no intervening writer to the relevant residual offset
```

Only under that stronger condition does a residual byte contribute zero *new* freedom relative to its earlier value.

## Next proof obligations
1. Map the post-gather frame/register slot to the shipped-SP3 `GatherRandomKey` return sequence and pool-free argument.
2. Recover or capture per-cycle working-buffer contents at allocation return and immediately before `VeryLargeHashUpdate`.
3. Build the SP3-native write mask only over the actually hashed prefix `[0,0x258)`.
4. For cycle 2 -> cycle 7, classify each unwritten hashed byte as `ZERO`, `SELF`, or `FOREIGN` using last-writer provenance.

## Research rule
Do not promote this finding to a kernel-residual collapse theorem merely from repeated addresses. Address reuse is a prerequisite for SELF provenance, not proof of preserved content.