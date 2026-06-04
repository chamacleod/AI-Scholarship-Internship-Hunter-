# Workflow: Match & Score Opportunities

## Objective
Score every opportunity in the database against each user's profile, storing a 0–100 match score and a one-line reason in `user_opportunities`.

## When This Runs
- **Automatically** (triggered from the profile save endpoint) when a user updates their profile.
- **Manually** if you need to force a re-score (e.g., after updating scoring prompts).

## How It Works
`tools/ai/matcher.py` calls Claude Haiku with:
- System prompt (cached with `cache_control: ephemeral`): scoring rubric
- User message: student profile JSON + opportunity details

One API call per (user, opportunity) pair. The system prompt is cached so repeat calls for the same rubric are cheaper.

## Trigger via Profile Save
Scoring is automatically triggered when a user saves their profile (`/profile` POST). Only **unscored** opportunities are processed — opportunities already in `user_opportunities` for that user are skipped.

## Manual Re-score
If you need to wipe and re-score all opportunities for all users (e.g., after changing the scoring prompt):
1. Open a Python shell: `python`
2. Run:
```python
from tools.db.database import init_db, SessionLocal
from tools.db.models import UserOpportunity
db = SessionLocal()
db.query(UserOpportunity).delete()
db.commit()
```
3. Have each user visit `/profile` and save (re-triggers scoring), OR implement a CLI command.

## Cost Considerations
- Each score = ~150 input tokens + ~50 output tokens with Haiku (~$0.0003 per call).
- 100 opportunities × 50 users = 5,000 calls ≈ $1.50. Fine for moderate scale.
- If scaling beyond ~10,000 calls/day, add a batch scoring job instead of inline scoring.
