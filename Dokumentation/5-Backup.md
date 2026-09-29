---
layout: default
title: Backup
---

# Backup

## Backup-Konzept

Für die Sicherung der virtuellen Maschinen wird ein zentraler **Proxmox Backup Server (PBS)** eingesetzt.

## Proxmox Backup Server

| Parameter | Wert |
|---|---|
| VM-ID | 211 |
| VM-Name | PBS-PBS-20-10 |
| IPv4 | 10.10.20.10 |
| Zweck | Proxmox Backup Server |
| Größe | 32 GB |
| RAM | 4 GB |
| vCPU | 2 |

## Ziel

Das Backup-System dient dazu, virtuelle Maschinen und deren Daten zentral zu sichern.

Dadurch können Systeme bei einem Fehler oder Ausfall wiederhergestellt werden.

## Wiederherstellung

Ein wichtiger Bestandteil des Backup-Konzepts ist die regelmäßige Überprüfung der Wiederherstellbarkeit.

Ein Backup gilt erst dann als praktisch nutzbar, wenn eine Wiederherstellung erfolgreich getestet wurde.
