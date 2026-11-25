---
layout: default
title: "[SRV] VM – Docker / Podman / Container"
---
# NOCH NICHT KONFIGURIERT
# VM – Docker / Podman / Container

**VM Parameter**

|      **Parameter** | Wert                        |       **Parameter** | Wert          |
| -----------------: | --------------------------- | ------------------: | ------------- |
|          **VM-ID** | 257                         |       **CPU / RAM** | 2 Core / 4 GB |
|       **vLAN Tag** | 20                          |       **Hard Disk** | 80 GB         |
| **VM / Host-Name** | doc-lu-10-50                |            **IPv4** | 10.0.20.50    |
|          **Zweck** | Docker / Podman / Container |         **Gateway** | 10.0.20.1     |
| **Betriebssystem** | Linux Ubuntu 24.04 LTS      |          **Domäne** | srv.test      |
|    **Local Admin** | ladministrator              | **Netzwerkadapter** |               |

# Übersicht

Docker dient als zentrale Container-Plattform der Umgebung. Damit können wir Anwendungen und zusätzliche Netzwerkdienste flexibel und unabhängig vom Betriebssystem bereitstellen. Ziel ist es, Docker-Dienste sowohl innerhalb der bestehenden Domäne als auch domänenübergreifend in den verschiedenen Netzwerkbereichen einzusetzen. Die Verwaltung der Container erfolgt über die grafische Weboberfläche Portainer CE.

## Vorbereitung und Integration

1. **Hostname** wurde laut Namenkonvention angepasst: 
2. Einbinden in das geplante **Netzwerk**
3. Einbinden in der vorgesehene Domäne
4. Einrichtung von **SSH** Verbindung mit dem Jump VM
5. Integration in **Zabbix** Monitoring System:  [Zabbix Integration](./Integration.md#zabbix)
6. Integration in **Wazuh** Security Monitoring System: [Wazuh Integration](./Integration.md#wazuh)
# Docker - Installation und Konfiguration


## Konfiguration


# Anwendung


# Tests
