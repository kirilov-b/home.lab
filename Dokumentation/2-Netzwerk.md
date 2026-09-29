---
layout: default
title: Netzwerk
---

# Netzwerk

## Netzwerkübersicht

Das Home Lab ist logisch in mehrere Netzwerkbereiche aufgeteilt.

| Netzwerk | Bereich | Verwendung |
|---|---|---|
| 10.10.10.0/24 | Infrastruktur / Management | Administration und zentrale Infrastruktur |
| 10.10.20.0/24 | Server | zentrale Serverdienste |
| 10.10.30.0/24 | Clients / Testsysteme | Windows- und Testclients |

## Zentrale Dienste

Im Servernetz befinden sich unter anderem:

- Domain Controller
- FreeIPA / Identity Management
- DNS
- Nginx Reverse Proxy
- PostgreSQL
- Zabbix Monitoring
- zentrales Logging
- Wazuh Security Monitoring
- Gitea Git-Server
- Container-Umgebung
- Proxmox Backup Server

## Firewall und Routing

Die Netzbereiche werden über die Firewall voneinander getrennt und kontrolliert miteinander verbunden.

Die Firewall übernimmt dabei unter anderem:

- Routing zwischen den Netzen
- Filterung des Netzwerkverkehrs
- Zugriffskontrolle
- Absicherung der einzelnen Segmente

## DHCP und DNS

Die Clients beziehen ihre IP-Konfiguration per DHCP. DNS wird für die Namensauflösung innerhalb der Laborumgebung verwendet.
