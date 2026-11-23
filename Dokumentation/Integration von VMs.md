
# Standard Installation Ubuntu

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



# VM Integration
Folgende Absatz beschreibt die notwendige Schritte, um unsere Umgebung zu betreten.
Eine **VM Clone Image** mit Grundkonfiguration und wiederholende Schritte wurde vorbereitet um die Integration zu erleichtern


## 1. Grundkonfiguration VM
Beschreibt die notwendige Tools und Einstellungen, um die VM in der Umgebung zu funktionieren


1. **Proxmox** - VLAN Tag muss das Schema entsprechen
2. **IP Konfiguration** - muss die Vorgaben entsprechen
	-  IP - 10.10.xx.xx /24
	je nach VLAN 
	-  Gateway - 10.10.20.1 / 10.10.30.1
	-  DNS -  10.10.20.2 / 10.10.20.3
	-  Zeitzone anpassen
		- prüfen - `timedatectl`
		- anpassen - `timedatectl set-timezone Europe/Berlin`
3. **FQDN** - VM Namenkonvention + Domain muss stimmen
4. **VM Tools**
	-  **Nano** Installieren `dnf install -y nano`
	-  Firewallregeln anpassen
	-  **Dig** installieren - `dnf install -y bind-utils` 



## 2. SSH Integration
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


### Windows

## 3. Domain Join - FreeIPA
Die obengenannten Voraussetzungen müssen erfüllt sein um den Domain zu betreten.

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

## 4. Monitoring - Zabbix Integration
Anleitung in VM-Zabbis

## 5. Wazuh Integration


</details>

.....

</details>