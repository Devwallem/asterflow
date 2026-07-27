# Asterflow

Asterflow is currently in architecture-research phase. The goal is to design a
small, observable agent harness that combines:

- OMO-Slim's OpenCode plugin and specialist-agent runtime;
- Matt Pocock Skills' composable engineering workflows;
- MyOutBrain's project-independent memory and reviewed reflection loop.

The conceptual control-plane architecture has now been selected. Current
foundation documents:

- [`CONTEXT.md`](CONTEXT.md) defines the canonical domain language.
- [`docs/adr/0001-wayfinder-single-control-plane.md`](docs/adr/0001-wayfinder-single-control-plane.md)
  records the single-Wayfinder control-plane decision.
- [`docs/architecture/wayfinder-control-plane.md`](docs/architecture/wayfinder-control-plane.md)
  consolidates the architecture, invariants, and open questions.
- [`docs/architecture/roguelike-work-graph-and-evolving-mob.md`](docs/architecture/roguelike-work-graph-and-evolving-mob.md)
  captures the dynamic map, universal Exploration, evolving MOB, and
  annealable route direction agreed during the next design session.

Supporting research:

- [`docs/research/fusion-architecture-survey.md`](docs/research/fusion-architecture-survey.md)
  compares the upstream projects and proposes the initial control plane.
- [`docs/research/wayfinder-integration-study.md`](docs/research/wayfinder-integration-study.md)
  studies Wayfinder as a persistent workflow mode and integration seam.
- [`docs/handoffs/2026-07-26-wayfinder-foundation.md`](docs/handoffs/2026-07-26-wayfinder-foundation.md)
  is the continuation handoff for the next design session.

## Repository layout

```text
asterflow/
├── CONTEXT.md                       # Canonical domain language
├── docs/
│   ├── adr/                         # Durable architecture decisions
│   ├── architecture/                # Current system design
│   ├── handoffs/                    # Cross-session continuation records
│   └── research/                    # Source research and earlier proposals
└── upstreams/
    ├── oh-my-opencode-slim/         # OMO-Slim evolution baseline
    ├── matt-skills/                 # Matt Skills evolution baseline
    └── myoutbrain/                  # MyOutBrain evolution baseline
```

The imported source trees are pinned, provenance-recorded starting points for
Asterflow rather than immutable dependencies. Their nested Git metadata is not
included, so future changes are tracked directly by this repository. See
[`upstreams/README.md`](upstreams/README.md) for exact commits and licensing
constraints. MyOutBrain private-instance data remains outside this repository.
