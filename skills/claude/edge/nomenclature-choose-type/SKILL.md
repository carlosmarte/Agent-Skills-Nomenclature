---
name: nomenclature-choose-type
description: Recommend which kind of skill to build, as an Agent Skills Nomenclature triple (Structure / Execution / Lifecycle), from the task-shape table and escalation rule in this repo's AGENT.md section 4, naming the failure mode of the chosen type and the identifiers to use. Use before writing a new skill, when asked "what type of skill should this be", "should this be procedural or hybrid", "do I need a composite for this", "should this skill be adaptive", or "what kind of skill fits this task".
allowed-tools: Read
argument-hint: "<what the new skill will do, in a sentence or two>"
metadata:
  nomenclature:
    structure: atomic
    execution: procedural
    lifecycle: static
    short-prefix: atm_prc_nomenclature_choose_type
    urn: nomenclature.atm.prc.static.choose_type
    extension: nomenclature_choose_type.atm.prc.md
---

# nomenclature-choose-type

Recommend a triple for a skill that does not exist yet. This is the step before writing
anything; it decides the shape, not the content.

## Before answering

1. Read `${CLAUDE_SKILL_DIR}/references/AGENT.md` sections **4. Choosing a type when building
   a new skill** (the task-shape table and the escalation rule) and **5. Reviewing an existing
   skill** (item 4 lists the failure mode per type).
2. Read `${CLAUDE_SKILL_DIR}/references/CONTEXT.md` for the profile of any type you are
   about to recommend (its **Best For** and **Trade-offs** fields), so the recommendation
   quotes the trade-off rather than paraphrasing it.
3. If either file is missing, stop and say which one this skill needs and that it normally
   lives at `<repo>/skills/claude/edge/nomenclature-choose-type/references/`. Do not answer
   from recollection.

## Procedure

1. Restate the task in one sentence: what the skill does, what it mutates, what it depends
   on, whether the parameter space is fully known, whether it needs to change between runs.
2. Find the first row of the task-shape table in AGENT.md §4 that matches. If none does, say
   so and reason from the profiles in CONTEXT.md, naming which profile fields you used.
3. Apply the escalation rule in AGENT.md §4 explicitly. The default is Procedural, Static,
   non-Meta. Leaving that default requires a specific reason of the kind the rule names:
   - Hybrid: name the step that has caused, or clearly could cause, an unsafe or
     non-reproducible mutation.
   - Executable: state that the parameter space is fully known, and what it is.
   - Adaptive or Meta: name the Static, non-Meta version that was shown insufficient, or say
     plainly that none has been tried and recommend the simpler type first.
4. Name the failure mode of the recommended type from AGENT.md §5 item 4 and one mitigation
   drawn from the type's Trade-offs field in CONTEXT.md.
5. Produce the three identifiers by following `nomenclature-name` with the recommended triple
   and a snake_case name you propose from the task (ask for `domain` and `action` if the URN
   needs them and none are obvious).

## Output

Lead with the recommendation:

```
Recommended: <Structure> / <Execution> / <Lifecycle>
Matched: AGENT.md §4 row "<task shape>" | no row; reasoned from <profile> Best For / Trade-offs
Escalation: stays at the default | leaves it because <specific reason of the kind the rule names>
Failure mode to guard: <drift | rigidity | retrieval miss | unbounded loop | whole-graph failure> — mitigate by <...>
Names: <Short-Prefix> · <URN> · <Extension>
Also considered: <the next-closest triple and why it lost>
```
