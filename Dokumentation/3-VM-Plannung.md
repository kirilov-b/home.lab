---
layout: default
title: VM-Plannung
---

# VM-Plannung

## Übersicht

Die folgende Tabelle zeigt die geplanten virtuellen Maschinen des Home Labs.

| VM | ID | IPv4 | Zweck |
|---|---:|---|---|
| JUMP-LM-10-3 | 100 | – | Administration / Dokumentation |
| FW-OPSense | 101 | – | Firewall |
| ANS-LU-10-20 | 102 | 10.10.10.20 | Ansible / Automation |
| DC-WS-20-1 | 201 | 10.10.20.1 | Domain Controller |
| PBS-PBS-20-10 | 211 | 10.10.20.10 | Proxmox Backup Server |
| IPA-LA-20-20 | 222 | 10.10.20.20 | FreeIPA / Identity / DNS |
| NGX-LA-20-30 | 233 | 10.10.20.30 | Nginx / Reverse Proxy |
| DB-LD-20-40 | 244 | 10.10.20.40 | PostgreSQL |
| MON-LA-20-50 | 255 | 10.10.20.50 | Zabbix Monitoring |
| DOC-LU-20-60 | 266 | 10.10.20.60 | Docker / Podman / Container |
| LOG-LA-20-70 | 277 | 10.10.20.70 | Zentrales Logging |
| SEC-LA-20-80 | 288 | 10.10.20.80 | Wazuh / Security Monitoring |
| GIT-LD-20-90 | 299 | 10.10.20.90 | Gitea / Git-Server |
| CL1-WC-30 | 301 | DHCP | Testclient |
| CL2-WC-30 | 302 | DHCP | Testclient |
| CL3-MAC-30 | 303 | DHCP | Testclient |

## Namensschema

Die Namen der VMs enthalten Informationen über Funktion, Betriebssystem bzw. Plattform und Netzwerkbereich.

Die VM-IDs entsprechen den IDs innerhalb der Proxmox-Umgebung.
