# Claude Code Commands

Personal slash commands for Claude Code.

## Available Commands

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

**Requirements:**
- Notion MCP authenticated (`/mcp` → Notion → login)
- Meeting Notes database with AI summaries
- Customer Tasks database

## Installation

Copy any `.md` file to your `.claude/commands/` directory:

```bash
cp cyera.sync-meeting-tasks.md ~/.claude/commands/
# or
cp cyera.sync-meeting-tasks.md /path/to/your/project/.claude/commands/
```
