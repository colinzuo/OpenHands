
## README.md - your index (5 minutes to save hours)

This is your entry point when you come back after a break.

Keep this short and current.

## architecture.md - evolving mental model

This is not a design doc, it’s your understanding of the system.

Update this when:

- You realize something was wrong
- You finally “get” a confusing part

### initial

Chat
`Explain this repository at a high level. Focus on architecture and data flow`

`update notes\020-architecture.md based on this, the current content is just sample example, not really real`

## decisions.md — this one is critical

Any non-trivial choice goes here

This prevents:

- Second-guessing
- Re-litigating old debates
- Claude (or humans) undoing past work

## investigations/ — where real thinking happens

Use one file per question / bug / idea.

Name it with date + topic

This pairs extremely well with Claude:
You can paste sections in and say:
`Given this investigation, what would you try next?`

## TODO.md — keep it brutal and actionable

Avoid fluffy tasks.

Bonus:

- Mark blockers explicitly
- Cross-reference investigations

## How to integrate Claude into your notes (important)

### Pattern 1: Notes → Claude

Paste:

- Problem
- Observations
- Hypotheses

Ask:
`Based on these notes, what am I missing?`

### Pattern 2: Claude → Notes

After a good Claude explanation, summarize in your own words and store:

- Key insight
- One diagram or bullet list

Never paste raw AI output verbatim — future-you hates that.
