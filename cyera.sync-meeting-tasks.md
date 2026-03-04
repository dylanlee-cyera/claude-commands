---
description: Sync action items from recent Meeting Notes to Customer Tasks in your Mind Palace.
---

## User Input

```text
$ARGUMENTS
```

## Overview

This command extracts action items from your recent Meeting Notes and creates corresponding tasks in your Customer Tasks database. Run this daily to keep your task list current with meeting commitments.

**What it does:**

1. Queries Meeting Notes from the last 24-48 hours
2. Extracts action items from AI-generated summaries
3. Creates tasks in Customer Tasks with proper fields
4. Links tasks back to source meetings
5. Skips duplicates (checks if task already exists)

## Database IDs

- **Meeting Notes DB**: `collection://311afcab-7b49-8069-a646-000b374b1306`
- **Customer Tasks DB**: `collection://313afcab-7b49-809f-9e46-000bd3a57795`

## Arguments

| Argument | Description |
|----------|-------------|
| (none) | Process meetings from the last 24 hours |
| `week` | Process meetings from the last 7 days |
| `all` | Process all meetings with unprocessed action items |
| `<meeting-name>` | Process a specific meeting by name |

## Execution Flow

### Step 1: Query Recent Meeting Notes

Use `mcp__Notion__notion-query-data-sources` to find recent meetings:

```sql
SELECT * FROM "collection://311afcab-7b49-8069-a646-000b374b1306"
WHERE createdTime >= date('now', '-1 day')
ORDER BY createdTime DESC
```

For `week` argument, use `-7 day`. For `all`, remove the date filter.

### Step 2: Fetch Each Meeting's Content

For each meeting returned, use `mcp__Notion__notion-fetch` to get the full page content including the AI-generated summary with action items.

Look for action items in the `<summary>` section, typically formatted as:
- `### Action Items`
- `- [ ] Action item text [^reference]`

### Step 3: Extract Action Items

Parse each action item and extract:
- **Task**: The action item text (without the checkbox or reference)
- **Customer**: Map from the meeting's Customer property
- **Owner**: Extract if mentioned (e.g., "Dylan to...", "Team to...")
- **Due Date**: Extract if mentioned (e.g., "by Friday", "next week")
- **Source Meeting**: The meeting page URL for the relation

### Step 4: Check for Duplicates

Before creating a task, query Customer Tasks to see if a similar task already exists:

```sql
SELECT * FROM "collection://313afcab-7b49-809f-9e46-000bd3a57795"
WHERE "Task" LIKE '%<key words from action item>%'
AND "Customer" = '<customer name>'
```

Skip if a matching task exists.

### Step 5: Create Tasks

Use `mcp__Notion__notion-create-pages` with:

```json
{
  "parent": {"data_source_id": "313afcab-7b49-809f-9e46-000bd3a57795"},
  "pages": [
    {
      "properties": {
        "Task": "Action item text",
        "Status": "To do",
        "Customer": "Customer Name",
        "Priority": "Medium",
        "Owner": "Person name if extracted",
        "Source Meeting": "[\"https://notion.so/meeting-page-url\"]",
        "Notes": "From meeting: Meeting Title"
      }
    }
  ]
}
```

### Customer Mapping

Map Meeting Notes Customer values to Customer Tasks options:

| Meeting Notes | Customer Tasks |
|---------------|----------------|
| Black & Veatch | Black & Veatch |
| Mountain America CU | Mountain America CU |
| Netflix | Netflix |
| Ross | Ross |
| Western Mutual | Western Mutual |
| Internal | Internal |
| Chep | CHEP |
| 1:1 | Internal |
| Brain | Internal |

### Priority Mapping

Set priority based on keywords in action item:
- **High**: "urgent", "ASAP", "critical", "blocker", "today", "immediately"
- **Medium**: Default
- **Low**: "when possible", "nice to have", "eventually"

## Output Format

After processing, show a summary:

```
## Meeting Notes Sync Complete

**Processed:** X meetings from [date range]
**Tasks Created:** Y new tasks
**Skipped:** Z duplicates

### New Tasks Created:

| Customer | Task | Priority | Source Meeting |
|----------|------|----------|----------------|
| CHEP | Follow up on credentialing | High | CHEP Weekly |
| Netflix | Review taxonomy mapping | Medium | Netflix - Taxonomy |
...

### Skipped (Already Exist):

- "Send QBR slides" (CHEP) - exists as task #123
...
```

---

## Error Handling

**If no meetings found:**
```
No meetings found in the specified time range.

Try:
- `/cyera.sync-meeting-tasks week` - Check last 7 days
- `/cyera.sync-meeting-tasks all` - Check all meetings
```

**If Notion not authenticated:**
```
Notion not authenticated. Run `/mcp`, select Notion, and login.
```

## Examples

```
/cyera.sync-meeting-tasks          # Sync yesterday's meetings
/cyera.sync-meeting-tasks week     # Sync last 7 days
/cyera.sync-meeting-tasks CHEP     # Sync specific customer meetings
```

## Related Commands

- `/cyera.notion` - Search and fetch Notion pages
- `/cyera.my-day` - Daily briefing including tasks
