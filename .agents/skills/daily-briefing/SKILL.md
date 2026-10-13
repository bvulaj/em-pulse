---
name: daily-briefing
description: Comprehensive daily briefing with calendar, JIRA activity, Slack updates, and email alerts. Use when user asks for "daily briefing", "brief me", "brief me on my day", or similar morning/EOD briefing requests.
---

# Daily Briefing

Comprehensive daily briefing with calendar, JIRA activity, Slack updates, and email alerts.

## When to use this skill

When user requests:
- "brief me"
- "daily briefing"
- "brief me on my day"
- "give me my briefing"
- "morning briefing"
- "brief me eod" (end of day)

## Implementation

1. **Track timing:** Update "last briefing request" timestamp in memory
2. **Calculate timeframe:** Use stored timestamp from previous briefing to current time
3. **Gather data:**
   - Calendar: `gog calendar events` from last briefing to now
   - Email: `gog mail search` from last briefing to now (key people + "action required")
   - JIRA: Search team issues from last briefing timeframe
   - Slack: Monitor specified channels from last briefing timeframe

4. **Format output** based on briefing type (morning vs EOD)

## Data Sources

**Calendar:** `gog calendar events` with date ranges

**Email:** `gog mail search` - Focus on key leadership and "action required" messages
- Target contacts: [YOUR_MANAGER_EMAIL], [SKIP_LEVEL_EMAIL], [OTHER_KEY_LEADERS]
- Search patterns: "action required", leadership names

**JIRA:** Use Atlassian MCP with cloudId `[YOUR_JIRA_CLOUD_ID]` (format: 12345678-1234-1234-1234-123456789abc) and team filter:
- High priority items
- New blockers
- Critical/Major severity
- Recently completed items
- Items assigned to direct reports

**Configure your team filter - examples:**
```jql
# Team field (recommended) - replace with your team's UUID
customfield_10001 = "[YOUR_TEAM_UUID]"

# By project and assignees - replace with your project and team members
project = "[YOUR_PROJECT]" AND assignee in ([USER1], [USER2], [USER3])

# By component - replace with your team's component
component = "[YOUR_TEAM_COMPONENT]"
```

**Slack Channels - customize for your organization:**

**Management Channels:**
- [your-eng-managers-channel]
- [your-leadership-channel]
- [your-skip-level-channel]

**Working Group Channels:**
- [cross-functional-project-channels]
- [technical-working-groups]
- [strategic-initiative-channels]

**Team Channels:**
- [your-team-channel]
- [your-team-technical-discussions]

**DM Monitoring:**
- [YOUR_MANAGER_NAME]
- [SKIP_LEVEL_NAME]
- [KEY_STAKEHOLDER_NAMES]
- Direct reports (work-related only)

**Configuration Notes:**
- Replace all [PLACEHOLDER] values with your specific details
- Add/remove channels based on your organizational structure
- Ensure you have access permissions for all monitored channels

### Slack sweep method — read this, the naive version silently loses threads

`get_channel_history` with `oldest` set to the last-briefing timestamp returns **only messages whose own parent falls inside the window.** Replies posted today onto a thread started last week do not come back. Long-running decision threads are exactly where teams make decisions, so the naive sweep misses the most important traffic in a channel while looking complete.

**`include_threads: true` is necessary but not sufficient — always pass it.** One measured example, a single busy channel over a one-day window, with identical `oldest` and `limit` and the flag as the only variable:

| call | messages returned |
|---|---|
| without the flag | **4** (parents only) |
| with `include_threads: true` | **18** (4 parents + 14 replies) |
| with the flag **and** a 21-day `oldest` | **21** — adds the replies whose parents predate the window |

Treat the ratios as illustrative rather than typical: they depend entirely on how threaded a given channel is, and a channel of mostly standalone posts will show little difference. The *ordering* is what generalises — the flag recovers replies whose parent is inside the window, the widened window recovers the rest, and you need both. Passing the flag alone still loses the oldest, longest-running threads, which are the ones carrying standing decisions.

**Which channels get the full method.** Not all of them — the full sweep costs roughly 10× a window read, so spend it where decisions land. A channel qualifies if **either** test passes:

- **Threaded decisions.** Discussion happens in long-running threads that stay alive for weeks — resourcing, prioritisation, cross-team design. These are the channels where the naive sweep is actively misleading rather than merely incomplete.
- **You are accountable for what happens there.** Your org's scope, or a forum where your silence reads as a position.

Everything else — announcement feeds, alert channels, low-traffic management channels, self-DMs — gets the plain window read. If a channel is mostly bot posts, standups, or single-message announcements, the full method buys nothing. Re-judge when channels are added; do not grow the list by default.

**The method:**

1. Call `get_channel_history` with `oldest` set to **21 days back**, `include_threads: true`.
2. Filter client-side on each message's **own** timestamp `>= last briefing`. Replies come back tagged `[thread:<parent_ts>]` and carry their own ts, so this is a one-line filter.
3. Large results exceed the tool-output cap and are written to a file instead — parse that file rather than re-querying. **Do not shrink the window to avoid the cap; shrinking the window is the bug.**

```python
import json, datetime
CUTOFF = <last-briefing epoch>
msgs = json.load(open(f))['result']
for m in msgs:
    ts = float(m[1:m.index(']')])
    if ts < CUTOFF: continue
    print(datetime.datetime.utcfromtimestamp(ts), m[m.index(']')+1:])
```

Fast path: `get_thread` directly on any thread_ts noted as live in the previous briefing. Carry those forward in `memory/reference_briefing_timestamp.md` under **Live threads**.

#### `get_thread` takes the PARENT ts, never a reply's own ts

Every message returned by search or history carries its own ts first, and a reply also carries `[thread:<parent_ts>]` at the end. **Calling `get_thread` with a reply's own ts returns exactly that one message** — visually identical to a thread where nobody responded. There is no error and no empty result to warn you.

Always read the `[thread:...]` suffix and pass **that** value. A message with no `[thread:...]` suffix is a parent, and its own ts is correct.

```python
# ts = "1790958838.515049", tagged [thread:1790862457.806859]
get_thread(channel_id=C, thread_ts="1790862457.806859")   # correct — returns the full thread
get_thread(channel_id=C, thread_ts="1790958838.515049")   # WRONG — returns 1 message, looks unanswered
```

**Never report a message as unanswered, ignored, or awaiting a reply without a parent-ts `get_thread` showing what came after it.** This failure mode has been observed repeatedly in practice — first by reading flat channel history, then again by calling `get_thread` on the reply ts after the first lesson. In every observed case the message had in fact been answered, sometimes within two minutes. "Nobody replied to X" is a claim about a person's responsiveness and it is read as one, so the verification bar is a thread fetch, not an inference from silence.

Corollary: building a Slack permalink for a reply already requires the parent ts in `?thread_ts=`. If you have the parent ts for the link, you have it for the fetch.

## Morning Briefing Format

```
# [Day], [Date] Daily Brief 🕒

## Your Agenda Today
[Calendar events with clock emoji for meeting time (🕐 🕜 🕑 🕝 etc.) rounded to nearest half-hour, in local time]
**CRITICAL:** Always include full agenda section. Never show "No meetings" without careful verification.
**EXCLUDE:** Do not mention or track "Heads Down" recurring time blocks.
**SHOW:** Distinguish between completed meetings and remaining meetings based on current time.

## Recent Activity (Since Last Briefing)
**Team JIRA Highlights:** [High priority, blockers, Critical/Major items, completions with timestamps]
**Slack Signals:** [Key updates from monitored channels - focus on work-related content only]
**Email Alerts:** [Important messages requiring action from key stakeholders]

## Action Items for Today
**🎯 Ongoing Priorities:** [Persistent items until marked complete - confirm before adding/removing]
**📋 Daily Focus:** [Generated based on calendar/context and strategic priorities]

## Key Meeting Prep

**[Meeting Name] ([Time]):**
- **Prior Context:** [Key points from most recent Gemini notes, if available]
- **Today's Agenda:** [Current planned topics from today's agenda attachment]
- **Recent Activity:** [Relevant JIRA updates, Slack discussions related to meeting topics]
- **Prep Recommendations:** [Specific action items, decisions needed, or topics to surface]
- **Strategic Context:** [How this meeting connects to broader initiatives]

*[Repeat for each significant meeting, prioritizing 1:1s and strategic sessions]*

## Strategic Intelligence
[Executive-level insights connecting today's activities to broader goals and positioning]

[Closing question about priorities or areas needing deeper analysis]
```

## EOD Briefing Format

```
# [Day] EOD Summary 📊

## Today's Accomplishments
[Summary of business day activity from current business day]

## Tomorrow's Priorities
[Preview of next day's calendar + key prep items]

## Action Items Status
[Updates on ongoing items, newly completed items]

## Overnight/Tomorrow Morning Prep
[Any urgent items to address]
```

## Recurring Obligations Check

Before writing **🎯 Ongoing Priorities**, check `memory/user_weekly_obligations.md` for anything due today — recurring commitments the EM owes upward or outward on a fixed cadence (weekly status to a management chain, a biweekly review, a monthly report).

These are **pre-approved standing obligations, not candidate items** — surface them as due, without asking permission. The distinction matters: action items need approval because they represent new scope, whereas a recurring obligation was already agreed and the briefing is only reminding.

Where an obligation has preparatory work attached, run the work rather than just the reminder. For a periodic accomplishment review, that means sweeping Jira for items resolved since the document's last-updated date, Slack for the EM's own messages since that date, and the calendar for meetings they ran or organized — then drafting entries rather than presenting a blank page.

## Brag Capture

A brag document, in [Julia Evans'](https://jvns.ca/blog/brag-documents/) sense, is a running record of your work kept by you, because nobody else is keeping it and you will not remember six months later. The briefing is a natural capture point: it already reads the same sources where evidence appears, every day, while the detail is fresh.

Watch each briefing for brag-worthy moments and surface them in the Suggestions block as a **separate group from action-item candidates**, clearly labeled. Same approval gate as action items: **never write to the brag document without explicit permission.**

**Bar for capture:** a decision the EM made, a position they took, a person outcome they caused, or something a third party credited them with. Not meetings attended, not team throughput, not routine management activity. The strongest candidate is something someone else noticed without the EM reporting it.

**Every entry needs a WHAT and a HOW.** The what is the outcome; the how is the mechanism — the specific thing this person did that caused it. A what without a how is indistinguishable from team output that would have happened anyway, and that ambiguity is the single most common reason good work fails to count at review time. A good how names the judgment call: what was decided, what was rejected, what was sequenced differently.

**Where candidates go:** append to a holding file (e.g. `brag-candidates.md`) under Pending, with the date observed and the source, whether or not the EM responds. This deliberately differs from the action-item rule of "surface once, then drop" — an unpicked action item is noise, but an unpicked brag is lost evidence, which is the failure the document exists to prevent. To stop the queue accumulating, **candidates are surfaced once and not re-raised in later briefings.** They are drained at a periodic review.

Approved → write into the brag document, remove from Pending. Rejected → move to Declined with a reason so it is not re-proposed.

**Report an empty result rather than padding.** Most briefing windows contain no brag candidate, and that is the normal case — the bar is deliberately high. When nothing clears it, say so in one line ("no candidate this window") and move on. Do not reach for team throughput, a clean deploy, an incident someone else handled, or a meeting the EM merely attended in order to have something to show. The common failure is narrating participation as initiative. A padded entry is worse than none: it dilutes the real ones and it will not survive a follow-up question.

## Action Item Management
- **Persistence:** Maintain ongoing list in memory
- **Addition:** ALWAYS ask permission before adding new items - NEVER add unilaterally
- **Removal:** ALWAYS ask permission before removing items
- **Completion:** Mark done when user confirms, notify user
- **Carryover:** Continue until explicitly told otherwise or user confirms completion
- **Scope:** Only track items that are the EM's direct responsibility
- **Suggestions:** Surface potential action items as suggestions at the end of the briefing, grouped separately, awaiting approval

## Meeting Prep Logic

**Enhanced Context Gathering:**
- Search JIRA for relevant tickets involving meeting context
- Check recent Slack conversations related to meeting topics
- **Gemini Notes Integration:** Extract context from calendar meeting attachments
- **Include context for 1:1s** — see the person-centric sweep below
- Flag any blockers or urgent items for discussion

**Person-centric sweep — required for direct reports and up the chain**

**Run it for:** every **direct report**, and every **super-report** — the EM's own manager, their skip-level, and anyone above. These are the relationships where being caught unaware is most expensive, in both directions.

**Do not run it by default for:** peer managers, architects, PMs, and cross-functional partners. Sweep one of those only when something specific is live between them and the EM — an open thread, a decision pending, a commitment outstanding. Otherwise the topic-scoped gathering above is enough.

The reason for the limit is cost: a full sweep is 4–6 extra calls per person. On a day with five 1:1s, doing all of them crowds out the rest of the briefing.

The gathering bullets above are *topic*-scoped. A 1:1 has no topic, so they do no work and the meeting would otherwise be prepped from notes alone. For a qualifying 1:1, sweep the **person** across every source before reading any meeting notes:

- **JIRA** — what they moved, closed, or let sit since the last 1:1. Assignee-scoped, not topic-scoped. Include tickets they created and did not self-assign; that gap is itself signal.
- **Slack** — `from:@<handle>` over the interval, plus thread replies (the window sweep will not return those; see **Slack sweep method** above). Read for posture as well as content — whether they owned a problem or routed it is often the thing worth discussing.
- **Email** — anything to or from them, and anything about them from elsewhere in the org.
- **Calendar** — meetings they ran or organized, and PTO or absence that explains a quiet interval before it gets misread as disengagement.
- **Carried commitments** — what they said they would do at the previous 1:1, and whether it happened. This is the one the other sources will not surface on their own.

Then layer the meeting notes on top. Notes tell you what was *said* last time; the sweep tells you what was *done* since, and the gap between those two is usually the conversation worth having.

**Gemini Notes & Agenda Processing:**

For each meeting on today's calendar:

1. **Extract attachments from calendar events:**
   ```bash
   gog calendar events --json --today
   ```

2. **Identify relevant attachments:**
   - Look for attachments with titles containing "Notes by Gemini", "Notes -", or "Agenda"
   - Extract `fileId` from attachment metadata

3. **Read meeting context:**
   ```bash
   gog docs read <fileId>
   ```

4. **Fallback when the attachment route comes up empty** — do not conclude the notes do not exist. They are frequently in Drive without being attached to the event. Search by meeting title: `gog drive search "<Meeting Title> Notes by Gemini"`. Notes typically land 5–10 minutes after the meeting ends, so a recently finished meeting may need a second look later.

5. **Apply recency filter:**
   - **Current day agendas**: Use full content for today's planned topics
   - **Historical notes**: Only surface context from the single most recent prior session
   - **Ignore**: Older stale historical notes covering topics already resolved

**Handling AI-generated meeting notes — attribution and fidelity**

Automated notes are a model's transcription of a conversation, not a record of one. Treat them accordingly.

- **Attribute, never assert.** Personnel, org, and compensation content sourced from AI notes is reported as *"per the meeting notes,"* never stated as fact.
- **An inferred "Next steps" section is not a transcript.** Items there were not necessarily agreed, assigned, or even said aloud. Never log one as an action item without confirmation — this has produced false action items in practice.
- **Expect mangled names and domain terms.** Product names, acronyms, and non-Anglophone names are routinely garbled. A term that looks unfamiliar is more often a transcription error than a new thing; check before treating it as a new initiative.
- **Summary and full-notes sections differ.** The full notes carry detail and direct quotes that the summary drops. Read both before concluding something was not discussed.

**Context Prioritization:**
- **1:1 meetings**: Always check for Gemini notes - critical for direct report context
- **Team meetings**: Look for notes containing action items, decisions, blockers
- **Strategic meetings**: Focus on outcomes, next steps, and dependencies
- **Cross-functional meetings**: Extract stakeholder perspectives and alignment issues

**Meeting-Specific Prep Recommendations:**
- **New meetings (no prior notes)**: Provide general context from recent JIRA/Slack activity
- **Recurring meetings with notes**: Highlight follow-ups from previous session + today's agenda
- **1:1s with direct reports**: Surface their recent work, blockers, and career development items
- **Leadership meetings**: Connect to strategic priorities and organizational context

## Clock Emoji Mapping (Local Time)
Round to nearest half-hour:
- :00-:14 → hour emoji (🕐 🕑 🕒...)
- :15-:44 → half-hour emoji (🕜 🕝 🕞...)
- :45-:59 → next hour emoji

## Steps

1. **Get the current time:** run `date`. Do this first and do not skip it. Every "completed / in progress / next up" label in the agenda depends on it, and an inferred clock silently inverts the whole section — in one observed failure the briefing placed the time 30 minutes later than it was, marked an upcoming 1:1 as already finished, and therefore produced no prep for the one meeting that most needed it. Re-check the clock if data gathering ran long.
2. **Update timestamp:** Edit memory/reference_briefing_timestamp.md with current time
3. **Get calendar:** Run `gog calendar events` and parse carefully for completed/remaining meetings, scoring each against the clock from step 1
4. **Check email:** Run `gog mail search` for key people and "action required"
5. **Query JIRA:** Use Atlassian MCP to search team activity with detailed analysis
6. **Check Slack:** Sweep monitored channels per the **Slack sweep method** above — 21-day fetch plus `include_threads: true` and a client-side filter for the decision-bearing channels, plain window sweep for the rest. Never narrow the fetch window to dodge the output cap.
7. **Synthesize strategically:** Combine data with executive-level analysis and insights
8. **Format output:** Use appropriate briefing format with strategic intelligence
9. **Update action items:** Review and update ongoing priorities with confirmation

## Covering Meetings That Already Happened

A meeting that **completed inside the briefing window** is not simply dropped from the agenda. Pull its notes and report what came out of it — decisions, commitments, follow-ups. On a multi-day window this is often where the most consequential content is, and a window-based Slack or JIRA sweep will never surface it: decisions made in a meeting leave no trace in either source until someone writes them down, which may be never.
