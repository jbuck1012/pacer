---
description: Add a workout to the PACER interval timer app. Accepts a workout definition and commits it to the repo.
---

# Add Workout

Commit a workout file to the PACER app repo and make it the active workout.

## Input

The user will provide a workout — either as a complete file, a description to convert, or pasted text. If the input is ambiguous, ask for clarification.

## Workout File Format

```
WORKOUT: Name
THRESHOLD: pace/mile  (optional)

TYPE: M:SS | Zone | Pace | Cue
```

- **Valid types:** WARMUP, COOLDOWN, BASE, PUSH, INTERVAL, RECOVERY, REST, ALL-OUT
- **Duration:** `M:SS` or `MM:SS`, must be > 0:00, seconds always 2 digits
- **All 4 pipe-delimited fields required** per block line
- **REPEAT: N** expands blocks between WARMUP and COOLDOWN N times
- Lines starting with `#` are comments. Blank lines ignored.
- Keep cues under ~8 words (spoken aloud by TTS)

See `COACHING.md` for the full format specification and examples.

## Steps

1. **Validate** the workout content against the format rules above. Fix any issues.
2. **Choose a filename:** `workouts/<kebab-case-name>.txt`
3. **Write the file** to `workouts/<filename>.txt`
4. **Update `workouts/manifest.json`** — replace the `workouts` array with exactly one entry:
   ```json
   {
     "workouts": [
       { "file": "<filename>.txt", "name": "<Workout Name>", "description": "<brief description>" }
     ]
   }
   ```
   Do NOT delete old `.txt` files from the `workouts/` directory.
5. **Show the user** the file contents and manifest entry for confirmation.
6. **Commit** both files with message: `Add workout: <Workout Name>`
7. **Push** to `origin main`
