# Research Changelog

This file is the fastest handoff path for a new researcher. **Read `HANDOFF.md` first**, then newest version, then referenced findings.

## v0.1.8 — Recovered-log consolidation + canonical handoff
- Added `HANDOFF.md` as the canonical successor entry point; prior chat transcripts are no longer required for continuity.
- Migrated the validated subset of later research that had existed only in prior research logs.
- Added F010 and promoted C-015: within one RSAENH CGR64 invocation, the second `SystemFunction036` call reuses the same `[ebp-0x18]` local and adds no independent 160-bit stack-prehistory root.
- Added F011: target-SP3 validated direct VLH pool width is `0x258 = 600B`; recovered pinned analysis narrows the direct pre-QSI allocation-history region to 24B. Explicitly forbids interpreting 24B as 192 bits of entropy without provenance proof.
- Added F012: class-05 `SystemProcessInformation` contributes a captured `0xDC8 = 3528B` region that is SHA-1 projected to 20B, so that local boundary has image <=2^160. The stronger `0x1b8 written + untouched ~3KB tail` interpretation remains OPEN pending exact XP SP3 `ExpGetProcessInformation` cold-block closure.
- Added F013: existing 80-byte registry Seed is read by the exact-SP3 path; universal `X0=0` initialization is falsified. Zero state remains only the missing-Seed branch.
- Updated README, STATUS and CLAIMS to make the branch internally consistent.
- Main next work: class-05 byte-exact partial-write semantics -> 24B last-writer provenance -> persistent `X0` reachable family -> OpenSSL/BN composition.

## v0.1.7 — SP3 pool-provenance checkpoint
- Synchronized the ledger after v0.1.6: F008 and `versions/v0.1.6.md` already existed, while STATUS/CLAIMS/CHANGELOG had remained at v0.1.5.
- Added F009 as a deliberately OPEN machine-trace candidate.
- Eight V24 post-gather frame captures contain pointer-like values `e1337008, e1379008, e137f008, e138e008, e12cc008, e134d008, e1379008, e137e008`.
- Cycle 2 and cycle 7 repeat `0xe1379008` exactly; all eight values end in `0x008`, consistent with but not proving an XP x86 page+8 pool-payload interpretation.
- Explicitly separated three propositions: repeated address observation = confirmed; identity as KSecDD `pbWorkingBuffer` = open; preserved residual contents / zero fresh freedom = open.
- Reasserted that only the used 0x258-byte prefix matters for VLH reachability, not the full 0xE00 allocation capacity.
- Main next proof is shipped-SP3 semantic mapping of the candidate pointer plus byte-level last-writer provenance over `[0,0x258)`.

## v0.1.6 — Persistent KSecDD ratchet eliminates independent-M-root model
- Added F008 from V24 SP3 long-path trace/replay.
- Established the state model `X_{t+1}=VLH(X_t,P_t)` and `M_t=C_t XOR KS_{X_{t+1}}[0:256]` for the validated long-path cycles.
- Full 80-byte `seedbase_after` values are handed across cycle boundaries and reused as RC4 KSA keys; representative boundaries were checked 80/80 bytes exactly.
- Therefore M1..M6 are not six independent KSecDD roots. Their joint image depends on one starting state `X0`, sequential gather inputs `P1..P6`, and the reduced caller-side startup family.
- Reframed the Windows-side blocker as the fresh differential dimension/provenance of `P1..P6` plus the reachable image of `X0`.
- No `<2^128` search bound is claimed from the 80-byte state width.

## v0.1.5 — Reachable-image reframing + six-root first-key ancestry
- Reframed the primary research object from “Public-Key-Only” to the reachable first-private-scalar image `R_C = Image(Phi_C)`; PK-only recovery is now the final corollary/attack model.
- Added F007 from V17 raw trace + KSA replay.
- Exact useful-output root map for first four SystemFunction036 calls: `[M1,M1,M2,M3]`.
- PRGA #1 and #9 are consecutive segments of the same entry1 stream; PRGA #10 directly KSA-matches IOCTL #2 and PRGA #11 directly KSA-matches IOCTL #3.
- Composed the standard first-key seven-SystemFunction call sequence to root ancestry `[M1,M1,M2,M3,M4,M5,M6]`.
- Therefore M7/M8 are startup-created but not output ancestors of the modeled first-key path.
- Explicitly prohibited treating M1..M6 as six independent kernel secrets; their joint KSecDD persistent-state ancestry became the primary Windows-side reachability question.
- D640 within-CGR64 `[ebp-18h]` second-round fresh-root proposition remained OPEN here and was promoted in v0.1.8 after recovered exact write-set work.

## v0.1.4 — Early RSAENH ancestry pruning
- Added F006: composed fixed pre-init RSAENH state + F005 same-stream reuse through provider initialization and CryptAcquireContext bridge.
- Formal early-state dependency is now `S2 = F(M1,L1,L2,B1,B2)` for fixed shipped constants.
- KSecDD rekey roots M2..M8 are proven absent from the data-flow ancestry of S2 even though their rekeys occur chronologically between the first two useful SystemFunction036 outputs.
- Promoted the fixed post-AlgorithmCheck RSAENH state into the active PK-only model rather than counting it as a hidden 160-bit root.
- Next target is exact last-writer provenance of L1/L2 and rsaenh caller-prefixes B1/B2. V17 raw values are strongly structured but are not yet assigned a support bound.

## v0.1.3 — NT threshold mutation + same-stream reuse
- Directly read V17 raw SP3 trace and all eight 256-byte IOCTL input blobs.
- Resolved first/second call asymmetry: `g_dwRC4RekeyParam` changes from 512 to 16384 **inside the first NT rekey**.
- First SystemFunction036: entry1 rekey + immediate 20-byte output.
- Second SystemFunction036: seven zero-byte first-use rekeys for entries2..0, then returns to entry1 and emits the next 20 bytes from the same RC4 stream.
- Added F005: first two SystemFunction036 outputs share one cached ADVAPI RC4-key root; intervening KSecDD transitions #2..#8 are not ancestors of the second output.
- Corrected F004: C1 uses MD4 root D1; C2..C8 repeatedly embed a second digest D2 at 7-byte offsets. Raw blobs confirm the pattern exactly.
- v0.1.2's claim that all eight rekey inputs share one D is **superseded/falsified**.

## v0.1.2 — Superseded exploratory model
- Detected that zero-byte first-use rekey passes exist in source semantics.
- Proposed an eight-rekey-before-first-byte model and one-D C1..C8 family.
- Raw V17 validation in v0.1.3 showed the first call is exceptional because the global NT rekey threshold mutates mid-call. Preserve this version as an audit trail; do not reuse its one-D/eight-before-first-byte conclusions.

## v0.1.1 — ADVAPI first-use reductions
- Added F001: eight rc4_safe structs are not eight hidden initial keys under reference semantics; first use forces rekey.
- Added F002: 20-byte caller-buffer contribution is projected through MD4 to at most 128 bits before rekey.
- Recorded selector order `1,2,3,4,5,6,7,0,...` for the reference user-mode path.

## v0.1.0 — PK-only baseline
- Froze Public-Key-Only invariant and proof ladder.
- Corrected SP1/SP3 FIPS path confusion.
- Fixed SP3 OpenSSL/CryptoAPI request shape at 64 bytes -> RSAENH 40+24.
- Retained positive RSAENH machine-algebra result `|Image(out40)| <= 2^160`.
- Moved main bottleneck to ADVAPI/SystemFunction036/KSecDD effective hidden-state dimension.

## Handoff rule
Every future milestone must:
1. create `versions/vX.Y.Z.md`;
2. update `STATUS.md`;
3. update `CLAIMS.md` when a proposition changes status;
4. create a separate `findings/FNNN-*.md` for any nontrivial machine-level result;
5. explicitly list which older assumption is invalidated, if any;
6. keep `HANDOFF.md` synchronized whenever the immediate successor proof obligations change.
