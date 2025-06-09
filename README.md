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

## 📦 Installation & Setup

### 1. Clone the repository

```bash
git clone git@github.com:yourusername/your-log-repo.git ~/github-log-backups
cd ~/github-log-backups
```

### 2. Make the script executable

```bash
chmod +x save_logs.sh
```

### 3. Edit the configuration in `save_logs.sh`

Update these variables at the top of the script:

```bash
UNIT_NAME="your_service_name"         # e.g. nginx, docker, your-custom-service
REPO_DIR="$HOME/github-log-backups"  # path to this repo
```

### 4. Add your SSH key to GitHub (if not already)

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
