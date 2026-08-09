# F013 — Existing registry Seed is read; universal zero-state initialization is false

Status: **CONFIRMED EXACT-SP3 RECOVERED WORK**

## Claim

The attractive hypothesis

```text
ReadSeed failure/bug => every XP SP3 boot starts KSecDD with X0 = 0^640
```

is false for the pinned exact-SP3 implementation.

Recovered exact-SP3 analysis of `_ReadSeed@8` shows that an existing 80-byte `REG_BINARY` Seed is read into the kernel RNG state. The correct high-level branch model is:

```text
Seed exists  -> X0 = persisted 80-byte Seed/state
Seed absent  -> zero-initialized global branch remains possible
```

The zero state is therefore a reachable branch, not a universal fact.

## Reachability relevance

This is an important negative result. A universal Public-Key-Only collapse cannot assume `X0=0`.

The major remaining question becomes the **reachable family of persisted X0 values** under setup, boot, rekey, and writeback transitions. The 80-byte state width alone is not an entropy proof, but neither can it be discarded.

## Next proof obligations

1. Reconstruct setup/boot-time transitions that create the first persisted Seed.
2. Characterize writeback/ratchet mapping between old and new persistent states.
3. Test clone/image scenarios separately from the universal path; shared images may restrict the reachable family but cannot be assumed in the general theorem.
4. Compose only exact ancestors that survive into the first Bitcoin scalar.

## Do not regress

Do not revive `X0=0` as a default or universal assumption without a target-specific missing-Seed proof.
