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
- **DNS** -
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
# SERVICES

## 2. Active Directory Installieren

<details markdown="1">
  <summary><strong><span style="color: #3399FF;">AD-Konfiguration anzeigen</span></strong></summary>
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
## 3. Share Folder

<details markdown="1">
  <summary><strong><span style="color: #3399FF;">Share Konfiguration anzeigen</span></strong></summary>

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
  <summary><strong><span style="color: #3399FF;">WSB-Konfiguration anzeigen</span></strong></summary>


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