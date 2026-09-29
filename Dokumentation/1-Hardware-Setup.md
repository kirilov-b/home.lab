---
layout: default
title: Hardware & Setup
---

# Hardware & Setup

## Übersicht

Diese Seite dokumentiert den grundlegenden Hardware- und Virtualisierungsaufbau des Home Labs.

## Virtualisierung

Als Virtualisierungsplattform wird **Proxmox VE** eingesetzt. Die virtuellen Maschinen werden zentral auf dem Proxmox-System verwaltet und bilden die Grundlage für die verschiedenen Dienste und Testsysteme.

## Aufbau

Der Aufbau ist modular gehalten:

- Proxmox VE als Virtualisierungsplattform
- virtuelle Maschinen für Infrastruktur- und Serverdienste
- getrennte Netzwerkbereiche für Infrastruktur, Server und Clients
- zentrale Verwaltung und Dokumentation
- Backup der virtuellen Maschinen über Proxmox Backup Server

## Ziel

Der Aufbau dient dazu, typische Aufgaben aus den Bereichen:

- Virtualisierung
- Netzwerkadministration
- Windows- und Linux-Administration
- Identity Management
- Monitoring
- Logging
- Backup
- Automatisierung

in einer kontrollierten Laborumgebung praktisch umzusetzen.
