# Homelab - Ubuntu Server Backup, Storage & Restore (BSR)

> **Taal / Language:** [Nederlands](README.nl.md) | [English](README.md)

## Doel
Het implementeren van een complete Back-up, Opslag en Herstel (BSR) cyclus op een Ubuntu Server door het toevoegen van extra opslag, het schrijven van een geautomatiseerd back-upscript, het inplannen via cron, en het controleren van de herstelprocedure via een restore-test.

- **Niveau:** Junior Systems / BSR Admin Fundamentals
- **Platform:** VirtualBox (lokaal homelab)
- **Doel OS:** Ubuntu Server 26.04 LTS

---

## BSR Architectuur Overzicht

```text
+-----------------------------------------------------------------+
|                        Ubuntu Server VM                         |
|                                                                 |
|  [ Systeemschijf: /dev/sda ]      [ Back-upschijf: /dev/sdb ]   |
|  - Root bestandssysteem (/)       - Geformatteerd: ext4         |
|  - /etc (Configuraties)           - Mountpoint: /backup         |
|             |                                   ^               |
|             |--- Cron Job (Dagelijks 02:00) ----|               |
|                  Voert backup.sh uit                            |
+-----------------------------------------------------------------+

Stap 1 - Opslag Inrichten & Permanent Koppelen
Actie

Een extra virtuele schijf van 10 GB (/dev/sdb) geformatteerd met het ext4 bestandssysteem, een koppelpunt aangemaakt op /backup, en automatische koppeling ingesteld bij het opstarten.
Commando's
Bash

# Formatteer schijf met ext4 bestandssysteem
sudo mkfs.ext4 /dev/sdb

# Maak koppelpunt aan en koppel de schijf
sudo mkdir /backup
sudo mount /dev/sdb /backup

# Configureer permanente koppeling in /etc/fstab
sudo nano /etc/fstab

Toegevoegde regel aan /etc/fstab:
Plaintext

/dev/sdb    /backup    ext4    defaults    0    2

Verificatie

sudo mount -a uitgevoerd en de uitvoer gecontroleerd met lsblk:
Plaintext

sdb          8:16   0   10G  0 disk /backup

Stap 2 - Geautomatiseerd Back-upscript
Actie

Een Bash-script aangemaakt op /usr/local/bin/backup.sh dat een gecomprimeerd .tar.gz archief met datumstempel maakt van kritieke configuratiebestanden (/etc) en dit rechtstreeks opslaat op de back-upschijf (/backup).
Inhoud van het script (/usr/local/bin/backup.sh)
Bash

#!/bin/bash
# BSR Back-upscript - Back-up van /etc naar /backup

DATUM=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_DIR="/backup"

# Maak een gecomprimeerd archief
tar -czf $BACKUP_DIR/etc_backup_$DATUM.tar.gz /etc 2>/dev/null

echo "Backup succesvol afgerond op $DATUM"

Rechten & Handmatige Test
Bash

sudo chmod +x /usr/local/bin/backup.sh
sudo /usr/local/bin/backup.sh

Stap 3 - Automatisering via Cron
Actie

Een automatische taak ingesteld onder de root-gebruiker om het back-upscript elke nacht om 02:00 uur uit te voeren.
Commando's
Bash

sudo crontab -e

Ingevoerde cron-regel:
Plaintext

0 2 * * * /usr/local/bin/backup.sh

Stap 4 - Herstelcontrole (Disaster Recovery)
Actie

De integriteit van de back-up gecontroleerd door het archief uit te pakken in een tijdelijke testmap (/tmp/restore_test) en de herstelde bestanden te inspecteren.
Commando's
Bash

# Maak tijdelijke testmap aan
mkdir /tmp/restore_test

# Pak archief uit naar testmap
sudo tar -xzf /backup/etc_backup_<TIMESTAMP>.tar.gz -C /tmp/restore_test

# Controleer herstelde bestanden
ls -l /tmp/restore_test/etc

Verificatie

Succesvol gecontroleerd dat systeemconfiguraties (zoals ssh/, sudoers, cron.d/) intact en zonder fouten zijn hersteld.

Demonstratie van Vaardigheden

- Opslagbeheer: Partitioneren, ext4-formatteren, koppelen en /etc/fstab configuratie.
- Databeveiliging: Shell scripting met tar en gzip voor gecomprimeerde back-ups.
- Automatisering: Taken inplannen met cron.
- Disaster Recovery: Uitpakken, controleren en herstel testen in een testomgeving.
