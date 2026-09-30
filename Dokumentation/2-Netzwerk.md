---
layout: default
title: Netzwerk
---

# Netzwerk

## VLAN, statt virtuelle Netzwerkadapter  
### Entscheidung  
Separate Netze oder **Virtuelle Netzwerke/Bridges** wäre in Proxmox die einfache Variante gewesen, die Trennung von **Management, Service und Clients** aufzubauen.  
Aus Übungs- und Lernzwecken habe ich mich bewusst dafür entschieden, mit VLAN zu arbeiten. Dadurch wird der praktische Aufbau und die Konfiguration von **VLAN (IEEE 802.1Q)**, **VLAN Trunks** und **VLAN-Tagging** geübt. Die geplante Trennung wird in drei VLANs aufgeteilt.  
  
### Ergebnis
Ich verwende eine **VLAN-Aware Bridge (LANbr0)** in Proxmox und einen **VLAN-Trunk zur OPNsense**. Darüber werden die VLANs 10, 20 und 30 übertragen. OPNsense übernimmt anschließend **Routing und Firewall-Regeln** für die einzelnen Netzwerke. Der **LANbr0** ist ohne zusätzliche Netzwerkkarte in OPNsense und Proxmox möglich.  
  
## Netzwerktabelle

| VM-ID  |             Dienst               |   IPv4 /24     | Netzwerk   | DNS |   Gateway    | 
| ------ | -------------------------------- | -------------- | ---------- | --- | ------------ |
|  101   |            Firewall              |  172.18.87.1   | Heim LAN   |     | 172.18.87.1  |
|  101   |   Firewall - Management (MGMT)   |  10.10.10.1    | vLAN - 10  |     | 172.18.87.1  |
|  101   |    Firewall - Service (SRV)      |  10.10.20.1    | vLAN - 20  |     | 172.18.87.1  |
|  101   |     Firewall - Clients (CL)      |  10.10.30.1    | vLAN - 30  |     | 172.18.87.1  |
|  100   | Administration, Dokumentation,   |  10.10.10.3    | vLAN - 10  |     | 10.10.10.1   |
|  201   | Domain Controller AD, DNS, DHCP  |  10.10.20.1    | vLAN - 20  |     | 10.10.20.1   |
|  301   |      Testclient - Windows        | 10.10.30.DHCP  | vLAN - 30  |     | 10.10.30.1   |
|  211   |      Proxmox Backup Server       |  10.10.20.10   | vLAN - 20  |     | 10.10.20.1   |
|  222   |    FreeIPA / Identity / DNS      |  10.10.20.20   | vLAN - 20  |     | 10.10.20.1   |
|  233   |      Nginx / Reverse Proxy       |  10.10.20.30   | vLAN - 20  |     | 10.10.20.1   |
|  244   |           PostgreSQL             |  10.10.20.40   | vLAN - 20  |     | 10.10.20.1   |
|  255   |        Zabbix Monitoring         |  10.10.20.50   | vLAN - 20  |     | 10.10.20.1   |
|  266   |   Docker / Podman / Container    |  10.10.20.60   | vLAN - 20  |     | 10.10.20.1   |
|  102   |      Ansible / Automation        |  10.10.10.20   | vLAN - 10  |     | 10.10.10.1   |
|  277   |        Zentrales Logging         |  10.10.20.70   | vLAN - 20  |     | 10.10.20.1   |
|  288   |   Wazuh / Security Monitoring    |  10.10.20.80   | vLAN - 20  |     | 10.10.20.1   |
|  299   |       Gitea / Git-Server         |  10.10.20.90   | vLAN - 20  |     | 10.10.20.1   |
|  302   |      Testclient - Windows        | 10.10.30.DHCP  | vLAN - 30  |     | 10.10.30.1   |
|  303   |       Testclient - MacOS         | 10.10.30.DHCP  | vLAN - 30  |     | 10.10.30.1   |
  
*Eine eine grafische Darstellung befindet sich in der Diagramm*  
  
## Netzwerk Konzept  
  
| Hosts      | Info                     |
| ---------- | ------------------------ |
| .1         | Gateway                  |
| .2-.9      | Netzwerk/Reserve         |
| .10-.89    | statische Infrastruktur  |
| .90-.99    | Reserve                  |
| .100-.200  | DHCP                     |
| .201-.254  | Reserve                  |
  
## VLAN Tabelle

| VLAN | Name   | Zweck                | Netzwerk      | Gateway    |  
| ---- | ------ | -------------------- | ------------- | ---------- |  
|   10 | MGMT   | Administration       | 10.10.10.0/24 | 10.10.10.1 |  
|   20 | SERVER | Infrastruktur/Server | 10.10.20.0/24 | 10.10.20.1 |  
|   30 | CLT    | Clients              | 10.10.30.0/24 | 10.10.30.1 |  
  
## Routing - Firewall  
### Netzwerkregeln  

| Quelle  | Ziel     |      Port | Zweck                                 |
| ------- | -------- | --------- | ------------------------------------- |

## DHCP und DNS


Die Clients beziehen ihre IP-Konfiguration per DHCP. DNS wird für die Namensauflösung innerhalb der Laborumgebung verwendet.
