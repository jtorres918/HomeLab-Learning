
# 10 - Docker Networking, Container Management, and Recovery Testing

**Dates:** November 19–20, 2026  
**Project:** HomeLab-Learning  
**Hypervisor:** Proxmox VE 9.2.26  
**Virtual Machine:** VM 100 - ubuntu-server  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Ubuntu IP:** 192.168.1.22  
**Docker Application:** Nginx  
**Container Name:** lab-web  
**Status:** Networking, container management, and automatic restart testing completed

---

## 1. Project Objectives

Continue learning Docker after successfully installing Docker Engine and running two Hello World containers.

Objectives:

- Examine container logs.
- Deploy a long-running Nginx web server.
- Understand Docker's bridge networking.
- Identify container and bridge IP addresses.
- Test HTTP communication using curl.
- Troubleshoot applications through their logs.
- Stop and restart existing containers.
- Understand and configure Docker restart policies.
- Verify automatic recovery after rebooting Ubuntu.
- Practice checking service status before making changes.

## 2. Home Lab Architecture

Docker Engine runs inside Ubuntu VM 100, which runs on Proxmox.

```text
Trigkey Mini PC
|
+-- Proxmox VE
    Management IP: 192.168.1.200
    |
    +-- vmbr0 (Proxmox Linux bridge)
         |
         +-- VM 100: Ubuntu Server
              LAN IP: 192.168.1.22/24
              Interface: ens18
              |
              +-- Docker Engine
                   |
                   +-- docker0 bridge
                        172.17.0.1/16
                        |
                        +-- lab-web (Nginx)
                             172.17.0.2
                             TCP port 80
```

### Two Different Bridges

**Proxmox vmbr0**

Connects the Ubuntu VM's virtual network interface to the physical home LAN.

**Docker docker0**

Creates a separate virtual network inside Ubuntu for Docker containers.

The two bridges serve different purposes and exist at different layers of the home lab.

## 3. Reviewing Hello World Container Logs

Executed:

```bash
sudo docker logs hello-test-2
```

The command displayed the output from the Hello World container.

### Explanation

| Component | Meaning |
|---|---|
| sudo | Execute with administrator privileges |
| docker | Docker CLI |
| logs | Display recorded container output |
| hello-test-2 | Container whose logs are requested |

The Hello World message confirmed that the earlier test container completed its intended task.

**Lesson:** Logs can provide evidence of what an application did, even after its container stops.

## 4. Deploying an Nginx Web Server

Executed:

```bash
sudo docker run -d --name lab-web nginx:alpine
```

### Command Breakdown

| Component | Meaning |
|---|---|
| docker run | Create and start a new container |
| -d | Detached mode; run in the background |
| --name | Assign a container name |
| lab-web | Chosen container name |
| nginx:alpine | Nginx image based on Alpine Linux |

Docker downloaded the Nginx image because it was not already stored locally.

The container started successfully.

### Why Nginx?

Nginx is web server software commonly used to serve websites, act as a reverse proxy, and handle HTTP traffic.

Unlike Hello World, Nginx is designed to run continuously.

### Important Networking Decision

We did not use the `-p` option when creating the container.

Therefore, TCP port 80 was not published on Ubuntu's LAN address.

The web server was accessible through its internal Docker network, but it was not exposed directly to other home LAN devices through a published host port.

## 5. Verifying the Running Container

Executed:

```bash
sudo docker ps
```

Observed:

```text
IMAGE: nginx:alpine
STATUS: Up
PORTS: 80/tcp
NAMES: lab-web
```

### Important Distinction

The display `80/tcp` indicates that the image/container exposes TCP port 80 internally.

It does not mean Ubuntu's port 80 has been published.

A published port would typically appear in a format such as:

```text
0.0.0.0:8080->80/tcp
```

No such host port mapping was configured for this container.

## 6. Inspecting Docker Networks

Executed:

```bash
sudo docker network ls
```

Docker displayed its default networks:

| Network | Driver | Purpose |
|---|---|---|
| bridge | bridge | Default isolated container network |
| host | host | Uses the host network namespace |
| none | null | Disables normal networking for the container |

The Nginx container used Docker's default bridge network.

### Important Lesson

Docker networking is separate from the Ubuntu LAN configuration.

Ubuntu's LAN IP is `192.168.1.22`, while Docker assigns addresses from its own private subnet.

## 7. Identifying the Container IP Address

Initially attempted:

```bash
sudo docker inspect -f '{{.NetworkSettings.IPAddress}}' lab-web
```

This did not return the expected value.

The inspection was changed to access the IP address within the container's network-specific information:

```bash
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' lab-web
```

### Result

```text
172.17.0.2
```

### Command Explanation

- `docker inspect`: Retrieve detailed Docker object information.
- `-f`: Format the information returned.
- `{{ }}`: Go-template expression syntax.
- `.NetworkSettings.Networks`: Collection of networks attached to the container.
- `range`: Iterate through those network entries.
- `.IPAddress`: Retrieve the container IP address.
- `end`: Finish the template loop.

**Troubleshooting lesson:** When an expected inspection field does not work, examine the data structure and use the correct network-specific field rather than assuming the container has no address.

The container IP is dynamically assigned and should not be assumed to remain unchanged after recreation or network changes.

## 8. Inspecting the Docker Bridge Interface

Executed:

```bash
ip -4 addr show docker0
```

Observed output:

```text
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 172.17.0.1/16 brd 172.17.255.255
    valid_lft forever preferred_lft forever
```

### Command Explanation

| Component | Meaning |
|---|---|
| ip | Linux networking utility |
| -4 | Show IPv4 information |
| addr | Work with IP addresses |
| show | Display information |
| docker0 | Docker's default bridge interface |

### Network Information

| Property | Value |
|---|---|
| Docker bridge address | 172.17.0.1 |
| Container address | 172.17.0.2 |
| Network | 172.17.0.0/16 |
| Subnet mask | 255.255.0.0 |
| Broadcast address | 172.17.255.255 |
| MTU | 1500 bytes |

Ubuntu and the container have addresses in the same Docker subnet.

The Docker bridge provides local connectivity between Ubuntu and containers attached to that network.

## 9. Testing Nginx Through HTTP

Executed from Ubuntu:

```bash
curl -I http://172.17.0.2
```

The HTTP request succeeded.

### Command Explanation

| Component | Meaning |
|---|---|
| curl | Transfer data or make requests to URLs |
| -I | Request HTTP response headers |
| http:// | HTTP protocol |
| 172.17.0.2 | Nginx container address |

Because the URL uses HTTP without an explicit port, curl connects to TCP port 80.

### Important Response

```text
HTTP/1.1 200 OK
```

HTTP status code `200` means Nginx successfully handled the request.

### Lesson

A container showing `Up` does not necessarily prove its application is working.

Testing the actual application protocol provides stronger evidence.

## 10. Analyzing Nginx Logs

Executed:

```bash
sudo docker logs --tail 10 lab-web
```

### Explanation

- `docker logs`: Display container output.
- `--tail 10`: Display the last 10 lines.
- `lab-web`: Nginx container.

Observed an Nginx access log entry:

```text
172.17.0.1 - - [20/Nov/2026:17:43:48 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.18.0"
```

### Log Analysis

| Entry | Meaning |
|---|---|
| 172.17.0.1 | Client IP address on the Docker bridge |
| 20/Nov/2026:17:43:48 +0000 | Request time in UTC |
| HEAD | HTTP request method |
| / | Requested root path |
| HTTP/1.1 | HTTP protocol version |
| 200 | Successful response |
| 0 | Response body bytes sent |
| curl/8.18.0 | HTTP client and version |

This confirmed that the HTTP request reached Nginx and was handled successfully.

### Troubleshooting Lesson

Correlating an application request with its log entry helps distinguish network connectivity problems from application problems.

## 11. Stopping an Existing Container

Executed:

```bash
sudo docker stop lab-web
```

Docker returned:

```text
lab-web
```

Then executed:

```bash
sudo docker ps -a
```

Observed:

```text
lab-web    Exited (0)
```

### Explanation

- `docker stop`: Request a graceful shutdown.
- `docker ps -a`: Show running and stopped containers.
- `Exited (0)`: Container stopped with a successful exit code.

Stopping a container does not delete its image, configuration, or writable filesystem layer.

## 12. Restarting the Existing Container

Executed:

```bash
sudo docker start lab-web
```

Verified with:

```bash
sudo docker ps
```

Observed:

```text
lab-web    Up 32 seconds
```

### Important Difference

```bash
docker run
```

Creates a new container from an image and starts it.

```bash
docker start
```

Starts an existing stopped container.

The original Nginx container was reused.

## 13. Understanding Docker Restart Policies

Docker restart policies specify how Docker handles container shutdowns and daemon restarts.

| Policy | Behavior |
|---|---|
| no | No automatic restart |
| always | Restart automatically, including after daemon restarts |
| unless-stopped | Restart automatically unless intentionally stopped |
| on-failure | Restart on nonzero exit code, subject to configured limits |

### Inspecting the Original Policy

Executed:

```bash
sudo docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' lab-web
```

Result:

```text
no
```

This was the default because no restart policy had been specified when creating the container.

## 14. Configuring Automatic Recovery

Executed:

```bash
sudo docker update --restart unless-stopped lab-web
```

Docker returned:

```text
lab-web
```

### Command Explanation

| Component | Meaning |
|---|---|
| docker update | Change an existing container's configuration |
| --restart | Specify the restart policy |
| unless-stopped | Restart automatically unless manually stopped |
| lab-web | Container being updated |

This changed the existing container without recreating it.

### Verification

Executed:

```bash
sudo docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' lab-web
```

Result:

```text
unless-stopped
```

Verified that the container remained running:

```bash
sudo docker ps
```

Observed:

```text
lab-web    Up 12 minutes
```

**Lesson:** Make a configuration change, verify the resulting state, and document the outcome.

## 15. Understanding the Startup Sequence

The home lab has multiple levels of startup configuration.

```text
Physical Trigkey powers on
        |
        v
Proxmox VE boots
        |
        v
Ubuntu VM 100 starts
(Proxmox Start at boot enabled)
        |
        v
Ubuntu starts Docker Engine
(systemd docker.service enabled)
        |
        v
Docker starts lab-web
(restart policy: unless-stopped)
        |
        v
Nginx serves HTTP requests
```

These settings work together but control different components.

### Important Distinction

A reboot inside Ubuntu tests Docker and container recovery.

It does not test whether Proxmox will automatically start VM 100 after the physical host reboots.

The physical computer must also power on before Proxmox can start.

## 16. Testing Recovery After Ubuntu Reboot

With Nginx running and the restart policy configured, executed inside Ubuntu:

```bash
sudo reboot
```

The SSH session disconnected as expected.

After Ubuntu completed its restart, reconnected from Windows PowerShell:

```powershell
ssh ubuntu-lab
```

No commands were used to manually start Docker or Nginx.

### Purpose

Verify that the services recover automatically after an operating system restart.

## 17. Verifying Docker Service Recovery

Executed:

```bash
sudo systemctl status docker
```

Observed:

```text
Loaded: loaded (...; enabled; preset: enabled)
Active: active (running) since Fri 2026-11-20 18:42:01 UTC
Main PID: 1183 (dockerd)
```

Additional log entries included:

```text
Loading containers: done.
Daemon has completed initialization
Started docker.service - Docker Application Container Engine.
```

### Interpretation

| Output | Meaning |
|---|---|
| enabled | Configured for automatic startup |
| active (running) | Docker successfully started |
| Main PID: 1183 | Docker daemon process ID during this boot |
| Started docker.service | systemd confirmed successful startup |

### Related Commands

```bash
systemctl is-active docker
```

Checks whether Docker is currently active.

```bash
systemctl is-enabled docker
```

Checks whether Docker is configured for automatic startup.

```bash
sudo systemctl status docker
```

Displays broader service state and recent logs.

**Result:** Docker Engine recovered successfully after the Ubuntu reboot.

## 18. Verifying Automatic Container Recovery

Without manually starting any containers, executed:

```bash
sudo docker ps
```

Observed:

```text
lab-web    Up 12 minutes
```

The Nginx container was running after the Ubuntu reboot.

This demonstrated that Docker started the container automatically using its configured restart policy.

### What Was Proven?

- Ubuntu restarted successfully.
- Docker Engine automatically started.
- Docker loaded its existing container information.
- The `lab-web` container recovered without a manual start command.

### What Was Not Tested?

- A full Proxmox host reboot.
- Physical power-loss recovery.
- Recovery from corrupted filesystems or storage.
- Application health checking beyond the basic HTTP test.
- Restore from external backup media.

These are potential future tests.

## 19. Application Verification After Recovery

The application-level check introduced after reboot was:

```bash
curl -I http://172.17.0.2
```

The user reported the check was OK.

The purpose was to confirm HTTP functionality, not merely container runtime status.

If the request fails after a future reboot, first inspect the container's currently assigned IP address before assuming the service is broken:

```bash
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' lab-web
```

An internal Docker container address should not be treated as a permanent application endpoint.

## 20. Troubleshooting Workflow

If a Docker application appears unavailable, investigate before deleting or recreating it.

### First: Check Running Containers

```bash
sudo docker ps
```

### Second: Check All Containers

```bash
sudo docker ps -a
```

This distinguishes a stopped container from one that does not exist.

### Third: Review Application Logs

```bash
sudo docker logs --tail 10 lab-web
```

### Fourth: Inspect Container Configuration

```bash
sudo docker inspect lab-web
```

### Fifth: Check Docker Engine

```bash
sudo systemctl status docker
```

### Sixth: Test the Application

```bash
curl -I http://172.17.0.2
```

Use the current container IP address when performing this test.

### Troubleshooting Principle

**Observe → Identify → Investigate → Change → Verify → Document**

Do not immediately delete or recreate resources when something stops working.

## 21. Security Considerations

Docker commands were executed with `sudo`.

Membership in the Docker group can grant privileges comparable to root access.

The Nginx container was not configured with published host ports.

The home router was not configured with any port forwarding for Nginx.

This exercise did not intentionally expose the web server to the public internet.

Future tasks should include:

- Understanding Docker port publishing.
- Evaluating Ubuntu firewall rules.
- Creating user-defined Docker networks.
- Reviewing container privileges.
- Learning volumes and persistent application data.
- Using Docker Compose for service deployment.

## 22. Commands Learned

| Command | Purpose |
|---|---|
| `sudo docker logs hello-test-2` | View completed container output |
| `sudo docker run -d --name lab-web nginx:alpine` | Create a background Nginx container |
| `sudo docker ps` | List running containers |
| `sudo docker ps -a` | List all containers |
| `sudo docker network ls` | List Docker networks |
| `sudo docker inspect lab-web` | Inspect container configuration |
| `ip -4 addr show docker0` | Inspect Docker bridge IPv4 settings |
| `curl -I http://172.17.0.2` | Test Nginx using HTTP |
| `sudo docker logs --tail 10 lab-web` | Review recent Nginx logs |
| `sudo docker stop lab-web` | Stop Nginx |
| `sudo docker start lab-web` | Restart the existing container |
| `sudo docker update --restart unless-stopped lab-web` | Configure automatic restarts |
| `sudo reboot` | Reboot Ubuntu |
| `sudo systemctl status docker` | Verify Docker Engine status |

## 23. Completion Checklist

- [x] Read Hello World container logs.
- [x] Downloaded the Nginx Alpine image.
- [x] Created a named, detached Nginx container.
- [x] Verified the running container.
- [x] Inspected Docker's default networks.
- [x] Identified the Nginx container IP.
- [x] Inspected Docker's `docker0` interface.
- [x] Identified the Docker bridge subnet.
- [x] Successfully tested Nginx through HTTP.
- [x] Analyzed an Nginx HTTP access log.
- [x] Gracefully stopped Nginx.
- [x] Restarted the existing Nginx container.
- [x] Inspected its original restart policy.
- [x] Changed its policy to `unless-stopped`.
- [x] Verified the updated configuration.
- [x] Rebooted Ubuntu VM 100.
- [x] Verified Docker Engine automatically started.
- [x] Verified Nginx automatically restarted.
- [x] Reviewed application-level verification after reboot.
- [x] Practiced a systematic troubleshooting approach.

## 24. Next Learning Objectives

Continue Docker training with:

1. Docker port publishing and host-to-container connectivity.
2. Testing a containerized web application from another LAN device.
3. Understanding Docker firewall behavior.
4. User-defined bridge networks.
5. Docker volumes and persistent data.
6. Docker Compose.
7. Deploying useful home lab services.
8. Backup and restoration of containerized application data.

These topics will build toward real-world Linux administration, networking, automation, and cybersecurity experience.

---

## Final Result

Successfully deployed and managed a containerized Nginx web server on Ubuntu VM 100.

Verified Docker bridge networking, HTTP communication, application logs, container lifecycle commands, and automatic recovery after an operating system reboot.

**The lab now demonstrates practical Docker deployment, service verification, and recovery testing.**

