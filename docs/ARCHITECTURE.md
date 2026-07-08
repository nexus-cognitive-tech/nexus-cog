# Architecture

## Layering

```
┌─────────────────────────────────────────────────┐
│  Interfaces: SDK · MCP · CLI · Proto            │
├─────────────────────────────────────────────────┤
│  Engines: palace · brain · causal · cognitive    │
│           patterns · provenance · intel          │
│           intent · antifragile                  │
├─────────────────────────────────────────────────┤
│  Foundation: core (types · store · config)      │
└─────────────────────────────────────────────────┘
```

Every layer depends only on layers below it. Foundation has no internal dependencies; engines depend on core; interfaces depend on engines and core.

## Multi-namespace

Each SQLite database holds many palaces scoped by `palace_id`. One palace is the default namespace; others can be created via `PersistentPalace::new(backend, "name")` or `ensure_palace()`.

## Recall

The palace supports three recall strategies, descending priority:

1. **Vector** — pluggable embedder (`Embedder` trait) with cosine similarity
2. **FTS5** — SQLite full-text search with bm25 ranking
3. **Word overlap** — in-memory fallback for terms the other engines miss

All three bump the access counter and stamp `last_accessed` on recall, so popular items survive decay.
