# 09 - Docker Installation and First Containers

**Date:** November 19, 2026  
**Project:** HomeLab-Learning  
**Platform:** Proxmox VE 9.2.26  
**Virtual Machine:** 100 - ubuntu-server  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Status:** Installation and first-container exercises complete; continued in file 10

---

## 1. Project Objective

Install Docker Engine on Ubuntu Server and begin learning container fundamentals.

The goals were to:

- Understand the difference between virtual machines and containers.
- Install Docker Engine using Docker's official Ubuntu repository.
- Understand Linux package management and repository signing keys.
- Verify that the Docker service is running.
- Download and run a Docker image.
- Create multiple containers from one image.
- Inspect container status and exit codes.
- Begin learning Docker troubleshooting commands.

## 2. Home Lab Environment

Docker was installed inside the existing Ubuntu virtual machine, not directly on the Proxmox host.

| Component | Configuration |
|---|---|
| Physical Host | Trigkey Mini PC |
| Hypervisor | Proxmox VE 9.2.26 |
| VM ID | 100 |
| Hostname | ubuntu-server |
| Ubuntu IP | 192.168.1.22 |
| Ubuntu Version | 26.04.1 LTS |
| RAM | 2048 MiB |
| CPU | 2 virtual cores |
| Virtual Disk | 32 GiB |
| Network Interface | ens18 |
| Kernel After Update | 7.0.0-44-generic |

Windows PowerShell was used to connect to Ubuntu through SSH.

```powershell
ssh ubuntu-lab
```

Once connected, commands were executed inside the Ubuntu virtual machine.

## 3. Understanding Docker

Docker is a platform for creating, running, and managing applications inside containers.

### Virtual Machines vs. Containers

A virtual machine runs its own operating system and virtualized hardware environment.

A Docker container shares the host operating system's kernel while providing an isolated environment for an application.

Our architecture:

```text
Trigkey Mini PC
    |
    +-- Proxmox VE
          |
          +-- VM 100: Ubuntu Server
                |
                +-- Docker Engine
                      |
                      +-- Container 1
                      +-- Container 2
                      +-- Future Applications
```

### Docker Terminology

| Term | Meaning |
|---|---|
| Docker Engine | Software that runs and manages containers |
| Docker CLI | Command-line interface for interacting with Docker |
| Docker Daemon | Background service responsible for managing containers |
| Docker Image | Reusable template used to create containers |
| Docker Container | Instance created from an image |
| Docker Hub | Online registry that distributes container images |
| Docker Compose | Tool for defining and managing multi-container applications |

**Important lesson:** Docker containers are generally more lightweight than virtual machines, but containers do not provide the same default isolation boundary as separate virtual machines.

## 4. Checking for an Existing Docker Installation

Executed:

```bash
command -v docker
```

### Command Explanation

- `command`: Shell built-in command.
- `-v`: Display how a command would be located.
- `docker`: Executable being checked.

**Result:** No output.

Ubuntu could not locate a Docker executable in the command search path.

## 5. Updating Ubuntu Before Installation

Executed:

```bash
sudo apt update
sudo apt upgrade
```

### Command Explanation

- `sudo`: Execute with administrator privileges.
- `apt`: Ubuntu's package management tool.
- `update`: Refresh available package information.
- `upgrade`: Install available upgrades for installed packages.

The upgrade included a newer Linux kernel.

**Previous kernel:** `7.0.0-43-generic`

**Updated kernel:** `7.0.0-44-generic`

After the package upgrade, Ubuntu reported a pending kernel update requiring a reboot.

Executed:

```bash
sudo reboot
```

After reconnecting through SSH, verified the running kernel:

```bash
uname -r
```

Output:

```text
7.0.0-44-generic
```

### Phased Updates

Ubuntu deferred three packages:

```text
fwupd
libfwupd3
ubuntu-drivers-common
```

These packages were not forced to upgrade.

**Lesson learned:** An installed kernel update generally requires a reboot before the new kernel becomes active.

## 6. Checking for Conflicting Container Packages

Executed:

```bash
dpkg -l docker.io docker-compose containerd runc
```

### Command Explanation

- `dpkg`: Debian package management utility.
- `-l`: List package information.
- Remaining arguments: Package names being checked.

The command reported that the specified packages were not found.

**Result:** No conflicting packages from this list were installed.

## 7. Installing Docker Prerequisites

Executed:

```bash
sudo apt install ca-certificates curl
```

### Package Explanation

**ca-certificates**

Provides trusted certificate authorities used to verify HTTPS connections.

**curl**

Command-line utility used to transfer data to or from URLs.

### Result

Both packages were already installed and up to date.

No additional packages needed to be installed during this step.

## 8. Creating the APT Keyring Directory

Executed:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

### Command Explanation

| Component | Meaning |
|---|---|
| sudo | Execute as administrator |
| install | Utility for creating directories and installing files |
| -m | Specify permission mode |
| 0755 | Directory permission setting |
| -d | Create a directory |
| /etc/apt/keyrings | Directory for repository signing keys |

### Understanding Linux Permissions

Linux permissions are divided into:

- Owner
- Group
- Others

Permission values:

| Permission | Letter | Numeric Value |
|---|---|---|
| Read | r | 4 |
| Write | w | 2 |
| Execute | x | 1 |

For the permission mode `0755`:

- Leading `0`: Octal notation, with no special permission bits set.
- Owner `7`: Read + write + execute.
- Group `5`: Read + execute.
- Others `5`: Read + execute.

Symbolic representation:

```text
drwxr-xr-x
```

For a directory, execute permission means permission to traverse the directory.

**Result:** Directory preparation completed without errors.

## 9. Downloading Docker's Signing Key

Executed:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

### Command Explanation

| Component | Meaning |
|---|---|
| curl | Download data from a URL |
| -f | Report HTTP errors |
| -s | Silent mode |
| -S | Display errors despite silent mode |
| -L | Follow redirects |
| -o | Save downloaded content to a file |

The signing key was downloaded from Docker's official website.

### Purpose of the Signing Key

APT uses trusted public signing keys to verify repository metadata.

This helps establish that repository information was signed using the corresponding private key.

The downloaded key is public information, not a password or private secret.

The command completed without reporting errors.

## 10. Configuring Signing-Key Permissions

Executed:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Command Explanation

- `chmod`: Change file permissions.
- `a`: All users.
- `+r`: Add read permission.
- File path: Docker's downloaded public signing key.

Verified permissions with:

```bash
ls -l /etc/apt/keyrings/docker.asc
```

Expected permissions:

```text
-rw-r--r--
```

These correspond to `0644`:

- Owner: Read and write.
- Group: Read only.
- Others: Read only.

## 11. Identifying Ubuntu's Release Codename

Executed:

```bash
. /etc/os-release && echo "$VERSION_CODENAME"
```

### Command Explanation

- `.`: Source a file into the current shell.
- `/etc/os-release`: Contains operating system identification information.
- `&&`: Execute the next command if the previous command succeeds.
- `echo`: Display text or variable contents.
- `$VERSION_CODENAME`: Operating system release codename.

### Result

```text
resolute
```

Ubuntu 26.04 uses the codename `resolute`.

This value was needed for the Docker repository configuration.

## 12. Configuring Docker's Official Repository

Created the repository configuration using:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: resolute
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### Understanding the Command

- `sudo tee`: Write text to a file with administrator privileges.
- `>`: Redirect command output.
- `/dev/null`: Discard unwanted output.
- `<<EOF`: Begin a here-document containing multiple lines.
- Final `EOF`: End the here-document.

### Repository Settings

| Setting | Purpose |
|---|---|
| Types | Debian-format packages |
| URIs | Official Docker repository |
| Suites | Ubuntu resolute |
| Components | Stable release channel |
| Architectures | AMD64 / x86-64 |
| Signed-By | Repository verification key |

Verified the file using:

```bash
cat /etc/apt/sources.list.d/docker.sources
```

The expected configuration was displayed.

**Lesson learned:** APT repository definitions specify where software can be downloaded and which signing key should be used to validate the repository.

## 13. Updating Package Information After Adding Docker

Executed:

```bash
sudo apt update
```

Ubuntu successfully retrieved Docker repository information.

Relevant output included:

```text
https://download.docker.com/linux/ubuntu resolute InRelease
https://download.docker.com/linux/ubuntu resolute/stable amd64 Packages
```

This confirmed that APT could access Docker's repository.

An additional `sudo apt upgrade` was run. No packages were upgraded, and the three previously deferred packages remained deferred.

## 14. Installing Docker Engine

Executed:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Installed Components

| Package | Purpose |
|---|---|
| docker-ce | Docker Engine |
| docker-ce-cli | Docker command-line interface |
| containerd.io | Container runtime |
| docker-buildx-plugin | Advanced image-building functionality |
| docker-compose-plugin | Docker Compose |

The installation also installed required dependencies.

### Observed Installation Details

- Docker Engine package version: `29.10.0`
- Docker Compose plugin package version: `5.8.0`
- Docker Buildx plugin package version: `0.39.0`
- Packages downloaded: approximately 94.2 MB

The installation completed successfully.

Systemd service links were created for Docker and containerd, allowing them to start automatically when Ubuntu boots.

## 15. Verifying Docker Engine

Executed:

```bash
sudo systemctl status docker
```

### Observed Output

```text
Loaded: loaded (...; enabled; preset: enabled)
Active: active (running)
Main PID: 2147 (dockerd)
```

### Interpretation

| Output | Meaning |
|---|---|
| enabled | Docker is configured to start automatically |
| active (running) | Docker Engine is currently running |
| dockerd | Docker's background daemon |
| Main PID | Process ID of the daemon |

Docker also reported:

```text
API listen on /run/docker.sock
```

This Unix socket is used for communication with the Docker daemon.

### Linux Daemons

A daemon is a program that runs in the background to provide a service.

Examples:

- `sshd`: SSH server daemon.
- `dockerd`: Docker daemon.

**Result:** Docker Engine operational.

## 16. Running the First Docker Container

Executed:

```bash
sudo docker run hello-world
```

### Command Explanation

- `sudo`: Execute with administrator privileges.
- `docker`: Docker CLI.
- `run`: Create and start a container from an image.
- `hello-world`: Image used to create the container.

### What Happened

1. Docker searched for the image locally.
2. The image was not found locally.
3. Docker downloaded the image from Docker Hub.
4. Docker created a container from that image.
5. The container displayed a verification message.
6. The container exited after completing its task.

### Observed Output

```text
Hello from Docker!
This message shows that your installation appears to be
working correctly.
```

**Result:** First Docker container executed successfully.

## 17. Listing Docker Containers

Executed:

```bash
sudo docker ps -a
```

### Command Explanation

- `docker ps`: Display containers.
- `-a`: Include stopped containers.

The first container appeared with:

```text
IMAGE: hello-world
COMMAND: "/hello"
STATUS: Exited (0)
NAME: epic_cohen
```

Docker automatically generated the name `epic_cohen`.

### Understanding Exit Codes

| Exit Code | Meaning |
|---|---|
| 0 | Successful execution |
| 1 | General error |
| 127 | Command not found |
| 137 | Commonly indicates termination by SIGKILL |

**Important lesson:** A container that has exited is not necessarily broken. Some containers are designed to perform a task and stop.

## 18. Inspecting Docker Images

Executed:

```bash
sudo docker images
```

### Result

Docker displayed the locally stored image:

```text
hello-world:latest
```

Additional information included an image ID, disk usage, and content size.

### Image vs. Container

An image is a reusable template.

A container is an instance created from an image.

Stopping a container does not automatically delete its image.

The image can be reused to create additional containers without downloading it again.

## 19. Creating a Second Container

Executed:

```bash
sudo docker run --name hello-test-2 hello-world
```

### Command Explanation

- `docker run`: Create and start a container.
- `--name`: Assign a custom container name.
- `hello-test-2`: The chosen container name.
- `hello-world`: Image being used.

Docker reused the already downloaded image.

The new container displayed the Hello from Docker message and exited successfully.

### Verification

Executed:

```bash
sudo docker ps -a
```

### Observed Results

| Container | Image | Status |
|---|---|---|
| epic_cohen | hello-world | Exited (0) |
| hello-test-2 | hello-world | Exited (0) |

Both containers were independent instances created from the same image.

**Lesson learned:** Multiple containers can share the same underlying Docker image.

## 20. Inspecting Container Details

Executed:

```bash
sudo docker inspect hello-test-2
```

### Command Explanation

- `docker`: Docker command-line program.
- `inspect`: Display detailed information about a Docker object.
- `hello-test-2`: Container being inspected.

Docker displayed information in JSON format.

### JSON

JSON stands for JavaScript Object Notation.

It is a structured data format commonly used in:

- APIs
- Application configuration
- Cloud platforms
- Cybersecurity tools
- Automation workflows

### Values Verified

```json
"Status": "exited",
"ExitCode": 0
```

These results showed that the container had stopped after completing its task successfully.

**Lesson learned:** Troubleshooting begins by understanding what an application is expected to do. A stopped container may indicate normal completion rather than a failure.

## 21. Verifying Docker Container Logs

Executed:

```bash
sudo docker logs hello-test-2
```

### Command Explanation

- `sudo`: Execute with administrator privileges.
- `docker`: Docker CLI.
- `logs`: Display output recorded from a container.
- `hello-test-2`: Container whose logs are requested.

### Result

The command displayed the expected Hello World output from the previously completed container.

**Lesson learned:** Container logs remain useful after a short-lived container exits. They provide evidence of what the application printed and help distinguish normal completion from problems.

**Continuation:** The next exercise deployed and tested a long-running Nginx container. Full commands, Docker bridge networking, HTTP tests, lifecycle management, and recovery results are documented in [10 - Docker Networking, Container Management, and Recovery Testing](10-docker-networking-container-management-and-recovery.md).

## 22. Docker Security Considerations

Docker was operated using `sudo` during this installation.

Access to the Docker daemon grants powerful privileges over the host operating system.

Adding an ordinary user to Docker's `docker` group can effectively grant root-level control through Docker.

For now, Docker commands will continue to use `sudo`.

Before deploying network-facing applications, we will also evaluate firewall behavior, published ports, and container security.

## 23. Troubleshooting Concepts Learned

This session introduced a repeatable approach:

1. Determine the expected behavior.
2. Check whether the service is running.
3. Inspect available images and containers.
4. Check container status.
5. Examine exit codes.
6. Read logs for additional evidence.
7. Make changes only after gathering information.

### Useful Commands

| Command | Purpose |
|---|---|
| `command -v docker` | Check whether Docker is available |
| `sudo apt update` | Refresh package information |
| `uname -r` | Display running kernel version |
| `sudo systemctl status docker` | Check Docker daemon status |
| `sudo docker run hello-world` | Create and run a test container |
| `sudo docker ps -a` | List all containers |
| `sudo docker images` | List local images |
| `sudo docker inspect hello-test-2` | Inspect a container |
| `sudo docker logs hello-test-2` | View container output |

## 24. Completion Checklist

- [x] Learned Docker fundamentals.
- [x] Updated Ubuntu.
- [x] Installed and booted into the newer Linux kernel.
- [x] Checked for conflicting container packages.
- [x] Verified prerequisite packages.
- [x] Prepared the APT keyring directory.
- [x] Downloaded Docker's public signing key.
- [x] Configured Docker's official repository.
- [x] Installed Docker Engine.
- [x] Installed Docker CLI, containerd, Buildx, and Compose.
- [x] Verified Docker daemon was running.
- [x] Downloaded the Hello World image.
- [x] Ran the first container.
- [x] Listed stopped containers.
- [x] Inspected locally stored images.
- [x] Created a second named container.
- [x] Inspected container status and exit code.
- [x] Verified container logs (Step 21).
- [x] Learned basic management of a long-running Nginx container (continued in file 10).
- [x] Deployed a long-running containerized web service (continued in file 10).
- [x] Practiced Docker bridge networking and HTTP connectivity (continued in file 10).
- [x] Practiced container troubleshooting through status checks and logs (continued in file 10).
- [x] Configured and verified an automatic container restart policy (continued in file 10).
- [ ] Learn Docker volumes and persistent application data.
- [ ] Learn Docker Compose.
- [ ] Explore published ports and user-defined networks.

## 25. Continuation and Next Learning Objectives

**The installation and first-container learning phase is complete.** The subsequent hands-on work is documented in:

**[10 - Docker Networking, Container Management, and Recovery Testing](10-docker-networking-container-management-and-recovery.md)**

File 10 covers the Nginx deployment, `docker0` bridge, internal HTTP testing, Nginx logs, stopping and starting a container, the `unless-stopped` restart policy, and verified automatic recovery after an Ubuntu VM reboot.

Next topics include Docker port publishing, persistent volumes, user-defined networks, Docker Compose, and deploying useful home lab applications.

**Final Status: Docker Engine installed and operational. Two Hello World containers executed successfully; logs verified. Nginx management, networking, and recovery testing completed and documented in file 10.**
