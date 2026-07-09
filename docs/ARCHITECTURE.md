# Architecture

## Layering

```
┌──────────────────────────────────────────────────────────────┐
│  Interfaces: MCP server · CLI binary                        │
│  (every CLI subcommand is also an MCP tool)                  │
├──────────────────────────────────────────────────────────────┤
│  Brain: nexus-cog-neural — Cortex orchestrator              │
│  (thalamus · hippocampus · cortical hierarchy · basal       │
│   ganglia · amygdala · working memory · attention ·         │
│   neuromodulators · global workspace · sleep cycle ·        │
│   replay buffer)                                            │
├──────────────────────────────────────────────────────────────┤
│  Orthogonal engines: causal · provenance · patterns ·       │
│  antifragile                                                │
├──────────────────────────────────────────────────────────────┤
│  Foundation: core (types · store · config)                   │
│            embeddings                                       │
└──────────────────────────────────────────────────────────────┘
```

Every layer depends only on layers below it. Foundation has no internal
dependencies; engines depend on core; the cortex depends on core;
orthogonal engines depend on core and storage; the CLI depends on every
layer above.

## Brain: nexus-cog-neural

A single orchestrator ([`Cortex`]) replaces the historical
palace/brain/cognitive/intel/intent engines. Every brain-like cognitive
operation goes through this crate. The architecture is built around
**eight biologically-inspired subsystems** that work together on
every tick:

| Subsystem | Module | Role |
| --- | --- | --- |
| Spike coding | [`spike`] | Replaces continuous SDR activation with sparse temporal codes — spike trains, phase-aligned oscillation, first-spike latency. |
| Synapse + plasticity | [`synapse`] | Permanence, eligibility traces, three-factor Hebbian LTP / LTD gated by local neuromodulator concentration. |
| Astrocyte network | [`astrocyte`] | Tripartite synapse glia — calcium waves, gliotransmitter release, global gliosis pressure. |
| Neurogenesis | [`neurogenesis`] | Birth / apoptosis controller — novelty × dopamine × NE gates new columns; sustained low permanence prunes old ones. |
| Cortical column (6 layers) | [`cortical_column`] | Canonical microcircuit (L1 apical context, L2/3 associatives, L4 thalamic input, L5 output, L6 feedback, lateral L2/3). |
| Recurrent hierarchy | [`hierarchy`] | DAG of columns with bottom-up, top-down and lateral recurrence. |
| Sensorimotor loop | [`sensorimotor`] | Perception → action → perception cycle with explicit LoopPhase and PredictionError per tick. |
| SDR (legacy helper) | [`sdr`] | 2048-bit sparse distributed representations — used as a hash-based text → SDR helper. |
| Thalamus | [`thalamus`] | Sensory relay + gating by salience × attention. |
| Hippocampus | [`hippocampus`] | Fast, capacity-bounded episodic memory. |
| Sleep cycle | [`sleep`] | NREM replay + REM re-activation on the spike-based hierarchy. |
| Basal ganglia | [`basal_ganglia`] | Action selection via winner-take-all. |
| Amygdala | [`amygdala`] | 3-D valence (reward/threat/novelty). |
| Working memory | [`working_memory`] | Miller's 7±2 slots. |
| Attention | [`attention`] | Top-down + bottom-up spotlight. |
| Neuromodulators | [`neuromodulators`] | Dopamine / serotonin / norepinephrine global signals. |
| Global workspace | [`global_workspace`] | Coalition selection. |
| Replay buffer | [`replay`] | Tick recording for Studio viz. |

## Orthogonal engines

These engines are kept as separate crates because they implement
specialised analyses that don't belong in the cortex.

* **nexus-cog-causal** — directed causal graph, forward / backward
  traversal, blast-radius, counterfactual and pre-mortem.
* **nexus-cog-provenance** — artifact lineage with SHA-256 content
  hashing and short-id / fuzzy record lookup.
* **nexus-cog-patterns** — code-pattern matching and recommendation.
* **nexus-cog-antifragile** — adversarial input generation, edge-case
  exploration, robustness scoring.

## Foundation

* **nexus-cog-core** — shared types, serde-friendly representations,
  store primitives, runtime configuration.
* **nexus-cog-embeddings** — pluggable text encoders + vector store.
* **nexus-cog-storage** — the only crate that depends on `rusqlite`.

## Interfaces

### CLI binary (`nexus-cog-cli`)

Every brain-related subcommand (`palace`, `brain`, `cognitive`,
`intel`, `intent`) is a thin wrapper over the cortex. Every orthogonal
subcommand (`causal`, `provenance`, `patterns`, `antifragile`,
`backup`, `decay`, `repl`) delegates to its own engine.

### MCP server

Run `nexus-cog mcp` to expose every tool over stdio. Each MCP tool is a
thin wrapper around the same `commands::*` function used by the CLI.

## Persistence

### Per-workspace DB

The SQLite database lives at `<workspace>/.nexus-cog/palace.db` where
`<workspace>` is the directory supplied via `--workspace`
(`NEXUS_COG_WORKSPACE`) or, failing that, the current working directory.
The cortex itself is in-memory; the orthogonal persistent engines
(causal graph, provenance, patterns) share the same SQLite file via
`PersistenceBackend`.

## Memory model

The cortex holds:

* a **thalamus** with N channels of sensory input;
* a **hierarchy** of six-layer cortical columns (L1, L2/3, L4, L5, L6)
  with full bottom-up / top-down / lateral recurrence;
* a **spiking population** per layer — phase-aligned spike trains
  replace continuous SDR activations;
* a **synaptic fan-out** per projection — permanence, eligibility,
  three-factor Hebbian plasticity modulated by local dopamine /
  serotonin / norepinephrine;
* an **astrocyte network** — one glia cell per column, slow
  calcium-wave coupling, gliotransmitter release boosts plasticity;
* a **neurogenesis controller** — births and prunes columns based
  on novelty × dopamine × NE gated by sustained low permanence;
* a **hippocampus** of capacity-bounded episodic memory;
* a **working memory** of 7±2 SDR slots with active maintenance;
* a **basal ganglia** that selects the next action;
* an **amygdala** that tags every event with valence;
* a **global workspace** that broadcasts the winning coalition;
* a **sensorimotor loop** that closes perception → action →
  perception and reports prediction error per tick;
* **neuromodulators** that tune learning rate, attention width and
  risk-aversion;
* a **replay buffer** that records every tick for Studio.

### `palace_recall`

BM25 (Robertson / Zaragoza defaults: `k1 = 1.5`, `b = 0.75`) over the
combined `(key, value, tags)` corpus of every hippocampal episode,
re-weighted with confidence: `0.8 * normalised_bm25 + 0.2 * confidence`.

### `intel_recall`

Three-stage pipeline:
1. SQLite FTS5 BM25 candidate set (best 4× limit).
2. Jaccard / coverage re-rank over `(key, value, tags)`.
3. Importance + recency boost.

## Security / drift detection

The `intent_check` MCP tool runs every Nexus Cog source snippet
through a multi-rule detector (regex-based for hard-coded
credentials, weak crypto, JWT bypass, injection sinks, missing error
handling on security paths, missing authorisation). Each finding is
classified, weighted by severity, and folded into the per-module IPI:

```
ipi = 0.7 × (100 - penalty_score(findings, strict))
    + 0.3 × amygdala_signal × 100
penalty_score(finding) = severity_weight(finding.severity)
  Critical=20, Error=14, High=10, Warning=6, Medium=4, Low=2, Info=0 (1 in strict mode)
```

## Workspace layout

The poly-repo root `nexus-cognitive-tech/` is **not** a git repository —
each `nexus-cog-*/` directory is its own independent git checkout.

```
nexus-cognitive-tech/                  ← plain directory, not a repo
├── nexus-cog-core/        (.git)
├── nexus-cog-storage/     (.git)
├── nexus-cog-embeddings/  (.git)
├── nexus-cog-neural/      (.git)        ← brain-like cortex
├── nexus-cog-causal/      (.git)
├── nexus-cog-patterns/    (.git)
├── nexus-cog-provenance/  (.git)
├── nexus-cog-antifragile/ (.git)
├── nexus-cog/             (.git)        ← meta repo, docs only
└── nexus-cog-cli/         (.git)        ← binary
```

Sibling git dependencies (`nexus-cog-core = { git = "...", tag = "..."}`)
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

# Test the cortex
cd nexus-cog-neural && cargo test --lib

# Test everything
for d in nexus-cog-{core,storage,embeddings,neural,causal,patterns,provenance,antifragile,cli}; do
  (cd $d && cargo test --lib)
done
```
