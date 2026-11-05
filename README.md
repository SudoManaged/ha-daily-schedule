# Daily schedule for Home Assistant

[![Validate](https://github.com/SudoManaged/ha-daily-schedule/actions/workflows/validate.yml/badge.svg)](https://github.com/SudoManaged/ha-daily-schedule/actions/workflows/validate.yml)
![Home Assistant 2025.4+](https://img.shields.io/badge/Home%20Assistant-2025.4%2B-41BDF5)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Today's schedule from all your calendars in one list: on your phone every morning, or inside your own briefing.

```
Today's schedule
• All day: Office closed (Work)
• 7:00 AM Run
• 9:00 AM Board call (Work, Home), Teams
• 11:30 AM Lunch with Sam (Work), Harbor Café, Pier 4
• 6:00 PM Dinner (Home), Google Meet
```

A free, open-source Home Assistant blueprint by SudoManaged. It runs on your own Home Assistant and uses only its
built-in calendar integrations: no add-ons, no custom integrations and no extra accounts. It works with any
calendar Home Assistant can read, including Outlook (work or personal), Google Calendar and iCloud. It started as
part of the morning briefing we use every day on our own Home Assistant, and was then generalized and tested for
release.

## Contents

- [What it does](#what-it-does)
- [Requirements and tested versions](#requirements-and-tested-versions)
- [How it works](#how-it-works)
- [Installation](#installation)
- [1. Add your calendars to Home Assistant](#1-add-your-calendars-to-home-assistant)
- [2. Create the schedule script](#2-create-the-schedule-script)
- [3. Get it every morning](#3-get-it-every-morning)
- [4. Test it](#4-test-it)
- [What it looks like](#what-it-looks-like)
- [Using it in your own briefing or automation](#using-it-in-your-own-briefing-or-automation)
- [A calendar card for your dashboard](#a-calendar-card-for-your-dashboard)
- [Keeping calendar details out of the history](#keeping-calendar-details-out-of-the-history)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Updating and removing](#updating-and-removing)
- [Security and privacy](#security-and-privacy)
- [Reporting problems and contributing](#reporting-problems-and-contributing)
- [More free Home Assistant tools from SudoManaged](#more-free-home-assistant-tools-from-sudomanaged)
- [Changelog and license](#changelog-and-license)

## What it does

One script blueprint, [Daily schedule (several calendars)](blueprints/script/sudomanaged/daily_schedule.yaml). A
script made from it:

- **Reads several calendars and merges them** into one list for today (or tomorrow), sorted by time.
- **Labels each appointment with its calendar**, using short labels you choose (`Work`, `Home`, ...).
- **Shows an appointment that is on two calendars once**, with both labels: `9:00 AM Board call (Work, Home)`.
  This is common when a meeting from one account is copied to another.
- **Puts all-day events first** (`All day: Office closed`). An event that runs through the whole day, such as a
  three-day conference, also counts as all day.
- **Shows events still running from the day before** with their end time: `Until 1:30 AM Overnight deploy`.
- **Adds each appointment's location**, cut to a length you choose, and shows **"Teams", "Zoom", "Google Meet" or
  "Webex"** instead of a long meeting link.
- **Hides meetings that Outlook marks as cancelled** ("Canceled: ..." in the title), which some calendar links
  still carry.
- **Refreshes the calendars first.** Home Assistant's Remote Calendar integration (used for Outlook and other
  calendar links) reads its link only once a day on its own, so a meeting added this morning would otherwise be
  missing.
- **Says when a calendar cannot be read** (`Work: calendar unavailable`) instead of quietly leaving it out, so a
  broken calendar is never mistaken for a free day. Each calendar is read on its own, so one failing calendar
  does not hide the others.
- **Sends the list to your phone** (or another notify action) if you choose one, and **returns it** (also when the
  sending fails), so your own automations can use it: in a longer morning briefing, on a speaker, in a text
  helper.

## Requirements and tested versions

- **Home Assistant 2025.4 or newer.** Reading each calendar on its own needs a script feature of 2025.4 (older
  releases would report every calendar as unavailable). 2025.4 is also the release that added Remote Calendar,
  which Outlook and other calendar links need. The blueprint declares this, and Home Assistant refuses to import
  it from the UI on an older release; a hand-copied blueprint file is not checked.
- **Your calendars in Home Assistant**, as calendar entities (`calendar.something`). See
  [1. Add your calendars to Home Assistant](#1-add-your-calendars-to-home-assistant).
- **Optional:** a notify action to send the list to, for example `notify.mobile_app_my_phone` from the Home
  Assistant Companion app.
- The blueprint passes Home Assistant's configuration check on **2025.4.0**, **2026.3.3** and **2026.11.0**, and
  was run live on all three with Local Calendar calendars full of test events:
  - **Merging and labels:** three calendars merged and sorted, labels in order, a calendar without a label.
  - **Duplicates:** the same appointment on two calendars, shown once with both labels.
  - **All day and overnight:** all-day events first, a multi-day all-day event, a timed event running through the
    whole day (shown as all day), an event still running from the night before ("Until"), and an event running
    past midnight (shown today at its start, tomorrow with "Until").
  - **Locations:** Teams, Zoom, Google Meet and Webex shortened; a room together with a Teams meeting ("Room 4.12,
    Teams"); a location that merely contains "teams" ("Steamship Terminal") left alone; long locations cut; no
    locations at all.
  - **Settings:** cancelled meetings hidden and shown; tomorrow; the 24-hour clock; your own heading; no labels.
  - **Free days and errors:** a free day (with and without sending), and a calendar that does not exist (named as
    unavailable, the rest still listed).
  - **Size:** a meeting invite with a 200 KB description on two calendars.
  - **Sending:** the list sent to a notify action; a send that fails still returns the list.
  - **Published calendar links:** a Remote Calendar reading a published calendar link. With the refresh off it
    still showed the old copy of a calendar that had changed; with it on, the new event appeared, and a cancelled
    meeting was hidden. When the link stopped working, that calendar was named as unavailable and the others were
    still listed.
- Every change to this repository is checked automatically against Home Assistant 2025.4.0 and the current
  release (see [`.github/workflows/validate.yml`](.github/workflows/validate.yml)).

## How it works

When the script runs:

1. **Refresh.** It asks each calendar for its latest events (`homeassistant.update_entity`). A calendar that
   cannot refresh does not stop the script.
2. **Read.** It reads every event that overlaps the day, from local midnight to midnight
   (`calendar.get_events`), in Home Assistant's time zone, one calendar at a time. Days on which the clocks change
   are handled.
3. **Build the list.** Hidden cancelled meetings are dropped; each event becomes one line with its time, title,
   label and location; the same appointment on several calendars (same start time and same title, also when the
   calendars report it in different time zones) becomes one line with all its labels; all-day events come first,
   then events still running from the day before, then the day's events by start time. Calendars that could not
   be read are named at the end.
4. **Send and return.** If you set a notify action, it sends the list (unless you chose to send nothing on a free
   day). It always returns the list and its parts as the script's response.

```
  Work calendar ─┐
  Home calendar ─┼─▶ refresh ─▶ read today ─▶ merge, label, de-duplicate, sort ─▶ notify action (optional)
  Club calendar ─┘                                                              └─▶ response for your automations
```

## Installation

Import the blueprint with this button:

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FSudoManaged%2Fha-daily-schedule%2Fblob%2Fmain%2Fblueprints%2Fscript%2Fsudomanaged%2Fdaily_schedule.yaml)

Or by hand: **Settings, Automations & scenes, Blueprints, Import blueprint**, and paste
`https://github.com/SudoManaged/ha-daily-schedule/blob/main/blueprints/script/sudomanaged/daily_schedule.yaml`.

Or copy the file: put [`daily_schedule.yaml`](blueprints/script/sudomanaged/daily_schedule.yaml) in
`<config>/blueprints/script/sudomanaged/` and restart Home Assistant (or reload scripts).

## 1. Add your calendars to Home Assistant

Each calendar becomes an entity such as `calendar.work`. Add every calendar you want in the list; they can come
from different integrations and different accounts.

### Outlook (Microsoft 365 work or school accounts, and Outlook.com)

Outlook can publish a calendar as a private link that Home Assistant's **Remote Calendar** integration reads.

1. In **Outlook on the web**, open **Settings, Calendar, Shared calendars**.
2. Under **Publish a calendar**, choose the calendar and a level of detail that **includes titles and
   locations**. With the lowest level (free/busy only) every entry arrives as "Busy".
3. Select **Publish** and copy the **ICS** link (not the HTML one).
4. In Home Assistant: **Settings, Devices & services, Add integration, Remote Calendar**. Give it a name (for
   example `Work`; the entity becomes `calendar.work`) and paste the link.

Repeat for each account: three accounts become three Remote Calendars, each with its own label in the list.
Your organization can turn calendar publishing off; if **Publish a calendar** is missing, ask your administrator.

**Treat the link like a password:** anyone who has it can read the calendar. Do not paste it into forum posts or
issues. If it leaks, the same Outlook page can reset the links, which issues new ones; then edit the Remote
Calendar entry in Home Assistant with the new link.

### Google Calendar

Either:

- the built-in **Google Calendar** integration, which brings in every calendar of a Google account (it needs
  Google Cloud credentials set up once; its documentation walks you through them), or
- the calendar's **Secret address in iCal format** (Google Calendar settings, the calendar, **Integrate
  calendar**) in a **Remote Calendar**, as for Outlook. The same "treat it like a password" note applies.

### iCloud and other CalDAV calendars

The built-in **CalDAV** integration reads iCloud (with an app-specific password from your Apple account), Nextcloud
and other CalDAV servers. A public iCloud calendar link (`webcal://...`) also works in a Remote Calendar, as it
is.

### A calendar kept in Home Assistant

The built-in **Local Calendar** integration keeps a calendar inside Home Assistant; you can add events from the
calendar page.

After adding them, check **Settings, Devices & services, Entities** (search for `calendar.`) for their entity IDs.

## 2. Create the schedule script

From the UI: **Settings, Automations & scenes, Scripts, Create script**, and choose **Daily schedule (several
calendars)** among the blueprints. Fill in the settings and save; give the script a name such as
"Today's schedule" (its entity becomes `script.todays_schedule`).

| Setting | Default | Explanation |
|---|---|---|
| **Calendars** | (required) | The calendars to read. Their order is the order of the labels. |
| **Labels** | empty | Short labels for the calendars, in the same order, separated by commas, for example `Work, Home, Club`. A calendar without a label shows none; empty means no labels at all. |
| **Day** | Today | Today or tomorrow (for an evening "tomorrow" message). |
| **Refresh the calendars first** | on | Asks each calendar for its latest events before reading. Keep it on for Remote Calendar (Outlook and other links), which otherwise reads its link once a day. |
| **Notify action** | empty | Where to send the list, for example `notify.mobile_app_my_phone`, or a script of your own that takes `title` and `message`. Empty: only return it (for your own automation). Notify entities (`notify.send_message`) are not supported. |
| **Notification title** | empty | Empty: "Today's schedule" or "Tomorrow's schedule". |
| **Send nothing on a free day** | off | Skip the notification when nothing is scheduled (it is still sent when a calendar could not be read). |
| **Time format** | 12-hour | `9:00 AM` or `09:00`. |
| **Longest location** | 70 | Longer locations are cut to this many characters, followed by "...". 0 shows no locations. |
| **Shorten online meetings** | on | "Teams", "Zoom", "Google Meet" or "Webex" instead of the meeting link or the long meeting text. A room given together with the meeting stays: "Room 4.12, Teams". |
| **Hide cancelled meetings** | on | Hides events whose title starts with "Canceled:" or "Cancelled:". |
| **Heading** | empty | The first line of the full text. Empty: "Today:" or "Tomorrow:". |
| **Text for a free day** | `nothing scheduled.` | |
| **Word for all-day events** | `All day` | |
| **Word for events still running from the day before** | `Until` | |
| **Text for a calendar that cannot be read** | `calendar unavailable` | |

The wording settings let you translate the list. Times use your Home Assistant time zone.

From YAML (`scripts.yaml`):

```yaml
todays_schedule:
  alias: Today's schedule
  use_blueprint:
    path: sudomanaged/daily_schedule.yaml
    input:
      calendars:
        - calendar.work
        - calendar.home
        - calendar.club
      labels: "Work, Home, Club"
      notify_action: notify.mobile_app_my_phone
```

## 3. Get it every morning

The script does not run by itself; an automation runs it when you want. From the UI: **Settings, Automations &
scenes, Create automation, Create new automation**; add a **Time** trigger (for example 7:00) and, as the action,
**Scripts** and your schedule script. In YAML:

```yaml
- alias: Morning schedule
  triggers:
    - trigger: time
      at: "07:00:00"
  conditions:
    - condition: time
      weekday: [mon, tue, wed, thu, fri]
  actions:
    - action: script.todays_schedule
```

For an evening look at tomorrow, make a second script from the blueprint with **Day** set to Tomorrow and run it
at, say, 8:00 PM.

## 4. Test it

Run the script: on the **Scripts** page, open its menu and choose **Run**. The list arrives through your notify
action. To see what it returns without sending anything, go to **Developer tools, Actions**, choose your script
(for example `script.todays_schedule`) and **Perform action**; the response appears below. (With a notify action
set, it is sent then too.)

## What it looks like

A normal day (the default 12-hour clock, labels set):

```
Today's schedule
• All day: Office closed (Work)
• Until 1:30 AM Overnight deploy (Work)
• 9:00 AM Board call (Work, Home), Teams
• 11:30 AM Lunch with Sam (Work), Harbor Café, Pier 4
• 4:00 PM Dentist (Home), 123 Long Street Name, Suite 400, Some City, Some State 00000, Building...
```

Tomorrow, on the 24-hour clock:

```
Tomorrow's schedule
• All day: Vacation (Home)
• Until 01:00 Late call (Work), Zoom
• 10:00 Planning (Work)
```

A free day: `nothing scheduled.` (the full text reads `Today: nothing scheduled.`). A calendar that could not be
read is named at the end:

```
Today's schedule
• 7:00 AM Run (Club)
• Work: calendar unavailable
```

## Using it in your own briefing or automation

Run the script with `response_variable` and use what it returns. Leave **Notify action** empty if your own
automation does the sending.

```yaml
actions:
  - action: script.todays_schedule
    response_variable: schedule
  - action: notify.mobile_app_my_phone
    data:
      title: Good morning
      message: "{{ states('sensor.weather_summary') }}\n{{ schedule.text }}"
```

The response:

| Field | Example | Meaning |
|---|---|---|
| `text` | `Today:` + the list | The heading and the list, one appointment per line (or `Today: nothing scheduled.`). |
| `message` | the list | The list without the heading (what the notification carries). |
| `count` | `5` | How many appointments. |
| `items` | a list | One entry per appointment: `kind` (`allday`, `until` or `timed`), `time` (as shown), `title`, `labels`, `location` (as shown), `calendars` (entity IDs), `start` and `end` (as the calendar gives them). |
| `unavailable` | `["Work"]` | Calendars that could not be read, by label (or name). |
| `day` | `2026-11-05` | The date the list is for. |

Examples: `{{ schedule.count }} appointments today` for a speaker;
`{{ schedule['items'] | selectattr('kind', 'eq', 'timed') | map(attribute='title') | list }}` for just the
titles of the timed ones. (Write `schedule['items']` rather than `schedule.items`: `items` is also the name of a
dictionary method.)

## A calendar card for your dashboard

For the whole week at a glance, add Home Assistant's calendar card with the same calendars:

```yaml
type: calendar
initial_view: listWeek
entities:
  - calendar.work
  - calendar.home
  - calendar.club
```

## Keeping calendar details out of the history

Each calendar entity carries its next appointment in its attributes, and Home Assistant's history records its
title, location and times like any other state (the description is never recorded). If you would rather not keep
that history (for example because titles or locations name people, places or dial-in numbers), exclude the
calendars from the recorder in `configuration.yaml`:

```yaml
recorder:
  exclude:
    entities:
      - calendar.work
      - calendar.home
```

The schedule script does not need the history; it reads the calendars directly.

## Troubleshooting

- **"calendar unavailable".** Home Assistant could not read that calendar: open it under **Settings, Devices &
  services** and look for an error. For a Remote Calendar the usual causes are a link that was reset in Outlook (or
  Google), or no internet at the moment. A calendar entity that no longer exists (renamed or removed) shows up the
  same way; edit the script's calendars. The log then has an error from the script ("Service call requested
  response data but did not match any entities"): that is the unreadable calendar, and the script carries on.
- **An appointment added this morning is missing.** Keep **Refresh the calendars first** on. Outlook can also take
  a while to update a published link after a change.
- **Every entry says "Busy".** The Outlook calendar was published with free/busy only. Publish it again with a
  level that includes titles and locations; the link may change, so update the Remote Calendar.
- **Times are off by some hours.** Check Home Assistant's time zone under **Settings, System, General**.
- **An appointment appears twice.** The two copies differ in start time or title (for example a renamed copy), so
  they are not recognised as the same appointment.
- **A cancelled meeting still shows.** Its title does not start with "Canceled:" or "Cancelled:" (for example
  Outlook in another language). Newer Home Assistant releases also drop events that the calendar itself marks as
  cancelled.
- **No notification.** Open the script's **Traces** (in its menu): the trace shows the notify step and any error.
  A notify action that does not exist, or that rejects the message, stops the script with an error (a setup
  mistake to fix); a send that fails with an ordinary Home Assistant error is skipped and the list is still
  returned. With a script of your own as the notify action, an error in its templates also stops the schedule
  script.
- **The second of two schedules misses a change.** Home Assistant refreshes the same calendar at most once every
  10 seconds; when two schedule scripts start together (for example today's and tomorrow's at 7:00), only the
  first one refreshes. Start them a minute apart.
- **The script stops with an error from CalDAV (iCloud).** Home Assistant's CalDAV integration lets some server
  errors through to the script, which no blueprint can catch; the trace shows the error. It usually passes on the
  next run.

## Limitations

- **One day at a time:** today or tomorrow. For the week, use the calendar card above.
- **Labels follow the order of the calendars.** If you remove or reorder calendars, check the labels.
- **The same appointment** is recognised by the same start time and the same title (letter case ignored).
- **Cancelled meetings** are recognised by the title only, as described above.
- **"Untitled"** is used for an event without a title; it is the one word you cannot change.
- **A CalDAV (iCloud) server error** can stop the whole script instead of being reported as an unavailable
  calendar (see [Troubleshooting](#troubleshooting)).
- **Long lists:** phones may shorten a long notification until you expand it.
- **It reads, it never changes a calendar.**

## Updating and removing

- **Updating:** replace the blueprint file with the new version (or, from the blueprint's menu, **Re-import
  blueprint**), then reload scripts or restart Home Assistant.
- **Removing:** delete the scripts made from it and any automations that run them, then delete the blueprint.
  Your calendars are not affected.

## Security and privacy

- The blueprint runs inside your Home Assistant. It opens nothing to the network and uses no webhooks.
- It reads only the calendars you choose, and sends only to the notify action you set. A phone notification
  through the Home Assistant Companion app travels through Home Assistant's push notification relay and then
  Google's or Apple's push service, like every Companion app notification.
- **Published calendar links (Outlook, Google, iCloud) are secret links:** anyone with one can read that calendar.
  Keep them out of screenshots, posts and issues, and reset them if they leak.
- Calendar details also live in Home Assistant's history unless you exclude the calendars (see
  [Keeping calendar details out of the history](#keeping-calendar-details-out-of-the-history)).
- Like every script that reads a calendar, the script's **traces** (its last few runs, kept across restarts) hold
  what the calendars answered, including event descriptions such as meeting invites with dial-in numbers. Keep
  that in mind before sharing a trace.

## Reporting problems and contributing

- **Bugs and ideas:** open an issue with your Home Assistant version, the script's settings and, if possible, its
  trace. Remove calendar links and anything private first.
- **Security problems:** report them privately; see [SECURITY.md](SECURITY.md).
- **Changes:** pull requests are welcome. Every change must pass the configuration check in
  [`.github/workflows/validate.yml`](.github/workflows/validate.yml).

## More free Home Assistant tools from SudoManaged

- [Webhook alerts](https://github.com/SudoManaged/ha-webhook-alerts)
- [Bluetooth self-heal](https://github.com/SudoManaged/ha-bluetooth-self-heal)
- [Yealink desk phone call log](https://github.com/SudoManaged/ha-yealink-call-log)
- [Dehumidifier kit](https://github.com/SudoManaged/ha-dehumidifier-kit)
- [Internet failover alerts](https://github.com/SudoManaged/ha-internet-failover)

## Changelog and license

See [CHANGELOG.md](CHANGELOG.md). Released under the [MIT License](LICENSE). Copyright (c) 2026 SudoManaged.
