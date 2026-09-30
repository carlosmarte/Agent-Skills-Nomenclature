---
name: nomenclature-review
description: Review or audit an existing skill against the Agent Skills Nomenclature using the six checks in this repo's AGENT.md section 5 (triple stated, codes match content, name valid, trade-offs acknowledged, Adaptive retry ceiling, Ephemeral cleanup path), delegating to nomenclature-classify and nomenclature-name, and returning a PASS or FAIL verdict with path:line evidence per check. Use when asked to "review this skill against the nomenclature", "audit this SKILL.md", "does this skill follow the taxonomy", "is this skill filed correctly", or "nomenclature check before merge". Reports only; never edits.
allowed-tools: Read, Glob, Grep, Skill
argument-hint: "<path to the skill directory or SKILL.md>"
metadata:
  nomenclature:
    structure: composite
    execution: procedural
    lifecycle: static
    short-prefix: cmp_prc_nomenclature_review
    urn: nomenclature.cmp.prc.static.review
    extension: nomenclature_review.cmp.prc/
---

# nomenclature-review

Run the six-item review from `AGENT.md` on one skill and return one verdict with evidence.
This skill is a sequence: it delegates check 1 to `nomenclature-classify` and check 3 to
`nomenclature-name`, then performs checks 2, 4, 5, and 6 itself, so a review never disagrees
with a classify or name answer on the same input.

## Before answering

1. Read `${CLAUDE_SKILL_DIR}/references/AGENT.md` section **5. Reviewing an existing skill**
   for the six checks, and section **3. Naming** for what "valid" means in check 3.
2. If the file is missing, stop and say this skill needs `AGENT.md` from the
   Agent-Skills-Nomenclature repository, normally at
   `<repo>/skills/claude/edge/nomenclature-review/references/AGENT.md`. Do not review from
   recollection.
3. Read the target skill: its `SKILL.md` or manifest, its frontmatter, and a listing of its
   directory. Note every `path:line` you will cite.

## Procedure

Run the checks in the order AGENT.md §5 lists them. Each is **pass**, **fail**, or **n/a**
with a reason.

1. **Triple stated.** Find the three axes in frontmatter, manifest, or filename plus manifest.
   Invoke `nomenclature-classify` on the target to get the triple the content implies, and
   compare. A missing Lifecycle is acceptable only if the skill is Static.
2. **Codes match content.** Compare the declared or filename codes with what the directory
   holds: a `.prc.md` that ships `scripts/` is misfiled; a `.hyb/` with no scripts is
   misfiled; a declared Atomic that invokes other skills is misfiled. Cite the file and line.
3. **Name valid.** Invoke `nomenclature-name` in validation mode on each identifier the skill
   declares or is filed under, for the convention where it is used. Report each result.
4. **Trade-offs acknowledged.** The skill's documentation names the failure mode its type is
   prone to (the list is in AGENT.md §5 item 4) and how it is mitigated. Quote the line.
5. **Adaptive skills have a retry ceiling.** n/a unless the skill is Adaptive. An Adaptive
   skill with no explicit ceiling is a **fail** and the verdict is FAIL, never a warning.
6. **Ephemeral skills have a cleanup path.** n/a unless the skill is Ephemeral. Something
   must pop the skill from state after its node completes; cite it or fail.

Verdict is PASS only if no check failed. Do not edit the skill; say what to change and where.

## Output

```
Verdict: PASS | FAIL   (<n> of 6 pass, <m> n/a)

| # | Check | Result | Evidence |
| 1 | Triple stated | pass/fail | <path:line>; declared <triple>, content implies <triple> |
| 2 | Codes match content | ... | ... |
| 3 | Name valid | ... | <identifier>: valid/invalid (<convention>) |
| 4 | Trade-offs acknowledged | ... | <path:line> or "not found" |
| 5 | Adaptive retry ceiling | pass/fail/n/a | ... |
| 6 | Ephemeral cleanup path | pass/fail/n/a | ... |

To fix: <one line per failing check: what to change, in which file>
```
