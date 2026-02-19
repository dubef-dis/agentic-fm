---
name: script-preview
description: Generates a human-readable preview of a proposed FileMaker script before XML generation. Use when the user wants to preview, outline, draft, or review script steps in plain English before committing to fmxmlsnippet output. Triggers on phrases like "preview the script", "show me the steps", "outline the logic", "draft the script", or "before you generate".
---

# Script Preview

Produce a human-readable script outline for review and iteration before generating fmxmlsnippet XML.

## Step 1: Read context

Read `agent/CONTEXT.json`. Extract:

- `task` — what the script should do
- `current_layout` — the starting context
- Any relevant fields, scripts, layouts, and value lists needed

## Step 2: Output the preview

Format the script as a numbered, indented step list — the same style as `xml_parsed/scripts_sanitized/`. Rules:

- One step per line, numbered sequentially from `1`
- Nested blocks (If/End If, Loop/End Loop, etc.) are indented with 4 spaces per level
- Show parameters inline: `Set Variable [ $name ; Value: <expression> ]`
- Use plain-English calculations where exact syntax isn't critical yet — the goal is clarity, not precision
- Lead with the script name as a heading

**Example format:**

```
Script: Process Invoice

1. Set Variable [ $invoiceID ; Value: Get ( ScriptParameter ) ]
2. If [ IsEmpty ( $invoiceID ) ]
3.     Show Custom Dialog [ "No invoice ID provided." ]
4.     Exit Script [ False ]
5. End If
6. Go to Layout [ "Invoice Details" (Invoice) ]
7. Perform Find [ invoiceID = $invoiceID ]
8. If [ Get ( FoundCount ) = 0 ]
9.     Show Custom Dialog [ "Invoice not found." ]
10.    Exit Script [ False ]
11. End If
12. Set Field [ Invoice::Status ; "Processed" ]
13. Commit Records [ No dialog ]
14. Exit Script [ True ]
```

## Step 3: Invite iteration

After the preview, ask:

```
AskQuestion:
{
  "question": "Does this logic look right?",
  "options": [
    { "id": "good", "label": "Looks good — generate the XML" },
    { "id": "changes", "label": "I have changes to make" }
  ]
}
```

- If **changes**: ask the user to describe them, update the preview, and loop back to Step 3.
- If **good**: proceed directly to full fmxmlsnippet generation following the standard script creation workflow in AGENTS.md.

## Notes

- The preview is a planning artifact — calculations don't need to be exact FileMaker syntax yet
- Line numbers in the preview are for the developer's reference during iteration, not final output
- When iterating, show the full updated preview each time (not just the changed lines)
