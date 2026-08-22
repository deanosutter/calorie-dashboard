# Calorie + Workout Logger — Schema v4

## Canonical repository
`deanosutter/calorie-dashboard`

## Governing architecture
Mutable day-level tracking data is stored in `data/daily/YYYY-MM-DD.json`. Each date is an independent record. `data/manifest.json` lists dates with v4 files. `data/settings.json` stores tracker settings. The legacy root `data.json` is read-only historical/reference data during migration and must not receive new day-level entries.

## Mandatory read-before-write rule
For any explicit log, track, save, sync, set, or update request:
1. Determine the target date in America/Chicago.
2. Fetch `data/manifest.json`.
3. Fetch `data/daily/YYYY-MM-DD.json` if it exists.
4. If it exists, preserve all unrelated fields and entries, append/update only the requested information, and write that same daily file using its current SHA.
5. If it does not exist, create it using the daily schema and add the date to the manifest.
6. Never overwrite another day's file to log the current day.
7. Never modify legacy `data.json` for ordinary logging.

## Daily schema
```json
{
  "schema_version": 4,
  "date": "YYYY-MM-DD",
  "food_entries": [],
  "workout_entries": [],
  "daily_active_calories": null,
  "weight_entries": [],
  "updated_at": "ISO-8601 America/Chicago timestamp"
}
```

## Food logging
Use saved reference values from legacy `data.json` or later dedicated reference files when available. Preserve calories, protein, carbs, fat, source, meal, quantity, unit, and reference IDs when known. Do not invent exact nutrition when the amount or product is materially ambiguous.

## Workout logging
Store routine, time when known, source, and detailed notes including exercise, weight, reps, sets, and per-arm/per-side distinctions. Apple Health workout metrics may be merged into the same day's file later without replacing user-reported strength details.

## Active calories and Health
Apple Health Active Energy is day-level data. Store it in the target daily file under `daily_active_calories`. Synced body weight belongs in `weight_entries`. Health syncs must merge with existing food/workout data rather than replace it.

## Safety / integrity
- Read the current target daily file immediately before every write.
- Preserve unrelated data.
- Do not create duplicates when correcting an existing entry.
- Use GitHub SHA-based updates.
- A failure affecting one day must not alter any other date.
- Routine confirmations should be short.

## Migration policy
Legacy `data.json` remains intact as a historical archive and reference library until all historical dates and reusable reference data have been verified in v4. The dashboard may merge legacy history with v4 daily files during this transition. Once verified, legacy data can be split into dedicated reference files and archived; it should not be deleted merely to complete the migration.
