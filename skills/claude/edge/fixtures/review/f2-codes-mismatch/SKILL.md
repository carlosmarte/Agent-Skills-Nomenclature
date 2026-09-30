---
name: codemod-imports
description: Rewrite import paths across a package. Filed as atm/prc (procedural). Failure mode: drift; mitigated by a checklist.
metadata:
  nomenclature:
    structure: atomic
    execution: procedural
    lifecycle: static
    extension: codemod_imports.atm.prc.md
---
# codemod-imports
Decide which files need the rewrite, then run `scripts/apply_codemod.mjs <file>` on each one.
