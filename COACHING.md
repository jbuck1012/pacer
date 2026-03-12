# PACER Workout Format & Coaching Reference

This document is the complete reference for writing valid workout files for the [PACER](https://jbuck1012.github.io/pacer/) interval timer app. Give this to any AI agent that needs to produce workout files for this repo.

---

## How the App Loads Workouts

1. The app fetches `workouts/manifest.json` on startup
2. If the manifest has exactly **one** entry, it auto-loads that workout (no selection screen)
3. The workout `.txt` file is parsed and rendered as a timed interval sequence with audio cues

**Rule: Keep exactly one workout in the manifest at a time.** This gives the best UX — open the app, workout is ready.

---

## Workout File Format

Files live in `workouts/` and are plain text (`.txt`).

### Header Lines

```
WORKOUT: 5K Tempo Session
DESCRIPTION: Sustained threshold effort with warmup and cooldown
THRESHOLD: 9:30/mile
```

- `WORKOUT: <name>` — Display name (defaults to "Untitled Workout" if omitted)
- `DESCRIPTION: <text>` — Short description shown in the workout list (optional, used by auto-manifest)
- `THRESHOLD: <value>` — Informational threshold pace (optional)

### Block Lines

```
TYPE: M:SS | Zone | Pace | Cue
```

**All four pipe-delimited fields are required.** Fields can be empty strings but the pipes must be present.

| Field | Format | Notes |
|-------|--------|-------|
| Duration | `M:SS` or `MM:SS` | Seconds must be 2 digits. Must be > 0:00. |
| Zone | Free text | Displayed in UI. e.g. "Easy", "Threshold", "VO2max" |
| Pace | Free text | Spoken by TTS. Leading `~` and `/mile` are stripped for speech. e.g. "~9:30/mile" |
| Cue | Free text | Spoken aloud + displayed. Keep under ~8 words (short intervals). |

### Valid Block Types

| Type | Color | Typical Use |
|------|-------|-------------|
| `WARMUP` | Blue | Opening easy effort |
| `COOLDOWN` | Blue | Closing easy effort |
| `BASE` | Amber | Steady aerobic effort |
| `PUSH` | Red | Sustained hard effort (tempo) |
| `INTERVAL` | Orange | Structured speed work |
| `RECOVERY` | Green | Easy effort between hard blocks |
| `REST` | Green | Complete rest / walk |
| `ALL-OUT` | Purple | Maximum effort sprint |

Types are **case-insensitive** in the file but stored uppercase internally.

### REPEAT

```
REPEAT: N
```

- `N` must be a positive integer
- Expands all blocks **between the last WARMUP and the COOLDOWN** N times
- WARMUP blocks before the repeat group are **not** repeated
- COOLDOWN blocks after the repeat group are **not** repeated
- Only one REPEAT directive is supported per workout

### Comments & Blank Lines

- Lines starting with `#` are ignored
- Blank lines are ignored

---

## Validation Rules

The parser will reject files with these errors:

| Check | Error |
|-------|-------|
| Unknown block type | `Line N: Unknown block type "X"` |
| Bad duration format | `Line N: Invalid duration "X" – expected M:SS or MM:SS` |
| Zero duration | `Line N: Duration cannot be zero` |
| Wrong field count | `Line N: Expected 4 fields (Duration \| Zone \| Pace \| Cue), got X` |
| No colon in line | `Line N: Unrecognized line format` |
| Bad REPEAT value | `Line N: REPEAT value must be a positive integer` |

---

## Manifest (Auto-Generated)

**File:** `workouts/manifest.json` — **do NOT edit manually.**

A GitHub Actions workflow automatically regenerates this file whenever workout files in `workouts/` are pushed. It scans every non-manifest file, extracts `WORKOUT:` and `DESCRIPTION:` lines, and writes the manifest.

The manifest format:

```json
{
  "workouts": [
    {
      "file": "tempo-30min.txt",
      "name": "30-Minute Tempo Run",
      "description": "Sustained threshold effort with warmup and cooldown"
    }
  ]
}
```

All workout files in the directory are included automatically. If only one workout exists, the app auto-loads it (no selection screen).

---

## Adding a Workout — Step by Step

1. **Create the file:** `workouts/<kebab-case-name>.txt`
2. **Include a `WORKOUT:` line** for the display name and optionally a `DESCRIPTION:` line
3. **Commit and push** with message: `Add workout: <Workout Name>`
4. The manifest is auto-updated by GitHub Actions — no manual manifest editing needed

---

## Example Workouts

### Easy Run (25 min)

```
WORKOUT: Easy Recovery Run
# Simple aerobic run, keep heart rate low

WARMUP: 5:00 | Easy | ~12:00/mile | Easy warmup
BASE: 15:00 | Easy | ~11:00/mile | Comfortable pace
COOLDOWN: 5:00 | Easy | ~12:00/mile | Cool down
```

### Tempo Run (30 min)

```
WORKOUT: 30-Minute Tempo
THRESHOLD: 9:30/mile

WARMUP: 5:00 | Easy | ~11:30/mile | Warm up gradually
BASE: 3:00 | Moderate | ~10:30/mile | Build into it
PUSH: 15:00 | Threshold | ~9:30/mile | Hold tempo pace
BASE: 2:00 | Moderate | ~10:30/mile | Ease off
COOLDOWN: 5:00 | Easy | ~12:00/mile | Cool down and stretch
```

### Intervals with REPEAT (35 min)

```
WORKOUT: 6x800m Intervals
THRESHOLD: 9:00/mile

WARMUP: 5:00 | Easy | ~11:30/mile | Easy warmup

# The interval set — repeated 6 times
INTERVAL: 3:00 | VO2max | ~8:00/mile | Push hard!
RECOVERY: 2:00 | Recovery | ~12:00/mile | Recover

REPEAT: 6

COOLDOWN: 5:00 | Easy | ~12:00/mile | Cool down
```

**Expanded result:** 5:00 warmup → (3:00 interval + 2:00 recovery) × 6 → 5:00 cooldown = 40:00 total

### Fartlek (30 min)

```
WORKOUT: 30-Minute Fartlek
# Unstructured speed play — no REPEAT needed

WARMUP: 5:00 | Easy | ~11:30/mile | Warm up
PUSH: 1:30 | Hard | ~8:30/mile | Pick it up!
RECOVERY: 1:00 | Easy | ~11:30/mile | Jog easy
PUSH: 2:00 | Hard | ~8:30/mile | Longer push
RECOVERY: 1:30 | Easy | ~11:30/mile | Recover
PUSH: 1:00 | Sprint | ~7:30/mile | Fast!
RECOVERY: 2:00 | Easy | ~11:30/mile | Catch your breath
PUSH: 3:00 | Threshold | ~9:00/mile | Sustained effort
RECOVERY: 2:00 | Easy | ~11:30/mile | Easy
PUSH: 0:30 | Sprint | ~7:00/mile | All out!
BASE: 5:00 | Easy | ~11:00/mile | Settle in
COOLDOWN: 5:00 | Easy | ~12:00/mile | Cool down
```
