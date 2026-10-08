---
layout: default
title: VM – FreeIPA / Identity / DNS
---
# NOCH NICHT KONFIGURIERT
# VM – FreeIPA / Identity / DNS

**VM Parameter**

| Parameter      | Wert                     |
| -------------- | ------------------------ |
| VM-ID          | 222                      |
| SSH-Nickname   | ipa                      |
| VM-Name        | ipa-la-20-2              |
| Zweck          | FreeIPA / Identity / DNS |
| Betriebssystem | Linux Alma               |
| IPv4           | 10.10.20.2               |
| Domain         | lab.service              |
| Größe          | 20 GB                    |
| RAM            | 2 GB                     |
| vCPU           | 2                        |

# Übersicht

FreeIPA dient als Identity MManagement, eine zentrale Vertrauenstelle für Linux Systeme. **LDAP** speichert Benutzer, Gruppen und weitere Identitätsinformationen. **Kerberos** ist ein Teil von FreeIPA und erlaubt sichere Authentifizierung ohne ständig Passwörter zu übertragen
**DNS** ist auch ein Teil von **FreeIPA**  und löst die Namen in der Linux Domäne. Auch die Rechtevergabe werden durch **FreeIPA** verteilt.

# Konfiguration Planung

1. Netzwerk prüfen
2. Hostname setzen
3. DNS konfigurieren
4. Zeit/NTP prüfen
5. AlmaLinux aktualisieren
6. FreeIPA installieren
7. FreeIPA konfigurieren
8. DNS/Kerberos/LDAP testen 



# Tests


