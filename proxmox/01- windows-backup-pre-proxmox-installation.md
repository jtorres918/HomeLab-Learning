# 01 - Windows backup pre Proxmox Installation

## Goal

Backup Windows in preparation to intall Proxmox VE directly on the TRIGKEY and prepare it as the primary hypervisor for the home lab.

## Hardware

- TRIGKEY mini PC
- AMD Ryzen 9 6900HX
- 16 GB RAM
- 500 GB NVMe SSD
- 2.5 Gb Ethernet
- Proxmox installation USB

## What I am learning

- Bare-metal hypervisor installation
- BIOS/UEFI boot configuration
- Disk installation and partitioning
- Linux-based server installation
- Network configuration
- Proxmox management interface

## Starting Environment

The TRIGKEY originally came with Windows installed.

Before installing Proxmox, I backed up:

- Windows activation information
- BitLocker recovery information
- Windows drivers
- Any important personal files

## Windows Backup Before Proxmox Installation

Before wiping Windows from the TRIGKEY, I created a backup of the important Windows information and files I might need later.

The backup USB was organized like this:

```text
TRIGKEY BACKUP
├── DRIVERS
├── INSTALLED PROGRAMS
├── PERSONAL FILES
├── WINDOWS ACTIVATION
├── Network-Info.txt
└── System-Info.txt
```

### Windows Activation

I confirmed:

- Windows 11 Pro was installed.
- Windows was activated with a digital license.
- The digital license was linked to my Microsoft account.
- BitLocker was turned off.

I saved a screenshot of the Windows activation page to the backup USB.

### OEM Product Key Check

I checked for an OEM Windows product key with PowerShell:

```powershell
(Get-CimInstance -ClassName SoftwareLicensingService).OA3xOriginalProductKey
```

### Driver Backup

I exported the installed third-party Windows drivers using:

```cmd
dism /online /export-driver /destination:"D:\TRIGKEY BACKUP\DRIVERS"
```

#### Issue Encountered: Spaces in File Paths

The command initially failed because the folder name contained a space:

```text
TRIGKEY BACKUP
```

This incorrect command did not work:

```cmd
dism /online /export-driver /destination:D:\TRIGKEY BACKUP\DRIVERS
```

Command Prompt interpreted the space between `TRIGKEY` and `BACKUP` as a separator.

The fix was to place quotation marks around the full path:

```cmd
dism /online /export-driver /destination:"D:\TRIGKEY BACKUP\DRIVERS"
```

### Installed Programs Backup

I saved a list of installed programs with:

```cmd
winget list > "D:\TRIGKEY BACKUP\INSTALLED PROGRAMS\Installed-Programs.txt"
```

### System Information Backup

I saved Windows system information with:

```cmd
systeminfo > "D:\TRIGKEY BACKUP\System-Info.txt"
```

### Network Configuration Backup

I saved the current Windows network configuration with:

```cmd
ipconfig /all > "D:\TRIGKEY BACKUP\Network-Info.txt"
```

### Personal Files

I checked the following folders for anything important:

```text
Desktop
Documents
Downloads
Pictures
Videos
Music
```

Any important files were copied to:

```text
D:\TRIGKEY BACKUP\PERSONAL FILES
```

### Backup Verification

Before wiping Windows, I verified that:

- The driver export completed successfully.
- The `DRIVERS` folder contained exported drivers.
- `Installed-Programs.txt` opened correctly.
- `System-Info.txt` opened correctly.
- `Network-Info.txt` opened correctly.
- The Windows activation screenshot was saved.
- BitLocker was disabled.
- Any important personal files were backed up.

### What I Learned

- How to export Windows drivers with DISM
- How to inventory installed software with `winget`
- How to redirect command output into a text file using `>`
- How to collect system information with `systeminfo`
- How to inspect network configuration with `ipconfig /all`
- Why Windows command-line paths containing spaces require quotation marks
- Why recovery information should be saved before a bare-metal OS installation
## Installation Notes

## What I Learned

_To be completed after the project._ 
