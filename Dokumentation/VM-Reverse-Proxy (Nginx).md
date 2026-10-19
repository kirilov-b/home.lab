---
layout: default
title: "[SRV] VM – Nginx / Reverse Proxy"
---
# NOCH NICHT KONFIGURIERT
# VM – Nginx / Reverse Proxy

**VM Parameter**

|      **Parameter** | Wert                  | **Parameter** | Wert        |
| -----------------: | --------------------- | ------------: | ----------- |
|          **VM-ID** | 233                   |      **IPv4** | 10.10.20.20 |
|       **vLAN Tag** | 20                    |    **Domain** | srv.test    |
|        **VM-Name** | ngc-la-20-20          |     **Größe** | 20 GB       |
|          **Zweck** | Nginx / Reverse Proxy |       **RAM** | 2 GB        |
| **Betriebssystem** | Linux Alma            |      **vCPU** | 2           |


# Übersicht

Nginx auf dient als zentraler **Reverse Proxy** für deine Homelab-Webdienste. Es ruft Anwendungen über gut merkbare DNS-Namen und HTTPS auf; Nginx nimmt die Verbindung entgegen und leitet sie an den jeweiligen internen Server weiter. So kann man den direkten Browserzugriff vom Jump-Host auf Backend-WebGUIs reduzieren, TLS zentral verwalten und die Dienste einheitlich erreichbar machen.

# Konfigurationsschritte 

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Schritte anzeigen</span></strong></summary>

## 1. Nginx installieren und starten

**Auf NGX `10.10.20.20`, Shell als `root`:**

```
dnf install -y nginx
systemctl enable --now nginx
systemctl status nginx
```

Damit wird Nginx installiert, beim Systemstart aktiviert und sofort  
gestartet.


</details>
# Tests
