# 06 - Proxmox Ubuntu VM Backup

**Project:** HomeLab-Learning  
**Lab Date:** November 16, 2026  
**Platform:** Proxmox VE 9.2.26  
**Node:** `pve`  
**Virtual Machine:** VM 100 - `ubuntu-server`  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Status:** Backup created successfully; archive presence verified; restore not yet tested

---

## 1. Objective

Create the first recovery-point backup of Ubuntu Server VM 100 before installing Docker or making other significant home lab changes.

The goals were to:

- Verify that Proxmox had a storage destination capable of holding backups.
- Check available storage capacity before starting.
- Back up a running virtual machine using **Snapshot** mode.
- Use **ZSTD** compression to reduce archive size.
- Mark this initial backup as **Protected** against routine deletion or pruning.
- Confirm that the backup task finished successfully.
- Confirm that the resulting archive appeared in Proxmox's backup list.
- Understand what the backup protects against and what it does **not** protect against.

---

## 2. Home Lab Environment

| Component | Configuration |
|---|---|
| Physical host | TRIGKEY Mini PC |
| Hypervisor | Proxmox VE 9.2.26 |
| Proxmox node | `pve` |
| Proxmox management IP | `192.168.1.200` |
| Ubuntu VM ID | `100` |
| Ubuntu VM name | `ubuntu-server` |
| Ubuntu VM IPv4 | `192.168.1.22` (DHCP reservation) |
| Ubuntu VM virtual disk | 32 GiB |
| Virtual disk storage | `local-lvm` |
| Backup destination | `local` |

Proxmox was accessed at:

```text
https://192.168.1.200:8006
```

The Ubuntu VM was running during the backup.

---

## 3. Verify Backup Storage Support

In the Proxmox web interface, navigated to:

`Datacenter → Storage`

The configuration displayed:

| Storage | Type | Relevant Content |
|---|---|---|
| `local` | Directory | Backup, ISO images, and other file content |
| `local-lvm` | LVM-Thin | VM disk images and container storage |

**Result:** `local` was enabled and supported **Backup** content.

### What I Learned

The place where a VM's virtual disk is stored does not have to be the same storage destination used for its backup archive.

In this lab:

- The VM's **32 GiB virtual disk** is on `local-lvm`.
- The **compressed backup archive** is stored on `local`.

Both are areas of storage on the same physical TRIGKEY SSD in the current setup.

---

## 4. Verify Available Storage Space

Navigated to:

`local (pve) → Summary`

Proxmox displayed:

| Measurement | Value |
|---|---|
| Total local capacity | 100.86 GB |
| Used capacity | 9.33 GB |
| Used percentage | 9.25% |
| Available capacity (calculated) | Approximately 91.53 GB |
| Storage status | Enabled and active |

The available capacity was sufficient to attempt a backup of the 32 GiB virtual disk.

### Important Storage Concept

The virtual disk's **32 GiB capacity** is not the same as the final compressed backup size.

Backup size depends on the actual data contained in the virtual disk, which blocks are allocated or used, and how well the data compresses.

A compressed backup may be substantially smaller than the virtual disk's advertised capacity.

---

## 5. Open the VM Backup Dialog

Navigated to:

`VM 100 (ubuntu-server) → Backup → Backup now`

Before starting, reviewed the available options.

### Settings Selected

| Setting | Selected Value |
|---|---|
| Storage | `local` |
| Mode | `Snapshot` |
| Compression | `ZSTD (fast and good)` |
| Notification | Use global settings |
| Protected | Enabled |
| Notes | Ubuntu Server VM 100 - Initial backup before Docker and additional lab configuration - 2026-11-16 |

After reviewing the settings, clicked **Backup**.

---

## 6. Understanding Backup Mode: Snapshot

**Snapshot mode** allows Proxmox to back up a running VM without requiring a normal full shutdown for the entire backup operation.

This was useful because Ubuntu could remain available while the archive was created.

### Important Distinction: Backup vs. VM Snapshot

The **Snapshot backup mode** selected in the backup dialog creates a backup archive.

It is **not** the same as creating a regular VM snapshot under the VM's separate **Snapshots** menu.

A Proxmox backup archive can be used for a restore operation. A VM snapshot is a different recovery mechanism tied to the VM's storage and snapshot capabilities.

### Consistency Limitation

The backup log stated:

```text
INFO: skipping guest filesystem freeze - disabled in VM options
```

This means guest filesystem freeze was **not performed** for this backup.

The result is best treated as a **crash-consistent recovery point**, not a guaranteed application-consistent backup.

For future backups, the QEMU guest agent may be installed and enabled to support guest filesystem freeze/thaw operations when correctly configured.

A backup completing successfully does **not** prove that every application inside the VM could be restored without issue.

---

## 7. Understanding ZSTD Compression

The selected compression setting was:

```text
ZSTD (fast and good)
```

ZSTD stands for **Zstandard**.

It is a compression algorithm designed to provide a useful balance of compression ratio and speed.

### Why Use Compression?

Compression can:

- Reduce the amount of storage needed for backup archives.
- Reduce transfer size if the archive is later copied elsewhere.
- Make it practical to retain more backups within a given storage budget.

The amount of compression depends on the type of data being backed up.

---

## 8. Understanding Protected Backups

Enabled the **Protected** option for the backup.

The task log later confirmed:

```text
INFO: marking backup as protected
```

The backup list also showed its protection indicator.

### What Protection Means

A protected backup is safeguarded against ordinary removal and automated pruning operations until its protection is removed, subject to the storage and administrative controls in use.

**Protected does not mean:**

- The archive has been encrypted.
- The archive exists on a separate physical drive.
- The archive is protected against SSD failure.
- The archive has passed a restore test.

The initial backup was protected because it is a useful baseline before further home lab changes.

---

## 9. Backup Task Log and Actual Results

The Proxmox task viewer was opened automatically after the backup began.

The output included the following observations:

```text
INFO: include disk 'scsi0' 'local-lvm:vm-100-disk-0' 32G
INFO: backup mode: snapshot
INFO: creating vzdump archive '/var/lib/vz/dump/vzdump-qemu-100-2026_11_16-10_10_04.vma.zst'
INFO: skipping guest filesystem freeze - disabled in VM options
INFO: starting backup via QMP command
INFO: resuming VM again
INFO: 100% (32.0 GiB of 32.0 GiB) in 12s
INFO: backup is sparse: 28.45 GiB (88%) total zero data
INFO: transferred 32.00 GiB in 12 seconds (2.7 GiB/s)
INFO: archive file size: 1.80GB
INFO: adding notes to backup
INFO: marking backup as protected
INFO: Finished Backup of VM 100 (00:00:12)
INFO: Backup finished at 2026-11-16 10:10:16
INFO: Backup job finished successfully
TASK OK
```

The task also reported notification activity through the globally configured `mail-to-root` target. This log entry does not, by itself, prove that a recipient received an email.

### Results Summary

| Property | Observed Result |
|---|---|
| VM ID | 100 |
| Disk included | `scsi0` on `local-lvm` |
| Virtual disk capacity | 32 GiB |
| Backup mode | Snapshot |
| Compression | ZSTD |
| Duration | 12 seconds |
| Sparse/zero data reported | 28.45 GiB (88%) |
| Task-log archive size | Approximately 1.80 GiB |
| Final task status | `TASK OK` |
| Protected | Yes |

### Why Did It Finish So Quickly?

The output reported a substantial amount of zero data in the virtual disk, and the resulting compressed archive was much smaller than the 32 GiB disk capacity.

This is expected for a relatively new Ubuntu Server installation with limited application data.

The speed and size of future backups may change as additional services and data are added.

---

## 10. Verify the Backup Archive Exists

After the task completed, closed the task viewer.

Remained in:

`VM 100 → Backup`

The backup list displayed one new entry.

### Verified Backup File

```text
vzdump-qemu-100-2026_11_16-10_10_04.vma.zst
```

### Verified Details

| Field | Value |
|---|---|
| Storage | `local` |
| VM | 100 |
| Filename | `vzdump-qemu-100-2026_11_16-10_10_04.vma.zst` |
| Date displayed | 2026-11-16 10:10:04 |
| File size displayed in backup list | 1.93 GB |
| Format | `vma.zst` |
| Protection indicator | Present |
| Notes | Initial Ubuntu Server backup description |

The list's **1.93 GB** and task log's approximately **1.80 GiB** reflect different ways of displaying file sizes (decimal GB versus binary GiB).

### Backup File Location

The task log identified the archive path:

```text
/var/lib/vz/dump/vzdump-qemu-100-2026_11_16-10_10_04.vma.zst
```

### Understanding the Filename

- `vzdump`: Proxmox backup tooling.
- `qemu`: Backup of a QEMU/KVM virtual machine.
- `100`: Virtual machine ID.
- `2026_11_16-10_10_04`: Timestamp included in the filename.
- `.vma`: Proxmox virtual machine archive format.
- `.zst`: ZSTD-compressed archive.

---

## 11. What the Backup Protects Against

This backup provides a recovery point for certain VM-level failures, such as:

- Accidental removal of important files inside Ubuntu.
- A broken configuration following future experiments.
- An unsuccessful application installation.
- VM corruption where the backup itself remains usable.
- The need to restore Ubuntu to a previous state.

### Important Limitations

**The backup is on the same physical SSD as Proxmox and the VM.**

It is **not** an independent backup against:

- Physical SSD failure.
- Complete loss or damage of the TRIGKEY.
- Theft of the server.
- Storage-wide corruption affecting both the VM and backup.
- Certain security incidents affecting all files on the same host.

A stronger backup plan will include a separate storage device or system and verified off-host copies.

---

## 12. Restore Testing and Future Recovery

The backup task completed successfully and the archive appeared in storage.

However, **a restore test has not yet been performed**.

A successful backup job and visible archive are good initial checks, but a restore test is the stronger evidence that the VM can be recovered.

### Future Restore-Test Plan

When appropriate, restore the backup as a **separate test VM** rather than overwriting the working Ubuntu VM 100.

Before starting a restored copy:

- Choose a different, unused VM ID.
- Plan for network isolation or avoid concurrent identical network identities.
- Confirm the restored VM does not conflict with the original Ubuntu server's hostname, MAC, IP, or SSH identity on the home LAN.
- Verify the guest boots, files are available, and expected services function.

**No restore was attempted during this exercise.**

---

## 13. Commands and Concepts Learned

This backup was performed using the **Proxmox web interface**, not through shell commands.

Key terms:

| Term | Meaning |
|---|---|
| `vzdump` | Proxmox VM/container backup utility |
| Backup archive | Stored copy of VM data and configuration for recovery |
| Snapshot backup mode | Backup of a running VM |
| ZSTD | Zstandard compression algorithm |
| Sparse/zero data | Disk regions that contain zeroes and can be efficiently represented or compressed |
| QMP | QEMU Machine Protocol used to manage QEMU virtual machines |
| QEMU guest agent | Software in the guest that can assist host/guest management and consistency operations |
| Protected backup | Archive protected from ordinary deletion and pruning |
| Restore test | Verification that a backup can successfully recreate a functioning system |

---

## 14. Completion Checklist

- [x] Confirmed `local` supports backups.
- [x] Verified sufficient free local storage.
- [x] Opened VM 100's Backup panel.
- [x] Selected `local` storage.
- [x] Selected Snapshot mode.
- [x] Selected ZSTD compression.
- [x] Enabled backup protection.
- [x] Added a descriptive backup note.
- [x] Started backup of running Ubuntu VM.
- [x] Confirmed task ended with `TASK OK`.
- [x] Verified backup archive appeared in VM 100's Backup list.
- [x] Verified backup protection indicator.
- [ ] Install and configure QEMU guest agent.
- [ ] Test restoration into a separate, isolated VM.
- [ ] Create an off-host backup copy.
- [ ] Establish an appropriate backup schedule and retention policy.

---

## 15. What I Learned

1. Proxmox distinguishes VM disk storage from backup archive storage.
2. Backups should be planned around available capacity.
3. Snapshot-mode backup can be used while a VM is running.
4. Snapshot backup mode is not the same as a regular VM snapshot.
5. ZSTD compression can substantially reduce archive size.
6. Empty or zero-filled disk areas help explain why a compressed archive can be much smaller than the virtual disk.
7. Protected backups reduce the risk of accidental deletion or pruning.
8. The Proxmox task log provides evidence of the backup operation and its results.
9. A successful backup job is not the same as a verified restore.
10. Backups stored on the same physical drive remain vulnerable to that drive failing.

---

## 16. Final Outcome

On November 16, 2026, successfully created the first compressed, protected backup of Ubuntu Server VM 100.

**Backup archive:**

```text
vzdump-qemu-100-2026_11_16-10_10_04.vma.zst
```

**Proxmox backup task result:** `TASK OK`  
**Displayed archive size:** 1.93 GB  
**Backup mode:** Snapshot  
**Compression:** ZSTD  
**Backup protection:** Enabled  
**Restore verification:** Not yet completed

**Final Status: Initial Ubuntu VM backup successfully created and confirmed present in Proxmox storage.**

---

**Next Project:** Install and configure QEMU guest agent; then improve backup consistency and plan a separate backup destination.
