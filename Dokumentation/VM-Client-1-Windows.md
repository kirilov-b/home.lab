---
layout: default
title: "[CL] VM – Windows 11 - Testclient 1"
---

# VM – Windows 11 Pro

**VM Parameter**

|      **Parameter** | Wert           |       **Parameter** | Wert         |
| -----------------: | -------------- | ------------------: | ------------ |
|          **VM-ID** | 301            |       **IPv4 DHCP** | 10.0.30.DHCP |
|       **vLAN Tag** | 30             |          **Domain** | cl.lab.test  |
| **VM / Host-Name** | cl1-wc-30      |           **Größe** | 60 GB        |
|          **Zweck** | Testclient     |             **RAM** | 4 GB         |
| **Betriebssystem** | Windows 11 Pro |            **vCPU** | 2            |
|    **Local Admin** | ladministrator | **Netzwerkadapter** | Intel E1000  |

# Übersicht

Der Windows-11-Testclient dient dazu, die Active-Directory-Umgebung aus Sicht eines Windows-Arbeitsplatzes zu testen. Außerdem können Windows-11-Funktionen, Gruppenrichtlinien und die zentrale Benutzerverwaltung praktisch erprobt werden.

Folgende Punkte werden behandelt:

- **Integration in die Proxmox-Umgebung**  
    Anlegen und Einrichten einer Windows-11-VM über die Proxmox-Weboberfläche.
- **Installation und Grundkonfiguration**  
    Windows 11 installieren, Hostnamen vergeben, Updates und virtuelle Hardware-Treiber einrichten.
- **Integration in das Netzwerk**  
    VLAN 30 verwenden und die Netzwerkkonfiguration per DHCP beziehen.
- **Integration in die Active-Directory-Domäne**  
    Windows 11 der Domäne `cl.lab.test` beitreten lassen und die Anmeldung mit Domänenkonten ermöglichen.
- **Remoteverwaltung**  
    RDP für die grafische Administration und OpenSSH für den Kommandozeilenzugriff bereitstellen.


# Konfiguration

## 1. Vorbereitung und Installation


<details>
<summary>
<strong>
$\color{#00813F}{\textsf{Vorbereitungsschritte anzeigen}}$
</strong>
</summary>


### 1.1 Installationsmedium vorbereiten

Für die Installation wird eine Windows-11-ISO-Datei verwendet. Die ISO-Datei wird in Proxmox auf den gewünschten Storage hochgeladen und anschließend als virtuelles CD/DVD-Laufwerk an die VM angeschlossen.

- **Installationsmedium:** Windows 11 ISO (64 Bit)
- **Edition:** Windows 11 Pro
- **Ziel:** virtuelle Festplatte der VM `cl1-wc-30`

**Hinweis:** Windows 11 Pro ist erforderlich, wenn der Client einer lokalen Active-Directory-Domäne beitreten und eingehende Remotedesktopverbindungen (RDP) annehmen soll. Windows 11 Home unterstützt diese beiden Funktionen nicht in gleicher Weise.

Windows 11 benötigt eine UEFI-basierte Startumgebung und TPM 2.0. Die virtuelle TPM-Komponente wird in Proxmox über *Hardware > Add > TPM State* hinzugefügt. Die EFI-Disk wird ebenfalls über die Hardware-Ansicht angelegt. Die passende VirtIO-Treiber-ISO zusätzlich als zweites CD/DVD-Laufwerk einbinden. Mithilfe der Treiber ist die Virtuelle Festplatte für Installation sichtbar

### 1.2 Installation und Einrichtung

Die Installation wird standardgemäß durchgeführt. Ein lokales Administrationskonto `ladministrator`wurde eingerichtet. Der Hostname wurde in *Einstellungen  > System > Info > Diese PC umbenennen* laut der Namenkonvention angepasst. `cl1-wc-30`

### 1.3 Netzwerkkonfiguration

Die VM wird in der Proxmox Server an der Bridge `LANbr0` mit `VLAN TAG 30`gebunden. Die Netzwerkeinstellungen an der Windows sind auf **automatisch** eingerichtet. Somit wird die VM automatisch an der VLAN 30 gebunden und die Netzwerkkonfiguration läuft über **DHCP.**
Die VM bekommt, **IP, Gateway, DNS** automatisch zugewissen.

</details>

## 2. Integration

<details markdown="1"> <summary><strong><span style="color: #00813F;">Integrationsschritte anzeigen</span></strong></summary>

### 2.1 Integration in Active-Directory

Der Domänenbeitritt erfolgt über die Windows-Einstellungen:

1. **Einstellungen → Konten → Auf Arbeits- oder Schulkonto zugreifen** öffnen.
2. **Verbinden** auswählen.
3. Im folgenden Fenster die Option **Dieses Gerät einer lokalen Active-Directory-Domäne hinzufügen** auswählen.
4. Als Domäne `cl.lab.test` eintragen und mit **Weiter** bestätigen.
5. Anmeldedaten eines Kontos eingeben, das Computer der Domäne hinzufügen darf. Für die Laboreinrichtung kann das Domänenadministratorkonto verwendet werden.
6. Die Bestätigung des erfolgreichen Domänenbeitritts abwarten.
7. Windows neu starten.

Nach dem Neustart steht der Client als Domänenmitglied zur Verfügung. Die Anmeldung erfolgt über **Anderer Benutzer** am Windows-Anmeldebildschirm.

**Anmeldeformate:**

- Domänenkonto: `CL\Benutzername`
- UPN-Format: `Benutzername@cl.lab.test`
- Lokales Konto: `.\ladministrator` (lokaler Computer; je nach Anmeldeoberfläche alternativ `COMPUTERNAME\ladministrator`)

### 2.2 Domänenbeitritt überprüfen

Die Überprüfung kann über die grafische Oberfläche erfolgen:

1. **Einstellungen → System → Info** öffnen.
2. Unter den Geräte- bzw. Domäneninformationen kontrollieren, ob der Computer der Domäne `cl.lab.test` zugeordnet ist.
3. Alternativ **Systemeigenschaften → Computername** öffnen. Dort muss als Mitgliedschaft die Domäne `cl.lab.test` angezeigt werden.
4. Am Anmeldebildschirm **Anderer Benutzer** auswählen und mit einem berechtigten Domänenkonto anmelden.

Eine erfolgreiche Anmeldung mit einem Domänenkonto bestätigt, dass der Client die Domänenidentität auflösen und authentifizieren kann.
### 2.3 Windows-Firewall und Zugriffsschutz

Die Windows-Firewall soll aktiviert bleiben. Für die Remoteverwaltung sind nur die benötigten eingehenden Verbindungen freizugeben:

|Dienst|Protokoll/Port|Zweck|
|---|---|---|
|RDP|TCP `3389`|Grafische Fernverwaltung|
|SSH|TCP `22`|Administration über OpenSSH|

Für RDP und SSH sollte der Zugriff auf vertrauenswürdige Quellnetze bzw. den Jump-Host beschränkt werden. Die Firewallregeln können über **Windows Defender Firewall mit erweiterter Sicherheit → Eingehende Regeln** geprüft und unter **Bereich** auf zulässige Remote-IP-Adressen eingeschränkt werden.
### 2.4 OpenSSH-Server aktivieren

Der OpenSSH-Server ermöglicht die Administration des Windows-Clients über SSH. Die Installation und Aktivierung erfolgt über die grafische Oberfläche.

**OpenSSH Server installieren:**

1. **Einstellungen → System → Optionale Features** öffnen. Alternativ im Startmenü nach **Optionale Features** suchen.
2. Bei **Optionales Feature hinzufügen** bzw. **Features anzeigen** nach **OpenSSH-Server** suchen.
3. **OpenSSH-Server** auswählen und installieren.
4. Nach Abschluss im Startmenü die App **Dienste** öffnen.
5. Den Dienst **OpenSSH SSH Server (sshd)** suchen und dessen Eigenschaften öffnen.
6. Starttyp auf **Automatisch** setzen.
7. **Starten** auswählen und die Änderung übernehmen.

Bei der Installation wird üblicherweise eine eingehende Windows-Firewallregel für OpenSSH auf TCP-Port `22` angelegt. In **Windows Defender Firewall mit erweiterter Sicherheit → Eingehende Regeln** kontrollieren, ob die Regel **OpenSSH-Server-In-TCP** aktiviert ist.
### 2.4 Remotedesktop (RDP) aktivieren

Mit RDP wird der Windows-11-Client grafisch vom Jump-Host aus verwaltet. Für eingehende Remotedesktopverbindungen wird Windows 11 Pro benötigt.

1. **Einstellungen → System → Remotedesktop** öffnen.
2. **Remotedesktop** aktivieren.
3. Die Sicherheitsabfrage bestätigen.
4. Unter **Remotedesktopbenutzer** prüfen, welche Benutzer zugreifen dürfen.
5. Bei Bedarf über **Hinzufügen** die gewünschten Domänenkonten oder Domänengruppen freigeben.

Die Windows-Firewall aktiviert beim Einschalten von Remotedesktop die erforderlichen Regeln. Für die Lab-Umgebung sollte der Zugriff zusätzlich auf den Jump-Host bzw. das Management-Netz begrenzt werden.

**Remmina-Profil auf dem Jump-Host:**

| Einstellung  | Wert                                              |
| ------------ | ------------------------------------------------- |
| Protokoll    | RDP                                               |
| Server       | `cl1-wc-30.cl.lab.test`                           |
| Benutzername | `CL\Benutzername` oder `Benutzername@cl.lab.test` |
| Passwort     | AD-Kennwort                                       |
| Domäne       | CL                                                |

</details>

## 3. Weitere Dienste

<details markdown="1"> <summary><strong><span style="color: #00813F;">Dienste anzeigen</span></strong></summary>

### 3.1 Zabbix Monitoring
Die Linux VM wurde an unsere Zabbix Server gebunden, um die Auslastung der Hardware zu überwachen.
#### 3.1.1 Zabbix Server 
Über die Weboberfläche wird ein neuen Host hinzugefügt.
*Data collection > Hosts > Create host*

Hostangaben

| Parameter        | Einstellung             |
| ---------------- | ----------------------- |
| **Host name**    | cl1-wc-30               |
| **Visible name** | CL1 - Windows 11        |
| **Host groups**  | Windows servers         |
| **Templates**    | Windows by Zabbix agent |
| **Status**       | Enable                  |
| **Type**         | Agent                   |
| **Connect to**   | DNS                     |
| **DNS name**     | cl1-wc-30.cl.lab.test   |
| **IP address**   | leer                    |
| **Port**         | 10050                   |

### 3.1.2 Zabbix Agent 2 Installation und Konfiguration

**Installation**
1. Zabbix-Downloadseite öffnen:  
    [https://www.zabbix.com/download_agents](https://www.zabbix.com/download_agents)
2. Folgende Auswahl treffen:
    - Zabbix Version: `7.4`
    - Betriebssystem: Windows
    - Architektur: AMD64 (64 Bit)
    - Komponente: Zabbix Agent 2
    - Packaging: MSI
    - Encryption: OpenSSL
3. MSI-Datei herunterladen.
4. Installationsdatei starten und folgen
5. Standardinstallationspfad übernehmen.

**Konfiguration**
Im Installationsassistenten folgende Parameter eintragen:

|Parameter|Wert|
|---|---|
|Host name|`cl1-wc-30`|
|Zabbix server IP/DNS|`10.10.20.40`|
|Agent listen port|`10050`|
|Server or Proxy for active checks|Leer lassen|

**Windows-Firewall konfigurieren** - Der notwendige Regel wurde durch die Installation des Agents erstellt. 
Der Dienst wird auf der Zabbix Weboberfleche grün angezeigt, was ein Zeichnen für eine fehlerfreie Verbindung ist.


</details>