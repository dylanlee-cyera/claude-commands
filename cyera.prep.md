---
description: Pre-meeting preparation combining all Notion data sources for a customer.
---

## User Input

```text
$ARGUMENTS
```

## Overview

Generate a comprehensive pre-meeting briefing for a customer by pulling data from your Mind Palace and DataWatcher resources.

**What it does:**

1. Fetches the customer's DataWatcher page (contacts, value statements, documentation)
2. Queries recent Meeting Notes (last 30 days)
3. Lists open Customer Tasks
4. Searches Salesforce for Positive Business Outcomes (PBOs)
5. Searches Notion for other relevant docs

## Arguments

| Argument | Description |
|----------|-------------|
| `<customer-name>` | Customer name to prep for |

## Handle Empty Arguments

If `$ARGUMENTS` is empty, show usage:

```
**Customer Prep**

Usage: `/cyera.prep <customer-name>`

This generates a pre-meeting briefing with:
- Customer contacts and value statements
- Salesforce Positive Business Outcomes (PBOs)
- Recent meeting history (last 30 days)
- Open tasks
- Relevant Notion docs
```

## Database IDs

- **Meeting Notes DB**: `collection://311afcab-7b49-8069-a646-000b374b1306`
- **Customer Tasks DB**: `collection://313afcab-7b49-809f-9e46-000bd3a57795`

## Finding Customer Pages

Search Notion for the customer's DataWatcher page using `mcp__Notion__notion-search`:
- `query`: `<customer-name> DataWatcher` or just `<customer-name>`
- `query_type`: "internal"

Look for pages under "DataWatcher Customer Pages" that match the customer name.

## Execution Flow

### Step 1: Find and Fetch DataWatcher Customer Page

1. Search Notion for `<customer-name>` to find their DataWatcher page
2. Use `mcp__Notion__notion-fetch` with the page ID

Extract and display:
- **Main Contacts** - Names and roles
- **Value Statements** - Why they bought DataWatcher
- **Key Links** - Documentation, Slack channels, incident support

### Step 2: Query Recent Meeting Notes

Use `mcp__Notion__notion-query-data-sources`:

```sql
SELECT * FROM "collection://311afcab-7b49-8069-a646-000b374b1306"
WHERE "Customer" = '<customer-name>'
AND createdTime >= date('now', '-30 day')
ORDER BY createdTime DESC
LIMIT 10
```

Note: Customer field values in Meeting Notes may vary (e.g., "CHEP" vs "Chep"). Try variations if needed.

Display meeting titles, dates, and status.

### Step 3: Query Open Tasks

Use `mcp__Notion__notion-query-data-sources`:

```sql
SELECT * FROM "collection://313afcab-7b49-809f-9e46-000bd3a57795"
WHERE "Customer" = '<customer-name>'
AND "Status" != 'Done'
ORDER BY "Priority" DESC
```

Display task name, owner, priority, and status.

### Step 4: Search Salesforce for Positive Business Outcomes

Use `mcp__Notion__notion-search` with:
- `query`: `<customer-name> Positive Business Outcomes`
- `query_type`: "internal"

Look for results from **Salesforce** connected source (type will show as Salesforce Opportunity). Extract PBOs from the search results - they typically describe business value realized from the deployment.

If no Salesforce PBOs found, try searching for:
- `<customer-name> business outcomes`
- `<customer-name> value realized`

Display the PBOs as bullet points showing the customer's achieved outcomes.

### Step 5: Search for Related Docs

Use `mcp__Notion__notion-search` with:
- `query`: `<customer-name>`
- `query_type`: "internal"

Show top 5-10 relevant results (filter out Meeting Notes and Salesforce records already shown).

## Output Format

```
# <Customer Name> - Pre-Meeting Prep

**Generated:** <current date/time>

---

## Key Contacts

| Name | Role |
|------|------|
| <contact name> | <role> |

---

## Value Statements

Why they bought DataWatcher:
- <value statement 1>
- <value statement 2>

---

## Positive Business Outcomes (from Salesforce)

Business value realized:
- <PBO 1 - e.g., accelerated AI innovation without data compromise>
- <PBO 2 - e.g., reduced privacy risk and improved compliance>
- <PBO 3 - e.g., operational efficiency gains>

---

## Recent Meetings (Last 30 Days)

| Date | Meeting | Status |
|------|---------|--------|
| <date> | <meeting title> | <status> |

---

## Open Tasks

| Task | Owner | Priority | Status |
|------|-------|----------|--------|
| <task> | <owner> | <priority> | <status> |

---

## Key Links

- **Slack Channel**: <channel link>
- **Incident Support**: <folder link>
- **DataWatcher Page**: <Notion link>

---

## Related Docs

- [Doc title](url) - <brief description>

---

*Run `/cyera.prep <customer>` to regenerate*
```

## Error Handling

**If customer not found:**
```
Customer "<input>" not found in Notion.

Try:
- Check spelling
- Search manually: `/cyera.notion <customer-name>`
```

**If no recent meetings:**
```
No meetings found in the last 30 days for <customer>.

Previous meetings may exist - try:
`/cyera.notion <customer> meeting`
```

## Related Commands

- `/cyera.notion` - Search and fetch Notion pages
- `/cyera.sync-meeting-tasks` - Sync action items to tasks
