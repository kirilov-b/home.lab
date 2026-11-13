---
layout: default
title: "[CL] VM – Testclient - 3 - MacOS"
---
# VM – macOS Sonoma (Testclient)

**VM Parameter**

|      **Parameter** | Wert         |       **Parameter** | Wert           |
| -----------------: | ------------ | ------------------: | -------------- |
|          **VM-ID** | 303          |       **IPv4 DHCP** | 10.0.30.xxx    |
|       **vLAN Tag** | 30           |          **Domain** | cl.lab.test    |
|        **VM-Name** | cl3-mac-30   |       **Hard Disk** | 80 GB (SATA)   |
|          **Zweck** | Testclient   |             **RAM** | 8 GB           |
| **Betriebssystem** | MacOS Sonoma |            **vCPU** | 2              |
|           **BIOS** | OVMF/UEFI    | **Netzwerkadapter** | VMware vmxnet3 |


# Übersicht

Zweck diese Mac Testclient ist unsere Domäne Umgebung aus der MacOS Perspektive zu betrachten. Zusäztlich dient die VM als Testumgebung für das MacOS Betriebsystem selbst. Folgende Punkte zu üben.

- **Integration eine MacOS in der Proxmox Umgebung**
  Recherche, Vorbereitung und Ausführung der Installation
- **Integration in der VM in unsere Netzwerk**
  Netzwerk Grundkonfiguration für MacOS - Ziel ist Plug an Play via DHCP
- **Integration der VM in der Domäne**
  Domäne beitreten und Auswirkung der Active Direktory  auf Mac Geräte

# Konfigurationsschritte 

## Vorbereitung

<strong><span style="color: #00813F;">Vorbereitungsschritte anzeigen</span></strong></summary>


## 1. Long QT-OpenCore - v07.iso 

Für Proxmox/QEMU wurde das vorgefertigte Projekt **LongQT-sea/OpenCore-ISO** (GitHub) verwendet. Das Repository stellt ein speziell für Proxmox VE und QEMU/KVM vorbereitetes, bootfähiges OpenCore-CD/DVD-Image bereit.
OpenCore wird vor dem eigentlichen macOS-Start geladen und bereitet die Startumgebung so vor, dass macOS auf der bereitgestellten virtuellen Hardware starten kann.
Es bildet die benötigte Mac-Startumgebung über eine konfigurierbare EFI-Schicht nach beziehungsweise stellt macOS passende Plattforminformationen und Treiber bereit.

Die OpenCore ISO Datei wird an einem CD/DVD Laufwerk in der Proxmox VM Konfigurtion angeähngt. Diese Laufwerk muss in der Bootsequenz ganz oben stehen.
Der Startablauf sieht vereinfacht so aus:

1. Proxmox startet die VM mit OVMF als virtueller UEFI-Firmware.
2. Die UEFI-Firmware findet das OpenCore-CD/DVD-Laufwerk und startet `EFI/BOOT/BOOTx64.efi`.
3. Dieser EFI-Loader startet OpenCore.
4. OpenCore liest `EFI/OC/config.plist`, lädt die aktivierten UEFI-Treiber und ACPI-Tabellen und stellt die konfigurierten Plattforminformationen bereit.
5. Das OpenCore-Auswahlmenü zeigt verfügbare Startziele an.
6. Über das Menü wird der macOS-Installer auf der zweiten virtuellen DVD gestartet.
7. Danach übernimmt Apples eigener Bootloader `boot.efi` und startet den macOS-Installationsprozess.

Einrichtung das System Booten ohne den OpenCore Laufwerk:
Die Einrichtung wird in der Dokumentation von LonQT-sea/OpenCore-ISO bechrieben.
Damit macOS Sonoma ohne separate OpenCore-ISO starten kann, haben wir OpenCore auf der EFI-Partition der virtuellen Systemfestplatte installiert. Der Script **Mount-EFI.command** wurde ausgeführt und die **Boot** und **OS** aus der OpenCore Laufwerk auf der EFI Systemfestplatte kopiert.
## 2. macOS-Installationsmedium über GitHub erstellen

Für die Erstellung des Installationsmediums wurde das GitHub-Projekt **LongQT-sea/macos-iso-builder** verwendet. Das GitHub Projekt lädt den vollständigen macOS-Installer von Apples Servern herunter und erstellt daraus ein bootfähiges ISO-Image. Die Erstellung kann über GitHub Actions ausgeführt werden, sodass dafür kein eigener Mac benötigt wird. 

ISO: **macOS_Sonoma_14.8.9.iso**
Größe: **14.9 GB**

Die MacOS ISO wird an einem CD/DVD Laufwerk in der Proxmox VM Konfigurationa angehängt. Diese Laufwerk steht an der zweite Stelle in der Bootreihenfolge.

Beim Booten wird zuerst **Disk Utility** gestartet, um die Festplatte in Mac geeignete Format zu konfigurieren.
Name: **Mac HardDisk**
Format: **APFS**
Scheme: **GUID Partition Map**

Nach dem Formatieren wird die Installation von MacOS Sonoma gestartet.

## 3. Proxmox VM Konfiguration Besonderheiten

- **Hard Disk**: "SATA 0" (anstatt "virtio-scsi-single") - Sonst ist die Festplatte nicht sichtbar
- **Proxmox SecureBoot**: "Aus"
- **EFI SecureBoot**: "Aus"
- **PX Firewall**: "Aus" - Führt nach gewisse Zeit zum Absturz des Systems wenn aktiv ist
- **CPU**: "host" - Die "x86-64-v2-AES" führt zu ein Kernel Panic
- **Netzwerkadapter**: "VMware vmxnet3" (statt "Intel E1000") - sonst funktioniert die ACK der VM bei der Verbindung mit DHCP Server.

</details>

# Tests
