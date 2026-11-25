---
layout: default
title: "[SRV] VM – Nginx / Reverse Proxy"
---
# NOCH NICHT KONFIGURIERT
# VM – Nginx / Reverse Proxy

**VM Parameter**

|      **Parameter** | Wert                  |       **Parameter** | Wert          |
| -----------------: | --------------------- | ------------------: | ------------- |
|          **VM-ID** | 224                   |       **CPU / RAM** | 2 Core / 4 GB |
|       **vLAN Tag** | 20                    |       **Hard Disk** | 20 GB         |
| **VM / Host-Name** | ngc-la-20-20          |            **IPv4** | 10.0.20.20    |
|          **Zweck** | Nginx / Reverse Proxy |         **Gateway** | 10.0.20.1     |
| **Betriebssystem** | Linux Alma 10.2       |          **Domäne** | srv.test      |
|    **Local Admin** | root                  | **Netzwerkadapter** |               |

# Übersicht

Nginx auf dient als zentraler **Reverse Proxy** für deine Homelab-Webdienste. Es ruft Anwendungen über gut merkbare DNS-Namen und HTTPS auf; Nginx nimmt die Verbindung entgegen und leitet sie an den jeweiligen internen Server weiter. So kann man den direkten Browserzugriff vom Jump-Host auf Backend-WebGUIs reduzieren, TLS zentral verwalten und die Dienste einheitlich erreichbar machen.

## Vorbereitung und Integration

1. **Hostname** wurde laut Namenkonvention angepasst: 
2. Einbinden in das geplante **Netzwerk**
3. Einbinden in der vorgesehene Domäne
4. Einrichtung von **SSH** Verbindung mit dem Jump VM
5. Integration in **Zabbix** Monitoring System:  [Zabbix Integration](./Integration.md#zabbix)
6. Integration in **Wazuh** Security Monitoring System: [Wazuh Integration](./Integration.md#wazuh)



# NGINX - installation und Konfiguration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Schritte anzeigen</span></strong></summary>

## 1. Nginx installieren und starten

**Auf NGX `10.0.20.20`, Shell als `root`:**

```
dnf install -y nginx
systemctl enable --now nginx
systemctl status nginx
```

Damit wird Nginx installiert, beim Systemstart aktiviert und sofort  
gestartet.


</details>
# Tests
