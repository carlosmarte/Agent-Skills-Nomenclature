# Naming fixtures

Run each through `nomenclature-name`. Regexes are the three in `AGENT.md` section 3.

## Produce

| # | Input | Expected Short-Prefix | Expected URN | Expected Extension |
| --- | --- | --- | --- | --- |
| N1 | Atomic / Executable / Static, name `sqlite_fts`, domain `data`, action `sqlite_fts_query` | `atm_exe_sqlite_fts` | `data.atm.exe.static.sqlite_fts_query` | `sqlite_fts.atm.exe.ts` (or another language extension; must be a file) |
| N2 | Composite / Hybrid / Static, name `figma_sync`, domain `design`, action `figma_sync` | `cmp_hyb_figma_sync` | `design.cmp.hyb.static.figma_sync` | `figma_sync.cmp.hyb/` (directory) |
| N3 | Meta / Procedural / Ephemeral, name `planner`, domain `core`, action `plan` | `mta_prc_planner-eph` | `core.mta.prc.eph.plan` | `planner.mta.prc.md` (lifecycle not in the filename; goes in frontmatter) |
| N4 | Atomic / Executable / Adaptive, name `github_retry`, domain `integrations`, action `retry` | `atm_exe_github_retry-adp` | `integrations.atm.exe.adp.retry` | `github_retry.atm.exe.rs` (or another language extension) |
| N5 | Atomic / Procedural / Static, name given as `Code Review` | `atm_prc_code_review` | needs domain and action; skill must ask or state a placeholder | `code_review.atm.prc.md` |
| N6 | "Streaming" as a fourth Execution value, name `feed` | refused: no such code; suggest disambiguating `name` | refused | refused |

## Validate

| # | Input | Expected | Convention | Decoded triple | Fix (if invalid) |
| --- | --- | --- | --- | --- | --- |
| V1 | `atm_exe_sqlite_fts-adp` | valid | Short-Prefix | Atomic / Executable / Adaptive | |
| V2 | `Atm_exe_sqlite_fts` | invalid | Short-Prefix | | lowercase the code: `atm_exe_sqlite_fts` |
| V3 | `atm_exe_sqlite-fts` | invalid | Short-Prefix | | hyphen only introduces a lifecycle code: `atm_exe_sqlite_fts` |
| V4 | `core.mta.prc.eph.plan` | valid | URN | Meta / Procedural / Ephemeral | |
| V5 | `core.mta.prc.plan` | invalid | URN | | four segments; lifecycle must be written: `core.mta.prc.static.plan` |
| V6 | `figma_sync.cmp.hyb/` | valid | Extension | Composite / Hybrid / Static | |
| V7 | `figma_sync.cmp.hyb.md` | valid by regex, but misfiled: Hybrid must be a directory | Extension | Composite / Hybrid / Static | `figma_sync.cmp.hyb/` |
| V8 | `planner.mta.prc.eph.md` | invalid | Extension | | lifecycle is not encoded in filenames: `planner.mta.prc.md` + `lifecycle: ephemeral` in frontmatter |
