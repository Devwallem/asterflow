# Asterflow

Asterflow coordinates human and machine work toward an observable outcome
through one user-facing authority and one canonical task model.

## Language

**Destination**:
The observable outcome envelope the user wants Asterflow to reach. It may
admit several acceptable endings, but it is complete only when its acceptance
conditions are supported by evidence.
_Avoid_: Goal, root todo

**Wayfinder Orchestrator**:
The single user-facing authority that maintains task direction, assigns work,
incorporates results, and reports progress.
_Avoid_: Main Agent, Router

**Work Graph**:
The canonical representation of a task, preserving both explanatory
parent-child structure and execution dependencies.
_Avoid_: Spec tree, todo database

**Work Item**:
A revealed, addressable node in the Work Graph whose exploration scope is
bounded even when its contents still contain Fog.
_Avoid_: Todo, subtask

**Fog**:
Explicit uncertainty inside a revealed Work Item about its contents, route,
constraints, or evidence. Fog may shrink, grow, or reveal more Work Items as
the node is explored.
_Avoid_: Unknown state, uncreated task

**Exploration**:
One bounded visit to a Work Item that may change what is known, reveal more of
the Work Graph, produce evidence, or establish a Resolution.
_Avoid_: Research phase, execution mode

**Graph Delta**:
A proposed or accepted change to the revealed Work Graph produced by an
Exploration.
_Avoid_: Agent edit, map rewrite

**Agent Frontier**:
The projection of Work Items currently ready for Specialist Agents. It is not
an independently maintained list.
_Avoid_: Agent todo list, worker backlog

**Human Inbox**:
The projection of Work Items currently requiring a decision, approval,
inspection, permission, credential, or external action from the user.
_Avoid_: Human todo database

**Human Action**:
A Work Item in the Human Inbox that presents what is needed, why a human is
needed, what it blocks, and the acceptable forms of response.
_Avoid_: Question, notification

**Specialist Agent**:
A temporary solver assigned to one Work Item according to the capabilities,
methods, tools, permissions, and cost appropriate to that item.
_Avoid_: Persona, sub-orchestrator

**Claim**:
A temporary grant allowing one Specialist Agent to perform an Exploration of
a Work Item. It does not transfer control of the Work Graph.
_Avoid_: Ownership

**Checkpoint**:
A durable handoff recording completed progress, evidence, artifacts,
assumptions, and remaining work when execution pauses or becomes blocked.
_Avoid_: Chat summary

**Artifact**:
A concrete, inspectable result such as a document, page, prototype, program,
patch, report, or test result.
_Avoid_: Agent output

**Resolution**:
The structured result of a Work Item, including its conclusion, evidence,
artifacts, assumptions, and remaining uncertainty.
_Avoid_: Final answer, completion

**Episode**:
An immutable record of a route through one Destination, including its
Explorations, choices, Graph Deltas, costs, evidence, and terminal outcome.
_Avoid_: Chat transcript, mutable run state

**Pattern Family**:
A reviewed MOB memory that groups reusable Route Variants for the same problem
shape together with their applicability and comparative evidence.
_Avoid_: Cached answer, canonical workflow

**Route Variant**:
One conditional solution path in a Pattern Family, supported by Episodes and
selected according to the current context.
_Avoid_: Universal best practice, fixed Skill

**Hard Envelope**:
The acceptance, evidence, safety, permission, and human-approval constraints
that every Route Variant must preserve.
_Avoid_: Mandatory workflow

**Exploration Temperature**:
The current willingness to deviate from a proven Route Variant while remaining
inside the Hard Envelope.
_Avoid_: Model temperature, randomness
