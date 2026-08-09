# Claims Ledger

Statuses: `CONFIRMED`, `FALSIFIED`, `OPEN`, `SOURCE-ONLY`, `SUPERSEDED`.

| ID | Claim | Status | PK-only / reachability relevance |
|---|---|---|---|
| C-001 | OpenSSL 0.9.8h Windows RAND_poll asks CryptoAPI for 64 bytes | CONFIRMED | fixes exact RSAENH request shape |
| C-002 | XP SP3 RSAENH implements the SP1 20-byte FIPS loop verbatim | FALSIFIED | prevents false SHA_mod_q attack composition |
| C-003 | XP SP3 64-byte request uses two 40-byte provider blocks (40+24) | CONFIRMED | two RSAENH generator transitions per request |
| C-004 | RSAENH ignores SystemFunction036 failure and consumes stale local bytes | FALSIFIED | removes easy deterministic fallback |
| C-005 | Caller output buffer is the sole entropy input to RSAENH | FALSIFIED | underlying SystemFunction036 remains a blocker |
| C-006 | SP1 SHA_mod_q else branch copies only 5 bytes | SOURCE-ONLY | historical source defect; not target-SP3 break evidence |
| C-007 | The 5-byte SHA_mod_q copy discards 120 bits of fresh NewGenRandom output | FALSIFIED | untouched suffix already contains current RNG bytes |
| C-008 | One RSAENH out40 block has effective image <=2^160 through XVAL | CONFIRMED | real dimensional collapse, still above 2^128 |
| C-009 | ADVAPI rc4_safe has eight entries with first-use forced rekey | CONFIRMED SOURCE + TRACE-CONSISTENT | removes arbitrary initial-key roots |
| C-010 | The eight ADVAPI entries are eight arbitrary hidden initial RC4 keys | FALSIFIED | they are keyed by KSecDD-derived rekey material |
| C-011 | Reachable first-private-scalar image is searchable below generic ECDLP | OPEN | primary break theorem target |
| C-012 | KSecDD persistent-state history contains an irreducible >=128-bit hidden root on the actual first-key dependency graph | OPEN | may kill simple enumeration route |
| C-013 | Shipped normal-success machine code drops/aliases/partially updates RNG state beyond intended source semantics | OPEN | high-value flaw class |
| C-014 | First V17 SP3 ADVAPI rekey input equals MD4(B1)||0^240 | CONFIRMED TRACE/BLOB | first caller-side KSecDD input <=128-bit image |
| C-015 | Within one shipped-SP3 RSAENH CGR64 call, the second SystemFunction036 input has no independent stack-prehistory root | OPEN | exact D681..D693 audit still required |
| C-016 | Reference selector immediately reuses one entry before visiting all eight | FALSIFIED | sequential order is 1,2,3,4,5,6,7,0 |
| C-017 | Every first-use rekey pass necessarily emits zero bytes from process start | FALSIFIED FOR FIRST NT CALL | first call is exceptional due threshold mutation |
| C-018 | First SystemFunction036 call performs all eight rekeys before first output | FALSIFIED BY V17 | first call rekeys entry1 then emits 20B |
| C-019 | C1..C8 all share one MD4 root | SUPERSEDED/FALSIFIED | C1 uses D1; C2..C8 share D2 |
| C-020 | Early model `one SystemFunction036 call => one KSecDD rekey` | FALSIFIED | second call contains seven rekeys then cached output |
| C-021 | NT detection mutates rekey threshold 512->16384 during first rekey, creating first/second-call asymmetry | CONFIRMED SOURCE + SP3 TRACE | startup machine-state anomaly |
| C-022 | V17 C2..C8 differ by the same 16B D2=MD4(B2) embedded at successive +7 byte positions | CONFIRMED RAW BLOBS | seven caller-side rekey inputs share one 128-bit root |
| C-023 | First two observed SystemFunction036 outputs use consecutive 20B segments of the same entry1 RC4 stream | CONFIRMED SP3 TRACE | removes one independent KSecDD/ADVAPI output root |
| C-024 | KSecDD rekeys #2..#8 occurring between outputs #1 and #2 influence output #2 | FALSIFIED | they key other cached entries; R2 returns to entry1 |
| C-025 | Provenance of SystemFunction036 pre-call buffers L1/L2 collapses below two independent 160-bit roots | OPEN | next high-value reachability target |
| C-026 | RSAENH state immediately before provider initialization is the fixed AlgorithmCheck/self-test expected 20-byte vector | CONFIRMED MACHINE/TRACE | removes an entire hidden 160-bit provider root |
| C-027 | RSAENH state after provider init + acquire bridge has no ancestry from KSecDD rekey roots M2..M8 | CONFIRMED BY COMPOSITION | seven chronologically interposed kernel transitions vanish from early-state support accounting |
| C-028 | Early provider state can be written as `S2=F(M1,L1,L2,B1,B2)` for fixed shipped constants | CONFIRMED BY COMPOSITION | isolates the exact remaining roots before runtime CGR |
| C-029 | V17 L1/L2 values are structured stale-frame data with a provably small support | OPEN | raw structure is observed, but last-writer/support theorem is not yet proved |
| C-030 | V17 SystemFunction outputs #1..#4 have KSecDD/RC4-root ancestry `[M1,M1,M2,M3]` | CONFIRMED TRACE + KSA REPLAY | exact early ancestry map; removes chronology-based overcount |
| C-031 | Under the standard seven-SystemFunction first-key call sequence, only roots `M1..M6` are consumed; `M7,M8` are not output ancestors | CONFIRMED BY TRACE-GROUNDED COMPOSITION, PATH-CONDITIONAL | reduces first-key Windows-side root count from eight created roots to six consumed roots |
| C-032 | The joint reachable image of `M1..M6` factors as six independent kernel secrets | OPEN / DO NOT ASSUME | next task is to prove further ratchet/provenance collapse or an irreducible blocker |

## Rule
No target-SP3 claim moves to CONFIRMED from source inspection alone unless independently anchored by exact binary/trace/replay evidence. Correlation is only promoted when it removes an actual data-flow ancestor or produces a formal search/image reduction. A local <=2^128 projection is not by itself a Bitcoin private-key break unless the downstream joint search is also below generic ECDLP. Chronological RNG events are never counted as independent roots without a data-flow proof that they survive into `d_first`.
