# Claims Ledger

Statuses: `CONFIRMED`, `FALSIFIED`, `OPEN`, `SOURCE-ONLY`, `TRACE-PARTIAL`.

| ID | Claim | Status | PK-only relevance |
|---|---|---|---|
| C-001 | OpenSSL 0.9.8h Windows RAND_poll asks CryptoAPI for 64 bytes | CONFIRMED | fixes exact RSAENH request shape |
| C-002 | XP SP3 RSAENH implements the SP1 20-byte FIPS loop verbatim | FALSIFIED | prevents false SHA_mod_q attack composition |
| C-003 | XP SP3 64-byte request uses two fresh 40-byte provider blocks (40+24) | CONFIRMED | two RSAENH generator transitions per request |
| C-004 | RSAENH ignores SystemFunction036 failure and consumes stale local bytes | FALSIFIED | removes easy deterministic fallback |
| C-005 | Caller output buffer is the sole entropy input to RSAENH | FALSIFIED | underlying SystemFunction036 remains a blocker |
| C-006 | SP1 SHA_mod_q else branch copies only 5 bytes | SOURCE-ONLY | implementation defect, shipped SP3 relevance not shown |
| C-007 | The 5-byte SHA_mod_q copy discards 120 bits of fresh NewGenRandom output | FALSIFIED | untouched suffix already contains current fresh RNG bytes |
| C-008 | One RSAENH out40 block has an effective image <=2^160 through XVAL | CONFIRMED | genuine dimensional collapse, still above 2^128 |
| C-009 | Reference ADVAPI user-mode rc4_safe has 8 entries and forces first-use rekey via BytesUsed=0xffffffff | SOURCE-ONLY | removes hypothetical hidden initial RC4 keys if shipped behavior matches |
| C-010 | The 8 ADVAPI entries constitute eight independent hidden initial RC4 keys | FALSIFIED (SOURCE MODEL) | first-use keys are overwritten by KSecDD-derived rekeys |
| C-011 | Consecutive SystemFunction036 outputs before first Bitcoin key lie in a joint image <2^128 | OPEN | primary break theorem target |
| C-012 | KSecDD persistent-state history contains an irreducible >=128-bit hidden root | OPEN | if confirmed, likely kills simple enumeration route |
| C-013 | Shipped machine code contains a normal-success path that drops/aliases/partially updates RNG state beyond source semantics | OPEN | highest-value machine-level flaw class |
| C-014 | First SP3 ADVAPI rekey input equals MD4(pre-call 20B) followed by 240 zero bytes | CONFIRMED (TRACE/REPLAY) | first user-side 256B rekey input has <=128-bit image |
| C-015 | Within one shipped-SP3 RSAENH CGR64 call, the second SystemFunction036 input has no independent stack-prehistory root because aux20 survives loop-back | OPEN | could remove caller-history independence |
| C-016 | Sequential reference user-mode rc4_safe selector immediately reuses an entry before all 8 are visited | FALSIFIED (SOURCE MODEL) | order is 1,2,3,4,5,6,7,0 |
| C-017 | Reference first-use RandomFillBuffer rekey pass emits zero bytes because local RC4BytesUsed is set to threshold before max-length calculation | SOURCE-ONLY | permits multiple rekeys inside one API request |
| C-018 | Exact shipped SP3 first SystemFunction036 call performs all eight ADVAPI entry rekeys before returning its first byte | OPEN / TRACE-CONSISTENT | radically changes kernel-transition count and dependence model |
| C-019 | Under first-use/no-interleaving conditions, first 8 ADVAPI→KSecDD user input snapshots are deterministic functions of one D=MD4(B), so joint user-side image <=2^128 | SOURCE-THEOREM; C1 CONFIRMED | potentially strong dimensional collapse; C2..C8 pending |
| C-020 | Earlier model `first 7 SystemFunction036 calls => 7 KSecDD rekeys` is established target behavior | SUSPENDED / REQUIRES REVALIDATION | likely call-boundary interpretation error |

## Rule
No target-SP3 claim moves to CONFIRMED from source inspection alone. Exact binary, controlled trace, or bit-exact replay evidence is required. A local <=2^128 projection is not by itself a Bitcoin private-key break unless the downstream joint hidden state/search is also bounded below generic ECDLP.