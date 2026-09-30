---
name: nomenclature-name
description: Produce or validate skill identifiers under the Agent Skills Nomenclature's three naming conventions (Short-Prefix for LangGraph nodes and CLI, URN dot-notation for telemetry, Extension for monorepo files) from a triple plus a name, using the abbreviation table and validation regexes in this repo's CONTEXT.md and AGENT.md section 3. Use when asked to "name this skill", "what should I call this skill", "give me the short-prefix / URN / file name for X", "is atm_exe_foo a valid name", "validate this skill name", or "decode this skill name into its triple".
allowed-tools: Read
argument-hint: "<Structure/Execution/Lifecycle + name [domain] [action]> | <an identifier to validate>"
metadata:
  nomenclature:
    structure: atomic
    execution: procedural
    lifecycle: static
    short-prefix: atm_prc_nomenclature_name
    urn: nomenclature.atm.prc.static.name
    extension: nomenclature_name.atm.prc.md
---

# nomenclature-name

Turn a triple and a name into the three identifiers, or validate and decode an identifier
someone else wrote. Every code and every regex comes from the reference documents.

## Before answering

1. Read `${CLAUDE_SKILL_DIR}/references/CONTEXT.md` section **Naming Conventions** (the
   Abbreviation Table and the three conventions), and
   `${CLAUDE_SKILL_DIR}/references/AGENT.md` section **3. Naming** (rules and the three
   validation regexes).
2. If either file is missing, stop and say which one this skill needs and that it normally
   lives at `<repo>/skills/claude/edge/nomenclature-name/references/`. Do not answer from
   recollection.
3. If the caller gave a path or a description instead of a triple, tell them to run
   `nomenclature-classify` first, or ask for the triple. Do not classify here.

## Producing names

Inputs: Structure, Execution, Lifecycle, `name`; for the URN also `domain` and `action`. If
`domain` or `action` is missing, ask for it or state the placeholder you used.

1. Normalize `name`, `domain`, and `action` to `snake_case` (AGENT.md §3 rules).
2. Look up each axis value in the Abbreviation Table. If any value has no code, refuse: the
   codes are frozen (AGENT.md §6); suggest disambiguating the `name` segment instead.
3. Build all three forms exactly as the three convention patterns in AGENT.md §3 specify.
   Static is **omitted** in Short-Prefix and Extension and written as the literal `static` in
   the URN.
4. Extension form: Hybrid and Composite skills are directories (`.../`), never a file; for the
   others choose the extension from the Extension Convention examples (`.md` for Procedural,
   a language extension for Executable) and say why.
5. Never place lifecycle in an Extension name; say it belongs in the manifest or frontmatter.
6. Test each produced identifier against its regex and show the regex used.

## Validating a name

1. Detect the convention from shape: underscores and an `atm|cmp|mta` prefix mean
   Short-Prefix; five dot-separated segments mean URN; a name ending in an extension or `/`
   with two code segments means Extension.
2. Test it against that convention's regex from AGENT.md §3. Report valid or invalid.
3. Decode the codes back to the triple using the Abbreviation Table. In Short-Prefix and
   Extension forms an absent lifecycle code means Static.
4. If invalid, name the regex it failed and the smallest change that makes it valid.

## Output

Lead with the result:

```
Triple: <Structure> / <Execution> / <Lifecycle>
Short-Prefix: <id>      regex ok
URN:          <id>      regex ok
Extension:    <id>      regex ok   (directory because <Hybrid|Composite> | file because <reason>)
Notes: <lifecycle goes in the manifest | placeholder used for domain | ...>
```

or, for validation:

```
<identifier>: valid | invalid
Convention: <Short-Prefix | URN | Extension>
Decoded triple: <...>   (or: cannot decode because <segment> is not a frozen code)
Fix: <smallest change>  (only when invalid)
```
