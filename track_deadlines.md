# Workflow: Track Deadlines

## Objective
Ensure users can see upcoming deadlines for saved opportunities, sorted by urgency.

## How Deadlines Are Stored
`opportunities.deadline` is a UTC `datetime`. Scrapers set it to `None` when they can't parse a deadline from the source page.

## Parsing Deadlines from Scraped Pages
Most scrapers currently set `deadline=None`. To parse deadlines:
1. Find the deadline text in the scraper (e.g., `deadline_el.get_text()`).
2. Parse with `dateutil.parser.parse(text)` — add `python-dateutil` to `requirements.txt`.
3. Assign the resulting `datetime` object to the `"deadline"` key in the returned dict.
4. Update `upsert_opportunities` in `tools/scrapers/run.py` if deadline is a string that needs conversion.

## Deadline View
`/deadlines` shows all saved opportunities with a non-null deadline on or after today, sorted ascending. Opportunities with no deadline are excluded. Color coding:
- Red: ≤7 days
- Amber: ≤30 days
- Gray: >30 days

## Adding Email Notifications (Future)
1. Add `SMTP_HOST`, `SMTP_USER`, `SMTP_PASS` to `.env`.
2. Create `tools/notifications/email.py` with a function that sends a digest of deadlines within 7 days.
3. Run it as a daily cron or scheduled task.
4. Filter to only send if the user has opted in (add `notify_email` boolean to `Profile`).
