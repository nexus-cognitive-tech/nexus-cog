# Architecture

## Layering

```
┌──────────────────────────────────────────────────────────────┐
│  Interfaces: MCP server · CLI binary                        │
│  (every CLI subcommand is also an MCP tool)                  │
├──────────────────────────────────────────────────────────────┤
│  Engines: palace · brain · causal · cognitive                │
│           patterns · provenance · intel                      │
│           intent · antifragile                              │
├──────────────────────────────────────────────────────────────┤
│  Foundation: core (types · store · config)                   │
└──────────────────────────────────────────────────────────────┘
```

Every layer depends only on layers below it. Foundation has no internal
dependencies; engines depend on core; interfaces depend on engines and
core.

## Engines

### Palace (`nexus-cog-palace`)

Room-based memory model. Each room is a category (concept, pattern,
decision, bug, learning, tool, user, project); items inside rooms are
individual memories; connections between rooms form the conceptual
graph.

### Brain (`nexus-cog-brain`)

Static analysis: code verification (8-check adaptive), risk
classification, semantic search, architecture analysis, semantic diff,
A/B hypothesis testing with a real decision matrix.

### Causal (`nexus-cog-causal`)

Directed causal graph with forward / backward traversal, blast-radius,
counterfactual analysis and pre-mortem scenarios derived from the live
graph.

### Cognitive (`nexus-cog-cognitive`)

6-phase scaffold protocol (Understand → Analyze → Design → Implement →
Verify → Reflect), thought chains, response analysis, and the Cognitive
Mirror (audit of an agent's reasoning chain).

### Patterns (`nexus-cog-patterns`)

Code-pattern matching and recommendation.

### Provenance (`nexus-cog-provenance`)

Artifact lineage graph with SHA-256 content hashing, short-id /
fuzzy record lookup, and structured record / explain / search.

### Intel (`nexus-cog-intel`)

Long-term memory with **hybrid BM25 + FTS5 + recency + importance**
ranking; adaptive learner with structured `suggest_approach`
responses (never `null`).

### Intent (`nexus-cog-intent`)

Module purpose declaration plus a real **security / intent drift
detector**: hard-coded credentials, weak crypto (MD5 / SHA1), JWT
bypass (`alg=none`, `verify=false`, empty `kid`), SQL / shell injection
sinks, missing error handling on security paths, missing authorisation
on public mutators. Detector findings fold into the per-module IPI
(Intent Preservation Index).

### Antifragile (`nexus-cog-antifragile`)

Adversarial input generation (paginated, category-filtered), edge-case
exploration, robustness scoring.

## Foundation

### Core (`nexus-cog-core`)

Shared types, serde-friendly representations, store primitives, runtime
configuration.

### Storage (`nexus-cog-storage`)

The only crate that depends on `rusqlite`. Every other engine uses
`PersistenceBackend` (or `TransactionalBackend`) so the storage
technology can change without rippling outwards.

### Embeddings (`nexus-cog-embeddings`)

Pluggable text encoders (`Embedder` trait) and a SQLite-backed vector
store.

## Interfaces

### CLI binary (`nexus-cog-cli`)

Clap subcommands, one per engine. Every subcommand also doubles as an
MCP tool — single source of truth via `nexus_cog_cli::commands::*`.

### MCP server

Run `nexus-cog mcp` to expose all 32 tools over stdio. The server is a
thin wrapper around the same `commands::*` functions used by the CLI,
so there is one implementation per tool.

## Persistence

### Per-workspace DB

By default the database lives at `<workspace>/.nexus-cog/palace.db`
where `<workspace>` is the directory supplied via `--workspace`
(`NEXUS_COG_WORKSPACE`) or, failing that, the current working directory.
The previous global default (`~/.local/share/nexus-cog/palace.db`)
leaked state across unrelated agents and has been removed.

### Schema ownership

Each engine owns its tables and registers its schema through
`PersistenceBackend::apply_migrations`. The backend tracks applied
migrations in `engine_migrations (owner, version)` so re-opening the DB
is idempotent.

## Multi-namespace

Each SQLite database holds many palaces scoped by `palace_id`. One
palace is the default namespace; others can be created via
`PersistentPalace::new(backend, "name")` or `ensure_palace()`.

## Recall

### Palace (`palace_recall`)

BM25 (Robertson / Zaragoza defaults: `k1 = 1.5`, `b = 0.75`)
re-weighted with confidence: `0.8 * normalised_bm25 + 0.2 * confidence`.

### Intel (`intel_recall`)

Three-stage pipeline:
1. SQLite FTS5 BM25 candidate set (best 4× limit).
2. Jaccard / coverage re-rank over `(key, value, tags)` so every
   matched query token boosts the score.
3. Importance + recency boost so freshly-touched, important entries
   surface above equally-relevant ancient ones.

All three bump the access counter and stamp `last_accessed` on recall,
so popular items survive decay.

## Security / drift detection

The `intent_check` MCP tool runs every Nexus Cog source snippet
through a multi-rule detector (regex-based for hard-coded
credentials, weak crypto, JWT bypass, injection sinks, missing error
handling on security paths, missing authorisation). Each finding is
classified, weighted by severity, and folded into the per-module IPI:

```
ipi = max(0, 100 - penalty_score(findings, strict))
penalty_score(finding) = severity_weight(finding.severity)
  Critical=20, Error=14, High=10, Warning=6, Medium=4, Low=2, Info=0 (1 in strict mode)
```

The result is never `0` without reason: even an empty drift set
yields `ipi = 100`, and a single `Critical` finding caps the score at
`80`.

## Workspace layout

The repository is a poly-repo: every `nexus-cog-*/` directory is its
own independent git repository (`nexus-cog-*/.git`). The poly-repo
root `nexus-cognitive-tech/` is **not** a git repository — it is the
directory under which every member crate is checked out.

```
nexus-cognitive-tech/                  ← plain directory, not a repo
├── nexus-cog-core/        (.git)
├── nexus-cog-storage/     (.git)
├── nexus-cog-embeddings/  (.git)
├── nexus-cog-palace/      (.git)
├── nexus-cog-brain/       (.git)
├── nexus-cog-cognitive/   (.git)
├── nexus-cog-causal/      (.git)
├── nexus-cog-patterns/    (.git)
├── nexus-cog-provenance/  (.git)
├── nexus-cog-intel/       (.git)
├── nexus-cog-intent/      (.git)
├── nexus-cog-antifragile/ (.git)
├── nexus-cog/             (.git)        ← meta repo, docs only
└── nexus-cog-cli/         (.git)        ← binary
```

Sibling git dependencies (`nexus-cog-core = { git = "...", tag = "..." }`)
keep each member crate independently buildable. The CLI builds from
`nexus-cog-cli/` against sibling `path` deps; to restore the upstream
git deps for a release build, swap the `path =` lines in
`nexus-cog-cli/Cargo.toml` for `git = "..."` with the appropriate `tag`.

> **Cargo 1.96 quirk**: `[patch."https://github.com/...<crate>"]`
> used to redirect git deps to local paths triggers an upstream
> `patch for <transitive dep> in registry crates-io resolved to more
> than one candidate` error. We therefore pin sibling deps via `path`
> directly inside `nexus-cog-cli/Cargo.toml`. Tracking:
> `cargo` issue tracker.

## Build & test

```bash
# Build the CLI binary
cd nexus-cog-cli && cargo build --bin nexus-cog

# Test a single engine crate
cd nexus-cog-intel && cargo test --lib

# Test everything (run from nexus-cog-cli)
cd nexus-cog-cli && cargo test --lib
```
