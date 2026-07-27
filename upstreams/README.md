# Upstream evolution baselines

`upstreams/` contains exact source baselines that Asterflow intends to evolve,
combine, and eventually replace in place. They are tracked directly by the
Asterflow repository rather than as nested Git repositories or immutable
runtime dependencies.

## Imported baselines

### OMO-Slim

- Path: `upstreams/oh-my-opencode-slim/`
- Source: <https://github.com/alvinunreal/oh-my-opencode-slim>
- Commit: `1c0e1f4abe217b6965997201c37ff1de6720c13d`
- Package version: `2.2.8`

This baseline is an exact, content-preserving relocation of the repository
that Asterflow started from.

### Matt Pocock Skills

- Path: `upstreams/matt-skills/`
- Source: <https://github.com/mattpocock/skills>
- Commit: `ed37663cc5fbef691ddfecd080dff42f7e7e350d`
- License at this baseline: MIT; see `upstreams/matt-skills/LICENSE`

### MyOutBrain

- Path: `upstreams/myoutbrain/`
- Source: <https://github.com/Devwallem/MyOutBrain>
- Commit: `0734b45acab58c90f0347055fafce1d0ba119d4d`
- License at this baseline: no license file or public redistribution grant is
  present in the imported source

The missing MyOutBrain license must be resolved before Asterflow or a derived
distribution is shared outside an owner-authorized environment. MyOutBrain
private-instance data is not source code and must always remain outside this
repository.

These commit identifiers describe provenance, not compatibility promises.
Asterflow is expected to diverge from all three baselines as its dynamic Work
Graph, evolving MOB memory, and adaptive Skill execution are implemented.
