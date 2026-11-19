# 08 - Proxmox Backup Restoration Test

**Date:** November 19, 2026  
**Environment:** HomeLab-Learning  
**Hypervisor:** Proxmox VE 9.2.26  
**Original VM:** 100 (ubuntu-server)  
**Restored VM:** 101 (ubuntu-restore-test)  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Result:** Successful

---

## 1. Objective

The objective of this exercise was to verify that a Proxmox virtual machine backup could successfully restore a working Ubuntu Server installation.

A backup completing successfully does not necessarily guarantee that the operating system can be restored.

This exercise tested our ability to:

- Restore a virtual machine from an existing backup.
- Preserve the original production VM.
- Isolate the restored VM from the network.
- Boot the restored operating system.
- Verify user access and system configuration.
- Confirm SSH security settings survived restoration.
- Safely shut down the restored VM.

## 2. Existing Home Lab Environment

### Physical Hardware

- Trigkey Mini PC
- Proxmox VE 9.2.26 installed directly on physical hardware
- NETGEAR Nighthawk RAX70 router
- Home LAN: `192.168.1.0/24`

### Original Ubuntu Server

| Setting | Value |
|---|---|
| VM ID | 100 |
| VM Name | ubuntu-server |
| Operating System | Ubuntu Server 26.04.1 LTS |
| Hostname | ubuntu-server |
| IP Address | 192.168.1.22 |
| CPU | 2 cores |
| RAM | 2048 MiB |
| Virtual Disk | 32 GiB |
| Network Bridge | vmbr0 |
| SSH | Public-key authentication enabled |

The original Ubuntu server remained running throughout the restoration exercise.

## 3. Backup Used for Restoration

The backup was created November 19, 2026, after successfully enabling QEMU Guest Agent integration.

**Backup filename:**

`vzdump-qemu-100-2026_11_19-09_55_35.vma.zst`

**Backup location:**

`local (pve) → Backups`

**Backup configuration:**

| Setting | Value |
|---|---|
| Backup Mode | Snapshot |
| Compression | ZSTD |
| Storage | local |
| Guest Agent | Enabled |
| Filesystem Freeze | Observed in backup log |
| Filesystem Thaw | Observed in backup log |
| Backup Result | TASK OK |

The backup task log confirmed that Proxmox issued filesystem freeze and thaw commands through QEMU Guest Agent.

## 4. Opening the Restore Interface

In the Proxmox web interface:

1. Expanded the `pve` node.
2. Selected `local (pve)`.
3. Opened **Backups**.
4. Selected the November 19 backup archive.
5. Clicked **Restore**.

### Important Discovery: Overwrite Restore

An initial restore window displayed:

`Overwrite Restore: VM 100 (ubuntu-server)`

This would have targeted the original Ubuntu VM for replacement.

**No restoration was initiated from this window.**

The window was closed, and the backup was opened from the `local (pve) → Backups` view.

This provided a Restore window allowing selection of a new VM ID.

**Lesson learned:** Always verify the target VM ID before starting a restoration.

## 5. Restoring to a Separate VM

The following configuration was selected:

| Setting | Value |
|---|---|
| VM ID | 101 |
| Name | ubuntu-restore-test |
| Storage | From backup configuration |
| Unique | Enabled |
| Start After Restore | Disabled |
| Add to HA | Disabled |
| Memory | 2048 MiB |
| CPU Cores | 2 |
| CPU Sockets | 1 |

The **Unique** option was selected so Proxmox would generate a different MAC address for the restored virtual machine.

The **Start after restore** option remained disabled so network isolation could be configured before the first boot.

Clicked **Restore**.

**Result:** `TASK OK`

VM 101 was successfully created from the backup.

## 6. Network Isolation

The restored Ubuntu installation contained network configuration copied from VM 100.

Because both virtual machines originated from the same installation, starting them simultaneously with active network connections could create conflicts.

To prevent this, the restored VM's network interface was disconnected.

### Procedure

1. Selected VM 101.
2. Opened **Hardware**.
3. Selected **Network Device (net0)**.
4. Clicked **Edit**.
5. Expanded **Advanced**.
6. Enabled the **Disconnect** option.
7. Saved the configuration.

### Verification

The network device displayed:

```text
virtio=BC:24:11:5E:E7:F4,bridge=vmbr0,firewall=1,link_down=1
```

The important setting was:

`link_down=1`

This confirms that Proxmox configured the virtual network link as disconnected.

### MAC Address Comparison

| Virtual Machine | MAC Address |
|---|---|
| VM 100 — Original | BC:24:11:23:80:D9 |
| VM 101 — Restored | BC:24:11:5E:E7:F4 |

The restored VM received a different virtual MAC address.

**Lesson learned:** A unique MAC address does not automatically eliminate every possible identity or IP conflict. Isolating the restored VM provides additional protection.

## 7. Booting the Restored VM

After confirming network isolation:

1. Selected VM 101.
2. Clicked **Start**.
3. Opened **Console**.
4. Observed the Ubuntu boot process.
5. Logged in using the existing local Ubuntu account.

### Result

The restored VM successfully booted Ubuntu Server.

The login screen displayed:

```text
Ubuntu 26.04.1 LTS ubuntu-server tty1
```

Successfully logged in with the `jansen` account.

This confirmed that the restored virtual disk contained a bootable operating system and usable local account.

## 8. Verifying System Information

Executed:

```bash
hostnamectl
```

### Command Explanation

`hostnamectl` displays information about the Linux system, including its hostname, operating system, kernel, architecture, and virtualization environment.

### Verified Information

| Property | Result |
|---|---|
| Static Hostname | ubuntu-server |
| Operating System | Ubuntu 26.04.1 LTS |
| Kernel | Linux 7.0.0-43-generic |
| Architecture | x86-64 |
| Virtualization | KVM |
| Hardware Vendor | QEMU |

### Proxmox Name vs. Linux Hostname

Proxmox displayed:

`ubuntu-restore-test`

However, inside Ubuntu, the hostname remained:

`ubuntu-server`

This is expected because the restore copied the original operating system's configuration.

**Lesson learned:** The VM name in Proxmox is separate from the hostname configured inside the guest operating system.

## 9. Checking the SSH Service

Executed:

```bash
systemctl status ssh
```

### Observed Output

```text
Loaded: loaded
Active: inactive (dead)
TriggeredBy: ssh.socket
```

Initially, the inactive SSH service appeared concerning.

However, Ubuntu was using **systemd socket activation**.

With socket activation, the system can listen for incoming SSH connections and start the SSH service when needed.

### Lesson Learned

An inactive service does not always indicate that the associated network service is unavailable.

The corresponding systemd socket should also be inspected.

## 10. Verifying SSH Socket Activation

Executed:

```bash
systemctl status ssh.socket
```

### Observed Output

```text
Active: active (listening)
Listen: 0.0.0.0:22
        [::]:22
Triggers: ssh.service
```

### Explanation

| Output | Meaning |
|---|---|
| active (listening) | SSH socket is accepting connection requests |
| 0.0.0.0:22 | Listening on all IPv4 interfaces |
| [::]:22 | Listening on all IPv6 interfaces |
| Triggers: ssh.service | SSH service can be started when required |

Because the restored VM's network interface was disconnected, remote SSH access from Windows was intentionally not tested.

**Result:** SSH socket activation successfully verified.

## 11. Verifying SSH Authentication Security

The original Ubuntu server had been configured to use SSH public-key authentication while disabling SSH password authentication.

To verify these settings survived restoration, executed:

```bash
sudo sshd -T | grep authentication
```

### Command Breakdown

- `sudo`: Execute with administrative privileges.
- `sshd`: OpenSSH server daemon.
- `-T`: Display effective SSH server configuration.
- `|`: Pipe output to another command.
- `grep authentication`: Display lines containing the word `authentication`.

### Observed Results

```text
hostbasedauthentication no
pubkeyauthentication yes
kerberosauthentication no
gssapiauthentication no
passwordauthentication no
kbdinteractiveauthentication no
authenticationmethods any
```

### Security Verification

| Setting | Expected | Actual | Result |
|---|---|---|---|
| Password Authentication | no | no | Passed |
| Public-Key Authentication | yes | yes | Passed |
| Keyboard-Interactive Authentication | no | no | Passed |

The restored Ubuntu VM retained the expected SSH authentication security configuration.

**Important:** Disabling SSH password authentication does not disable the local Ubuntu password used for console login and `sudo`.

## 12. Proxmox noVNC Console Experience

During restoration testing, commands were entered through the Proxmox **noVNC console** rather than Windows PowerShell.

This was necessary because the restored VM was disconnected from the network.

### Challenge Encountered

The usual terminal shortcut:

`Ctrl + Shift + V`

did not paste commands into the noVNC console.

The toolbar displayed an extra-keys menu rather than the clipboard interface expected from a normal terminal.

### Workaround

Used a shorter command that could be entered manually:

```bash
sudo sshd -T | grep authentication
```

This provided the authentication settings needed for verification without requiring a long regular expression.

**Lesson learned:** Console input behavior varies between terminal applications, browser-based consoles, and remote access tools.

## 13. Shutting Down the Restored VM

After completing verification, the test VM was shut down.

Executed:

```bash
sudo shutdown -h now
```

### Command Breakdown

| Component | Meaning |
|---|---|
| sudo | Execute with administrator privileges |
| shutdown | Initiate system shutdown |
| -h | Halt the system; normally results in power-off |
| now | Perform shutdown immediately |

### Related Commands

```bash
sudo shutdown -r now
```

Restart immediately.

```bash
sudo shutdown -h +10
```

Schedule shutdown in 10 minutes.

```bash
sudo shutdown -c
```

Cancel a pending scheduled shutdown.

```bash
sudo poweroff
```

Power off the system.

VM 101 was shut down after the recovery tests.

The restored test VM was retained in Proxmox with its network disconnected.

## 14. Recovery Verification Checklist

- [x] Original backup located.
- [x] Original VM 100 preserved.
- [x] Backup restored to new VM 101.
- [x] Restore task completed successfully.
- [x] Unique virtual MAC address assigned.
- [x] Restored VM network disconnected.
- [x] Ubuntu booted successfully.
- [x] Local account login successful.
- [x] Hostname verified.
- [x] Operating system and kernel verified.
- [x] SSH socket activation verified.
- [x] SSH authentication hardening verified.
- [x] Test VM shut down safely.
- [ ] Additional application and file-level recovery testing.
- [ ] Independent or external backup storage configured.

## 15. Important Lessons Learned

### A Backup Is Not the Same as a Verified Recovery

A backup task reporting `TASK OK` confirms that Proxmox completed the backup operation.

Restoring that archive and booting the VM provides stronger evidence that recovery is possible.

### Always Verify the Restore Target

Choosing the wrong restore option could overwrite an existing virtual machine.

Restoring to a separate VM ID allows testing without replacing the original VM.

### Network Isolation Matters

Restoring a VM creates a copy of its operating system and configuration.

Network isolation prevents potential conflicts involving duplicate IP addresses, hostnames, or other network identities.

### Virtualization Separates Systems

VM 100 and VM 101 operate as separate virtual machines.

The restored VM could boot and run without replacing or interrupting the original VM.

### Recovery Testing Has Multiple Levels

This exercise verified the restored operating system, local login, and SSH configuration.

It did not test every file, application, or service.

Comprehensive recovery planning should include application-specific checks and backup copies stored independently of the physical host.

## 16. Final Result

**Backup Restoration Test: PASSED**

On November 19, 2026, a Proxmox backup of Ubuntu VM 100 was successfully restored into a separate VM 101.

The restored system:

- Booted successfully.
- Allowed local user login.
- Preserved its Linux hostname and operating system configuration.
- Retained SSH socket activation.
- Retained SSH authentication hardening.
- Remained isolated from the home network during testing.
- Shut down safely after verification.

The original Ubuntu VM 100 was not overwritten.

This demonstrates that the November 19 backup can restore a bootable Ubuntu Server environment with the essential configurations tested.

---

## 17. Next Project

1. Continue preparing the original Ubuntu Server VM 100 for Docker.
2. Install Docker Engine and learn container fundamentals.
3. Deploy a first containerized application.
4. Explore backup storage independent of the Trigkey's internal SSD.
5. Continue documenting commands, troubleshooting, and lessons learned in GitHub.

**Final Status: Proxmox VM backup restoration successfully tested.**
