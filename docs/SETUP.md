# Detailed Setup Guide

This guide walks through every step to get Kai Fitness running.

## Prerequisites

### Python 3.8+

Check your version:
```bash
python3 --version
```

If you don't have Python 3.8+:
- **macOS**: `brew install python3`
- **Ubuntu/Debian**: `sudo apt install python3`
- **Windows**: Download from [python.org](https://www.python.org/downloads/)

### Claude Code CLI

Kai's WhatsApp integration requires Claude Code. Install it following the [official guide](https://docs.anthropic.com/en/docs/claude-code/getting-started).

```bash
# Verify installation
claude --version
```

### WhatsApp Channel Plugin

The WhatsApp integration uses Claude Code's channel plugin system. Set it up following the plugin documentation.

## Step-by-Step Setup

### 1. Clone the Repository

```bash
git clone https://github.com/moltbot0912/kai-fitness.git
cd kai-fitness
```

### 2. Run the Setup Script

```bash
chmod +x setup.sh
./setup.sh
```

This creates:
- `.env` -- Environment configuration
- `config/profile.json` -- Your fitness profile
- `data/kai_health.db` -- SQLite database

### 3. Configure Your Profile

Edit `config/profile.json`:

```json
{
  "user_name": "YourName",
  "fitness_goals": {
    "primary_goal": "Build muscle and increase strength",
    "current_stats": {
      "height_cm": 175,
      "weight_kg": 70
    },
    "target_weight_kg": 75
  },
  "nutrition_targets": {
    "calories_per_day": 2200,
    "protein_g_per_day": 120
  },
  "workout_preferences": {
    "frequency_per_week": "3-4",
    "gym_locations": {
      "My Gym": {
        "equipment": ["Dumbbells", "Barbell", "Bench"],
        "categories": ["Barbell", "Dumbbell", "Bodyweight"]
      }
    }
  }
}
```

**Equipment categories** determine which exercises get suggested:
- `Barbell` -- Barbell exercises (bench press, squats, rows)
- `Dumbbell` -- Dumbbell exercises (curls, presses, flies)
- `Cable` -- Cable machine exercises (crossovers, pushdowns)
- `Machine` -- Gym machines (lat pulldown, leg press)
- `Bodyweight` -- No equipment needed (push-ups, pull-ups, planks)

### 4. Configure Environment

Edit `.env`:

```bash
# Required for WhatsApp reminders
KAI_WHATSAPP_CHAT_ID=your-group-id@g.us

# Optional
KAI_TIMEZONE=America/New_York
```

To find your WhatsApp group chat ID, check the WhatsApp channel plugin documentation.

### 5. Test the CLI

```bash
# Log a test weight
python3 src/kai-cli.py log-weight 70.0

# Check status
python3 src/kai-cli.py quick-status

# Get a workout suggestion
python3 src/kai-cli.py suggest-workout
```

### 6. Set Up WhatsApp Integration

1. Copy the group config to your WhatsApp channel's group directory:
   ```bash
   cp config/group-config.example.md /path/to/whatsapp-channel/groups/YOUR_GROUP_ID/config.md
   ```

2. Edit the config to update file paths to point to your kai-fitness installation.

3. Test by sending a message to your WhatsApp group.

### 7. Install Cron Jobs (Optional)

```bash
./cron/install-cron.sh
```

This installs two daily reminders:
- **10:00 AM** -- Morning check-in (sleep report, workout reminder)
- **7:30 PM** -- Evening check-in (nutrition/workout status)

Customize times by editing your crontab: `crontab -e`

## Troubleshooting

### "No module named 'db_manager'"
Make sure you're running from the project root or using the full path:
```bash
python3 /path/to/kai-fitness/src/kai-cli.py quick-status
```

### Database not found
Run the setup script again or manually create the database:
```bash
mkdir -p data
python3 src/db_manager.py data/kai_health.db
```

### Cron jobs not firing
1. Check cron is running: `crontab -l`
2. Check the log file: `cat cron.log`
3. Ensure full paths are used in the crontab entry
4. On macOS, grant cron Full Disk Access in System Settings > Privacy & Security

### WhatsApp messages not sending
1. Verify Claude Code is authenticated: `claude --version`
2. Check the WhatsApp plugin status
3. Verify `KAI_WHATSAPP_CHAT_ID` is correct in `.env`
