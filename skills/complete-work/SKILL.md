---
name: complete-work
description: Complete the three-step wrap-up ritual for a finished issue on the board — move it to Done, clear the assignee, and append a one-line summary of what changed to the description. Use this skill whenever a developer says they're finished with, done with, wrapping up, closing out, or shipping an issue or task — regardless of how they phrase it. Trigger on phrases like "I'm done with #3", "wrap up the board-layout issue", "mark the API issue as complete", "close out that task", "that's done", "ship it to done", or anything that signals completion of work on a tracked issue. Do not wait for the dev to use a specific command name.
---

When a developer signals they've finished work on an issue — in whatever words come out — perform all three wrap-up steps so none get forgotten.

## Step 1 — Identify the issue

Fetch the full list:

```
GET http://localhost:3000/api/issues
```

Match what the developer said to an issue. Two patterns:

- **Positional reference** ("I'm done with #3", "issue 2"): use the 1-based rank in the list returned by GET (sorted by `order`). The UI doesn't display numbers, but devs often count by position. Always confirm: "That's issue #3: *Write API routes* — wrapping that up?"
- **Title fragment** ("the board-layout issue", "that auth task"): find the closest title match. If exactly one issue matches clearly, still echo the title for confirmation before patching. If two or more match, list them and ask which one.

If the developer already included enough context that confirmation feels redundant (e.g., they named the exact title), a single sentence like "Wrapping up *Design board layout* — one moment" is enough before you proceed.

**If the server isn't running** (connection refused, network error): stop and tell the dev the app needs to be running — `npm run dev` — before the wrap-up can happen.

## Step 2 — Get a one-line summary

The summary records *what changed in the code*, not just that the issue is closed. Append it to the issue description as a permanent record. Collect it in this order:

1. **Inline in the dev's message** — if they said "I'm done with #3 — extracted the layout into its own component", use that phrase as the summary.
2. **Recent git commits** — run `git log --oneline -5`. If a commit clearly relates to this issue, synthesize a one-liner from it (e.g., "Refactored Board to use CSS grid and extracted Column component").
3. **Session context** — if you've been actively working on this issue in the conversation and know what changed, write a concise sentence from that knowledge.
4. **Minimal fallback** — if none of the above gives you signal, use `"Completed: <issue title>"` (e.g., `"Completed: Write API routes"`). This keeps the record useful without blocking the wrap-up. You can note to the dev: "No commit found — used the issue title as the summary; update it if you want more detail."

Format: past tense, one sentence, starts with a capital letter, no trailing period.

## Step 3 — Apply all three changes in a single PATCH

```
PATCH http://localhost:3000/api/issues/<id>
Content-Type: application/json

{
  "status": "done",
  "assignee": null,
  "description": "<original description>\n\nDone: <summary>"
}
```

If the original `description` is an empty string, set `description` to just `"Done: <summary>"` (no leading newlines).

## Step 4 — Confirm

After a successful response, report briefly — one or two lines:

> Wrapped up *Design board layout* (#2). Moved to Done, assignee cleared, summary appended: "Moved column layout from flexbox to CSS grid and extracted Column into its own component."

If the PATCH fails, show the error and don't claim success.
