# Claude Code skills for the nomenclature

Five read-only skills that run the procedures in [`AGENT.md`](../../../AGENT.md) and
[`CONTEXT.md`](../../../CONTEXT.md) from inside Claude Code. They read those documents at
invocation time and never restate them, so a change to a tie-breaker or a regex changes their
behavior with no edit here. Specified by PRD 0001 (`.ai/planners/Agent-Skills-Nomenclature/PRDs/0001-claude-edge-skills/`).

## The skills

| Skill | Runs | Triple | Short-Prefix | URN | Extension |
| --- | --- | --- | --- | --- | --- |
| [`nomenclature-classify`](nomenclature-classify/SKILL.md) | `AGENT.md` §2, classify on three axes | Atomic / Procedural / Static | `atm_prc_nomenclature_classify` | `nomenclature.atm.prc.static.classify` | `nomenclature_classify.atm.prc.md` |
| [`nomenclature-name`](nomenclature-name/SKILL.md) | `AGENT.md` §3 and `CONTEXT.md` Naming Conventions, produce and validate names | Atomic / Procedural / Static | `atm_prc_nomenclature_name` | `nomenclature.atm.prc.static.name` | `nomenclature_name.atm.prc.md` |
| [`nomenclature-choose-type`](nomenclature-choose-type/SKILL.md) | `AGENT.md` §4, recommend a type for a new skill | Atomic / Procedural / Static | `atm_prc_nomenclature_choose_type` | `nomenclature.atm.prc.static.choose_type` | `nomenclature_choose_type.atm.prc.md` |
| [`nomenclature-review`](nomenclature-review/SKILL.md) | `AGENT.md` §5, six-check review with a verdict | Composite / Procedural / Static | `cmp_prc_nomenclature_review` | `nomenclature.cmp.prc.static.review` | `nomenclature_review.cmp.prc/` |
| [`nomenclature-langgraph-map`](nomenclature-langgraph-map/SKILL.md) | `CONTEXT.md` Mapping to LangGraph | Composite / Procedural / Static | `cmp_prc_nomenclature_langgraph_map` | `nomenclature.cmp.prc.static.langgraph_map` | `nomenclature_langgraph_map.cmp.prc/` |

Triggers, in the words a request uses:

- "classify this skill", "what type of skill is this" → `nomenclature-classify`
- "name this skill", "is `atm_exe_foo` valid", "decode this name" → `nomenclature-name`
- "what type of skill should this be", said before building → `nomenclature-choose-type`
- "review / audit this skill against the nomenclature" → `nomenclature-review`
- "how does this map to LangGraph", "node, subgraph, or reducer" → `nomenclature-langgraph-map`

## How they were classified (applying the procedure to itself)

Each skill was run through `AGENT.md` §2 as written, first matching row wins:

- **Execution:** all five are instructions the model reads and act through generic tools
  (`Read`, `Glob`, `Grep`, `Skill`); none ships `scripts/`. Procedural.
- **Lifecycle:** Claude Code loads the descriptions at boot and the body on invocation, and
  nothing pops a skill from context afterward; the tie-breaker "available all session but only
  used sometimes is Static" applies. Static.
- **Structure:** `classify`, `name`, and `choose-type` do one thing and invoke no other skill:
  Atomic (first row). `review` runs `classify` and `name` in sequence for its checks 1 and 3,
  and `langgraph-map` delegates to `classify` when handed a path: Composite (second row).
  Note that the Meta row was never reached for any of them, although all five operate on
  skills rather than on a business problem; see Q3 and Q7 in the PRD's section 06.

The three identifiers are recorded in each skill's frontmatter `metadata.nomenclature`
block, because Claude Code restricts skill names to lowercase letters, digits, and hyphens,
so the Extension-convention name cannot be the directory name.

## How Claude Code finds them

Claude Code discovers skills only under `.claude/skills/` directories (and personal, plugin,
and `--add-dir` locations), not under an arbitrary path. This tree is the source of truth, and
the repo commits one symlink per skill:

```
.claude/skills/<skill-name>  ->  ../../skills/claude/edge/<skill-name>
```

Adding a skill here means adding its symlink there. Each skill also carries
`references/AGENT.md` and `references/CONTEXT.md`, symlinks five levels up to the repo's own
documents; they resolve through the `.claude/skills/` link because symlinks resolve relative
to where they really live.

To use these skills from another checkout or from `~/.claude/skills/`, **symlink** the skill
directory rather than copying it. A copied directory loses the two reference links, and the
skill will then stop and name the file it needs rather than answer from memory.

## Fixtures

[`fixtures/classification.md`](fixtures/classification.md) (nine types, seven tie-breakers),
[`fixtures/naming.md`](fixtures/naming.md) (six produce, eight validate), and
[`fixtures/review/`](fixtures/review/README.md) (six skills, each violating one check). Run
them by invoking the skills by hand; record a mismatch as a defect in either the fixture or
`AGENT.md`, never by adjusting the fixture to match.
