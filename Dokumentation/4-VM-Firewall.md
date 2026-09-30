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
Netzwerk Interface anpassen:
- WAN - vtnet0: IPv4 (via DHCP) **172.18.87.1 /24**
- VLAN-10-MGMT - IPV4 (statisch) **10.10.10.1 /24**
- VLAN-20-SRV  - IPV4 (statisch) **10.10.20.1 /24**
- VLAN-30-CLT  - IPV4 (statisch) **10.10.30.1 /24**
### GUI - 10.10.10.1

## Regeln IPv4

| Betrifft | Bennenung                          | Interface        | Protocol | Source           | S-Port | Destination         | D-Port         |
| -------- | ---------------------------------- | ---------------- | -------- | ---------------- | ------ | ------------------- | -------------- |
| SRV      | Block Firewall vLAN20              | any              | TCP      | VLAN-20          | any    | This Firewall       | https(443)     |
| CLIENT   | Block Firewall vLAN30              | any              | TCP      | VLAN-30          | any    | This Firewall       | https(443)     |
| MGMT     | WireGuard > MGMT                   | WireGuard(Group) | any      | WireGuard(Group) | any    | any                 |                |
| MGMT     | WireGuard VPN                      | WAN              | UDP      | any              | any    | WAN address         | 51820          |
|          | Default allow LAN to any rule<br>  | LAN              | any      | LAN Network      | any    | any                 | any            |
|          | Default allow LAN IPv6 to any rule | LAN              | any      | LAN Network      | any    | any                 | any            |
| JUMP     | RDP Laptop > JUMP                  | WAN              | TCP      | 172.18.87.1      | any    | 10.10.10.3          | ???            |
| CLIENT   | Block 30 > 10                      | VLAN-30          | any      | VLAN-30-Network  | any    | opt1                |                |
| DC       | Allow v30 DNS > DC                 | VLAN-30          | TCP/UDP  | VLAN-30-Network  | any    | 10.10.20.2          | Domain(53)     |
| DC       | Allow v30 AD > DC                  | VLAN-30          | TCP/UDP  | VLAN-30-Network  | any    | 10.10.20.2          | 88,135,389,445 |
| CLIENT   | Block 30 > 20                      | VLAN-30          | any      | VLAN-30-Network  | any    | VLAN-20-Network     |                |
| SRV      | Block 20 > 10                      | VLAN-20          | any      | VLAN-20-Network  | any    | VLAN-10-Network     |                |
| SRV      | Block 20 > 30                      | VLAN-20          | any      | VLAN-20-Network  | any    | VLAN-30-<br>Network |                |
| CLIENT   | Allow VLAN30                       | VLAN-30          | any      | VLAN-30-Network  | any    | any                 |                |
| SRV      | Allow VLAN20                       | VLAN-20          | any      | VLAN-20-Network  | any    | any                 |                |
| MGMT     | Allow VLAN10                       | VLAN-10          | any      | VLAN-10-Network  | any    | any                 |                |
|          | Allow hLAN > WAN OPNs              |                  |          |                  |        | wanip               |                |
