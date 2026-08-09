# F003 — NT rekey-threshold mutation creates asymmetric first-use behavior
Status: CONFIRMED IN REFERENCE SOURCE + SP3 TRACE
PK-only effect: explains first-call/second-call asymmetry and invalidates the simple one-rekey-per-call model.

## Correction history
The original F003 draft predicted eight zero-byte first-use rekeys before the first returned byte. V17 raw SP3 trace falsifies that for the first `SystemFunction036` call. The reason is a subtle **mid-call mutation of the global rekey threshold**, already explainable from the reference source.

## Source mechanism
User-mode randlib starts with:

`g_dwRC4RekeyParam = 512`.

On NT, `IsRNGWinNT()` later changes the global to:

`g_dwRC4RekeyParam = 16384`.

In `RandomFillBuffer()`, on a first-use entry:

1. `rc4_safe_select()` returns stored `BytesUsed=0xffffffff`.
2. The rekey branch assigns the **local** `RC4BytesUsed = g_dwRC4RekeyParam`.
3. It then calls `GatherRandomKey()`.
4. On the process's first NT gather, `GatherRandomKeyFastUserMode()` reaches `IsRNGWinNT()`, which changes the **global** rekey threshold from 512 to 16384.
5. After rekey, max output is computed as:

`global g_dwRC4RekeyParam - local RC4BytesUsed`.

For the first call this becomes:

`16384 - 512 = 15872`,

so a 20-byte request can be emitted immediately after the first rekey.

For later first-use entries, the global is already 16384 before the branch, so local is also set to 16384 and:

`16384 - 16384 = 0`.

Those later first-use passes rekey but emit zero bytes.

## Exact SP3 trace anchors
V17 first `SystemFunction036` call shows:
- selected entry has `BytesUsed=0xffffffff`;
- at the select-return site the global rekey parameter is `0x200`;
- after the KSecDD IOCTL/rekey, `rc4_safe` is called with `len=0x14` and PRGA emits 20 bytes.

At the second `SystemFunction036` call, the select-return site shows global rekey parameter `0x4000`. The campaign then records PRGA #2..#8 with length zero before a later 20-byte PRGA use.

This precisely matches the threshold-mutation model.

## Consequence
The historical startup transient is not a stationary round-robin process. The first NT RNG call is special because a global policy parameter changes **inside** the first first-use rekey operation.

Any model that assumes one fixed rekey threshold from process start, or equates one early `SystemFunction036` call with one KSecDD rekey, is invalid.

## PK-only relevance
The mutation controls which cached RC4 state produces subsequent outputs. In particular it enables the first two observed `SystemFunction036` outputs to consume consecutive segments of the same entry-1 RC4 stream (formalized in F005), eliminating an otherwise expected independent RNG root.
