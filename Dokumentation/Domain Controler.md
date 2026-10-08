---
layout: default
title: VM – Domain Controller
---

# VM – Windows Domain Controller

## Übersicht

Diese Seite dokumentiert die virtuelle Maschine **DC-WS-20-1**.

**Proxmox-VM-Konfiguration**

**VM Parameter**

| Parameter      | Wert           |
| -------------- | -------------- |
| VM-ID          | 201            |
| SSH-Nickname   | dc             |
| VM-Name        | dc-ws-20-1     |
| Zweck          | AD, DNS, DHCP  |
| Betriebssystem | Windows Server |
| IPv4           | 10.10.20.2     |
| Domain         | lab.home       |
| Größe          | 80 GB          |
| RAM            | 8 GB           |
| vCPU           | 4              |
## Aufgabe

**AD, DNS, DHCP**

# Proxmox Konfiguration

## Technische Daten

- **Betriebssystem:** winServer 2022
- **IPv4:** 10.10.20.1
- **Speicher:** 80 GB
- **RAM:** 8 GB
- **vCPU:** 4

**Proxmox Besonderheit** - _1. Eine Treiber müsste heruntergeladen werden um die Virtuelle Festplatte lessen zu können (Red Hat).  
**Windows Besonderheit** - klassische Installationsproblem ohne Microsoft Account problematisch  
Lösung - Installation ohne Netzwerk Verbindung.  
-Shift + F10. Rechner bootet in CMD.  
-Eingabe OOBE\BYPASSNRO  
-Installation weitermachen_

# Grundkonfiguration

## Name ändern > DC-WS-20-1

## Statische IP zugewiesen

- Problem - Internetzugang, Kein IPv4 Netzwerk Adapter
    - Problem Lösung
        1. Ethernet Adapter manuell über den CD/DVD Virtio ISO suchen und installieren
        2. IPv4 statische Adresse vergeben. 10.10.20.1
        3. Gateway: 10.10.1.1

## DNS einrichten

1. Interne DNS-Zone: **lab-home**
2. Forwarder: **1.1.1.1** und **8.8.8.8**
3. **DNS-Rolle:**

- Auf DC installiert
- Wird für die Namensauflösung innerhalb der Active-Directory-Domäne verwendet.
- Externe DNS-Anfragen werden an die konfigurierten Forwarder weitergeleitet.

**Tests:**  
nslookup lab.home → erfolgreich  
nslookup google.com → erfolgreich  
Resolve-DnsName google.com → erfolgreich

### Erklärung

_DC01 übernimmt im HomeLab die DNS-Rolle, da Active Directory auf eine funktionierende DNS-Infrastruktur angewiesen ist. Die interne Zone `lab.home` ermöglicht die Namensauflösung innerhalb der Domäne. Für externe DNS-Anfragen werden öffentliche DNS-Server als Forwarder verwendet._

# SERVICES

## 1. DC an Firewall (FW-OPNsense) anhängen - 10.10.20.0 /24

- **NIC ändern** - Neue Netzwerkkarte erstellen für LAN 10.10.20.1 /24
- **IPv4 anpassen** - Statische IP, Gateway, DNS. - Außerhalb der DHCP Bereich so, dass die Daten konstant bleiben.
- Internet zugang Funktioniert -
    - Test:
    - nslookup dc.lab.home -
    - ping google.com
    

## 2. Active Directory Installieren

### Active Directory konfigurieren.

1. **Erstellen von Organisational Unit (OU)**
    - LAB-Users
    - LAB-Admins
    - LAB-Groups
    - LAB-Servers
    - LAB-Computers
        - Verschieben von **lab.home\Computers** zu **lab.home\LAB-Computers**
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

## 3. Share Folder

- **1.Ordnerstruktur:**
    - C:\Shares
        - C:\Shares\Buchhaltung
        - C:\Shares\Produktion
        - C:\Shares\Management
        - C:\Shares\IT
- **2. Berechtigung geben
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
        - Location: **\\DC1\Buchhaltung Share**
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

## 4. Windows Server Backup

- #### 1. Installation
    
    - Server Manager > Manage > Add Roles and Features > Next
    - Role-Based or Feature ... > Next
    - DC.lab.home auswählen > Next
    - Features > **Windows Serve Backup** aktivieren
    - Next > Install
- #### 2. Virtuelle Festplatte erstellen/konfigurieren
    
    - In Proxmox unter Hardware eine Hard Disk erstellen
    - Parameter: local-lvm, **20GB**
    - DC neustarten
    - Create and Format Hard Disk Partition
    - Festplatte initialisieren, Formatieren auf NTFS, Name: **SSB (B:)**
- #### 3. System State Backup einrichten
    
    - Pfad - Server Manager > Tools > Windows Server Backup > Local Backup > (rechts) Backup Once > Select Item for Backup > Add Items > **System State**
    - Testen - Recover > Get Startet - Hier wird den Backup Quelle gezeigt.
    - Ein Schedule Backup könnte man einrichten. Für meine Zwecke ist es nicht nötig, da der Server nicht dauerhaft an ist.
