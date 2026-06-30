---
name: creating-obsidian-trackers
description: Use when the user wants to track a collection of items in Obsidian with filtered, queryable views — a study plan, reading or watch list, job-application pipeline, task/project board, habit or progress log, spaced-repetition review queue, or any "list of things with status" they want to slice by status, date, or category.
---

# Creating Obsidian Trackers

## Overview

A tracker = **one note per item** (data lives in each note's YAML frontmatter) **+ one `.base` file** that queries those notes into filtered table views.

The Base is a *saved query, not a spreadsheet.* You never store rows inside it — each item is its own note, each frontmatter property is a column. Add a note → it appears; edit a property → the views re-sort.

**REQUIRED BACKGROUND:** Use the `obsidian-bases` skill for the full `.base` YAML syntax. This skill is the recipe for assembling a *tracker* out of that syntax.

## When to use

- A set of items the user works through and wants status/coverage views over.
- Especially when they ask for "what's open / not started / done," "group by category," "what's due," or a review queue.

**Don't use for:** a one-off checklist (a single note with `- [ ]` is enough) or fewer than ~10 mostly-static items.

## Recipe

1. **Decide the schema.** Every tracker has a `status`; add the fields *your* domain needs. Pick a tag — the Base scopes by tag, not folder.
2. **Generate one note per item with a script** — never hand-create dozens. The body is a template the user fills in per item.
3. **Write the `.base`** with the views they need.
4. **(Optional) Embed** it in a plan note via `![[Name.base]]`.

The pattern adapts by schema — pick the shape that fits:

| Tracker kind | `status` values | other properties | useful views |
|--------------|-----------------|------------------|--------------|
| **Pipeline / board** (job apps, tasks) | `To Do` → `In Progress` → `Done` (or `applied`→`screen`→`onsite`→`offer`) | `priority`, `date`, `link` | by status, active, done |
| **Collection** (reading/watch list) | `to-read` → `reading` → `done` | `rating`, `author`, `category` | by category, in progress, finished |
| **Review / recall** (study, flashcards) | `New` → `In Rotation` → `Mastered` | `confidence` 🔴🟡🟢, `next_review` | **due today**, weak spots, not started |

## Conventions that matter (learned the hard way)

- **Sanitize filenames.** No `/ \ : * ? " < > |`. Replace `/` and `:` before writing, or the file lands in a subfolder / fails.
- **Put the `.base` OUTSIDE the notes folder** (a sibling or parent) so it isn't listed as one of the items.
- **First column = `file.name`, not a text property.** Only `file.name` renders as a clickable link that opens the note.
- **Scope by tag** (`file.hasTag("x")`), not folder — keeps the base's location flexible and the query stable if notes move.
- **Give each tracker its own unique tag.** Shared tags bleed into each other's views. The note script and the `.base` filter must use the *same* tag.
- **Guard date formulas** so empty dates don't error: `if(due, date(due) <= today(), false)`.
- **Obsidian rewrites `.base` YAML** when opened (re-quotes strings, appends `sort:`). Harmless — re-read before editing.

## Example — a status tracker

Neutral default (a pipeline/board). **Note template** (`Items/<name>.md`):

```markdown
---
category: ""
status: To Do        # To Do | In Progress | Done
tags:
  - tracked          # ← a tag UNIQUE to this tracker
---

# <name>
```

(Don't add a text property that mirrors the filename — the Base shows it via `file.name`.)

**Generate the notes:**

```bash
# DATA lines are "category|name"
while IFS='|' read -r cat name; do
  [ -z "$name" ] && continue
  printf -- '---\ncategory: "%s"\nstatus: To Do\ntags:\n  - tracked\n---\n\n# %s\n' \
    "$cat" "$name" > "Items/$name.md"
done <<< "$DATA"
```

**The `.base`** (sibling of `Items/`, not inside it):

```yaml
filters:
  and:
    - file.hasTag("tracked")
properties:
  file.name:
    displayName: Item
views:
  - type: table
    name: Active
    filters:
      and:
        - status != "Done"
    order: [file.name, category, status]
  - type: table
    name: By Category
    groupBy:
      property: category
      direction: ASC
    order: [file.name, status]
  - type: table
    name: Done
    filters:
      and:
        - status == "Done"
    order: [file.name, category]
```

### Variation: spaced-repetition / review trackers

When items are reviewed on a recurring schedule (study, flashcards, concepts), add two properties — `confidence` (🔴/🟡/🟢) and `next_review` — and after each review schedule the next on the ladder **1 → 3 → 7 → 16 → 35 days → Mastered**. Add a guarded "Due" view:

```yaml
formulas:
  due: if(next_review, date(next_review) <= today() && status != "Mastered", false)
views:
  - type: table
    name: Due Today
    filters:
      and:
        - formula.due
    order: [file.name, category, confidence, next_review]
    sort:
      - property: next_review
        direction: ASC
```

## Common mistakes

- **Text property as the first column** → rows aren't clickable. Use `file.name`.
- **Storing items as rows in one note/table** → Bases can't query that. One note per item.
- **Base inside the notes folder** → it shows up as an item. Move it up a level.
- **Unsanitized `/` in a filename** → wrong path or write failure.
- **Unguarded `date(empty)` in a formula** → the view errors for un-dated items.
