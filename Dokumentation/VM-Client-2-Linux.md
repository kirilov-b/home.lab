---
layout: default
title: "[CL] VM – Linux Mint - Testclient 2"
---
# VM – Linux Mint 22.3 (Testclient)

## VM-Parameter

| Parameter          | Wert            | Parameter           | Wert          |
| ------------------ | --------------- | ------------------- | ------------- |
| **VM-ID**          | 302             | **IPv4**            | `10.0.30.32`  |
| **VLAN Tag**       | 30              | **Domain**          | `cl.lab.test` |
| **VM-Name**        | `cl2-lm-30`     | **Festplatte**      | 60 GB         |
| **Zweck**          | Testclient      | **RAM**             | 4             |
| **Betriebssystem** | Linux Mint 22.3 | **vCPU**            | 2             |
| **Hostname**       | `cl2-lm-30`     | **Netzwerkadapter** | VirtIO        |
| **Local Admin**    | ladministrator  |                     |               |

# Übersicht

Der Linux-Mint-Testclient dient dazu, die bestehende Active-Directory-Umgebung aus Sicht eines Linux-Clients zu testen. Zusätzlich wird die Integration eines Linux-Systems in eine Windows-basierte Netzwerkumgebung geübt.

Folgende Themen werden behandelt:

- **Integration in Proxmox:** Installation und Konfiguration einer Linux-Mint-VM.
- **Integration in das Netzwerk:** Einbindung in VLAN 30 und Konfiguration der Netzwerkdienste.
- **Integration in Active Directory:** Beitritt zur Domäne `cl.lab.test` und Anmeldung mit Domänenkonten.
- **Remote Administration:** Verwaltung des Systems über SSH vom Jump-Host aus.


# Konfigurationsschritte

## 1. Vorbereitung und Installation

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Vorbereitungsschritte anzeigen</span></strong></summary>

### 1.1 Linux Mint installieren

Als Betriebssystem wird **Linux Mint 22.3** verwendet.
Die Installation erfolgt über das ISO-Image in Proxmox. Nach dem Start des Installationsmediums wird die virtuelle Festplatte für die Linux-Installation verwendet.
Während der Installation wird ein lokales Administratorkonto angelegt.

### 1.2 Lokales Administratorkonto

Für die lokale Systemverwaltung wurde folgendes Konto eingerichtet:

|Einstellung|Wert|
|---|---|
|Benutzername|`ladministrator`|
|Home-Verzeichnis|`/home/ladministrator`|
|Berechtigung|Lokaler Administrator (`sudo`)|

Das lokale Administratorkonto wird für Installationen und administrative Änderungen am Linux-System verwendet.

### 1.3 Hostname festlegen

Der Hostname der VM wurde auf `cl2-lm-30` festgelegt.
Überprüfung über das Terminal:`hostnamectl`
Der Hostname wird für die DNS-Auflösung und den Zugriff über SSH verwendet.

</details>

## 2. Integration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Integrationsschritte anzeigen</span></strong></summary>

### 2.1 Netzwerkkonfiguration und DNS-Auflösung 

Die VM‑Netzwerkkonfiguration ist automatisch eingerichtet über DHCP. Das entspricht der realen Anbindung eines Endgeräts in einem Unternehmen. Vorkonfiguriert ist nur die Schnittstelle des Netzwerkes. Nach dem Anschließen übernimmt DHCP.

Vor dem Domain Join muss der Linux-Client den Domain Controller über DNS erreichen können.
Namensauflösung ist getestet: Active-Directory-Dienste werden geteste. Der Domain Controller ist über die DNS-Abfrage gefunden worden. Ab diesem Moment übernimmt der DNS und DHCP Server.

`nslookup dc-ws-20-3.cl.lab.test`
`nslookup -type=SRV _ldap._tcp.dc._msdcs.cl.lab.test`

### 2.2 Integration in Active Directory

#### 2.2.1 Voraussetzungen

Für die Integration in Active Directory werden unter Linux Mint die Komponenten `realmd`, `SSSD` und Kerberos verwendet.
Die Domänenumgebung:

| Einstellung             | Wert                     |
| ----------------------- | ------------------------ |
| Active-Directory-Domäne | `cl.lab.test`            |
| Domain Controller       | `dc-ws-20-3.cl.lab.test` |
| IP-Adresse              | `10.0.20.3`              |

#### 2.2.2 Domäne erkennen

Zunächst wird geprüft, ob Linux Mint die Domäne erkennen kann. Die Ausgabe zeigt Informationen zur Active-Directory-Domäne anzeigen.

```
realm discover cl.lab.test
```

#### 2.2.3 Domain Join durchführen

Der Beitritt zur Domäne erfolgt über das Terminal. Anschließend wird das Passwort des AD-Administrators eingegeben.

```
sudo realm join cl.lab.test -U administrator
```

#### 2.2.4 Domain Join überprüfen

Mit folgendem Befehl wird der Status der Domänenanbindung kontrolliert:

```
realm list
```

Zusätzlich kann geprüft werden, ob ein Lokale Konto aufgelöst wird:

```
getent passwd 'administrator@cl.lab.test'
```

Eine erfolgreiche Ausgabe bestätigt, dass der Benutzer über die Domänenanbindung gefunden werden kann.

#### 2.2.5 Anmeldung mit Domänenkonten

Nach erfolgreicher Integration können sich berechtigte AD-Benutzer am Linux-System anmelden.
Beispiel für das AD-Administratorkonto:

```
administrator@cl.lab.test
```

Die Anmeldung erfolgt mit dem jeweiligen Active-Directory-Passwort.

### 2.3 Remote Administration

#### 2.3.1 SSH-Server aktivieren

Der SSH-Dienst ermöglicht die Administration der Linux-VM über eine Terminalverbindung.  
Status des SSH-Dienstes überprüfen: Die SSH‑Verbindung wird schließlich für VM‑Installation und -Konfiguration verwendet, da der AD‑Administrator dafür nicht berechtigt ist.
Der SSH-Dienst ist für den Remote-Zugriff aktiviert und verwendet standardmäßig den TCP-Port `22`.
```
sudo systemctl status ssh
```

**Anmeldung über SSH testen**

Die Verbindung istvon der Jump-VM aus getestet. Die Anmeldung erfolgt mit dem Passwort des entsprechenden lokale Konto.

```
ssh -l 'ladministrator@cl.lab.test' cl2-lm-30.cl.lab.test
```

**SSH-Alias auf der Jump-VM**

Damit die Verbindung nicht jedes Mal vollständig eingegeben werden muss, wurde auf der Jump-VM in `~/.ssh/config` ein Alias eingerichtet. Die Berechtigungen der SSH-Konfigurationsdatei ist eingeschränkt. Anschließend kann die Verbindung mit Alias hergestellt werden
Da der Alias den DNS-Namen verwendet, bleibt der SSH-Verbindungsbefehl auch bei einer Änderung der IP-Adresse gleich. Voraussetzung ist, dass der DNS-Eintrag korrekt aktualisiert wird.

```
Host linux
    HostName cl2-lm-30.cl.lab.test
    User ladministrator@cl.lab.test
```
 
`chmod 600 ~/.ssh/config`

`ssh linux`

#### 2.3.2 RDP-Verbindung – Linux Mint mit Active Directory

**xrdp installieren und aktivieren**
Auf der Linux-Mint-VM als lokaler Administrator ausführen. Den xrdp-Benutzer ist zur Gruppe `ssl-cert` hinzufügen. Dienst ist überprüft

```
sudo apt install xrdp
sudo systemctl enable --now xrdp
```

```
sudo adduser xrdp ssl-cert
sudo systemctl restart xrdp
```

```
sudo systemctl status xrdp
```


**AD-Anmeldung über SSSD konfigurieren**
In der Konfigurationsdatei Im Abschnitt `[domain/cl.lab.test]` folgende Zeile ergänzen. 
- Konfiguration ist überprüft und SSSD neu gestartet.
- AD-Anmeldung ist überprüft.
- Erwartetes Ergebnis:

```
sudo nano /etc/sssd/sssd.conf
```

```
ad_gpo_map_remote_interactive = +xrdp-sesman
```

```
sudo sssctl config-check
sudo systemctl restart sssd
```

```
sudo sssctl user-checks 'administrator@cl.lab.test' -a acct -s xrdp-sesman
```

```
pam_acct_mgmt: Success
```

**Remmina auf Jump-Host konfigurieren**

Verbindungsparameter:

| Einstellung       | Wert                        |
| ----------------- | --------------------------- |
| Protokoll         | RDP                         |
| Server            | `cl2-lm-30.cl.lab.test`     |
| Benutzername      | `administrator@cl.lab.test` |
| Passwort          | AD-Passwort                 |
| Domain            | Leer                        |
| Session           | Xorg                        |
| Security protocol | RDP protocol security       |

**Verbindung testen**

Verbindung in Remmina starten und mit dem AD-Benutzer anmelden.

**Hinweis zur Sicherheit:** RDP protocol security verwenden wir derzeit nur als funktionierende Übergangslösung, da die TLS-Aushandlung noch Probleme verursacht. Vor einem dauerhaften Betrieb sollte die Verbindung auf TLS umgestellt werden.

**Wichtige Prüfungsbefehle**

```
sudo systemctl status xrdp
sudo ss -ltnp | grep 3389
sudo sssctl user-checks 'administrator@cl.lab.test' -a acct -s xrdp-sesman
```

**Ergebnis:** Die Linux-Mint-VM ist über Remmina per RDP erreichbar und authentifiziert Domänenbenutzer über Active Directory und SSSD.

</details>

### 2.3 Weitere Dienste

# Tests
