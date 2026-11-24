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



Die Docker-VM wurde mit Ubuntu Server 24.04.5 LTS eingerichtet und besitzt die IP-Adresse `10.10.20.50`. Anschließend wurden Docker Engine und das Docker Compose Plugin über das offizielle Docker-Repository installiert. Zur grafischen Verwaltung wurde Portainer CE als Docker-Container eingerichtet und über HTTPS auf Port `9443` erreichbar gemacht. Zusätzlich wurden der Zabbix Agent 2 und der Wazuh Agent installiert, sodass die VM in die bestehende Monitoring- und Sicherheitsinfrastruktur integriert ist. NGINX wurde zunächst nicht installiert, da ein Reverse Proxy erst bei Bedarf für weitere Webdienste ergänzt werden soll.

# Anwendung


# Tests
