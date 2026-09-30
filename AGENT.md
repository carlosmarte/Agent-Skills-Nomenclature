# AGENT.md

Operating instructions for agents that classify, name, build, or review skills using this nomenclature. `README.md` explains the taxonomy; `CONTEXT.md` is the loadable reference; this file tells you what to *do*.

## 1. What a skill is here

A skill is any packaged capability an agent can be given: a prompt file, a script behind a schema, a bundle of both, or a node in an orchestration graph. Every skill is described by exactly one value on each of three axes:

| Axis | Values | Default |
|---|---|---|
| Execution | Procedural · Executable · Hybrid | none, always state it |
| Lifecycle | Static · Ephemeral · Adaptive | Static |
| Structure | Atomic · Composite · Meta | none, always state it |

A skill is never "procedural *or* atomic". It is procedural *and* atomic *and* (implicitly) static. Always classify on all three axes before naming.

## 2. Classification procedure

Answer the three questions in order. Stop at the first matching row in each table.

### 2.1 Execution: who does the work?

| If the skill... | Then it is |
|---|---|
| Consists only of instructions the model reads, and the model uses generic tools (shell, file reader) to act | **Procedural** (`prc`) |
| Is a script, binary, or MCP tool with a fixed parameter schema, and the model only fills in parameters | **Executable** (`exe`) |
| Has instructions that tell the model *when* to call bundled scripts in a `scripts/` folder | **Hybrid** (`hyb`) |

Tie-breakers:
- Instructions that merely *mention* an existing generic tool (e.g. "use `grep`") are still Procedural. Hybrid requires scripts shipped *with* the skill.
- A script that also ships a long usage guide is still Executable if the model never reasons about intermediate steps. If the model decides an execution plan across several script calls, it is Hybrid.

### 2.2 Lifecycle: when is it in context?

| If the skill... | Then it is |
|---|---|
| Is loaded once at boot or graph compile and stays for the whole session | **Static** (omit suffix) |
| Is retrieved, injected, or synthesized at a specific step and removed afterward | **Ephemeral** (`eph`) |
| Rewrites its own configuration (retry budget, prompt weights, tool args) from runtime feedback and reads the new config on the next pass | **Adaptive** (`adp`) |

Tie-breakers:
- A skill that is *available* all session but only *used* sometimes is Static. Ephemeral is about presence in context, not frequency of use.
- A skill that retries with the *same* parameters is not Adaptive. Adaptive requires the parameters to change between attempts and to be persisted in state.
- A skill can be both Ephemeral and Adaptive in practice; when forced to pick one code, choose the one that dominates its failure mode (retrieval risk vs. drift risk) and record the other in the manifest.

### 2.3 Structure: how does it relate to other skills?

| If the skill... | Then it is |
|---|---|
| Does one thing and never invokes another skill | **Atomic** (`atm`) |
| Coordinates two or more skills through sequence, branch, parallel fan-out, loop, or delegation | **Composite** (`cmp`) |
| Selects, evaluates, generates, or modifies other skills rather than doing domain work | **Meta** (`mta`) |

Tie-breakers:
- A Composite that also decides *which* children to run based on state is still Composite if the child set is fixed at authoring time. It becomes Meta when the child set is open (any skill in the catalog).
- A skill that writes a new `SKILL.md` is Meta even if it is a single node.

## 3. Naming

Pick the convention by where the identifier lives. Use the abbreviation table in `README.md`. Never invent new codes.

| Where the name is used | Convention | Pattern | Example |
|---|---|---|---|
| LangGraph node names, CLI subcommands | Short-Prefix Matrix | `[structure]_[execution]_[name][-lifecycle]` | `atm_exe_sqlite_fts`, `mta_prc_planner-eph` |
| Telemetry spans, checkpoint keys | URN / Dot-Notation | `[domain].[structure].[execution].[lifecycle].[action]` | `core.mta.prc.eph.plan` |
| Files and directories in a monorepo | Extension Convention | `[name].[structure].[execution].[ext]` or `[name].[structure].[execution]/` | `planner.mta.prc.md`, `figma_sync.cmp.hyb/` |

Rules:
- `name`, `domain`, and `action` are `snake_case`. Codes are lowercase three-letter.
- Static lifecycle is **omitted** in Short-Prefix and Extension forms and written as the literal `static` in URN form so URNs always have five segments.
- Lifecycle is never encoded in a filename. Put it in the skill's manifest or frontmatter (`lifecycle: ephemeral`).
- Hybrid and Composite skills are directories, not single files, because they carry a `scripts/` folder or child skills.
- If a name would collide with an existing skill under any convention, disambiguate the `name` segment, not the codes.

Validation regexes:

```
short-prefix:  ^(atm|cmp|mta)_(prc|exe|hyb)_[a-z][a-z0-9_]*(-(eph|adp))?$
urn:           ^[a-z][a-z0-9_]*\.(atm|cmp|mta)\.(prc|exe|hyb)\.(static|eph|adp)\.[a-z][a-z0-9_]*$
extension:     ^[a-z][a-z0-9_]*\.(atm|cmp|mta)\.(prc|exe|hyb)(\.[a-z0-9]+|/)$
```

## 4. Choosing a type when building a new skill

Use this before writing anything. It maps the task shape to a recommended triple and points at the profile in `README.md` for the trade-offs.

| Task shape | Recommended | Why |
|---|---|---|
| Set standards, define a review rubric, plan architecture | Atomic / Procedural / Static | Open-ended reasoning; no deterministic step to protect. |
| Call one API, run one query, mutate one file deterministically | Atomic / Executable / Static | Predictable, permission-gateable; the model only routes. |
| Multi-step change where *what* to change is a judgment call but *how* must be exact (refactors, migrations) | Atomic or Composite / Hybrid / Static | Model plans, scripts mutate. See Hybrid trade-offs. |
| A fixed pipeline of known steps (lint, test, build, report) | Composite / Executable or Hybrid / Static | Encapsulate the topology; children stay reusable. |
| Knowledge needed at one phase of a long session | any / any / Ephemeral | Keeps context lean; accept retrieval risk. |
| Flaky dependency needing escalating backoff or graded regeneration | Atomic / Executable or Hybrid / Adaptive | Self-correcting; set an explicit retry ceiling. |
| Route between skills, detect gaps, author or repair skills | Meta / Procedural / Static or Ephemeral | Control-plane work; test the routing table, not the output. |

Escalation rule: start Procedural. Move to Hybrid only when a step has caused, or clearly could cause, an unsafe or non-reproducible mutation. Move to Executable only when the parameter space is fully known. Do not build Adaptive or Meta skills until a Static, non-Meta version has been shown to be insufficient.

## 5. Reviewing an existing skill

When asked to review or audit a skill against this nomenclature, check and report each item:

1. **Triple stated.** All three axes are recorded (frontmatter, manifest, or filename plus manifest). Missing Lifecycle is acceptable only if the skill is Static.
2. **Codes match content.** A `.prc.md` that ships `scripts/` is misfiled; a `.hyb/` with no scripts is misfiled.
3. **Name valid** under the convention for where it is used (regexes in section 3).
4. **Trade-offs acknowledged.** The skill's documentation names the failure mode its type is prone to (drift for Procedural, rigidity for Executable, retrieval miss for Ephemeral, unbounded loop for Adaptive, whole-graph failure for Meta) and how it is mitigated.
5. **Adaptive skills have a retry ceiling.** Refuse to pass an Adaptive skill without one.
6. **Ephemeral skills have a cleanup path.** Something pops the skill from state after the node completes.

## 6. Working in this repository

- `README.md` is the source of truth for the taxonomy text. `CONTEXT.md` reuses the same sections in a different order. When you change a skill-type profile or a naming rule, change it in both files in the same commit.
- Every skill-type section keeps the same four fields in the same order: **Implementation**, **Execution**, **Best For**, **Trade-offs**. Do not add a fifth field to one type without adding it to all nine.
- The abbreviation codes (`atm cmp mta prc exe hyb eph adp`) are frozen. Adding a fourth value to an axis is a breaking change to every name in every downstream repo and needs a version bump and a migration note.
- Keep examples concrete and in the same domain family already used (SQLite FTS, Figma sync, OSV, planner) so readers can compare types side by side.
- Do not read or search `.ai/` or `.archives/` in this or any repo unless explicitly asked.
- A change to sections 2 through 5 of this file changes the behavior of the skills in section 7, which read this file at invocation time. After such a change, re-run the fixtures under `skills/claude/edge/fixtures/` and report any mismatch as a defect in the fixture or the procedure; do not edit a fixture to match an output.

## 7. Running these procedures from Claude Code

The procedures above are packaged as read-only skills under `skills/claude/edge/`, discovered through symlinks in `.claude/skills/`. They read this file and `CONTEXT.md` when invoked and contain no copy of the taxonomy. Use them instead of pasting a section into a prompt.

| Procedure | Skill | Invoke when |
|---|---|---|
| Section 2, classify on three axes | `nomenclature-classify` | You have a skill (path or description) and need its triple |
| Section 3, name under the three conventions | `nomenclature-name` | You have a triple and need identifiers, or have an identifier and need it validated or decoded |
| Section 4, choose a type before building | `nomenclature-choose-type` | You are about to write a new skill |
| Section 5, review an existing skill | `nomenclature-review` | A skill needs a PASS/FAIL verdict with evidence before merge |
| `CONTEXT.md` Mapping to LangGraph | `nomenclature-langgraph-map` | You are placing a skill in a `StateGraph` |

Each skill records its own triple and identifiers in its frontmatter; the index at `skills/claude/edge/README.md` shows how the procedure classified them and lists the fixtures.
