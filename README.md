# Home Lab

Technische Projektdokumentation meines Home-Lab-Projekts.

Das Projekt beinhaltet zwei Umgebungen. 
- Ein **Service (SRV)** Domäne: Der besteht nur aus Server hauptsächlich Linux + ein Windows Domain Controller (relevant für die zweite Umgebung) 
  Hier werden Dienste für Überwachung, Automatisierungen, Datensicherungen
- Ein **Client (CL)** Domäne: Diese Umgebung simuliert eine Unternehmungsumgebung mit Clienets in eine separate Netzwerk und beschäftigt sich mit der Integration von Active Directory und unterschiedliche Clients OS: Windows, MacOS, Linux

## Dokumentation

→ [Home Lab Dokumentation öffnen](https://kirilov-b.github.io/home.lab/)

Die ausführliche und aufbereitete Dokumentation ist über GitHub Pages verfügbar.

## Inhalt

Die Dokumentation umfasst aktuell:

- **Hardware & Setup**
  - Grundinstallation und Aufbau der Home-Lab-Umgebung
- **Netzwerk**
  - Netzwerkstruktur, Adressierung und Konfiguration
- **VM-Planung**
  - Übersicht und Planung der virtuellen Maschinen
- **Backup**
  - Proxmox Backup Server und Backup-Konzept
- **Netzwerkdiagramm**
  - Übersicht der Infrastruktur und Netzwerkverbindungen
- **Alle weitere VMs**

## Virtuelle Infrastruktur

Die aktuelle VM-Planung umfasst unter anderem folgende Systeme:

| VM-NameServer    | Zweck                         | Betriebsystem       |
| ---------------- | ----------------------------- | ------------------- |
| **jump-lm-10-3** | Administrator, Dokumentation, | Linux Mint          |
| **fw-opnense**   | Firewall                      | OPNsense            |
| **ans-la-10-20** | Ansible / Automation          | Linux Alma          |
| **ipa-la-20-2**  | FreeIPA / Identity / DNS      | Linux Alma          |
| **dc-ws-20-3**   | AD, DNS, DHCP                 | Windows Server 2025 |
| **pbs-ld-20-10** | Proxmox Backup Server         | PBS(Debian)         |
| **ngx-la-20-20** | Nginx / Reverse Proxy         | Linux Alma          |
| **db-ld-20-30**  | PostgreSQL                    | Linux Debian        |
| **mon-la-20-40** | Zabbix Monitoring             | Linux Alma          |
| **doc-lu-20-50** | Docker / Podman / Container   | Linux Ubuntu Server |
| **log-la-20-60** | Rsyslog                       | Linux Alma          |
| **sec-la-20-70** | Wazuh / Security Monitoring   | Linux Alma          |
| **git-ld-20-80** | Gitea / Git-Server            | Linux Debian        |
| **cl1-wc-30**    | Testclient                    | Windows 11          |
| **cl2-lm-30**    | Testclient                    | Linux Mint          |
| **cl3-mac-30**   | Testclient                    | MacOS               |

## Netzwerk

Die Infrastruktur ist logisch in mehrere Netzbereiche aufgeteilt:

- `10.10.10.0/24` – Infrastruktur / Management
- `10.10.20.0/24` – Server und zentrale Dienste
- `10.10.30.0/24` – Clients und Testsysteme

Die zentrale Firewall übernimmt Routing und die Netzsegmentierung. DNS, DHCP, Identitätsdienste, Monitoring, Logging, Backup und weitere Infrastruktur-Dienste werden innerhalb der virtuellen Umgebung bereitgestellt.

## Netzwerkdiagramm

Das aktuelle Infrastrukturdiagramm befindet sich im Repository:

- [Diagramm als PDF](Dokumentation/diagramm.pdf)
- [Diagramm in DRAWIO öffnen](https://app.diagrams.net/#Hkirilov-b%2Fhome.lab%2Fmain%2FDokumentation%2Fdiagramm.drawio)

## Repository

Dieses Repository enthält die technischen Dokumentationen, Konfigurationsdateien und Diagramme des Home-Lab-Projekts.

Die ausführliche und aufbereitete Dokumentation ist über GitHub Pages verfügbar:

**https://kirilov-b.github.io/home.lab/**

---

*Home Lab · Technische Projektdokumentation*
