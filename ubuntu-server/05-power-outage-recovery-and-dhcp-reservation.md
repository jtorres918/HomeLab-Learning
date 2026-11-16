# 05 - Power Outage Recovery and DHCP Reservation

**Project:** HomeLab-Learning  
**Date:** November 16, 2026  
**Hypervisor:** Proxmox VE  
**Virtual Machine:** VM 100 - ubuntu-server  
**Router:** NETGEAR Nighthawk RAX70  
**Status:** SSH restored and DHCP reservation configured

---

## 1. Objective

Restore remote access to the Ubuntu Server VM following a power outage and prevent its IPv4 address from changing unexpectedly in the future.

The exercise included:

- Recovering from an unplanned power outage
- Checking the Proxmox host and Ubuntu VM
- Troubleshooting an SSH connection timeout
- Identifying a changed DHCP address
- Reconnecting using the new IP address
- Configuring a DHCP reservation
- Updating the Windows SSH client configuration
- Understanding the importance of reliable server startup

---

## 2. Initial Problem

A power outage caused the physical TRIGKEY server to shut down.

After power was restored, the Proxmox host was powered on manually.

However, attempts to reconnect to Ubuntu using the existing Windows SSH shortcut were unsuccessful.

Command:

```powershell
ssh ubuntu-lab
```

Result:

The connection timed out.

### Initial Possible Causes

- Proxmox was not running.
- Ubuntu VM 100 had not started.
- Ubuntu's network interface was unavailable.
- Ubuntu's IP address had changed.
- The SSH service was unavailable.

Rather than modifying configuration immediately, I investigated the state of the virtualization host and guest.

---

## 3. Verify Proxmox

Accessed the Proxmox web interface:

```text
https://192.168.1.200:8006
```

The Proxmox interface was accessible.

This demonstrated that the physical server had booted and its management interface was reachable.

### Discovery

Ubuntu VM 100 had not automatically started after the host restarted.

Selected VM 100 and manually started it.

### Lesson Learned

A running Proxmox host does not necessarily mean every virtual machine is running.

Virtual machines can have individual startup settings.

---

## 4. SSH Still Timed Out

After manually starting Ubuntu, attempted to reconnect:

```powershell
ssh ubuntu-lab
```

The connection still failed.

The next step was to verify Ubuntu's actual IPv4 address using the Proxmox console.

### Why Use the Proxmox Console?

The Proxmox console provides direct access to the virtual machine through the hypervisor.

It does not depend on the Ubuntu SSH service being reachable.

This makes it an important recovery method.

---

## 5. Inspect Ubuntu's Network Interface

Opened:

`Proxmox → VM 100 → Console`

Logged into Ubuntu.

Executed:

```bash
ip -4 addr show ens18
```

### Actual Output

The network interface reported:

```text
ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
inet 192.168.1.22/24 metric 100 brd 192.168.1.255 scope global dynamic ens18
```

### Command Explanation

| Component | Meaning |
|---|---|
| `ip` | Linux networking utility |
| `-4` | IPv4 information only |
| `addr` | Address information |
| `show` | Display configuration |
| `ens18` | Ubuntu's virtual Ethernet interface |

### Important Discovery

Previous Ubuntu IP:

`192.168.1.17`

New Ubuntu IP:

`192.168.1.22`

The interface was up and had a dynamically assigned address.

### Root Cause of SSH Failure

The Windows SSH shortcut still pointed to `192.168.1.17`.

Ubuntu had received `192.168.1.22` from DHCP.

The SSH client was attempting to connect to the outdated address.

---

## 6. Reconnect Using the New IP Address

From Windows PowerShell:

```powershell
ssh -i "$env:USERPROFILE\.ssh\homelab_ubuntu" jansen@192.168.1.22
```

### Command Explanation

**`ssh`**

Starts the OpenSSH client.

**`-i`**

Specifies which private key to use.

**`$env:USERPROFILE`**

References the Windows user's profile directory.

**`jansen@192.168.1.22`**

Specifies the Ubuntu user and updated destination IP address.

### Result

Successfully connected to Ubuntu using the existing ED25519 SSH private key.

This confirmed that the SSH server and key-based authentication were functioning.

No changes to Ubuntu's SSH security configuration were necessary.

---

## 7. Understand Why the IP Changed

Ubuntu was configured to obtain its IPv4 address automatically through DHCP.

DHCP stands for Dynamic Host Configuration Protocol.

The NETGEAR RAX70 provides DHCP services to devices on the home network.

Without an address reservation, a device can receive a different address when requesting or renewing a DHCP lease.

### Why This Matters

Remote management tools need an accurate destination address.

If a server's IP changes, previously configured SSH clients, application connections, monitoring tools, and automation may fail.

A consistent server address simplifies administration.

---

## 8. Configure a DHCP Reservation

Accessed the NETGEAR RAX70 management interface:

```text
http://192.168.1.1
```

Navigated to:

`Advanced → Setup → LAN Setup → Address Reservation`

Created a reservation for the Ubuntu Server VM.

### Reservation Configuration

| Setting | Value |
|---|---|
| Device name | ubuntu-server |
| Reserved IP | 192.168.1.22 |
| MAC address | BC:24:11:23:80:D9 |
| Interface | ens18 |
| DHCP server | NETGEAR RAX70 |

The reservation appeared in the router's Address Reservation table.

### What Is a MAC Address?

MAC stands for Media Access Control.

A MAC address identifies a network interface at the data-link layer.

The router uses the VM's MAC address to identify the device associated with the reservation.

### Why Reserve the MAC Address?

The DHCP server can recognize the same network interface and assign it the specified address.

The VM continues using DHCP, while the router maintains the address assignment.

### DHCP Reservation vs. Static IP

**Static IP**

The IP address is manually configured on the device itself.

**DHCP reservation**

The IP address is configured on the DHCP server and associated with a device's MAC address.

Ubuntu can continue using DHCP without a manually assigned static address.

### Result

The NETGEAR router displayed the Ubuntu reservation:

```text
Device: ubuntu-server
IP: 192.168.1.22
MAC: BC:24:11:23:80:D9
```

The reservation was configured. A subsequent DHCP renewal or reboot test would provide additional confirmation that the router continues assigning the reserved address.

---

## 9. Update the Windows SSH Shortcut

The Windows SSH configuration still referenced Ubuntu's previous IP address.

Returned from the Ubuntu SSH session to Windows PowerShell using:

```bash
exit
```

This closed the SSH session without shutting down Ubuntu.

### Open the SSH Configuration

From Windows PowerShell:

```powershell
notepad "$env:USERPROFILE\.ssh\config"
```

### Previous Configuration

```sshconfig
Host ubuntu-lab
    HostName 192.168.1.17
    User jansen
    IdentityFile ~/.ssh/homelab_ubuntu
    IdentitiesOnly yes
```

Changed only the HostName address.

### Updated Configuration

```sshconfig
Host ubuntu-lab
    HostName 192.168.1.22
    User jansen
    IdentityFile ~/.ssh/homelab_ubuntu
    IdentitiesOnly yes
```

Saved the file without adding a `.txt` extension.

### Test the Updated Shortcut

From Windows PowerShell:

```powershell
ssh ubuntu-lab
```

### Result

Successfully reconnected to Ubuntu using the updated SSH shortcut.

The existing ED25519 authentication configuration continued working.

---

## 10. Understanding the Troubleshooting Process

This incident involved two separate issues.

### Issue 1 — Ubuntu VM Was Not Running

**Symptom:** SSH connection timed out.

**Discovery:** Proxmox was running, but Ubuntu VM 100 was stopped.

**Resolution:** Manually started Ubuntu through Proxmox.

### Issue 2 — Ubuntu's DHCP Address Had Changed

**Symptom:** SSH connection still failed after the VM was started.

**Discovery:** Ubuntu's current address was `192.168.1.22`, but Windows SSH configuration referenced `192.168.1.17`.

**Resolution:** Connected using the current IP, configured a router DHCP reservation, and updated the Windows SSH shortcut.

### Important Lesson

A network timeout is a symptom, not a complete diagnosis.

Checking the system's running state and network configuration revealed the causes without requiring reinstallation or disabling SSH security controls.

---

## 11. Additional Network Observation

During router administration, an address of `192.168.1.200` was displayed for a device named `DESKTOP-AAPKA7U` with MAC address `B0:41:6F:12:D3:A4`.

This IP is also the configured Proxmox management address.

The observation requires further investigation to determine whether the router's device information reflects the Proxmox host, a previously identified computer, or a potential addressing conflict.

No duplicate IP conflict has been confirmed.

The existing Proxmox network configuration should not be changed until the device's identity is verified.

---

## 12. Future Startup Configuration

The power outage also highlighted the need to configure the VM's startup behavior.

Planned Proxmox configuration:

`VM 100 → Options → Start at boot`

This setting allows Proxmox to automatically start the VM during host startup when enabled.

At the time of this documentation, the setting still requires confirmation.

Automatic startup is not a substitute for reliable power protection or tested recovery procedures.

---

## 13. Commands Learned

| Command | Purpose |
|---|---|
| `ssh ubuntu-lab` | Connect using the saved Windows SSH alias |
| `ip -4 addr show ens18` | Inspect Ubuntu's IPv4 address |
| `ssh -i ...` | Select a private key explicitly |
| `exit` | Close the current SSH session |
| `notepad "$env:USERPROFILE\.ssh\config"` | Edit the Windows SSH client configuration |

---

## 14. Completion Checklist

- [x] Proxmox host powered on after outage.
- [x] Proxmox management interface accessed.
- [x] Ubuntu VM manually started.
- [x] SSH timeout investigated.
- [x] Ubuntu network interface inspected.
- [x] New DHCP address identified.
- [x] SSH connection restored using new IP.
- [x] Ubuntu MAC address identified.
- [x] DHCP reservation added on NETGEAR RAX70.
- [x] Windows SSH configuration updated.
- [x] `ssh ubuntu-lab` connection verified.
- [ ] Confirm persistent reservation after a future DHCP renewal.
- [ ] Confirm or enable Proxmox VM Start at boot.
- [ ] Investigate `192.168.1.200` device identity.
- [ ] Consider a UPS for power outage protection.

---

## 15. What I Learned

1. Proxmox and its virtual machines have separate startup states.
2. The Proxmox console can be used when SSH is unavailable.
3. DHCP addresses can change after network interruptions.
4. Servers benefit from consistent network addressing.
5. A DHCP reservation associates an IP address with a MAC address.
6. A DHCP reservation is different from configuring a static IP on the server.
7. SSH client configurations must reference the correct destination address.
8. A successful SSH connection can verify the server is reachable and authentication is working.
9. Troubleshooting should isolate individual causes instead of assuming a single failure.
10. Power outages demonstrate the importance of recovery planning.

---

## 16. Final Outcome

Following a power outage, Proxmox was brought online and Ubuntu VM 100 was manually restarted.

The Ubuntu VM received a different DHCP address, preventing the existing Windows SSH shortcut from connecting.

The updated address was identified using Linux networking commands.

Secure SSH access was restored, a DHCP reservation was configured on the NETGEAR RAX70, and the Windows SSH shortcut was updated.

**Current Ubuntu IP:** `192.168.1.22`  
**SSH shortcut:** `ssh ubuntu-lab`  
**Status:** Operational

**Next Task:** Investigate the identity of the device displayed at `192.168.1.200` and verify that the Proxmox management IP does not conflict with another device.
