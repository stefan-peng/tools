# Coffee Recipe Learning Spec (v1)

## Purpose
Define how the coffee tool should:
- let users choose roast types (`light`, `medium`, `dark`) for generation,
- persist those preferences in browser storage,
- save generated recipes with thumbs-up/thumbs-down feedback,
- derive preferred brew parameters from saved ratings over time.

This document is implementation-oriented and scoped to a single static HTML app using `localStorage`.

## Goals
1. Roast filter is user-configurable and persistent across reloads.
2. Users can save a generated recipe snapshot and rate it quickly.
3. Ratings are editable after save.
4. App computes understandable preference insights from rated recipes.
5. Data schema is versioned and migration-safe.

## Non-goals (v1)
1. Account sync or cloud backup.
2. Advanced ML/recommendation models.
3. Multi-user support.
4. External database or server APIs.

## High-level UX
1. Controls area includes roast checkboxes: `light`, `medium`, `dark`.
2. Roast choices apply immediately to new generations.
3. `Save recipe` stores the currently rendered recipe.
4. Saved recipes list supports:
- thumbs up/down toggle,
- quick delete,
- optional search/filter in later phases.
5. Insights panel summarizes preferred ranges and confidence.

## Data Model

### Storage keys
- `coffee.settings.v1`
- `coffee.recipes.v1`

### `coffee.settings.v1` schema
```json
{
  "version": 1,
  "roasts": ["light", "medium", "dark"],
  "updatedAt": "2026-02-11T00:00:00.000Z"
}
```

Rules:
1. `roasts` must contain at least one valid roast.
2. Invalid or empty settings reset to defaults.

### `coffee.recipes.v1` schema
```json
{
  "version": 1,
  "items": [
    {
      "id": "r_1739260800000_ab12cd",
      "createdAt": "2026-02-11T00:00:00.000Z",
      "updatedAt": "2026-02-11T00:00:00.000Z",
      "rating": "up",
      "recipe": {
        "methodName": "Pour-over",
        "metaPill": "V60 / flat-bottom",
        "roast": "light",
        "fields": [
          ["Dose", "18.0 g"],
          ["Ratio", "1:16.0"]
        ],
        "params": {
          "doseG": 18,
          "ratio": 16,
          "waterG": 288,
          "tempC": 94,
          "opus": 6.8,
          "bloomSec": 40,
          "pours": 3,
          "intervalSec": 35,
          "swirls": 1,
          "targetTotalSec": 210
        },
        "steps": [
          "Heat water to 94°C. Rinse filter and preheat brewer."
        ]
      }
    }
  ]
}
```

Rules:
1. `rating` enum: `"up" | "down" | null`.
2. `recipe` is immutable snapshot content from generation time.
3. Keep max `items` count (recommended: `500`) by dropping oldest when exceeded.

## Recipe generation requirements
1. Every generated recipe must include a `roast` field.
2. Roast is sampled only from enabled `settings.roasts`.
3. If generated candidate violates constraints, retry up to `MAX_ATTEMPTS`.
4. Fallback recipe must honor currently enabled roast selection.

## UI/interaction requirements

### Roast selection
1. Render 3 checkboxes (`light`, `medium`, `dark`).
2. Prevent unchecking the final active roast.
3. Persist on each change.

### Save + rate flow
1. Current recipe view has `Save recipe` action.
2. On save:
- assign unique `id`,
- persist snapshot with `rating: null`,
- show success status.
3. Saved list row includes:
- recipe metadata (date, method, roast, key params),
- thumbs-up button,
- thumbs-down button,
- delete button.
4. Clicking same thumb again clears rating (toggle behavior).
5. Rating update changes `updatedAt` and re-runs insights.

## Preference insights (v1 algorithm)
Compute from `coffee.recipes.v1.items` where `rating !== null`.

### Minimum data thresholds
1. Global minimum rated recipes: `10`.
2. Slice minimum for method+roast insights: `5`.
3. If below thresholds, show “Need more ratings”.

### Slicing
Compute stats per:
1. method (`AeroPress`, `Pour-over`),
2. roast (`light`, `medium`, `dark`),
3. method+roast.

### Numeric parameter stats
For each numeric param (for example: `doseG`, `ratio`, `tempC`, `opus`, timing params):
1. Separate `up` and `down` sets.
2. For `up` set compute:
- median,
- p25/p75 (preferred range).
3. Compute directional contrast against `down` median if sample exists.

Output format example:
- `Temp (Pour-over, light): prefer 92–95°C, median 94°C, confidence medium`.

### Categorical parameter stats
For categorical params (for example technique type):
1. Compute up-rate by option:
- `up_rate = up_count / (up_count + down_count)`.
2. Require at least `3` ratings per option before display.

### Confidence heuristic
Confidence label per insight:
1. `low`: 5-9 rated samples.
2. `medium`: 10-24 rated samples.
3. `high`: 25+ rated samples.

## Adaptive generation (phase 2+)
Not required for initial storage/rating release, but schema should support it.

When enabled:
1. Keep baseline random behavior.
2. Bias sampling toward preferred p25-p75 ranges for highly confident params.
3. Do not hard lock to preferences; maintain exploration (for example 30% unbiased samples).

## Storage handling and migration
1. Wrap all `JSON.parse` calls in try/catch.
2. Validate loaded objects with explicit runtime checks.
3. On invalid payload:
- keep backup in memory for debugging,
- reset to defaults,
- continue without crashing.
4. Future migration entrypoint:
- `migrateSettings(raw)` and `migrateRecipes(raw)`.

## Performance and limits
1. Recompute insights incrementally or on-demand after mutations.
2. Avoid full DOM rerender of saved list for single rating change if practical.
3. Keep localStorage payload under browser limits (cap list size).

## Accessibility requirements
1. Roast controls and thumbs buttons must be keyboard accessible.
2. Rating state must be exposed with `aria-pressed`.
3. Save/rate/delete actions announce status via live region.

## Testing plan

### Unit-level logic tests (manual or lightweight harness)
1. Settings validator enforces at least one roast.
2. Recipe serialization/deserialization round-trip.
3. Rating toggles (`null -> up -> null`, `null -> down -> null`, `up -> down`).
4. Insight calculations:
- median and quartiles correctness,
- confidence buckets,
- threshold gating.

### Integration checks
1. Roast selection persists after reload.
2. Saved recipes appear after reload.
3. Rating update immediately changes insights.
4. Corrupt localStorage does not break app load.

## Rollout phases
1. Phase 1: settings persistence + roast-based generation.
2. Phase 2: save recipe + thumbs rating + saved list.
3. Phase 3: insights panel from rated history.
4. Phase 4: optional adaptive generation bias.

## Open decisions
1. Should `rating` be required at save time or optional (recommended: optional)?
2. Keep deleted recipes recoverable in-session (undo) or hard delete?
3. Show one global insights view or split by method tabs by default?

