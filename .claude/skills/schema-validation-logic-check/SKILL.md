---
name: schema-validation-logic-check
description: Runs a reasonableness/logic check across a bioschemas profile's two draft source files (the cardinality/marginality file and the JSON-Schema-rules file) and writes a markdown report of things a curator should double-check -- properties that only exist in one file, properties with no real JSON-Schema validation keywords, and properties marked owl:cardinality "many" that don't use oneOf/anyOf. Use this whenever the user asks to sanity-check, QA, validate, or logic-check a bioschemas profile merge, or asks things like "does this merge make sense", "check for mismatches between the two profiles", "check the cardinality against the schema rules", or "is anything missing before I merge these". Can run before or after schema-validation-merger's actual merge -- it only reads the two source files directly and does not require a merged output to exist yet.
---

# Schema Validation Logic Check

## What this does

Companion check to [`schema-validation-merger`](../schema-validation-merger/SKILL.md). Where that skill combines
the two source files into one profile, this skill inspects the two source files
*without* merging them and flags things worth a curator's attention:

1. **Properties present in only one source file** — flagged with which kind of
   information is missing (cardinality/marginality info, or JSON-Schema validation
   rules).
2. **Properties with no real JSON-Schema validation rules** — a property can nominally
   "exist" in the schema-rules file while only ever having carried a `description`; once
   that's set aside there's nothing left to validate against, which check 1 alone
   wouldn't catch.
3. **`owl:cardinality: many` without `oneOf`/`anyOf`** — a property marked as repeatable
   should almost always express that in its JSON-Schema rules as a choice between a
   single value and an array of that value (`oneOf`/`anyOf`). If it's marked `many` but
   the rules don't offer that, it's flagged so a curator can check whether the rules
   were actually written to support multiple values.

This is a **draft QA pass**, not an auto-fix: every flag is a prompt for human review,
not a claim that something is definitely wrong. Nothing here modifies the source files.

## Directory layout

```
schema_validation_merger/                     <- project root; run Claude Code from here
├── .claude/
│   └── skills/
│       ├── schema-validation-merger/         <- companion merge skill (see its own SKILL.md)
│       └── schema-validation-logic-check/    <- this skill
│           ├── SKILL.md                      (this file)
│           └── scripts/
│               └── validation_logic_check.ipynb  <- the notebook that runs the checks
├── files/to merge/                             <- same input files this check reads
└── output/                                      <- merged-*.json and *_validation_check_*.md land here
```

Everything this skill needs lives under `schema_validation_merger/`, and the notebook's
`ROOT_DIR` parameter points at that folder by absolute path — running Claude Code with
`schema_validation_merger/` as the working directory is enough for both skills to be
auto-discovered later.

## Environment

Run the notebook's kernel against the **`bioschemas`** conda environment
(`Anaconda3\envs\bioschemas`). Only the standard library is used (`json`, `pathlib`,
`datetime`) — no extra installs needed.

## Workflow

1. **Open [scripts/validation_logic_check.ipynb](scripts/validation_logic_check.ipynb)**
   and confirm the Parameters cell's filenames match what's in `files/to merge/` (this
   is a hint only — role detection cross-checks actual content, same as the merger).
2. **Run all cells top to bottom.** There is no confirmation gate in this notebook (it
   doesn't write to or alter the source files, and doesn't touch the merged output) —
   unlike the merger's mandatory pause before its full merge, this one is safe to run
   straight through.
3. The last cell writes a markdown report to `output/<ClassName>_validation_check_<date>.md`,
   where `<ClassName>` comes from the profile's `rdfs:label` (falling back to the local
   part of its `@id` if that's missing).
4. Read the report back to the user (or open it) and summarize the counts from each of
   the three checks — don't just say "done," since the value here is in what got
   flagged.

## Notes for future runs

- If a property is flagged in check 1 or 2, that's information for the curator merging
  the files, not something this skill or the merger should silently resolve on its own.
- If check 3 flags a property, don't assume the JSON-Schema rules are wrong — it might
  be that `owl:cardinality` itself is stale. Surface both possibilities rather than
  picking one.
- Re-run this after editing either source file — it's cheap and has no side effects on
  the inputs.
