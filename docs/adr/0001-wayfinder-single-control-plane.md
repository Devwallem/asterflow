# Use Wayfinder as the single control plane

Asterflow will expose one Wayfinder Orchestrator to the user and maintain one
canonical Work Graph. Specialist Agents solve bounded Work Items and return
structured outcomes, but cannot mutate the graph or create their own control
flows. The Agent Frontier and Human Inbox are projections of that graph rather
than independently maintained todo stores. This trades some central scheduler
complexity for deterministic ownership, recoverable execution, asynchronous
human collaboration, and freedom to replace models, Skills, and storage
adapters without changing the task semantics.
