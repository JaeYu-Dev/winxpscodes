# F010 — CGR64 second-round local is SELF history

Status: CONFIRMED MACHINE-DATAFLOW
Version: v0.1.8
Date: 2026-08-09

## Statement
Within one shipped XP SP3 RSAENH 64-byte provider invocation, the second 40-byte round reuses the same 20-byte local `[ebp-0x18]` as the first round. It does **not** introduce a second independent stack-prehistory root.

This is a narrow provenance result. It does not remove the fresh `SystemFunction036` output of round two and it does not claim that the first-round local prehistory is known.

## Target machine path
The validated RSAENH core loops inside the same `D640` stack frame when output remains:

```asm
6800d717  add [ebp-48h],esi   ; out += n
6800d71a  sub [ebp-44h],esi   ; remaining -= n
6800d71d  jne 6800d681        ; second round in same frame
```

The normal runtime branch then computes only the caller-XOR length:

```asm
push 14h
pop  edi
cmp  [ebp-44h],edi
jae  ...
mov  edi,[ebp-44h]
```

and immediately reuses the same local as the next `SystemFunction036` destination:

```asm
6800d693  push 14h
6800d695  lea  eax,[ebp-18h]
6800d698  push eax
6800d699  call SystemFunction036
```

The nonzero auxiliary-override branch is a separate self-test mechanism. Production runtime uses the normal branch.

## First-round last writer before loop-back
After the first `SystemFunction036` succeeds, the same local is mixed with the caller output-buffer prefix and passed through the helper path before the provider state transition. Representative instructions include:

```asm
6800d6ab  lea eax,[ebp-18h]
...
6800d6b4  mov dl,[esi+eax]
6800d6b7  xor [eax],dl
...
6800d6be  lea eax,[ebp-18h]
6800d6c1  push eax
6800d6c2  call 6800d544
```

Therefore, on the second loop entry, the pre-call contents of `[ebp-0x18:ebp-0x4]` are descendants of the first round rather than a new unrelated stack allocation.

## Reachability algebra
Let `L0` denote the first-round local prehistory, `R0` the first `SystemFunction036` result, and `B0` the first caller-prefix contribution. The exact helper details can remain abstract:

`L1 = F(L0, R0, B0, helper_state)`.

The key result is not that `L1` is predictable; it is that no new independent stack-prehistory variable `U1` is introduced between rounds:

`fresh_root_stack(round2 | round1 history) = 0`.

For one `CryptGenRandom(64)` call, do **not** model two independent 160-bit stack-prehistory roots merely because two `SystemFunction036(20)` calls occur.

## Public-key-only relevance
This removes one nominal ancestry root per 64-byte RSAENH call from the reachable-image model. It is a dependency reduction, not a complete entropy/search bound. The second round still depends on fresh/upstream ADVAPI/KSecDD state and on the first-round history.

## Scope / falsifier
This finding would need revision if exact shipped-SP3 code showed an intervening store that reinitializes `[ebp-0x18:ebp-0x4]` on the production `D681 -> D693` path. The recovered exact write-set audit and control-flow reconstruction found no such store.

It makes no claim about the last writer of `[ebp-0x18]` before the **first** round of a `D640` invocation; that remains part of C-025/C-029 provenance work.