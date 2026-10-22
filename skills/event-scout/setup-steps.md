# Setup steps only the user can do

Claude does everything else. When one of these steps is needed, Claude gives the user these exact steps in chat and waits for them.

### 1. Connect Google Calendar to Claude
You do this inside the Claude app; no browser needed.
1. Claude shows a **Connect** button for **Google Calendar** right in the chat. Click it.
   - If you don't see a button, open **Settings** in the Claude app, go to **Connectors**, find **Google Calendar** and click **Connect**.
2. Sign in with the Google account whose calendar should receive events, and click **Allow**.
3. Optional, for Outlook users: connect **Microsoft 365** the same way, so the routine can check your Outlook calendar for duplicates and conflicts.
4. Tell Claude when you're done. It checks that the connector shows up.

Connectors belong to your Claude account, so once you connect them in the app, the weekly cloud routine can use them too.

### 2. Turn on internet access for the routine
The routine runs in a cloud environment, and the default environment can block websites. If it's blocked, the routine can't open any event pages and misses most events. Its summary will start with "NETWORK BLOCKED" if that happens.
1. Open your routine at https://claude.ai/code/routines (Claude gives you the exact link).
2. Open the settings for the environment the routine uses (usually **Default**).
3. Set **Network access** to **Full**. If you only see an allowlist option, choose it and ask Claude for the list of domains to add.
4. Save, and tell Claude, so it can confirm with a test run.

### 3. Connect Google Calendar to Outlook (Outlook users only)
The routine adds events to Google Calendar. To see them in Outlook, subscribe Outlook to that Google calendar. This is a one-time step, and events then appear automatically.
1. Go to **calendar.google.com**, signed in to the same Google account you connected in step 1.
2. Click the **gear icon**, then **Settings**.
3. In the left sidebar under **Settings for my calendars**, click your calendar's name.
4. Scroll down to **Integrate calendar** and copy the **Secret address in iCal format**. It contains `/private-` and ends in `basic.ics`.
   - Do **not** use the "Public address in iCal format" (it contains `/public/`). It fails with "Couldn't import calendar" unless your calendar is public, and you should not make your calendar public.
   - If there's no Secret address section, your Google Workspace admin has turned it off. Ask Claude about other options.
5. Add it to Outlook, in whichever Outlook you use:
   - **Outlook on the web, or the new Outlook for Windows or Mac:** go to **Calendar**, click **Add calendar**, then **Subscribe from web**. Paste the secret address, name it "Event Scout", and click **Import**.
   - **Classic Outlook for Windows:** go to **Calendar**. On the **Home** tab, click **Add Calendar**, then **From Internet**. Paste the secret address, click **OK**, then click **Yes** to add it.
   - Not sure which one you have? Subscribing in Outlook on the web (outlook.office.com) always works, and the calendar then usually shows up in your other Outlook apps and on your phone after they sync.

How it behaves once set up:
- The events show in Outlook as a separate "Event Scout" calendar you can turn on and off.
- Outlook refreshes subscribed calendars every few hours, so new events show up later the same day.
- These events don't block your free/busy time. If you decide to go to one, copy it to your main calendar.
- Keep the secret address private. If it leaks, click **Reset** next to it in Google Calendar settings and subscribe again with the new address.

Why not just invite your Outlook address? If your Google account uses the same email address as Outlook, you are the organizer of every event, and Google doesn't send invitations to the organizer, so nothing arrives. The subscription avoids that.

## Troubleshooting
| What you see | What it means | Fix |
|---|---|---|
| Summary starts with "NETWORK BLOCKED" | The routine's environment can't reach websites | Step 2 above |
| Summary starts with "CALENDAR NOT UPDATED" | The calendar connector isn't attached or isn't working | Reconnect Google Calendar (step 1), then ask Claude to check the routine's connectors |
| "Couldn't import calendar" in Outlook | You used the public address | Use the Secret address (step 3) |
| Events show up at the wrong time | The time zone wasn't set | Ask Claude to check the routine prompt; every event must set your time zone |
| Events still say "(time TBC)" | The organizer hasn't published a time yet | Nothing to do; the routine fills it in once the time is posted |
| Few events found | The search was shallow, or the network was blocked | Ask Claude to review the last run log |
