# Separate the live map, exploration, and learned routes

Asterflow will keep the mutable Work Graph, runtime Exploration records,
append-only Episodes, and reviewed MOB Pattern Families as distinct facts.
Work Items will not persist solver-method labels: the Orchestrator selects a
method for each Exploration, accepts proposed Graph Deltas into the map, and
only sends evidence-backed learning proposals to MOB. This costs explicit
coordination between records, but prevents runtime state, route history, and
long-term knowledge from silently overwriting one another.
