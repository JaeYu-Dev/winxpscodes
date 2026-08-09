# F004 — First eight first-use rekey inputs share one 128-bit MD4 root
Status: SOURCE-THEOREM; FIRST SNAPSHOT SP3 BIT-EXACT CONFIRMED; SNAPSHOTS 2..8 OPEN
PK-only effect: potentially collapses the entire user-side first-use ADVAPI rekey-input family to <=128 bits.

## Preconditions
- This is the process's first circular-hash use, so the static/BSS `CircBuf[256]` starts zero.
- F003 source control flow holds: the first eight first-use rekey passes emit zero bytes.
- No concurrent caller mutates the shared circular hash during the burst.

## Source algebra
Let the unchanged 20-byte caller buffer be `B` and:

`D = MD4(B)`  (16 bytes = 128 bits).

The circular hash uses no feedback, a 256-byte buffer, and advances its XOR start position by 7 bytes per update. Because each first-use pass outputs zero bytes, `B` is unchanged across passes 1..8.

Define `E_p(D)` as XOR-embedding the 16-byte digest D into the 256-byte circular buffer starting at byte offset p modulo 256. Then:

`C_0 = 0`

`C_j = C_{j-1} XOR E_{7(j-1)}(D)`, for j=1..8.

Hence every first-use KSecDD rekey input snapshot `(C_1,...,C_8)` is a deterministic function of a **single 128-bit value D**:

`|(C_1,...,C_8) image| <= 2^128`.

This is not `8 * 128` independent bits. The apparent 8 x 256-byte user-side rekey inputs belong to one 128-bit family under the preconditions.

## First-snapshot SP3 validation
A prior V17 SP3 trace captured pre-call B:

`98 19 03 68 00 00 00 00 00 00 00 68 00 ae 25 00 00 00 00 00`

and the first 256-byte KSecDD IOCTL input:

`59 ca 1e 7a f9 c2 7a 9a 31 43 d4 ef 10 71 d2 f8 || 00 x 240`.

Independent MD4(B) is exactly:

`59ca1e7af9c27a9a3143d4ef1071d2f8`.

Therefore `C_1 = MD4(B) || 0^240` is bit-exact confirmed for that SP3 trace.

## Kernel composition
For each rekey j, KSecDD's REKEY_ONLY path evolves its own persistent state independently of the user seed, then RC4-transforms the METHOD_BUFFERED system buffer in place. Conceptually:

`X_j = F_kernel(X_{j-1}, P_j)`

`M_j = C_j XOR KS_RC4(X_j)[0:256]`.

Thus F004 removes user-side input independence but does **not** remove the kernel roots `(X_0,P_1,...,P_8)`.

## Next decisive validation
Recover V17 IOCTL input snapshots C2..C8. Starting only from the known D above, replay the 7-byte-offset circular updates and compare all 256 bytes for each snapshot.

If all eight match, the ADVAPI user-side first-use dependency is experimentally closed to one 128-bit root for that execution.

## Falsifiers
- any output byte modifies B before all eight first-use rekeys complete;
- another circular-hash update interleaves;
- shipped SP3 uses a different digest/offset/circular state;
- C2..C8 disagree with the one-D replay.