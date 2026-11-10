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

**Node:** The physical Proxmox host running the VM.

### Operating System

Selected the Ubuntu Server ISO from `local` storage.

Guest operating system type:

Linux.

### System

Used Proxmox's Linux-compatible virtual hardware settings.

Configured a VirtIO SCSI controller for the virtual hard disk.

### Disks

| Setting | Value |
|---|---|
| Bus/Device | SCSI |
| SCSI Controller | VirtIO SCSI single |
| Storage | local-lvm |
| Disk size | 32 GiB |
| Cache | Default (No cache) |
| Discard | Enabled |
| IO thread | Enabled |

#### SCSI

SCSI stands for Small Computer System Interface.

In Proxmox, SCSI is one of the supported ways to present virtual storage devices to a guest operating system.

#### VirtIO SCSI

VirtIO provides virtualization-optimized device interfaces.

Using VirtIO SCSI helps provide efficient virtual disk access.

#### Discard

Discard allows the guest operating system to inform the storage layer when blocks are no longer needed.

This can help reclaim storage space when supported by the underlying storage configuration.

#### IO Thread

An IO thread allows supported virtual disk operations to be handled by a dedicated thread.

This can improve disk I/O efficiency in certain workloads.

### CPU

| Setting | Value |
|---|---|
| Sockets | 1 |
| Cores | 2 |
| CPU Type | host |
| Total vCPUs | 2 |

#### What is a vCPU?

A virtual CPU is a processing resource presented to a virtual machine.

The hypervisor schedules virtual CPU execution using the physical host's available CPU resources.

The VM was allocated two virtual CPUs.

#### Why use CPU type `host`?

The `host` CPU setting exposes supported features of the physical processor to the virtual machine.

This can improve performance and compatibility with CPU-specific instructions, although it may reduce portability between different physical hosts.

### Memory

| Setting | Value |
|---|---|
| RAM | 2048 MiB |
| Equivalent | 2 GiB |

Two GiB was selected as an initial allocation appropriate for a small Linux server learning environment.

Memory usage and VM performance will be monitored as additional services are installed.

### Network

| Setting | Value |
|---|---|
| Bridge | vmbr0 |
| Model | VirtIO (paravirtualized) |
| VLAN Tag | None |
| MAC Address | Automatically generated |
| Firewall flag | Enabled |

#### vmbr0

`vmbr0` is the Proxmox virtual network bridge.

It connects the virtual network adapter to the host's bridged network.

This allows the Ubuntu VM to communicate with other devices on the home LAN.

#### VirtIO Network Adapter

VirtIO is a virtualization-optimized network interface.

It reduces some overhead associated with emulated hardware and is supported by modern Linux systems.

#### VLAN

VLAN stands for Virtual Local Area Network.

No VLAN tag was configured for this initial VM.

VLAN segmentation will be studied and implemented during later networking projects.

#### Firewall

The Proxmox virtual NIC firewall flag was enabled.

This flag does not, by itself, establish a complete firewall policy. Actual filtering depends on the relevant Proxmox firewall configuration and whether firewall enforcement is enabled.

---

## 6. Confirm VM Configuration

Reviewed the settings on the confirmation screen.

| Component | Final Configuration |
|---|---|
| VM ID | 100 |
| Name | ubuntu-server |
| CPU | 2 cores |
| CPU Type | host |
| RAM | 2048 MiB |
| Disk | 32 GiB |
| Disk Storage | local-lvm |
| Network | VirtIO |
| Bridge | vmbr0 |
| ISO | Ubuntu Server 26.04.1 LTS |
| Controller | VirtIO SCSI single |

Clicked **Finish**.

The virtual machine was successfully created.

---

## 7. Start the Ubuntu Installation

Selected:

`VM 100 (ubuntu-server) → Start`

Opened the Proxmox console.

The Ubuntu Server installer booted from the mounted ISO.

### Installation Type

Selected:

**Ubuntu Server**

Did not select the minimized installation.

The standard server installation includes a broader selection of commonly used packages and utilities.

### Third-Party Drivers

Left the third-party driver option unchecked.

The virtual machine uses virtual hardware supported by Ubuntu.

---

## 8. Configure Ubuntu Networking

During installation, Ubuntu detected the network interface:

`ens18`

The interface received an address automatically through DHCP.

### Network Configuration

| Property | Value |
|---|---|
| Interface | ens18 |
| IPv4 Address | 192.168.1.17 |
| Prefix | /24 |
| Subnet Mask | 255.255.255.0 |
| Gateway | 192.168.1.1 |
| IP Assignment | DHCP |

### What is DHCP?

DHCP stands for Dynamic Host Configuration Protocol.

DHCP allows devices to automatically receive network configuration such as:

- IP address
- Subnet mask
- Default gateway
- DNS server information

The Netgear router assigned the Ubuntu VM an address of `192.168.1.17`.

This is a dynamically assigned address and may change.

A DHCP reservation will be configured in a future networking step.

### Why does the Ubuntu VM have a different IP from Proxmox?

Proxmox is the physical host and has the management IP:

`192.168.1.200`

Ubuntu is a separate virtual machine with its own virtual network adapter.

It received:

`192.168.1.17`

Although both systems run on the same physical computer, they operate as separate network endpoints.

---

## 9. Configure Proxy

At the Ubuntu installer proxy configuration screen, left the proxy address blank.

### What is a proxy?

A proxy server is an intermediary between a client and another network service.

Organizations sometimes use proxies for traffic filtering, monitoring, caching, and access control.

This home network does not require an HTTP proxy for Ubuntu's internet access.

---

## 10. Configure Ubuntu Package Mirror

The installer selected:

`http://us.archive.ubuntu.com/ubuntu/`

The mirror initially passed the installer connectivity test.

### What is a package mirror?

A package mirror is a server hosting copies of software packages and repository information.

Ubuntu uses these servers to download system components, software, and updates.

During installation, package downloads later appeared to stall. This is documented separately in:

`02-ubuntu-installation-troubleshooting.md`

The installer ultimately completed without manually changing the mirror.

---

## 11. Configure Storage

Selected:

**Use an entire disk**

The installer detected the Proxmox virtual hard drive:

`QEMU_QEMU_HARDDISK_drive-scsi0`

Disk size:

`32.00 GB`

### Storage Options

Selected:

- Use entire virtual disk
- Set up the disk as an LVM group

Did not enable LUKS encryption.

### What is LVM?

LVM stands for Logical Volume Manager.

LVM provides a flexible way to manage disk space.

Rather than assigning all storage directly to fixed partitions, LVM allows space to be managed through physical volumes, volume groups, and logical volumes.

### Storage Layout

The installation produced approximately:

| Component | Size |
|---|---|
| BIOS boot partition | 1 MiB |
| /boot partition | 2 GiB |
| LVM volume group | 29.996 GiB |
| Root logical volume | 14.996 GiB |
| Initial free space within LVM | 15 GiB |

The root filesystem uses ext4.

### Understanding the Linux Filesystem

**`/`**

The root directory of the Linux filesystem.

All other directories exist beneath this filesystem hierarchy.

**`/boot`**

Contains boot-related files, including Linux kernels and other components required during startup.

**ext4**

A widely used Linux filesystem.

**Free LVM space**

Storage available within the volume group that can potentially be allocated to logical volumes later.

This should be reviewed before running storage-heavy applications.

### Important Distinction

The 32 GiB disk belongs to Ubuntu VM 100.

It is not the entire physical SSD inside the Trigkey.

Proxmox manages the physical host storage and provides a virtual disk to the Ubuntu guest.

---

## 12. Configure Ubuntu User Profile

Created a Linux user account during installation.

| Setting | Value |
|---|---|
| Server Name | ubuntu-server |
| Username | jansen |
| Password | Not documented |

The user account is used for normal Linux administration.

Administrative commands can be executed using `sudo` when authorized.

### Understanding sudo

`sudo` allows an authorized user to execute commands with elevated privileges.

For example:

`sudo apt update`

The Ubuntu user password is required when prompted.

Passwords are intentionally not included in this GitHub repository.

---

## 13. Ubuntu Pro

An existing Ubuntu Pro personal account was attached during installation.

Ubuntu Pro provides additional security maintenance capabilities.

The subscription and services were later verified using:

`sudo pro status`

The verification and update process will be documented in:

`03-ubuntu-networking-and-updates.md`

---

## 14. Configure OpenSSH Server

Selected:

**Install OpenSSH Server**

Initially allowed password authentication during installation.

This enabled the first remote connection from Windows.

Later, ED25519 SSH key authentication was configured and password-based SSH login was disabled.

The complete security procedure is documented separately in:

`04-ssh-configuration-and-hardening.md`

### What is SSH?

SSH stands for Secure Shell.

SSH is a protocol for encrypted remote administration.

It allows a Windows computer to connect to a Linux server and execute commands without directly using the Proxmox console.

---

## 15. Featured Server Snaps

The installer offered optional packages including:

- MicroK8s
- Nextcloud
- Docker-related alternatives
- Prometheus
- LXD
- Other server applications

No featured snaps were selected during installation.

### Reason

The goal is to learn the installation and administration of services manually.

Docker, monitoring, and other tools will be installed and documented individually in future home lab projects.

---

## 16. Complete the Installation

After the configuration screens, Ubuntu began installing the operating system.

The installer performed several operations:

- Extracted the Ubuntu system image
- Configured APT repositories
- Installed required packages
- Installed and configured the Linux kernel
- Configured networking
- Configured storage
- Installed the GRUB bootloader
- Installed OpenSSH Server
- Processed system updates

### Installation Issue

The kernel download appeared stalled for an extended period.

Network tests were performed from both the Proxmox host and the Ubuntu VM.

The installation eventually progressed and completed successfully without manually restarting the VM.

The full diagnostic procedure and test results are recorded in:

`02-ubuntu-installation-troubleshooting.md`

---

## 17. Reboot Ubuntu

After installation completed, selected:

**Reboot Now**

Ubuntu rebooted from its newly installed virtual hard disk.

The login prompt appeared.

Logged in successfully using the Ubuntu account created during installation.

---

## 18. Verify Installation

After rebooting, Ubuntu reported:

| Property | Result |
|---|---|
| Operating System | Ubuntu Server 26.04.1 LTS |
| Kernel | 7.0.0-43-generic |
| Hostname | ubuntu-server |
| User | jansen |
| Network Interface | ens18 |
| IPv4 Address | 192.168.1.17 |
| Root Filesystem | Approximately 14.66 GiB |

### Verification Commands

Display current user:

`whoami`

Expected result:

`jansen`

Display server hostname:

`hostname`

Expected result:

`ubuntu-server`

Display IPv4 information:

`ip -4 addr`

Expected result:

`192.168.1.17/24`

Inspect the Linux kernel:

`uname -r`

Expected installed kernel:

`7.0.0-43-generic`

### Command Explanations

**whoami:** Shows the currently authenticated Linux username.

**hostname:** Displays the computer's configured hostname.

**ip:** Linux command-line utility for network configuration and inspection.

**-4:** Restricts output to IPv4 information.

**addr:** Displays network interface addresses.

**uname:** Displays information about the operating system and kernel.

**-r:** Displays the Linux kernel release.

---

## 19. Initial Resource Usage

After the installation and reboot, Ubuntu reported relatively low memory usage while idle.

This was lower than the memory usage observed during the installer process.

This demonstrated the importance of comparing resource usage during different workload conditions.

Installing the operating system is more resource-intensive than running an otherwise idle server.

Resource monitoring will be revisited after deploying Docker and other services.

---

## 20. Installation Verification Checklist

- [x] Ubuntu Server ISO downloaded.
- [x] ISO uploaded to Proxmox.
- [x] VM 100 created.
- [x] Two virtual CPUs allocated.
- [x] Two GiB RAM allocated.
- [x] 32 GiB virtual disk created.
- [x] VirtIO network adapter configured.
- [x] vmbr0 bridge selected.
- [x] Ubuntu Server installation started.
- [x] DHCP networking established.
- [x] LVM storage configured.
- [x] Ubuntu user account created.
- [x] OpenSSH Server installed.
- [x] Kernel installation troubleshooting completed.
- [x] Ubuntu installation completed.
- [x] VM rebooted successfully.
- [x] Ubuntu login verified.
- [x] Hostname and IPv4 address verified.

---

## 21. What I Learned

This project provided hands-on experience with:

1. Creating virtual machines using Proxmox VE.
2. Understanding the relationship between a hypervisor and guest operating systems.
3. Assigning virtual CPUs, RAM, storage, and networking resources.
4. Using ISO images for operating system installation.
5. Understanding VirtIO and virtual hardware.
6. Configuring Ubuntu Server without a graphical desktop.
7. Understanding DHCP and virtual network interfaces.
8. Using LVM for Linux storage.
9. Understanding the Linux filesystem hierarchy.
10. Configuring Linux users and OpenSSH.
11. Troubleshooting installation problems without immediately restarting.
12. Verifying Linux system information from the command line.

---

## 22. Next Steps

The Ubuntu Server installation is complete.

The next documentation files are:

- `02-ubuntu-installation-troubleshooting.md`
- `03-ubuntu-networking-and-updates.md`
- `04-ssh-configuration-and-hardening.md`

Future technical objectives include:

- DHCP reservation for the Ubuntu VM
- Linux firewall configuration
- Docker installation
- Linux automation and scripting
- Monitoring and centralized logging
- Network segmentation and VLANs
- Security analysis and testing

**Final Result:** Successfully deployed Ubuntu Server 26.04.1 LTS as the first Linux virtual machine in the Proxmox home lab.
