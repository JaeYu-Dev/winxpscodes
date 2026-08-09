# Claims Ledger

Statuses: `CONFIRMED`, `FALSIFIED`, `OPEN`, `SOURCE-ONLY`.

| ID | Claim | Status | PK-only relevance |
|---|---|---|---|
| C-001 | OpenSSL 0.9.8h Windows RAND_poll asks CryptoAPI for 64 bytes | CONFIRMED | fixes exact RSAENH request shape |
| C-002 | XP SP3 RSAENH implements the SP1 20-byte FIPS loop verbatim | FALSIFIED | prevents false SHA_mod_q attack composition |
| C-003 | XP SP3 64-byte request uses two fresh 40-byte provider blocks (40+24) | CONFIRMED | two OS-RNG auxiliary draws per request |
| C-004 | RSAENH ignores `SystemFunction036` failure and consumes stale local bytes | FALSIFIED | removes easy deterministic fallback |
| C-005 | Caller output buffer is the sole entropy input to RSAENH | FALSIFIED | underlying SystemFunction036 remains a blocker |
| C-006 | SP1 SHA_mod_q else branch copies only 5 bytes | SOURCE-ONLY | implementation defect, shipped SP3 relevance not shown |
| C-007 | The 5-byte SHA_mod_q copy discards 120 bits of fresh NewGenRandom output | FALSIFIED | untouched suffix already contains current fresh RNG bytes |
| C-008 | One RSAENH out40 block has an effective image <= 2^160 through XVAL | CONFIRMED | genuine dimensional collapse, but still above 2^128 |
| C-009 | ADVAPI user-mode rc4_safe has 8 entries and forces first-use rekey via BytesUsed=0xffffffff in reference source | SOURCE-ONLY | removes hypothetical hidden random initial RC4 keys if shipped behavior matches |
| C-010 | The 8 ADVAPI entries constitute eight independent hidden initial RC4 keys | FALSIFIED (SOURCE MODEL) | initial keys are overwritten before normal first-use output; shipped SP3 validation still required |
| C-011 | Consecutive SystemFunction036 outputs before first Bitcoin key lie in a joint image < 2^128 | OPEN | primary break theorem target |
| C-012 | KSecDD persistent-state history contains an irreducible >=128-bit hidden root | OPEN | if confirmed, likely kills simple enumeration route |
| C-013 | Shipped machine code contains a normal-success path that drops/aliases/partially updates RNG state beyond source semantics | OPEN | highest-value machine-level flaw class |
| C-014 | A 20-byte SystemFunction036 caller-buffer update influences ADVAPI circular state through at most a 16-byte MD4 digest in the reference source | SOURCE-ONLY | local 160 -> <=128 dimensional projection |
| C-015 | Within one shipped-SP3 RSAENH CGR64 call, the second SystemFunction036 input has no independent stack-prehistory root because aux20 survives loop-back | OPEN | could remove one 128-bit caller-history term per 64-byte request |
| C-016 | Sequential user-mode rc4_safe selector immediately reuses an entry before all 8 entries are visited | FALSIFIED (SOURCE MODEL) | reference order is 1,2,3,4,5,6,7,0 |

## Rule
No claim moves to target-binary CONFIRMED from source inspection alone when the target is SP3 shipped behavior. Exact binary or controlled trace/replay evidence is required.