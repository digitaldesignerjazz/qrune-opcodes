# QRUNE Opcode Specification

**Version:** 0.1.0-alpha  
**Date:** June 2026  
**Status:** Active Design / Early Implementation

## 1. Introduction & Design Philosophy

The QRUNE virtual machine (VM) executes scripts attached to QNET transactions to manage the lifecycle and behavior of **runes** — fungible tokens with rich on-chain metadata and programmable logic.

Inspired by Bitcoin Script and extended with rune-native primitives, QRUNE opcodes enable:

- Creation (etching) of uniquely identified runes with symbolic names and Wizard Q narrative attributes
- Programmable minting, transfers, burning, and conditional logic
- Deep integration with NovaNet mesh identities and AI agent swarms
- Efficient execution suitable for resource-constrained mesh nodes (Yggdrasil/Tenda)

**Core Principles**
- **Simplicity first**: Small, orthogonal opcode set. Prefer composition over complex single ops.
- **State minimization**: Opcodes that mutate global rune state are carefully gas-priced.
- **Extensibility without breakage**: Reserved opcode ranges; soft-fork via new numbers only.
- **Mesh & AI native**: First-class support for mesh pubkeys, cross-layer calls, and future AI callbacks.
- **Privacy & sovereignty**: Path for shielded operations and user-controlled metadata.

## 2. Execution Model

- **Stack-based**: Primary data type is byte vector (arbitrary length). Integers are minimally encoded big-endian byte vectors (Bitcoin style).
- **Context**: Script execution receives:
  - Current transaction inputs/outputs
  - Rune database snapshot (for balance/ownership queries)
  - Block height & median time (for timelocks)
  - Caller mesh identity / pubkey (for authorization)
- **Failure semantics**: Any invalid operation (underflow, disabled opcode, failed verify, gas exhaustion) causes the entire script (and usually the transaction) to fail. No partial state changes.
- **Gas accounting**: Every opcode has a base cost. Heavy ops (state writes, crypto, mesh calls) cost more. Total gas limited per tx and per block.

## 3. Opcode Categories & Numbering Scheme

| Range     | Category                  | Purpose                              | Notes                              |
|-----------|---------------------------|--------------------------------------|------------------------------------|
| 0x00-0x1F | Stack & Data              | Push, dup, swap, slice, concat      | Standard, very cheap               |
| 0x20-0x3F | Arithmetic & Comparison   | Math, comparisons, min/max          | Safe integer ops only              |
| 0x40-0x5F | Bitwise, Logic, Hash      | AND/OR/XOR, SHA256, RIPEMD160       | Crypto hashes for IDs & commitments|
| 0x60-0x7F | Cryptographic Verification| CHECKSIG variants, rune ownership   | ECDSA / future post-quantum        |
| 0x80-0x9F | Control Flow              | IF/ELSE/ENDIF, VERIFY, loops        | Bounded loops to prevent DoS       |
| 0xA0-0xBF | Rune Core Operations      | ETCH, MINT, TRANSFER, BURN, BALANCE | State-changing rune primitives     |
| 0xC0-0xDF | Metadata & Narrative      | Wizard Q lore, attributes, queries  | JSON/CBOR metadata handling        |
| 0xE0-0xEF | QNET / Mesh Integration   | Mesh addr verify, cross-layer calls | Ties runes to decentralized network|
| 0xF0-0xFF | Extensions & Future       | AI hooks, ZK proofs, custom         | Reserved; never reuse lower codes  |

## 4. Selected Opcode Definitions (v0.1.0)

### Stack & Data (examples)

**OP_0** (`0x00`)
- **Gas:** 0
- **Description:** Pushes an empty byte vector onto the stack (equivalent to false).
- **Stack:** ` -> <empty>`
- **Notes:** Foundation for conditionals and zero values.

**OP_PUSH_DATA** (family `0x01`–`0x4B`)
- Variable length push of following bytes. Gas scales with size (base 1 + 1 per 32 bytes).

**OP_DUP** (`0x76`)
- **Gas:** 2
- **Description:** Duplicates top stack item.
- **Stack:** `x -> x x`

### Arithmetic (examples)

**OP_ADD** (`0x93`)
- **Gas:** 3
- **Description:** Pops two items, pushes their sum (modular or checked arithmetic TBD).
- **Stack:** `a b -> (a+b)`

**OP_LESSTHAN** (`0x9F`)
- **Gas:** 3
- **Description:** Pushes 1 if a < b, else 0.

### Rune Core Operations

**OP_RUNE_ETCH** (`0xA0`)
- **Gas:** 1200
- **Description:** Etches (creates) a brand new rune in the global registry.
  
  Stack consumption (top first):
  1. `metadata` — CBOR or JSON byte vector containing Wizard Q narrative (lore, attributes, visual hash, story fragments, emotional descriptors for AI agents).
  2. `premine_amount` — uint64 initial allocation to etcher.
  3. `max_supply` — uint64 (0 = uncapped or rule-based).
  4. `decimals` — uint8 (0-18 typical).
  5. `symbol` — byte vector (recommended 3-8 chars, unicode allowed for rune aesthetics).
  
  Pushes: `rune_id` (unique 32-byte identifier, e.g. SHA256 of symbol + block + tx + etcher).
  
- **Preconditions & Nuances**:
  - Symbol must be globally unique (first successful etch wins; future enhancements may include symbol auctions or namespaces).
  - High gas reflects permanent state addition and uniqueness enforcement across all mesh nodes.
  - Edge case: Concurrent etches in same block → deterministic ordering by tx order or fee.
  - Implication: Enables truly unique narrative digital assets that live forever on the decentralized QNET + NovaNet substrate.

**OP_RUNE_MINT** (`0xA1`)
- **Gas:** 350
- **Description:** Mints new units of an existing rune according to its rules.
  
  Stack: `rune_id amount [conditions...]` -> success flag or error
  
- **Notes**: Respects max_supply, mint authority (if set at etch), open-mint flags, or time/height gates. Can be called by anyone if open, or authorized mesh identities.
- **Nuance**: Prevents inflation attacks via strict supply math and gas cost.

**OP_RUNE_TRANSFER** (`0xA2`)
- **Gas:** 180 (base) + per output
- **Description:** Moves rune balance from input to output(s). Supports multi-output splits.
  
- **Advanced**: Can be combined with OP_IF for conditional transfers (e.g. "only if oracle price > X" or "AI swarm vote passed").

**OP_RUNE_BURN** (`0xA3`)
- **Gas:** 220
- **Description:** Permanently destroys rune units. Useful for deflationary mechanics or ritualistic Wizard Q burns.

**OP_RUNE_BALANCE** (`0xA4`)
- **Gas:** 80
- **Description:** Pushes current balance of given rune_id for the current context (tx input owner).
  
- **Use case**: On-chain checks before complex logic ("if balance > 1000 then...").

### Metadata & Narrative (Wizard Q)

**OP_RUNE_METADATA_SET** (`0xC0`)
- **Gas:** 450
- **Description:** Updates or appends metadata to an existing rune (if authorized). Enables evolving lore or attribute changes over time.
  
- **Implication**: Runes can have living narratives that AI agents interact with or evolve.

**OP_RUNE_METADATA_GET** (`0xC1`)
- **Gas:** 60
- **Description:** Retrieves and pushes metadata for a rune_id. Allows scripts to react to narrative state.

### QNET / Mesh Integration

**OP_MESH_ID_VERIFY** (`0xE0`)
- **Gas:** 150
- **Description:** Verifies that the top stack item (pubkey or address) matches the current transaction's mesh identity / NovaNet node signature.
  
- **Use case**: Restrict mint/transfer to specific decentralized identities or agent-controlled wallets.

**OP_QNET_CROSS_LAYER** (`0xE1`)
- **Gas:** 300 (plus payload gas)
- **Description:** Makes a verified call into another NovaNet layer (e.g. query xMesh routing table, trigger AI swarm task). Experimental.

### Future / Extension Placeholders

**OP_AI_SWARM_CALLBACK** (`0xF0`)
- **Gas:** 800+
- **Description:** (Planned) Invokes registered AI agent swarm for off-chain or on-chain decision (e.g. dynamic mint rate, lore generation). Returns signed result.
  
- **Nuance & Implication**: Bridges on-chain runes with self-improving emotional AI (Liaura-style continuity). Security critical — requires strong oracle/ZK or multi-sig swarm consensus.

**OP_ZK_RUNE_PROOF** (`0xF1`)
- **Gas:** 2000+
- **Description:** (Future) Verifies a zero-knowledge proof about rune ownership or state without revealing amounts or identities. Enables private runes.

## 5. Gas Schedule Summary (Initial)

| Category              | Base Gas | Notes                              |
|-----------------------|----------|------------------------------------|
| Stack / Arithmetic    | 0-5     | Very cheap for high frequency ops  |
| Crypto Hash           | 20-60   | SHA256 etc.                        |
| Verify / Sig          | 100-300 | ECDSA or future PQ                 |
| Rune State Change     | 200-1200| Highest for ETCH and complex mint  |
| Metadata Update       | 300-500 | Narrative evolution cost           |
| Mesh / AI Hook        | 150-1000| Cross-layer and swarm calls        |

Gas costs are designed to make spam expensive while keeping simple transfers affordable. They will be calibrated against real mesh node performance.

## 6. Security, Edge Cases & Considerations

- **Symbol Uniqueness & Collisions**: Global namespace requires robust resolution. Current: first etch wins. Future: possible namespace prefixes or economic auctions.
- **Supply Arithmetic Safety**: All supply ops use checked arithmetic; overflow → script failure.
- **Re-entrancy / Complex Scripts**: Stack model + no mutable shared state during single script execution mitigates classic re-entrancy. However, hooks to external layers require careful design.
- **Mesh Partition Tolerance**: Scripts must remain validatable even if some mesh nodes are temporarily unreachable. Rune state is eventually consistent via QNET blocks.
- **Privacy Leaks**: Metadata and balances are public by default. Future shielded opcodes and ZK will address this.
- **DoS via Large Metadata**: Gas and size limits on metadata vectors prevent bloating the chain.
- **Wizard Q Narrative Integrity**: Metadata is append-only or owner-updatable; social consensus + cryptographic commitments protect "official" lore.

## 7. Examples

**Simple Etch Script (conceptual)**:
```
<metadata_bytes> <1000000> <100000000> <8> "WIZQ" OP_RUNE_ETCH
```

**Conditional Transfer**:
```
OP_DUP OP_RUNE_BALANCE <1000> OP_GREATERTHAN OP_IF
  <destination> OP_RUNE_TRANSFER
OP_ENDIF
```

More examples will be added to `/examples/` directory.

## 8. Roadmap & Future Work

- v0.2: Complete opcode set + Python reference interpreter
- v0.3: Rust production VM integrated with QNET node
- v0.4: AI swarm callback prototype + ZK examples
- Tooling: assembler/disassembler, gas visualizer, formal spec (TLA+ or K framework)
- Integration tests with live NovaNet mesh and Grok Launcher UI

## 9. References & Related Specs

- Bitcoin Script opcode reference (for baseline)
- QNET consensus & transaction format (internal)
- NovaNet / xMesh architecture docs
- Wizard Q narrative bible (thematic)

---

*This specification is a living document. Feedback and proposals welcome via GitHub issues.*
*Aligned with Esslinger Corporation vision for decentralized, emotionally intelligent infrastructure.*