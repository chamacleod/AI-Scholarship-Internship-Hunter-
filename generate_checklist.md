# Workflow: Generate Application Checklist

## Objective
Produce a step-by-step application checklist for a saved opportunity, personalized to that opportunity's requirements.

## Trigger
User visits `/checklists/{opp_id}` and clicks "Generate with AI", or clicks "Regenerate checklist".

## How It Works
`tools/ai/checklist.py` calls Claude Haiku with the opportunity title, type, description, and deadline.
Output: a JSON array of 6–12 action strings. Stored as `[{"text": "...", "done": false}, ...]` in `checklists.steps_json`.

## Toggling Steps
Each step has a checkbox. Clicking it POSTs to `/checklists/{opp_id}/toggle/{step_index}`, which flips the `done` boolean in place and redirects back.

## Improving Checklist Quality
If the generated checklists are too generic, enrich the prompt in `tools/ai/checklist.py` by:
1. Including the user's profile (citizenship, education level) so steps like "confirm citizenship eligibility" are surfaced.
2. Providing the `ai_summary` bullets for more targeted steps.
3. Adjusting the system prompt's step count or specificity instructions.

## Regeneration
Regenerating overwrites the existing checklist steps — **all progress (checked items) is lost**. The UI warns the user before they confirm. If you want to preserve progress, modify `checklists.py` to only append new steps instead of replacing.
