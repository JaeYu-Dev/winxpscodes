# F007 — Early SystemFunction036 outputs consume only six KSecDD rekey roots

Status: CONFIRMED BY SP3 TRACE + KSA REPLAY; composed to the standard first-key call sequence
Version: v0.1.5
Date: 2026-08-09

## Statement
In the V17 XP SP3 trace, the first four useful `SystemFunction036(20)` outputs are generated from ADVAPI RC4 entries as follows:

- output #1: entry1, freshly keyed from KSecDD rekey/IOCTL #1;
- output #2: the **same entry1**, continuing the same RC4 stream from byte 20 to byte 40;
- output #3: entry2, freshly keyed from KSecDD rekey/IOCTL #2;
- output #4: entry3, freshly keyed from KSecDD rekey/IOCTL #3.

Writing `M_i` for the KSecDD-derived material that initializes the corresponding cached ADVAPI RC4 entry and `KS(M)[a:b]` for the RC4 keystream segment, the output-side ancestry is:

`R1 <- KS(M1)[0:20]`

`R2 <- KS(M1)[20:40]`

`R3 <- KS(M2)[0:20]`

`R4 <- KS(M3)[0:20]`.

The SystemFunction in/out pre-buffer XOR is a separate dependency and is not erased by this finding; F007 concerns the independent KSecDD/RC4-key roots only.

## Raw evidence
V17 report:

- 8 KSecDD/ADVAPI IOCTL rekeys;
- 11 RC4 PRGA invocations;
- 4 SystemFunction036 invocations.

Useful PRGA mapping:

- PRGA #1: state pointer `0025ba84`, len=20, `i:0 -> 20`, output == SystemFunction036 #1;
- PRGA #9: state pointer `0025ba84`, len=20, `i:20 -> 40`, output == SystemFunction036 #2;
- PRGA #10: state pointer `0025bba4`, len=20, output == SystemFunction036 #3;
- PRGA #11: state pointer `0025bcc4`, len=20, output == SystemFunction036 #4.

KSA replay scan independently maps:

- PRGA #1 fresh state to IOCTL #1 output;
- PRGA #10 fresh state to IOCTL #2 output;
- PRGA #11 fresh state to IOCTL #3 output.

PRGA #9 is deliberately not a fresh-KSA match: it is the already-advanced entry1 state, as shown by the same state pointer and RC4 index continuation.

## Composition to the modeled first Bitcoin key path
The standard first-key path previously fixed for OpenSSL 0.9.8h / RSAENH SP3 contains seven `SystemFunction036(20)` consumptions before the first scalar draw:

- first RAND_poll/provider use: provider init 1 + CryptAcquireContext bridge 1 + CGR64 runtime 2 = 4;
- second RAND_poll: acquire bridge 1 + CGR64 runtime 2 = 3;
- total = 7.

After the startup transient, all eight ADVAPI entries have been keyed and the selector continues sequentially. Therefore, under this exact call sequence and absent an extra intervening SystemFunction036 consumer, the seven output roots are:

`[M1, M1, M2, M3, M4, M5, M6]`.

Thus KSecDD rekey roots `M7` and `M8` are chronologically created during startup but are **not ancestors of the seven Windows RNG outputs consumed on this first-key path**.

## Reachability consequence
A naive chronology-based model would count eight KSecDD rekey roots as relevant before useful generation. F007 proves that the early output dependency graph is narrower:

`8 created roots -> 6 consumed roots before first-key Windows RNG path`,

with the first root reused for two consecutive outputs.

This is a structural ancestry reduction, not yet an entropy-bit bound: the joint support of `M1..M6` remains open and may share additional KSecDD persistent-state ancestry.

## Falsifier / scope boundary
This composition must be revised if an exact Bitcoin trace shows an additional `SystemFunction036` consumer between the modeled seven calls, or a selector reset/reinitialization before the first key. The V17 root mapping itself remains valid for that observed SP3 process execution.

## Next
1. Trace whether `M2..M6` are independent kernel roots or a deterministic ratchet from one persistent kernel state root.
2. Close the D640 within-CGR64 `[ebp-18h]` loop-carried prestate proposition.
3. Compose the six-root Windows image into the two OpenSSL RAND_poll state transitions rather than treating the six roots as independent entropy credits.
