---
layout: default
title: "[CL] VM – MacOS - Testclient - 3"
---
# VM – macOS Sonoma (Testclient)

**VM Parameter**

|      **Parameter** | Wert         |         **Parameter** | Wert           |
| -----------------: | ------------ | --------------------: | -------------- |
|          **VM-ID** | 303          |         **IPv4 DHCP** | 10.0.30.xxx    |
|       **vLAN Tag** | 30           |            **Domain** | cl.lab.test    |
|        **VM-Name** | cl3-mac-30   |       **Hard Disk** * | 80 GB (SATA)   |
|          **Zweck** | Testclient   |               **RAM** | 8 GB           |
| **Betriebssystem** | MacOS Sonoma |              **vCPU** | 2              |
|           **BIOS** | OVMF/UEFI    | **Netzwerkadapter** * | VMware vmxnet3 |


# Übersicht

Zweck diese Mac Testclient ist unsere Domäne Umgebung aus der MacOS Perspektive zu betrachten. Zusäztlich dient die VM als Testumgebung für das MacOS Betriebsystem selbst. Folgende Punkte zu üben.

- **Integration eine MacOS in der Proxmox Umgebung**
  Recherche, Vorbereitung und Ausführung der Installation
- **Integration in der VM in unsere Netzwerk**
  Netzwerk Grundkonfiguration für MacOS - Ziel ist Plug an Play via DHCP
- **Integration der VM in der Domäne**
  Domäne beitreten und Auswirkung der Active Direktory  auf Mac Geräte

# Konfigurationsschritte 

## 1. Vorbereitung

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Vorbereitungsschritte anzeigen</span></strong></summary>

### 1.1 Long QT-OpenCore - v07.iso 

Für Proxmox/QEMU wurde das vorgefertigte Projekt **LongQT-sea/OpenCore-ISO** (GitHub) verwendet. Das Repository stellt ein speziell für Proxmox VE und QEMU/KVM vorbereitetes, bootfähiges OpenCore-CD/DVD-Image bereit.
OpenCore wird vor dem eigentlichen macOS-Start geladen und bereitet die Startumgebung so vor, dass macOS auf der bereitgestellten virtuellen Hardware starten kann.
Es bildet die benötigte Mac-Startumgebung über eine konfigurierbare EFI-Schicht nach beziehungsweise stellt macOS passende Plattforminformationen und Treiber bereit.

Die OpenCore ISO Datei wird an einem CD/DVD Laufwerk in der Proxmox VM Konfigurtion angeähngt. Diese Laufwerk muss in der Bootsequenz ganz oben stehen.
Der Startablauf sieht vereinfacht so aus:

1. Proxmox startet die VM mit OVMF als virtueller UEFI-Firmware.
2. Die UEFI-Firmware findet das OpenCore-CD/DVD-Laufwerk und startet `EFI/BOOT/BOOTx64.efi`.
3. Dieser EFI-Loader startet OpenCore.
4. OpenCore liest `EFI/OC/config.plist`, lädt die aktivierten UEFI-Treiber und ACPI-Tabellen und stellt die konfigurierten Plattforminformationen bereit.
5. Das OpenCore-Auswahlmenü zeigt verfügbare Startziele an.
6. Über das Menü wird der macOS-Installer auf der zweiten virtuellen DVD gestartet.
7. Danach übernimmt Apples eigener Bootloader `boot.efi` und startet den macOS-Installationsprozess.

Einrichtung das System Booten ohne den OpenCore Laufwerk:
Die Einrichtung wird in der Dokumentation von LonQT-sea/OpenCore-ISO bechrieben.
Damit macOS Sonoma ohne separate OpenCore-ISO starten kann, haben wir OpenCore auf der EFI-Partition der virtuellen Systemfestplatte installiert. Der Script **Mount-EFI.command** wurde ausgeführt und die **Boot** und **OS** aus der OpenCore Laufwerk auf der EFI Systemfestplatte kopiert.
### 1.2 macOS-Installationsmedium über GitHub erstellen

Für die Erstellung des Installationsmediums wurde das GitHub-Projekt **LongQT-sea/macos-iso-builder** verwendet. Das GitHub Projekt lädt den vollständigen macOS-Installer von Apples Servern herunter und erstellt daraus ein bootfähiges ISO-Image. Die Erstellung kann über GitHub Actions ausgeführt werden, sodass dafür kein eigener Mac benötigt wird. 

- ISO: **macOS_Sonoma_14.8.9.iso**
- Größe: **14.9 GB**

Die MacOS ISO wird an einem CD/DVD Laufwerk in der Proxmox VM Konfigurationa angehängt. Diese Laufwerk steht an der zweite Stelle in der Bootreihenfolge.

Beim Booten wird zuerst **Disk Utility** gestartet, um die Festplatte in Mac geeignete Format zu konfigurieren.

- Name: **Mac HardDisk** 
- Format: **APFS** 
- Scheme: **GUID Partition Map** 

Nach dem Formatieren wird die Installation von MacOS Sonoma gestartet und klassisch durchgefüht.

### 1.3 Proxmox VM Konfiguration Besonderheiten

- **Hard Disk** * : "SATA 0" (anstatt "virtio-scsi-single") - Sonst ist die Festplatte nicht sichtbar
- **Proxmox SecureBoot**: "Aus"
- **EFI SecureBoot**: "Aus"
- **PX Firewall**: "Aus" - Führt nach gewisse Zeit zum Absturz des Systems wenn aktiv ist
- **CPU**: "host" - Die "x86-64-v2-AES" führt zu ein Kernel Panic
- **Netzwerkadapter** * : "VMware vmxnet3" (statt "Intel E1000") - sonst funktioniert die ACK der VM bei der Verbindung mit DHCP Server nicht.


</details>


## 2. Integration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Integrationsschritte anzeigen</span></strong></summary>


### 2.1 Domain Join
Das Beitreten in unsere Active Directory "client" Domäne geht ziemlich einfach an MacOS
#### 2.1.1 Voraussetzungen prüfen
Vor dem Domain Join wurde die DNS-Konfiguration unter *Systemeinstellungen → Netzwerk → Ethernet → Details → DNS* kontrolliert.

| Einstellung | Wert          |
| ----------- | ------------- |
| IP-Adresse  | DHCP          |
| DNS-Server  | `10.0.20.3`   |
| Suchdomäne  | `cl.lab.test` |
| Gateway     | `10.0.30.1`   |

Anschließend wurde die DNS-Auflösung im Terminal getestet:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.cl.lab.test
```

Der Domain Controller `dc-ws-20-3.cl.lab.test` wurde erfolgreich gefunden.

#### 2.1.2. Active Directory Domain Join
Der Domain Join wurde über die grafische Oberfläche durchgeführt:
*Directory Utility > Sercice > Active Directory > Edit* 

- Active Directory Domain: `cl.lab.test`
- Computer ID: `cl3-mac-30`

Klicken auf **Bind** zu betreten. Authentifizieren über den Administrator Konto von der Domain Controller (Username / Password) und danach dern Vorgang bestätigen.

#### 2.1.3 Domain Join überprüfen

**Grafische Überprüfung**
Unter *Directory Utility → Services → Active Directory* wurden folgende Angaben angezeigt:

- Active Directory Forest: `cl.lab.test`
- Active Directory Domain: `cl.lab.test`
- Computer ID: `cl3-mac-30`

Die Schaltfläche **Unbind** bestätigte die bestehende Domain-Bindung. Zusätzlich wurde unter *Users & Groups → Network account server* der Status der Domäne angezeigt.

**Überprüfung über Terminal**

Eingabe:

```bash
dsconfigad -show
```

Die Ausgabe bestätigte:

```text
Active Directory Forest  = cl.lab.test
Active Directory Domain  = cl.lab.test
Computer Account         = cl3-mac-30$
```

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Troubleshooting anzeigen</span></strong></summary>


#### Probleme
- Anmeldung mit andere Users war nicht auswählbar.
- Am Anmeldebildschirm erschien jedoch „Network Accounts are unavailable“.
- AD-Benutzer konnten zunächst nicht aufgelöst werden.
- Der Directory Editor meldete Fehler `2100`.
- Kerberos funktionierte bereits, aber die Benutzerauflösung über macOS Directory Services bereitete Probleme.

#### Lösung

- **Anmeldefenster auf Benutzername und Passwort umstellen**
Zunächst wurde über das Terminal die Anzeige des macOS-Anmeldefensters angepasst:                    Diese Option war in der GUI nicht sichtbar deswegen mit dem Befehl.                                                                                                                                                                                                                                               
```
sudo defaults write /Library/Preferences/com.apple.loginwindow SHOWFULLNAME -bool true
```

- **Active Directory in Directory Utility konfigurieren**
Den Eintrag aus *Directory Utility → Search policy → Authentication→ Edit*  **/Active Directory/CL/All Domains** müsste gelöscht werden und mit **/Active Directory/CL/cl.lab.test** ersetzt werden.
- *Directory Utility > Services > Active Directory > Show Opitons > Administrative*
  Hier müsste **Allow authentication from any Domain in the forest** deaktiviert werden

</details>

### 2.2 SSH & VNC (statt RDP)

#### 2.2.1 SSH-Server aktivieren
  Auf dem Mac unter *System Settings → General → Sharing:*

1. **Remote Login** aktiviert.
2. Zugriff auf berechtigte Benutzer beschränkt; der AD-Benutzer `administrator` wurde zugelassen.
3. SSH hört auf TCP-Port `22`.
#### 2.2.2 Screen Sharing / VNC aktivieren
Unter *System Settings → General → Sharing → Screen Sharing:*

1. **Screen Sharing** aktiviert.
2. Zugriff auf ausgewählte Benutzer beschränkt; der AD-Benutzer `administrator` wurde zugelassen.
3. Der Zugriff wurde auf ausgewählte Benutzer beschränkt; der AD-Benutzer `administrator` wurde freigegeben.
4. Die Remmina-Anmeldung mit den AD-Zugangsdaten von `administrator` war erfolgreich.

Der VNC-Zugriff öffnet den macOS-Anmeldebildschirm. Die anschließende Anmeldung erfolgt mit dem jeweiligen macOS-Konto. Die Anmeldung soll weiterhin für den lokalen macOS-Administrator sowie die AD-Konten `administrator` und `Users` möglich bleiben.

**Sicherheitshinweis:** Klassisches VNC bietet keine verlässliche Ende-zu-Ende-Transportverschlüsselung. Den Zugriff auf TCP-Port 5900 daher auf vertrauenswürdige Quellen beschränken oder bei Bedarf über einen geschützten SSH-Tunnel führen.

#### 2.2.3 Jump-VM konfigurieren
**Remmina für VNC**  - In Remmina wurde ein Verbindungsprofil angelegt:

- Protokoll: **VNC Plugin** (nicht VNC Listener Plugin)
- Server: `cl3-mac-30.cl.lab.test:5900`
- Verbindung: vom Jump-Host zum macOS Screen Sharing-Dienst

MacOS unterstütz nativ den RDP Verbindung stattdessen verwendet **VNC** (*Virtual Network Computing)*. Der VNC-Dienst des Macs ist über TCP-Port `5900` erreichbar. 

- Auf der Jump-VM wurde in `~/.ssh/config` ein Alias angelegt:

```
Host mac
    HostName cl3-mac-30.cl.lab.test
    User ladministrator
```

- Der vorhandene Ed25519-Schlüssel der Jump-VM wurde weiterverwendet; es wurde kein neues Schlüsselpaar erstellt: Öffentlicher Schlüssel: `~/.ssh/id_ed25519.pub` – wurde auf dem Mac beim AD-Benutzer `administrator` hinterlegt.

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub mac
```

Danach funktionierte `ssh mac` ohne Passwortabfrage. Die Passwortanmeldung bleibt als alternative Anmeldemöglichkeit bestehen.



</details>


### 2.3 Weitere Dienste


<details markdown="1">
  <summary><strong><span style="color: #00813F;">Dienste anzeigen</span></strong></summary>

#### Zabbix Monitoring

</details>
# Tests
