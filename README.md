# Homelab — Ubuntu Server Backup, Storage & Restore (BSR) Strategy

> **Language / Taal:** [English](README.md) | [Nederlands](README.nl.md)

## Goal
Implement a complete Back-up, Storage, and Restore (BSR) lifecycle on an Ubuntu Server by adding dedicated storage, writing an automated backup script, scheduling execution via cron, and validating data recovery via a restore test.

- **Level:** Junior Systems / BSR Admin Fundamentals
- **Platform:** VirtualBox (local homelab)
- **Target OS:** Ubuntu Server 26.04 LTS

---

## BSR Architecture Overview

```text
+-----------------------------------------------------------------+
|                        Ubuntu Server VM                         |
|                                                                 |
|  [ System Disk: /dev/sda ]        [ Backup Disk: /dev/sdb ]     |
|  - Root filesystem (/)            - Formatted: ext4             |
|  - /etc (Config files)            - Mountpoint: /backup         |
|             |                                   ^               |
|             |--- Cron Job (Daily @ 02:00) ------|               |
|                  Executes backup.sh                             |
+-----------------------------------------------------------------+

Step 1 - Storage Provisioning & Persistent Mounting
Action

Added a dedicated 10 GB virtual disk (/dev/sdb), formatted it with the ext4 filesystem, created a mount point at /backup, and configured persistent mounting upon system reboot.
Commands
Bash

# Format disk with ext4 filesystem
sudo mkfs.ext4 /dev/sdb

# Create mount directory and mount disk
sudo mkdir /backup
sudo mount /dev/sdb /backup

# Configure persistent mount in /etc/fstab
sudo nano /etc/fstab

Added line to /etc/fstab:
Plaintext

/dev/sdb    /backup    ext4    defaults    0    2

Verification

Ran sudo mount -a and verified output using lsblk:
Plaintext

sdb          8:16   0   10G  0 disk /backup

Step 2 - Automated Backup Script
Action

Created a Bash script located at /usr/local/bin/backup.sh that generates a timestamped, gzip-compressed tarball archive of critical system configuration files (/etc) and stores it directly on the secondary storage drive (/backup).
Script Content (/usr/local/bin/backup.sh)
Bash

#!/bin/bash
# BSR Backup Script - Backup /etc to /backup

DATUM=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_DIR="/backup"

# Create a compressed archive
tar -czf $BACKUP_DIR/etc_backup_$DATUM.tar.gz /etc 2>/dev/null

echo "Backup completed successfully on $DATUM"

Permissions & Manual Test
Bash

sudo chmod +x /usr/local/bin/backup.sh
sudo /usr/local/bin/backup.sh

Step 3 - Job Scheduling via Cron (Operations Management)
Action

Configured a system cron job under the root user to automate the backup execution every night at 02:00 AM.
Commands
Bash

sudo crontab -e

Cron schedule entry:
Plaintext

0 2 * * * /usr/local/bin/backup.sh

Step 4 - Disaster Recovery & Restore Validation
Action

Validated the integrity of the backup by extracting the archived configurations into an isolated sandbox directory (/tmp/restore_test) and inspecting the restored file tree.
Commands
Bash

# Create sandbox restore location
mkdir /tmp/restore_test

# Extract archive to restore location
sudo tar -xzf /backup/etc_backup_<TIMESTAMP>.tar.gz -C /tmp/restore_test

# Inspect restored files
ls -l /tmp/restore_test/etc

Verification

Successfully verified that system configurations (e.g., ssh/, sudoers, cron.d/) were extracted and intact without corruptions.

Key Technical Skills Demonstrated

- Storage Management: Partitioning, ext4 formatting, mounting, and /etc/fstab persistent configuration.
- Data Protection: Shell scripting using tar and gzip for compressed system backups.
- Automation: Task scheduling with cron.
- Disaster Recovery: Extraction, verification, and sandbox restore testing.
