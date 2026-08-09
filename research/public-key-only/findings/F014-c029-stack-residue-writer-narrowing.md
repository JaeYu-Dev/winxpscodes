# F014 — C-029 stack-residue writers are narrowed to specific prior machine frames

Status: **OPEN theorem / machine ancestry substantially narrowed**

## Scope

This finding records exact XP SP3 machine-level progress on C-029 without promoting the support bound beyond the evidence.

Primary target remains the pinned `rsaenh.dll 5.1.2600.5507` build. The evidence below is recovered from the historical replay repository at commit:

```text
Melik159/xp-cgr-replay
cc63079f1524b9eaced3feb97c940076181307ff
```

The relevant retained campaigns are V26, V28, and V29.

## C-029a / initialization local L1

On the normal-success provider-initialization path, exact disassembly shows:

```asm
680120b2 call 68011c4b
680120b7 test eax,eax
...
680120bb call 68026578
680120c0 test eax,eax
680120c2 jne  680120d8
680120c4 push esi
680120c5 push 28h
680120c7 lea  eax,[ebp-2Ch]
680120ca push eax
680120cb push esi
680120cc push esi
680120cd push esi
680120ce push esi
680120cf call 6800d640
```

The call to `68026578` has no caller-pushed arguments and its success result is tested before the `D640` call. Therefore the stale five-word vector observed below the later `D640` frame cannot be the outer arguments of `68026578`; its producer must lie inside that call (for example a nested call frame or equivalent stores), or in code dominated by it before return.

Across retained V28/V29 observations the pre-`SystemFunction036` 20-byte local has the form:

```text
680266c6      probable internal return-PC residue
68031998      word 1
00000000      word 2
68000000      word 3
H             word 4
00000000      word 5
```

with observed `H` values including `00156eb8` and `00154e90`. The two nearby code words `68026678` and `680266c6` both lie inside the `68026578` function region, strongly localizing the producer to that self-test call.

`SystemFunction036` subsequently overwrites exactly the five words after `680266c6`, confirming that this stale region is the actual 20-byte auxiliary destination consumed by `D640`.

The strongest current conditional model is therefore:

```text
L1 = (B + 0x31998, 0, B, H, 0)
```

and if the missing internal writer sequence is proved invariant, then conservatively allowing both `B` and `H` to range over all 32-bit words gives:

```text
|Supp(L1)| <= 2^64
```

No ASLR, alignment, fresh-install, or address-distribution assumption is needed for that conditional bound.

### Missing proof obligation

Do **not** promote C-029a to CONFIRMED yet. The retained public replay does not contain the exact internal writer instructions ending at/near return address `680266c6`.

The decisive artifact remains target-SP3 bytes or an unredacted disassembly for roughly:

```text
68026578 ... 680266c6
```

The theorem is proved only if the last-writer/use-def chain establishes the five-word shape on every normal-success path reaching the initialization `D640` call.

## C-029b / bridge local L2

V29 records the second useful `D640` auxiliary local before `SystemFunction036` as:

```text
L2 = (0x10, 0, 0x00154668, 0x0013fbdc, 0x8)
```

A new exact-machine correction is important here. V26 contains the complete wrapper at `6800d74b`:

```asm
6800d74b mov  edi,edi
6800d74d push ebp
6800d74e mov  ebp,esp
6800d750 xor  eax,eax
6800d752 push eax
6800d753 push dword ptr [ebp+18h]
6800d756 push dword ptr [ebp+14h]
6800d759 push eax
6800d75a push eax
6800d75b push dword ptr [ebp+10h]
6800d75e push dword ptr [ebp+0Ch]
6800d761 call 6800d640
6800d766 pop  ebp
6800d767 ret  14h
```

This wrapper allocates no locals. In V29 its frame geometry places the future `D640 [ebp-18h]` auxiliary region below the stack pointer that existed before the wrapper's five arguments were pushed. Consequently L2 is **not** a wrapper local and should not be modeled as one. It is older dead-stack residue left by a deeper prior call in the `6800fac7` ancestry.

This makes L2 mechanistically more similar to L1 (stale prior-call residue), but it does not yet prove the five-word support family universally.

### Falsified sub-hypothesis

The adjacent word `7c97b140` must not be treated as an x86 return address. For XP SP3 `ntdll.dll` it falls in the module data region, not the executable text region. Therefore the tempting interpretation of the surrounding L2 bytes as a six-argument frame beginning at `7c97b140` is rejected.

### Missing proof obligation

Recover the exact prior call inside the ancestry ending at:

```text
D640 -> 6800d766 -> 6800fac7 -> 6800fb7a
```

that last-writes the future L2 destination, then derive its machine-word support. A second independent pre-`SystemFunction036` capture of the same bridge path should be used as a falsifier before any `<= 2^64` universal L2 claim is promoted.

## Reachable-image consequence

This finding does **not** prove a public-key-only key-space collapse. It only narrows two ancillary RSAENH stack-prehistory roots.

Current safe status:

```text
L1: very strong <=2^64 conditional machine-family candidate
L2: strong structured stale-residue candidate; universal support bound still OPEN
```

The next highest-value proof remains the exact internal writer for `680266c6`. After that, the highest-value joint question is whether the L2 heap/stack words share ancestry with the L1 machine words, rather than treating two nominal 64-bit families as independent.
