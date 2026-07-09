# Nexus Cog

The [Nexus Cognitive Technologies](https://github.com/nexus-cognitive-tech)
cognitive stack — composable engines that turn any agent into a
memory-bearing, reasoning, self-correcting system.

## Engines

| Crate | Description |
|---|---|
| [nexus-cog-core](https://github.com/nexus-cognitive-tech/nexus-cog-core) | Foundation: types, storage, config |
| [nexus-cog-storage](https://github.com/nexus-cognitive-tech/nexus-cog-storage) | SQLite persistence (rusqlite) |
| [nexus-cog-embeddings](https://github.com/nexus-cognitive-tech/nexus-cog-embeddings) | Pluggable text encoders + vector store |
| [nexus-cog-neural](https://github.com/nexus-cognitive-tech/nexus-cog-neural) | Brain-like cortex — spikes, six-layer columns, astrocyte network, neurogenesis, sensorimotor loop |
| [nexus-cog-causal](https://github.com/nexus-cognitive-tech/nexus-cog-causal) | Causal reasoning: forward, backward, counterfactual, pre-mortem, blast radius |
| [nexus-cog-patterns](https://github.com/nexus-cognitive-tech/nexus-cog-patterns) | Pattern library: builtin patterns, matcher, relevance scorer |
| [nexus-cog-provenance](https://github.com/nexus-cognitive-tech/nexus-cog-provenance) | Artifact provenance: lineage, graph queries, SHA-256 |
| [nexus-cog-antifragile](https://github.com/nexus-cognitive-tech/nexus-cog-antifragile) | Adversarial input generation, edge-case exploration, robustness scoring |

## Interfaces

| Crate | Description |
|---|---|
| [nexus-cog-cli](https://github.com/nexus-cognitive-tech/nexus-cog-cli) | Command-line interface — every brain + orthogonal engine is a subcommand; `nexus-cog mcp` exposes the same surface over Model Context Protocol |

## Cloud

- [nexus-cog-cloud](https://github.com/nexus-cognitive-tech/nexus-cog-cloud) — **private** — managed sync, telemetry, billing

## Quick start

```sh
# Build and run the CLI
cargo install --git https://github.com/nexus-cognitive-tech/nexus-cog-cli

# Open an interactive workspace
nexus-cog --workspace ./myproject palace summary

# Run the brain-like cortex
nexus-cog --workspace ./myproject cognitive think "implement LRU cache"
nexus-cog --workspace ./myproject intent check auth --current-code 'fn verify(u: &str, p: &str) -> bool { u == "admin" && p == "hunter2" }'

# Run one sleep cycle (NREM replay + REM re-activation)
nexus-cog --workspace ./myproject decay

# Expose the same tools over MCP
nexus-cog mcp
```

## License

Most components are Apache-2.0. `nexus-cog-cloud` is proprietary.
