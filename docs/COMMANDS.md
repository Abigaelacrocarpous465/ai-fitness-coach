# CLI Command Reference

All commands are run via `python3 src/kai-cli.py <command> [args]`.

## Global Options

These flags can be added to any command:

| Flag | Description |
|---|---|
| `--db PATH` | Override SQLite database path |
| `--profile PATH` | Override profile.json path |
| `--exercises PATH` | Override exercises.md path |

Example:
```bash
python3 src/kai-cli.py --db /custom/path.db quick-status
```

---

## Logging Commands

### `log-food`

Log a meal or food item.

```
log-food <description> <calories> <protein_g> <carbs_g> <fat_g> [--image PATH] [--notes TEXT]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `description` | string | Yes | Food description |
| `calories` | float | Yes | Calories (kcal) |
| `protein_g` | float | Yes | Protein in grams |
| `carbs_g` | float | Yes | Carbs in grams |
| `fat_g` | float | Yes | Fat in grams |
| `--image` | string | No | Path to food image |
| `--notes` | string | No | Additional notes |

**Example:**
```bash
python3 src/kai-cli.py log-food "Grilled chicken salad" 450 42 20 18 --notes "lunch"
```

---

### `log-weight`

Log a body weight measurement.

```
log-weight <weight_kg>
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `weight_kg` | float | Yes | Weight in kilograms |

**Example:**
```bash
python3 src/kai-cli.py log-weight 71.2
```

---

### `log-workout`

Log a completed workout session.

```
log-workout <location> <target_muscle> <duration_min> <exercises_json> [--notes TEXT]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `location` | string | Yes | Where (e.g. "Home Gym", "Planet Fitness") |
| `target_muscle` | string | Yes | Primary muscle group |
| `duration_min` | int | Yes | Duration in minutes |
| `exercises_json` | string | Yes | Exercises performed (text or JSON) |
| `--notes` | string | No | Additional notes |

**Example:**
```bash
python3 src/kai-cli.py log-workout "Home Gym" "Chest" 35 "Bench Press 3x10, Push-ups 3x15"
```

---

### `log-sleep`

Log a sleep entry.

```
log-sleep <sleep_date> <bedtime> <wake_time> <duration_hours> [--quality good|ok|bad] [--notes TEXT]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `sleep_date` | string | Yes | Date (YYYY-MM-DD) |
| `bedtime` | string | Yes | Bedtime (e.g. "23:30") |
| `wake_time` | string | Yes | Wake time (e.g. "07:00") |
| `duration_hours` | float | Yes | Hours slept |
| `--quality` | choice | No | good, ok, or bad |
| `--notes` | string | No | Additional notes |

**Example:**
```bash
python3 src/kai-cli.py log-sleep 2025-01-15 23:30 07:00 7.5 --quality good
```

---

### `log-exercise`

Log a single exercise for progressive overload tracking.

```
log-exercise <exercise_name> <muscle_group> <weight_lbs> <sets> <reps> [--rpe FLOAT] [--notes TEXT]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `exercise_name` | string | Yes | Exercise name (e.g. "Bench Press") |
| `muscle_group` | string | Yes | Muscle group (e.g. "Chest") |
| `weight_lbs` | float | Yes | Weight in pounds |
| `sets` | int | Yes | Number of sets |
| `reps` | int | Yes | Reps per set |
| `--rpe` | float | No | Rate of Perceived Exertion (1-10) |
| `--notes` | string | No | Additional notes |

**Example:**
```bash
python3 src/kai-cli.py log-exercise "Bench Press" "Chest" 135 3 10 --rpe 7.5
```

---

## Query Commands

### `quick-status`

Quick overview of current state.

```
quick-status
```

Shows: last workout, latest weight, last sleep, today's meals and calories.

---

### `daily-summary`

Show nutrition summary for a day.

```
daily-summary [date]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `date` | string | No | Date (YYYY-MM-DD), defaults to today |

---

### `weekly-summary`

Show the last 7 days of nutrition and workout data.

```
weekly-summary
```

---

### `weight-trend`

Show weight history.

```
weight-trend [entries]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `entries` | int | No | Number of entries to show (default: 14) |

---

### `sleep-trend`

Show sleep history and average.

```
sleep-trend [days]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `days` | int | No | Number of days to show (default: 7) |

---

### `last-workout`

Show details of the most recent workout.

```
last-workout
```

---

## Smart Commands

### `suggest-workout`

Generate an intelligent workout suggestion based on your history, equipment, and sleep.

```
suggest-workout [--duration MINUTES] [--focus MUSCLE]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `--duration` | int | No | Workout duration in minutes (default: 40) |
| `--focus` | string | No | Muscle group focus |

**Focus options:**
| Value | Maps to |
|---|---|
| `chest` | Chest |
| `back` | Back |
| `legs` | Legs |
| `shoulders` | Shoulders |
| `biceps` | Biceps |
| `triceps` | Triceps |
| `arms` | Biceps + Triceps |
| `core` / `abs` | Core |
| `upper` | Chest + Back + Shoulders |
| `lower` | Legs + Core |
| `push` | Chest + Shoulders + Triceps |
| `pull` | Back + Biceps |

**Example:**
```bash
python3 src/kai-cli.py suggest-workout --duration 30 --focus push
```

---

### `weekly-plan`

Show weekly progress and catch-up suggestions.

```
weekly-plan
```

Shows:
- Workouts done this week vs target
- Muscle groups trained and not yet trained
- Days since each untrained group was last worked
- Suggestions for what to do next

---

### `strength-trend`

Show progressive overload trends.

```
strength-trend [exercise_name] [--limit N]
```

| Argument | Type | Required | Description |
|---|---|---|---|
| `exercise_name` | string | No | Specific exercise (omit for all) |
| `--limit` | int | No | Number of entries (default: 10) |

**Examples:**
```bash
# Summary of all exercises
python3 src/kai-cli.py strength-trend

# Detailed history for one exercise
python3 src/kai-cli.py strength-trend "Bench Press" --limit 20
```
