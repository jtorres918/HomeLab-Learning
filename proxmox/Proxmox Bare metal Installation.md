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

This is why the Windows backup process was completed before installation.

The backup procedure is documented in File 01.

## 6. Storage Installation

Proxmox was installed onto the TRIGKEY's internal 500 GB NVMe SSD.

### What Is NVMe?

NVMe stands for Non-Volatile Memory Express.

It is a high-performance storage protocol designed for solid-state drives.

### Why Storage Matters

Proxmox uses storage for several purposes:

- Host operating system files
- Virtual machine disk images
- Linux container filesystems
- ISO installation images
- Backups
- Configuration and operational data

Different storage configurations are possible in Proxmox, including LVM-thin, ZFS, and directory-based storage.

The exact storage layout selected during installation should be verified in the Proxmox interface before documenting it as the final configuration.

## 7. Initial Network Configuration

Proxmox requires network configuration so the host can be accessed and managed remotely.

During the installation, network settings were configured for the home network.

Important settings include:

| Setting | Purpose |
|---|---|
| Hostname | Identifies the server |
| IP Address | Identifies the server on the network |
| Subnet Mask / CIDR | Defines the network address range |
| Default Gateway | Provides access to other networks |
| DNS Server | Resolves domain names into IP addresses |

A static management IP address is particularly useful because it provides a consistent address for accessing the Proxmox server.

Detailed router and DHCP configuration will be documented separately in File 03.

## 8. Completing Installation and Rebooting

After completing the installation process, the TRIGKEY rebooted into Proxmox VE.

Unlike Windows 11, Proxmox does not require a traditional desktop environment for routine administration.

The host runs a Linux-based operating system and provides a web management interface.

### Important Concept: Headless Server

A headless server is a computer that operates without requiring a directly connected monitor, keyboard, or mouse for normal management.

After initial setup, Proxmox can be administered remotely from another computer.

This is an important concept in enterprise server infrastructure.

## 9. Accessing the Proxmox Web Interface

Once Proxmox was running, I accessed its web management interface from a separate Windows computer.

The default Proxmox management URL uses the following format:

\`\`\`text
https://<SERVER-IP-ADDRESS>:8006
\`\`\`

### Understanding the URL

**HTTPS:** Encrypted communication protocol used by web browsers.

**Server IP Address:** The network address assigned to the Proxmox host.

**Port 8006:** The default TCP port used for the Proxmox web interface.

### Authentication

Proxmox supports several authentication methods.

For the initial installation, the Linux PAM authentication realm commonly uses the root administrator account.

The root account has extensive administrative privileges and must be protected with a strong password.

Passwords and authentication secrets must never be committed to GitHub.

## 10. Initial Web Interface Navigation

After accessing Proxmox through the browser, I began exploring its management interface.

Important areas include:

### Datacenter

The top-level management area for Proxmox infrastructure.

### Node

Represents an individual physical Proxmox server.

My TRIGKEY is currently the primary node in the home lab.

### Storage

Displays storage resources used for virtual machines, containers, ISO images, and other data.

### Network

Provides access to network interface and bridge configuration.

### Shell

Allows administrators to execute Linux commands directly on the Proxmox host.

### Repositories and Updates

Provide access to software package sources and system updates.

These areas will be covered in more detail in future documentation.

## 11. Troubleshooting and Challenges

### Challenge 1: Understanding Installation Media

Initially, I had to learn the difference between downloading an ISO file and creating bootable installation media.

**Resolution:** Used Rufus to write the Proxmox ISO image to USB media.

**Lesson:** Operating system installers usually require bootable media rather than a simple file copy.

### Challenge 2: Replacing Windows Safely

Installing a bare-metal hypervisor required removing the existing Windows operating system.

**Resolution:** Backed up Windows activation information, drivers, installed program inventory, system information, network configuration, and important personal files before proceeding.

**Lesson:** Data protection and recovery preparation are essential before modifying server operating systems.

### Challenge 3: Understanding Remote Server Administration

Proxmox uses a different management approach from a traditional Windows desktop.

**Resolution:** Accessed the Proxmox web management interface through another computer on the network.

**Lesson:** Servers can be managed remotely without a locally running graphical desktop.

### Challenge 4: Learning Linux Administration

Proxmox uses Debian Linux as its foundation.

The administration process introduced Linux-based commands, configuration, and troubleshooting practices.

**Lesson:** Understanding Linux fundamentals is important for managing virtualization and network security infrastructure.

## 12. Installation Verification

The installation was considered successful after the following milestones:

- Proxmox installed onto the TRIGKEY.
- The TRIGKEY rebooted into the Proxmox environment.
- Proxmox was reachable from a separate Windows computer.
- The web management interface was accessible.
- Initial host configuration could continue.

These milestones confirmed that the physical system was operating as a Proxmox host.

## 13. What I Learned

This installation provided hands-on experience with:

1. Preparing bootable Linux installation media.
2. Understanding BIOS/UEFI boot processes.
3. Installing a bare-metal hypervisor.
4. Preparing and protecting data before replacing an operating system.
5. Understanding host storage.
6. Understanding server management networking.
7. Accessing a remote web-based administration interface.
8. Recognizing the difference between physical hardware and virtualized infrastructure.
9. Understanding fundamental Linux server administration concepts.
10. Documenting technical procedures and troubleshooting steps using GitHub and Markdown.

## 14. Relevance to Cybersecurity

Virtualization is an important foundation for cybersecurity laboratories.

With Proxmox installed, I can eventually build isolated environments for:

- Network security testing
- Linux administration
- Firewall configuration
- Security monitoring
- Log analysis
- Vulnerability management
- Network segmentation
- Automation and scripting

A virtualized lab makes it possible to learn how systems communicate while limiting the risk of accidentally disrupting production or household devices.

## 15. Current Status

**Installation: Completed**

The TRIGKEY is running Proxmox VE, and its management interface has been accessed successfully.

Initial network and software configuration have also been started.

### Next Documentation

**File 03 - Proxmox Network Configuration**

Topics will include:

- NETGEAR Nighthawk RAX70 settings
- DHCP address range
- Static management IP planning
- Subnet and gateway configuration
- Accessing Proxmox over the LAN
- Troubleshooting network connectivity

---

## Lab Summary

This exercise transformed the TRIGKEY mini PC from a Windows desktop system into a dedicated Proxmox virtualization host.

The installation established the foundation for building a practical home lab focused on Linux, networking, virtualization, security engineering, and automation.

Future exercises will build on this infrastructure by deploying and securing virtual machines, containers, and networking services.
