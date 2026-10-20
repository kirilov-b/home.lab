# VM Integration
Folgende Absatz beschreibt die notwendige Schritte, um unsere Umgebung zu betreten.
Eine **VM Clone Image** mit Grundkonfiguration und wiederholende Schritte wurde vorbereitet um die Integration zu erleichtern

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Integrationsschritte anzeigen</span></strong></summary>

## 1. Grundkonfiguration VM
Beschreibt die notwendige Tools und Einstellungen, um die VM in der Umgebung zu funktionieren

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Grundkonfiguration anzeigen</span></strong></summary>

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

</details>

## 2. SSH Integration
Beschreibt die Verbindung zu dem Administrations VM (jump-lm-10-3)

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

### Windows

## 3. Domain Join - FreeIPA
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
ipa dnsrecord-add srv.test app-la-20-90 --a-rec=10.10.20.90
```

### FreeIPA Client installieren

Auf der neue VM:
1. Installationsbefehlt -  `dnf install -y ipa-client`
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

</details>

## 4. Monitoring - Zabbix

<details markdown="1">
  <summary><strong>Zabbix Integration anzeigen</span></strong></summary>

## 1. Agent 2 installieren

### AlmaLinux 10

```
dnf install -y https://repo.zabbix.com/zabbix/7.4/release/alma/10/noarch/zabbix-release-latest-7.4.el10.noarch.rpm
dnf clean all
dnf makecache
dnf install -y zabbix-agent2
```

Für andere AlmaLinux-/RHEL-Versionen oder Debian/Ubuntu das Repository  
passend zur jeweiligen Distribution und Version wählen.

### Linux Mint

```
dnf install -y https://repo.zabbix.com/zabbix/7.4/release/alma/10/noarch/zabbix-release-latest-7.4.el10.noarch.rpm
dnf clean all
dnf makecache
```



## 2. Agent konfigurieren

Konfigurationsdatei öffnen:

```
nano /etc/zabbix/zabbix_agent2.conf
```

Diese Werte setzen bzw. vorhandene Zeilen bearbeiten:

```
Server=10.10.20.40
ServerActive=10.10.20.40
Hostname=VM-FQDN.srv.test
```

- `Server`: Zabbix-Server, der passive Agent-Abfragen stellen darf.
- `ServerActive`: Server für aktive Checks.
- `Hostname`: muss exakt dem später in der Zabbix-Oberfläche  
    eingetragenen Hostnamen entsprechen.

Keine doppelten Einträge anlegen. Speichern mit `Ctrl+O`, `Enter`, dann  
`Ctrl+X`.

## 3. Agent starten und testen

```
systemctl enable --now zabbix-agent2
systemctl status zabbix-agent2 --no-pager
zabbix_agent2 -t system.hostname
```

Der Dienst sollte `active (running)` zeigen. Der lokale Test sollte den  
konfigurierten Hostnamen zurückgeben.

## 4. Firewall gezielt öffnen

Auf der Ziel-VM prüfen:

```
systemctl is-active firewalld
```

Wenn die Ausgabe `active` lautet, Port `10050/tcp` nur für den  
Zabbix-Server freigeben:

```
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.10.20.40/32" port port="10050" protocol="tcp" accept'
firewall-cmd --reload
firewall-cmd --list-rich-rules
```

Wenn `firewalld` inaktiv ist, diese Befehle nicht ausführen. Dann  
prüfen, welche Firewall oder Netzwerkregeln tatsächlich verwendet  
werden.

## 5. Host in der Zabbix-Weboberfläche hinzufügen

1. Öffne `http://10.10.20.40/zabbix`.
2. Gehe zu **Data collection → Hosts** und klicke **Create host**.
3. Trage ein:
    - **Host name:** exakt derselbe Wert wie `Hostname` in  
        `zabbix_agent2.conf`, z. B. `ngx-la-20-20.srv.test`
    - **Host groups:** `Linux servers` (Gruppe auswählen oder anlegen)
    - **Templates:** `Linux by Zabbix agent`
    - **Interface:** `Agent`
    - **IP address:** IP-Adresse der VM, z. B. `10.10.20.20`
    - **Connect to:** `IP`
    - **Port:** `10050`
4. Klicke **Add**.

Für eine normale passive Agent-Überwachung ist `Linux by Zabbix agent`  
passend. Falls du stattdessen ausschließlich aktive Checks einrichten  
möchtest, müssen Template und Agent-Konfiguration entsprechend auf  
aktive Checks abgestimmt werden.

## 6. Verbindung prüfen

Warte 30--60 Sekunden und aktualisiere die Hostliste. Das `ZBX`-Symbol  
unter **Availability** sollte grün werden.

Wenn es rot oder grau bleibt, prüfe:

```
systemctl status zabbix-agent2 --no-pager
tail -n 50 /var/log/zabbix/zabbix_agent2.log
```

Prüfe zusätzlich: - Stimmt `Hostname` exakt mit **Host name** in Zabbix  
überein? - Ist `Server=10.10.20.40` korrekt? - Ist TCP-Port `10050` von  
`10.10.20.40` aus erreichbar? - Zeigt das Zabbix-Interface auf die  
richtige IP-Adresse?

## Sicherheitshinweise

- Agent-Port `10050/tcp` nicht unnötig für das gesamte Netzwerk  
    freigeben.
- Keine Passwörter oder Geheimnisse in Dokumentationen ablegen.
- Für produktivere Umgebungen später TLS/PSK oder  
    Zertifikatsverschlüsselung und passende Netzwerkregeln einrichten.

---

**Beispiel erfolgreich getestet:** `ngx-la-20-20.srv.test`  
(`10.10.20.20`) mit AlmaLinux 10.2 und Zabbix Agent 2 7.4.15.

</details>

.....

</details>