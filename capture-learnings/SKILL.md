---
name: capture-learnings
description: Use when the user wants to save takeaways, insights, or learnings from the current conversation into their Obsidian vault at end of session
---

# Capture Learnings

Save key takeaways from the current conversation as an Obsidian note in the user's vault.

## Workflow

1. **Ask the user** what the key learnings or takeaways from this conversation were. Do NOT guess — the user decides what was valuable. Wait for their response before proceeding.

2. **Write the note** to `<YOUR_OBSIDIAN_VAULT_INBOX_PATH>` using the format below. The filename should be descriptive and use kebab-case: `<topic>.md`. Do NOT prefix filenames with "learnings-" — the `learnings` tag handles categorization.

3. **Confirm** by telling the user the file path and a brief summary of what was captured.

## Note Format

```markdown
---
date: <YYYY-MM-DD>
tags:
  - <topic-tag>
---

## Key Takeaways

- <Takeaway 1>
- <Takeaway 2>
- ...

## Details

<Optional expanded notes on any takeaway that benefits from more context. Use wikilinks to connect to existing vault notes where relevant. Only include this section if the user provides enough detail to warrant it.>
```

## Rules

- **Always ask first.** Never auto-generate learnings without user input.
- **One note per invocation.** If the user has learnings across multiple topics, create separate notes.
- **Inbox only.** Per vault conventions, new notes go to `Inbox/` — the user sorts them later.
- **Keep it concise.** Bullet points over paragraphs. The user is writing notes for future-self review, not documentation.
- **Use wikilinks** (`[[Note Name]]`) for any references to concepts that might exist in the vault.
- **Add relevant topic tags** beyond just `learnings` — e.g., `python`, `system-design`, `debugging`.
