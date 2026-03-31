# AWS / Cloud VM Deployment Guide

Run Kai Fitness 24/7 on a cloud VM for about $3-5/month. This ensures your automated WhatsApp reminders fire reliably even when your laptop is off.

## Option 1: AWS Lightsail (Recommended)

### 1. Create Instance

1. Go to [AWS Lightsail](https://lightsail.aws.amazon.com/)
2. Click "Create instance"
3. Choose **Linux/Unix** > **OS Only** > **Ubuntu 22.04 LTS**
4. Select the **$3.50/month** plan (512 MB RAM, 1 vCPU)
5. Name it `kai-fitness` and create

### 2. Connect via SSH

```bash
# From the Lightsail console, click "Connect using SSH"
# Or use your own terminal:
ssh -i your-key.pem ubuntu@your-instance-ip
```

### 3. Install Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python 3
sudo apt install -y python3 python3-pip

# Install Node.js (required for Claude Code)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Install Claude Code
npm install -g @anthropic-ai/claude-code

# Authenticate Claude Code
claude auth login
```

### 4. Install Kai Fitness

```bash
git clone https://github.com/moltbot0912/kai-fitness.git
cd kai-fitness
chmod +x setup.sh
./setup.sh
```

### 5. Configure WhatsApp

1. Set up the WhatsApp channel plugin for Claude Code
2. Pair your WhatsApp device
3. Set `KAI_WHATSAPP_CHAT_ID` in `.env`

### 6. Install Cron Jobs

```bash
./cron/install-cron.sh
```

### 7. Verify

```bash
# Check cron is set up
crontab -l

# Run a test
python3 src/kai-cli.py quick-status

# Check logs after a cron run
tail -f cron.log
```

## Option 2: Any VPS Provider

The setup is the same on any Linux VPS. Popular alternatives:
- **DigitalOcean Droplet** -- $4/month (Basic plan)
- **Vultr** -- $3.50/month
- **Hetzner** -- EUR 3.29/month
- **Oracle Cloud** -- Free tier (Always Free ARM instance)

## Tips

### Keep It Running

The VM's cron daemon runs automatically. No need for screen/tmux for the cron jobs.

If you want to interact with Kai directly via SSH:
```bash
cd kai-fitness
python3 src/kai-cli.py suggest-workout
```

### Data Backup

Your data lives in `data/kai_health.db`. Back it up periodically:

```bash
# Simple backup
cp data/kai_health.db data/kai_health.db.backup

# Or set up a daily backup cron
echo "0 2 * * * cp /home/ubuntu/kai-fitness/data/kai_health.db /home/ubuntu/kai-fitness/data/kai_health.db.backup" | crontab -
```

### Monitoring

Check that cron jobs are running:
```bash
# View recent log entries
tail -20 cron.log

# Check if cron ran today
grep "$(date +%Y-%m-%d)" cron.log
```

### Updating

```bash
cd kai-fitness
git pull
# Your data and config files are in .gitignore, so they won't be overwritten
```

### Costs

| Provider | Plan | Monthly Cost |
|---|---|---|
| AWS Lightsail | 512 MB / 1 vCPU | $3.50 |
| DigitalOcean | Basic Droplet | $4.00 |
| Vultr | Cloud Compute | $3.50 |
| Oracle Cloud | Always Free ARM | $0.00 |
