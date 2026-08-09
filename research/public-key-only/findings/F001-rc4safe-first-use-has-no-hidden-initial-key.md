# F001 — rc4_safe first-use adds no hidden initial RC4-key dimension
Status: SOURCE-CONFIRMED / SHIPPED-SP3-VALIDATION-REQUIRED
PK-only effect: positive reduction of the model; not a break.

## Machine/source fact
User-mode `rc4_safe_startup()` allocates eight entries and initializes each entry's `BytesUsed` to `0xffffffff`. It does not initialize the RC4 key bytes themselves. `rc4_safe_select()` increments a static selector and returns the selected entry's `BytesUsed`.

`RandomFillBuffer()` then checks `RC4BytesUsed >= g_dwRC4RekeyParam`. On first use this is necessarily true in the reference source. It obtains a 256-byte rekey buffer via `GatherRandomKey()`, calls `rc4_safe_key()` for the selected entry, zeroes the temporary key buffer, and only *after that* calls `rc4_safe()` to produce output.

## Algebraic consequence
Let `K_i^alloc` be arbitrary allocator residue in entry i's RC4 key struct and `M_i` the rekey material. On normal first use:

`K_i^alloc -> overwritten by KSA(M_i) -> output`

There is no data-flow edge from `K_i^alloc` to the output. Therefore the eight nominal RC4 structs do **not** imply eight independent hidden initial RC4 keys.

## Selector consequence
The user-mode selector is static-zero initialized and performs `circular++` before masking by `Entries-1`. With 8 entries, a sequential first-use sequence is:

`1,2,3,4,5,6,7,0,1,...`

Thus the easy hypothesis "first calls repeatedly reuse one unkeyed/default entry" is false under this source semantics.

## PK-only relevance
This removes a false hidden-state term from the model. The effective unknown root moves downward to the rekey materials `M_i`, which are produced by the KSecDD/reseed chain. It does not by itself reduce that root below 128 bits.

## Shipped-binary obligations
For XP SP3 ADVAPI32, confirm:
1. 8 user-mode entries.
2. first-use `BytesUsed=0xffffffff` initialization.
3. threshold branch occurs before RC4 output.
4. rekey writes the selected entry before output.
5. selector arithmetic/order matches reference semantics.

## Falsifier
Any normal-success SP3 path that invokes RC4 output from an entry before `rc4_safe_key()` on first use invalidates this finding for the target binary.