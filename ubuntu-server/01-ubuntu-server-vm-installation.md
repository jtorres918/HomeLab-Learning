# Ubuntu Server VM Installation

Date: November 9, 2026

Status: Completed

# 01 - Ubuntu Server VM Installation

**Date:** November 9, 2026  
**Project:** HomeLab-Learning  
**Hypervisor:** Proxmox Virtual Environment 9.2.26  
**Physical Host:** Trigkey Mini PC  
**Virtual Machine:** VM 100 - ubuntu-server  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Installation Status:** Successfully Completed

---

## 1. Objective

The objective of this project was to create and install my first Linux virtual machine using Proxmox VE.

This virtual machine will provide a foundation for learning:

- Linux server administration
- Networking and TCP/IP
- Virtualization
- SSH remote administration
- System security and hardening
- Docker and containers
- Automation and scripting
- Monitoring and cybersecurity tools

The goal is to develop hands-on experience relevant to network administration, security engineering, and cloud security.

---

## 2. Home Lab Environment

### Physical Hardware

The home lab runs on a Trigkey Mini PC with Proxmox installed directly on its internal NVMe SSD.

Proxmox is a Type 1 hypervisor, meaning it runs directly on the host hardware rather than inside another desktop operating system.

### Proxmox Host Information

| Component | Configuration |
|---|---|
| Hypervisor | Proxmox VE 9.2.26 |
| Hostname | pve |
| Management IP | 192.168.1.200 |
| Subnet | 192.168.1.0/24 |
| Default gateway | 192.168.1.1 |
| Virtual network bridge | vmbr0 |
| Storage for VM disks | local-lvm |
| Storage for ISO images | local |

Proxmox management interface:

`https://192.168.1.200:8006`

### Home Network

The home network uses a Netgear Nighthawk RAX70 router.

The router provides DHCP services, automatically assigning network addresses to connected devices.

The Proxmox host uses a statically configured IP address, while the new Ubuntu VM initially receives its address through DHCP.

---

## 3. Download Ubuntu Server

Downloaded the Ubuntu Server 26.04.1 LTS installation ISO using a Windows 11 laptop.

**ISO Filename:**

`ubuntu-26.04.1-live-server-amd64.iso`

### What is an ISO?

An ISO is a disk image containing the files required to install an operating system.

Rather than writing the ISO to a physical USB drive, Proxmox can mount it as a virtual CD/DVD drive.

This allows virtual machines to boot directly into an operating system installer.

### Why Ubuntu Server?

Ubuntu Server was selected because:

- It is widely used in enterprise and cloud environments.
- It is suitable for running services without a desktop interface.
- It uses the APT package manager.
- It supports SSH administration.
- It supports Docker and container-based applications.
- It has extensive community and vendor documentation.
- It provides opportunities to learn Linux commands and system security.

---

## 4. Upload Ubuntu ISO to Proxmox

Accessed the Proxmox web interface from the Windows laptop.

Navigation:

`Datacenter → pve → local (pve) → ISO Images`

Selected **Upload**, located the downloaded ISO, and uploaded it to Proxmox.

The upload completed successfully.

### What I learned

Proxmox separates different types of storage.

**local (pve):**

Used for ISO images, container templates, backups, and other file-based content.

**local-lvm (pve):**

Used primarily to store virtual machine disks and container volumes.

The Ubuntu ISO was uploaded to `local`, while the virtual machine's hard disk was created on `local-lvm`.

---

## 5. Create Virtual Machine

Selected:

`Create VM`

Configured the following settings.

### General

| Setting | Value |
|---|---|
| Node | pve |
| VM ID | 100 |
| Name | ubuntu-server |

**VM ID:** A unique identifier Proxmox uses to distinguish virtual machines.

**Node
