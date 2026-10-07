---
layout: default
title: Hardware & Setup
---

# Hardware & Setup

## Übersicht

Diese Seite dokumentiert den grundlegenden Hardware- und Virtualisierung Aufbau des Home Labs.
Als Virtualisierungsplattform wird **Proxmox VE** eingesetzt. Die virtuellen Maschinen werden zentral auf dem Proxmox-System (über ein Admin VM) verwaltet und bilden die Grundlage für die verschiedenen Dienste und Testsysteme.

## Aufbau
Der Aufbau ist modular gehalten:
- Proxmox VE als Virtualisierungsplattform
- virtuelle Maschinen für Infrastruktur- und Serverdienste
- getrennte Netzwerkbereiche für Infrastruktur, Server und Clients
- zentrale Verwaltung und Dokumentation
- Backup der virtuellen Maschinen über Proxmox Backup Server

## Ziel
Der Aufbau dient dazu, typische Aufgaben aus den Bereichen:Virtualisierung, Netzwerkadministration, Windows- und Linux-Administration,Identity Management,Monitoring,Logging,Backup,Automatisierung
in einer kontrollierten Laborumgebung praktisch umzusetzen.

# Host - Dell Precision 7740
- Intel Core i9 - 9880H
- RAM - 64 GB
- GPU - NVIDIA RTX 3000 Mobile
- 1.NVMe m.2 - 1 TB (Proxmox)
- 2.NVMe m.2 - 1 TB (Linux Mint irrelevant für das Proekt)
- 3.NVMe m.2 - 1 TB (NTFS Storage - irrelevant für das Proekt)
- 4.NVMe m.2 - 500 GB ( Windows - irrelevant für das Proekt)

# Controller - Dell Latitude 3400
- Intel Core i5 - 8265U CPU @ 1.60GHz × 4
- RAM - 16 GB
- NVMe - 256 GB
- Externe Monitor
  
# Router - Fritzbox 6690
- NAS - 500 GB SSD - Dokumentation

# Set-Up
## **Ursprungsidee - Bedienung von Proxmox an der gleichen Maschine**
Die meisten Setups für eine Proxmox Umgebung sind entweder auf 19 Zoll Rack Server oder Tower PC. Meine Proxmox Hardware weicht stark von eine Standard Proxmox Umgebung ab. Der Dell Precision 7740 Workstation Laptop bietet im Vergleich zu Racks und Tower PCs ein integriertes KVM-Interface. Daraus ist die Idee entstanden, die Proxmox so einzurichten, dass man die Umgebung vom gleichen Rechner bedienen kann.

***Wegbeschreibung:***
*Einen zweiten Rechner wurde benötigt um die Umgebung über den Standardweg einzurichten.*
*- Login über den Standard Proxmox Browser weg.*
*- Installation von ein **Admin VM***
*- Einrichtung von einem **GPU Passthrough** der Admin VM über die NVIDIA RTX.* 
*- Interface Übergabe von Admin VM an der GPU.*
*- Bedienung von Proxmox über der VM in dem Browser.*

## **Ergebnis**
Das Ziel wurde nicht zu 100 % realisiert. Die Bedienung ist an dem gleichen Rechner möglich.  Das Problem ist, dass der Laptop, Tastatur, Display und Touchpad nicht direkt an der GPU gebunden sind. 
D.h. Eine Videoausgabe ist möglich, aber nur über den Mini- Display-Port an den externen Monitor. Die Bedienung ist auch möglich, allerdings über externe Mouse und externe Tastatur (via USB)
Aus Bequemlichkeit habe ich mich für ein Zweiten Laptop und Bedienung über den Netzwerk entschieden.

### Dokumentation
Die Dokumentation ist auf dem **NAS** außerhalb der Proxmox Umgebung gespeichert. Ein Zugang ist sowohl in Proxmox Umgebung möglich, als auch über den **Controller Dell**

# Externe VM Backup
- Die Proxmox VMs werden regelmäßig auf eine Externe Datenträger gesichert.
