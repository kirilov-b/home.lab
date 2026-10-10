---
layout: default
title: Virtuelle Machienen - Planung
---

# Virtuelle Machienen - Planung

## Namenskonventionen
### VMID (Name - Betriebssystem - vLAN - IP Host *letzte Oktett*)
- **VMID** *(nur in Proxmox GUI relevant)*
	- Erste Zahl - **VLAN**
	- Zweite Zahl - **Host**
	- Dritte Zahl - **Reihenfolge**
	Beispiel **201**: VLAN 20, erst aufgesetzte Machine
- **Name Abkürzung**
	- **fw** - Firewall
	- **dc** - Domain Controler
	- **pbs** - Proxmox Backup Server
	- **cl1** - Client 1
	- **sec** - Security
	- **mon** - Monitoring
- **Betriebsysteme Abkürzung**
	- **wc** - Windows Client
	- **ws** - Windows Server
	- **lm** - Linux Mint
	- **la** - Linux Alma
	- **ld** - Linux Debian
	- **lu** - Linux Ubuntu
- **Name vLAN**
	- **10** - Menagement - vLAN .10
	- **20** - Service - vLAN .20
	- **30** - Client - vLAN .30
	
	Beispiel: 
  **dc-ws-20-1**
	*Außnahme : **fw-OPNsense**, Clients (DHCP)*

### Domain Planung

Es sind zwei Domäne geplant.
Für Service & Management - **srv.test**
Für Domain Controller - **lab.home**

**Beispiel FQDNs**
	ans-la-10-20.srv.test
	dc-ws-20-02.lab.home
  
## VM-Tabelle

| VM-ID | SSH nickname | VM-NameServer    | Zweck                         | Betriebsystem       | IPv4 /24    | Größe<br>GB | RAM<br>GB | vCPU |
| ----- | ------------ | ---------------- | ----------------------------- | ------------------- | ----------- | ----------- | --------- | ---- |
| 100   | jump         | **jump-lm-10-3** | Administrator, Dokumentation, | Linux Mint          | 10.10.10.3  | 80          | 8         | 4    |
| 101   | ops          | **fw-opnense**   | Firewall                      | OPNsense            |             | 20          | 4         | 2    |
| 102   | ansible      | **ans-la-10-20** | Ansible / Automation          | Linux Alma          | 10.10.10.20 | 20          | 4         | 2    |
| 222   | ipa          | **ipa-la-20-2**  | FreeIPA / Identity / DNS      | Linux Alma          | 10.10.20.2  | 40          | 4         | 2    |
| 201   | dc           | **dc-ws-20-3**   | AD, DNS, DHCP                 | Windows Server 2022 | 10.10.20.3  | 80          | 4         | 4    |
| 211   | pbs          | **pbs-ld-20-10** | Proxmox Backup Server         | PBS(Debian)         | 10.10.20.10 | 20          | 4         | 2    |
| 233   | ngx          | **ngx-la-20-20** | Nginx / Reverse Proxy         | Linux Alma          | 10.10.20.20 | 20          | 2         | 2    |
| 244   | postgres     | **db-ld-20-30**  | PostgreSQL                    | Linux Debian        | 10.10.20.30 | 20          | 4         | 2    |
| 255   | zabbix       | **mon-la-20-40** | Zabbix Monitoring             | Linux Alma          | 10.10.20.40 | 20          | 4         | 2    |
| 266   | docker       | **doc-lu-20-50** | Docker / Podman / Container   | Linux Ubuntu Server | 10.10.20.50 | 40          | 4         | 2    |
| 277   | log          | **log-la-20-60** | Rsyslog                       | Linux Alma          | 10.10.20.60 | 40          | 4         | 2    |
| 288   | wazuh        | **sec-la-20-70** | Wazuh / Security Monitoring   | Linux Alma          | 10.10.20.70 | 64          | 4         | 4    |
| 299   | gitea        | **git-ld-20-80** | Gitea / Git-Server            | Linux Debian        | 10.10.20.80 | 32          | 2         | 2    |
| 301   | cl1          | **cl1-wc-30**    | Testclient                    | Windows 11          | DHCP        | 60          | 4         | 2    |
| 302   | cl2          | **cl2-wc-30**    | Testclient                    | Windows 11          | DHCP        | 40          | 4         | 2    |
| 303   | cl3          | **cl3-mac-30**   | Testclient                    | MacOS               | DHCP        | 40          | 4         | 2    |

