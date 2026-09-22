---
name: generate_missing_validation
description: Asks the user which schema.org (or bioschemas) type(s) a property should accept, then generates a JSON-Schema "anyOf" validation rule object in the same house style already used across the bioschemas profile drafts (bare object references for classes like CreativeWork/ImageObject, string/format shapes for primitives like Text/URL, plus an array-of variant for each), and writes it to a dated JSON file. Use this whenever a property is missing its JSON-Schema validation rules -- especially the properties schema-validation-logic-check flags as "no real JSON-Schema validation rules" -- or whenever the user asks to draft/generate/fill in validation rules for a property, asks "what types should this property accept", or wants a standalone anyOf/oneOf rule object without touching the actual merge or profile files.
---

# Generate Missing Validation

## What this does

Companion to [`schema-validation-logic-check`](../schema-validation-logic-check/SKILL.md):
that skill flags properties with no real JSON-Schema validation rules (or that only
exist in one of the two source files); this skill drafts replacement rules for one such
property by asking what type(s) of value it should accept, then building the `anyOf`
rule object the same way it would be hand-written elsewhere in these profiles.

This does **not** touch `files/to merge/`, the merged output, or either skill's source
files — it only produces a standalone JSON file with the drafted rule, for a curator to
review and paste into whichever file needs it.

## Directory layout

```
schema_validation_merger/                        <- project root; run Claude Code from here
├── .claude/
│   └── skills/
│       ├── schema-validation-merger/
│       ├── schema-validation-logic-check/
│       └── generate_missing_validation/           <- this skill
│           ├── SKILL.md                            (this file)
│           └── scripts/
│               └── generate_missing_validation.ipynb
└── output/                                          <- <date>_expectedtypes.json lands here
```

## Environment

Run the notebook's kernel against the **`bioschemas`** conda environment
(`Anaconda3\envs\bioschemas`). Only the standard library is used (`json`, `pathlib`,
`datetime`) — no extra installs needed.

## Workflow

1. **Figure out what property this is for and what types it should accept.** If you
   (Claude) are running this on the user's behalf, ask them directly in chat which
   schema.org type(s) — or bioschemas CURIEs like `bioschemastypesdrafts:Sample` — the
   property should be able to hold. Don't guess from the property name alone; if it's
   ambiguous, offer the schema-rules source file's existing patterns as reference points
   (e.g. "this looks similar to how `citation` accepts `CreativeWork`").

2. **Open [scripts/generate_missing_validation.ipynb](scripts/generate_missing_validation.ipynb).**
   In the Parameters cell, set `expected_types` to the confirmed list (e.g.
   `["CreativeWork", "ImageObject", "URL", "Text", "bioschemastypesdrafts:Sample"]`) and
   optionally `PROPERTY_NAME` for a clearer printed summary. Setting `expected_types`
   here skips the notebook's own interactive `input()` prompt, which is meant for a
   human running the notebook directly in Jupyter, not for a non-interactive run.

3. **Run the rest of the notebook.** It classifies each type — schema.org primitive
   datatypes (`Text`, `URL`, `Number`, `Integer`, `Boolean`, `Date`, `DateTime`, `Time`)
   become a `string`/`number`/... shape; anything else (a schema.org class or a
   bioschemas CURIE) becomes a bare object reference — and builds an `anyOf` list with
   both a singular and an array-of variant for each type, matching how properties like
   `citation` and `alternateIdentifier` are already written in these profiles.

4. **Show the printed rule object to the user** before treating the file as final —
   this is a draft for a curator to review, not a guaranteed-correct rule, especially
   for types the classifier hasn't seen before.

5. The last cell writes the result to `output/<date>_expectedtypes.json` (e.g.
   `2026-09-22_expectedtypes.json`). If a file for today already exists — say this was
   already run once today for a different property — it appends a numeric suffix
   instead of overwriting the earlier run.

## Notes for future runs

- This intentionally leaves referenced classes "bare" (`properties: {}`, `required:
  []`) rather than expanding their own sub-properties, matching how `citation` (bare)
  differs from `collector` (fully expanded `Organization`/`Person`). If the user wants a
  fully expanded object shape instead, that's a bigger ask than this skill covers — flag
  it rather than guessing at sub-properties.
- `oneOf` vs `anyOf`: this always builds `anyOf`, matching how every multi-shape
  property in Ginger's file (not just the two-branch string/array cases) uses `anyOf`
  rather than `oneOf`.
- The output filename is generic (`<date>_expectedtypes.json`), not tied to the property
  name — if the user is running this repeatedly across a session for several properties,
  say which file corresponds to which property when reporting back, since the filename
  alone won't tell them.
