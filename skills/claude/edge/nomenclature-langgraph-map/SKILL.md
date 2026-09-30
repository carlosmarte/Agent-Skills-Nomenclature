---
name: nomenclature-langgraph-map
description: Map a skill, or an Agent Skills Nomenclature triple, onto its LangGraph constructs (LLM node, ToolNode, subgraph, State reducer, checkpointed config, supervisor with conditional edges) using the Mapping to LangGraph section of this repo's CONTEXT.md, delegating to nomenclature-classify when given a path. Use when asked "how does this skill map to LangGraph", "what LangGraph construct is a hybrid / ephemeral / meta skill", "where does this skill go in the StateGraph", or "node, subgraph, or reducer for this skill". Names constructs; does not generate code.
allowed-tools: Read, Glob, Grep, Skill
argument-hint: "<Structure/Execution/Lifecycle triple> | <path to a skill>"
metadata:
  nomenclature:
    structure: composite
    execution: procedural
    lifecycle: static
    short-prefix: cmp_prc_nomenclature_langgraph_map
    urn: nomenclature.cmp.prc.static.langgraph_map
    extension: nomenclature_langgraph_map.cmp.prc/
---

# nomenclature-langgraph-map

Say what a skill *is* in a LangGraph `StateGraph`: which construct carries each axis of its
triple, and what that obliges the graph author to provide.

## Before answering

1. Read `${CLAUDE_SKILL_DIR}/references/CONTEXT.md` section **Mapping to LangGraph**, all
   four subsections and the **Quick Reference** table.
2. If the file is missing, stop and say this skill needs `CONTEXT.md` from the
   Agent-Skills-Nomenclature repository, normally at
   `<repo>/skills/claude/edge/nomenclature-langgraph-map/references/CONTEXT.md`. Do not
   answer from recollection.
3. If the input is a path or a description rather than a triple, invoke
   `nomenclature-classify` first and use the triple it returns. Do not classify here.

## Procedure

1. For each axis value, take the construct from the Quick Reference row for that value and
   quote the row.
2. From the matching subsection, state the responsibilities the construct implies:
   - **Execution**: which node type does the work and what it returns to state.
   - **Lifecycle**: Static names where it is bound before compile; Ephemeral names the
     reducer, the retrieval node, and the cleanup step; Adaptive names the evaluator node, the
     checkpointer, the mutated configuration, and the retry ceiling.
   - **Structure**: Atomic is one node; Composite names the control pattern of the subgraph
     (ordered sequence, fan-out, loop, delegation); Meta names the conditional edge its
     routing string feeds.
3. Name constructs and responsibilities only. Do not write LangGraph code.

## Output

```
Triple: <Structure> / <Execution> / <Lifecycle>   (from input | from nomenclature-classify)

Execution  → <construct>   (Quick Reference: "<row>")
  provides: <what the graph author must supply>
Lifecycle  → <construct>   (Quick Reference: "<row>")
  provides: <reducer / retrieval / cleanup | evaluator / checkpointer / ceiling | compile-time binding>
Structure  → <construct>   (Quick Reference: "<row>")
  provides: <one node | subgraph pattern | conditional edge fed by routing strings>
```
