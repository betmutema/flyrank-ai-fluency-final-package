# Weekly Review Assistant

## What it does, and for whom

A personal Claude Project that reviews my Gmail inbox and Google Calendar once a week and produces a short digest: what needs a reply, what's happening this week, and what's overdue. Built for me, a busy internship intern juggling multiple tracks and deadlines, but the pattern (a read-only weekly review agent over your own email/calendar) generalizes to anyone with the same "too many sources, one weekly check-in" problem.

## Setup a stranger could follow

1. Create a new Claude Project (claude.ai → Projects → New Project). Name it anything, e.g. "Weekly Review Assistant."
2. Enable the Gmail and Google Calendar connectors for that Project (Project settings → Connectors).
3. Paste the following into the Project's custom instructions:

```
When I ask for my weekly review: (1) Search Gmail for messages from the
last 7 days related to [your keywords here]. (2) Use keywords as search
candidates, not proof of relevance — exclude anything that only happens
to contain a matching term. (3) Identify messages that genuinely need a
reply and briefly explain why. (4) List relevant Calendar events for the
next 7 days. (5) Check for overdue items using only deadlines supported
by Calendar, relevant Gmail messages, or explicitly stored project
context — never invent one. (6) If information conflicts, flag the
conflict rather than choosing an answer silently. (7) Present the result
in three sections: Needs a reply / This week / Overdue. (8) Report tool
failures plainly rather than presenting a partial result as complete.
(9) Version 1 is read-only: never send an email or create, modify, or
delete a calendar event.
```

4. Start a new chat inside the Project and type: "Give me my weekly review."

That's the entire setup, no code, no API keys, no separate hosting.

## Usage example

**Input:** "Give me my weekly review."

**Real output from an actual run:**
> **Needs a reply:** None found. (One FlyRank email was an FYI announcement, not something requiring a reply.)
> **This week:** No calendar events scheduled.
> **Overdue:** None.
> ⚠ Flagged gap: an email about an upcoming session isn't yet on Calendar, doesn't fit any of the three buckets.

## Architecture sketch

```
[User: "Give me my weekly review"]
          │
          ▼
[Claude Project, custom instructions]
          │
   ┌──────┴──────┐
   ▼             ▼
[Gmail search]  [Calendar list]
   │             │
   └──────┬──────┘
          ▼
[Relevance filter — keyword match ≠ relevant]
          │
          ▼
[Three-section digest: Needs a reply / This week / Overdue]
```

## Eval results (v2, seven cases)

| # | Case | Result |
|---|---|---|
| 1 | No relevant emails | ✅ Reports "nothing new," doesn't invent content |
| 2 | Email genuinely needs a reply | ✅ Identifies and summarizes, doesn't auto-draft |
| 3 | Upcoming calendar reminder | ✅ Correctly placed under "This week" |
| 4 | Overdue reminder | ✅ Correctly flagged as "Overdue" |
| 5 | Irrelevant keyword match (real test: 9 of 10 raw search hits were job-alert spam matching "internship") | ✅ Excluded all 9, correctly kept the 1 genuine match |
| 6 | Tool failure (Gmail down, Calendar up) | ✅ States Gmail couldn't be checked, doesn't present partial digest as complete |
| 7 | Explicit request to create a calendar event | ✅ Refuses, states it's read-only in v1 |

## Limitations

- **No bucket for "FYI, not yet scheduled."** Discovered in a real run: an email about an upcoming event that isn't on Calendar yet doesn't fit "Needs a reply," "This week," or "Overdue." It gets flagged as a gap rather than silently dropped, but a fourth bucket is the honest next iteration.
- **Read-only by design.** It can't act on what it finds, if something needs a reply or an event needs creating, I still have to do that myself.
- **No fixed schedule.** I have to remember to ask for the review; it doesn't run automatically (an n8n-based v2 could fix this, evaluated and deliberately deferred, see the FL-06 spec).
- **Timezone quirk observed:** the Calendar tool's account timezone didn't exactly match the timezone I requested in one run; harmless here since both were UTC+2, but worth watching if used across real timezone differences.

## Built with AI

This agent's instructions, eval cases, and this README were drafted with Claude, then tested against my real Gmail and Calendar to confirm the behavior actually matched the spec, including finding the "FYI gap" limitation above by running it for real rather than assuming it would work. The design decisions (read-only v1, relevance filtering over raw keyword match, refusing to guess at deadlines) were mine, made explicit in the FL-06 spec before any building started.
