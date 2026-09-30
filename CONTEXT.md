# Agent Skills Nomenclature: Context

Reference context for the skill taxonomy. Load this file into an agent session when the agent needs to classify, name, or map skills. For the agent-facing operating procedure see `AGENT.md`; for the human-facing overview see `README.md`.

To keep track of a skill's purpose, execution boundary, and cognitive weight across a complex orchestration framework, you need a highly scannable nomenclature. Because skills operate across three different dimensions (Execution, Lifecycle, Structure), relying on a single word is insufficient. Three standardization models follow, chosen by where the identifier is used: monorepo file structures, LangGraph node names, or telemetry traces.

---

## Naming Conventions

Because skills operate across three dimensions, a single word is insufficient. Three standardization models follow, chosen by where the identifier is used: LangGraph node names and CLI commands, telemetry traces, or monorepo file structures.

### Abbreviation Table

| Dimension | Value | Code |
|---|---|---|
| Structure | Atomic | `atm` |
| Structure | Composite | `cmp` |
| Structure | Meta | `mta` |
| Execution | Procedural | `prc` |
| Execution | Executable | `exe` |
| Execution | Hybrid | `hyb` |
| Lifecycle | Static | *(omitted, implicit default)* |
| Lifecycle | Ephemeral | `eph` |
| Lifecycle | Adaptive | `adp` |

### 1. The Short-Prefix Matrix (For LangGraph Nodes & CLI Commands)

A two-to-three part prefix format `[Structure]_[Execution]_[Name]-[Lifecycle]` gives immediate context to the execution engine and to the developer reading the orchestration graph.

```
atm_exe_sqlite_fts          # atomic, executable, static
cmp_hyb_figma_sync          # composite, hybrid, static
mta_prc_planner-eph         # meta, procedural, ephemeral
atm_exe_github_retry-adp    # atomic, executable, adaptive
```

### 2. The URN / Dot-Notation (For Telemetry & State Checkpointing)

For observability pipelines (mapping to `gen_ai.skill.*` OpenTelemetry semantic conventions) or serializing active skills into session checkpoint files, a dot-notation namespace prevents collision across polyglot architectures.

**Format:** `[domain].[structure].[execution].[lifecycle].[action]`

```
core.mta.prc.eph.plan
data.atm.exe.static.sqlite_fts_query
design.cmp.hyb.static.figma_sync
```

Wildcards make filtering trivial: `*.exe.*` monitors every deterministic script; `core.mta.*` traces all control-plane operations. Unlike the other two conventions, lifecycle is written explicitly here (`static`) so the segment count stays fixed for parsers.

### 3. The Extension Convention (For Monorepo File Structures)

When managing skills across organization, team, and project levels in a monorepo, encoding the taxonomy into the file extension or directory name clarifies how the harness loads and executes the skill.

**Format:** `[name].[structure].[execution].[ext]`

- **Procedural Skills:** `planner.mta.prc.md` (recognizable as a pure prompt/markdown skill).
- **Executable Skills:** `sqlite_fts.atm.exe.ts` or `github_retry.atm.exe.rs` (deterministic scripts/binaries).
- **Hybrid/Composite Skills:** `figma_sync.cmp.hyb/` (a directory bundle containing the procedural `SKILL.md` orchestrator and the local `scripts/` folder).

With a strict extension pattern, the agent bootloader can glob `*.prc.md` files into LLM prompt injection and `*.exe.ts` files into sandboxed subprocess or MCP server initialization. Lifecycle is not encoded in the filename because it is a property of how the harness loads the file, not of the file itself; record it in the skill's manifest or frontmatter instead.

---

## Mapping to LangGraph

Mapping this taxonomy onto a LangGraph architecture means treating skills not as isolated functions but as distinct topological constructs: nodes, subgraphs, conditional edges, and state reducers.

### 1. State Management & Lifecycle Modifiers

*How skills exist within the `StateGraph` memory payload.*

- **Static Skills:** Bound during graph initialization (before `.compile()`). They exist either as hardcoded `tools=[...]` bound to the primary LLM nodes or as immutable system instructions injected into the base `State` payload at kickoff. They persist across all checkpoints.
- **Ephemeral Skills:** Map to custom `State` reducers (e.g., `active_skills: Annotated[list, add_skills]`). A retrieval node dynamically fetches the skill (e.g., via vector search on current context) and appends it to the state. The reducer logic or a cleanup node pops the skill once the task completes, keeping the memory payload compact for serialization.
- **Adaptive Skills:** Use LangGraph's checkpointer and thread-level state memory. A node evaluates the output of an execution; if it fails or requires optimization, it mutates the skill's configuration parameters *within the state thread* before a conditional edge loops back for a retry. The skill "adapts" by reading its updated configuration from the checkpoint.

### 2. Node Implementations

*How work is executed via `add_node`.*

- **Procedural Skills:** Standard LLM reasoning nodes (`def procedural_node(state):`). The node ingests the natural-language instructions from state, runs an inference cycle, and appends an AI message (the "thought" or "plan") to the state log.
- **Executable Skills:** Map directly to LangGraph's `ToolNode`. The agent node outputs a `tool_call`, and a conditional edge routes to the `ToolNode`, a deterministic wrapper around standalone scripts or MCP servers that takes strict schema inputs and returns stdout/stderr as a `ToolMessage`.
- **Atomic Skills:** The purest 1:1 mapping. A single isolated node (one LLM node or one `ToolNode`) with no internal routing.

### 3. Subgraphs & Complex Topologies

*How higher-order capabilities are built from nested `CompiledGraph` instances.*

- **Hybrid Skills:** A nested subgraph. The entry node is the "Orchestrator" (procedural reasoning) with a specific prompt. It uses conditional edges to route to internal "Leaf" nodes (executable helper scripts), evaluates the leaf outputs, then resolves the subgraph and returns control to the parent.
- **Composite Skills:** Also subgraphs, but focused on workflow orchestration. A composite subgraph defines a control structure, such as a parallel topology (`Send()` API mapping over a list) or a strictly ordered sequence of atomic nodes, encapsulating that complexity away from the main graph.

### 4. Routing & Edge Execution

*How control flow is manipulated via `add_conditional_edges`.*

- **Meta Skills:** "Supervisor" or "Router" nodes. A Meta Skill node rarely does business logic. It inspects the current `State` (a plan, a reported execution gap), selects which subordinate nodes or skills to invoke next, and returns routing strings. The conditional edge uses that output to dictate the graph's path, letting the architecture orchestrate itself.

### Quick Reference

| Skill type | LangGraph construct |
|---|---|
| Procedural | LLM node |
| Executable | `ToolNode` |
| Hybrid | Subgraph: orchestrator LLM node + leaf `ToolNode`s |
| Static | `tools=[...]` / base `State` at compile time |
| Ephemeral | `State` reducer, populated by a retrieval node, popped by cleanup |
| Adaptive | Checkpointed config mutated by an evaluator node, retry via conditional edge |
| Atomic | One node |
| Composite | Subgraph with `Send()` fan-out or ordered sequence |
| Meta | Supervisor node feeding `add_conditional_edges` |

---

## The Three Dimensions

Every skill has exactly one value on each of three independent axes. The axes are orthogonal: a skill's execution paradigm says nothing about when it is loaded, and neither says anything about whether it stands alone or coordinates others.

| Dimension | Question it answers | Values |
|---|---|---|
| **Execution** | *Who does the work: the model's reasoning, a script, or both?* | Procedural · Executable · Hybrid |
| **Lifecycle** | *When does the skill enter and leave the agent's context?* | Static · Ephemeral · Adaptive |
| **Structure** | *Does the skill stand alone, coordinate others, or operate on the architecture itself?* | Atomic · Composite · Meta |

A fully qualified skill therefore reads as a triple, e.g. `figma_sync` = Composite / Hybrid / Static.

---

## Execution & Compute Paradigms

*How the work gets done.*

### 1. Procedural Skills (Pure Agent-Driven)

*Also known as: Declarative, Semantic, Prompt-Driven, or Agent-Heavy.*

Procedural skills dictate *how* an agent should think, approach a problem, or behave. They rely entirely on the LLM's internal reasoning and its access to pre-existing, generic tools (like a standard file reader or shell).

- **Implementation:** Markdown files (e.g., a `SKILL.md`), YAML manifests, or injected system prompt directives.
- **Execution:** The orchestrator appends the skill text to the agent's context window.
- **Best For:** Open-ended tasks like architectural planning, defining code review standards, translating natural language into BDD/TDD requirements, or applying structural patterns (e.g., EVOLVE principles).
- **Trade-offs:** Highly adaptable to novel edge cases, but prone to behavioral drift, hallucination, or getting stuck in reasoning loops.

### 2. Executable Skills (Script-Heavy)

*Also known as: Deterministic, Tool-Native, Imperative, or Code-Heavy.*

Executable skills are bounded software functions wrapped in an agent-compatible schema (like JSON Schema). The agent acts merely as a semantic router: it understands the user's intent, maps it to the skill's required parameters, and triggers execution.

- **Implementation:** Standalone Python/Node.js scripts, WASM binaries, or Model Context Protocol (MCP) servers.
- **Execution:** The agent outputs a structured tool call; the harness executes the script in a sandbox and returns the stdout/stderr.
- **Best For:** Exacting state mutations and data extraction, such as syncing OSV vulnerabilities, querying a local SQLite FTS5 index, or making Figma REST API calls.
- **Trade-offs:** Maximum safety, predictability, and easily gated by permissions, but completely inflexible if the user's request falls outside the script's strict parameter definitions.

### 3. Hybrid Skills (Agent-Driven with Helper Scripts)

*Also known as: Delegated, Orchestrator-Leaf, or Orchestrated.*

This pattern pairs a procedural reasoning framework with dedicated, sandboxed helper scripts. The agent retains autonomy over the execution plan, but is instructed to delegate specific, high-risk, or complex actions to the provided scripts rather than trying to write the code itself on the fly.

- **Implementation:** A modular directory structure containing a `SKILL.md` (the orchestration rules) alongside a `scripts/` folder, `assets/`, and `references/`.
- **Execution:** The agent reads the procedural guide, which instructs it on *when* and *how* to invoke the local helper scripts to achieve the goal.
- **Best For:** Complex, multi-step workflows like automated code refactoring, where the agent decides *what* to change (procedural) but must use a specific AST-parsing script to actually mutate the files (executable).
- **Trade-offs:** The ideal balance of cognitive flexibility and execution safety, though it requires heavier upfront engineering to build the skill bundle.

---

## Lifecycle & Adaptation Models

*When the skill exists in the agent's context, and whether it changes while it is there.*

### 4. Ephemeral Skills (Just-In-Time / Dynamic)

*Also known as: Dynamic, Context-Injected, or Transient.*

Beyond static, boot-time loading, advanced harnesses utilize dynamic skill injection. An ephemeral skill exists in memory only for the duration of a specific execution state or task, then is removed so the context payload stays compact.

- **Implementation:** RAG-based skill registries or context appenders; a state reducer (e.g., `active_skills: Annotated[list, add_skills]`) that a retrieval node appends to and a cleanup node pops from; or a Meta-Agent that synthesizes a bespoke skill on demand.
- **Execution:** Instead of loading all skills at boot, the orchestration graph loads specific skills (procedural or executable) only when a specific state node is reached, or synthesizes a bespoke skill via a Meta-Agent based on an identified context gap. The skill is dropped from state once the node completes.
- **Best For:** Large skill libraries that would exhaust the context window if loaded wholesale; long-running sessions with many distinct phases; domain knowledge needed at exactly one step (e.g., a database-migration guide loaded only while the migration node runs).
- **Trade-offs:** Keeps the context window lean and reduces distraction from irrelevant tools, but adds retrieval latency, depends on retrieval quality (a skill that is never retrieved fails silently), and weakens reproducibility because the active skill set differs from run to run.

### 5. Adaptive Skills (Context-Responsive)

*Also known as: Self-Tuning, Feedback-Driven, or Reflective.*

Adaptive skills autonomously adjust their execution parameters, underlying logic, or prompt weightings based on runtime feedback, environmental constraints, or historical execution gaps. The skill definition is stable; its configuration is not.

- **Implementation:** A skill whose tunable parameters (prompt weightings, retry budgets, tool arguments, model selection) live in thread-level state rather than in the skill file, paired with an evaluator node and a checkpointer that persists the mutated configuration.
- **Execution:** A node evaluates the output of an execution. If it fails or needs optimization, the node mutates the skill's configuration *within the state thread*, and a conditional edge loops back for a retry. On the next pass the skill reads its updated configuration from the checkpoint.
- **Best For:** Flaky integrations (rate-limited APIs that need escalating backoff), quality gates where output is graded and regenerated against a rubric, and environments whose constraints vary at runtime (memory limits, tool versions, permission scopes).
- **Trade-offs:** Resilient and self-correcting without human intervention, but hard to audit (behavior on run *N* depends on runs 1 through *N-1*), prone to unbounded loops without an explicit retry ceiling, and liable to drift from original intent if the feedback signal is noisy.

### 6. Static Skills (Predefined / Persistent)

*Also known as: Boot-Time, Global, or Always-On.*

Static skills are immutably defined capabilities loaded globally at agent boot time and persistently available across the entire session lifecycle. This is the implicit default; the other two lifecycles are modifiers on it.

- **Implementation:** Hardcoded `tools=[...]` bound to the primary LLM node, or immutable system instructions injected into the base `State` payload at kickoff; version-controlled files read once at startup.
- **Execution:** Bound during graph initialization, before `.compile()`. They persist across every checkpoint for the life of the session and are never popped from state.
- **Best For:** Foundational capabilities every task needs: file and shell access, safety rules, house coding standards, and the core toolset the agent should never be without.
- **Trade-offs:** Maximum predictability, zero retrieval latency, and trivial reproducibility, but every static skill permanently consumes context budget, and a large static set dilutes attention and raises the odds of the agent selecting the wrong tool.

---

## Structural Composition & Hierarchy

*How the skill relates to other skills.*

### 7. Atomic Skills (Single-Responsibility)

*Also known as: Primitive, Leaf, or Unit.*

Atomic skills are highly focused, indivisible primitives designed to perform exactly one targeted function or cognitive task without delegating to sub-dependencies.

- **Implementation:** A single function, script, or prompt fragment with one well-defined input/output contract and no internal routing logic.
- **Execution:** The purest 1:1 mapping to a graph: one isolated node (either one LLM node or one `ToolNode`), invoked and resolved in a single step.
- **Best For:** Building blocks you would unit-test in isolation and reuse inside many composites: read a file, run one query, format one value, apply one lint rule, fetch one record.
- **Trade-offs:** Easy to test, reason about, permission-gate, and compose, but an over-granular atom set produces chatty tool-call sequences and pushes orchestration burden up to whatever coordinates them.

### 8. Composite Skills (Higher-Order Capability)

*Also known as: Workflow, Pipeline, or Macro.*

A composite skill is a reusable control structure that coordinates two or more subordinate skills, tools, or executable primitives through defined orchestration patterns (sequence, branching, parallelism, delegation, and loops).

- **Implementation:** A nested subgraph or a declarative workflow definition that references its child skills and encodes the control pattern: strictly ordered sequence, conditional branch, parallel fan-out via the `Send()` API, bounded loop, or delegation to a sub-agent.
- **Execution:** The parent graph invokes the composite as a single node. Internally it runs the orchestration pattern over its children, aggregates their results, and returns one consolidated output, hiding the internal topology from the main graph.
- **Best For:** Repeated multi-step workflows such as lint-test-build-report; mapping one operation over a list of files or services; a review pipeline that chains several specialized checks into one verdict.
- **Trade-offs:** Encapsulates complexity and makes whole workflows reusable, but failures inside the composite are harder to localize than in a flat graph, and children that are tightly coupled to the composite lose their standalone reuse value.

### 9. Meta Skills (Control-Plane Operations)

*Also known as: Supervisor, Router, or Self-Referential.*

Meta skills operate on the architecture itself rather than on the business problem. They evaluate, synthesize, orchestrate, or modify other skills.

- **Implementation:** A supervisor or router node whose prompt is oriented toward inspecting state and the available skill catalog rather than doing domain work; optionally a skill-generation template used to author new skills at runtime.
- **Execution:** The node inspects the current `State` (a plan, a reported execution gap, a failed check), dynamically selects which subordinate nodes or skills to invoke next, and returns routing strings that the conditional edge consumes to set the graph's path. It rarely produces business output of its own.
- **Best For:** Planning and task decomposition, skill selection and routing, gap detection, self-evaluation, and skill authoring (a skill that writes or repairs other skills).
- **Trade-offs:** Enables self-organizing, self-extending architectures, but adds an inference hop to every routing decision, is the hardest category to test deterministically, and a faulty meta skill fails the whole graph rather than a single task.

---

## Summary Matrix

| # | Skill type | Dimension | Also known as | One-line definition |
|---|---|---|---|---|
| 1 | Procedural | Execution | Declarative, Semantic, Prompt-Driven | Text that shapes how the model reasons |
| 2 | Executable | Execution | Deterministic, Tool-Native, Imperative | A schema-wrapped script the model only routes to |
| 3 | Hybrid | Execution | Delegated, Orchestrator-Leaf | Procedural guide that delegates risky steps to bundled scripts |
| 4 | Ephemeral | Lifecycle | Just-In-Time, Dynamic | Loaded at a specific node, dropped afterward |
| 5 | Adaptive | Lifecycle | Self-Tuning, Feedback-Driven | Reconfigures itself from runtime feedback via checkpointed state |
| 6 | Static | Lifecycle | Boot-Time, Always-On | Bound before compile, persistent for the session |
| 7 | Atomic | Structure | Primitive, Leaf | One node, one responsibility, no routing |
| 8 | Composite | Structure | Workflow, Pipeline | Subgraph coordinating children through a control pattern |
| 9 | Meta | Structure | Supervisor, Router | Operates on the skill graph itself |
