# CovenantClock: Trustless Time-Aware Contracts on Bitcoin

**CovenantClock** is a modular Bitcoin covenant structure that combines the power of deterministic time coordination with expressive, verifiable contract logic - all enforced on Bitcoin Layer 1.

This system brings together:

- **Epoch Clock**: A shared, trustless time anchor using confirmed transaction timestamps and fixed durations.
- **BitVM**: A logic proving framework using Taproot fraud proofs for conditional paths.
- **Taproot Trees**: Enforceable logic paths based on time or computation.
- **PSBT Coordination**: Collaborative transaction assembly using standard Bitcoin tooling.

>  **Design Philosophy**
> - No new opcodes
> - No soft forks
> - Minimal assumptions
> - Taproot-compatible
> - Everything is auditable and enforced by existing Bitcoin primitives

---

##  What It Enables

### Why This Is Important

Bitcoin block timestamps are subject to minor 'drift' — they're allowed to vary slightly from real-world time within consensus rules. This typically introduces a small timing uncertainty (up to a few minutes), which historically made precise coordination difficult. CovenantClock neutralises drift by anchoring time only once, at the moment the Epoch Clock's defining transaction is confirmed. That single block timestamp becomes the shared reference for all parties. No further sync or wall-time assumptions are required - all contract logic compares only to on-chain time. This preserves trustlessness and avoids cumulative drift effects.

Bitcoin has lacked a shared, native way to coordinate contract expiries or enforce time-based logic — until now. Prior attempts required off-chain coordination, trusted oracles, or application-specific logic.

CovenantClock solves this by introducing a global, deterministic time anchor (the Epoch Clock), paired with expressive, trustless execution via Taproot paths and BitVM. This unlocks an entire design space:

- Enforce deadlines and challenge periods on Bitcoin Layer 1
- Enable multi-party coordination without central servers
- Minimise trust and attack surfaces

It brings Bitcoin closer to having composable, interlocking covenant systems - all enforceable on-chain, without protocol changes.

>  Note: The term 'contract' here refers to enforceable spending conditions and Taproot paths coordinated by time, not off-chain legal agreements.

CovenantClock allows trustless coordination of Bitcoin-native covenants (or 'contracts' if you prefer broader terminology), enabling:

- Trustless loan expiries: Lenders and borrowers can define a shared loan period that auto-expires using on-chain time.
- Vault unlock conditions: Access to funds can shift to a different key after a set duration.
- Sealed-bid auctions: Automatically enforce closing times without trusted servers.
- BitVM fraud challenge periods: Enforce bounded logic windows for proofs or fallback.
- Escrows or insurance payout triggers after delays.
- Modular coordination between independent covenant systems, using one shared clock anchor.

All without:
- Soft forks
- New opcodes
- Off-chain trust assumptions



---

## How They Combine in CovenantClock

### Epoch Clock defines **when**
- Start and end times for a contract (e.g. loans, auctions, challenge periods).
- Time-gated branches in Taproot trees (e.g. one branch for logic, one branch for timeout fallback).
- Acts like a global clock shared across protocols.

### BitVM defines **what** *(execution model relies on a prover-verifier game, enforced through Taproot spending conditions)*
- What logic runs inside the valid time window (e.g. verify computation, unlock with condition).
- Allows rich branching logic or fraud checks during the clock’s active period.
- Provides expressive conditional enforcement — Bitcoin-native and trustless.

---

## 📄 Epoch Clock Spec

The Epoch Clock used in this system is inscribed as an Ordinal:
🔗 [https://ordinals.com/inscription/96733659](https://ordinals.com/inscription/96733659)

Each covenant or spending condition refers to a standardised JSON inscription or embedded object with:

```json
{
  "timer_id": "utc-epoch-100yr",
  "start_timestamp": 1748400000,
  "duration_seconds": 3153600000,
  "format": "unix_seconds",
  "notes": "This is a 100-year decentralized UTC time reference beginning on 2025-05-25 00:00:00 UTC. It is not connected to any oracle or server, and can be used to anchor start and expiry times for contracts including loans, auctions, escrows, or scheduled events. Inscribed by Rosie for trustless time coordination in Bitcoin-native DeFi."
}
```

- **start_timestamp**: Block time of the defining transaction (UTC)
- **duration_seconds**: Time period before expiry (in seconds)
- **format**: Always `unix_seconds`

---

##  Spending Condition Flow Example

Here’s a simplified Taproot script branch example using a time-gated covenant:

```asm
<pubkey> CHECKSIGVERIFY
<epoch_expiry_timestamp> CLTV
DROP
<logic_or_unlock_condition>
```

## Alternative fallback branch (post-expiry path)
```asm
<timeout_pubkey> CHECKSIG
```

This pattern enforces:
- The covenant can only be spent before or after the `CLTV` timestamp
- Different Taproot branches allow either fraud-proof logic or fallback

While a full BitVM integration would include a challenge/response tree, this example shows basic L1 enforcement using Epoch Clock time. 

###  BitVM Challenge Coordination in CovenantClock

In CovenantClock, BitVM’s challenge-response logic is embedded as a Taproot script branch available during the active window defined by the Epoch Clock.

BitVM branches assume pre-committed traces and challenge-response windows enforced by `OP_CHECKSIG` gates and script branches in the Taproot tree. The prover and verifier construct PSBTs cooperatively:

- The prover precommits to a computation trace via hash
- The verifier can challenge a step during the clock’s active period
- Fraud proofs are resolved on-chain via Taproot spend paths

>  Multistage BitVM challenges must complete within the active clock window, or fall back via a timeout path

This enables contract enforcement without trust or servers, with the Epoch Clock acting as a bounded time window for interactive verification.

1. **Clock Set**: A confirmed transaction inscribes or references the Epoch Clock.
2. **Taproot Tree Built**:
   - Branch A: Valid before expiry, runs a BitVM fraud-proof logic
   - Branch B: Activates after expiry via `OP_CLTV`
3. **User signs and broadcasts** using PSBTs. using PSBTs.


---

## Taproot Tree Example Diagram

Below is a basic conceptual layout for a Taproot tree using CovenantClock:

```
Taproot Root
├── Branch A (active before expiry)
│   └── BitVM challenge logic
└── Branch B (active after expiry)
    └── Timeout fallback key
```

## Repository Overview

- `epoch_clock_spec.md` — Canonical format and usage rules
- `bitvm_branch_template.md` — Sample BitVM script branch
- `taproot_tree_examples/` — Ready-to-inscribe PSBT contract examples
- `scripts/` — Time parsing, Taproot tree builders
- `integration_guides/` — How to adopt CovenantClock in your Bitcoin-native app

---

## Get Started

CovenantClock is fully trustless - all logic and coordination rely solely on Bitcoin Layer 1 primitives. There are no oracles, no servers, and no external dependencies.

CovenantClock is a standalone contract coordination system that builds on top of the Epoch Clock. If you're looking for the time standard that anchors these contracts, see the original Epoch Clock repo:

🔗 [https://github.com/rosieRRRRR/The-Epoch-Clock](https://github.com/rosieRRRRR/The-Epoch-Clock)


If you're building covenants or contract-like mechanisms that:
- Expire
- Need a challenge window
- Require shared time without trusting servers

...CovenantClock gives you the coordination standard to do it all natively on L1.

---

##  License
MIT

---

**Conceived and developed by Rosie (@rosiea)** — creator of the Epoch Clock and originator of the CovenantClock concept.

---

## 📖 Further Reading

CovenantClock builds directly on the [Epoch Clock](https://github.com/rosieRRRRR/The-Epoch-Clock) standard. The full documentation for the Epoch Clock is included below for context, integration, and extension.

---

### Epoch Clock Documentation

> The Epoch Clock is a deterministic, decentralised time reference system for Bitcoin-native contracts. It enables trustless time coordination using only on-chain data — no oracles, no servers, no external clocks.

```json
{
  "timer_id": "utc-epoch-100yr",
  "start_timestamp": 1748400000,
  "duration_seconds": 3153600000,
  "format": "unix_seconds",
  "notes": "This is a 100-year decentralized UTC time reference beginning on 2025-05-25 00:00:00 UTC. It is not connected to any oracle or server, and can be used to anchor start and expiry times for contracts including loans, auctions, escrows, or scheduled events. Inscribed by Rosie for trustless time coordination in Bitcoin-native DeFi."
}
```

For deeper context and use cases, visit the full repo:  
🔗 [https://github.com/rosieRRRRR/The-Epoch-Clock](https://github.com/rosieRRRRR/The-Epoch-Clock)

---

### BitVM Reference

CovenantClock leverages ideas and logic branches from the BitVM framework originally proposed by Robin Linus.

> BitVM enables arbitrary computation to be verified on Bitcoin through Taproot fraud proofs and challenge-response execution.

Original repo:  
🔗 [https://github.com/RobinLinus/bitvm](https://github.com/RobinLinus/bitvm)

**Credit to Robin Linus and the contributing developers** for creating BitVM — a foundational component enabling conditional execution within CovenantClock.

