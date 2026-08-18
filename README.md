# em-pulse

*Daily Intelligence for Engineering Managers*

## What This Is

An AI-powered daily briefing system that integrates calendar, JIRA, Slack, and email to provide executive-level strategic intelligence. Saves 30-60 minutes daily on information gathering and context switching.

## Quick Start

1. **Install Required Tools:**
  - **gog** (Google CLI): [https://github.com/sashabaranov/gogcli](https://github.com/sashabaranov/gogcli)
  - **Atlassian MCP**: [https://github.com/atlassian/atlassian-mcp-server](https://github.com/atlassian/atlassian-mcp-server)  
  - **Slack MCP**: [https://github.com/redhat-community-ai-tools/slack-mcp](https://github.com/redhat-community-ai-tools/slack-mcp)
2. **Hand Off to Claude:**
  - Give your Claude assistant the `SETUP.md` file
  - Say: *"Set up my daily briefing assistant using this guide"*
  - Follow the customization prompts
3. **Enable Gemini Notes** (Optional but Recommended):
  - Enable Gemini note-taking in your Google Calendar settings
  - Attach notes/agendas to recurring meetings for enhanced context
4. **Test:**
  - Say "brief me" to get your first briefing
  - Verify all data sources are working
  - Check meeting prep quality with and without Gemini notes



## What You Get

**Daily Morning Briefing:**

- Your agenda with enhanced meeting prep context
- Recent activity across JIRA/Slack/email since last briefing
- **Smart meeting preparation** with Gemini notes integration and agenda parsing
- Strategic intelligence synthesis connecting daily activities to broader goals

**Enhanced Meeting Intelligence:**

- **Gemini Notes Integration**: Automatically extracts context from Google Calendar meeting notes and agendas
- **Recency Filtering**: Focuses on current agenda items and most recent session notes, filtering out stale historical context
- **1:1 Optimization**: Prioritizes direct report context, career development items, and follow-up tracking
- **Strategic Context**: Connects meeting topics to broader organizational initiatives and priorities

**Persistent Context:**

- Maintains memory of ongoing initiatives and team dynamics
- Tracks strategic priorities across weeks and months
- Remembers stakeholder relationships and organizational context

**Deep Dive Capability:**

- Ask follow-up questions on any topic: "Dig deeper into the schema unification delays"
- Get detailed analysis with full context
- Maintain conversation thread across multiple briefings



## Privacy & Security

- All data remains in your local environment
- No external data sharing
- Configure least-privilege access to required systems
- Review auto-generated memory files for sensitive content



## Value Delivered

- **Time Savings**: 30-60 minutes daily on manual information gathering
- **Context Retention**: Never lose track of ongoing strategic initiatives  
- **Decision Quality**: Better-informed choices through comprehensive intelligence
- **Proactive Detection**: Early identification of issues and opportunities
- **Meeting Efficiency**: Show up prepared with relevant context for every meeting
- **Enhanced 1:1 Quality**: Automatic context from previous sessions ensures continuity and follow-through
- **Strategic Continuity**: Connect daily activities to long-term goals through intelligent synthesis



## Customization

During setup, you'll configure:

- **JIRA integration:** Your Cloud ID and team filtering (project, component, or team field)
- **Slack monitoring:** Specific channels for management, team, and working groups  
- **Email contacts:** Key leadership and stakeholder email addresses
- **Team context:** Direct reports, organizational structure, and strategic priorities
- **Google Calendar**: Ensure gog CLI has access to calendar events and attachments
- **Gemini Notes** (Optional): Enable automatic note-taking for enhanced meeting context

The system will guide you through configuring these during the setup process.

## Advanced Features

**Gemini Notes Integration**: When enabled, the system automatically:
- Extracts context from previous meeting notes for continuity
- Parses current-day agendas for meeting preparation
- Filters out stale historical context to focus on relevant information
- Prioritizes 1:1 context for direct report management

**Smart Context Filtering**: Uses recency algorithms to surface only the most relevant information, avoiding information overload while maintaining strategic continuity.



---

**Ready to get started?** Give `SETUP.md` to your Claude assistant and say "set up em-pulse for me".