---
layout: default
title: "[SRV] VM – Wazuh / Security Monitoring"
---
# NOCH NICHT KONFIGURIERT
# VM – Wazuh / Security Monitoring

**VM Parameter**

|          Parameter | Wert                        | **Parameter** | Wert       |
| -----------------: | --------------------------- | ------------: | ---------- |
|          **VM-ID** | 236                         |      **IPv4** | 10.0.20.30 |
|       **vLAN Tag** | 20                          |    **Domain** | srv.test   |
|        **VM-Name** | sec-lu-20-30                |     **Größe** | 64 GB      |
|          **Zweck** | Wazuh / Security Monitoring |       **RAM** | 8 GB       |
| **Betriebssystem** | Ubuntu 24.04.4 LTS          |      **vCPU** | 4          |


# Übersicht

**Wazuh** soll als zentrale Security-Monitoring-Plattform zur Erkennung, Analyse und Überwachung von Sicherheitsvorfällen in unserer Infrastruktur dienen. Der Wechsel zu **Ubuntu Server 24.04 LTS** erfolgte, weil Ubuntu offiziell als Wazuh-Serverplattform unterstützt wird, während AlmaLinux 10 nicht ausdrücklich aufgeführt ist. Dadurch gewährleisten wir eine bessere dokumentierte Kompatibilität und langfristige Wartbarkeit.
# Konfigurationsschritte 
## 1. OS Installation und Integration

Ubuntu server wurde installiert. Ich verwende die version ohne GUI, um den Speicherbedarf niedrig zu halten.
**Firewalld** wurde installiert um die Einheitlichkeit der Umgebung zu halten

## 2. Wazuh Server Installation und Konfiguration


<details markdown="1">
  <summary><strong><span style="color: #00813F;">Wazuh Insstallationsschritte anzeigen</span></strong></summary>


### 2.1 Wazuh-Installationsskript herunterladen

Das offizielle Installationsskript für Wazuh 4.14 wurde direkt auf dem  
Server heruntergeladen.

```
cd /root
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
ls -lh /root/wazuh-install.sh
```

Anschließend wurde die Skriptversion geprüft:

```
bash /root/wazuh-install.sh -V
```

### 2.2 Wazuh All-in-one installieren

Die All-in-one-Installation wurde mit folgendem Befehl gestartet:

```
bash /root/wazuh-install.sh -a
```

Dabei wurden die zentralen Wazuh-Komponenten auf derselben VM  
eingerichtet:

- **Wazuh Manager:** verarbeitet die Sicherheitsereignisse und  
    Agent-Daten.
- **Wazuh Indexer:** speichert und indexiert die gesammelten Daten.
- **Wazuh Dashboard:** stellt die Weboberfläche zur Verwaltung und  
    Auswertung bereit.
- **Filebeat:** überträgt die Wazuh-Daten an den Indexer.

### 2.3 Installation und Dienste überprüfen

Nach Abschluss der Installation wurde der Status der Dienste einzeln  
kontrolliert:

```
systemctl status wazuh-manager --no-pager
systemctl status wazuh-indexer --no-pager
systemctl status wazuh-dashboard --no-pager
systemctl status filebeat --no-pager
```

Zusätzlich wurde geprüft, ob die Dienste aktiv sind:

```
systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard filebeat
```

Die Netzwerk-Listener wurden mit folgendem Befehl kontrolliert:

```
ss -tulpn
```

Die lokalen HTTPS-Endpunkte wurden anschließend getestet:

```
curl -k -I https://localhost:55000
curl -k -I https://localhost:443
```

Damit wurde geprüft, ob die Wazuh-API auf Port `55000` und das Dashboard  
auf Port `443` lokal erreichbar sind.

### 2.4 Firewall auf dem Wazuh-Server konfigurieren

Der Zugriff wurde gezielt auf die benötigten Systeme beschränkt. Für den  
NGINX-Reverse-Proxy wurde HTTPS von `10.0.20.20` zum Wazuh-Server  
freigegeben:

```
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.20.20/32" port port="443" protocol="tcp" accept'
firewall-cmd --reload
```

Für die Agent-Kommunikation wurden außerdem die folgenden Ports  
verwendet:

Port Protokoll Zweck
- `1514` TCP Kommunikation der Wazuh-Agenten mit dem Manager  
- `1515` TCP Registrierung neuer Agenten  
- `443` TCP HTTPS-Zugriff des NGINX-Reverse-Proxys  
- `10050` TCP Zabbix-Agent-Abfragen durch den Zabbix-Server

Die Freigaben für `1514` und `1515` wurden auf dem Wazuh-Server für die  
jeweilige Agent-Quelladresse eingerichtet. Für den ersten Agenten  
(FreeIPA `10.0.20.2`) wurden die Verbindungen entsprechend auf diese IP  
begrenzt.

</details>

## 3. Reverse Proxy einrichten (Nginx)

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Einrichtungsschritte anzeigen</span></strong></summary>

# NGINX-Weiterleitung einrichten

### 3.1 DNS-Eintrag erstellen

In FreeIPA wurde ein DNS-A-Record für den Dashboard-Namen angelegt:
DNS-Name Ziel-IP

`wazuh.srv.test` `10.0.20.20`

Der Name zeigt bewusst auf den NGINX-Server und nicht direkt auf den  
Wazuh-Server. NGINX nimmt die HTTPS-Verbindung entgegen und leitet sie  
intern an Wazuh weiter.

## 2.2 TLS-Zertifikat über FreeIPA ausstellen

Auf dem NGINX-Server wurde ein privater Schlüssel und eine  
Zertifikatsanforderung (CSR) für `wazuh.srv.test` erstellt. Der DNS-Name  
wurde dabei als Subject Alternative Name (SAN) aufgenommen.

In FreeIPA wurde der HTTP-Service-Principal für den NGINX-Host angelegt  
und um den Alias `HTTP/wazuh.srv.test` ergänzt:

```
ipa service-add HTTP/ngx-la-20-20.srv.test
ipa service-add-principal HTTP/ngx-la-20-20.srv.test HTTP/wazuh.srv.test
```

Die CSR wurde anschließend bei FreeIPA eingereicht:

```
ipa cert-request /root/wazuh.csr --principal=HTTP/wazuh.srv.test --profile-id=caIPAserviceCert
```

Das ausgestellte Zertifikat wurde exportiert und zusammen mit dem  
privaten Schlüssel auf dem NGINX-Server abgelegt.

Verwendete Dateien auf NGINX:

- Zertifikat: `/etc/pki/nginx/wazuh.crt`
- Privater Schlüssel: `/etc/pki/nginx/wazuh-privkey.pem`

Die Zertifikatskette wurde gegen die FreeIPA-CA geprüft:

```
openssl verify -CAfile /etc/ipa/ca.crt /etc/pki/nginx/wazuh.crt
```

Erwartetes Ergebnis: `OK`.

## 2.3 NGINX-Konfiguration erstellen

Auf dem NGINX-Server wurde eine eigene Konfigurationsdatei angelegt:

```
nano /etc/nginx/conf.d/wazuh.conf
```

Die Konfiguration leitet HTTP auf HTTPS um und proxyt HTTPS-Anfragen an  
den Wazuh-Server `10.10.20.30` weiter.

```
server {
    listen 80;
    server_name wazuh.srv.test;
    return 301 https://wazuh.srv.test$request_uri;
}

server {
    listen 443 ssl;
    server_name wazuh.srv.test;

    ssl_certificate     /etc/pki/nginx/wazuh.crt;
    ssl_certificate_key /etc/pki/nginx/wazuh-privkey.pem;

    location / {
        proxy_pass https://10.10.20.30;
        proxy_http_version 1.1;

        proxy_set_header Host sec-lu-20-30.srv.test;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_ssl_verify off;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}
```

**Hinweis zur Backend-TLS-Prüfung:** In dieser Konfiguration ist  
`proxy_ssl_verify off` gesetzt. Die Verbindung zwischen NGINX und Wazuh  
ist damit HTTPS-verschlüsselt, aber NGINX prüft die Identität des  
Backend-Zertifikats nicht. Eine spätere Härtung sollte die Prüfung mit  
der passenden CA aktivieren.

## 2.4 NGINX-Konfiguration prüfen und laden

Vor dem Neuladen wird die Konfiguration auf Syntaxfehler geprüft:

```
nginx -t
```

Wenn der Test erfolgreich ist, wird NGINX neu geladen:

```
systemctl reload nginx
```

Der Backend-Zugriff vom NGINX-Server zum Wazuh-Server kann geprüft  
werden mit:

```
curl -k -I https://10.0.20.30
```

Eine Antwort des Wazuh-Dashboards (beispielsweise eine Weiterleitung zur  
Login-Seite) bestätigt, dass der Backend-Webserver erreichbar ist.

## 2.5 Dashboard aufrufen

Nach Einrichtung von DNS, Zertifikat, Firewall und Reverse Proxy ist das  
Dashboard über folgende Adresse erreichbar:

**[https://wazuh.srv.test](https://wazuh.srv.test)**

Der Browser verbindet sich dabei mit NGINX (`10.0.20.20`). NGINX leitet  
die Anfrage intern per HTTPS an den Wazuh-Server (`10.0.20.30`) weiter.

</details>
## 4. Wazuh Agents Installation und Konfiguration

### Alma Linux Agent

<details markdown="1">
  <summary><strong><span style="color: #00813F;">AlmaLinux Agent Integrationsschritte anzeigen</span></strong></summary>

**Wazuh-Repository einrichten**

1. GPG-Schlüssel importieren**
```
rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH
```

2. Repository erstellen**
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

3. Repository prüfen - `dnf repolist | grep wazuh'
4. Agent installieren - `dnf install wazuh-agent-4.14.10 -y´

**Manager Adresse konfigurieren**
- In der Konfigurationsdatei - `nano /var/ossec/etc/ossec.conf´ 
`<address>MANAGER_IP</address>` anpassen auf `<address>10.0.20.30</address>'

**Agent Registrieren**
`/var/ossec/bin/agent-auth -m 10.0.20.30 -p 1515 -A "Hostname"´
erwartet wird "Valid key recieved"

**Diens Aktivieren**
`systemctl daemon-reload`
`systemctl enable --now wazuh-agent`

**Dienst Prüfen**
`systemctl status wazuh-aegnt`

</details>

### Debian Agent

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Debian Agent Integration anzeigen</span></strong></summary>

**Wazuh-Repository einrichten**

1. GPG-Schlüssel importieren**
```
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
```

2. Repository erstellen**
```
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" > /etc/apt/sources.list.d/wazuh.list
```

3. Agent installieren - `apt install wazuh-agent=4.14.10-1´
4. Installation Kontrolieren - `dpkg -l | grep wazuh-agent`


**Manager Adresse konfigurieren**
- In der Konfigurationsdatei - `nano /var/ossec/etc/ossec.conf´ 
`<address>MANAGER_IP</address>` anpassen auf `<address>10.0.20.30</address>'

**Agent Registrieren**
`/var/ossec/bin/agent-auth -m 10.0.20.30 -p 1515 -A "Hostname"´
erwartet wird "Valid key recieved"

**Diens Aktivieren**
`systemctl daemon-reload`
`systemctl enable --now wazuh-agent`

**Dienst Prüfen**
`systemctl status wazuh-aegnt`
  
</details>

### Ubuntu & Mint Agent Integration


<details markdown="1">
  <summary><strong><span style="color: #00813F;">Ubuntu & Mint Agent Integrationsschritte anzeigen</span></strong></summary>

**Wazuh-Repository einrichten**

1. GPG-Schlüssel importieren**
```
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
```

2. Repository erstellen**
```
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
```

3. Paketlisten aktualisieren - `apt update`
4. Agent installieren - `apt install wazuh-agent=4.14.10-1´
5. Version prüfen - `dpkg -l | grep wazuh-agent`

**Manager Adresse konfigurieren**
- In der Konfigurationsdatei - `nano /var/ossec/etc/ossec.conf´ 
`<address>MANAGER_IP</address>` anpassen auf `<address>10.0.20.30</address>'

**Agent Registrieren**
`/var/ossec/bin/agent-auth -m 10.0.20.30 -p 1515 -A "Hostname"´
erwartet wird "Valid key recieved"

**Diens Aktivieren**
`systemctl daemon-reload`
`systemctl enable --now wazuh-agent`

**Dienst Prüfen**
`systemctl status wazuh-aegnt`

</details>

### Windows Server / Client Agent Integration

<details markdown="1">
  <summary><strong><span style="color: #00813F;">Windows Server Agent Integrationsschritte anzeigen</span></strong></summary>
  
**Windows Version prüfen**
`Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber`

**Wazuh Agent herunterladen**
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

**Wazuh Agent herunterladen**
`curl -O https://packages.wazuh.com/4.x/macos/wazuh-agent-4.14.10-1.intel64.pkg
`
**Wazuh-Agent installieren und Konfigurieren**
`sudo installer -pkg ~/Downloads/wazuh-agent-4.14.10-1.intel64.pkg -target /`

**Wazuh Konfigurationdatei anpassen**
`sudo nano /Library/Ossec/etc/ossec.conf`

**Verbindung testen**
`nc -vz 10.10.20.30 1514`
`nc -vz 10.10.20.30 1515`

**Wazuh Agent registrieren**
` sudo /Library/Ossec/bin/agent-auth -m 10.0.20.30 -A cl3-mac-30 ` 

**Wazuh Agent starten und prüfen**
`sudo launchctl load /Library/LaunchDaemons/com.wazuh.agent.plist`
`sudo /Library/Ossec/bin/wazuh-control status`
`

</details>
# Tests
