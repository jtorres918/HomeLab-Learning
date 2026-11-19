# 03 - Ubuntu Server Networking, Updates, and System Verification

**Project:** HomeLab-Learning  
**Hypervisor:** Proxmox VE 9.2.26  
**Virtual Machine:** VM 100 - ubuntu-server  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Status:** Completed — Networking verified and system updated

---

## 1. Objective

After installing Ubuntu Server, the next objective was to verify network connectivity, inspect the Linux network configuration, apply system updates, and confirm that Ubuntu Pro security services were enabled.

This exercise introduced foundational Linux administration and networking concepts, including:

- IPv4 addressing and CIDR notation
- Subnets and subnet masks
- Default gateways and routing tables
- DHCP
- Virtual network interfaces
- SSH connectivity
- Linux package management with APT
- Ubuntu Pro security services
- Reboot verification

All commands were performed against the Ubuntu VM unless otherwise noted.

---

## 2. Home Lab Network Overview

### Network Configuration

| Component | IP Address | Purpose |
|---|---|---|
| Netgear Nighthawk RAX70 | `192.168.1.1` | Router / default gateway |
| Proxmox host | `192.168.1.200` | Virtualization server |
| Ubuntu Server VM 100 | `192.168.1.22` | Linux virtual machine (current DHCP reservation) |
| Windows laptop | `192.168.1.12` | Administration workstation |
| Local subnet | `192.168.1.0/24` | Home network |

The Ubuntu Server virtual machine runs inside Proxmox, but it has its own IP address and network identity.

> **Current network update — November 16, 2026:** Ubuntu VM 100 originally used the DHCP-assigned address `192.168.1.17`. After a power outage it received `192.168.1.22`, which is now reserved on the NETGEAR RAX70 for MAC address `BC:24:11:23:80:D9`. Historical installation commands and captured output showing `.17` are intentionally preserved. The current address for SSH and lab connectivity is `192.168.1.22/24`.


### Network Path

```text
Windows Laptop
192.168.1.12
       |
       | Home LAN
       |
Netgear RAX70
192.168.1.1
       |
       | Ethernet / LAN
       |
Trigkey Mini PC
Proxmox Host: 192.168.1.200
       |
       | Virtual Bridge: vmbr0
       |
Ubuntu Server VM 100
192.168.1.22
Interface: ens18
```

**Important:** This diagram illustrates the physical and virtual connectivity. Devices on the same LAN can communicate through Layer 2 switching without sending their traffic through the router's Layer 3 default gateway.

---

## 3. Understanding IPv4 Addresses

At the time of this original networking exercise, Ubuntu's IPv4 address was:

```text
192.168.1.17/24
```

The current reserved DHCP address is `192.168.1.22/24`.

An IPv4 address contains 32 bits divided into four groups of eight bits, called octets.

### Understanding `/24`

The `/24` is CIDR notation.

CIDR stands for **Classless Inter-Domain Routing**.

A `/24` prefix means that the first 24 bits identify the network portion of the address.

For this subnet:

```text
192.168.1.17/24
|--------|  |
 Network   Host
```

With `/24`, the first three octets are the network portion.

The last octet identifies the host.

### Subnet Information

| Property | Value |
|---|---|
| Ubuntu IPv4 (current) | `192.168.1.22` |
| Network address | `192.168.1.0` |
| Prefix length | `/24` |
| Subnet mask | `255.255.255.0` |
| First usable address | `192.168.1.1` |
| Last usable address | `192.168.1.254` |
| Broadcast address | `192.168.1.255` |
| Usable host addresses | 254 |

### What I Learned

Devices within the same subnet can communicate directly using local network delivery, assuming no VLAN isolation or firewall rule prevents communication.

For example:

- Ubuntu: `192.168.1.17/24`
- Proxmox: `192.168.1.200/24`

Both belong to `192.168.1.0/24`.

They do not need the default gateway to route packets between their IP addresses.

However, a device at `192.168.2.50` is on a different `/24` subnet.

Ubuntu would normally send traffic for that destination to its default gateway, unless another more-specific route exists.

---

## 4. Understanding DHCP

DHCP stands for **Dynamic Host Configuration Protocol**.

DHCP automatically provides network configuration information to devices.

This may include:

- IPv4 address
- Subnet mask
- Default gateway
- DNS servers
- Lease duration

The Netgear RAX70 provides DHCP services on the home network.

During installation, Ubuntu automatically received:

```text
192.168.1.17/24
```

The Ubuntu network interface was configured dynamically.

### Dynamic vs Static IP Addresses

**Dynamic IP address**

An address assigned through DHCP that may change when its lease or network configuration changes.

**Static IP address**

An address configured manually on a device.

**DHCP reservation**

A setting on the DHCP server that associates a particular device's MAC address with a specific IP address.

A DHCP reservation lets a device continue using DHCP while normally receiving the same address.

### Current Home Lab Configuration

| Device | Address Assignment |
|---|---|
| Proxmox | Static IP |
| Ubuntu Server | DHCP |
| Windows laptop | DHCP |

**Future Task:** Configure a DHCP reservation for the Ubuntu VM so its IP address remains consistent.

This step has not yet been performed.

---

## 5. Understanding Network Interfaces

Ubuntu detected the following virtual network interface:

```text
ens18
```

This interface is connected to Proxmox's virtual network bridge:

```text
vmbr0
```

The virtual bridge connects Ubuntu to the home LAN.

### What is a Network Interface?

A network interface is the connection point through which a computer communicates with a network.

Interfaces may represent:

- Physical Ethernet adapters
- Wi-Fi adapters
- Virtual Ethernet adapters
- Loopback interfaces
- VLAN interfaces
- Virtual private network interfaces

### Interface Naming

Linux uses interface names such as:

- `lo`
- `ens18`
- `eth0`
- `wlan0`

The names depend on the system and its network configuration.

### Loopback Interface

```text
lo
```

The loopback interface allows the computer to communicate with itself.

A common IPv4 loopback address is:

```text
127.0.0.1
```

Loopback communication does not require packets to leave the computer through the physical network.

---

## 6. Inspect IPv4 Addresses

Command:

```bash
ip -4 addr
```

### Command Explanation

| Component | Meaning |
|---|---|
| `ip` | Linux networking utility |
| `-4` | Display IPv4 information |
| `addr` | Display network interface addresses |

### Observed Results

Ubuntu displayed the loopback address:

```text
127.0.0.1/8
```

And the IPv4 address on `ens18`:

```text
192.168.1.17/24
```

Additional interface information included:

```text
scope global dynamic
```

### Terminology

**`inet`**

Identifies an IPv4 address.

**`inet6`**

Identifies an IPv6 address.

**`scope global`**

Indicates the address has global scope in the Linux address model. It does not mean the address is publicly routable on the internet.

**`dynamic`**

Indicates that the address was dynamically assigned.

**`brd`**

Represents the IPv4 broadcast address.

**`metric`**

A value used in route preference calculations.

---

## 7. Understanding the Default Gateway

The Ubuntu default gateway is:

```text
192.168.1.1
```

This is the IP address of the Netgear Nighthawk RAX70 router.

### What is a Default Gateway?

A default gateway is the next-hop router used when a device has no more-specific route to a destination.

For example, if Ubuntu wants to communicate with:

```text
8.8.8.8
```

It recognizes that `8.8.8.8` is outside its local subnet.

Ubuntu therefore forwards the packet toward:

```text
192.168.1.1
```

The router then determines how to forward it toward the destination.

### Important Distinction

The Ubuntu VM's traffic passes through Proxmox's virtual bridge because Ubuntu runs on the Proxmox host.

However, Proxmox is not acting as Ubuntu's default IP gateway.

The Netgear RAX70 is the default gateway.

**Switching and bridging operate primarily at Layer 2. Routing operates at Layer 3.**

---

## 8. Inspect Ubuntu's Routing Table

Command:

```bash
ip route
```

This displays the Linux kernel's IPv4 routing table.

### Actual Output

```text
default via 192.168.1.1 dev ens18 proto dhcp src 192.168.1.17 metric 100
192.168.1.0/24 dev ens18 proto kernel scope link src 192.168.1.17 metric 100
192.168.1.1 dev ens18 proto dhcp scope link src 192.168.1.17 metric 100
```

### Route 1 — Default Route

```text
default via 192.168.1.1 dev ens18 proto dhcp src 192.168.1.17 metric 100
```

| Field | Meaning |
|---|---|
| `default` | Route used when no more-specific destination route matches |
| `via 192.168.1.1` | Next-hop router |
| `dev ens18` | Outgoing network interface |
| `proto dhcp` | Route learned through DHCP |
| `src 192.168.1.17` | Preferred source IP address |
| `metric 100` | Route preference value |

In plain English:

If Ubuntu does not have a more-specific route to a destination, send the traffic to the Netgear router through interface `ens18`.

### Route 2 — Directly Connected Subnet

```text
192.168.1.0/24 dev ens18 proto kernel scope link src 192.168.1.17 metric 100
```

This route identifies Ubuntu's directly connected local network.

**`proto kernel`** means the route was created by the Linux kernel as part of interface configuration.

**`scope link`** indicates that the destination is directly reachable through the specified network link.

Ubuntu can communicate directly with other reachable devices in `192.168.1.0/24`.

### Route 3 — Specific Gateway Address

```text
192.168.1.1 dev ens18 proto dhcp scope link src 192.168.1.17 metric 100
```

This is a DHCP-installed route specifically identifying `192.168.1.1` as reachable on the local link.

It is more specific than the general `/24` route.

### How Linux Selects a Route

Linux generally chooses the **longest matching prefix**.

This means the most-specific matching route takes priority.

For example:

Destination: `192.168.1.50`

Ubuntu selects:

```text
192.168.1.0/24 dev ens18
```

Destination: `8.8.8.8`

Ubuntu selects:

```text
default via 192.168.1.1 dev ens18
```

### Understanding Metrics

A routing metric helps select between otherwise comparable routes.

Generally, a lower metric is preferred when multiple eligible routes have the same prefix length.

However, route specificity is considered before comparing metrics for ordinary routing decisions.

### What I Learned

The routing table does not show the exact path packets traveled through the network.

Instead, it shows the rules Linux uses to determine where packets should be forwarded next.

---

## 9. Verify SSH Connectivity from Windows

OpenSSH Server was installed during the Ubuntu Server installation.

The SSH service was verified using:

```bash
sudo systemctl status ssh
```

Result:

```text
Active: active (running)
```

### Initial Windows SSH Connection

From PowerShell:

```powershell
ssh jansen@192.168.1.17
```

Successfully connected using the Ubuntu account credentials.

### Verify Remote Identity

After connecting:

```bash
whoami
```

Result:

```text
jansen
```

Command:

```bash
hostname
```

Result:

```text
ubuntu-server
```

These commands confirmed that the SSH session was connected to the intended Linux server and account.

### Later Improvement

Configured a Windows SSH shortcut:

```powershell
ssh ubuntu-lab
```

SSH public-key authentication and hardening are documented separately in:

`04-ssh-configuration-and-hardening.md`

---

## 10. Update Ubuntu Package Information

Ubuntu uses the APT package management system.

APT stands for **Advanced Package Tool**.

### Refresh Package Information

Command:

```bash
sudo apt update
```

### Command Explanation

| Component | Meaning |
|---|---|
| `sudo` | Execute with elevated privileges |
| `apt` | Package management utility |
| `update` | Refresh package index information |

This command downloads updated package metadata from configured software repositories.

It does not install the available upgrades.

### Actual Result

Ubuntu reported:

```text
47 packages can be upgraded
```

This indicated that updates were available for installed packages.

---

## 11. Install Available Package Upgrades

Command:

```bash
sudo apt upgrade
```

### What It Does

APT evaluates installed packages and available newer versions in the configured repositories.

The `upgrade` operation installs available upgrades where dependencies and package-management rules allow.

### Actual Results

| Operation | Result |
|---|---|
| Packages upgraded | 47 |
| New packages installed | 0 |
| Packages removed | 0 |
| Packages not upgraded | 0 |
| Download size | Approximately 65.9 MB |

The upgrade completed successfully.

### Difference Between Update and Upgrade

**`sudo apt update`**

Refreshes the package lists.

**`sudo apt upgrade`**

Installs available updates to installed packages.

An easy way to remember:

**Update checks what's available. Upgrade installs newer versions.**

---

## 12. Verify All Packages Are Up to Date

After installing the available upgrades, ran:

```bash
sudo apt update
```

Result:

```text
All packages are up to date.
```

This confirmed that no newer package versions were currently available through the configured repositories under that package check.

### Additional Useful Command

```bash
apt list --upgradable
```

Displays installed packages for which upgrades are available.

This is useful when reviewing pending software updates.

---

## 13. Check Whether a Reboot Is Required

After package upgrades, some Linux systems may require a restart.

For example, a reboot may be required to activate a newly installed Linux kernel.

Executed:

```bash
test -f /run/reboot-required && echo "Reboot required" || echo "No reboot required"
```

### Actual Result

```text
No reboot required
```

### Command Breakdown

**`test -f /run/reboot-required`**

Checks whether the regular file `/run/reboot-required` exists.

Ubuntu uses this file as an indication that a reboot is requested following certain package changes.

**`&&`**

Runs the next command if the preceding command succeeds.

**`||`**

Runs the next command if the preceding command fails.

**`echo`**

Prints text to the terminal.

### What I Learned

Linux can indicate whether a reboot is required after package changes.

However, the absence of this file does not prove that every running process is using its newest available software version.

---

## 14. Verify Ubuntu Pro

Ubuntu Pro was attached during the Ubuntu Server installation.

To inspect its status, ran:

```bash
sudo pro status
```

### Actual Results

| Service | Status |
|---|---|
| ESM Apps | Enabled |
| ESM Infra | Enabled |
| Livepatch | Enabled |
| Landscape | Disabled |
| Anbox Cloud | Disabled |

The Ubuntu Pro personal subscription was active.

### What is Ubuntu Pro?

Ubuntu Pro is a subscription service from Canonical that provides additional security maintenance and related services for supported Ubuntu releases.

The personal subscription was used for this home lab.

### ESM Apps

ESM stands for **Expanded Security Maintenance**.

ESM Apps provides additional security maintenance for supported packages in Ubuntu's Universe repository.

### ESM Infra

ESM Infra provides extended security maintenance for supported infrastructure packages.

### Livepatch

Livepatch can apply certain Linux kernel security fixes without an immediate reboot.

It does not eliminate every reason to reboot a Linux server.

### Landscape

Landscape is a Canonical platform for managing and monitoring Ubuntu systems.

It was not enabled in this installation.

### Anbox Cloud

Anbox Cloud supports Android application workloads in cloud environments.

It was not enabled.

---

## 15. Linux Command Fundamentals

Several command-line concepts were introduced during installation and verification.

### PowerShell vs. Linux Shell

PowerShell is the shell used on the Windows administration workstation.

After connecting with SSH, commands are executed in the Ubuntu Linux shell.

For example:

```powershell
ssh ubuntu-lab
```

Connects from Windows to Ubuntu.

Then:

```bash
ip route
```

Executes on Ubuntu.

### The Pipe Operator

Example:

```bash
sudo sshd -T | grep passwordauthentication
```

The pipe character:

```text
|
```

Sends standard output from the command on the left into the command on the right.

### Grep

`grep` searches text for matching patterns.

Example:

```bash
ip -4 addr | grep inet
```

This filters the command output for lines containing `inet`.

### grep -E

The `-E` option enables extended regular expressions.

Example:

```bash
sudo sshd -T | grep -E '^(passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication) '
```

This command was used during SSH hardening to display selected authentication settings.

### Useful Linux Commands

| Command | Purpose |
|---|---|
| `whoami` | Display current username |
| `hostname` | Display system hostname |
| `ip -4 addr` | Display IPv4 address information |
| `ip route` | Display IPv4 routing table |
| `ping -c 4 192.168.1.1` | Test gateway ICMP connectivity |
| `sudo apt update` | Refresh package metadata |
| `sudo apt upgrade` | Install available package upgrades |
| `sudo pro status` | Display Ubuntu Pro service status |
| `sudo systemctl status ssh` | Inspect SSH service |
| `uname -r` | Display running Linux kernel version |
| `exit` | End the shell or SSH session |

---

## 16. Security Considerations

The Ubuntu VM currently resides on the home LAN.

It has the following configuration:

| Security Area | Current State |
|---|---|
| OpenSSH Server | Installed and running |
| Ubuntu security updates | Applied |
| Ubuntu Pro | Enabled |
| SSH key authentication | Configured |
| SSH password authentication | Disabled |
| Public internet SSH forwarding | Not configured |
| DHCP reservation | Not configured |
| Guest firewall policy | To be reviewed |

The SSH hardening process is documented in:

`04-ssh-configuration-and-hardening.md`

### Important Practices

- Apply security updates regularly.
- Avoid exposing administrative services directly to the public internet.
- Use SSH public-key authentication.
- Protect private SSH keys and passwords.
- Verify firewall behavior rather than assuming a checkbox provides complete protection.
- Document network changes.
- Maintain working backups before high-risk modifications.

---

## 17. Final Verification

The following checks were successfully completed:

- [x] Confirmed Ubuntu hostname.
- [x] Confirmed Ubuntu username.
- [x] Verified IPv4 address `192.168.1.17`.
- [x] Identified network interface `ens18`.
- [x] Identified local subnet `192.168.1.0/24`.
- [x] Confirmed default gateway `192.168.1.1`.
- [x] Inspected Linux routing table.
- [x] Understood local versus remote subnet routing.
- [x] Confirmed OpenSSH service running.
- [x] Established Windows-to-Ubuntu SSH connectivity.
- [x] Refreshed APT package metadata.
- [x] Installed 47 package upgrades.
- [x] Confirmed packages were up to date.
- [x] Checked reboot-required status.
- [x] Verified Ubuntu Pro service status.

---

## 18. What I Learned

This project strengthened my understanding of:

1. How virtual machines connect to a physical LAN through Proxmox.
2. Why Proxmox and Ubuntu have separate IP addresses.
3. How IPv4 addresses and subnet masks identify networks.
4. What the `/24` CIDR prefix means.
5. How devices communicate on the same subnet.
6. When a default gateway is required.
7. How to read a Linux routing table.
8. Why route specificity matters.
9. How DHCP assigns network settings.
10. How SSH enables remote Linux administration.
11. How Linux packages are maintained using APT.
12. How Ubuntu signals that a reboot is required.
13. How Ubuntu Pro provides additional security maintenance.

The most important networking concept learned was the distinction between local network communication and communication requiring a router.

---

## 19. Next Steps

### Documentation

- [x] Ubuntu installation documented.
- [x] Installation troubleshooting documented.
- [x] Ubuntu networking and updates documented.
- [ ] SSH configuration and hardening documentation.

### Technical Work

- [ ] Configure DHCP reservation for Ubuntu.
- [ ] Learn ARP and MAC address resolution.
- [ ] Review Linux DNS configuration.
- [ ] Review Linux firewall configuration.
- [ ] Inspect LVM storage allocation.
- [ ] Create and verify Proxmox VM backups.
- [ ] Install Docker.
- [ ] Begin network security and monitoring exercises.

---

## 20. Final Status

Ubuntu Server VM 100 is running successfully on Proxmox.

Networking is functioning through the `ens18` virtual interface connected to `vmbr0`.

At the time of this exercise, Ubuntu had the DHCP-assigned address `192.168.1.17/24`. Following the November 16 recovery and DHCP reservation, its current address is `192.168.1.22/24`, with the Netgear RAX70 at `192.168.1.1` as its default gateway.

System package upgrades completed successfully, no reboot was required at the time of verification, and Ubuntu Pro security services were confirmed as enabled.

**Outcome:** Successfully configured, verified, updated, and remotely administered the first Ubuntu Server VM in the home lab.

---

**Next Document:** `04-ssh-configuration-and-hardening.md`
