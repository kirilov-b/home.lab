---
layout: default
title: "[MGMT] VM – Administrator"
---

# Verwaltung, Dokumentation, Remote Verbindung

**VM Parameter**

|      **Parameter** | Wert                      | **Parameter** | Wert       |
| -----------------: | ------------------------- | ------------: | ---------- |
|          **VM-ID** | 100                       |      **IPv4** | 10.10.10.3 |
|       **vLAN Tag** | 10                        |    **Domain** | keinen     |
|        **VM-Name** | jump-lm-10-3              |     **Größe** | 80 GB      |
|          **Zweck** | Verwaltung, Dokumentation |       **RAM** | 8 GB       |
| **Betriebssystem** | Linux Mint                |      **vCPU** | 4          |




# Konfiguration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Konfiguration anzeigen</span></strong></summary>
## RDP Verbindung
Remote Destkop Protocol **(RDP)** wird auf der Controller Laptop und auf dem jump-lm-10-3 (**JUMP**) installiert, um eine Verbindung zwischen beide Machinen ohne die Proxmox Browser Verbindung benutzen zu müssen.
Ziel ist der **jump** ein Jumphost zu machen und von dort aus die komplette Umgebung zu verwalten. Die **jump** VM befindet sich in VLAN 10 - Management Netz (**MGMT**). Über den OPNsens Firewall wird an der Managment Netz hat folgende Zugriff ermöglicht:
- Verbindung zu HTTPS: **Proxmox WebGUI**
- Verbindung zu **Service VLAN 20** (**SRV)**
- Verbindung zu **Clients VLAN 30** **(CL)**
- Verbindung zu **OPNsense Web GUI** 

Zusätzlich wird zu jede VM von **jump** aus eine SSH Verbindung eingerichtet.

## SSH Einrichtung

Auf **jump-lm-10-3**:

1. Ordner für den SHA256 Schlüssel erstellen
	 `mkdir -p ~/.ssh`
	 `chmod 700 ~/.ssh`
2. Schlüssel generieren
	 `ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519`
	 Ein Passphrase (Password) setzen: laut Vorgaben
		Zwei keys werden erstellt:
  	_~/.ssh/id_ed25519 (geheim)
   	~/.ssh/id_ed25519.pub_
3. Schlüssel prüfen:
	`eval "$(ssh-agent -s)`
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
     `chmod 600 ~/.ssh/config`


## VPN via WireGuard

<details markdown="1">
  <summary><strong><span style="color: #00813F;">VPN Einrichtung anzeigen</span></strong></summary>

## Ziel und Netzwerkübersicht

Ziel ist es, von einem externen Windows-Rechner über eine verschlüsselte WireGuard-VPN-Verbindung auf den Jumphost im Management-VLAN zuzugreifen. Anschließend wird eine Remote-Desktop-Verbindung (RDP) hergestellt.

### Netzwerkparameter

| Komponente                 | IP-Adresse / Einstellung |
| -------------------------- | ------------------------ |
| Heimrouter                 | `172.16.87.1`            |
| OPNsense WAN               | `172.16.87.2`            |
| OPNsense VLAN 10 / Gateway | `10.10.10.1`             |
| Linux Mint Admin-PC        | `10.10.10.3`             |
| WireGuard-Netzwerk         | `10.10.123.0/24`         |
| OPNsense WireGuard-IP      | `10.10.123.1`            |
| Externer Windows-Client    | `10.10.123.2`            |
| WireGuard-Port             | UDP `51821`              |
| RDP-Port                   | TCP `3389`               |

## 1. Einrichtung auf OPNsense und im Heimnetz

### 1.1 WireGuard konfigurieren

1. WireGuard auf OPNsense aktivieren.
2. Einen lokalen Tunnel mit der VPN-Adresse `10.10.123.1/24` konfigurieren.
3. Einen Peer für den externen Windows-Rechner anlegen.
4. Dem Peer die Adresse `10.10.123.2/32` zuweisen.
5. Den UDP-Port `51821` als Listen-Port verwenden.

### 1.2 Heimrouter konfigurieren

Die Freigabe ermöglicht den Verbindungsaufbau von außerhalb des Heimnetzes.
In der Heimrouter eine Portfreigabe einrichten:

- Protokoll: UDP
- Externer Port: `51821`
- Zielgerät: OPNsense-WAN
- Ziel-IP: `172.16.87.2`
- Ziel-Port: `51821`

### 1.3 Firewall-Regeln auf OPNsense

Folgende Regeln müssen entsprechend der vorhandenen Konfiguration eingerichtet und geprüft werden. Die Regeln sollten möglichst auf die benötigten Quellnetze, Zieladressen und Ports beschränkt sein.

- **WAN:** Eingehenden WireGuard-Verkehr über UDP-Port `51821` zulassen.
- **WireGuard/VPN:** Verkehr vom VPN-Netz `10.10.123.0/24` zum Management-Netz `10.10.10.0/24` erlauben.
- **Zielzugriff:** RDP zu `10.10.10.3` über TCP-Port `3389` zulassen.

## 2. Einrichtung auf dem Jumphost

### 2.1 RDP-Dienst installieren und aktivieren

Falls noch nicht vorhanden:

```bash
sudo apt update
sudo apt install xrdp
sudo systemctl enable --now xrdp
```

Prüfen, ob der Dienst läuft und Port 3389 geöffnet ist:

```bash
sudo systemctl status xrdp
sudo ss -lntp | grep ':3389'
```

Der Dienst sollte auf Port `3389` lauschen.

### 2.2 UFW-Firewall konfigurieren

Die UFW-Firewall erlaubt RDP nur von freigegebenen Quelladressen. Deshalb muss auch das WireGuard-Netz freigeschaltet werden.

**Erforderliche Freigabe:**

```bash
sudo ufw allow from 10.10.123.0/24 to 10.10.10.3 port 3389 proto tcp
```

Regeln kontrollieren:

```bash
sudo ufw status numbered
```

Damit dürfen VPN-Clients aus `10.10.123.0/24` den RDP-Dienst auf dem Jumphost-PC erreichen. Andere Quellen werden dadurch nicht automatisch freigegeben.

## 3. Einrichtung auf dem externen Windows-Rechner

### 3.1 WireGuard installieren

1. WireGuard für Windows installieren. *(Via WireGuard Seite)*
2. Einen Tunnel erstellen.
3. Die Client-Schlüssel und die Peer-Konfiguration entsprechend der OPNsense-Konfiguration eintragen.

Tunnelkonfiguration:

```ini
[Interface]
PrivateKey = <PRIVATER_CLIENT_SCHLUESSEL>
Address = 10.10.123.2/32

[Peer]
PublicKey = <OEFFENTLICHER_OPNSENSE_SCHLUESSEL>
Endpoint = <OEFFENTLICHE_IP_ODER_DYNDNS>:51821
AllowedIPs = 10.10.123.0/24, 10.10.10.0/24, 10.10.1.0/24
PersistentKeepalive = 25
```

**Hinweise:**

- Platzhalter durch die tatsächlichen Werte ersetzen.
- `AllowedIPs` legt fest, welche Zielnetze über den VPN-Tunnel erreichbar sein sollen.
- `Endpoint` muss auf die von außen erreichbare Adresse des Heimnetzes zeigen.
- Die Schlüssel müssen zu den konfigurierten Peers passen.

### 3.2 VPN-Verbindung testen

Den Tunnel in WireGuard aktivieren.
In PowerShell prüfen:

```powershell
ping 10.10.10.3
```

Anschließend testen, ob der RDP-Port erreichbar ist:

```powershell
Test-NetConnection 10.10.10.3 -Port 3389
```

### 3.3 RDP-Verbindung herstellen

1. Via `mstsc` in Run.
2. Als Computer `10.10.10.3` eintragen.
3. Auf **Verbinden** klicken.
4. Mit dem entsprechenden Linux-Jumphost-Benutzerkonto anmelden.

## 4. Ergebnis

Nach erfolgreicher Einrichtung kann der externe Windows-Rechner über WireGuard eine verschlüsselte Verbindung zum Heimnetz aufbauen und per RDP auf den Linux-Jumphost-PC unter `10.10.10.3` zugreifen.

**Wichtig:** WireGuard muss auf dem Windows-Rechner aktiv sein, bevor die RDP-Verbindung aufgebaut wird.


</details>


</details>

# Tests
