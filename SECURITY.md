# Security policy

## Reporting a vulnerability

Please report security problems **privately**: on this repository's **Security** tab choose
**Report a vulnerability**. Do not open a public issue for them.

Include the blueprint, your Home Assistant version, what an attacker could do, and the steps to reproduce it.
We aim to reply within a few days and to publish a fix, with credit if you want it, once it is ready.

## Scope

This blueprint runs entirely inside your own Home Assistant. It uses no webhooks and opens nothing to the
network; it reads the calendars you choose through Home Assistant's own calendar integrations and sends only to
the notify action you set. Reports about Home Assistant or its calendar integrations belong with the Home
Assistant project. Never include a published calendar link in a report: anyone with one can read that calendar.
