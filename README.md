# Home Lab

Technische Projektdokumentation meines Home-Lab-Projekts.

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

| VM | ID | Funktion |
|---|---:|---|
| JUMP-LM-10-3 | 100 | Verwaltung / Dokumentation |
| FW-OPSense | 101 | Firewall / Router |
| ANS-LU-10-20 | 102 | Ansible / Automation |
| DC-WS-20-1 | 201 | Domain Controller |
| PBS-PBS-20-10 | 211 | Proxmox Backup Server |
| IPA-LA-20-20 | 222 | FreeIPA / Identity / DNS |
| NGX-LA-20-30 | 233 | Nginx / Reverse Proxy |
| DB-LD-20-40 | 244 | PostgreSQL |
| MON-LA-20-50 | 255 | Zabbix Monitoring |
| DOC-LU-20-60 | 266 | Docker / Podman / Container |
| LOG-LA-20-70 | 277 | Zentrales Logging |
| SEC-LA-20-80 | 288 | Wazuh / Security Monitoring |
| GIT-LD-20-90 | 299 | Git-Server |
| CL1-WC-30 | 301 | Windows-11-Testclient |
| CL2-WC-30 | 302 | Windows-11-Testclient |
| CL3-MAC-30 | 303 | Testclient |

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
