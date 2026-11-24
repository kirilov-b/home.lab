---
layout: default
title: "[SRV] VM – Domain Controller [SRV]"
---

# VM – Windows Domain Controller

**VM Parameter**

|      **Parameter** | Wert              | **Parameter** | Wert        |
| -----------------: | ----------------- | ------------: | ----------- |
|          **VM-ID** | 203               |      **IPv4** | 10.0.20.3   |
|       **vLAN Tag** | 20                |    **Domain** | cl.lab.test |
|        **VM-Name** | dc-ws-20-3        |     **Größe** | 70 GB       |
|          **Zweck** | AD, DNS, DHCP     |       **RAM** | 8 GB        |
| **Betriebssystem** | Windows Server 25 |      **vCPU** | 4           |

# Übersicht

Der Domain Controller bildet die zentrale Identitäts- und Netzwerkverwaltungsinstanz unserer Lab-Umgebung. Er basiert auf Windows Server 2025 und stellt die Dienste **Active Directory Domain Services (AD DS), DNS und DHCP** bereit.

Die Umgebung simuliert die IT-Infrastruktur eines fiktiven Unternehmens. Benutzer, Computer und Gruppen werden zentral verwaltet und in einer hierarchischen Organisationsstruktur abgebildet.

### Aufgaben in unserer Umgebung
- **Active Directory:** Zentrale Verwaltung von Benutzern, Computern, Gruppen und Domänenmitgliedschaften.
- **DNS:** Namensauflösung innerhalb der Domäne und Bereitstellung der benötigten AD-DNS-Einträge.
- **DHCP:** Automatische IP-Adressvergabe für Clients im VLAN 30.
- **Zeitsynchronisierung:** Bereitstellung der Domänenzeit für Windows-Clients.
- **Windows Server Backup:** Sicherung des System State zur Wiederherstellung wichtiger Active-Directory-Komponenten.
- **Remote Administration:** Verwaltung des Servers und der Windows-Clients über SSH und optional RDP.

# Integration und Konfiguration

## Integration 

Folgende Integrationsschritte wurden durchgeführt bevor man mit der Konfiguration des Dienstes beginnt.

1. Einbinden in das geplante Netzwerk
2. Einrichtung von SSH und RDP Verbindung mit dem Jump VM
3. Integration in Zabbix Monitoring System
4. Integration in Wazuh Security Monitoringsystem


## Konfiguration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Konfigurationsschirtte anzeigen</span></strong></summary>
  

**1. In Proxmox** - Setzen den VLAN Tag: **20**

**2. Windows Server installieren:** Windows Server 2025 Standard  
Evaluation mit Desktop Experience installieren, Hostname setzen und  eine statische IP konfigurieren.
- Uhrzeit und Zeitzone Richtig einstellen - Der Server Dient als Master Uhr für alle andere Clients

### Wichtige Ports für Domain - Client Verbindung

Der Domain Controller befindet sich ein andere vLAN als die Clients deswegen auf dem Router (OPNsense) müssen folgende Ports erlaubt werden um die Verbindung zu ermöglichen.

**TCP / UDP**
1. **53** - DNS Abfragen und größere DNS-Antworten
2. **88** - Kerberos Authentifizierung
3. **123** - NTP - Network Time Protocol (Zeitsynchronisation)
4. **135** - RPC - Remote Procedure Call 
5. **389** - LDAP - Verzeichnisabfragen und DC-Ermittlung
6. **445** - SMB - Server Message Block - SYSVOL und Gruppenrichtlinien
7. **464** - Kerberos - Passwortänderung und Verwaltung
8. **49152:65535** TCP Dynamische RPC-Ports bei  modernen Windows-Versionen;  für bestimmte  AD-/Netlogon-Funktionen  relevant

Dieser Abschnitt dient als Anleitung für die Konfiguration über **Server Manager (GUI)** und die  **PowerShell** Variante.

### 1. Server vorbereiten
#### 1.1 Hostname setzen

1. Öffne **Server Manager > Local Server**.
2. Klicke neben **Computer name** auf den aktuellen Namen.
3. Klicke auf **Change**, trage `dc-ws-20-3` ein und bestätige.
4. Starte den Server neu, falls Windows dazu auffordert.

#### 1.2 Statische IPv4-Adresse konfigurieren

1. Öffne **Server Manager > Local Server**.
2. Klicke neben **Ethernet** auf den Netzwerk-Link.
3. Rechtsklick auf den aktiven Adapter > **Properties**.
4. Öffne **Internet Protocol Version 4 (TCP/IPv4) > Properties**.
5. Wähle **Use the following IP address** und trage ein:
    - IP: `10.0.20.3`
    - Subnet mask: `255.255.255.0`
    - Default gateway: `10.0.20.1`
6. Wähle **Use the following DNS server addresses**:
    - Preferred DNS: `10.0.20.3`
    - Alternate DNS: leer lassen
7. Bestätigen.

### 2. Active Directory und DNS 

#### 2.1 Rollen installieren

1. Öffne **Server Manager > Manage > Add Roles and Features**.
2. Wähle **Role-based or feature-based installation**.
3. Wähle den lokalen Server.
4. Markiere **Active Directory Domain Services** und bestätige **Add  
    Features**.
5. Markiere **DNS Server** und bestätige erforderliche Features.
6. Klicke **Next** bis **Install**, dann installiere die Rollen.

#### 2.2 Neue Domäne erstellen

1. Im Server Manager auf das Benachrichtigungsfähnchen klicken.
2. **Promote this server to a domain controller** wählen.
3. **Add a new forest** wählen und als Root-Domäne `cl.lab.test`  
    eintragen.
4. Unter **Domain Controller Options** DNS Server und Global Catalog  
    aktiviert lassen.
5. Ein DSRM-Kennwort setzen und sicher aufbewahren.
6. Prüfen, dass der NetBIOS-Name `CL` lautet.
7. Durch die restlichen Seiten gehen; Voraussetzungen prüfen und  
    **Install** wählen.
8. Der Server startet normalerweise automatisch neu.
9. Danach als `CL\\Administrator` oder `Administrator@cl.lab.test`  
    anmelden.

#### 2.3 AD/DNS prüfen

- **Server Manager > Tools > Active Directory Users and Computers**  
    öffnen und `cl.lab.test` prüfen.
- **Server Manager > Tools > DNS** öffnen.
- Unter **Forward Lookup Zones** die Zone `cl.lab.test` und die  
    AD-DNS-Einträge kontrollieren.









<details markdown="1">
  <summary><strong><span style="color: #00813F;">PowerShell Konfiguration anzeigen</span></strong></summary>
  
#### Hostname und statische IP

Setze den gewünschten Hostnamen:

```
Rename-Computer -NewName "dc-ws-20-3" -Restart
```

Nach dem Neustart konfigurierst wird die statische IP über die  
Netzwerkeinstellungen oder mit  
`New-NetIPAddress`/`Set-DnsClientServerAddress`. Vorhandene  
IP-Konfiguration prüfen:

```
Get-NetIPConfiguration
Get-NetIPAddress -AddressFamily IPv4
Get-DnsClientServerAddress -AddressFamily IPv4
```

**Achtung:** Vor `New-NetIPAddress` die korrekte Netzwerkschnittstelle  
mit `Get-NetAdapter` ermitteln. Keine zweite, widersprüchliche  
IP-Konfiguration auf derselben Schnittstelle anlegen.

#### DNS auf den DC zeigen lassen

Nachdem die statische IP korrekt konfiguriert ist, den bevorzugten  
DNS-Server der DC-Netzwerkkarte auf `10.0.20.3` setzen. Den  
tatsächlichen Interface-Alias vorher mit `Get-NetAdapter` ermitteln:

```
Get-NetAdapter
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.20.3
```

Ersetze `"Ethernet"`, falls der Adapter anders heißt.

</details>

### 3. DHCP installieren und konfigurieren

#### 3.1 DHCP-Rolle installieren

1. **Server Manager > Manage > Add Roles and Features** öffnen.
2. **DHCP Server** markieren und erforderliche Features hinzufügen.
3. **Next > Install** wählen.
4. Nach der Installation die DHCP-Nachkonfiguration im Server Manager  
    abschließen, falls sie angezeigt wird.

#### 3.2 DHCP-Server autorisieren

1. Öffne **Server Manager > Tools > DHCP**.
2. Rechtsklick auf `dc-ws-20-3.cl.lab.test`.
3. **Authorize** wählen, falls noch nicht autorisiert.
4. **Refresh** wählen und Status kontrollieren.

#### 3.3 IPv4-Scope erstellen

1. Im DHCP-Manager den Server erweitern.
2. Rechtsklick auf **IPv4 > New Scope**.
3. Name: `VLAN30-Clients`.
4. Start-IP `10.0.30.100`, End-IP `10.0.30.200`.
5. Subnetzmaske `255.255.255.0`.
6. Ausschlussbereiche nur bei Bedarf eintragen.
7. Lease Duration festlegen (im bisherigen Lab waren es 8 Tage).
8. **Configure these options now** wählen.
9. Router: `10.0.30.1`.
10. Domain Name: `cl.lab.test`.
11. DNS Server: `10.0.20.3`.
12. WINS leer lassen, sofern nicht benötigt.
13. Scope-Optionen prüfen und den Scope anschließend mit Rechtsklick >  
    **Activate** aktivieren.

#### 3.4 Scope-Optionen kontrollieren

Unter **IPv4 > VLAN30-Clients > Scope Options** prüfen:

- `003 Router`: `10.0.30.1`
- `006 DNS Servers`: `10.0.20.3`
- `015 DNS Domain Name`: `cl.lab.test`

### 4. Windows Server Backup

- #### 1. Installation

    - Server Manager > Manage > Add Roles and Features > Next
    - Role-Based or Feature ... > Next
    - DC.lab.test auswählen > Next
    - Features > **Windows Server Backup** aktivieren
    - Next > Install

- #### 2. Virtuelle Festplatte erstellen/konfigurieren
    
    - In Proxmox unter Hardware eine Hard Disk erstellen
    - Parameter: local-lvm, **70GB**
    - DC neustarten
    - Create and Format Hard Disk Partition
    - Festplatte initialisieren, Formatieren auf NTFS, Name: **SSB (B:)**

- #### 3. System State Backup einrichten
    
    - Pfad - *Server Manager > Tools > Windows Server Backup > Local Backup > (rechts) Backup* Once > Select Item for Backup > Add Items > **System State**
    - Testen - *Recover > Get Started* - Hier wird den Backup Quelle gezeigt.
    - Ein Schedule Backup könnte man einrichten. Für meine Zwecke ist es nicht nötig, da der Server nicht dauerhaft an ist.


</details>

# Anwendung 

## Active Directory
Eine fiktives Unternehmen und die Hierarchische Struktur

### 1. Unternehmensstruktur
### 1.2. OU-Struktur
Benutzer, Computer und Gruppen sind nach Abteilungen getrennt. GPOs können gezielt an passende OUs gebunden werden. OUs sind organisatorische Container und ersetzen keine Berechtigungen.

### 1. 3. Benutzerkonten

| Abteilung   | Anzeigename | Anmeldename           |
| ----------- | ----------- | --------------------- |
| Management  | Thomas      | `thomas.management`   |
| Management  | Maria       | `maria.management`    |
| Buchhaltung | Anna        | `anna.buchhaltung`    |
| Buchhaltung | Petra       | `petra.buchhaltung`   |
| IT          | Stefan      | `stefan.it`           |
| IT          | Julia       | `julia.it`            |
| Mitarbeiter | Michael     | `michael.mitarbeiter` |
| Mitarbeiter | Lisa        | `lisa.mitarbeiter`    |
| Mitarbeiter | Daniel      | `daniel.mitarbeiter`  |
| Mitarbeiter | Sophie      | `sophie.mitarbeiter`  |

### 1.4. Gruppenmodell (AGDLP)
**Prinzip:** Benutzer → globale Gruppe (GG) → domänenlokale Berechtigungsgruppe (DL) → Ressourcenrechte.

#### Globale Sicherheitsgruppen
Alle GG-Gruppen sind als **Global / Security** angelegt.

| Gruppe               | Mitglieder                    |
| -------------------- | ----------------------------- |
| `BSI-GG-Management`  | Thomas, Maria                 |
| `BSI-GG-Buchhaltung` | Anna, Petra                   |
| `BSI-GG-IT`          | Stefan, Julia                 |
| `BSI-GG-Mitarbeiter` | Michael, Lisa, Daniel, Sophie |
| `BSI-GG-Alle`        | alle zehn Benutzer            |

**Nutzen:** Abteilungsmitgliedschaften und Ressourcenrechte werden getrennt verwaltet. Neue Benutzer benötigen die passende GG-Mitgliedschaft; allgemeiner Zugriff setzt die Mitgliedschaft in `BSI-GG-Alle` voraus.














