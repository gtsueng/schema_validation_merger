---
name: schema-validation-merger
description: Merges the $validation block of two bioschemas profile draft JSON files (one holding descriptions/owl:cardinality/required-recommended-optional marginality, the other holding the actual JSON-Schema validation rules like anyOf/oneOf/type/items) into a single combined profile, normalizing draft namespace prefixes (e.g. NicktempSample, gtest) to "bioschemasdrafts" along the way. Use this whenever the user wants to combine, reconcile, or merge bioschemas/schema.org profile drafts, especially phrases like "merge the validation", "combine Nick's and Ginger's profile", "merge $validation", "merge cardinality with the schema rules", or anything about reconciling two draft bioschemas JSON profiles in schema_validation_merger. Always run this as a Jupyter notebook under the bioschemas conda environment, and always pause for human confirmation on a single test property before merging everything else.
---

# Schema Validation Merger

## What this does

Two people often draft the same bioschemas profile independently:

- One person's file has good **descriptions**, **`owl:cardinality`** values, and the
  **marginality** lists (`required` / `recommended` / `optional`) — but weak or missing
  JSON-Schema validation rules.
- The other person's file has solid **JSON-Schema validation rules** (`anyOf`, `oneOf`,
  `type`, `items`, `format`, etc.) for each property — but the `description` fields and
  marginality lists are placeholders or empty.

This skill merges the two `$validation` blocks into one profile: descriptions +
cardinality + marginality from the first file, JSON-Schema rules from the second,
combined per-property. It also normalizes any draft namespace prefixes (e.g.
`NicktempSample`, `gtest`) to the shared `bioschemasdrafts` namespace.

**This produces a draft to accelerate a curator's review — it is not an authoritative
auto-merge.** Anything ambiguous (a property missing from one side, a role that can't be
auto-detected) must be surfaced in the notebook's output, not silently guessed.

## Directory layout

```
schema_validation_merger/                     <- project root; run Claude Code from here
├── .claude/
│   └── skills/
│       ├── schema-validation-merger/         <- this skill
│       │   ├── SKILL.md                      (this file)
│       │   └── scripts/
│       │       └── merge_validation.ipynb    <- the notebook that does the actual merge
│       └── schema-validation-logic-check/    <- companion QA skill (see its own SKILL.md)
├── files/
│   └── to merge/                     <- input JSON files live here
│       ├── Nick-Sample-Profile.json  (example: cardinality/marginality source)
│       └── Ginger-Sample-Profile.json (example: JSON-Schema rules source)
└── output/                           <- merged-*.json lands here (created on run)
```

Everything this skill needs — inputs, notebook, and output — lives under
`schema_validation_merger/`, and the notebook's `ROOT_DIR` parameter points at that
folder by absolute path. Running Claude Code with `schema_validation_merger/` as the
working directory is enough for this skill to be auto-discovered later; nothing outside
this directory is required.

Filenames will vary run to run — the notebook auto-detects which input file plays which
role rather than hardcoding "Nick" or "Ginger" (see Step 2 below), so the same skill
works on future profile pairs dropped into `files/to merge/`.

## Environment

Run the notebook's kernel against the **`bioschemas`** conda environment
(`Anaconda3\envs\bioschemas`). The notebook only uses Python's standard library
(`json`, `pathlib`, `copy`) — no extra packages need to be installed in that env.

## Workflow — follow these steps in order

1. **Open [scripts/merge_validation.ipynb](scripts/merge_validation.ipynb)** and confirm
   the Parameters cell points at the right files: `INPUT_DIR`, the two source filenames,
   and the namespace-replacement map. Update these if the filenames in `files/to merge/`
   have changed.

2. **Run the "load + detect roles" cells.** The notebook doesn't just trust the
   configured filenames blindly — it loads whichever JSON files are actually in
   `files/to merge/`, and cross-checks role assignment: whichever file has non-empty
   top-level `$validation.required`/`recommended`/`optional` is the
   cardinality/marginality source, the other is the JSON-Schema-rules source. If the
   configured filenames and the auto-detected roles disagree, the notebook prints a
   warning — read it before continuing.

3. **Run the single-property test merge cell.** This merges exactly one property (the
   first one common to both files) and pretty-prints the result.

   ### ⏸ STOP — human confirmation gate
   **Do not run any cell after the test merge until a human has looked at the printed
   test-merge output and confirmed it's correct** (right description, right
   `owl:cardinality`, right marginality bucket, right JSON-Schema keywords, no
   unexpected duplication of `description` from the schema-rules file). If you are
   Claude running this notebook on the user's behalf, this means: stop, show the test
   merge to the user in the chat, and explicitly ask "does this single-property merge
   look right before I merge the rest?" — do not proceed on your own judgment alone.
   Fix the merge logic in the notebook if anything looks off, rerun the test cell, and
   re-confirm before moving on.

4. **Only after confirmation**, run the full-merge cell. It merges every property found
   in either file (union, not intersection), and prints a list of any property names
   missing from one side — these are flagged for the curator, not silently dropped or
   invented.

5. **Run the namespace-normalization + write-output cells.** These replace the
   configured namespace prefixes (default: `NicktempSample`, `gtest` →
   `bioschemasdrafts`) anywhere they appear — in `@context` keys, `@id`/`@type` CURIEs,
   or nested inside `$validation` — and write the final merged document to
   `output/merged-<name>.json` with `indent=2` pretty formatting.

6. Tell the user where the output file landed and summarize any flagged
   (single-sourced) properties so a curator knows what still needs manual review.

## Notes for future runs

- If a third profile needs merging in, or the roles can't be auto-detected because both
  files have populated marginality lists, don't guess — ask the user which file should
  supply cardinality/marginality vs. JSON-Schema rules, and set the Parameters cell
  explicitly.
- Never delete or overwrite files in `files/to merge/` or `files/back up/` — only ever
  write new output into `output/`.
