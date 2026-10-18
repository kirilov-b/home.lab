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
|   **SSH-Nickname** | dc                |    **Domain** | cl.lab.test |
|        **VM-Name** | dc-ws-20-3        |     **Größe** | 70 GB       |
|          **Zweck** | AD, DNS, DHCP     |       **RAM** | 8 GB        |
| **Betriebssystem** | Windows Server 25 |      **vCPU** | 4           |


# Grundkonfiguration

**1. vLAN Tag 20 in Proxmox auf setzen**

**2. Windows Server installieren:** Windows Server 2025 Standard  
Evaluation mit Desktop Experience installieren, Hostname setzen und  
eine statische IP konfigurieren.
- Uhrzeit und Zeitzone Richtig einstellen - Der Server Dient als Master Uhr für alle andere Clients

**3. Dienste Installieren** und konfigurieren
- **DNS** - Der Dienst wird als DNS Server für die Komplette VLAN30 dienen. Alle Clients
- **AD DS** - Domäne `cl.lab.test`, NetBIOS-Name `CL`. SRV-Einträge und DC-Erkennung testen.
- **DHCP installieren und autorisieren:** Scope für `10.10.30.0/24` anlegen, Gateway/DNS/DNS-Suffix setzen und Scope aktivieren. 
- **DHCP-Relay und Firewall setzen**:Relay aus VLAN 30 zum  DHCP-Server `10.10.20.3`; notwendige Verbindungen nur von den  benötigten Netzen zum DC erlauben. 

**4. SSH und RDP einrichten**


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
8. **49152--65535** TCP Dynamische RPC-Ports bei  modernen Windows-Versionen;  für bestimmte  AD-/Netlogon-Funktionen  relevant


# Unternehmenstruktur
Eine Fiktive Unternehmen und die Hierarchische Struktur

# 2. Active Directory Struktur

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
# 3. File Share Struktur

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
# 4. Windows Server Backup

<details markdown="1">
  <summary><strong><span style="color: #00813F;">WSB-Konfiguration anzeigen</span></strong></summary>


- #### 1. Installation
- 
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

# Konfigurationschritte 

Diese Abschnitt dient als Hilfsmittel für die Konfiguration über **Server Manager** und die  **PowerShell** Variante.


<details markdown="1">
  <summary><strong><span style="color: #00813F;">Konfiguration anzeigen</span></strong></summary>

## 1. Server vorbereiten

<details markdown="1">
  <summary><strong><span style="color: #00813F;">PowerShell Konfiguration anzeigen</span></strong></summary>
### Hostname und statische IP

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

### DNS auf den DC zeigen lassen

Nachdem die statische IP korrekt konfiguriert ist, den bevorzugten  
DNS-Server der DC-Netzwerkkarte auf `10.10.20.3` setzen. Den  
tatsächlichen Interface-Alias vorher mit `Get-NetAdapter` ermitteln:

```
Get-NetAdapter
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.20.3
```

Ersetze `"Ethernet"`, falls der Adapter anders heißt.

</details>
## 2. AD DS und DNS installieren


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
## 3. DHCP installieren und konfigurieren

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
## 4. Client prüfen und der Domäne beitreten

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
`CL\\Administrator` oder `Administrator@cl.lab.test`.

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
## 5. Zeitsynchronisierung prüfen

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
## 6. Windows-Firewall für AD

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
## 7. OpenSSH Server auf einem Windows-Host (optional)

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
Set-Content -Path "C:\\ProgramData\\ssh\\administrators_authorized_keys" -Value "ssh-ed25519 HIER_DEN_VOLLSTAENDIGEN_PUBLIC_KEY_EINFUEGEN support@jump"
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
## 8. RDP auf einem Windows-Host (optional)

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
## 9. Windows Server Backup

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
