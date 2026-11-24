---
layout: default
title: "[SRV] VM – Docker / Podman / Container"
---
# NOCH NICHT KONFIGURIERT
# VM – Docker / Podman / Container

**VM Parameter**

|      **Parameter** | Wert                        | **Parameter** | Wert       |
| -----------------: | --------------------------- | ------------: | ---------- |
|          **VM-ID** | 257                         |      **IPv4** | 10.0.20.50 |
|       **vLAN Tag** | 20                          |    **Domain** | srv.test   |
|        **VM-Name** | doc-lu-10-50                |     **Größe** | 80 GB      |
|          **Zweck** | Docker / Podman / Container |       **RAM** | 4 GB       |
| **Betriebssystem** | Linux Ubuntu 24.04 LTS      |      **vCPU** | 2          |


# Übersicht

Docker dient als zentrale Container-Plattform der Umgebung. Damit können wir Anwendungen und zusätzliche Netzwerkdienste flexibel und unabhängig vom Betriebssystem bereitstellen. Ziel ist es, Docker-Dienste sowohl innerhalb der bestehenden Domäne als auch domänenübergreifend in den verschiedenen Netzwerkbereichen einzusetzen. Die Verwaltung der Container erfolgt über die grafische Weboberfläche Portainer CE.


# Konfigurationsschritte 

## Vorkonfiguration
Folgende Integrationsschritte werden durchgeführt bevor der Docker Dienst installiert wurde.

## Integration

1. **Hostname** wurde laut Namenkonvention angepasst: `dc-ws-20-3`
2. Einbinden in das geplante **Netzwerk** 
3. Einrichtung von **SSH**
4. Integration in **Zabbix** Monitoring System:  [Zabbix Integration](./Integration.md#zabbix)
5. Integration in **Wazuh** Security Monitoring System: [Wazuh Integration](./Integration.md#wazuh)

## Konfiguration


# Anwendung


# Tests
