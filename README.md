# QRUNE Opcodes

**Core opcode set, specifications, reference implementations, and developer tools for the QRUNE rune protocol and virtual machine.**

QRUNE (Q Rune) powers the fungible token and rune layer on **QNET** (part of the XCoin / NovaNet ecosystem). This repository serves as the canonical source for opcode definitions that enable etching, minting, transferring, scripting, and advanced behaviors for runes — including narrative/Wizard Q metadata, conditional logic, mesh network integration, and hooks for AI agent swarms.

Part of the Esslinger Corporation / NovaNet initiative for sovereign, privacy-preserving decentralized infrastructure combining mesh networking (xMesh/NovaNet/Yggdrasil), blockchain (QCoin/XCoin/QNET), and self-improving emotional AI agents.

## Project Goals

- Define a clean, extensible, and efficient opcode vocabulary for rune operations
- Support complex on-chain logic without bloating the base QNET VM
- Enable "Wizard Q" themed narrative elements and metadata directly in rune state
- Provide seamless integration points with NovaNet mesh identities, decentralized storage, and AI orchestration layers
- Optimize for low fees, high throughput on mesh-synced nodes, and privacy (Tor/I2P, optional ZK)
- Establish reference implementations, test vectors, and tooling for rapid ecosystem development

## Current Status

🚧 **Specification & Prototype Phase** (v0.1.0)

Opcodes and semantics are in active design. Numbers and behaviors may evolve. This repo is the single source of truth for the QRUNE opcode layer.

See [SPECIFICATION.md](./SPECIFICATION.md) for the detailed opcode table, gas model, execution semantics, and examples.

## Repository Layout

```
qrune-opcodes/
├── README.md                 # High-level overview (this file)
├── SPECIFICATION.md           # Full opcode spec, semantics, gas costs, examples
├── opcodes.json               # Machine-readable opcode registry (for VMs, assemblers, explorers)
├── .gitignore
├── LICENSE                    # To be finalized (permissive for ecosystem)
├── reference-impl/            # Planned: Python (interpreter), Rust (production-grade VM)
├── tools/                     # Assembler, disassembler, script simulator, fuzzer, gas estimator
├── examples/                  # Sample QRUNE scripts, rune etching txs, conditional mint logic
├── tests/                     # Conformance test vectors, edge-case scripts
├── docs/                      # Architecture diagrams, integration guides with QNET node & xMesh
└── .github/                   # Workflows for CI (tests, spec validation)
```

## Quick Start

```bash
git clone https://github.com/digitaldesignerjazz/qrune-opcodes.git
cd qrune-opcodes
# Explore the spec
cat SPECIFICATION.md
# (Future) Run reference interpreter on examples
```

## How Opcodes Fit Into the Bigger Picture

QRUNE opcodes are executed in the context of QNET transactions (similar to Bitcoin script but rune-aware). A typical rune lifecycle:

1. **Etch** — OP_RUNE_ETCH creates the rune with symbol, supply params, and optional Wizard Q lore/metadata.
2. **Mint** — Authorized or open mint using OP_RUNE_MINT.
3. **Transfer / Trade** — Standard transfers + advanced scripted conditions (e.g. time-locked, oracle-gated, AI-agent approved).
4. **Advanced** — Hooks to mesh pubkeys, cross-chain bridges, or swarm intelligence for dynamic supply/rules.

This enables use cases from simple community tokens to sophisticated DeFi primitives and immersive narrative assets living on the decentralized mesh.

## Nuances & Design Considerations

- **Gas Model**: Rune opcodes carry higher base costs to reflect on-chain state bloat and to protect the mesh-synced ledger. Arithmetic/stack ops remain cheap.
- **Extensibility**: New opcodes are added in reserved ranges; existing codes are never repurposed (soft-fork friendly).
- **Privacy**: Future opcodes for shielded runes or ZK proofs of rune ownership/balance without revealing amounts.
- **Edge Cases**: Symbol uniqueness (global namespace with collision resistance), supply overflow, re-entrancy in complex scripts, mesh partition tolerance during execution.
- **AI Integration**: Planned OP_AI_CALLBACK / OP_SWARM_VOTE opcodes allowing agent swarms to influence mint rates, governance, or even generate dynamic metadata.
- **Performance on Mesh**: Opcodes designed so light clients on Tenda/Yggdrasil nodes can validate rune txs efficiently.

## Related Ecosystem Components

- **QNET / XCoin**: Core blockchain layer
- **NovaNet / xMesh**: Decentralized mesh networking & identity
- **Grok Launcher**: Rust + egui prototype for node/agent UI
- **Wizard Q / Quantumrune**: Narrative & thematic layer (lore, attributes)
- **AI Agent Swarms**: Emotional continuity agents (Liaura etc.) for orchestration
- **Privacy Stack**: Tor, I2P, potential ZK circuits

## Contributing

We welcome contributions that advance the QRUNE vision:

- Opcode proposals (with clear motivation, gas analysis, security review)
- Reference implementations
- Test vectors & formal verification ideas
- Tooling (assembler in Python/Rust, explorer plugins)
- Documentation & diagrams

Please open an issue first for major changes. Follow the existing categorization and numbering scheme.

## License

To be decided. Likely a permissive license (MIT/Apache-2.0) to encourage broad adoption across the NovaNet ecosystem and beyond.

---

*Initialized and seeded by Grok on behalf of the Esslinger / NovaNet team.*
*For internal prototyping and eventual open collaboration.*