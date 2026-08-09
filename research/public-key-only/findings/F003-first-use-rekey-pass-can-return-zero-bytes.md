# F003 — First-use rekey passes can return zero bytes
Status: SOURCE-CONFIRMED / TARGET-SP3-CALL-BOUNDARY-OPEN
PK-only effect: major call-count/state-chain correction if preserved in shipped SP3.

## Exact source control flow
For user-mode randlib, `RandomFillBuffer(pbBuffer, &len)` performs:

1. `UpdateCircularHash(..., pbBuffer, len)`.
2. `rc4_safe_select(..., &KeyId, &RC4BytesUsed)`.
3. If `RC4BytesUsed >= g_dwRC4RekeyParam`:
   - set local `RC4BytesUsed = g_dwRC4RekeyParam`;
   - obtain rekey material with `GatherRandomKey()`;
   - `rc4_safe_key()` selected entry (which internally resets the entry's stored `BytesUsed` to 0).
4. Compute `dwMaxPossibleBytes = g_dwRC4RekeyParam - RC4BytesUsed` using the **local** variable.
5. Truncate `*pdwLength` to that maximum.
6. `rc4_safe(..., *pdwLength, pbBuffer)`.

On a first-use entry, startup set stored `BytesUsed=0xffffffff`, so the rekey branch runs. The local variable is then explicitly assigned the threshold (512 in user mode), making:

`dwMaxPossibleBytes = 512 - 512 = 0`.

Therefore that `RandomFillBuffer` pass rekeys the entry but emits **zero bytes**. `GenRandom()` adds zero to `dwFilledBytes` and loops again.

## Sequential first-use consequence
The selector source order is `1,2,3,4,5,6,7,0,1,...`. If all eight entries are still first-use, a single initial non-empty `GenRandom()` request can execute:

- pass 1: select 1, rekey, 0 bytes
- pass 2: select 2, rekey, 0 bytes
- ...
- pass 8: select 0, rekey, 0 bytes
- pass 9: select 1, no rekey, finally emit requested bytes

Thus the reference-source semantics predict an **eight-rekey initialization burst before the first returned byte**, not necessarily one rekey per API invocation.

## Why this matters
Earlier research notes modeled the first several `SystemFunction036` calls as each causing one KSecDD rekey. That interpretation must be suspended. If shipped XP SP3 preserves this control flow, the first call itself can initialize all eight ADVAPI RC4 entries, while later calls consume already-keyed streams without KSecDD rekey until their byte thresholds are reached.

## Existing SP3 evidence consistent with this model
Prior V17/V20 campaign notes report that the first eight KSecDD IOCTL outputs initialize the eight ADVAPI RC4 states, after which PRGA reuses initialized states. This is consistent with an initialization burst, but the existing notes do not yet prove all eight IOCTLs are nested inside one `SystemFunction036` invocation.

## Target-binary proof obligations
1. In exact XP SP3 ADVAPI32, identify the post-rekey assignment to the local `RC4BytesUsed` value.
2. Confirm the max-output calculation uses that local threshold value, yielding zero bytes on first-use rekey passes.
3. Recover call nesting/timestamps from V17 or a controlled trace to show whether eight KSecDD rekeys occur before the first `SystemFunction036` return.

## Falsifier
If shipped SP3 sets the post-rekey local value to 0 (or otherwise emits bytes immediately after each rekey), the eight-rekey-per-first-call model is false for the target.