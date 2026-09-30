---
layout: default
title: VM – Firewall - OPNsense
---

# VM – Firewall - OPNsense

## Übersicht

Diese Seite dokumentiert die virtuelle Firewall **FW-OPSense**. 
Die Firewall übernimmt die zentrale Netzwerkfunktion innerhalb des Home Labs. Sie stellt die Verbindung zwischen den definierten Netzwerksegmenten her und kontrolliert den Datenverkehr zwischen ihnen.
Dient als Managed Switch für VLAN.
### Aufgaben
- Routing zwischen den Netzwerksegmenten
- Filterung von Verbindungen
- Zugriffskontrolle
- Trennung von Management-, Service- und Client-Netz (VLAN)
- zentrale Netzwerkverwaltung

## Proxmox-Konfiguration
- General > VM ID: 202
- System > Machine q35
- System > BIOS: OVMF (UEFI)
- Disk > BUS: SCSI
- Disk > Size: 20 GB
- CPU > Cores: 2
- CPU > Type: Host
- Memory > 4096
- NIC 1 > WAN > Bridge: vmbr0
- NIC 1 > WAM > Model: VirtIO
### Zweiten NIC hinzufügen
**prox > System > Network > Create > Linux Bridge**
**FW1 > Hardware > Add > Network Device**
- NIC 2 > LAN > Bridge: vmbr1
- NIC 2 > LAN > Model: VirtIO

## Installation
- EFI Disk - Pre-Enrolled Keys deaktivieren!
- Install UFS
- Deutsch Layout einfügen
- Hard Disk auswählen

## OPNsense - Grundkonfiguration
- die Netzwerk Interface anpassen:
	- WAN - vtnet0: IPv4 (via DHCP) **172.18.87.1/24**
	- VLAN-10-MGMT - IPV4 (statisch) **10.10.10.1 / 24**
    - VLAN-20-SRV  - IPV4 (statisch) **10.10.20.1 / 24**
    - VLAN-30-CLT  - IPV4 (statisch) **10.10.30.1 / 24**
### GUI - 10.10.10.1
