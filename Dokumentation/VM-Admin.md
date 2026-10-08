---
layout: default
title: VM – Administrator
---

# VM – Verwaltung, Dokumentation, Remote Verbindung

**VM Parameter**

| Parameter      | Wert                      |
| -------------- | ------------------------- |
| VM-ID          | 100                       |
| SSH-Nickname   | jump                      |
| VM-Name        | jump-lm-10-3              |
| Zweck          | Verwaltung, Dokumentation |
| Betriebssystem | Linux Mint                |
| IPv4           | 10.10.10.3                |
| Domain         | lab.service               |
| Größe          | 80 GB                     |
| RAM            | 8 GB                      |
| vCPU           | 4                         |

# Konfiguration

<details markdown="1">
<summary><strong>Konfiguration anzeigen</strong></summary>
## RDP Verbindung
Remote Destkop Protocol **(RDP)** wird auf der Controller Laptop und auf dem jump-lm-10-3 (**JUMP**) installiert, um eine Verbindung zwischen beide Machinen ohne die Proxmox Browser Verbindung benutzen zu müssen.
Ziel ist der **jump** ein Jumphost zu machen und von dort aus die komplette Umgebung zu verwalten. Die **jump** VM befindet sich in VLAN 10 - Management Netz (**MGMT**). Über den OPNsens Firewall wird an der Managment Netz hat folgende Zugriff ermöglicht:
- Verbindung zu HTTPS: **Proxmox WebGUI**
- Verbindung zu **Service VLAN 20** (**SRV)**
- Verbindung zu **Clients VLAN 30** (CL)
- Verbindung zu **OPNsense Web GUI** 

Zusätzlich wird zu jede VM von **jump** aus eine SSH Verbindung eingerichtet.

## SSH Einrichtung

Auf **jump-lm-10-3**:

1. Ordner für den SHA256 Schlüssel erstellen
	 `mkdir -p ~/.ssh`
	 `chmod 700 ~/.ssh
2. Schlüssel generieren
	 `ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519`
	 Ein Passphrase (Password) setzen: laut Vorgaben
		Zwei keys werden erstellt:
		~/.ssh/id_ed25519 (geheim)
		~/.ssh/id_ed25519.pub
3. Schlüssel prüfen:
	`eval "$(ssh-agent -s)"
	 `ssh-add ~/.ssh/id_ed25519`
	 `ssh-add -l`> schlüssel abrufen
### Wiederholende Schritte für jede weitere VM-Einrichtung
Auf jump-lm-10-3:
1. Öffentliche Schlüssel in dementsprechende VM setzen:
     `ssh-copy-id -i ~/.ssh/id_ed25519.pub root@10.10.20.xx`
2. Bequeme Verbindung:
	   `nano ~/.ssh/config`
3. Host Daten eintragen:
	   *Host pbs*
		   *Hostname 10.10.20.10*
		   *User root*
		   *Identify file ~/.ssh/ided25519*
		   *Identify Only yes*
4. Rechte anpassen
     `chmod 600 ~/.ssh/config


### OPNsense


## VPN via WireGuard

</details>

# Tests