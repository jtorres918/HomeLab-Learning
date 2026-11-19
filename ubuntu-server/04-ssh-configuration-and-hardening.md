# 04 - SSH Configuration and Hardening

**Project:** HomeLab-Learning  
**Date:** November 9, 2026  
**Hypervisor:** Proxmox VE 9.2.26  
**Virtual Machine:** VM 100 - ubuntu-server  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Client Operating System:** Windows 11  
**Status:** Completed - SSH key authentication configured and verified

---

## 1. Objective

The objective of this exercise was to securely configure remote administration of my Ubuntu Server virtual machine from a Windows 11 computer.

The goals were to:

- Understand how SSH works.
- Verify that OpenSSH Server was running.
- Connect to Ubuntu remotely using Windows PowerShell.
- Generate an ED25519 public/private SSH key pair.
- Transfer the public key to Ubuntu.
- Configure key-based SSH authentication.
- Create a simplified SSH connection shortcut.
- Disable password-based SSH authentication.
- Verify that SSH configuration changes were valid.
- Test that key-based login still worked.
- Confirm that password-only login was rejected.
- Maintain an alternative recovery method through Proxmox.

This exercise introduced important Linux administration and cybersecurity concepts.

---

## 2. Home Lab Environment

### System Information

| Component | Configuration |
|---|---|
| Physical host | TRIGKEY Mini PC |
| Hypervisor | Proxmox VE |
| Proxmox IP | `192.168.1.200` |
| Ubuntu VM | VM 100 |
| Ubuntu hostname | `ubuntu-server` |
| Ubuntu IP (current) | `192.168.1.22` |
| Ubuntu user | `jansen` |
| Network interface | `ens18` |
| Windows client | Windows 11 |
| SSH port | TCP 22 |
| SSH shortcut | `ubuntu-lab` |
| Key algorithm | ED25519 |

The Ubuntu VM connects to the home LAN through the Proxmox `vmbr0` bridge.

> **Current network update — November 16, 2026:** Ubuntu VM 100 originally used the DHCP-assigned address `192.168.1.17`. After a power outage it received `192.168.1.22`, which is now reserved on the NETGEAR RAX70 for MAC address `BC:24:11:23:80:D9`. Historical installation commands and captured output showing `.17` are intentionally preserved. The current address for SSH and lab connectivity is `192.168.1.22/24`.


All SSH configuration was performed on the local home network.

No public internet SSH port forwarding was configured.

---

## 3. Understanding SSH

SSH stands for **Secure Shell**.

SSH is a protocol that provides encrypted communication between a client and a remote system.

It is commonly used for:

- Remote Linux administration
- Secure command execution
- Server maintenance
- File transfers
- Automation
- Infrastructure management
- Secure tunneling

### SSH Client vs. SSH Server

**SSH Client**

The application initiating the connection.

In this project, the SSH client was OpenSSH running in Windows PowerShell through Windows Terminal.

**SSH Server**

The service accepting incoming SSH connections.

In this project, the SSH server was OpenSSH Server running on Ubuntu.

### SSH Connection

```text
Windows 11 Laptop
SSH Client
      |
      | Encrypted SSH connection
      | TCP Port 22
      |
      v
Ubuntu Server VM 100
192.168.1.22
OpenSSH Server
```

SSH encrypts traffic between the client and server, protecting communications against passive network eavesdropping.

---

## 4. Verify the SSH Server

OpenSSH Server was installed during the Ubuntu Server installation.

Before configuring key authentication, verified that the service was running.

### Command

```bash
sudo systemctl status ssh
```

### Result

```text
Active: active (running)
```

### Command Explanation

| Component | Meaning |
|---|---|
| `sudo` | Execute with elevated privileges |
| `systemctl` | Manage and inspect systemd services |
| `status` | Show the current state of a service |
| `ssh` | OpenSSH server service |

### What I Learned

A service must be running and reachable before clients can connect to it.

`systemctl` is an important Linux administration tool for managing system services.

---

## 5. Initial SSH Connection from Windows

Opened Windows Terminal using PowerShell.

Connected to Ubuntu using:

```powershell
ssh jansen@192.168.1.17
```

### Command Breakdown

**`ssh`**

Starts the OpenSSH client.

**`jansen`**

The username on the remote Ubuntu server.

**`@`**

Separates the username from the destination hostname or IP address.

**`192.168.1.17`**

The IP address of the Ubuntu VM.

### First Connection

During the initial connection, SSH requested confirmation of the server's host key.

The server's host key is used to establish the identity of the SSH server.

After accepting the host key, Ubuntu requested the account password.

The connection succeeded.

### Result

Successfully opened an interactive Ubuntu shell from Windows PowerShell.

### Security Note

The server host-key fingerprint should ideally be verified through a trusted channel before accepting it for the first time.

Accepting an unfamiliar host key without verification can expose a connection to impersonation risks.

---

## 6. Understanding SSH Authentication Methods

SSH supports several authentication mechanisms.

Two important methods are:

### Password Authentication

The client authenticates using the remote account's password.

Advantages:

- Easy to configure.
- Convenient for initial access.

Disadvantages:

- Passwords can be guessed or reused.
- Internet-exposed services may receive automated password attacks.
- Security depends heavily on password strength and server controls.

### Public-Key Authentication

The client proves possession of a private cryptographic key.

The server uses the corresponding authorized public key to verify authentication.

Advantages:

- Does not require transmitting the Linux account password.
- Supports strong cryptographic authentication.
- Suitable for server administration and automation.
- Enables additional protection with a private-key passphrase.

For this home lab, public-key authentication was selected.

---

## 7. Generate an ED25519 SSH Key Pair

Returned to Windows PowerShell.

Executed:

```powershell
ssh-keygen -t ed25519 -C "homelab-ubuntu"
```

### Command Breakdown

| Option | Purpose |
|---|---|
| `ssh-keygen` | Generate and manage SSH keys |
| `-t` | Specify the key algorithm |
| `ed25519` | Use the ED25519 signature algorithm |
| `-C` | Attach a comment to the key |
| `homelab-ubuntu` | Descriptive key comment |

### Why ED25519?

ED25519 is a modern digital signature algorithm.

It is widely supported by current OpenSSH implementations and provides strong cryptographic security with relatively small key sizes.

### Selected Key Location

Instead of overwriting an existing SSH key, created a dedicated key for the home lab Ubuntu server.

Private key:

```text
C:\Users\janse\.ssh\homelab_ubuntu
```

Public key:

```text
C:\Users\janse\.ssh\homelab_ubuntu.pub
```

### Important

The key-generation process also requested a passphrase.

A passphrase was configured to protect the private key.

**The private key passphrase is not documented in GitHub.**

---

## 8. Understanding Public and Private Keys

SSH key authentication uses asymmetric cryptography.

A key pair contains two mathematically related keys.

### Private Key

The private key must remain secret.

In this project:

```text
homelab_ubuntu
```

It is stored on the Windows client.

The private key is used by the client to prove its identity during SSH authentication.

### Public Key

The public key can be distributed to systems where authentication should be permitted.

In this project:

```text
homelab_ubuntu.pub
```

The public key is installed on Ubuntu.

### How SSH Key Authentication Works

1. Windows initiates an SSH connection.
2. Ubuntu identifies an authorized public key.
3. The SSH protocol performs a cryptographic authentication exchange.
4. Windows proves possession of the matching private key.
5. Ubuntu verifies the proof using the public key.
6. Authentication succeeds if the verification and other applicable checks pass.

The private key is not transferred to Ubuntu during this process.

### Security Practices

- Never upload private keys to GitHub.
- Never share private-key passphrases.
- Do not copy private keys unnecessarily.
- Protect private-key backups.
- Use separate keys when appropriate.
- Revoke compromised public keys from servers.

---

## 9. Inspect the Public Key in PowerShell

Used PowerShell to display the public key.

### Command

```powershell
Get-Content "$env:USERPROFILE\.ssh\homelab_ubuntu.pub"
```

### Command Explanation

**`Get-Content`**

Reads the contents of a file.

**`$env:USERPROFILE`**

Represents the current Windows user's profile directory.

**`.ssh`**

The directory commonly used for SSH configuration and key files.

**`.pub`**

Identifies the public-key file.

### Result

The public key began with:

```text
ssh-ed25519
```

And included the descriptive comment:

```text
homelab-ubuntu
```

The full key value was not necessary to include in the project documentation.

---

## 10. Transfer the Public Key to Ubuntu

The next step was to install the Windows public key into Ubuntu's SSH authorized-keys file.

Executed from Windows PowerShell:

```powershell
Get-Content "$env:USERPROFILE\.ssh\homelab_ubuntu.pub" | ssh jansen@192.168.1.17 "umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

The initial Ubuntu account password was used to authenticate this connection.

### Command Breakdown

**`Get-Content`**

Reads the public key from Windows.

**`|`**

The PowerShell pipeline passes the public-key text to the SSH command.

**`ssh jansen@192.168.1.17`**

Connects to Ubuntu.

**`umask 077`**

Sets restrictive default permissions for newly created files and directories.

**`mkdir -p ~/.ssh`**

Creates the user's `.ssh` directory if it does not already exist.

**`cat >> ~/.ssh/authorized_keys`**

Appends the public key to the `authorized_keys` file.

### What is authorized_keys?

The `authorized_keys` file lists public keys permitted to authenticate to a Linux user account.

Location:

```text
~/.ssh/authorized_keys
```

The `~` symbol represents the current user's home directory.

For the Ubuntu account in this project:

```text
/home/jansen/.ssh/authorized_keys
```

### Why Append Instead of Overwrite?

The operator:

```text
>>
```

Appends data to an existing file.

The operator:

```text
>
```

Overwrites the file.

Appending preserves other authorized public keys that might already be present.

Running the same append command repeatedly can create duplicate key entries, so the file should be inspected before repeating it.

### Result

The Windows public key was successfully installed on Ubuntu.

---

## 11. Test SSH Public-Key Authentication

After installing the public key, tested authentication using the private key.

### Command

```powershell
ssh -i "$env:USERPROFILE\.ssh\homelab_ubuntu" jansen@192.168.1.17
```

### Command Explanation

The `-i` option identifies the private-key file that OpenSSH should use.

### Result

Windows requested the private key's passphrase.

After entering the passphrase, Ubuntu accepted the connection.

The Ubuntu account password was not required.

### Verification

**Public-key authentication was successfully configured.**

This was tested before disabling password-based login.

---

## 12. Create a Windows SSH Configuration Shortcut

The SSH command was functional but lengthy.

To simplify future access, created an OpenSSH client configuration entry.

### Configuration File Location

```text
C:\Users\janse\.ssh\config
```

### Check Whether the File Already Exists

Executed:

```powershell
Test-Path "$env:USERPROFILE\.ssh\config"
```

### Result

```text
False
```

The SSH configuration file did not yet exist.

### Configuration Added (original, November 9)

```sshconfig
Host ubuntu-lab
    HostName 192.168.1.17
    User jansen
    IdentityFile ~/.ssh/homelab_ubuntu
    IdentitiesOnly yes
```

### Configuration Explanation

| Directive | Function |
|---|---|
| `Host` | Defines an SSH connection alias |
| `HostName` | Specifies the server's IP or hostname |
| `User` | Specifies the remote username |
| `IdentityFile` | Identifies the private key |
| `IdentitiesOnly` | Limits which configured identities are offered |

### Why Use a Shortcut?

Instead of entering:

```powershell
ssh -i "$env:USERPROFILE\.ssh\homelab_ubuntu" jansen@192.168.1.17
```

The connection could be initiated using:

```powershell
ssh ubuntu-lab
```

This makes administration faster and reduces the chance of entering an incorrect username, address, or key path.

---

## 13. Troubleshooting - Windows Notepad Saved config.txt

### Problem

While creating the SSH configuration file, Windows Notepad saved the file as:

```text
config.txt
```

Instead of:

```text
config
```

When attempting to connect:

```powershell
ssh ubuntu-lab
```

The SSH client reported:

```text
Could not resolve hostname ubuntu-lab: No such host is known.
```

### Root Cause

OpenSSH was expecting a client configuration file named:

```text
config
```

Not:

```text
config.txt
```

Because the file had the wrong name, OpenSSH did not load the alias configuration.

### Resolution

Used Windows PowerShell to rename the file.

```powershell
Rename-Item "$env:USERPROFILE\.ssh\config.txt" "config"
```

### Verify the Directory Contents

```powershell
Get-ChildItem "$env:USERPROFILE\.ssh" -Force | Select-Object Name
```

Confirmed the directory contained:

```text
config
homelab_ubuntu
homelab_ubuntu.pub
```

Other existing SSH files, including earlier key pairs and known-host files, were preserved.

### Retest

```powershell
ssh ubuntu-lab
```

Result:

Successfully connected to Ubuntu using the configured ED25519 key.

### What I Learned

- SSH configuration files must use the correct names and paths.
- Windows applications may automatically append file extensions.
- PowerShell can rename and inspect files.
- Error messages can help identify client-side configuration problems.
- Not every SSH connection failure is caused by network connectivity.

---

## 14. Inspect Ubuntu SSH Server Settings

After verifying key-based authentication, the next goal was to disable SSH password login.

Before making changes, inspected the effective SSH server configuration.

### Command

```bash
sudo sshd -T | grep -E '^(passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication) '
```

### Command Breakdown

**`sshd`**

The OpenSSH server daemon.

**`-T`**

Displays the effective SSH server configuration.

**`|`**

Passes output from the first command to the second command.

**`grep`**

Searches text for matching patterns.

**`-E`**

Enables extended regular expressions.

**`^`**

Matches the beginning of a line.

**`(pattern1|pattern2)`**

Matches one of the listed alternatives.

### Initial Result

```text
pubkeyauthentication yes
passwordauthentication yes
kbdinteractiveauthentication no
```

### Analysis

Public-key authentication was already enabled.

However, password-based SSH authentication was also enabled.

The next step was to disable password authentication while preserving public-key access.

---

## 15. Create a Dedicated SSH Hardening Configuration File

Instead of making extensive changes to the main OpenSSH configuration, created a dedicated hardening file.

### Command

```bash
sudo nano /etc/ssh/sshd_config.d/01-homelab-hardening.conf
```

### Configuration Added

```text
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
```

### Explanation

**PasswordAuthentication no**

Disables SSH password authentication.

**PubkeyAuthentication yes**

Enables SSH public-key authentication.

**KbdInteractiveAuthentication no**

Disables keyboard-interactive authentication.

### Why Use sshd_config.d?

The `/etc/ssh/sshd_config.d/` directory contains SSH server configuration snippets.

Using a dedicated file helps organize custom configuration and makes later auditing or modification easier.

The effective configuration must still be verified because OpenSSH configuration precedence and other included settings can influence the resulting behavior.

### Important Security Note

Disabling SSH password authentication does not disable the Ubuntu account password.

The password can still be used for authorized local login and `sudo` operations.

Only the relevant remote SSH authentication methods were changed.

---

## 16. Learning the Nano Text Editor

The hardening configuration was created using Nano.

### Nano Basics

Nano is a terminal-based text editor commonly available on Linux systems.

It allows administrators to create and edit configuration files without a graphical desktop.

### Keyboard Shortcuts

| Shortcut | Function |
|---|---|
| `Ctrl + O` | Write Out / save |
| `Enter` | Confirm filename |
| `Ctrl + X` | Exit |
| `Ctrl + W` | Search |
| `Ctrl + K` | Cut a line |
| `Ctrl + U` | Paste a cut line |

### Issue Encountered

After pressing:

```text
Ctrl + O
```

Nano displayed additional options at the bottom of the terminal.

This initially caused confusion.

### Explanation

`Ctrl + O` opens Nano's Write Out prompt.

The displayed options are normal.

To save:

1. Press `Ctrl + O`.
2. Confirm the filename.
3. Press `Enter`.
4. Press `Ctrl + X` to exit.

### Result

The SSH hardening configuration was successfully saved.

---

## 17. Validate SSH Configuration Syntax

Before applying the new configuration, checked it for syntax errors.

### Command

```bash
sudo sshd -t
```

### Result

The command returned no errors.

### Explanation

The `-t` option performs an SSH server configuration test.

If a configuration problem exists, OpenSSH normally reports an error.

No output with a successful exit status indicates that the syntax validation passed.

### Why This Matters

A malformed SSH configuration may prevent the service from accepting new connections.

Validating the configuration before applying it reduces the risk of administrative lockout.

---

## 18. Verify Effective SSH Authentication Settings

After saving the hardening configuration, inspected the effective settings again.

### Command

```bash
sudo sshd -T | grep -E '^(passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication) '
```

### Actual Result

```text
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

### Analysis

The effective configuration indicated:

- Public-key authentication enabled.
- Password authentication disabled.
- Keyboard-interactive authentication disabled.

This confirmed the intended authentication settings were being interpreted by OpenSSH.

---

## 19. Apply the SSH Configuration

Reloaded the SSH service.

### Command

```bash
sudo systemctl reload ssh
```

### Command Explanation

**`systemctl`**

Controls systemd services.

**`reload`**

Requests that the service reload its configuration without performing a full stop-and-start cycle.

**`ssh`**

The OpenSSH server service.

### Why Reload Instead of Restart?

A reload allows OpenSSH to apply updated configuration while generally avoiding unnecessary disruption of established connections.

### Security Procedure

Kept the original SSH session open while testing the new settings.

This provided an existing administrative connection in case the new configuration prevented fresh logins.

### Result

The SSH service reloaded successfully.

---

## 20. Verify SSH Key Authentication Still Works

Opened a second Windows Terminal tab.

Executed:

```powershell
ssh ubuntu-lab
```

### Result

The SSH client requested the ED25519 private key passphrase.

After entering the passphrase, Ubuntu accepted the connection.

### Verification

Key-based authentication continued functioning after disabling SSH password authentication.

This confirmed that the server could still be administered remotely.

---

## 21. Verify Password Authentication Is Disabled

The next step was to intentionally attempt SSH access without using a public key.

Opened another Windows PowerShell tab.

### Test Command

```powershell
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password,keyboard-interactive -o BatchMode=yes jansen@192.168.1.17
```

### Command Breakdown

| Option | Purpose |
|---|---|
| `-o` | Supply an SSH client option |
| `PubkeyAuthentication=no` | Prevent public-key authentication for this test |
| `PreferredAuthentications=password,keyboard-interactive` | Request password-related authentication methods |
| `BatchMode=yes` | Prevent interactive password prompts |
| `jansen@192.168.1.17` | Target Ubuntu user and server |

### Actual Result

```text
Permission denied
```

### Analysis

The password-related authentication attempt was rejected.

This was the expected result.

### Security Verification

Two separate tests were successful:

**Test 1:** Key-based login succeeded.

**Test 2:** Password-based login was denied.

Together, these tests confirmed that the intended SSH authentication policy was working for the tested account and connection.

---

## 22. Current SSH Connection Procedure

To access Ubuntu from the Windows laptop:

1. Open Windows Terminal.
2. Use PowerShell.
3. Execute:

```powershell
ssh ubuntu-lab
```

4. Enter the ED25519 private key passphrase if prompted.

### Configuration Summary

```text
SSH Alias: ubuntu-lab
Destination: 192.168.1.17
Username: jansen
Private Key: ~/.ssh/homelab_ubuntu
Port: 22
Authentication: ED25519 public key
```

### End SSH Session

To disconnect from Ubuntu:

```bash
exit
```

This closes the remote shell and returns to Windows PowerShell.

---

## 23. Recovery Through Proxmox

Disabling password-based SSH login increases the importance of maintaining an alternative administrative access method.

If SSH becomes unavailable, Ubuntu can still be accessed through the Proxmox VM console.

### Recovery Path

1. Open the Proxmox web interface.
2. Select VM 100.
3. Open the Console.
4. Log in using the Ubuntu account credentials.
5. Inspect the SSH configuration.
6. Correct any configuration problems.
7. Validate the configuration before reloading SSH.

Proxmox management URL:

```text
https://192.168.1.200:8006
```

### Important Distinction

The Proxmox virtual console is not the same as an SSH connection.

It provides access through the virtualization management interface rather than through Ubuntu's SSH service.

This makes it a useful recovery option if SSH networking or authentication is misconfigured.

---

## 24. Security Benefits

The following security measures were implemented:

### Public-Key Authentication

SSH uses cryptographic authentication instead of requiring the Linux account password for remote SSH login.

### Private-Key Passphrase

The SSH private key is protected using a passphrase.

### Password Authentication Disabled

Remote login through SSH password authentication was disabled.

### Keyboard-Interactive Authentication Disabled

Keyboard-interactive SSH authentication was disabled.

### Verified Configuration

Configuration syntax and effective settings were checked before reloading SSH.

### Controlled Testing

Successful key authentication and failed password authentication were both verified.

### Alternative Recovery Access

Proxmox console access remains available for recovery.

### Remaining Security Considerations

These settings improve SSH authentication security but do not provide complete server hardening by themselves.

Future work should include reviewing firewall rules, update practices, logs, account permissions, backups, and remote-access exposure.

---

## 25. Troubleshooting Summary

| Problem | Cause | Resolution |
|---|---|---|
| SSH alias did not resolve | Notepad saved `config.txt` | Renamed file to `config` |
| Confusion using Nano | Write Out prompt displayed additional options | Used Ctrl+O, Enter, Ctrl+X |
| Password login initially enabled | OpenSSH effective configuration allowed passwords | Added hardening configuration |
| Potential SSH lockout | Authentication policy was being changed | Kept original connection open and tested a second session |
| Need to verify security | Configuration alone does not prove behavior | Tested key login and deliberate password-authentication failure |

---

## 26. Commands Learned

### Windows PowerShell

Generate an SSH key:

```powershell
ssh-keygen -t ed25519 -C "homelab-ubuntu"
```

View the public key:

```powershell
Get-Content "$env:USERPROFILE\.ssh\homelab_ubuntu.pub"
```

Transfer the public key:

```powershell
Get-Content "$env:USERPROFILE\.ssh\homelab_ubuntu.pub" | ssh jansen@192.168.1.17 "umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

Connect with a specific private key:

```powershell
ssh -i "$env:USERPROFILE\.ssh\homelab_ubuntu" jansen@192.168.1.17
```

Check SSH config file existence:

```powershell
Test-Path "$env:USERPROFILE\.ssh\config"
```

Rename the SSH config file:

```powershell
Rename-Item "$env:USERPROFILE\.ssh\config.txt" "config"
```

Inspect the SSH directory:

```powershell
Get-ChildItem "$env:USERPROFILE\.ssh" -Force | Select-Object Name
```

Connect using the SSH shortcut:

```powershell
ssh ubuntu-lab
```

Test rejection of password-only SSH access:

```powershell
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password,keyboard-interactive -o BatchMode=yes jansen@192.168.1.17
```

### Ubuntu Linux

Inspect SSH service:

```bash
sudo systemctl status ssh
```

Inspect authentication settings:

```bash
sudo sshd -T | grep -E '^(passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication) '
```

Edit the hardening configuration:

```bash
sudo nano /etc/ssh/sshd_config.d/01-homelab-hardening.conf
```

Test configuration syntax:

```bash
sudo sshd -t
```

Reload the SSH service:

```bash
sudo systemctl reload ssh
```

End an SSH session:

```bash
exit
```

---

## 27. What I Learned

This exercise provided hands-on experience with:

1. Understanding SSH client-server communication.
2. Using Windows PowerShell for remote Linux administration.
3. Generating ED25519 cryptographic keys.
4. Understanding the difference between public and private keys.
5. Protecting private keys with passphrases.
6. Configuring the Linux `authorized_keys` file.
7. Using PowerShell commands and pipelines.
8. Creating OpenSSH client configuration aliases.
9. Troubleshooting Windows filename extensions.
10. Editing Linux files using Nano.
11. Understanding Linux file paths and permissions.
12. Inspecting effective OpenSSH settings.
13. Validating configuration syntax.
14. Reloading Linux services.
15. Disabling insecure or unnecessary authentication methods.
16. Testing security controls after configuration changes.
17. Maintaining recovery access during administrative changes.
18. Documenting operational and security procedures.

---

## 28. Completion Checklist

- [x] Verified OpenSSH Server was running.
- [x] Established initial SSH connection from Windows.
- [x] Generated ED25519 SSH key pair.
- [x] Protected the private key with a passphrase.
- [x] Installed the public key on Ubuntu.
- [x] Verified public-key authentication.
- [x] Created Windows SSH client configuration.
- [x] Corrected the `config.txt` filename issue.
- [x] Verified the `ssh ubuntu-lab` shortcut.
- [x] Inspected initial SSH authentication settings.
- [x] Created a dedicated SSH hardening configuration file.
- [x] Learned to save and exit Nano.
- [x] Validated SSH configuration syntax.
- [x] Verified password authentication was disabled.
- [x] Reloaded the SSH service.
- [x] Confirmed new SSH key login succeeded.
- [x] Confirmed password-only authentication was rejected.
- [x] Maintained Proxmox console recovery access.

---

## 29. Future Improvements

- [ ] Configure DHCP reservation for Ubuntu Server.
- [ ] Verify and document SSH host-key fingerprints.
- [ ] Configure firewall rules.
- [ ] Review SSH authentication logs.
- [ ] Learn additional Linux file permission commands.
- [ ] Explore SSH agents for passphrase management.
- [ ] Configure regular VM backups.
- [ ] Test restoration of SSH access.
- [ ] Review SSH account and user restrictions.
- [ ] Explore secure remote access through a VPN.

---

## 30. Final Outcome

Successfully configured SSH public-key authentication between Windows 11 and Ubuntu Server VM 100.

An ED25519 key pair was generated, the public key was installed on Ubuntu, and a Windows OpenSSH shortcut was created.

SSH password authentication and keyboard-interactive authentication were disabled.

The effective server configuration was verified, the SSH service was reloaded, and connection tests confirmed that public-key authentication worked while password-only authentication was denied.

The server can now be accessed from Windows Terminal with:

```powershell
ssh ubuntu-lab
```

The current Windows SSH alias uses `HostName 192.168.1.22`; the original `.17` commands above document the initial setup. See `05-power-outage-recovery-and-dhcp-reservation.md` for the change history.

**Final Status: SSH key-based remote administration configured, hardened, and verified.**

---

**Next Project:** Configure a DHCP reservation for Ubuntu Server and continue learning Linux networking fundamentals.
