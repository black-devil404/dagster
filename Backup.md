# 🚀 Remote Backup Automation with Dagster + SSH + Rsync

This project automates backup transfer using **Dagster**, **rsync**, and **SSH key authentication** between three different machines. The flow includes daily `.tar` file creation via cron, secure transfer via `rsync`, and orchestration via Dagster running on a centralized host machine.

## 🧱 System Architecture

| Role             | IP Address     | Username     | Description                                 |
|------------------|----------------|--------------|---------------------------------------------|
| Dagster Host     | 10.77.1.38     | root         | Runs Dagster job to initiate remote backup  |
| Target Machine   | 10.77.2.143    | srikantha    | Has data; creates `.tar` backup file        |
| Backup Machine   | 10.77.1.212    | test         | Receives backup file with timestamp         |

## 🔁 Flow Overview

1. Target machine creates `.tar` backup file via cron job.
2. Dagster job running on 10.77.1.38 SSHs into target machine and triggers a script.
3. The script uses `rsync` to copy the `.tar` file to the backup machine.
4. Each backup is saved in a timestamped format like `backup_2025-08-01_02-00.tar.gz`.
5. On success or failure, an email notification is sent.

## 📂 Directory Structure on Dagster Host

```
dagster_prod/
├── dagster_prod/
│   ├── repository.py
│   └── scripts/
│       └── backup.sh
└── README.md
```

## 🔐 SSH Key Setup

### 1. From Dagster ➝ Target

```bash
ssh-keygen -t rsa -f ~/.ssh/id_dagster_to_target
ssh-copy-id -i ~/.ssh/id_dagster_to_target.pub srikantha@10.77.2.143
```

### 2. From Target ➝ Backup

```bash
ssh-keygen -t rsa -f ~/.ssh/id_target_to_backup
ssh-copy-id -i ~/.ssh/id_target_to_backup.pub test@10.77.1.212
```

### Permissions

```bash
chmod 600 ~/.ssh/id_*
chmod 700 ~/.ssh
```

## 🕒 Cron Job on Target Machine

Edit crontab:

```bash
crontab -e
```

Add:

```cron
0 2 * * * tar -cf /home/srikantha/Backup/Test_bkp.tar /home/srikantha/important_data/
```

## 📝 Dagster Job (repository.py)

```python
from dagster import op, job, schedule, repository
import subprocess

@op
def run_backup_script():
    result = subprocess.run(
        ["/bin/bash", "/root/dagster_prod/dagster_prod/scripts/backup.sh"],
        capture_output=True,
        text=True
    )
    if result.returncode != 0:
        raise Exception(f"Backup failed: {result.stderr}")
    return result.stdout

@job
def backup_job():
    run_backup_script()

@schedule(cron_schedule="30 2 * * *", job=backup_job, execution_timezone="Asia/Kolkata")
def backup_schedule(_context):
    return {}

@repository
def my_repo():
    return [backup_job, backup_schedule]
```

## 🧠 What `backup.sh` Does

- SSH into target machine (`srikantha@10.77.2.143`)
- Run `rsync` to send `/home/srikantha/Backup/Test_bkp.tar` to `/home/test/backups/` on backup machine
- Add timestamp to the backup filename
- Send email alert based on success or failure

## 📤 Email Setup (Optional)

```bash
sudo apt install mailutils
echo "Test" | mail -s "Test Email" your@email.com
```

Make sure your `backup.sh` script has valid email ID configured in `EMAIL_TO`.

## 📁 Directories

### On Target Machine:

```bash
/home/srikantha/Backup/Test_bkp.tar
```

### On Backup Machine:

```bash
/home/test/backups/
```

## ✅ Checklist

- [x] SSH keys from Dagster ➝ Target and Target ➝ Backup
- [x] Cron job creates `.tar` daily
- [x] `backup.sh` present on Dagster host
- [x] Dagster job and schedule configured
- [x] Email optional but tested
- [x] Manual job run successful

## 🧪 Manual Test

```bash
dagster job execute -j backup_job
```

## 👨‍💻 Maintained By

**Srikanth A**  
Assistant System Analyst – Vehant Technologies  
Email: srikantha@vehant.com

