# Changelog

All notable changes. Versions follow [Semantic Versioning](https://semver.org/).

## 1.0.0 (2026-11-05)

First public release.

- **Daily schedule (several calendars)** (script blueprint): today's or tomorrow's appointments from any number
  of calendars (Outlook and other calendar links through Remote Calendar, Google Calendar, CalDAV, Local
  Calendar) in one list sorted by time, one line each with time, title, calendar label and location; all-day
  events first; events still running from the day before shown with their end time; the same appointment on
  several calendars shown once with every label; online meetings shortened to Teams, Zoom, Google Meet or Webex;
  long locations cut (a room given with an online meeting stays); Outlook's cancelled meetings hidden; calendars
  refreshed before reading and read one at a time, so a calendar that cannot be read is named as unavailable
  without hiding the others; 12- or 24-hour clock and every word adjustable. Sends the list to a notify action if
  set, and returns it (text, list, count, items) for other automations, also when the sending fails.
- Requires Home Assistant 2025.4 or newer. Tested live on 2025.4.0, 2026.3.3 and 2026.11.0.
