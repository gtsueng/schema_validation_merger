# Validation logic check: Sample

- **Date:** 2026-09-22
- **Cardinality / marginality source:** `Nick-Sample-Profile.json`
- **JSON-Schema rules source:** `Ginger_Sample-Profile.json`
- **Total properties checked (union):** 42

This is a draft reasonableness check to help a curator spot likely problems before or after merging -- it does not change either source file, and a flag here is a prompt for human review, not necessarily an error.

## 1. Properties present in only one source

| Property | Present in | Missing from | Missing info |
|---|---|---|---|
| `description` | `Nick-Sample-Profile.json` | `Ginger_Sample-Profile.json` | JSON-Schema validation rules |
| `identifier` | `Nick-Sample-Profile.json` | `Ginger_Sample-Profile.json` | JSON-Schema validation rules |
| `name` | `Nick-Sample-Profile.json` | `Ginger_Sample-Profile.json` | JSON-Schema validation rules |
| `sameAs` | `Nick-Sample-Profile.json` | `Ginger_Sample-Profile.json` | JSON-Schema validation rules |

## 2. Properties with no real JSON-Schema validation rules

Present in `Ginger_Sample-Profile.json` but carrying no validation keywords besides (possibly) a description.

| Property |
|---|
| `conformsTo` |
| `hasPart` |
| `isPartOf` |
| `isRelatedTo` |

## 3. `owl:cardinality: many` without `oneOf` / `anyOf`

Properties marked as repeatable (`owl:cardinality: many`) whose JSON-Schema rules don't offer a single-value/array choice via `oneOf` or `anyOf` -- worth double-checking these can actually accept multiple values as written.

| Property | owl:cardinality | Top-level JSON-Schema keywords present |
|---|---|---|
| `conformsTo` | many | (none) |
| `hasPart` | many | (none) |
| `isPartOf` | many | (none) |
| `isRelatedTo` | many | (none) |

