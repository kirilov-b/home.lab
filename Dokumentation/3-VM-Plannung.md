---
layout: default
title: Virtuelle Machienen - Planung
---

# Virtuelle Machienen - Planung

## Namenskonventionen
### VMID (Name - Betriebsystem - vLAN - IP Host)
- **VMID** *(nur in Proxmox GUI relevant)*
	- Erste Zahl - **VLAN**
	- Zweite Zahl - **Host**
	- Dritte Zahl - **Reihenfolge**
	Beispiel **201**: VLAN 20, erst aufgesetzte Machine
- **Name Abkürzung**
	- **fw** - Firewall
	- **dc** - Domain Controler
	- **pbs** - Proxmox Backup Server
	- **cl1** Client 1
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
	- **10** - Menagment - vLAN .10
	- **20** - Service - vLAN .20
	- **30** - Client - vLAN .30
	
	Beispiel: 
  **dc-ws-20-1**
	*Außnahme : **fw-OPNsense**, Clients (DHCP)*

### Domain Planung

Es sind zwei Domäne geplant.
Für Service & Management - **lab.service**
Für Domain Controller - **lab.home**

**Beispiel FQDNs**
	ans-la-10-20.lab.service
	dc-ws-20-02.lab.home
  
## VM-Tabelle

| VM-ID | SSH nickname | VM-NameServer    | Zweck                         | Betriebsystem       | IPv4 /24    | Größe<br>GB | RAM<br>GB | vCPU |
| ----- | ------------ | ---------------- | ----------------------------- | ------------------- | ----------- | ----------- | --------- | ---- |
| 100   | jump         | **jump-lm-10-3** | Administrator, Dokumentation, | Linux Mint          | 10.10.10.3  | 80          | 8         | 4    |
| 101   | ops          | **fw-OPSense**   | Firewall                      | OPNsense            |             | 20          | 4         | 2    |
| 201   | dc           | **dc-ws-20-1**   | AD, DNS, DHCP                 | winServer 2022      | 10.10.20.2  | 80          | 4         | 4    |
| 301   | cl1          | **cl1-wc-30**    | Testclient                    | Windows 11          | DHCP        | 60          | 4         | 2    |
| 211   | pbs          | **pbs-ld-20-10** | Proxmox Backup Server         | PBS(Debian)         | 10.10.20.10 | 20          | 4         | 2    |
| 222   | ipa          | **ipa-la-20-20** | FreeIPA / Identity / DNS      | Linux Alma          | 10.10.20.20 | 20          | 2         | 2    |
| 233   | ngx          | **ngx-la-20-30** | Nginx / Reverse Proxy         | Linux Alma          | 10.10.20.30 | 20          | 2         | 2    |
| 244   | postgres     | **db-ld-20-40**  | PostgreSQL                    | Linux Debian        | 10.10.20.40 | 20          | 4         | 2    |
| 255   | zabbix       | **mon-la-20-50** | Zabbix Monitoring             | Linux Alma          | 10.10.20.50 | 20          | 4         | 2    |
| 266   | docker       | **doc-lu-20-60** | Docker / Podman / Container   | Linux Ubuntu Server | 10.10.20.60 | 40          | 4         | 2    |
| 102   | ansible      | **ans-la-10-20** | Ansible / Automation          | Linux Alma          | 10.10.10.20 | 20          | 4         | 2    |
| 277   |              | **log-la-20-70** | Zentrales Logging             | Linux Alma          | 10.10.20.70 | 40          | 4         | 2    |
| 288   | wazuh        | **sec-la-20-80** | Wazuh / Security Monitoring   | Linux Alma          | 10.10.20.80 | 64          | 4         | 4    |
| 299   | gitea        | **git-ld-20-90** | Gitea / Git-Server            | Linux Debian        | 10.10.20.90 | 32          | 2         | 2    |
| 302   |              | **cl2-wc-30**    | Testclient                    | Windows 11          | DHCP        | 40          | 4         | 2    |
| 303   |              | **cl3-mac-30**   | Testclient                    | MacOS               | DHCP        | 40          | 4         | 2    |
|       |              |                  |                               |                     |             |             |           | 42   |
