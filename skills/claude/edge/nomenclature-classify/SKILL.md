---
name: nomenclature-classify
description: Classify a skill under the Agent Skills Nomenclature on its three axes (Execution, Lifecycle, Structure) by running the ordered tables and tie-breakers in this repo's AGENT.md section 2, and return the triple with the matched rule per axis. Use when asked to "classify this skill", "what type of skill is this", "is this skill procedural or hybrid", "atomic or composite", "what's the triple for X", or when another nomenclature skill needs a triple for a path or description.
allowed-tools: Read, Glob, Grep
argument-hint: "<path to a SKILL.md or skill bundle, or a description of the skill>"
metadata:
  nomenclature:
    structure: atomic
    execution: procedural
    lifecycle: static
    short-prefix: atm_prc_nomenclature_classify
    urn: nomenclature.atm.prc.static.classify
    extension: nomenclature_classify.atm.prc.md
---

# nomenclature-classify

Return the Execution / Lifecycle / Structure triple for one skill by following the procedure
in `AGENT.md`, not from memory.

## Before answering

1. Read `${CLAUDE_SKILL_DIR}/references/AGENT.md`, sections **1. What a skill is here** and
   **2. Classification procedure** (2.1, 2.2, 2.3, including every tie-breaker).
2. If that file is missing, stop and say: this skill needs `AGENT.md` from the
   Agent-Skills-Nomenclature repository, normally reachable at
   `<repo>/skills/claude/edge/nomenclature-classify/references/AGENT.md`. Do not answer from
   recollection of the taxonomy.
3. If the input is a path, read the skill's `SKILL.md` (or manifest) and list its directory.
   Note whether it ships a `scripts/` folder, whether its instructions tell the model when to
   call bundled scripts, whether it invokes other skills, and whether it mutates its own
   configuration from feedback. Cite `path:line` for each observation.
4. If the input is a description, work from the description alone and say so.

## Procedure

For each axis, walk the table in section 2 top to bottom and stop at the **first** matching row.
Then apply that axis's tie-breakers. Record which row matched and which tie-breaker, if any,
decided it.

- **Execution** (section 2.1): Procedural, Executable, or Hybrid.
- **Lifecycle** (section 2.2): Static, Ephemeral, or Adaptive. Static is the default when
  nothing in the input makes the skill Ephemeral or Adaptive. Always state it.
- **Structure** (section 2.3): Atomic, Composite, or Meta.

Never emit a value outside the nine listed in section 1. Never invent a code.

## When an axis is ambiguous

Do not guess. Name the two values in contention, the row or tie-breaker that separates them,
and the single fact about the skill that would settle it (for example: "Adaptive if the retry
budget is persisted in state between attempts; otherwise Static").

## Output

Lead with the result, then the evidence:

```
Triple: <Structure> / <Execution> / <Lifecycle>

Execution: <value> — matched "<row text, abbreviated>" (AGENT.md §2.1); tie-breaker: <which or none>
  evidence: <path:line or "from description">
Lifecycle: <value> — matched "<row>" (AGENT.md §2.2); tie-breaker: <which or none>
  evidence: ...
Structure: <value> — matched "<row>" (AGENT.md §2.3); tie-breaker: <which or none>
  evidence: ...

Open: <none, or the ambiguity and the fact that settles it>
```

If the caller also wants names, point them at `nomenclature-name` with this triple; do not
produce names here.
