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
	- **FW** - Firewall
	- **DC** - Domain Controler
	- **PBS** - Proxmox Backup Server
	- **CLT1** Client 1
	- **SEC** - Security
	- **MON** - Monitoring
- **Betriebsysteme Abkürzung**
	- **WC** - Windows Client
	- **WS** - Windows Server
	- **LM** - Linux Mint
	- **LA** - Linux Alma
	- **LD** - Linux Debian
	- **LU** - Linux Ubuntu
- **Name vLAN**
	- **10** - Menagment - vLAN .10
	- **20** - Service - vLAN .20
	- **30** - Client - vLAN .30
	
	Beispiel: 
  **DC-WS-20-1**
	*Außnahme : **FW-OPNsense**, Clients (DHCP)*
  
## VM-Tabelle

| VM-ID | SSH nickname | VM-NameServer     | Zweck                                 |     Betriebsystem     |    IPv4 /24    | Größe<br>GB | RAM<br>GB | vCPU |
| ----- | ------------ | ----------------- | ------------------------------------- | ----------------------| ----------- | ----------- | --------- | ---- |
|  100  | jump         | **JUMP-LM-10-3**  | Administrator, Dokumentation,		   |      Linux Mint       | 10.10.10.3  |     80      |     8     |  4   |
|  101  | ops          | **FW-OPSense**    | Firewall                              |       OPNsense        |             |     20      |     4     |  2   |
|  201  | dc           | **DC-WS-20-1**    | AD, DNS, DHCP                         |    winServer 2022     | 10.10.20.2  |     80      |     4     |  4   |
|  301  | cl1          | **CL1-WC-30**     | Testclient                            |      Windows 11       |    DHCP     |     60      |     4     |  2   |
|  211  | pbs          | **PBS-PBS-20-10** | Proxmox Backup Server                 | Proxmox Backup Server | 10.10.20.10 |     20      |     4     |  2   |
|  222  | ipa          | **IPA-LA-20-20**  | FreeIPA / Identity / DNS              |       AlmaLinux       | 10.10.20.20 |     20      |     2     |  2   |
|  233  | ngx          | **NGX-LA-20-30**  | Nginx / Reverse Proxy                 |       AlmaLinux       | 10.10.20.30 |     20      |     2     |  2   |
|  244  | postgres     | **DB-LD-20-40**   | PostgreSQL                            |        Debian         | 10.10.20.40 |     20      |     4     |  2   |
|  255  | zabbix       | **MON-LA-20-50**  | Zabbix Monitoring                     |       AlmaLinux       | 10.10.20.50 |     20      |     4     |  2   |
|  266  | docker       | **DOC-LU-20-60**  | Docker / Podman / Container           |     Ubuntu Server     | 10.10.20.60 |     40      |     4     |  2   |
|  102  | ansible      | **ANS-LU-10-20**  | Ansible / Automation                  |     Ubuntu Server     | 10.10.10.20 |     20      |     2     |  2   |
|  277  |              | **LOG-LA-20-70**  | Zentrales Logging                     |       AlmaLinux       | 10.10.20.70 |     40      |     4     |  2   |
|  288  | wazuh        | **SEC-LA-20-80**  | Wazuh / Security Monitoring           |       AlmaLinux       | 10.10.20.80 |     64      |     4     |  4   |
|  299  | gitea        | **GIT-LD-20-90**  | Gitea / Git-Server                    |        Debian         | 10.10.20.90 |     32      |     2     |  2   |
|  302  |              | **CL2-WC-30**     | Testclient                            |      Windows 11       |    DHCP     |     40      |     4     |  2   |
|  303  |              | **CL3-MAC-30**    | Testclient                            |         MacOS         |    DHCP     |     40      |     4     |  2   |
|       |              |                   |                                       |                       |             |     656     |     62    |  42  |
