# VM Integration
Folgende Absatz beschreibt die notwendige Schritte, um unsere Umgebung zu betreten.
Eine **VM Clone Image** mit Grundkonfiguration und wiederholende Schritte wurde vorbereitet um die Integration zu erleichtern

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Integrationsschritte anzeigen</span></strong></summary>

## Grundkonfiguration VM

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Grundkonfiguration anzeigen</span></strong></summary>

1. Proxmox - VLAN Tag muss das Schema entsprechen
2. IP Konfiguration - muss die Vorgaben entsprechen
	1. IP - 10.10.xx.xx /24
	je nach VLAN 
	2. Gateway - 10.10.20.1 / 10.10.30.1
	3. DNS -  10.10.20.2 / 10.10.20.3
3. FQDN - VM Namenkonvention + Domain muss stimmen
4. VM Tools
	1. **Nano** Installieren `dnf install -y nano`
	2. Firewallregeln anpassen
	3. **Dig** installieren - `dnf install -y bind-utils` 

</details>

## SSH Integration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">SSH Verbindung anzeigen</span></strong></summary>
  
### Linux
Aus Jump Host aus:
1. Öffentliche Schlüssel in dementsprechende VM setzen:
     `ssh-copy-id -i ~/.ssh/id_ed25519.pub root@10.10.20.xx`
2. Bequeme Verbindung:
	   `nano ~/.ssh/config`
3. Host Daten eintragen:
	   *Host pbs*
		   *Hostname 10.10.20.10*
		   *User root*
		   *Identify file ~/.ssh/ided25519*
		   *Identify Only yes*
4. Rechte anpassen
     `chmod 600 ~/.ssh/config`

</details>

## Windows

## Domain Join - FreeIPA
Die obengenannten Voraussetzungen müssen erfüllt sein um den Domain zu betreten.

<details markdown="1">
  <summary><strong><span style="color: #00813F;">IPA Integration anzeigen</span></strong></summary>

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

## DNS Eintrag auf FreeIPA erstellen

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
ipa dnsrecord-add srv.test app-la-20-90 --a-rec=10.10.20.90
```

## FreeIPA Client installieren

Auf der neue VM:
1. Installationsbefehlt -  `dnf install -y ipa-client`
2. Zeitstatus prüfen 
```
timedatectl
chronyc tracking
```
3. Zeit synchronisieren - `chronyc makestep`
4. Zeitserver prüfen - `chronyc sources -v`

## FreeIPA - betreten
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

</details>

## Monitoring - Zabb


</details>