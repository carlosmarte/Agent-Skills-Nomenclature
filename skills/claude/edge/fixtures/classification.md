# Classification fixtures

Inputs and the triple `AGENT.md` section 2 prescribes. Run each through `nomenclature-classify`
and compare. Every fixture is a prose description, so the skill must say it worked "from
description". Expected triples are written Structure / Execution / Lifecycle.

## One per skill type

| # | Input | Expected | Exercises |
| --- | --- | --- | --- |
| C1 | A `SKILL.md` that tells the agent how to write BDD acceptance criteria. No scripts. Loaded at boot. Invokes nothing. | Atomic / Procedural / Static | Procedural row |
| C2 | A Node script behind a JSON schema that runs one FTS5 query against a local SQLite index and returns rows. The agent only fills in the query. | Atomic / Executable / Static | Executable row |
| C3 | A bundle with a `SKILL.md` that tells the agent when to call `scripts/apply_codemod.mjs` to rewrite imports, after the agent decides which files need it. | Atomic / Hybrid / Static | Hybrid row; "scripts shipped with the skill" |
| C4 | A database-migration guide that a retrieval node appends to `active_skills` when the migration node starts and a cleanup node pops when it finishes. | Atomic / Procedural / Ephemeral | Ephemeral row |
| C5 | A GitHub API caller whose retry budget and backoff multiplier live in thread state; an evaluator node raises them after a rate-limit response and the next attempt reads the new values from the checkpoint. Ceiling: 5 attempts. | Atomic / Executable / Adaptive | Adaptive row |
| C6 | House coding standards injected into the base State at kickoff and never removed. | Atomic / Procedural / Static | Static row |
| C7 | A subgraph that runs lint, then test, then build, then report, each an existing executable skill, in fixed order, returning one verdict. | Composite / Executable / Static | Composite row; fixed child set |
| C8 | A `Send()` fan-out that maps one existing OSV-sync executable over a list of package manifests and aggregates results. | Composite / Executable / Static | Composite row; parallel pattern |
| C9 | A supervisor prompt that inspects the plan in State, picks which of any skill in the catalog to run next, and returns a routing string. Does no domain work. | Meta / Procedural / Static | Meta row; open child set |

## One per tie-breaker

| # | Input | Expected | Exercises |
| --- | --- | --- | --- |
| T1 | A `SKILL.md` whose instructions say "use `grep` and `sed` to find and replace the token". Ships no scripts. | Atomic / Procedural / Static | §2.1: mentioning a generic tool is still Procedural |
| T2 | A single script with a 300-line usage guide; the agent fills three parameters and never reasons about intermediate steps. | Atomic / Executable / Static | §2.1: a long guide does not make it Hybrid |
| T3 | A skill available for the whole session but invoked in perhaps one task in ten. | Atomic / Procedural / Static | §2.2: availability, not frequency, decides Static |
| T4 | A Figma sync that retries three times with identical parameters on failure. Nothing is persisted between attempts. | Atomic / Executable / Static | §2.2: same-parameter retry is not Adaptive |
| T5 | A skill retrieved at one node and popped afterward, whose prompt weights are also mutated from feedback and checkpointed; retrieval misses are its dominant failure. | Atomic / Procedural / Ephemeral, with Adaptive recorded in the manifest | §2.2: pick the dominant failure mode, record the other |
| T6 | A pipeline that reads State to decide whether to run the "test" child, but its child set (lint, test, build) is fixed at authoring time. | Composite / Executable / Static | §2.3: fixed child set stays Composite |
| T7 | A single node that writes a new `SKILL.md` for a gap it detected. | Meta / Procedural / Static | §2.3: writing a SKILL.md is Meta even as one node |

## Recording results

Record a run as: fixture id, date, session, returned triple, matched or mismatched. A mismatch
means either the fixture's expectation or the procedure in `AGENT.md` is wrong; report which,
do not adjust the fixture to match the output.
