# F013 — Existing 80-byte registry Seed is read; universal zero-state initialization is false

Status: CONFIRMED EXACT-SP3 RECOVERED WORK
Version: v0.1.8
Date: 2026-08-09

## Statement
The universal model `KSecDD initial state X0 = 0^640` is false for the target XP SP3 path. When the 80-byte persistent RNG `Seed` value exists and is successfully read, it enters the kernel RNG state. Zero state is the missing/read-failure branch, not a universal initialization fact.

## Why this matters
An earlier source-level observation was that a static/global 80-byte state is zero-initialized and a failed `ReadSeed` can leave it zero. That fact is real but insufficient for a universal Public-Key-Only model.

Recovered exact-SP3 work establishes the existing-Seed branch as reachable and functional. Therefore the first-key reachable family must include persistent prehistory carried through the registry ratchet.

## Reachability consequence
The initial KSecDD root must be modeled as a reachable family:

`X0 in R_seed`,

not as a fixed all-zero constant.

This is currently one of the largest blockers to a universal Public-Key-Only enumeration theorem. Conditional clone/rollback/missing-Seed scenarios can still be studied separately, but they must not be substituted for the all-reachable model.

## Persistence caution
The registry value is not simply equivalent to the prior live state in every step; the historical randlib persistence path transforms state during writeback. A separate recurrence analysis is required to bound `R_seed` across setup/boot/runtime history.

## Falsifier / scope
This finding would need revision only if exact target-SP3 evidence showed that the relevant first-key path bypasses an existing Seed or overwrites it with a fixed value before any consumed KSecDD output. No such universal bypass is established.

## Next
Characterize the reachable family of `X0` under real XP setup, boot, read/writeback, missing-Seed, failure, rollback, and clone branches. Keep universal and conditional attack classes separate.