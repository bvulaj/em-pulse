# Claude Setup Instructions for em-pulse

**You are setting up em-pulse, a daily intelligence system for an Engineering Manager. This system integrates calendar, JIRA, Slack, and email to provide comprehensive daily intelligence.**

## Step 1: Install the Briefing Skill

1. Create the skills directory if it doesn't exist:
```bash
mkdir -p .claude/skills/daily-briefing
```

2. Copy the provided `SKILL.md` file to `.claude/skills/daily-briefing/SKILL.md`

3. Confirm the skill is available by checking if you can recognize "brief me" requests

## Step 2: Set Up Memory System

Create the memory directory structure:
```bash
mkdir -p .claude/projects/[PROJECT_NAME]/memory
```

Create these memory files if they don't exist:

**memory/MEMORY.md** (Memory index):
```markdown
# Memory Index

## Team & Context
- [Team Context](user_team_context.md) — direct reports, stakeholders, organizational structure

## Projects & Strategic Context  
- [Strategic Priorities](project_strategic_priorities.md) — ongoing initiatives and key projects

## References & Tracking
- [Briefing Timestamp](reference_briefing_timestamp.md) — tracks last briefing for timeframe calculation

## Communication & Preferences
- [Communication Patterns](communication_patterns.md) — key channels, leadership contacts, monitoring preferences
```

**memory/reference_briefing_timestamp.md**:
```markdown
---
name: briefing-timestamp
description: Tracks last briefing request timestamp for calculating timeframes
metadata: 
  type: reference
---

# Last Briefing Timestamp

**Last briefing request:** [CURRENT_TIMESTAMP]

This timestamp is used by the daily-briefing skill to calculate timeframe for data gathering.
Update this each time a briefing runs.
```

## Step 3: Gather Configuration Information

Ask the user for these details and create appropriate memory files:

### Required Information:
1. **JIRA Configuration:**
   - **JIRA Cloud ID** - Found in several ways:
     - Check your JIRA URL: `https://yourcompany.atlassian.net` → Cloud ID is often in admin settings
     - Go to JIRA → Settings (gear icon) → System → General Configuration → look for "Cloud ID" or "Site ID"  
     - Or have the user run: `curl -u email:api_token https://yourcompany.atlassian.net/rest/api/3/serverInfo` and look for "cloudId" field
     - Format: `12345678-1234-1234-1234-123456789abc` (UUID format)
   - **Team filter preference:** How to find their team's issues (team UUID, project + assignees, component, etc.)

2. **Slack Channels:**
   - Management/leadership channels they want monitored
   - Team channels
   - Cross-functional/working group channels
   - Key DMs to monitor

3. **Email Monitoring:**
   - Direct manager email
   - Skip-level manager email  
   - Other key leadership contacts

4. **Team Context:**
   - Direct report names and roles
   - Current major team initiatives
   - Organizational structure

### Create memory files with this information:

**memory/user_team_context.md**:
```markdown
---
name: team-context
description: Team structure, direct reports, and organizational context
metadata: 
  type: user
---

# Team Context

## Direct Reports
[List team members with roles, focus areas]

## Organizational Structure
**Manager:** [Name and email]
**Skip Level:** [Name and email]
**Peer Managers:** [Names and context]

## Team Focus
[Current major initiatives and strategic priorities]
```

**memory/communication_patterns.md**:
```markdown
---
name: communication-patterns  
description: Key channels, contacts, and monitoring configuration
metadata:
  type: reference
---

# Communication Patterns

## JIRA Configuration
**Cloud ID:** [JIRA_CLOUD_ID]
**Team Filter:** [JQL_QUERY]

## Slack Channels
**Management:** [channel-list]
**Team:** [channel-list]  
**Working Groups:** [channel-list]

## Email Monitoring
**Key Contacts:** [email-list]
**Manager:** [manager-email]
**Skip Level:** [skip-level-email]

## DM Monitoring
[List of key people to monitor for work-related DMs]
```

## Step 4: Customize the Skill

Update the SKILL.md file with the user's specific:
- JIRA Cloud ID and team filter
- Slack channel lists  
- Email contacts
- Any other organizational specifics

## Step 5: Validate Each Component

**Test each data source individually before running full briefing:**

### JIRA Validation
```bash
# Test JIRA connectivity with their configuration
mcp__atlassian__searchJiraIssuesUsingJql with their Cloud ID and team filter
```
**Expected:** Should return team issues without errors
**If it fails:** See troubleshooting section below

### Slack Validation  
```bash
# Test Slack channel access
mcp__slack__search_messages with one of their configured channels
```
**Expected:** Should return recent messages from the channel
**If it fails:** Check MCP permissions and channel names

### Calendar Validation
```bash
# Test calendar access
gog calendar events --max 3
```
**Expected:** Should show upcoming calendar events
**If it fails:** Run `gog auth login` to re-authenticate

### Email Validation
```bash
# Test email search
gog mail search "from:[manager-email]" --max 3
```
**Expected:** Should find recent emails from manager
**If it fails:** Verify email address format and permissions

## Step 6: Full System Test

1. Update the timestamp in memory/reference_briefing_timestamp.md to current time
2. Run a test briefing: respond to "brief me" 
3. Verify comprehensive briefing with all data sources integrated

## Step 7: Success Confirmation

**Tell the user their setup is complete and summarize what was configured:**

"✅ **em-pulse is successfully configured!** Here's what I set up for you:

**JIRA Integration:**
- Cloud ID: [their-actual-cloud-id]  
- Team Filter: [their-actual-jql-query]
- Monitoring: High priority items, blockers, recent completions

**Slack Monitoring:**
- Management Channels: [list-actual-channels]
- Team Channels: [list-actual-channels]
- Working Groups: [list-actual-channels]
- Total: [N] channels being monitored

**Email Alerts:**
- Key Contacts: [manager-name], [skip-level-name], [other-contacts]
- Patterns: 'action required', leadership communications

**Team Context:**
- Direct Reports: [list-actual-team-members]
- Organizational Structure: Configured with reporting relationships
- Strategic Priorities: Template ready for your initiatives

**Ready to use!** Say 'brief me' anytime for your daily intelligence briefing."

## Step 8: Explain Usage to User

Tell them:
- **"Brief me"** triggers the daily briefing
- **System runs automatically in background** (agent-based execution)
- **Memory system maintains context** between briefings
- **They can ask follow-up questions** like "dig deeper into [topic]"
- **Briefings adapt to their schedule** (morning vs EOD format)

## Troubleshooting

### JIRA Issues
**"Invalid Cloud ID" or "Authentication failed":**
- Verify Cloud ID format: `12345678-1234-1234-1234-123456789abc`
- Check MCP server configuration and authentication
- Confirm user has access to specified JIRA instance

**"No issues found" or "JQL error":**
- Test basic query first: `project = "PROJECTNAME"`
- Verify team field access: try `customfield_10001 = "uuid"`
- Fall back to assignee filter: `assignee in (user1, user2)`
- Check issue permissions in JIRA

### Slack Issues  
**"Channel not found" or "Access denied":**
- Verify exact channel names (case-sensitive)
- Check MCP server has workspace permissions
- Test with public channels first, then private
- Confirm user is member of private channels

**"No messages found":**
- Try broader search terms initially
- Check date ranges (messages may be older than search window)
- Verify channel has recent activity

### Calendar Issues
**"No events found" or "Authentication error":**
- Re-run authentication: `gog auth login`
- Check account permissions for calendar access
- Verify correct Google account is authenticated
- Test with simple command: `gog calendar events --max 1`

### Email Issues
**"No emails found" or "Search failed":**
- Verify email addresses are exact (check spelling)
- Test broader search first: `gog mail search "action required"`
- Check Gmail API permissions and authentication
- Confirm target emails exist in accessible mailbox

### Memory System Issues
**"File not found" or "Permission denied":**
- Check directory creation: `ls -la .claude/projects/`
- Verify file permissions: `chmod 644 memory/*.md`
- Ensure memory directory structure is complete
- Re-run memory file creation commands

### General Debugging
**"Skill not found" or "Command not recognized":**
- Verify skill file location: `.claude/skills/daily-briefing/SKILL.md`
- Check skill file syntax and frontmatter
- Restart Claude session if needed
- Confirm all required tools are installed

## Success Criteria

The setup is complete when:
- [ ] User can say "brief me" and get a comprehensive briefing
- [ ] All data sources (calendar/JIRA/Slack/email) are working
- [ ] Memory system is tracking timestamp and context properly
- [ ] Briefing format matches expected structure with agenda, activity, prep, and strategic intelligence

---

**Remember:** This system is designed to save the user 30-60 minutes daily and provide executive-level strategic intelligence. Focus on comprehensive data integration and strategic synthesis.