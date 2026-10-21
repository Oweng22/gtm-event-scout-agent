---
name: event-scout
description: Interview a user about their company, who they want to meet, their city and travel radius, and the kinds of events they like, then find their local networking sources (chambers, BNI, Rotary, young professional groups, industry associations, regional conferences) and set up a weekly cloud routine that finds business and networking events, verifies them on official pages, adds them to the user's calendar (Google Calendar directly, or Outlook through a Google Calendar subscription), and re-checks and corrects its own events every week. Use when someone asks for a weekly event finder, a networking calendar, "find events near me to meet clients," a local event scout, or wants a scheduled agent that adds business events to their calendar. Works for any company and any city.
---

# GTM Event Scout Agent (v2)

Set up a weekly cloud routine that finds networking and business events for a specific company in a specific area, puts verified events on the user's calendar, and keeps those events correct as organizers publish times or change details. Work in five phases, in order. Don't skip the interview: the cloud agent starts with zero context, so the routine is only as good as what you learn here.

v2 includes the fixes from field testing. The "Lessons from testing" section at the end says why each rule exists; read it before you change any rule.

## Phase 0: Check prerequisites

Check each item and tell the user plainly what is missing. The user should never have to open any of these files. Do everything you can yourself. For the steps only the user can do (signing in to connectors on claude.ai, setting the environment's network access, subscribing Outlook to Google Calendar), give them the exact click-by-click steps from `setup-steps.md` (next to this file) in chat, at the moment each step is needed, then wait and confirm it worked before moving on.

1. **Cloud routines.** Load `RemoteTrigger` (`ToolSearch select:RemoteTrigger`) and call `action: "list"`. If routines aren't available, offer the fallback: you run the same search on demand in this session, using `event-scout-config.md` and the template as instructions.
2. **Calendar write path.** Run `session_connectors_status` (load `mcp__ccd_connectors__session_connectors_status` if needed) and decide which path applies:
   - **Google Calendar connector connected:** this is the write path. Call its `list_calendars` and note the calendar id and its time zone. Google calendars often default to a different time zone than the user lives in, so every event must set `timeZone` explicitly.
   - **Microsoft 365 connector connected:** call its `get_granted_scopes`. It usually has only `Calendars.Read` and no create-event tool, so it cannot write events. Use it read-only, to skip events the user already has and to flag conflicts.
   - **User lives in Outlook:** the routine writes to Google Calendar and the user subscribes Outlook to that Google calendar with its **secret** iCal address (`setup-steps.md`, step 3). Do not try to "invite" the user's Outlook address: if the Google account uses the same email address, the user is the organizer and Google sends no invite. The **public** iCal address fails with "Couldn't import calendar" unless the calendar is public. Never ask the user to make their calendar public.
   - **No writable calendar:** fall back to a weekly summary with one-click "Add to calendar" links (template, `[IF SUMMARY ONLY]` block).
   If the user connects an app during the session, re-run `session_connectors_status`; new connectors can take a moment to appear.
3. **Network access for the routine's cloud environment.** The default environment can block outbound internet. Then every WebFetch fails with `getaddrinfo ENOTFOUND` and the routine works only from search snippets, which misses most events. You can't change this through the API. Tell the user to set the environment's network access to Full (`setup-steps.md`, step 2), and confirm it in the first test run's log.

## Phase 1: Interview

Use AskUserQuestion, up to 4 questions per call, with concrete options and a recommended default. Ask only what you can't infer from what the user said, files in the working directory, or a skill or connector that holds company context. Save answers to `event-scout-config.md` in the working directory as you go.

**Company and goal**
1. Company name, what it sells, and a one-line description. If the user has a positioning or brand document, read it and take the description, ideal client profile and buyer personas from it.
2. Goals, ranked: new clients, referral partners, hiring, brand awareness, community presence.

**Who they want to meet**
3. Target attendees: roles, industries, company size and stage. Ask about referral partners too (CPA firms, bankers, PE/VC, agencies, technology partners).
4. Trigger signals: what is happening at a company when it is most likely to buy (for example a new CFO, a software renewal, fundraising, an acquisition). These shape which events to look for.
5. Competitors, and how to treat their events: add them (their customers may be prospects) and flag them as competitor-hosted, list them in the summary only, or skip them.
6. People or events to avoid.

**Where**
7. Home city or metro.
8. Travel tiers: the normal radius (for example 45 minutes); a second tier for strong-fit events (for example the nearest big city within about an hour); and where farther major conferences go (default: listed in the summary only, with drive time).
9. Areas to exclude.

**What kind of events**
10. Types to include (multi-select): chamber luncheons and mixers, after-hours socials, morning coffee networking, ribbon cuttings, expos and trade shows, young professional groups, industry and professional association meetings, founder, startup and tech meetups, AI and data events, conferences, galas and golf outings, nonprofit and community events.
11. Types to exclude.
12. Price cap per person. Ask whether they want events on the calendar even if they might not attend (many users do, so they can decide later).
13. In person only, or virtual too.
14. Days and times they can attend.

**Memberships**
15. Organizations they belong to (chambers, young professional groups, BNI, Rotary or Rotaract, associations). Members-only events from these are allowed. Ask whether to add the groups' regular recurring meetings or skip them.

**Calendar and delivery**
16. The write path from Phase 0, and the calendar account.
17. Teammates: v2 adds events with no attendees, so nobody gets invitations. If teammates want the events, they can subscribe to the same Google calendar.
18. Writing preferences for event text (for example "no em dashes").

**Schedule and limits**
19. Day, local time and time zone for the weekly run.
20. Lookahead (default 6 weeks).
21. Maximum new events per run (default 20; recurring meetings and register-by reminders don't count toward it).
22. Model: recommend `claude-opus-5-5` (thorough; a run takes about 8 to 10 minutes). `claude-sonnet-5-5` is cheaper but stopped early in testing.
23. Test run: explain that a test run adds real events to their calendar (no invitations are sent) and ask whether to do one right after setup.

Read back a short summary of the config and get a yes before Phase 2.

## Phase 2: Find the sources

Research live. Never rely on memory for organization names or URLs. Delegating this research to a subagent works well; give it the geography, targets and the checklist below.

For every city and town in the area:
- **Chambers of commerce:** the main regional chamber and each suburb's chamber. Find each one's event calendar URL.
- **Young professional groups** (often run by the chamber), **Rotary, Rotaract, Kiwanis, Lions** (public events), **BNI** (regional events and visitor days; weekly chapter meetings are not events).
- **SBDC, SCORE, library small business programs.**
- **Industry and professional associations** that match the targets (for example FEI, AFP, IMA, the state CPA society, ACG for PE and M&A, SHRM, the local nonprofit association).
- **Startup, tech and AI groups:** incubators, pitch nights, tech meetups, AI meetups, data and BI user groups, the regional economic development organization.
- **Regional conferences and annual series** within the travel tiers: search "<region> AI week", "<region> business summit", "<nearest big city> CFO event", university conferences, and big venue calendars (convention centers). Record each one's name and site so the routine searches for it by name every week.
- **Eventbrite, Luma and Meetup** searches for the city.
- **Trigger-signal and competitor events:** events aimed at the audiences behind the user's trigger signals (for example PE portfolio CFO events, IPO-readiness events, user groups of the software the user's buyers want to replace).

For each source, open the calendar URL and confirm it shows real upcoming events. Record one of: works, JS-only (WebFetch returns an empty shell), blocked (403 or 405), or stale or broken. For JS-only calendars, look for a server-rendered version. Many chamber platforms (ChamberMaster and GrowthZone, for example) serve readable event details at a `mobile/eventDetail.aspx?EventID=<n>` or print URL; record that pattern. If you can't find a calendar for an organization, leave it out and say so.

Show the user the source list grouped by type, ask them to add or remove any, and save it to `event-scout-config.md`.

## Phase 3: Build and create the routine

1. Read `routine-prompt-template.md` (next to this file) and fill in every `{{PLACEHOLDER}}`. Keep exactly one calendar block. The finished prompt must stand on its own. Save it to `event-scout-routine-prompt.md` in the working directory so the user can see and edit it.
2. Convert the run time to a UTC cron expression. Tell the user the conversion and that the local time shifts by an hour when daylight saving starts or ends.
3. Show the user the configuration: name, schedule (local and UTC), model, connectors, and where the full prompt is saved. Get a yes.
4. Create the routine with `RemoteTrigger action: "create"`:
   - `job_config.ccr.environment_id`: from the environment list the `schedule` skill shows (usually "Default").
   - `session_context`: `model`, `sources: []`, `allowed_tools: ["WebSearch", "WebFetch", "Bash", "Read", "Write"]`. MCP connector tools don't need to be listed. Don't send an effort setting; the routine API drops it.
   - `events[0].data`: a fresh lowercase v4 `uuid`, `session_id: ""`, `type: "user"`, `parent_tool_use_id: null`, `message: {"role": "user", "content": <filled prompt>}`.
   - `mcp_connections`: only the connectors the routine needs (the calendar it writes to, plus Microsoft 365 if used for read-only checks). Each entry needs `connector_uuid`, `name` (letters, digits, `-` and `_` only), `url` and `transport_type: "http"`.
5. **Check the connectors in the create response.** Some accounts attach every connected app by default. That response is also the easiest place to find each connector's real URL. Immediately call `action: "update"` with `mcp_connections` set to just the connectors you chose, and confirm in the update response that nothing else remains. Never leave email, file storage, finance or other apps attached.
6. `RemoteTrigger action: "get"` and confirm the saved prompt matches your file, the model is right, and the routine is enabled.
7. Give the user the link: `https://claude.ai/code/routines/<id>`.

Updating later: `job_config` updates replace the whole job config, so always send the full `ccr` object (environment, session_context and events with a new uuid), not just the changed field. Verify afterwards with `get`.

## Phase 4: Audit, test and review

**Audit before any test run:** enabled; cron and next run time; model; environment; connectors limited to the chosen ones, each with `transport_type: "http"`; prompt identical to the saved file.

**Test run** (only if the user said yes): `action: "run"`. A thorough run takes 5 to 10 minutes. Wait with a background `sleep` instead of polling, then `list_runs` (worker_status `idle` means finished) and `get_run_log`.

**Review the log:**
- WebFetch DNS errors on every site mean the network is blocked: give the user step 2 of `setup-steps.md`.
- Fewer than about 60 tool calls, or create_event calls before the search checklist was finished, means the search was too shallow. Tighten the prompt or switch to Opus.
- Check that the calendar tools loaded and that the calendar search ran.

**Review the calendar directly.** Use the calendar connector's `list_events` with `fullText` set to the marker, over the whole window. Results can be large, so parse the saved file with a script rather than reading it raw. Check:
- duplicates (same date and similar title);
- times correct in the user's time zone, including across daylight saving changes;
- no attendees on any event;
- how many events are still all-day or "(time TBC)";
- locations that came from official pages.

**Fix what you find.** Tighten the prompt and push the full job_config again. For events still marked TBC, research the real times yourself: agendas are often images or PDFs, so download them with curl and open them with Read. Then correct the routine's own events with `update_event` (`notificationLevel: NONE`, explicit `timeZone`). Reruns are safe: the routine skips events that already exist and corrects its own.

**Hand off.** Tell the user when the next scheduled run is, where the config and prompt files are, how to change things (edit `event-scout-config.md` and ask you to update the routine, or edit the routine on claude.ai), and that routines can only be deleted at https://claude.ai/code/routines.

## Guardrails written into every routine
- Verify every event on an official page. Never guess a time or location, and never assume this year's date from last year's.
- Find the real time before ever using "(time TBC)".
- Read existing calendars first and never create duplicates.
- Create events, and update only the routine's own events (marked "[Event Scout]" at the start of the description). Never touch, move, respond to or delete any other event, and never delete any event at all. Cancelled events get "CANCELLED:" in the title.
- No attendees, no invitations, `notificationLevel: NONE`.
- Microsoft 365 is read-only and only the calendar search tool may be used: no email, Teams, chat, SharePoint or people search.
- Never send messages, register, pay, submit forms or create accounts.
- Cap new events per run.
- End each run with a summary: added, recurring, worth a look, register-by reminders, updated, highest-value event, skipped duplicates, run notes.

## Lessons from testing (why the rules exist)
- **Network blocked:** the first real run had WebFetch fail with `getaddrinfo ENOTFOUND` on every site. It found 1 event. After the user set network access to Full, the next run found 20.
- **Shallow searching:** a run that didn't have to finish its search before adding events stopped after about 20 searches and missed a major regional AI conference 6 weeks out. The fix was the checklist file, month-by-month query sweeps, following leads to named conferences, and a known-conferences list. Sonnet still stopped at about 55 web calls; Opus completed the full checklist (about 190 steps).
- **A blocked site is not a network failure:** Eventbrite returns 405 to WebFetch, and the agent treated that as the end of Eventbrite. Search Eventbrite, Luma and Meetup with `site:` queries instead.
- **JS-only chamber calendars:** a chamber's main calendar was unreadable until the agent found its `mobile/eventDetail.aspx?EventID=` pages. That turned up the user's young professional group events.
- **Fake TBCs:** early runs marked events "(time TBC)" when times were published on a detail page, a registration page, or in an agenda image, and one homepage had a "9:00 PM-4:00 PM" typo. Hence the FIND THE REAL TIME steps and the weekly re-check that lets the routine fix its own events.
- **Multi-day events:** only one day of a two-day conference was added. Hence "one event per day".
- **Invented details:** the agent once filled in a street address from memory. Hence "an address that did not come from an official page" counts as an error to fix.
- **Time zones:** the Google calendar defaulted to Pacific time for an Eastern user. Always pass `timeZone`.
- **Outlook:** the Microsoft 365 connector was read-only; inviting the user's own address from a same-address Google account sent nothing; the public iCal address failed. The secret iCal address subscription worked.
- **Connectors:** creating the routine attached all 11 of the user's connectors (including email and finance apps) until they were trimmed.
