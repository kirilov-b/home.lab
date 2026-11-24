# OS Installation

## Ubuntu 24.04 LTS


<details markdown="1">
  <summary><strong><span style="color: #00813F;">Ubuntu 24 LTS Installationsschritte anzeigen</span></strong></summary>

**Ubuntu Server**: Minimized
**IP Konfiguration**
- IP method: Manual
- Subnet: 10.0.20.0/24
- Address: "geplante IP Adresse"
- Gateway: 10.0.20.1
- Name Server: 10.0.20.2
- Search domain: srv.test

**Proxy Address**: leer

**Profile configuration**
- Your Name: "Dienst" Server
- Your servers name: "Namenkonvention"
- Pick a username: ladministrator
- Passwort: "nach Vorgaben"

**Install OpenSSH server**: Aktivieren

Bei erste Frage für ein Restart, die ISO CD/DVD entfernen. Oder Boot Sequence ändern.


</details>


# Remote Steuerung
---
## SSH
Beschreibt die Verbindung zu dem Administrations VM (jump-lm-10-3)
### Linux
Aus Jump Host aus:
1. Öffentliche Schlüssel in dementsprechende VM setzen:
     `ssh-copy-id -i ~/.ssh/id_ed25519.pub root@10.0.20.xx`
2. Bequeme Verbindung:
	   `nano ~/.ssh/config`
3. Host Daten eintragen:
	   *Host pbs*
		   *Hostname 10.0.20.10*
		   *User root*
		   *Identify file ~/.ssh/ided25519*
		   *Identify Only yes*
4. Rechte anpassen
     `chmod 600 ~/.ssh/config`
## RDP

## VNC


# Domain Join
---
## Windows AD - Client Join
#### 1 DHCP/DNS prüfen

1. Am Client **Settings > Network & Internet > Advanced network  
    settings > More network adapter options** öffnen (oder `ncpa.cpl`  
    ausführen).
2. Rechtsklick auf Ethernet > **Properties**.
3. **Internet Protocol Version 4 (TCP/IPv4) > Properties** öffnen.
4. **Obtain an IP address automatically** und **Obtain DNS server  
    address automatically** auswählen.
5. In einer Eingabeaufforderung `ipconfig /all` ausführen.
6. Prüfen: IP aus dem DHCP-Bereich, Gateway `10.0.30.1`, DNS  
    `10.0.20.3`, DNS-Suffix `cl.lab.test`.

#### 2 Domain-Join

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

#### 3 Mitgliedschaft kontrollieren

- **Settings > System > About** öffnen und Domänenmitgliedschaft  
    prüfen.
- Auf dem DC **Active Directory Users and Computers > cl.lab.test >  
    Computers** öffnen.
- Prüfen, ob `CL1-WC-30` existiert und aktiviert ist.

## Domain Join - FreeIPA

### IPA - Relevante Ports

Vom Linux-Client zum FreeIPA-Server müssen je nach aktivierten  
Funktionen unter anderem erreichbar sein:

- DNS: TCP/UDP `53`
- Kerberos: TCP/UDP `88`
- LDAP: TCP `389`
- HTTP/HTTPS für Enrollment und Zertifikate: TCP `80` und ggf. `443`
- Kerberos-Passwortänderung: TCP/UDP `464`
- NTP: UDP `123`, falls du diesen Zeitdienst nutzt

Die konkrete Freigabe hängt von deiner FreeIPA-Konfiguration und deinen  
Firewall-Regeln ab. Öffne Ports nicht pauschal ins Internet; erlaube nur  
die benötigten Verbindungen zwischen den passenden Netzen.

### Verbindung Prüfung

**DNS Prüfen**
```
dig +short ipa-la-20-2.srv.test
dig +short SRV _ldap._tcp.srv.test
dig +short SRV _kerberos._tcp.srv.test
```

**Erwartet wird**
*10.10.20.2*
*0 100 389 ipa-la-20-2.srv.test.*
*0 100 88 ipa-la-20-2.srv.test.*

### DNS Eintrag auf FreeIPA erstellen

Auf FreeIPA VM: *Beispielname: app-la-20-90*

Anmelden und Authentifizieren
```
kinit admin
klist
```

Eintrag überprüfen 
```
ipa dnsrecord-show srv.test app-la-20-90
```

Eintrag (A-Record) anlegen
```
ipa dnsrecord-add srv.test app-la-20-90 --a-rec=10.0.20.90
```

### FreeIPA Client installieren

Auf der neue VM:
1. Installationsbefehlt -  
    `dnf install -y ipa-client`
    `apt install freeipa-client`
2. Zeitstatus prüfen 
```
timedatectl
chronyc tracking
```
3. Zeit synchronisieren - `chronyc makestep`
4. Zeitserver prüfen - `chronyc sources -v`

### FreeIPA - betreten
Auf der neue VM:
```
ipa-client-install --mkhomedir \
  --domain=srv.test \
  --server=ipa-la-20-2.srv.test \
  --realm=SRV.TEST
```

**Abfrage zum Beitreten**
1. **Proceed with fixed values and no DNS discovery?**  
    **YES**, um bewusst den oben angegebenen festen FreeIPA-Server  zu verwenden
    Achtung: Damit wird Failover auf andere FreeIPA-Server nicht automatisch eingerichtet.
2. **Do you want to configure chrony with NTP server or pool  
    address?**  
    **NO**, wenn die Uhrzeit bereits zuverlässig durch deine bestehende  
    Konfiguration synchronisiert wird. 
3. **Continue to configure the system with these values?**  
    Prüfe die Daten dann, **YES**.
4. **User authorized to enroll computers:**  
    Beispielsweise `admin`, sofern dieses Konto dafür berechtigt ist.




---
# Agents Installation
---
## Zabbix Monitoring - Client Integration
### 10.1. Agent 2 installieren

#### AlmaLinux 

<details markdown="1">
  <summary><strong>Alma Linux 10 Integration anzeigen</span></strong></summary>


Alma ist RHEL basiert. Folgende Besonderheiten bei der Installation beachten

```
dnf install -y https://repo.zabbix.com/zabbix/7.4/release/alma/10/noarch/zabbix-release-latest-7.4.el10.noarch.rpm
dnf clean all
dnf makecache
dnf install -y zabbix-agent2
```

danach weiter mit Agent Konfigurieren

</details>


<details markdown="1">
  <summary><strong>Linux Mint Integration anzeigen</span></strong></summary>
  

Mint ist Ubuntu basiert. Folgende Besonderheiten bei der Installation beachten

```
wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_latest+ubuntu24.04_all.deb
sudo apt update
sudo apt install -y zabbix-agent2
```

danach weiter mit Agent Konfigurieren

</details>

#### Debian

<details markdown="1">
  <summary><strong>Debian Integration anzeigen</span></strong></summary>
  
PBS und PVE sind Debian basiert. Folgende Besonderheiten bei der Installation beachten

```
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest+debian13_all.deb
dpkg -i zabbix-release_latest+debian13_all.deb
apt update
apt install zabbix-agent2
```

danach weiter mit Agent Konfigurieren

</details>

#### Windows

<details markdown="1">
  <summary><strong>Windows Integration anzeigen</span></strong></summary>


Die Intergration auf Windows Server oder Client läuft über den GUI. Der passende Agent aus der Webseite herunterladen. https://www.zabbix.com/download_agents. Installieren und die IP Konfiguration wie in /config eintragen.

**Danach die Firewall öffnen:**

in Powershell:

```
New-NetFirewallRule `
  -DisplayName "Zabbix Agent 2 from Zabbix Server" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 10050 `
  -RemoteAddress 10.0.20.40 `
  -Action Allow
```

Prüfen: ` Get-Service *Zabbix*`

danach weiter mit Agent Konfigurieren

</details>

### 10.2. Agent konfigurieren

Konfigurationsdatei öffnen:

```
nano /etc/zabbix/zabbix_agent2.conf
```

Diese Werte setzen bzw. vorhandene Zeilen bearbeiten:

```
Server=10.0.20.40
ServerActive=10.0.20.40
Hostname=VM-FQDN.srv.test
```

- `Server`: Zabbix-Server, der passive Agent-Abfragen stellen darf.
- `ServerActive`: Server für aktive Checks.
- `Hostname`: muss exakt dem später in der Zabbix-Oberfläche  
    eingetragenen Hostnamen entsprechen.

Keine doppelten Einträge anlegen. Speichern mit `Ctrl+O`, `Enter`, dann  
`Ctrl+X`.

### 10.3. Agent starten und testen

```
systemctl enable --now zabbix-agent2
systemctl status zabbix-agent2 --no-pager
zabbix_agent2 -t system.hostname
```

Der Dienst sollte `active (running)` zeigen. Der lokale Test sollte den  
konfigurierten Hostnamen zurückgeben.

### 10.4. Firewall gezielt öffnen

Auf der Ziel-VM prüfen:

```
systemctl is-active firewalld
```

Wenn die Ausgabe `active` lautet, Port `10050/tcp` nur für den  
Zabbix-Server freigeben:

```
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.20.40/32" port port="10050" protocol="tcp" accept'
firewall-cmd --reload
firewall-cmd --list-rich-rules
```

Wenn `firewalld` inaktiv ist, diese Befehle nicht ausführen. Dann  
prüfen, welche Firewall oder Netzwerkregeln tatsächlich verwendet  
werden. Wenn **Firewalld** inactive ist, ist esi wahrscheinlich nicht installiert.

### 10.5. Host in der Zabbix-Weboberfläche hinzufügen

1. Öffne `http://10.0.20.40/zabbix`.
2. Gehe zu **Data collection → Hosts** und klicke **Create host**.
3. Trage ein:
    - **Host name:** exakt derselbe Wert wie `Hostname` in  
        `zabbix_agent2.conf`, z. B. `ngx-la-20-20.srv.test`
    - **Host groups:** `Linux servers` (Gruppe auswählen oder anlegen)
    - **Templates:** `Linux by Zabbix agent`
    - **Interface:** `Agent`
    - **IP address:** IP-Adresse der VM, z. B. `10.0.20.20`
    - **Connect to:** `IP`
    - **Port:** `10050`
4. Klicke **Add**.

Für eine normale passive Agent-Überwachung ist `Linux by Zabbix agent`  
passend. Falls du stattdessen ausschließlich aktive Checks einrichten  
möchtest, müssen Template und Agent-Konfiguration entsprechend auf  
aktive Checks abgestimmt werden.

### 10.6. Verbindung prüfen

Warte 30--60 Sekunden und aktualisiere die Hostliste. Das `ZBX`-Symbol  
unter **Availability** sollte grün werden.

Wenn es rot oder grau bleibt, prüfe:

```
systemctl status zabbix-agent2 --no-pager
tail -n 50 /var/log/zabbix/zabbix_agent2.log
```

Prüfe zusätzlich: - Stimmt `Hostname` exakt mit **Host name** in Zabbix  
überein? - Ist `Server=10.10.20.40` korrekt? - Ist TCP-Port `10050` von  
`10.0.20.40` aus erreichbar? - Zeigt das Zabbix-Interface auf die  
richtige IP-Adresse?


</details>


## Wazu Monitoring - Client Integration

Diese Abschnitt beschreibt die Installation und Anpassung der Wazuh-Agent auf unterschiedliche OS Systeme. 

Voraussetzung ist die folgende Ports an der Wazuh Server freizuschalten um die Verbindung zwischen Server und Client zu ermöglichen.

- Port **1514** - Kommunikation mit der Server
`sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="Host-IP" port port="1514" protocol="tcp" accept' `

- Port **1515** - Registrierung der Wazuh-Agents.
`sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="Host-IP" port port="1515" protocol="tcp" accept' `

**Hinweis:**  *Port 1515 wird nach der Einrichtung wieder gesperrt.* 


<details markdown="1">
  <summary><strong><span style="color: #00813F;">OS Agents Integration anzeigen</span></strong></summary>

Die Reihenfolge bleibt unter alle OS gleich.

- **1. GPG Schlüssel importieren** 
- **2. Repository erstellen** 
- **3. Manager Adresse konfigurieren** 
- **4. Agent registrieren** 
- **5. Dienst aktivieren, Dienst prüfen**


### AlmaLinux Agent

<details markdown="1">
  <summary><strong><span style="color: #00813F;">AlmaLinux Agent Integrationsschritte anzeigen</span></strong></summary>

**Wazuh-Repository einrichten**

**GPG-Schlüssel importieren**
```
rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH
```

**Repository erstellen**
```
cat > /etc/yum.repos.d/wazuh.repo << 'EOF'
[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=Wazuh repository
baseurl=https://packages.wazuh.com/4.x/yum/
protect=1
EOF
```

**Repository prüfen** - `dnf repolist | grep wazuh'
**Agent installieren** - `dnf install wazuh-agent-4.14.10 -y´

**Manager Adresse konfigurieren**
- In der Konfigurationsdatei - `nano /var/ossec/etc/ossec.conf´ 
`<address>MANAGER_IP</address>` anpassen auf `<address>10.0.20.30</address>'

**Agent Registrieren**
`/var/ossec/bin/agent-auth -m 10.0.20.30 -p 1515 -A "Hostname"´
erwartet wird "Valid key recieved"

**Dienst aktivieren**
`systemctl daemon-reload`
`systemctl enable --now wazuh-agent`

**Dienst prüfen**
`systemctl status wazuh-aegnt`

</details>

### Debian Agent

**Anmerkung:** Proxmox VE ist auch Debian basiert. Die Konfiguration ist ähnlich wie unten beschriebene Ablauf mit kleine PVE Besonderheiten. Ein OPNsense Regel für Ports (1514, 1515) muss zwischen den WAN und 10.0.20.30 erstellt werden.


<details markdown="1">
  <summary><strong><span style="color: #00813F;">Debian Agent Integration anzeigen</span></strong></summary>

**GPG-Schlüssel importieren**
```
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
```

**Repository erstellen**
```
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" > /etc/apt/sources.list.d/wazuh.list
```

**Agent installieren** - `apt install wazuh-agent=4.14.10-1`
**Installation Kontrolieren** - `dpkg -l | grep wazuh-agent`


**Manager Adresse konfigurieren**
- In der Konfigurationsdatei - `nano /var/ossec/etc/ossec.conf´ 
`<address>MANAGER_IP</address>` anpassen auf `<address>10.0.20.30</address>'

**Agent Registrieren**
`/var/ossec/bin/agent-auth -m 10.0.20.30 -p 1515 -A "Hostname"´
erwartet wird "Valid key recieved"

**Dienst aktivieren**
`systemctl daemon-reload`
`systemctl enable --now wazuh-agent`

**Dienst prüfen**
`systemctl status wazuh-aegnt`
  
</details>

### Ubuntu & Mint Agent Integration


<details markdown="1">
  <summary><strong><span style="color: #00813F;">Ubuntu & Mint Agent Integrationsschritte anzeigen</span></strong></summary>


**GPG-Schlüssel importieren**
```
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
```

**Repository erstellen**
```
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
```

**Paketlisten aktualisieren** - `apt update`
**Agent installieren** - `apt install wazuh-agent=4.14.10-1`
**Version prüfen** - `dpkg -l | grep wazuh-agent`

**Manager Adresse konfigurieren**
- In der Konfigurationsdatei - `nano /var/ossec/etc/ossec.conf´ 
`<address>MANAGER_IP</address>` anpassen auf `<address>10.0.20.30</address>'

**Agent Registrieren**
`/var/ossec/bin/agent-auth -m 10.0.20.30 -p 1515 -A "Hostname"´
erwartet wird "Valid key recieved"

**Dienst aktivieren**
`systemctl daemon-reload`
`systemctl enable --now wazuh-agent`

**Dienst prüfen**
`systemctl status wazuh-aegnt`

</details>


### Windows Server / Client Agent Integration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Windows Server Agent Integrationsschritte anzeigen</span></strong></summary>
  
**Windows Version prüfen**
`Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber`

**Wazuh-Agent herunterladen**
`Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.10-1.msi -OutFile "$env:TEMP\wazuh-agent.msi"`

**Wazu-Agent installieren und Konfigurieren**
` msiexec.exe /i "$env:TEMP\wazuh-agent.msi" /qn WAZUH_MANAGER="10.0.20.30" WAZUH_REGISTRATION_SERVER="10.0.20.30" WAZUH_AGENT_NAME="dc-ws-20-3"`

**Starten und Prüfen**
`NET START WazuhSvc `
`get-service Wazuhsvc` 

</details>


### MacOS Agent Integration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">MacOS  Agent Integrationsschritte anzeigen</span></strong></summary>


**MacOS prüfen**
`sw-vers`
`hostnam`
`uname -m`

**Wazuh-Agent herunterladen**
`curl -O https://packages.wazuh.com/4.x/macos/wazuh-agent-4.14.10-1.intel64.pkg
`
**Wazuh-Agent installieren und Konfigurieren**
`sudo installer -pkg ~/Downloads/wazuh-agent-4.14.10-1.intel64.pkg -target /`

**Wazuh Konfigurationdatei anpassen**
`sudo nano /Library/Ossec/etc/ossec.conf`

**Verbindung testen**
`nc -vz 10.10.20.30 1514`
`nc -vz 10.10.20.30 1515`

**Wazuh-Agent registrieren**
` sudo /Library/Ossec/bin/agent-auth -m 10.0.20.30 -A cl3-mac-30 ` 

**Wazuh-Agent starten und prüfen**
`sudo launchctl load /Library/LaunchDaemons/com.wazuh.agent.plist`
`sudo /Library/Ossec/bin/wazuh-control status`
`

</details>

### OPNsense-Agent Integration


<details markdown="1">
  <summary><strong><span style="color: #00813F;">OPNsense Integration anzeigen</span></strong></summary>

Die Einrichtung von OPNsense läuft über den Web GUI.

**1. OPNsense vorbereiten**
Es wird nach Updates der Software geprüft und bei Bedarf ausgeführt. **os-wazuh-agent** aus der community plugins wird heruntergeladen und installiert.

**2. OPNsense Wazuh Agent konfigurieren**

- **Manager Hostname:** 10.0.20.30
- **Protocol:** TCP
- **Manager Port:** 1514
- **Enrollment Port:** 1515
- **Active response:** deaktivieren
- **Applications:** firewall

Nach dem **apply** wird die Verbindung zu Wazuh-Server hergestellt.


</details>