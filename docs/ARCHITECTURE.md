# Architecture

## System Overview

```
User (WhatsApp)
    |
    v
Claude Code + WhatsApp Plugin
    |
    v
Group Config (config.md) -- Defines Kai's personality and tool usage
    |
    v
kai-cli.py -- CLI interface for all data operations
    |
    v
db_manager.py -- SQLite database layer
    |
    v
kai_health.db -- Local SQLite database
```

## Components

### 1. kai-cli.py (CLI Interface)

The main entry point. It provides a command-line interface for all health/fitness operations.

**Responsibilities:**
- Parse command-line arguments
- Resolve configuration (CLI flags > env vars > .env file > defaults)
- Format and display output
- Implement smart features (workout suggestions, weekly planning)

**Key design decisions:**
- Uses `argparse` with subcommands for a clean CLI experience
- All path configuration is externalized (no hardcoded paths)
- Loads .env file from project root automatically
- Each command function receives `args` and resolves its own DB/profile paths

### 2. db_manager.py (Database Layer)

Standalone SQLite database module. All functions accept a `db_path` argument.

**Tables:**

| Table | Purpose |
|---|---|
| `body_metrics` | Weight, body fat, BMI, muscle mass, etc. |
| `food_logs` | Meal entries with calories and macros |
| `sleep_logs` | Sleep date, bedtime, wake time, duration, quality |
| `workout_logs` | Workout sessions with location, target muscle, duration |
| `exercise_logs` | Individual exercises with weight, sets, reps, RPE |

**Design principles:**
- Pure functions: each function opens and closes its own connection
- No global state or singletons
- Returns Python dicts/lists, not database rows
- Error handling returns None or empty lists (never raises)

### 3. exercises.md (Exercise Database)

A Markdown file containing 70+ exercises organized by muscle group and equipment type.

**Format:**
```markdown
## Muscle Group
### Equipment: Type
- Exercise Name
```

This format is parsed by `_parse_exercises_md()` in kai-cli.py into a nested dictionary:
```python
{
  "Chest": {
    "Barbell": ["Barbell Bench Press", "Incline Barbell Press"],
    "Bodyweight": ["Push-ups", "Dips"]
  }
}
```

### 4. profile.json (User Profile)

JSON file containing the user's fitness profile, goals, and equipment.

Used by:
- `suggest-workout` -- Equipment determines available exercises; goals affect set/rep schemes
- `weekly-plan` -- Frequency target and muscle rotation
- `daily-summary` -- Nutrition targets for remaining calorie calculations

### 5. Cron Reminders (workout-reminder.sh)

A shell script that:
1. Runs `kai-cli.py quick-status` and `last-workout`
2. Passes the output to Claude Code with a prompt
3. Claude generates a personalized message
4. Sends it to the WhatsApp group via the plugin

## Data Flow

### Logging Flow
```
User message: "I ate chicken breast with rice for lunch"
    |
    v
Claude (Kai persona) estimates macros
    |
    v
kai-cli.py log-food "Chicken breast with rice" 550 45 60 12
    |
    v
db_manager.insert_food_log() -> SQLite
    |
    v
kai-cli.py also calls get_daily_nutrition_summary()
    |
    v
Output: "Logged food: ... Today's totals: 1200/2200 kcal"
    |
    v
Claude formats response in friendly tone -> WhatsApp reply
```

### Suggestion Flow
```
User: "What should I train today?"
    |
    v
kai-cli.py suggest-workout --duration 40
    |
    1. Load profile -> equipment, rotation, goals
    2. Query recent workouts -> find least-recently-trained muscles
    3. Check missed muscle groups -> prioritize overdue (5+ days)
    4. Query sleep data -> adjust intensity (sets/reps)
    5. Parse exercises.md -> available exercises for equipment
    6. Build workout plan -> warm-up, exercises, cool-down
    |
    v
Output: Structured workout plan with exercise selection
```

### Reminder Flow (Cron)
```
Cron (10 AM daily)
    |
    v
workout-reminder.sh
    |
    v
kai-cli.py quick-status  -> "Last workout: 2 days ago, 0 meals today"
kai-cli.py last-workout   -> "Chest at gym, 40 min"
    |
    v
Claude Code (with WhatsApp plugin)
    |
    v
Claude reads status, generates context-aware message:
  "Hey! It's been 2 days since your chest day. Time to get moving!"
    |
    v
WhatsApp group message
```

## Smart Features

### Workout Suggestion Engine

1. **Muscle Rotation** -- Tracks days since each muscle group was last trained
2. **Overdue Detection** -- Flags muscle groups not trained in 5+ days
3. **Sleep-Based Intensity** -- Adjusts volume based on 3-day average sleep:
   - <6h sleep: 2-3 sets x 8-10 reps (recovery mode)
   - 6-7h sleep: 3 sets x 10-12 reps (moderate)
   - 7-8h sleep: 3-4 sets x 8-12 reps (standard)
   - 8h+ sleep: 4 sets x 8-12 reps (push hard)
4. **Equipment Matching** -- Only suggests exercises you can do with your equipment
5. **Duration Scaling** -- Adjusts number of muscle groups and exercises per group

### Weekly Planning

1. Counts workouts done this week (Monday-Sunday)
2. Compares against target frequency from profile
3. Shows which muscle groups have/haven't been trained
4. Generates catch-up suggestions if behind schedule
