# GTM Event Scout Agent

A Claude Code plugin that finds business and networking events near you every week, checks each one on its official page, and puts them on your calendar. Each week it also re-checks the events it added and fixes times, venues and cancellations as organizers update them.

It works for any company in any city. Claude interviews you about your company, who you want to meet, how far you'll travel and which groups you belong to. It then researches your local chambers, associations, meetups and regional conferences, and sets up a weekly routine that runs in the cloud.

## What you need
- Claude Code, and a Claude account with cloud routines (Claude Code on the web). Without routines, Claude can run the same search for you on demand.
- A Google Calendar you can connect to Claude. Events are added there. If you live in Outlook, you subscribe Outlook to that Google calendar, so the events show up in Outlook too.
- Optional: the Microsoft 365 connector, so the routine can read your Outlook calendar to skip events you already have and flag conflicts. It can't write to Outlook.

## Install

**As a plugin (recommended).** In Claude Code, run:

```
/plugin marketplace add oweng22/gtm-event-scout-agent
/plugin install gtm-event-scout-agent@oweng22-skills
```

**As a plain skill.** Clone this repo (or use **Code > Download ZIP** on GitHub) and copy the `skills/event-scout` folder to `~/.claude/skills/event-scout` (on Windows: `%USERPROFILE%\.claude\skills\event-scout`).

## Set it up

In Claude Code, say **"Set up my event scout."** Claude interviews you, researches your local event sources, writes and creates the weekly routine, runs a test if you want one, and reviews the results.

Claude does everything it can itself. A few steps need you to click in your own accounts, and Claude gives you the exact steps when each one comes up:
1. Connect Google Calendar (and optionally Microsoft 365) to Claude at claude.ai.
2. Turn on internet access for the routine's cloud environment.
3. Outlook users: subscribe Outlook to the Google calendar with its secret iCal address.

The full steps are in [`skills/event-scout/setup-steps.md`](skills/event-scout/setup-steps.md) if you want to read ahead.

It helps to have ready: a short description of your company (or a positioning document), your city and how far you'll drive, the groups you belong to, which calendar you use, and when you want the weekly run.

## What it will and won't do
- It adds only events it has verified on an official page, and looks hard for the real start time before marking anything "(time TBC)".
- It never adds the same event twice. Each week it re-checks the events it added: it fills in times as they're published, fixes changes and errors, and marks cancellations with "CANCELLED:" in the title.
- It only edits events it created (marked "[Event Scout]" at the start of the description). It never touches your other events and never deletes anything.
- It never sends invitations, emails or messages, never registers you for anything, and never pays.
- The routine can use only the calendar connectors you choose, never your email, files or other apps. Microsoft 365, if attached, is read-only and limited to calendar search.
- Each weekly run takes about 8 to 10 minutes on the recommended Opus model.

## Changing or removing it
- To change anything (area, sources, event types, schedule), ask Claude to update your event scout. Your settings live in `event-scout-config.md` and the routine's instructions in `event-scout-routine-prompt.md`, both in the folder where you set it up.
- To pause or delete the routine, go to https://claude.ai/code/routines. Routines can't be deleted from Claude Code.
- To remove the events, delete them in Google Calendar. Search for "Event Scout" to find them.

## Changing or removing it
- To change anything (area, sources, event types, schedule), ask Claude to update your event scout. Your settings live in `event-scout-config.md` and the routine's instructions in `event-scout-routine-prompt.md`, in the folder where you set it up.
- To pause or delete the routine, go to https://claude.ai/code/routines. Routines can't be deleted from Claude Code.
- To remove the events, delete them in Google Calendar. Search for "Event Scout" to find them.
- To remove the plugin: `/plugin uninstall gtm-event-scout-agent@oweng22-skills`.

## Repo layout
```
.claude-plugin/
  plugin.json                  plugin manifest
  marketplace.json             lets this repo be added with /plugin marketplace add
skills/
  event-scout/
    SKILL.md                   instructions Claude follows
    routine-prompt-template.md the weekly routine's prompt, with placeholders
    setup-steps.md             the steps only the user can do, plus troubleshooting
```

## Instructions for Claude (installing)
1. Preferred: run `/plugin marketplace add oweng22/gtm-event-scout-agent`, then `/plugin install gtm-event-scout-agent@oweng22-skills`. The skill is then available as `/gtm-event-scout-agent:event-scout`.
2. Plain-skill install: copy the `skills/event-scout` folder so it lands at `~/.claude/skills/event-scout/`, with `SKILL.md` directly inside it. If the user downloaded a ZIP from GitHub, the files are under a `gtm-event-scout-agent-main/` folder; copy only `skills/event-scout` from it. 3. Read every file before installing (they are plain markdown and JSON) and tell the user it's installed.
4. If the user wants to set it up now, read `SKILL.md` and follow it from Phase 0. Keep the user's files (`event-scout-config.md`, `event-scout-routine-prompt.md`) in their current working directory.
5. Don't modify any other skills or settings.

## License

MIT. See [LICENSE](LICENSE).
