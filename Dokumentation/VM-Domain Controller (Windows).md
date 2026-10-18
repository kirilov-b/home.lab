---
layout: default
title: "[SRV] VM – Domain Controller [SRV]"
---

# VM – Windows Domain Controller

## Ziel und Lab-Übersicht

Diese Dokumentation beschreibt den Aufbau des Labs mit Windows Server  
2025, **Active Directory, DNS, DHCP** und **Windows Server Backup** sowie Windows-Clients in getrennten  
VLANs.

**VM Parameter**

|      **Parameter** | Wert              | **Parameter** | Wert        |
| -----------------: | ----------------- | ------------: | ----------- |
|          **VM-ID** | 203               |      **IPv4** | 10.10.20.3  |
|       **vLAN Tag** | 20                |    **Domain** | cl.lab.test |
|        **VM-Name** | dc-ws-20-3        |     **Größe** | 70 GB       |
|          **Zweck** | AD, DNS, DHCP     |       **RAM** | 8 GB        |
| **Betriebssystem** | Windows Server 25 |      **vCPU** | 4           |

# Grundkonfiguration

**1. In Proxmox** - Setzen den VLAN Tag: **20**

**2. Windows Server installieren:** Windows Server 2025 Standard  
Evaluation mit Desktop Experience installieren, Hostname setzen und  
eine statische IP konfigurieren.
- Uhrzeit und Zeitzone Richtig einstellen - Der Server Dient als Master Uhr für alle andere Clients

## Wichtige Ports für Domain - Client Verbindung

Der Domain Controller befindet sich ein andere vLAN als die Clientes deswegen auf dem Router (OPNsense) müssen folgende Ports erlaubt werden um die Verbindung zu ermöglichen.

**TCP / UDP**
1. **53** - DNS Abfragen und großere DNS Antworten
2. **88** - Kerberos Authentifizierung
3. **123** - NTP - Network Time Protokoll (Zeitsynchronisation)
4. **135** - RPC - Remote Procedure Call 
5. **389** - LDAP - Verzeichnisabfragen und DC Emittlung
6. **445** - SMB - Server Message Block - SYSVOL und Gruppenrichlinien
7. **464** - Kerberos - Passwortänderung und Verwaltung
8. **49152:65535** TCP Dynamische RPC-Ports bei  modernen Windows-Versionen;  für bestimmte  AD-/Netlogon-Funktionen  relevant


# Unternehmenstruktur
Eine Fiktive Unternehmen und die Hierarchische Struktur

## 2. Active Directory Struktur

<details markdown="1">
  <summary><strong><span style="color: #00813F;">AD-Konfiguration anzeigen</span></strong></summary>
### Active Directory konfigurieren.

1. **Erstellen von Organisational Unit (OU)**
    - LAB-Users
    - LAB-Admins
    - LAB-Groups
    - LAB-Servers
    - LAB-Computers
        - Verschieben von **cl.lab.home\Computers** zu cl.lab.home\LAB-Computers**
2. **Erstellen von Gruppen (AD Users and Computers)**
    - GG-Mitarbeiter - Global, Security
        - max.mustermann - testuser
    - GG-IT-Admins - Global,Security - Administrative Rechte
    - GG-IT-Helpdesk - Global,Security
    - GG-Buchhaltung - Global,Security
    - GG-Management - Global,Security
    - GG-Produktion - Global,Security
3. **GPO - (Group Policy Objects)**
    - **GPO-LAB-Windows-Firewall**
        - **Action 1** - Firewall State: On
            - Firewall State: On
            - Inbound Connection: Block (default)
            - Outbound Connection: Allow (default)
        - **Action 2** - RDP Erlauben
            - Rule Type: **Port**
            - Protocol: **TCP**
            - Specific local Port: **3389**
            - Action: **Allow the connection**
            - Profile: **Domain aktivieren**
            - Name: **LAB-Allow RDP**
    - **GPO-LAB-Helpdesk-Workstations**
        - Ziel - Einlogen von Helpdesk an Clients per Remote Desktop ohne lokalen Administrationsrechte
        - Pfad: GPM > LAB-Computers > Create GPO / Computer Configuration > Preferences > Control Panel Settings > Local Users and Groups
    - **GPO-LAB-RemoteDesktop**
        - Ziel - Aktiviert den Remote Desktop (RDP) auf den zugewiesenen Windows-Client. Dadurch können berechtigte Benutzer (z.B. Mitglieder der Gruppe GG-IT-Helpdesk), per RDP auf die Clients zugreifen und remote administrieren.
        - Pfad: GPM > LAB-Computers > Create GPO / Computer Configuration > Policies > Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Sessions Host > Connections
        - Action: **Allow users to connect remotely by using Remote Desktop Services - Enabled**
    - **GPO-LAB-User-Background**
        - Ziel - Setzt ein Hintergrundbild anhand der Gruppe, in der der Benutzer sich befindet. (z.B. GG-Produktion > Produktion-Bild)
        - Datei in **Sorce Ordner** kopieren.
        - Unter _GPO > Users Configuragtion > Preferences > Windwos Settings > Files_ die Datei auswählen aus der Sorce Ordner und **Destination Ordner** auswählen _Public > Public Pictures_
        - Registry anpassen:  
            Hive: HKEY_CURRENT_USER  
            Key Path: Control Panel\Desktop  
            Value Name: Wallpaper  
            Value Type: REG_SZ  
            Value Data: ... \Public\Public Pictures\
            - Gruppenzuordnung: Unter Common die Gruppe zuweisen
    - **GPO-LAB-Local-IT-Admins**
        - Ziel: GG-IT-Admin Administrationsrechte für alle Clients zu vergeben.
        - Pfad: LAB-Computers > Create GPO ... > Edit > Computer Configuration > Preferences > Control Pannel Settings > Local Users and Groups >
        - New > Local Group
            - Action: Update
            - Group Name: Administrators (built-in)
            - Member: GG-IT-Admins
        - Testen: Via PS: "net localgroup Administrators" > soll GG-IT-Admins zu sehen sein

</details>

## 3. File Share Struktur

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Share Konfiguration anzeigen</span></strong></summary>

- **1.Ordnerstruktur:**
    - C:\Shares
        - C:\Shares\Buchhaltung
        - C:\Shares\Produktion
        - C:\Shares\Management
        - C:\Shares\IT
- **2. Berechtigung geben**
    - *Buchhaltung Ordner > Properties > Security > Advanced (Weiter für die andere Ordner)
        - Alle "User..." entfernen
        - Disable inheritance > Convert inheritance permissions ...
        - Addieren die Gruppe GG-Buchhaltung mit "modify" Privileges. GG-IT-Admins - Full Controll
- **3. Network Share erstellen - SMB Freigabe**
    - _Buchhaltung Ordner > Properties > Share > Advance Share (weiter für die andere Ordner)_
        - Share Folder
        - Permission > GG-Buchhaltung addieren, GG-IT-Admins - Full Controll
        - Permission > Everyone löschen, Change und Modify auswählen.  
            Zusammenfassung: Jede Gruppe hat nur Zugriff auf seine Abteilung. Nur Admin hat Full Controll auf alle Ordner.
- **4.GPO für Automatische Drive Erkennung (pro Abteilung)**
    - LAB-Users > GPO-LAB-ShareDrive > Edit > User Config... > Preferences > Windows Settings > DriveMaps
    - New > **Mapped Drive**
        - Action: **Update**
        - Reconnect: **Aktiv**
        - Location: **\\DC\Buchhaltung Share**
        - Lable as: **Buchhaltung**
        - Drive Letter: **B**
        - **Unter Common > Item Level Targeting > New Item > GG-Buchhaltung auswählen!**

| Benutzername       | Abteilung   | Gruppen                        |
| ------------------ | ----------- | ------------------------------ |
| Anna Helpdesk      | Helpdesk    | GG-Mitarbeiter; GG-IT-Helpdesk |
| Markus Buchhaltung | Buchhaltung | GG-Mitarbeiter; GG-Buchhaltung |
| Tom Admin          | Admin       | GG-Mitarbeiter; GG-IT-Admins   |
| Hans Buchhaltung   | Buchhaltung | GG-Mitarbeiter; GG-Buchhaltung |
| Jürgen Chief       | Management  | GG-Mitarbeiter; GG-Management  |
| Stephan Produktion | Produktion  | GG-Mitarbeiter; GG-Produktion  |
| Lara Produktion    | Produktion  | GG-Mitarbeiter; GG-Produktion  |
|                    |             |                                |
</details>

## 4. Windows Server Backup

<details markdown="1">
  <summary><strong><span style="color: #00813F;">WSB-Konfiguration anzeigen</span></strong></summary>


- #### 1. Installation

    - Server Manager > Manage > Add Roles and Features > Next
    - Role-Based or Feature ... > Next
    - DC.lab.test auswählen > Next
    - Features > **Windows Serve Backup** aktivieren
    - Next > Install

- #### 2. Virtuelle Festplatte erstellen/konfigurieren
    
    - In Proxmox unter Hardware eine Hard Disk erstellen
    - Parameter: local-lvm, **70GB**
    - DC neustarten
    - Create and Format Hard Disk Partition
    - Festplatte initialisieren, Formatieren auf NTFS, Name: **SSB (B:)**

- #### 3. System State Backup einrichten
    
    - Pfad - Server Manager > Tools > Windows Server Backup > Local Backup > (rechts) Backup Once > Select Item for Backup > Add Items > **System State**
    - Testen - Recover > Get Startet - Hier wird den Backup Quelle gezeigt.
    - Ein Schedule Backup könnte man einrichten. Für meine Zwecke ist es nicht nötig, da der Server nicht dauerhaft an ist.

</details>

## Konfigurationschritte 

Diese Abschnitt dient als Anleitung für die Konfiguration über **Server Manager (GUI)** und die  **PowerShell** Variante.


<details markdown="1">
  <summary><strong><span style="color: #00813F;">Konfiguration anzeigen</span></strong></summary>

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
    - IP: `10.10.20.3`
    - Subnet mask: `255.255.255.0`
    - Default gateway: `10.10.20.1`
6. Wähle **Use the following DNS server addresses**:
    - Preferred DNS: `10.10.20.3`
    - Alternate DNS: leer lassen
7. Bestätigen.

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
DNS-Server der DC-Netzwerkkarte auf `10.10.20.3` setzen. Den  
tatsächlichen Interface-Alias vorher mit `Get-NetAdapter` ermitteln:

```
Get-NetAdapter
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.20.3
```

Ersetze `"Ethernet"`, falls der Adapter anders heißt.

</details>

### 2. AD DS und DNS installieren

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

```
Install-WindowsFeature AD-Domain-Services, DNS -IncludeManagementTools
```

Neue Gesamtstruktur erstellen (der Server wird dabei üblicherweise neu  
gestartet):

```
Install-ADDSForest `
  -DomainName "cl.lab.test" `
  -DomainNetbiosName "CL" `
  -InstallDNS
```

Folge den Aufforderungen für das DSRM-Kennwort und den Neustart.

Nach dem Neustart prüfen:

```
Get-ADDomain
Get-ADForest
Get-Service NTDS, DNS, Netlogon
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.cl.lab.test
nltest /dsgetdc:cl.lab.test
```

Die SRV-Abfrage sollte den DC `dc-ws-20-3.cl.lab.test` mit seiner  
korrekten IP zurückgeben.

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
4. Start-IP `10.10.30.100`, End-IP `10.10.30.200`.
5. Subnetzmaske `255.255.255.0`.
6. Ausschlussbereiche nur bei Bedarf eintragen.
7. Lease Duration festlegen (im bisherigen Lab waren es 8 Tage).
8. **Configure these options now** wählen.
9. Router: `10.10.30.1`.
10. Domain Name: `cl.lab.test`.
11. DNS Server: `10.10.20.3`.
12. WINS leer lassen, sofern nicht benötigt.
13. Scope-Optionen prüfen und den Scope anschließend mit Rechtsklick >  
    **Activate** aktivieren.

#### 3.4 Scope-Optionen kontrollieren

Unter **IPv4 > VLAN30-Clients > Scope Options** prüfen:

- `003 Router`: `10.10.30.1`
- `006 DNS Servers`: `10.10.20.3`
- `015 DNS Domain Name`: `cl.lab.test`

<details markdown="1">
  <summary><strong><span style="color: #00813F;">PowerShell Konfiguration anzeigen</span></strong></summary>

DHCP-Rolle installieren:

```
Install-WindowsFeature DHCP -IncludeManagementTools
```

DHCP-Server in AD autorisieren:

```
Add-DhcpServerInDC -DnsName "dc-ws-20-3.cl.lab.test" -IPAddress 10.10.20.3
Get-DhcpServerInDC
```

Beispiel-Scope erstellen (Bereich und Netz an das eigene Lab anpassen):

```
Add-DhcpServerv4Scope `
  -Name "VLAN30-Clients" `
  -StartRange 10.10.30.100 `
  -EndRange 10.10.30.200 `
  -SubnetMask 255.255.255.0 `
  -State Inactive
```

Scope-Optionen setzen und Scope aktivieren:

```
Set-DhcpServerv4OptionValue -ScopeId 10.10.30.0 -Router 10.10.30.1
Set-DhcpServerv4OptionValue -ScopeId 10.10.30.0 -DnsServer 10.10.20.3
Set-DhcpServerv4OptionValue -ScopeId 10.10.30.0 -DnsDomain "cl.lab.test"
Set-DhcpServerv4Scope -ScopeId 10.10.30.0 -State Active
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 10.10.30.0
```

Auf OPNsense DHCP Relay auf dem Client-VLAN zum Server `10.10.20.3`  
konfigurieren. Die Firewall muss DHCP-Relay-Verkehr entsprechend  
zulassen. Die konkrete OPNsense-Konfiguration wird in deren Oberfläche  
vorgenommen.

</details>

### 4. Client prüfen und der Domäne beitreten

- OPNsense-Weboberfläche öffnen.
- **Services > DHCP Relay** öffnen (Menü kann je nach Version  
    abweichen).
- Client-Interface `VLAN_30_CLIENTS` auswählen.
- DHCP-Serverziel `10.10.20.3` eintragen.
- Relay aktivieren, speichern und anwenden.
- Status prüfen.

<details markdown="1">
  <summary><strong><span style="color: #00813F;">PowerShell Konfiguration anzeigen</span></strong></summary>

Auf dem Client zunächst prüfen:

```
ipconfig /all
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.cl.lab.test
nltest /dsgetdc:cl.lab.test
Test-NetConnection dc-ws-20-3.cl.lab.test -Port 53
Test-NetConnection dc-ws-20-3.cl.lab.test -Port 88
Test-NetConnection dc-ws-20-3.cl.lab.test -Port 123
Test-NetConnection dc-ws-20-3.cl.lab.test -Port 135
Test-NetConnection dc-ws-20-3.cl.lab.test -Port 389
Test-NetConnection dc-ws-20-3.cl.lab.test -Port 445
Test-NetConnection dc-ws-20-3.cl.lab.test -Port 464
```

Die DNS-Serveradresse des Clients muss auf `10.10.20.3` zeigen. Danach  
Domain-Join in administrativer PowerShell:

```
Add-Computer -DomainName "cl.lab.test" -Credential (Get-Credential) -Verbose
Restart-Computer
```

Nach dem Neustart mit einem Domänenkonto anmelden, z. B.  
`CL\Administrator` oder `Administrator@cl.lab.test`.

Prüfen:

```
Get-CimInstance Win32_ComputerSystem | Select-Object Name, Domain, PartOfDomain
Test-ComputerSecureChannel -Verbose
nltest /sc_query:cl.lab.test
```

Computerkonto auf dem DC kontrollieren:

```
Get-ADComputer CL1-WC-30 -Properties DNSHostName, ServicePrincipalName, Enabled |
  Format-List Name, DNSHostName, ServicePrincipalName, Enabled
```

Bei einem Fehler zuerst DNS, Zeit, Ereignisprotokolle und Erreichbarkeit  
untersuchen, bevor der Client aus der Domäne entfernt oder das  
Computerkonto gelöscht wird.

</details>

### 5. Zeitsynchronisierung prüfen

#### 5.1 DHCP/DNS prüfen

1. Am Client **Settings > Network & Internet > Advanced network  
    settings > More network adapter options** öffnen (oder `ncpa.cpl`  
    ausführen).
2. Rechtsklick auf Ethernet > **Properties**.
3. **Internet Protocol Version 4 (TCP/IPv4) > Properties** öffnen.
4. **Obtain an IP address automatically** und **Obtain DNS server  
    address automatically** auswählen.
5. In einer Eingabeaufforderung `ipconfig /all` ausführen.
6. Prüfen: IP aus dem DHCP-Bereich, Gateway `10.10.30.1`, DNS  
    `10.10.20.3`, DNS-Suffix `cl.lab.test`.

#### 5.2 Domain-Join

1. **Settings > System > About** öffnen.
2. **Domain or workgroup** bzw. **Advanced system settings > Computer  
    Name > Change** wählen (je nach Windows-Version).
3. **Change** anklicken und unter **Member of** die Option **Domain**  
    wählen.
4. `cl.lab.test` eingeben und bestätigen.
5. Berechtigte Anmeldedaten angeben, z. B. `CL\\Administrator`.
6. Die Willkommensmeldung bestätigen und den Client neu starten.
7. Am Anmeldebildschirm **Other user** wählen und `CL\\Administrator`  
    oder `Administrator@cl.lab.test` eingeben.

#### 5.3 Mitgliedschaft kontrollieren

- **Settings > System > About** öffnen und Domänenmitgliedschaft  
    prüfen.
- Auf dem DC **Active Directory Users and Computers > cl.lab.test >  
    Computers** öffnen.
- Prüfen, ob `CL1-WC-30` existiert und aktiviert ist.

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Power Shell Konfiguration anzeigen</span></strong></summary>

Auf DC und Client:

```
Get-Date
Get-TimeZone
w32tm /query /status
```

Auf dem Client die Domänenzeit neu erkennen und synchronisieren:

```
w32tm /resync /rediscover
```

Der DC sollte eine korrekte Systemzeit haben. In einem Lab kann die  
DC-Uhr lokal als Zeitquelle dienen; in produktiven Umgebungen sollte die  
Zeitquelle bewusst geplant werden. Die Zeitzone auf beiden Systemen  
passend setzen, z. B.:

```
Set-TimeZone -Id "W. Europe Standard Time"
```

</details>

### 6. Windows-Firewall für AD

- Auf DC und Client **Settings > Time & language > Date & time**  
    öffnen.
- Zeitzone auf **(UTC+01:00) Amsterdam, Berlin, Bern, Rome, Stockholm,  
    Vienna** stellen.
- Die Windows-Zeitquelle des Clients sollte in der Domäne  
    normalerweise der DC sein.
- Bei Abweichungen zuerst die tatsächliche Uhrzeit und Zeitzone auf  
    dem DC prüfen. Zeitzone und Systemuhr sind unterschiedliche Dinge;  
    die Uhrzeit nicht ohne Prüfung manuell ändern.

<details markdown="1">
  <summary><strong><span style="color: #00813F;">PowerShell Konfiguration anzeigen</span></strong></summary>

Auf OPNsense nur die benötigten Verbindungen vom Client-VLAN zum DC  
`10.10.20.3` zulassen. Für moderne Windows-RPC-Kommunikation kann  
dynamisches TCP-RPC im Bereich `49152–65535` erforderlich sein. Auf dem  
DC und Client muss die jeweilige Windows-Firewall ebenfalls die  
erforderlichen Dienste zulassen.

Vor Änderungen prüfen, welche Regeln bereits vorhanden sind:

```
Get-NetFirewallRule -Enabled True |
  Select-Object DisplayName, Profile, Direction, Action
```

</details>

### 7. OpenSSH Server auf einem Windows-Host

### 7.1 Installieren

1. **Settings > System > Optional features** öffnen.
2. **View features** bzw. **Add an optional feature** wählen.
3. **OpenSSH Server** suchen, auswählen und installieren.

### 7.2 Dienst starten

1. `services.msc` öffnen.
2. **OpenSSH SSH Server (sshd)** öffnen.
3. **Startup type** auf **Automatic** stellen.
4. **Start** klicken und Status **Running** kontrollieren.

### 7.3 Firewallregel prüfen

1. `wf.msc` öffnen.
2. **Inbound Rules** auswählen und **OpenSSH SSH Server (sshd)**  
    suchen.
3. Regel aktivieren.
4. In den Eigenschaften unter **Advanced > Profiles** das passende  
    Profil, typischerweise **Domain**, auswählen.
5. Die Regel muss TCP-Port 22 erlauben.

### 7.4 Administrator-Public-Key

1. Auf Linux Mint `cat ~/.ssh/id_ed25519.pub` ausführen.
2. Nur den öffentlichen Schlüssel kopieren; den privaten Schlüssel  
    `id_ed25519` niemals übertragen.
3. Auf Windows in  
    `C:\\ProgramData\\ssh\\administrators_authorized_keys` eintragen.
4. Die Datei muss restriktive Berechtigungen für `SYSTEM` und  
    `BUILTIN\\Administrators` haben.
5. Bestehende Schlüssel nicht überschreiben; neue Schlüssel ergänzen.

Für die zentrale Administrator-Schlüsseldatei ist PowerShell oft  
zuverlässiger als Notepad; die genaue Ein-Befehl-Variante steht in  
`Wie.md`.

### 7.5 SSH testen

Von Linux Mint:

```
ssh -l 'Administrator@cl.lab.test' 10.10.30.101
```

Mit Schlüssel:

```
ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes -l 'Administrator@cl.lab.test' 10.10.30.101
```


<details markdown="1">
  <summary><strong><span style="color: #00813F;">PowerShell Konfiguration anzeigen</span></strong></summary>

Prüfen ob OpenSSH Server installiert ist
```
Get-Service sshd
```

Falls nicht, installieren:

```
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

Dienst automatisch starten und starten:

```
Set-Service -Name sshd -StartupType Automatic
Start-Service sshd
Get-Service sshd
```

Vorhandene Standard-Firewallregel prüfen und für das Domänenprofil  
aktivieren:

```
Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" |
  Select-Object DisplayName, Enabled, Profile, Direction, Action

Set-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -Enabled True -Profile Domain
```

Falls die Regel fehlt, kann sie erstellt werden:

```
New-NetFirewallRule `
  -Name "OpenSSH-Server-In-TCP" `
  -DisplayName "OpenSSH Server (sshd)" `
  -Enabled True `
  -Direction Inbound `
  -Protocol TCP `
  -Action Allow `
  -LocalPort 22 `
  -Profile Domain
```

### Public Key für einen Domänenadministrator hinterlegen


Auf Linux Mint den **öffentlichen** Schlüssel anzeigen:

```
cat ~/.ssh/id_ed25519.pub
```

Nur die vollständige Zeile aus der `.pub`-Datei kopieren. Den privaten  
Schlüssel `~/.ssh/id_ed25519` niemals auf den Windows-Host kopieren.

Auf Windows in administrativer PowerShell die Public-Key-Zeile  
einsetzen:

```
Set-Content -Path "C:\\ProgramData\\ssh\\administrators_authorized_keys" -Value "ssh-ed25519 (Schlüssel) linuxuser@hostname"
icacls.exe "C:\\ProgramData\\ssh\\administrators_authorized_keys" /inheritance:r
icacls.exe "C:\\ProgramData\\ssh\\administrators_authorized_keys" /grant "SYSTEM:F" /grant "BUILTIN\\Administrators:F"
```

Falls `administrators_authorized_keys` bereits andere Schlüssel enthält,  
nicht einfach überschreiben: bestehende Schlüssel erhalten und den neuen  
Schlüssel ergänzen.

Von Linux Mint testen (UPN-Format für das Domänenkonto):

```
ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes -l 'Administrator@cl.lab.test' 10.10.30.101
```

Wenn SSH trotz laufendem Dienst nicht erreichbar ist, zuerst IP, Port  
22, Windows-Firewallprofil und OPNsense-Regel prüfen.

</details>

### 8. RDP auf einem Windows-Host (optional)

### 8.1 Remote Desktop aktivieren

1. Auf dem Zielhost **Settings > System > Remote Desktop** öffnen.
2. **Enable Remote Desktop** aktivieren und bestätigen.
3. Prüfen, ob der Domänenbenutzer zur RDP-Anmeldung berechtigt ist.  
    Andere Benutzer können bei Bedarf zu **Remote Desktop Users**  
    hinzugefügt werden.

### 8.2 Windows-Firewall prüfen

1. `wf.msc` öffnen.
2. Unter **Inbound Rules** die Regeln der Gruppe **Remote Desktop**  
    suchen.
3. Passende Regeln für das Domänenprofil aktivieren.
4. RDP verwendet standardmäßig TCP 3389; UDP 3389 kann optional für  
    bestimmte Funktionen genutzt werden.

### 8.3 Remmina von Linux Mint

1. Remmina öffnen und eine neue Verbindung erstellen.
2. Protocol: **RDP**.
3. Server: `10.10.30.101`.
4. Benutzername: `Administrator@cl.lab.test` (oder Benutzername plus  
    Domäne, je nach Remmina).
5. Domänenkennwort eingeben und verbinden.

Bei Problemen TCP 3389, Windows-Firewall und OPNsense-Regeln zwischen  
Jump-Host und Client prüfen.

<details markdown="1">
  <summary><strong><span style="color: #00813F;">PowerShell Konfiguration anzeigen</span></strong></summary>

Remote Desktop aktivieren und Dienst starten:

```
Set-ItemProperty `
  -Path "HKLM:\\SYSTEM\\CurrentControlSet\\Control\\Terminal Server" `
  -Name "fDenyTSConnections" -Value 0

Set-Service -Name TermService -StartupType Automatic
Start-Service -Name TermService
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

Prüfen:

```
Get-Service TermService
Get-NetTCPConnection -State Listen -LocalPort 3389
```

Von Linux Mint den TCP-Port testen:

```
nc -zv 10.10.30.101 3389
```

In Remmina RDP auswählen, als Server `10.10.30.101` eintragen und ein  
geeignetes Domänenkonto verwenden. RDP-Zugriff muss auch durch die  
Netzwerk-Firewall zwischen den VLANs erlaubt sein, falls der Jump-Host  
in einem anderen VLAN liegt.

</details>

### 9. Windows Server Backup

1. **Server Manager > Manage > Add Roles and Features** öffnen.
2. Im Wizard den lokalen Server auswählen.
3. Unter **Features** **Windows Server Backup** markieren.
4. **Next > Install** wählen.
5. Danach **Server Manager > Tools > Windows Server Backup** öffnen.
6. **Backup Schedule** für regelmäßige Sicherungen oder **Backup Once**  
    für eine einmalige Sicherung auswählen.
7. Umfang, Ziel und Zeitplan festlegen; Status kontrollieren und  
    Wiederherstellung testen.

Die Installation allein erstellt noch keinen Sicherungsplan.

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Power Shell Konfiguration anzeigen</span></strong></summary>

Feature installieren und Status prüfen:

```
Install-WindowsFeature Windows-Server-Backup
Get-WindowsFeature Windows-Server-Backup
```

Die Installation des Features allein erstellt noch keinen  
Backup-Zeitplan. Backup-Ziel, Zeitplan und Wiederherstellung müssen  
separat eingerichtet und getestet werden.

</details>

</details>
