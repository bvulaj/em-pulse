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

## Step 5: Test the System

1. Update the timestamp in memory/reference_briefing_timestamp.md to current time
2. Run a test briefing: respond to "brief me" 
3. Verify all data sources are working:
   - Calendar events appear
   - JIRA issues are found
   - Slack channels are monitored  
   - Email scanning works

## Step 6: Explain Usage to User

Tell them:
- **"Brief me"** triggers the daily briefing
- **System runs automatically in background** (agent-based execution)
- **Memory system maintains context** between briefings
- **They can ask follow-up questions** like "dig deeper into [topic]"
- **Briefings adapt to their schedule** (morning vs EOD format)

## Troubleshooting

**Common Issues:**
- JIRA connectivity: Verify Cloud ID and team filter syntax
- Slack access: Ensure MCP has proper permissions  
- Memory errors: Check file permissions and directory structure
- Missing data: Verify tool installations (gog, MCP servers)

## Success Criteria

The setup is complete when:
- [ ] User can say "brief me" and get a comprehensive briefing
- [ ] All data sources (calendar/JIRA/Slack/email) are working
- [ ] Memory system is tracking timestamp and context properly
- [ ] Briefing format matches expected structure with agenda, activity, prep, and strategic intelligence

---

**Remember:** This system is designed to save the user 30-60 minutes daily and provide executive-level strategic intelligence. Focus on comprehensive data integration and strategic synthesis.