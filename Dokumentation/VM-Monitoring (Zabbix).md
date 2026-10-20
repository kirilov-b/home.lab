---
layout: default
title: "[SRV] VM – Zabbix Monitoring"
---

# VM – Zabbix Monitoring

**VM Parameter**

|          Parameter | Wert              | **Parameter** | Wert        |
| -----------------: | ----------------- | ------------: | ----------- |
|          **VM-ID** | 255               |      **IPv4** | 10.10.20.40 |
|       **vLAN Tag** | 20                |    **Domain** | srv.test    |
|        **VM-Name** | mon-la-20-40      |     **Größe** | 40 GB       |
|          **Zweck** | Zabbix Monitoring |       **RAM** | 4 GB        |
| **Betriebssystem** | Linux Alma 10.2   |      **vCPU** | 2           |


# Übersicht
Zabbix ist ein Monitoring Plattform, die alle VMs überwachen soll unabhängig davon, ob sie ein Teil einer Domäne sind oder nicht. Alle existierende VMs inklusiv Proxmox VE und Proxmox PBS.
Das Plattform überwacht. Hardware Auslastung (CPU, RAM, HardDisk), Netzwerkverkehr.

**Anmerkung:** Zabbix verwendet ein PostgreSQL-Instanz. Sie ist unabhängig von der separaten PostgreSQL-VM 2

# Konfigurationsschritte 
Folgende Abschnitt zeigt die Vorbereitung, Installation und Konfiguration von der Zabbix Server auf der VM. 
Voraussetzung für eine reibungslose Integration sind (1. Grundkonfiguration) und (2. SSH Verbindung) von Dokument: [VM-Integration](VM-Integration.md): 

Schritte: 
1. **Zabbix Repository und Pakete installieren**
2. **PostgreSQL initialisieren und Starten**
3. **Zabbix-Datenbank und Benutzer erstellen**
4. **Zabbix-Datenbankschema importieren**
5. **Datenbankverbindung in Zabbix konfigurieren**
6. **PostgreSQL-Authentifizierung korrigieren**
7. **Zabbix-Server starten und prüfen**
8. **Apache starten und Firewall konfigurieren**
9. **Webinstaller und Anmeldung**

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Konfigurationschritte anzeigen</span></strong></summary>

## 1. Zabbix Repository und Pakete installieren

- Installation von offizielle Repository-Paket für Zabix 7.4 auf Alma Linux 10, damit **dnf**passende Packete findet
- Entfernt lokale Paket-Caches und Metadaten, damit keine veralteten Repository-Informationen verwendet werden. 
- Lädt Repository-Metadaten neu herunter und erstellt einen lokalen Cache. 

```
dnf install -y https://repo.zabbix.com/zabbix/7.4/release/alma/10/noarch/zabbix-release-latest-7.4.el10.noarch.rpm
dnf clean all
dnf makecache
```

- Installiert den Zabbix-Server mit PostgreSQL-Unterstützung,  Weboberfläche, Apache-Konfiguration, SQL-Skripte, SELinux-Richtlinien,  Agent 2 und PostgreSQL.

```
dnf install -y \
  zabbix-server-pgsql \
  zabbix-web-pgsql \
  zabbix-apache-conf \
  zabbix-sql-scripts \
  zabbix-selinux-policy \
  zabbix-agent2 \
  postgresql-server
```

## 2. PostgreSQL initialisieren und Starten

- Erstellt die anfängliche PostgreSQL-Datenstruktur.
- Aktiviert
- Prüft
```
postgresql-setup --initdb
systemctl enable --now postgresql
systemctl status postgresql --no-pager
```

## 3. Zabbix-Datenbank und Benutzer erstellen

Öffnet die PostgreSQL-Konsole als Betriebssystembenutzer `postgres`, der  
administrative Datenbankaktionen durchführen kann.

```
sudo -u postgres psql
```

In der PostgeSQL-Konsole:
- Erstellt den Datenbankbenutzer `zabbix` und setzt sein Passwort.
- Erstellt die Datenbank `zabbix` und setzt `zabbix` als Eigentümer.

```
CREATE USER zabbix WITH PASSWORD 'geheime Passwort';
CREATE DATABASE zabbix OWNER zabbix;
\q
```

## 4. Zabbix-Datenbankschema importieren

- Entpackt das komprimierte SQL-Schema und leitet es an **pslq** weiter. 
- Importiert als Betriebsystemnutzer **zabbix** in die Datenbank **zabbix**

```
zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

## 5. Datenbankverbindung in Zabbix konfigurieren

Die Server Konfigurationsdatei muss angepasst werden:

```
nano /etc/zabbix/zabbix_server.conf
```

Diese Einstellungen müssen aktiviert sein und angepasst werden:

```
DBUser=zabbix
DBPassword=`geheime Password`
```

Die Datei enthält ein Passwort im Klartext. Zugriff wird einschränken:

```
chown root:zabbix /etc/zabbix/zabbix_server.conf
chmod 640 /etc/zabbix/zabbix_server.conf
ls -l /etc/zabbix/zabbix_server.conf
```

## 6. PostgreSQL-Authentifizierung korrigieren

Beim ersten Start meldete Zabbix  
*Ident authentication failed for user "zabbix".*

Zeigt aktive lokale und TCP-Verbindungsregeln aus *pg_hba.conf*:

```
grep -nE '^[[:space:]]*(local|host)' /var/lib/pgsql/data/pg_hba.conf
```

Datei bearbeiten:

```
nano /var/lib/pgsql/data/pg_hba.conf
```

Die drei normalen lokalen Verbindungsregeln wurden auf  
Passwortauthentifizierung umgestellt:

```
local   all             all                                     scram-sha-256
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

Die Replikationsregeln blieben unverändert. 
Danach wird den PostgreSQL Dienst neugestartet:

```
systemctl reload postgresql
psql -h 127.0.0.1 -U zabbix -d zabbix -W
```

Bei Erfolg erscheint `zabbix=>`. Mit `\q` verlassen.

## 7. Zabbix-Server starten und prüfen

```
systemctl enable --now zabbix-server
systemctl status zabbix-server --no-pager
tail -n 30 /var/log/zabbix/zabbix_server.log
```

Nach der Korrektur der PostgreSQL-Authentifizierung wurde der Dienst neu  
gestartet:

```
systemctl restart zabbix-server
systemctl status zabbix-server --no-pager
tail -n 20 /var/log/zabbix/zabbix_server.log
```

Ergebnis: `zabbix-server.service` lief mit `active (running)`. Meldungen  
zu nicht aktivierten optionalen Pollern können bei einer frischen  
Installation normal sein. Es sollten keine wiederkehrenden  
Datenbankverbindungsfehler mehr auftreten.

## 8. Apache starten und Firewall konfigurieren

- Aktiviert Apache für den automatischen Start und startet den Webserver sofort.
- Prüft Dienst
- Prüft Firewall
```
systemctl enable --now httpd
systemctl status httpd --no-pager
systemctl is-active firewalld
```

Da **firewalld** aktiv war, wurden HTTP und HTTPS freigegeben:

```
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
firewall-cmd --list-services
```

Weboberfläche: **[http://10.10.20.40/zabbix](http://10.10.20.40/zabbix)**

Die Freigabe des HTTPS-Dienstes öffnet nur den Port; ein TLS-Zertifikat  
muss später separat eingerichtet werden.

## 9. Webinstaller und Anmeldung

Verwendete Werte:

- Datenbanktyp: **PostgreSQL**
- Datenbankhost: **localhost**
- Datenbankport: **5432**
- Datenbankname: **zabbix**
- Datenbankbenutzer: **zabbix**
- Datenbankpasswort: **geheime Passwort**
- Zabbix-Servername: **Zabbix.lab.server**
- Standard-Zeitzone: **Europe/Berlin**
- „Encrypt connection from web interface": nicht aktiviert

Nach erfolgreichem Abschluss des Assistenten war die  
Zabbix-Weboberfläche erreichbar.

Standard-Anmeldedaten einer neuen Installation:

- Benutzername: `Admin`
- Passwort: `zabbix`

Das System ist bereit für Verbindungen mit VMs

</details>

# Tests
