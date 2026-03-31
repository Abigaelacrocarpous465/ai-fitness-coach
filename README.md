# Kai Fitness

**AI-powered WhatsApp fitness coach** with workout tracking, nutrition logging, and smart recommendations.

Kai is a personal fitness agent that lives in your WhatsApp group. It tracks your workouts, nutrition, sleep, and weight -- then uses that data to give you smart workout suggestions, hold you accountable, and send personalized reminders via cron jobs.

Built on [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with the WhatsApp channel plugin.

## Features

- **Workout Tracking** -- Log sessions with muscle group, duration, location, and exercises
- **Smart Workout Suggestions** -- AI picks exercises based on what you haven't trained recently, adjusts intensity based on your sleep data
- **Progressive Overload Tracking** -- Log weight/sets/reps per exercise, see strength trends over time
- **Nutrition Logging** -- Track calories, protein, carbs, and fat for every meal
- **Weight Tracking** -- Log daily weigh-ins, see trends
- **Sleep Tracking** -- Log sleep times and quality, affects workout intensity recommendations
- **Weekly Planning** -- Smart rescheduling that shows what muscle groups are overdue
- **Automated Reminders** -- Cron-powered WhatsApp messages that check your data and send context-aware nudges
- **Exercise Database** -- 70+ exercises categorized by muscle group and equipment type

## Quick Start (5 minutes)

### Prerequisites

- **Python 3.8+** (uses only standard library -- no pip installs needed)
- **Claude Code** CLI ([install guide](https://docs.anthropic.com/en/docs/claude-code/getting-started))
- **WhatsApp channel plugin** for Claude Code (for the WhatsApp integration)

### Setup

```bash
# Clone the repo
git clone https://github.com/moltbot0912/kai-fitness.git
cd kai-fitness

# Run the setup script
chmod +x setup.sh
./setup.sh
```

The setup script will:
1. Check your Python version
2. Create config files from templates
3. Initialize the SQLite database
4. Guide you through profile customization
5. Optionally install cron jobs for automated reminders

### Manual Setup (if you prefer)

```bash
# 1. Copy config templates
cp config/.env.example .env
cp config/profile.example.json config/profile.json

# 2. Edit your profile
nano config/profile.json  # Set your name, goals, gym equipment, etc.

# 3. Initialize the database
python3 src/db_manager.py data/kai_health.db

# 4. Test it
python3 src/kai-cli.py quick-status
```

## CLI Commands

### Logging Data

```bash
# Log a meal
python3 src/kai-cli.py log-food "Chicken breast and rice" 550 45 60 12

# Log weight
python3 src/kai-cli.py log-weight 70.5

# Log a workout session
python3 src/kai-cli.py log-workout "Home Gym" "Chest" 40 "Bench Press, Dumbbell Fly, Push-ups"

# Log sleep
python3 src/kai-cli.py log-sleep 2025-01-15 23:30 07:00 7.5 --quality good

# Log a single exercise (for progressive overload tracking)
python3 src/kai-cli.py log-exercise "Bench Press" "Chest" 135 3 10 --rpe 8
```

### Querying Data

```bash
# Quick status overview
python3 src/kai-cli.py quick-status

# Today's nutrition
python3 src/kai-cli.py daily-summary

# Last 7 days overview
python3 src/kai-cli.py weekly-summary

# Weight trend
python3 src/kai-cli.py weight-trend

# Sleep history
python3 src/kai-cli.py sleep-trend

# Last workout details
python3 src/kai-cli.py last-workout
```

### Smart Features

```bash
# Get a workout suggestion (auto-picks muscle groups you haven't trained)
python3 src/kai-cli.py suggest-workout

# Request specific duration or focus
python3 src/kai-cli.py suggest-workout --duration 30 --focus chest

# Weekly plan with catch-up suggestions
python3 src/kai-cli.py weekly-plan

# Strength progression for all exercises
python3 src/kai-cli.py strength-trend

# Strength progression for a specific exercise
python3 src/kai-cli.py strength-trend "Bench Press"
```

## Deployment Options

### Option A: Local PC (always-on machine)

Best if you have a Mac/Linux machine that stays on.

1. Follow the Quick Start above
2. Install cron jobs: `./cron/install-cron.sh`
3. Keep your machine running for the cron reminders to fire

### Option B: Cloud VM ($3-5/month)

Best for reliability. See [docs/AWS_SETUP.md](docs/AWS_SETUP.md) for a step-by-step guide.

**TL;DR:**
1. Launch an AWS Lightsail or EC2 t3.micro instance (or any $5/month VPS)
2. Install Python 3, Claude Code, and the WhatsApp plugin
3. Clone this repo and run `setup.sh`
4. Install cron jobs
5. Done -- reminders run 24/7 even when your laptop is off

## Configuration

### Profile (`config/profile.json`)

Your fitness profile controls workout suggestions, nutrition targets, and equipment availability.

Key fields:
- `user_name` -- Your name (used in reminders)
- `fitness_goals.primary_goal` -- Drives set/rep recommendations
- `nutrition_targets` -- Daily calorie and macro goals
- `workout_preferences.frequency_per_week` -- Target workout frequency (e.g. "3-4")
- `workout_preferences.gym_locations` -- Your equipment (determines which exercises are suggested)
- `workout_preferences.preferred_muscle_group_rotation` -- Muscle groups to cycle through

### Environment Variables (`.env`)

| Variable | Default | Description |
|---|---|---|
| `KAI_DB_PATH` | `data/kai_health.db` | SQLite database location |
| `KAI_PROFILE_PATH` | `config/profile.json` | Profile JSON location |
| `KAI_EXERCISES_PATH` | `src/exercises.md` | Exercise database location |
| `KAI_TIMEZONE` | (system default) | Your timezone |
| `KAI_WHATSAPP_CHAT_ID` | (none) | WhatsApp group ID for reminders |

### WhatsApp Group Config (`config/group-config.example.md`)

If using the WhatsApp channel plugin, this file configures Kai's personality, tone, and behavior for your group chat. Copy it to your WhatsApp channel's group config directory.

## How It Works

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full architecture overview.

**In short:**
1. **You** send messages to your WhatsApp group (meal photos, workout reports, questions)
2. **Claude Code** (with the WhatsApp plugin) receives the message and reads the group config
3. **Kai's personality** (defined in the group config) processes the message
4. **kai-cli.py** logs data to / queries from a local SQLite database
5. **Cron jobs** run twice daily, fetch your status, and send personalized reminders

## Adding Exercises

Edit `src/exercises.md` to add your own exercises. The format is:

```markdown
## Muscle Group Name
### Equipment: Equipment Type
- Exercise Name
- Another Exercise
```

Equipment types used by the suggestion engine: `Barbell`, `Dumbbell`, `Cable`, `Machine`, `Bodyweight`.

## Contributing

Contributions are welcome! Here are some ideas:

- **New exercises** -- Add exercises to `src/exercises.md`
- **New commands** -- Add CLI commands for new tracking features
- **Localization** -- Add more language support
- **Integrations** -- Connect to fitness APIs, smartwatches, etc.
- **Charts/visualization** -- Generate progress charts

### Development

```bash
# Clone and set up
git clone https://github.com/moltbot0912/kai-fitness.git
cd kai-fitness
./setup.sh

# Run the CLI
python3 src/kai-cli.py --help

# Run tests (if any)
python3 -m pytest tests/
```

## License

MIT License. See [LICENSE](LICENSE).
