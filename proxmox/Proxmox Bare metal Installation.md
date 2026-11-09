# 02 - Proxmox Bare-Metal Installation

**Lab Date:** November 5, 2026  
**Status:** Installation completed successfully  
**Platform:** Proxmox Virtual Environment (VE)

---

## 1. Objective

Install Proxmox VE directly onto my TRIGKEY mini PC, replacing Windows 11 Pro, and establish a dedicated virtualization server for my home lab.

The purpose of this installation is to build practical experience with:

- Linux-based server administration
- Bare-metal virtualization
- Hardware and storage configuration
- Network infrastructure
- Remote server management
- Troubleshooting
- Cybersecurity and network security fundamentals

This server will eventually host Linux virtual machines, containers, networking services, monitoring tools, and security applications.

## 2. Hardware Specifications

| Component | Specifications |
|---|---|
| Manufacturer | TRIGKEY |
| Processor | AMD Ryzen 9 6900HX |
| Memory | 16 GB RAM |
| Storage | 500 GB NVMe SSD |
| Networking | 2.5 Gb Ethernet |
| Original Operating System | Windows 11 Pro |
| New Operating System | Proxmox VE |
| Installation Media | Bootable USB drive |

The TRIGKEY is the primary physical server for my home lab.

Because the system has limited resources compared with enterprise servers, virtual machines and containers will need to be sized appropriately.

## 3. What Is Proxmox VE?

Proxmox Virtual Environment is an open-source virtualization platform based on Debian Linux.

It provides a web-based interface for managing virtual machines, Linux containers, storage, networking, and other infrastructure components.

### Type 1 vs. Type 2 Hypervisors

A **Type 1 hypervisor**, commonly called a bare-metal hypervisor, runs directly on physical server hardware.

Examples include:

- Proxmox VE
- VMware ESXi
- Microsoft Hyper-V in a bare-metal deployment

A **Type 2 hypervisor** runs on top of a traditional host operating system.

Examples include:

- Oracle VirtualBox
- VMware Workstation

### Why I Chose Proxmox

Proxmox allows me to create a dedicated virtualization environment without relying on Windows as the host operating system.

This is useful for learning technologies and concepts found in enterprise IT infrastructure.

## 4. Preparing the Installation Media

Before installing Proxmox, I downloaded the Proxmox VE ISO installation image.

I used Rufus on a Windows computer to write the ISO image to a USB flash drive.

### Important Terminology

**ISO Image:** A disk image containing the files needed to install an operating system.

**Rufus:** A utility used to create bootable USB installation media.

**Bootable USB:** A USB device containing software that allows a computer to start an operating system installer.

**UEFI:** Unified Extensible Firmware Interface, firmware used to initialize computer hardware and start an operating system.

### Issue Encountered: USB Partitions

After writing the Proxmox ISO using Rufus, Windows displayed multiple partitions on the USB drive.

This occurred because writing an installation image changes the USB drive's partition structure.

The resulting partitions do not necessarily behave like normal Windows storage volumes.

### What I Learned

Creating a bootable USB is different from simply copying an ISO file onto a USB drive.

Rufus writes the installation image in a structure that allows the computer's firmware to boot from it.

## 5. Starting the Proxmox Installation

With the bootable Proxmox installation USB prepared, I connected it to the TRIGKEY mini PC.

The goal was to boot from the USB installation media instead of the previously installed Windows operating system.

### Installation Process

The installation process involved:

1. Booting the TRIGKEY from the Proxmox installation USB.
2. Launching the Proxmox VE installer.
3. Reviewing the installation options.
4. Selecting the appropriate installation storage device.
5. Configuring system settings.
6. Configuring the initial management network settings.
7. Completing the installation.
8. Rebooting into the installed Proxmox environment.

### Important Warning

Installing Proxmox directly onto the TRIGKEY's internal storage replaces the previous operating system and can erase existing data.

This is why the Windows backup process was
