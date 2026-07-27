# Handoff: Wayfinder foundation

Date: 2026-07-26

## Current state

Asterflow is still in architecture and repository-foundation work. No new
runtime has been implemented. The original OMO-Slim repository has been
preserved under `upstreams/oh-my-opencode-slim/`, while the root is becoming
the Asterflow project.

The discussion has converged on a conceptual architecture:

- one user-facing Wayfinder Orchestrator;
- one canonical Work Graph;
- recursive task decomposition without recursive control ownership;
- capability-specialized, temporary Agents that solve one Work Item;
- model and Skill selection based on Work Item requirements;
- structured `Work Item → Solver Outcome` handoff;
- Agent Frontier and Human Inbox as two projections of the same graph;
- asynchronous human review of decisions and versioned prototypes;
- MOB as compact recall and reviewed learning, not live task storage;
- independent evidence and synthesis before parent or Destination completion.

The previous Direct / Grill / Wayfinder top-level routing proposal is
superseded. Wayfinder is now always the control protocol. Direct, decision,
research, prototype, implementation, diagnosis, and verification are Work
Item kinds.

## Canonical documents

Read these in order:

1. [`CONTEXT.md`](../../CONTEXT.md) — canonical domain language.
2. [`docs/adr/0001-wayfinder-single-control-plane.md`](../adr/0001-wayfinder-single-control-plane.md)
   — accepted architectural direction.
3. [`docs/architecture/wayfinder-control-plane.md`](../architecture/wayfinder-control-plane.md)
   — consolidated architecture and open questions.
4. [`docs/research/fusion-architecture-survey.md`](../research/fusion-architecture-survey.md)
   — upstream comparison and earlier control-plane proposal.
5. [`docs/research/wayfinder-integration-study.md`](../research/wayfinder-integration-study.md)
   — detailed Wayfinder integration research.

Treat the architecture document and ADR as the current direction when they
conflict with the earlier research.

## Important distinctions

- The Wayfinder Orchestrator is the only graph writer and user-facing
  authority.
- The Work Graph is recursive; Specialist Agents are not sub-orchestrators.
- Agents may propose decomposition but cannot commit new nodes.
- Agent Frontier and Human Inbox are views, not separate todo stores.
- A Claim is a recoverable lease, not ownership.
- A Checkpoint allows any compatible Agent to resume blocked work.
- A Resolution does not automatically close its parent.
- Human Actions contain decisions, approvals, prototype review, permissions,
  credentials, or external actions that cannot be obtained from tools or MOB.
- MOB receives reviewed knowledge candidates, not claims or pending work.

## Repository and Git state

- Working directory: `D:\codex-test\asterflow`
- Branch: `first-light`
- The large staged change moves the original OMO-Slim tree under
  `upstreams/oh-my-opencode-slim/`.
- Root research, architecture, glossary, ADR, and handoff documents are part
  of the same staged foundation change.
- No commit has been created.
- No runtime tests were run because this work only reorganizes the upstream
  snapshot and adds Markdown documentation.

Preserve the archived upstream subtree. Do not edit it when implementing
Asterflow unless the task explicitly concerns the upstream snapshot.

## Next objective

Turn the conceptual architecture into a minimum state model and first vertical
slice without prematurely building the full Agent system.

Recommended next steps:

1. define the minimum Work Item, Work State, Human Action, Claim, Checkpoint,
   Resolution, and transition command schemas;
2. specify transition invariants and failure cases;
3. design the `inspect` / `apply` Interface against those scenarios;
4. define the project-local Markdown handoff directory, filename convention,
   and minimum frontmatter;
5. implement the demonstrator described in the architecture document;
6. verify restart recovery, lookup, and single-writer behaviour;
7. only then connect OMO Agent execution and Matt Skill routing.

## Suggested prompt for the next session

> Read `CONTEXT.md`,
> `docs/adr/0001-wayfinder-single-control-plane.md`,
> `docs/architecture/wayfinder-control-plane.md`, and
> `docs/handoffs/2026-07-26-wayfinder-foundation.md`. Continue the Asterflow
> architecture from the accepted single-Wayfinder control plane. Design the
> minimum persisted Work Graph schema, lifecycle transitions, and
> project-local Markdown handoff format for the first vertical slice. Do not
> implement the full Agent runtime yet.

## Handoff completeness

This handoff captures the current decisions, unresolved questions, repository
state, and a concrete continuation prompt. The new session should not need the
original conversation to resume the design.
