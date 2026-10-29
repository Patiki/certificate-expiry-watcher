# Certificate Expiry Watcher — ServiceNow + Workato

## What it does

A daily check that watches certificate records in ServiceNow and warns the team on Slack before a certificate expires, so an expiry never surprises anyone overnight.

- Runs every day at 8 AM (Workato Scheduler).
- Reads the `sys_certificate` table in a ServiceNow dev instance.
- For each certificate, works out how many days are left until expiry.
- **7 days or fewer → urgent:** Slack alert + a high-priority ServiceNow incident.
- **30 days or fewer → warning:** Slack alert + a medium-priority ServiceNow incident.
- **More than 30 days left → silence.** Quiet days are a feature, not a bug.

## Why it exists

Expired certificates are one of the most common reasons big systems break overnight — websites go down, integrations fail, and nobody knows why until morning. This is the same expiry-scan work done in ITOM, automated.

## Tools used, and why

- **ServiceNow** — the source of the certificate records. Real IT data, not a practice sheet.
- **Workato** — the recipe that runs the daily check, the day-count math, the Slack alerts, and the ServiceNow tickets.
- **Slack** — where the team already watches, so the alert lands where people will see it.

## What was proven in testing

Tested end to end with test certificate records:

- Exactly 1 urgent Slack alert for `TEST-api-example` (4 days out).
- Exactly 1 warning Slack alert for `TEST-vpn-example` (20 days out).
- Silence for `TEST-shop-example`, which is safely in date.
- A real high-priority incident for the urgent case, and a real medium-priority incident for the warning case.
- **Duplicate guard proven:** a second full run created zero new tickets — the search-before-create check found the existing incidents and skipped.

## Real problems found and fixed during the build

1. **OAuth scope block** — the ServiceNow connection would not read `sys_certificate` until the registry entry was changed to "Broadly scoped".
2. **Wrong step in the warning check** — the warning IF was reading Step 2 (the whole list) instead of Step 3 (the one certificate being checked). Fixed by switching both fields to the Step 3 versions. Inside a Repeat, always use the repeat-item pills.
3. **A plain IF under an IF is not an Else** — the warning branch fired for urgent certs too, until it was rebuilt as a true ELSE IF on the urgent IF's No line.
4. **Duplicate tickets** — fixed with a search-before-create guard: search incidents by Correlation ID = certificate Name before each Create, and create only if none is found.
5. **Urgency field quirk** — ServiceNow's Create Urgency field is free text, not a dropdown. Typing "2 - Medium" is dropped and defaults to low; typing just "2" gives 2 - Medium.

## In progress

- **Gemini one-line risk note** in the urgent branch: an HTTP call to Gemini that writes one line on what breaks if the certificate expires, added as a second line in the urgent Slack alert. The step is built; waiting on a Gemini API outage to clear.
- **Start the recipe** for the live 8 AM run.
- **Clean up test artefacts** — the early test Slack messages and duplicate test incidents — before portfolio screenshots are taken.

## Demo note

The test run is scoped to `TEST-` certificates so the demo is clean. The same recipe works on the full certificate list once that scope is removed. The duplicate guard stops repeat tickets, not repeat Slack alerts.
