---
layout: default
title: VM – FreeIPA / Identity /
---
# VM – FreeIPA / Identity / DNS

**VM Parameter**

| Parameter      | Wert                     |
| -------------- | ------------------------ |
| VM-ID          | 222                      |
| SSH-Nickname   | ipa                      |
| VM-Name        | ipa-la-20-2.srv.test     |
| Zweck          | FreeIPA / Identity / DNS |
| Betriebssystem | Linux Alma               |
| IPv4           | 10.10.20.2               |
| Domain         | srv.test                 |
| Größe          | 40 GB                    |
| RAM            | 4 GB                     |
| vCPU           | 2                        |

# Übersicht

FreeIPA stellt zentrale Identitäts-, Authentifizierungs- und Verzeichnisdienste bereit. Die folgenden Komponenten sind für diese Installation relevant.
Der Dienst besteht aus folgende Komponente:
**LDAP / 389 Directory Server**- Speichert und stellt Identitätsdaten wie Benutzer, Gruppen, Hosts und Richtlinien über ein Verzeichnis bereit.

**Kerberos KDC** - Stellt Kerberos-Tickets für die Anmeldung und den Zugriff auf unterstützte Dienste aus.

**kadmin** - Unterstützt die Verwaltung von Kerberos-Prinzipalen und Schlüsseln.

**DNS / BIND** - Löst Namen der FreeIPA-Domain auf und stellt DNS-Zonendaten bereit.

**Reverse-DNS** - Ordnet IP-Adressen den zugehörigen Hostnamen zu.

**Apache HTTP Server** - Stellt die FreeIPA-Weboberfläche und zugehörige HTTP(S)-Endpunkte bereit.

**Dogtag PKI / pki-tomcatd** - Stellt die Zertifizierungsstellen-Funktionen (CA) für die FreeIPA-Umgebung bereit.

**ipa-custodia** - Unterstützt die geschützte Verwaltung und Übertragung sensibler Schlüsselmaterialien zwischen FreeIPA-Komponenten.

**ipa-otpd** - Unterstützt die OTP-Authentifizierung, sofern diese eingerichtet und verwendet wird.

**ipa-dnskeysyncd** - Synchronisiert Schlüsselmaterial, das für DNSSEC in der integrierten DNS-Umgebung benötigt wird.

**SSSD auf Clients** - Kann Linux-Clients an FreeIPA anbinden und Identitäten sowie Anmeldungen zentral nutzbar machen; Client-Anbindung ist noch offen.

**NTP/Chrony-Zeitdienst** - Eine passende Systemzeit ist für Kerberos wichtig. Die Zeit wurde vor der Installation als synchronisiert angezeigt; der Zustand nach der Installation ist noch zu prüfen.

# 1. Prüfpunkte 
Vor Installation:


<details markdown="1">
<summary><strong>Prüfpunkte anzeigen</strong></summary>

## 1.1 Netzwerk
- FreeIPA-Server befindet sich im Servernetz `10.10.20.0/24`.
- Server-IP: `10.10.20.2/24`.
- Standardgateway: `10.10.20.1`.
- Erreichbarkeit des Gateways und des Windows-Domaincontrollers wurde vor der Installation geprüft.
- *Zugriffe aus anderen VLANs und passende Firewallregeln sind noch zu prüfen.*

## 1.2 Hostname und Namensauflösung
- Vollständiger Hostname: `ipa-la-20-2.srv.test`.
- Lokale Namensauflösung wurde vor der Installation vorbereitet.
- FreeIPA-DNS soll die Zone `srv.test` autoritativ bereitstellen.

## 1.3 Zeit
- Zeitzone: `Europe/Berlin`.
- Vor der Installation waren Systemzeit-Synchronisierung und NTP-Dienst aktiv.
- Zeitstatus nach der Installation ist noch zu kontrollieren.

</details>
# 2. Installation FreeIPA

<details markdown="1">
<summary><strong>Installation anzeigen</strong></summary>

## 2.1 Betriebssystem
- AlmaLinux 10.2 wurde aktualisiert.
- Nach dem Neustart wurde Kernel `6.12.0-211.64.1.el10_2.x86_64` festgestellt.

## 2.2 FreeIPA-Pakete
- Installiert wurden `ipa-server` und `ipa-server-dns`.
- Paketversion: `4.13.4-1.el10_2`.

## 2.3 Installationsergebnis
- Der FreeIPA-Installer wurde mit integriertem DNS ausgeführt.
- Der Installer meldete `Setup complete` und `The ipa-server-install command was successful`.

</details>

# 3. Konfiguration FreeIPA

<details markdown="1">
<summary><strong>Konfiguration anzeigen</strong></summary>

## 3.1 Identität und Kerberos
- DNS-Domain: `srv.test`.
- Kerberos-Realm: `SRV.TEST`.
- NetBIOS-Name: `SRV`.
- FreeIPA-Administrator: `admin`.

## 3.2 DNS
- Integrierter BIND-DNS ist aktiviert.
- DNS-Forwarder: `10.10.20.1` (OPNsense).
- Forward-Policy: `only`.
- Reverse-Zone: `20.10.10.in-addr.arpa.`.

## 3.3 Zertifikate
- Eigenständige, selbstsignierte FreeIPA-CA wurde eingerichtet.
- Die CA-Sicherungsdatei `/root/cacert.p12` muss noch sicher gesichert werden.

## 3.4 Beziehung zu Active Directory
- Active Directory **lab.home** (NetBIOS LAB) bleibt separat.
- Es wurde noch kein Trust zwischen AD und FreeIPA eingerichtet.


</details>

# 4. Test - Aktivität

<details markdown="1">
<summary><strong>Tests anzeigen</strong></summary>

## 4.1 FreeIPA-Dienste
- `ipactl status` wurde erfolgreich ausgeführt.
- Alle dabei aufgeführten Dienste meldeten `RUNNING`.

## 4.2 DNS
- A-Record: `ipa-la-20-2.srv.test` löst zu `10.10.20.2` auf.
- SOA-Abfrage für `srv.test` war erfolgreich.
- Reverse-DNS für `10.10.20.2` liefert `ipa-la-20-2.srv.test.`.

## 4.3 Kerberos
- `kinit admin` war erfolgreich.
- `klist` zeigte ein gültiges Ticket für `admin@SRV.TEST`.

## 4.4 LDAP
- LDAP und LDAPS lauschen auf TCP 389 und TCP 636.
- LDAP-Abfrage über GSSAPI/Kerberos war erfolgreich und lieferte `result: 0 Success`.
- *Eine separate verschlüsselte LDAPS-Verbindungsprüfung steht noch aus.*

## 4.5 Weboberfläche und externe Erreichbarkeit
- *Weboberfläche unter `https://ipa-la-20-2.srv.test/` ist noch zu prüfen.*
- *Zugriff aus anderen VLANs und gezielte Firewallregeln sind noch zu prüfen.*
- *Client-Anbindung und Client-DNS-Konfiguration sind noch offen.*

</details>

