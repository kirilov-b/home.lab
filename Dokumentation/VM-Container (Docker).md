---
layout: default
title: "[SRV] VM – Docker / Podman / Container"
---
# VM – Docker / Podman / Container
Das ist nur ein Test
**VM Parameter**

|      **Parameter** | Wert                        |       **Parameter** | Wert          |
| -----------------: | --------------------------- | ------------------: | ------------- |
|          **VM-ID** | 257                         |       **CPU / RAM** | 2 Core / 4 GB |
|       **vLAN Tag** | 20                          |       **Hard Disk** | 80 GB         |
| **VM / Host-Name** | doc-lu-20-50                |            **IPv4** | 10.0.20.50    |
|          **Zweck** | Docker / Podman / Container |         **Gateway** | 10.0.20.1     |
| **Betriebssystem** | Linux Ubuntu 24.04 LTS      |          **Domäne** | srv.test      |
|    **Local Admin** | ladministrator              | **Netzwerkadapter** |               |

# Übersicht

Docker dient als zentrale Container-Plattform der Umgebung. Damit können wir Anwendungen und zusätzliche Netzwerkdienste flexibel und unabhängig vom Betriebssystem bereitstellen. Ziel ist es, Docker-Dienste sowohl innerhalb der bestehenden Domäne als auch domänenübergreifend in den verschiedenen Netzwerkbereichen einzusetzen. Die Verwaltung der Container erfolgt über die grafische Weboberfläche **Portainer CE**.

## Vorbereitung und Integration

1. **Hostname** wurde laut Namenkonvention angepasst: 
2. Einbinden in das geplante **Netzwerk**
3. Einbinden in der vorgesehene Domäne
4. Einrichtung von **SSH** Verbindung mit dem Jump VM
5. Integration in **Zabbix** Monitoring System:  [Zabbix Integration](./Integration.md#zabbix)
6. Integration in **Wazuh** Security Monitoring System: [Wazuh Integration](./Integration.md#wazuh)
# Docker - Installation und Konfiguration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Schritte anzeigen</span></strong></summary>


## 1.Installation
Die Installation erfolgt über das offizielle Docker-APT-Repository. Das ist der bevorzugte Weg für eine wartbare Installation und ermöglicht spätere Updates über APT.

### 1.1 Paketindex aktualisieren und Voraussetzungen installieren

`apt update`
`apt install -y ca-certificates curl`
### 1.2 Offiziellen Docker-GPG-Schlüssel hinterlegen

```
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

### 1.3 Docker-Repository hinzufügen

```
cat > /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Nach diese Schritte müssen die Packete aktuallisiert werden `apt update`

### 1.4 Docker Engine und Compose installieren

Folgendes wird installiert: 
- Docker Engine (`docker-ce`)
- Docker CLI (`docker-ce-cli`)
- Container Runtime (`containerd.io`)
- Docker Buildx Plugin
- Docker Compose Plugin (`docker-compose-plugin`)

```
apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

### 1.5. Docker-Dienst starten und Installation testen

```
systemctl enable --now docker
systemctl status docker --no-pager
```

```
docker --version
docker compose version
```

```
docker run hello-world
```

Der Testcontainer gibt bei erfolgreicher Ausführung eine Bestätigung aus und beendet sich anschließend wieder. Die Dienste laufen und die Versionen werden angezeigt. Docker sagt "hallo"

## 2. Konfiguration

### 2.1 Portainer Einrichten: WebGUI
Ein Verzeichnis wird angelegt und ein Compose-Datei wurde erstellt. 
 
```
mkdir -p /opt/docker/portainer
cd /opt/docker/portainer
```

```
nano compose.yaml
```

**Compose.yaml**
```
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    ports:
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
```

### 2.1. Portainer starten
Zuerst wurde die Konfiguration überprüft, danach wird Portainer gestartet und Container kontrolliert.

`docker compose config`
`docker compose up -d`
`docker ps`
### 2.2. Portainer im Browser einrichten
Erfolgreich eingerichtet
Wir haben die Weboberfläche geöffnet:
`https://10.10.20.50:9443`

1. Administrator-Konto `admin` erstellt.
2. Ein eigenes Passwort festgelegt.
3. Edge Compute deaktiviert gelassen.
4. Edge Compute mit **Skip** übersprungen.
5. Die lokale Docker-Umgebung `local` verwendet.

</details>

# Anwendung


# Tests
