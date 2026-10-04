---
layout: default
title: VM – Proxmox Backup Server
---
# Proxmox VE und Proxmox Backup Server

**Proxmox-VM-Konfiguration**

|Parameter|Wert|
|---|---|
|VM-ID|211|
|SSH-Nickname|pbs|
|VM-Name|PBS-PBS-20-10|
|Zweck|Backup Server|
|Betriebssystem|Linux-PBS|
|IPv4|10.10.20.10|
|Größe|20 GB|
|RAM|8 GB|
|vCPU|4|

## 1. Überblick

Die Backup-Lösung besteht aus dem Proxmox-VE-Host, einer virtuellen
Maschine mit Proxmox Backup Server (PBS) und einem externen Datenträger.

Beim Start von Proxmox VE startet ein **systemd-Dienst** das Backup-Skript.
Dieses wartet, bis Netzwerk, PBS und Speicher bereit sind. Danach werden
die vorgesehenen virtuellen Maschinen gesichert. **Nur wenn alle Backups
erfolgreich sind, wird die PBS-VM automatisch heruntergefahren.** Bei
Fehlern bleibt sie eingeschaltet und das Skript verschickt eine
Status-E-Mail.

**Ablauf:** Proxmox-VMs → PVE-Speicher **Extern-Backup** → PBS-Datastore
**Backup** → externer Datenträger.

## 1. Logische Darstellung des Prozesses

![Vollständiger Ablauf der Proxmox-Backup-Lösung](/images/proxmox-backup-ablauf.png)

PVE bleibt danach eingeschaltet, bis es manuell regulär
heruntergefahren wird. Die Festplatte bleibt angeschlossen und eingeschaltet.
## 2. Festgehaltene Systemdaten

| Komponente            | Wert                       |
| --------------------- | -------------------------- |
| PVE-Host              | prox                       |
| PBS-VM                | ID 211, Name PBS-PBS-20-10 |
| PBS-Adresse           | 10.10.20.10                |
| PBS-Weboberfläche     | https: //10.10.20.10:8007  |
| PBS-Datastore         | Backup                     |
| Datastore-Pfad in PBS | /mnt/datastore/Backup      |
| PVE-Storage-Name      | Extern-Backup              |
| Datastore-Dateisystem | ext4                       |
| Physische Festplatte  | LaCie USB-Festplatte       |

## 3. PBS und Datenträger

PBS läuft als virtuelle Maschine auf PVE. Die VM-Konfiguration lässt
sich mit **qm config 211** ansehen; **qm status 211** zeigt, ob sie läuft.

Der vorgesehene Datenträger wird über seine **UUID** an PBS durchgereicht.

In PBS wird die Datenträgerpartition unter */mnt/datastore/Backup*
eingebunden und als Datastore **Backup** verwendet. Der Datastore wird in
der PBS-Weboberfläche verwaltet.
## 4. Verbindung zwischen PVE und PBS

PVE verwendet die Speicherdefinition **Extern-Backup**, um Sicherungen
an PBS zu senden. Der Eintrag wird in der PVE-Weboberfläche unter
**Datacenter → Storage** eingerichtet. Dabei werden PBS-Server,
Datastore Backup, Zugangsdaten und der Fingerprint angegeben.

Für den Zugriff wurde ein PBS-API-Token verwendet beziehungsweise
getestet. Die Token-Verwaltung befindet sich in PBS unter
**Configuration → Access Control → API Tokens**; die Rechte werden unter
**Permissions** verwaltet.
## 5. API - Token

- **Authentifizierung:** Proxmox VE weist sich beim PBS mit dem Token aus.
- **Zugriff:** Der PBS prüft, ob der Token die nötigen Berechtigungen hat, zum Beispiel um den Backup-Datastore abzufragen.
- **Automatisierung:** Proxmox kann dadurch ohne manuelle Passworteingabe mit dem PBS kommunizieren.

Das Token-Geheimnis wird auf PVE in einer geschützten Datei abgelegt:
*/root/.config/pbs-backup/token*
Der Ordner soll nur für root zugänglich sein (Modus 700), die Datei
selbst mit Modus 600.

Ein direkter API-Test wurde mit curl durchgeführt:
	`TOKEN=$(cat /root/.config/pbs-backup/token)`
	`curl -k -sS -o /tmp/pbs-api-test.json -w 'HTTP-Status: %{http_code}\n' -H "Authorization: PBSAPIToken=root@pam!check-backup:$TOKEN" https://10.10.20.10:8007/api2/json/admin/datastore/Backup/status`

Der Aufruf liest den Token aus der geschützten Datei und fragt den
Status des Datastores ab. **Ein erfolgreicher API-Test beweist nicht
automatisch, dass der PVE-Speichereintrag dieselben Zugangsdaten
verwendet.**

Der PVE-Speicher lässt sich mit diesen Befehlen prüfen:
-   `pvesm status` -- Status der konfigurierten Speicher.
-   `pvesm list Extern-Backup` -- Sicherungen, die über diesen Speicher
    sichtbar sind.
-   `cat /etc/pve/storage.cfg` -- konfigurierte Speicherdefinitionen
    anzeigen.
## 6. SSH-Prüfung des Datenträgers
Zusätzlich zum API-Zugriff nutzt das Backup-Skript SSH, um eine Prüfung
auf PBS auszuführen. Dafür wird ein eigener SSH-Schlüssel verwendet; er
ist unabhängig vom API-Token.

Der Schlüssel liegt auf PVE unter `/root/.ssh/pbs-disk-check`. Der
Erzeugungsbefehl lautet:
	`ssh-keygen -t ed25519 -f /root/.ssh/pbs-disk-check -N ""`

Der öffentliche Schlüssel (`/root/.ssh/pbs-disk-check.pub`) wird auf PBS
für den vorgesehenen Benutzer hinterlegt. Der private Schlüssel bleibt
auf PVE.

Der Verbindungstest verwendet:
	`ssh -i /root/.ssh/pbs-disk-check -o BatchMode=yes -o IdentitiesOnly=yes -o ConnectTimeout=10 -o StrictHostKeyChecking=yes root@10.10.20.10`

Das Skript erwartet für die Datenträgerprüfung die Rückmeldung OK. Der
genaue Remote-Prüfbefehl ist in den verfügbaren Notizen nicht
vollständig festgehalten.
## 7. Backup-Skript und automatischer Start
Das Skript `/usr/local/sbin/backup-on-boot.sh` führt die Schritte nach
dem Systemstart aus:

1.  Es wartet auf die Erreichbarkeit von OPNsense unter `192.168.178.2`.
2.  Es wartet, bis die PBS-VM 211 läuft und die PBS-API erreichbar ist.
3.  Es prüft, ob `Extern-Backup` verfügbar ist.
4.  Es prüft den Datenträger über SSH.
5.  Es sichert alle VMs außer PBS - 211.
6.  Es wertet die Ergebnisse aus und sendet eine Status-E-Mail.
7.  **Nur bei vollständig erfolgreichen Backups** fährt es PBS herunter.

Der Backup-Aufruf verwendet:
	`vzdump --storage Extern-Backup --mode snapshot --compress zstd`

**vzdump** erstellt die Sicherung,
**--storage** legt das Ziel fest,
**--mode snapshot** wählt den Snapshot-Modus
**--compress zstd** aktiviert die Komprimierung.

Der systemd-Dienst */etc/systemd/system/backup-on-boot.service* startet
das Skript automatisch beim Booten. Er ist als einmaliger Dienst
(**oneshot**) eingerichtet und wird nach **pve-guests.service** gestartet.
## 8. Kontrolle und Fehlersuche
Die wichtigsten Befehle auf dem PVE-Host sind:

-   `systemctl status backup-on-boot.service` -- Status des
    Backup-Dienstes.
-   `journalctl -u backup-on-boot.service -b --no-pager` -- Meldungen
    des Dienstes seit dem letzten Systemstart.
-   `pvesm status` -- prüfen, ob `Extern-Backup` verfügbar ist.
-   `pvesm list Extern-Backup` -- sichtbare Sicherungen anzeigen.

**Wichtig:** In der PBS-Weboberfläche können die Sicherungen im Datastore **Backup**
angesehen und mit **Verify** auf Integrität geprüft werden. **Not
verified** bedeutet, dass noch kein Verify-Lauf dokumentiert ist; es
bedeutet nicht automatisch, dass die Sicherung fehlerhaft ist. 
Eine automatische Verifizierung wurde jede 5 Tage nach der Backup in der WebGUI von PBS eingerichtet.


## Backup-Skript

Das Skript wird beim Start des Proxmox-Hosts automatisch ausgeführt. Es prüft die Voraussetzungen, sichert die VMs und fährt den PBS nur bei erfolgreichem Abschluss herunter.

<details markdown="1">
<summary><strong>Backup-Skript anzeigen</strong></summary>

```bash
#!/usr/bin/env bash

set -u
set -o pipefail


# Backup-Ergebnis per E-Mail melden, auch bei vorzeitigem Abbruch
notify_backup_result() {
    rc=$?
    trap - EXIT

    if [ "$rc" -eq 0 ]; then
        status="ERFOLGREICH"
    else
        status="FEHLER"
    fi

    {
        echo "Proxmox-Backup: $status"
        echo "Host: $(hostname)"
        echo "Zeit: $(date)"
        echo "Exit-Code: $rc"
        echo
        echo "Backup-Protokoll:"
        journalctl -u backup-on-boot.service -b --no-pager
    } | mail -s "Proxmox Backup $status auf $(hostname)" blablabla@gmail.com

    mail_rc=$?
    if [ "$mail_rc" -ne 0 ]; then
        logger -t backup-on-boot "E-Mail-Versand fehlgeschlagen (Exit-Code $mail_rc)"
    fi

    exit "$rc"
}
trap notify_backup_result EXIT

PBS_VMID=211
PBS_STORAGE="Extern-Backup"
PBS_URL="https://10.10.20.10:8007"
PBS_SSH_KEY="/root/.ssh/pbs-disk-check"
PBS_SSH_HOST="root@10.10.20.10"
OPNSENSE_IP="192.168.178.2"

echo "=== PBS-Backup gestartet: $(date) ==="

# 1. Auf OPNsense warten
echo "Warte auf OPNsense ($OPNSENSE_IP) ..."

FIREWALL_READY=0

for i in $(seq 1 30); do
    if ping -c 1 -W 2 "$OPNSENSE_IP" >/dev/null 2>&1; then
        FIREWALL_READY=1
        echo "OPNsense ist erreichbar."
        break
    fi
    sleep 10
done

if [ "$FIREWALL_READY" -ne 1 ]; then
    echo "FEHLER: OPNsense ist nicht erreichbar."
    echo "Es werden keine Backups gestartet."
    exit 1
fi

# 2. Warten, bis PBS laeuft - PBS wird nicht selbst gestartet
echo "Warte bis zu 10 Minuten auf den Start der PBS-VM ..."

PBS_RUNNING=0

for i in $(seq 1 120); do
    if qm status "$PBS_VMID" | grep -q running; then
        PBS_RUNNING=1
        echo "PBS-VM $PBS_VMID laeuft."
        break
    fi
    sleep 5
done

if [ "$PBS_RUNNING" -ne 1 ]; then
    echo "FEHLER: PBS-VM $PBS_VMID ist nicht rechtzeitig gestartet."
    echo "PBS wird nicht gestartet. Keine Backups."
    exit 1
fi

# 3. Auf PBS-API und Storage warten
echo "Warte auf PBS-API und Storage ..."

READY=0

for i in $(seq 1 120); do
    if curl -kfsS --connect-timeout 3 "$PBS_URL/" >/dev/null 2>&1 &&
       pvesm status | awk \
       '$1=="Extern-Backup" && $3=="active" {found=1}
        END {exit !found}'; then
        READY=1
        break
    fi
    sleep 5
done

if [ "$READY" -ne 1 ]; then
    echo "FEHLER: PBS-API oder Storage nicht erreichbar."
    echo "PBS bleibt eingeschaltet."
    exit 1
fi

# 4. Externe Backup-Festplatte auf PBS pruefen
echo "Pruefe externe Backup-Festplatte ..."

if ! DISK_CHECK=$(ssh \
    -T \
    -i "$PBS_SSH_KEY" \
    -o BatchMode=yes \
    -o IdentitiesOnly=yes \
    -o ConnectTimeout=10 \
    -o StrictHostKeyChecking=yes \
    -o LogLevel=ERROR \
    "$PBS_SSH_HOST" 2>/dev/null); then
    echo "FEHLER: SSH-Festplattenpruefung fehlgeschlagen."
    echo "PBS bleibt eingeschaltet."
    exit 1
fi

if [ "$DISK_CHECK" != "OK" ]; then
    echo "FEHLER: Die erwartete Backup-Festplatte wurde nicht bestaetigt."
    echo "Rueckgabe: $DISK_CHECK"
    echo "PBS bleibt eingeschaltet."
    exit 1
fi

echo "Backup-Festplatte erfolgreich geprueft."

# 5. Storage-Status nochmals kontrollieren
if ! pvesm status | awk \
    '$1=="Extern-Backup" && $3=="active" {found=1}
     END {exit !found}'; then
    echo "FEHLER: PBS-Storage ist nicht aktiv."
    echo "PBS bleibt eingeschaltet."
    exit 1
fi

# 6. Alle QEMU-VMs ermitteln, PBS ausschliessen
mapfile -t VMIDS < <(
    qm list | awk 'NR > 1 && $1 ~ /^[0-9]+$/ && $1 != 211 {print $1}'
)

if [ "${#VMIDS[@]}" -eq 0 ]; then
    echo "FEHLER: Keine zu sichernden VMs gefunden."
    echo "PBS bleibt eingeschaltet."
    exit 1
fi

echo "Zu sichernde VM-IDs: ${VMIDS[*]}"

# 7. Jede VM einzeln sichern
FAILED=0

for VMID in "${VMIDS[@]}"; do
    echo "=== Sichere VM $VMID: $(date) ==="

    if vzdump "$VMID" \
        --storage "$PBS_STORAGE" \
        --mode snapshot \
        --compress zstd; then
        echo "VM $VMID erfolgreich gesichert."
    else
        echo "FEHLER: Backup von VM $VMID fehlgeschlagen."
        FAILED=1
    fi
done

# 8. Bei Fehlern PBS eingeschaltet lassen
if [ "$FAILED" -ne 0 ]; then
    echo "FEHLER: Mindestens ein Backup ist fehlgeschlagen."
    echo "PBS bleibt zur Untersuchung eingeschaltet."
    exit 1
fi

echo "Alle Backups erfolgreich."

# 9. PBS sauber herunterfahren
echo "Fahre PBS herunter ..."

if ! qm shutdown "$PBS_VMID" --timeout 180; then
    echo "FEHLER: Herunterfahren von PBS fehlgeschlagen."
    echo "Bitte PBS manuell pruefen."
    exit 1
fi

for i in $(seq 1 36); do
    if qm status "$PBS_VMID" | grep -q stopped; then
        echo "PBS wurde erfolgreich heruntergefahren."
        echo "=== Backup-Ablauf beendet: $(date) ==="
        exit 0
    fi
    sleep 5
done

echo "FEHLER: PBS ist nach dem Shutdown-Zeitlimit noch nicht gestoppt."
exit 1
```

</details>




