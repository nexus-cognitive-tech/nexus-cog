# Nexus Cog

The [Nexus Cognitive Technologies](https://github.com/nexus-cognitive-tech) cognitive stack — composable engines that turn any agent into a memory-bearing, reasoning, self-correcting system.

## Engines

| Crate | Description |
|---|---|
| [nexus-cog-core](https://github.com/nexus-cognitive-tech/nexus-cog-core) | Foundation: types, storage, config |
| [nexus-cog-palace](https://github.com/nexus-cognitive-tech/nexus-cog-palace) | Memory palace: SQLite + FTS5 + vector recall + decay + multi-namespace |
| [nexus-cog-brain](https://github.com/nexus-cognitive-tech/nexus-cog-brain) | Static analysis: verifier, risk analyzer, search, architect, graph, diff, hypothesis |
| [nexus-cog-causal](https://github.com/nexus-cognitive-tech/nexus-cog-causal) | Causal reasoning: forward, backward, counterfactual, pre-mortem |
| [nexus-cog-cognitive](https://github.com/nexus-cognitive-tech/nexus-cog-cognitive) | Cognitive scaffolds, thought chains, response analysis, Cognitive Mirror |
| [nexus-cog-patterns](https://github.com/nexus-cognitive-tech/nexus-cog-patterns) | Pattern library: builtin patterns, matcher, relevance scorer |
| [nexus-cog-provenance](https://github.com/nexus-cognitive-tech/nexus-cog-provenance) | Artifact provenance: lineage, graph queries, blast radius |
| [nexus-cog-intel](https://github.com/nexus-cognitive-tech/nexus-cog-intel) | Adaptive learning, prediction, long-term memory, negative learning |
| [nexus-cog-intent](https://github.com/nexus-cognitive-tech/nexus-cog-intent) | Intent preservation: declare module purpose, detect drift |
| [nexus-cog-antifragile](https://github.com/nexus-cognitive-tech/nexus-cog-antifragile) | Adversarial input generation, robustness scoring |

## Interfaces

| Crate | Description |
|---|---|
| [nexus-cog-sdk](https://github.com/nexus-cognitive-tech/nexus-cog-sdk) | Single-import facade re-exporting all engines |
| [nexus-cog-mcp](https://github.com/nexus-cognitive-tech/nexus-cog-mcp) | Model Context Protocol server |
| [nexus-cog-cli](https://github.com/nexus-cognitive-tech/nexus-cog-cli) | Command-line interface |

## Contracts

| Crate | Description |
|---|---|
| [nexus-cog-proto](https://github.com/nexus-cognitive-tech/nexus-cog-proto) | Protocol contracts (.proto + serde mirrors) |

## Cloud

- [nexus-cog-cloud](https://github.com/nexus-cognitive-tech/nexus-cog-cloud) — **private** — managed sync, telemetry, billing

## Quick start

```toml
[dependencies]
nexus-cog-sdk = { git = "https://github.com/nexus-cognitive-tech/nexus-cog-sdk", tag = "v0.1.0" }
```

```rust,ignore
use nexus_cog_sdk::prelude::*;

let backend = std::sync::Arc::new(SqliteBackend::open("./palace.db")?);
let palace = PersistentPalace::new_default(backend);
palace.load()?;

let room = palace.add_room("Patterns", RoomType::Pattern)?;
palace.add_item(&room, MemoryItem::new("builder", "use the builder pattern", 0.9))?;
palace.save()?;
```

## License

Most components are Apache-2.0. `nexus-cog-cloud` is proprietary.
