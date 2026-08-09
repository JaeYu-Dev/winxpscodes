# F010 — RSAENH CGR64 second-round local is SELF history

Status: **CONFIRMED MACHINE-DATAFLOW**

## Claim

Within one shipped XP SP3 RSAENH 64-byte generation call, the second provider transition does **not** introduce a second independent 20-byte stack-prehistory root.

## Target

- Windows XP SP3
- `rsaenh.dll` 5.1.2600.5507
- runtime provider helper around `rsaenh+0xD640`
- local auxiliary buffer `[ebp-0x18 : ebp-0x04]`

## Relevant control flow

The provider calls SystemFunction036 with the 20-byte local:

```asm
6800d693  push 14h
6800d695  lea  eax,[ebp-18h]
6800d698  push eax
6800d699  call SystemFunction036
6800d69e  test al,al
6800d6a0  je   failure
```

The returned 20 bytes are then mixed with the current caller-buffer prefix in the same local. A 40-byte provider block is generated and copied. If bytes remain, the same invocation loops:

```asm
6800d717  add [ebp-48h],esi
6800d71a  sub [ebp-44h],esi
6800d71d  jne 6800d681
```

For a 64-byte request, the first loop emits 40 bytes and the second emits the remaining 24 bytes.

The second loop uses the same EBP and the same local address `[ebp-0x18]`. Recovered exact write-set analysis found no reinitializing store of that 20-byte local between the first-round post-mix state and the second `SystemFunction036` call.

## Data-flow consequence

Let `L0` be the local contents before the first SystemFunction036 call, `R0` its output, and `B0` the caller prefix mixed after that call. After round 0 the same local contains a value determined by the first-round history:

```text
L1 = f(R0, B0, helper/provider state)
```

The second loop does not source a new unrelated stack region. Therefore:

```text
fresh_root(L1 | round0 history) = 0
```

This is a dependency statement, not an assertion that `L0` itself is constant or predictable.

## Reachability relevance

Do not model a single CGR64 request as two independent 160-bit stack-prehistory roots. The second local belongs to the SELF/history class.

This removes a nominal independent variable from every validated 64-byte RSAENH request, but it does **not** by itself bound the first-round local or the final private-key image below 2^128.

## What this finding does NOT claim

- It does not prove the first-round local is constant.
- It does not prove SystemFunction036 output is predictable.
- It does not turn the 64-byte provider output into a 160-bit total-state theorem by itself.
- It does not prove a Public-Key-Only key break.

## Remaining related obligation

Recover the exact last writers of the first relevant local/pre-call bytes and classify them as `CONST | SELF | DERIVED | DELTA | FOREIGN`.
