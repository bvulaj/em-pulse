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
3. **Test:**
  - Say "brief me" to get your first briefing
  - Verify all data sources are working



## What You Get

**Daily Morning Briefing:**

- Your agenda with meeting prep context
- Recent activity across JIRA/Slack/email since last briefing
- Key meeting preparation with relevant background
- Strategic intelligence synthesis connecting daily activities to broader goals

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



## Customization

During setup, you'll configure:

- **JIRA integration:** Your Cloud ID and team filtering (project, component, or team field)
- **Slack monitoring:** Specific channels for management, team, and working groups  
- **Email contacts:** Key leadership and stakeholder email addresses
- **Team context:** Direct reports, organizational structure, and strategic priorities

The system will guide you through configuring these during the setup process.



---

**Ready to get started?** Give `SETUP.md` to your Claude assistant and say "set up em-pulse for me".