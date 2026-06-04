# Workflow: Scrape Opportunities

## Objective
Populate the database with fresh scholarship, internship, competition, and program listings from all configured sources.

## When to Run
- On first setup (after `python -m tools.db.seed` for dev data)
- Weekly to catch new postings and updated deadlines
- After adding a new scraper to `tools/scrapers/`

## Steps

1. **Activate environment** and ensure `.env` is present with valid keys.
2. **Run scrapers:**
   ```
   python -m tools.scrapers.run
   ```
   To skip LinkedIn (slower, uses Playwright):
   ```
   python -m tools.scrapers.run --no-linkedin
   ```
3. **Check output** — note how many new records were inserted vs. already present.
4. **Verify in app** — visit `/opportunities` and confirm new listings appear.

## Caching
Raw HTML is cached in `.tmp/` per URL (MD5-keyed). To force a fresh scrape, delete `.tmp/` files for that scraper:
```
rm .tmp/yconic_*.html
```

## When a Scraper Breaks
1. Check if the target site's HTML structure changed (inspect the live page).
2. Update CSS selectors in the relevant `tools/scrapers/*.py` file.
3. Delete the cached `.tmp/` file for that URL so it re-fetches.
4. Re-run and verify.
5. Update this workflow if the fix reveals a recurring pattern.

## Rate Limits
- Default: 1.5s between requests (`base.py`).
- LinkedIn: 3s. Do not reduce this — LinkedIn blocks fast scrapers.
- If a site returns 429, increase `rate_limit_seconds` in that scraper class.
