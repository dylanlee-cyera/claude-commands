# Claude Code Commands

Personal slash commands for Claude Code to streamline customer engagement workflows.

## Available Commands

### `/cyera.prep`

Generate a comprehensive pre-meeting briefing for a customer by pulling data from Notion and Salesforce.

**Usage:**
```
/cyera.prep <customer-name>
```

**Example:**
```
/cyera.prep Netflix
```

**What it does:**
1. Fetches the customer's DataWatcher page (contacts, value statements, documentation)
2. Queries recent Meeting Notes (last 30 days)
3. Lists open Customer Tasks
4. Searches Salesforce for Positive Business Outcomes (PBOs)
5. Searches Notion for other relevant docs

**Output includes:**
- Key contacts (customer and Cyera team)
- Value statements (why they bought)
- Positive Business Outcomes from Salesforce opportunities
- Recent meetings table with dates and status
- Open tasks with owner, priority, and status
- Key links (Slack channels, Drive folders, Notion pages)
- Related documentation

---

### `/cyera.sync-meeting-tasks`

Syncs action items from Meeting Notes to Customer Tasks in Notion.

**Usage:**
```
/cyera.sync-meeting-tasks          # Last 24 hours
/cyera.sync-meeting-tasks week     # Last 7 days
/cyera.sync-meeting-tasks all      # All unprocessed meetings
/cyera.sync-meeting-tasks <name>   # Specific customer meetings
```

**What it does:**
- Queries recent meeting notes from Notion
- Extracts action items from AI-generated summaries
- Creates tasks in Customer Tasks database with:
  - Proper customer mapping
  - Priority based on keywords (urgent → High)
  - Owner extraction
  - Source Meeting relation link
- Skips duplicates automatically

---

## Requirements

- Notion MCP authenticated (`/mcp` → Notion → login)
- Meeting Notes database with AI summaries
- Customer Tasks database
- DataWatcher customer pages in Notion

## Installation

Copy any `.md` file to your `.claude/commands/` directory:

```bash
# Global installation (available in all projects)
cp *.md ~/.claude/commands/

# Project-specific installation
cp *.md /path/to/your/project/.claude/commands/
```

## Database IDs

These commands use the following Notion databases:

| Database | Collection ID |
|----------|---------------|
| Meeting Notes | `collection://311afcab-7b49-8069-a646-000b374b1306` |
| Customer Tasks | `collection://313afcab-7b49-809f-9e46-000bd3a57795` |

## License

Personal use only.
