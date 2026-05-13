# Rezone Intel Prompt Maintenance Guide (Client Version)

This guide explains how to safely edit the AI prompt used by Rezone Intel Generator.

## 1) Where to edit

Open this file:
- docs/Rezone Intel Generator.html

Main prompt function:
- `buildPrompt(cities, focus, title)`

## 2) What you can change safely

You can usually update these parts without breaking the app:
- Report focus wording (intro sentence)
- Tone/style instructions (for example: more concise or more analytical)
- Output depth rules (for example: number of projects or news items)
- Model generation settings in the API body:
  - `temperature`
  - `maxOutputTokens`

## 3) What you should NOT change unless a developer updates rendering

The page renderer expects specific JSON fields. If you rename/remove them, cards and charts may fail.

Avoid changing field names like:
- `cities`, `name`, `state`, `region`
- `pipeline`, `pipeNum`, `vacancy`, `yoy`, `statusLabel`, `sc`, `color`
- `catalyst`, `profile`, `stats`, `projects`, `timeline`, `finance`, `news`

If you must change schema fields, also update the rendering logic in the same HTML file.

## 4) Safe edit workflow

1. Duplicate current prompt block in `buildPrompt(...)` as backup.
2. Make one small change at a time.
3. Test with 2-3 cities first.
4. Confirm:
   - API returns valid JSON
   - No console errors
   - City cards, timeline, and charts render correctly
5. If output breaks, revert the last change and retry.

## 5) Quick examples

### Example A: Make output shorter
- Reduce required items in Rules (for example, 5 news -> 3 news).
- Keep JSON field names unchanged.

### Example B: Ask for more policy detail
- In each city `news` and `timeline` instructions, request policy name + date + source.
- Keep the same JSON structure.

## 6) Recommended change control

- Save prompt edits with a dated note in git commit message.
- Keep a small changelog line in project docs, such as:
  - `2026-05-12: Prompt tuned for shorter city briefs and clearer policy citations.`
