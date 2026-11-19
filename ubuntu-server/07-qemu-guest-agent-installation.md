# 07 - QEMU Guest Agent Installation and Troubleshooting

**Project:** HomeLab-Learning  
**Date:** November 16, 2026  
**Hypervisor:** Proxmox VE 9.2.26  
**Virtual Machine:** VM 100 - ubuntu-server  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Status:** Completed - QEMU Guest Agent installed, enabled, and verified

---

## 1. Objective

Install and configure the QEMU Guest Agent on Ubuntu Server VM 100 to improve communication between the Ubuntu operating system and the Proxmox hypervisor.

The objectives were to:

- Install the QEMU Guest Agent package.
- Enable guest-agent support in Proxmox.
- Verify that the guest-agent service is running.
- Troubleshoot an initially inactive guest-agent configuration.
- Understand the difference between rebooting Ubuntu and fully restarting a virtual machine.
- Verify communication between Proxmox and Ubuntu.
- Prepare for improved backup consistency in future backups.

This exercise followed the successful creation of the first Ubuntu VM backup, documented in `06-proxmox-vm-backup.md`.

---

## 2. Home Lab Environment

| Component | Configuration |
|---|---|
| Physical server | TRIGKEY Mini PC |
| Hypervisor | Proxmox VE 9.2.26 |
| Proxmox node | pve |
| Proxmox IP | 192.168.1.200 |
| Virtual machine | VM 100 |
| VM hostname | ubuntu-server |
| Operating system | Ubuntu Server 26.04.1 LTS |
| Ubuntu IP | 192.168.1.22 |
| Network interface | ens18 |
| SSH alias | ubuntu-lab |
| Guest-agent package | qemu-guest-agent |

The Ubuntu VM is accessible through SSH from a Windows 11 laptop.

---

## 3. What Is QEMU Guest Agent?

The QEMU Guest Agent is a service installed inside a virtual machine that allows the hypervisor to communicate with the guest operating system.

Proxmox controls the virtual hardware, while Ubuntu operates inside that virtual environment.

The guest agent allows Proxmox to request certain information and operations from Ubuntu.

### Capabilities

- Retrieve guest IP address information.
- Coordinate supported guest shutdown operations.
- Report guest operating system information.
- Support filesystem freeze and thaw operations during supported backup processes.
- Improve communication between the hypervisor and guest.

### How It Works

```text
TRIGKEY Mini PC
       |
       v
Proxmox VE Hypervisor
       |
       | QEMU Guest Agent Communication Channel
       |
       v
Ubuntu Server VM 100
       |
       v
qemu-guest-agent Service
```

### Important Concept

The QEMU Guest Agent requires two components:

1. The agent software installed inside Ubuntu.
2. The guest-agent communication channel enabled in Proxmox.

Both components must be configured for the integration to work.

---

## 4. Connect to Ubuntu Using SSH

Opened Windows Terminal with PowerShell.

Executed:

```powershell
ssh ubuntu-lab
```

The SSH client connected using the previously configured ED25519 private key.

After authentication, the Ubuntu terminal displayed:

```text
jansen@ubuntu-server:~$
```

This confirmed that commands would be executed on Ubuntu rather than the Windows laptop.

---

## 5. Update Ubuntu Before Installation

Before installing the QEMU Guest Agent, refreshed Ubuntu's package database.

### Command

```bash
sudo apt update
```

Ubuntu reported that six packages were eligible for upgrades.

Installed the available upgrades:

```bash
sudo apt upgrade
```

The upgrade completed successfully.

### Understanding the Commands

| Component | Purpose |
|---|---|
| sudo | Execute with elevated privileges |
| apt | Ubuntu package manager |
| update | Refresh available package information |
| upgrade | Install eligible package upgrades |

### What I Learned

`apt update` does not install software updates.

It refreshes the package information Ubuntu uses to determine what newer software versions are available.

`apt upgrade` installs eligible upgrades to currently installed packages.

---

## 6. Install QEMU Guest Agent

Executed from the Ubuntu terminal:

```bash
sudo apt install qemu-guest-agent
```

### Command Explanation

**sudo**

Runs the command with elevated privileges.

**apt install**

Installs a package using Ubuntu's package management system.

**qemu-guest-agent**

The software package providing the QEMU Guest Agent service.

### Installation Output

The package was successfully unpacked and configured.

Relevant output:

```text
Unpacking qemu-guest-agent (1:10.2.1+ds-1ubuntu3.3)...
Setting up qemu-guest-agent (1:10.2.1+ds-1ubuntu3.3)...
qemu-guest-agent.service is a disabled or a static unit, not starting it.
```

### Initial Concern

The message indicated that the service was not started automatically during package installation.

This did not mean that the installation failed.

The package was installed, but the Proxmox guest-agent integration was not yet active.

---

## 7. Enable QEMU Guest Agent in Proxmox

Opened the Proxmox web management interface:

```text
https://192.168.1.200:8006
```

Selected:

`VM 100 (ubuntu-server) → Options`

Located:

**QEMU Guest Agent**

Edited the setting and selected:

**Enabled**

Saved the configuration.

### Expected Result

Once fully activated, Proxmox should be able to communicate with the guest agent and retrieve information from Ubuntu.

---

## 8. Initial Verification Failed

After enabling the QEMU Guest Agent option, restarted Ubuntu from the Linux terminal.

### Command

```bash
sudo reboot
```

The SSH session disconnected as expected.

Ubuntu restarted successfully.

Returned to:

`VM 100 → Summary`

### Unexpected Result

Proxmox displayed:

```text
IPs: No Guest Agent configured
```

The guest agent was installed inside Ubuntu, but Proxmox was still not detecting the integration.

### Investigation

Returned to:

`VM 100 → Options`

The QEMU Guest Agent option displayed:

```text
Default (Disabled)
Enabled
```

The new **Enabled** setting appeared in orange.

### What This Meant

Proxmox had accepted the configuration change, but it had not yet been applied to the running virtual machine.

The setting was pending until the virtual machine was fully stopped and started again.

---

## 9. Understanding Pending Configuration Changes

Certain Proxmox configuration changes can be applied while a VM is running.

Other changes require the virtual machine to be fully stopped and restarted.

The QEMU Guest Agent communication channel was one of the settings that required a full VM restart in this situation.

### Guest OS Reboot vs. VM Shutdown

**Ubuntu reboot:**

```bash
sudo reboot
```

This restarts the operating system running inside the virtual machine.

However, the underlying QEMU virtual machine process may remain active.

**Proxmox VM Shutdown and Start:**

```text
Proxmox → VM 100 → Shutdown
Proxmox → VM 100 → Start
```

This fully stops and recreates the virtual machine, allowing pending virtual hardware changes to take effect.

### Lesson Learned

Rebooting a guest operating system is not always equivalent to fully restarting its virtual machine.

---

## 10. Resolve the Configuration Issue

Performed a graceful shutdown through Proxmox.

### Procedure

1. Selected VM 100.
2. Clicked **Shutdown**.
3. Allowed Ubuntu to shut down.
4. Confirmed the VM was stopped.
5. Clicked **Start**.
6. Returned to the VM Summary page.

### Result

After the complete shutdown and startup, Proxmox successfully displayed Ubuntu's IP address information.

This confirmed that the guest-agent communication channel was functioning.

### Root Cause

The QEMU Guest Agent option had been saved but remained pending while the original VM process was active.

A full VM shutdown and startup activated the new configuration.

---

## 11. Verify the Guest Agent Service in Ubuntu

Reconnected to Ubuntu from Windows PowerShell:

```powershell
ssh ubuntu-lab
```

Then executed:

```bash
systemctl status qemu-guest-agent
```

### Actual Result

The service was reported as running without issues.

### Expected Status

```text
Active: active (running)
```

### Command Breakdown

| Component | Meaning |
|---|---|
| systemctl | Systemd service management utility |
| status | Display service information |
| qemu-guest-agent | Guest-agent service |

### Final Verification

Two separate checks confirmed successful operation.

**Proxmox verification:** The VM Summary displayed Ubuntu's IP address information.

**Ubuntu verification:** The QEMU Guest Agent service was active and running.

---

## 12. How QEMU Guest Agent Improves Backups

The first Ubuntu backup was created before the guest agent was installed and activated.

During that backup, the task log displayed:

```text
INFO: skipping guest filesystem freeze - disabled in VM options
```

This meant filesystem freeze operations were not performed during that backup.

### Filesystem Freeze

A filesystem freeze temporarily suspends supported filesystem write operations so a backup or snapshot can capture a more consistent filesystem state.

### Filesystem Thaw

A filesystem thaw resumes the suspended write operations.

### Why This Matters

If a virtual machine is writing data while a backup is being created, the backup may capture data in an intermediate state.

Coordinating filesystem operations can reduce this risk.

### Important Limitation

Enabling QEMU Guest Agent does not automatically guarantee application-consistent backups.

Consistency depends on the guest configuration, filesystem support, workloads, and the operations performed during the backup.

Future backup logs should be examined to verify whether filesystem freeze and thaw operations occur.

---

## 13. Troubleshooting Summary

| Issue | Explanation | Resolution |
|---|---|---|
| Guest-agent service did not start during installation | Package installation did not start the service | Enabled guest-agent support in Proxmox |
| Proxmox displayed No Guest Agent configured | Guest-agent integration was not active | Inspected VM Options |
| Enabled setting appeared orange | Proxmox configuration change was pending | Performed full VM shutdown and startup |
| Ubuntu reboot did not resolve issue | Guest reboot did not recreate virtual hardware | Restarted VM through Proxmox |
| Needed to confirm agent operation | Configuration alone was insufficient | Verified Proxmox IP reporting and Ubuntu service status |

---

## 14. Commands Learned

### Connect to Ubuntu

```powershell
ssh ubuntu-lab
```

Connects to the Ubuntu server from Windows.

### Refresh Package Information

```bash
sudo apt update
```

Refreshes package metadata.

### Upgrade Installed Packages

```bash
sudo apt upgrade
```

Installs eligible package upgrades.

### Install QEMU Guest Agent

```bash
sudo apt install qemu-guest-agent
```

Installs the guest-agent package.

### Reboot Ubuntu

```bash
sudo reboot
```

Restarts the guest operating system.

### Verify Guest-Agent Status

```bash
systemctl status qemu-guest-agent
```

Checks the status of the QEMU Guest Agent service.

---

## 15. Completion Checklist

- [x] Connected to Ubuntu through SSH.
- [x] Refreshed APT package lists.
- [x] Completed six eligible package upgrades.
- [x] Installed QEMU Guest Agent.
- [x] Enabled the QEMU Guest Agent option in Proxmox.
- [x] Rebooted Ubuntu for initial testing.
- [x] Identified the pending Proxmox configuration.
- [x] Performed a full VM shutdown.
- [x] Restarted VM 100 through Proxmox.
- [x] Verified Ubuntu IP information appeared in Proxmox.
- [x] Verified QEMU Guest Agent service was active.
- [ ] Verify filesystem freeze/thaw during a future backup.
- [ ] Test restoring a backup to a separate VM.
- [ ] Establish a backup destination outside the TRIGKEY.

---

## 16. What I Learned

This exercise provided hands-on experience with:

1. Installing Linux software using APT.
2. Understanding guest and hypervisor communication.
3. Configuring Proxmox virtual machine options.
4. Recognizing pending configuration changes.
5. Understanding the difference between guest OS reboots and full VM restarts.
6. Managing Linux services using systemctl.
7. Diagnosing an inactive guest-agent integration.
8. Verifying configuration changes from both Proxmox and Ubuntu.
9. Understanding filesystem consistency during backups.
10. Documenting installation problems and their resolutions.

---

## 17. Final Outcome

Successfully installed, configured, and verified the QEMU Guest Agent for Ubuntu Server VM 100.

The agent initially failed to become active because the Proxmox configuration change was pending.

A full VM shutdown and startup resolved the issue.

Proxmox now displays Ubuntu's network information through the guest-agent integration, and the guest-agent service is running successfully inside Ubuntu.

## 18. QEMU Guest Agent Backup Verification

**Date:** November 19, 2026  
**VM:** 100 - ubuntu-server  
**Result:** Successful

### Objective

Verify that Proxmox uses the QEMU Guest Agent to coordinate filesystem operations during a live virtual machine backup.

### Backup Configuration

| Setting | Value |
|---|---|
| Storage | local |
| Mode | Snapshot |
| Compression | ZSTD |
| Protected | No |
| Virtual disk | 32 GiB |
| Guest agent | Enabled |

### Procedure

1. Confirmed Ubuntu VM 100 was running.
2. Opened `VM 100 → Backup → Backup now`.
3. Selected local storage, Snapshot mode, and ZSTD compression.
4. Started the backup.
5. Reviewed the completed task log.

### Guest-Agent Verification

The Proxmox backup log displayed:

```text
INFO: issuing guest-agent 'fs-freeze' command
INFO: starting backup via QMP command
INFO: issuing guest-agent 'fs-thaw' command
```

These messages confirmed that Proxmox issued filesystem freeze and thaw commands through the QEMU Guest Agent during the backup operation.

### Backup Results

| Property | Result |
|---|---|
| Backup date | November 19, 2026 |
| Backup filename | vzdump-qemu-100-2026_11_19-09_55_35.vma.zst |
| Backup duration | 22 seconds |
| Archive size reported | 2.12 GB |
| Filesystem freeze/thaw | Commands issued |
| Backup task status | TASK OK |

### Comparison With Initial Backup

The first backup, completed November 16, 2026, skipped guest filesystem freezing because the guest-agent integration was not yet configured.

The November 19 backup successfully issued filesystem freeze/thaw commands through the guest agent.

This provides evidence that the guest-agent integration is working during backup operations.

Filesystem coordination improves backup consistency but does not guarantee full application consistency for every workload.

### Lessons Learned

- A successful backup does not automatically confirm that the guest agent was used.
- Proxmox task logs provide evidence of guest-agent operations.
- Filesystem freeze and thaw coordinate writes around the backup snapshot.
- Backup consistency is important when protecting running systems.
- A completed backup must still be tested through restoration to verify recoverability.

### Verification Checklist

- [x] QEMU Guest Agent installed and running.
- [x] Proxmox guest-agent communication confirmed.
- [x] Second snapshot backup completed.
- [x] Filesystem freeze command observed.
- [x] Filesystem thaw command observed.
- [x] Backup completed with TASK OK.
- [ ] Restore backup to a separate test VM.
- [ ] Verify restored operating system and files.
- [ ] Configure independent backup storage.

**Final Result:** QEMU Guest Agent functionality verified during a successful Proxmox backup.

**Final Status: QEMU Guest Agent operational.**

---

**Next Project:** Verify guest-agent integration during a new Proxmox backup, test backup restoration, and continue preparing the Ubuntu environment for Docker and future cybersecurity applications.
