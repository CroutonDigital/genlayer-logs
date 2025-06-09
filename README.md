# 🔄 Daily systemd Logs Backup Script

This project contains a Bash script that extracts daily logs from a specified systemd service and automatically backs them up to this GitHub repository. It is designed for system administrators, developers, and DevOps teams who want a simple, auditable way to preserve logs over time.

---

## 🚀 Features

- ✅ Extracts logs from any systemd unit using `journalctl`
- ✅ Saves logs to timestamped `.log` files
- ✅ Skips empty logs automatically
- ✅ Commits new logs to Git
- ✅ Pushes changes to the `main` branch of a GitHub repository

---

## 📜 Script: `save_logs.sh`

Copy and paste this into a file using `nano save_logs.sh`:

```bash
#!/bin/bash

# === Configuration ===
UNIT_NAME="your_service_name"             # ← Replace with your systemd service name
REPO_DIR="$HOME/github-log-backups"      # ← Local path to your cloned GitHub repo
LOG_DIR="$REPO_DIR/logs"

# Ensure the log directory exists
mkdir -p "$LOG_DIR"
cd "$REPO_DIR" || exit 1

# === Timestamps ===
NOW_HUMAN=$(date "+%Y-%m-%d 00:00:00")
YESTERDAY_HUMAN=$(date -d "1 day ago" "+%Y-%m-%d 00:00:00")
NOW_FILENAME=$(date "+%Y-%m-%d")
YESTERDAY_FILENAME=$(date -d "1 day ago" "+%Y-%m-%d")

# === Log file path ===
LOG_FILE="$LOG_DIR/${UNIT_NAME}_logs_${YESTERDAY_FILENAME}_to_${NOW_FILENAME}.log"
echo "📄 Saving logs from $YESTERDAY_HUMAN to $NOW_HUMAN into $LOG_FILE"

# === Collect logs ===
sudo journalctl -u "$UNIT_NAME" --since "$YESTERDAY_HUMAN" --until "$NOW_HUMAN" --no-pager -o short-iso > "$LOG_FILE"

# === Skip if log file is empty ===
if [ ! -s "$LOG_FILE" ]; then
  echo "⚠️ Log file is empty. Removing it."
  rm -f "$LOG_FILE"
  exit 0
fi

# === Git: Commit and Push ===
echo "✅ Logs saved. Committing to GitHub."

# Set Git identity if needed (optional for automation)
git config pull.rebase true
git config user.name "log-bot"
git config user.email "log-bot@example.com"

git add "$LOG_FILE"
git commit -m "Add ${UNIT_NAME} logs: ${YESTERDAY_FILENAME} to ${NOW_FILENAME}"
git pull --rebase origin main
git push origin main
```

---

## 📦 Installation & Setup

### 1. Create a GitHub repository

Create a **private GitHub repository** named for example: `log-backups`. Do **not** initialize it with a README.

---

### 2. Clone the repository using SSH

> ⚠️ Make sure your GitHub account has an SSH key added, and you're using the `git@github.com:...` SSH format.

```bash
git clone git@github.com:yourusername/log-backups.git ~/github-log-backups
cd ~/github-log-backups
```

---

### 3. Make the script executable

```bash
chmod +x save_logs.sh
```

---

### 4. Edit the configuration in `save_logs.sh`

Update these variables at the top of the script:

```bash
UNIT_NAME="your_service_name"         # e.g. nginx, docker, your-custom-service
REPO_DIR="$HOME/github-log-backups"  # path to this repo
```

---

### 5. Add your SSH key to GitHub (if not already)

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

Ensure your SSH public key is [added to GitHub](https://github.com/settings/keys).

---

## 🛠 How It Works

1. The script uses `journalctl` to pull logs from the past 24 hours.
2. It saves them to a file named like:
   ```
   logs/your_service_name_logs_2025-06-08_to_2025-06-09.log
   ```
3. If the log is not empty, it commits the file to Git and pushes it to GitHub.

---

## ⏰ Automate with Cron

To run the script every day at midnight:

1. Open crontab:

```bash
crontab -e
```

2. Add this line:

```bash
0 0 * * * /bin/bash $HOME/github-log-backups/save_logs.sh >> /var/log/save_logs.log 2>&1
```

This will run the script daily and write any output to `/var/log/save_logs.log`.

---

## 🔐 Optional: Avoid sudo in journalctl

If you want to run the script without `sudo`:

```bash
sudo usermod -aG systemd-journal $USER
```

Then log out and log back in.

---

## 📁 Output Example

```
logs/
├── nginx_logs_2025-06-08_to_2025-06-09.log
├── nginx_logs_2025-06-09_to_2025-06-10.log
```

---
