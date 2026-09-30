# Review fixtures

Six deliberately defective skills. Each violates exactly one of the six checks in `AGENT.md`
section 5. Run `nomenclature-review` on each directory; the expected result is a **FAIL**
verdict whose failing row is the one named here and no other.

| Fixture | Violates | What is wrong | Expected failing check |
| --- | --- | --- | --- |
| `f1-no-triple/` | 1 Triple stated | No axes in frontmatter, and the skill is Ephemeral, so omission is not excused | 1 |
| `f2-codes-mismatch/` | 2 Codes match content | Declared `atm.prc` and `.prc.md`, but ships `scripts/` and tells the model when to call it | 2 |
| `f3-invalid-name/` | 3 Name valid | `atm_exe_osv-sync` has a hyphen in the name segment; `security.atm.exe.osv_sync` has four segments | 3 |
| `f4-no-tradeoffs/` | 4 Trade-offs acknowledged | No failure mode or mitigation anywhere in the skill | 4 |
| `f5-adaptive-no-ceiling/` | 5 Adaptive retry ceiling | Adaptive, "retry until the call succeeds", no ceiling | 5 |
| `f6-ephemeral-no-cleanup/` | 6 Ephemeral cleanup path | Ephemeral, "nothing removes it afterward" | 6 |

These directories are fixtures, not skills: nothing under `fixtures/` is linked from
`.claude/skills/`, so Claude Code never loads them.
