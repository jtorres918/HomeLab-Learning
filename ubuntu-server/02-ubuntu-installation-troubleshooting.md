# 02 - Ubuntu Server Installation Troubleshooting

**Date:** November 9, 2026  
**Project:** HomeLab-Learning  
**Hypervisor:** Proxmox VE 9.2.26  
**Virtual Machine:** VM 100 - ubuntu-server  
**Operating System:** Ubuntu Server 26.04.1 LTS  
**Issue:** Ubuntu installation stalled during Linux kernel package download  
**Status:** Resolved - Installation completed successfully

---

## 1. Objective

During the installation of Ubuntu Server 26.04.1 LTS on Proxmox VM 100, the installer appeared to stop progressing while downloading a Linux kernel package.

The objective of this troubleshooting exercise was to identify whether the problem originated from:

- Proxmox host networking
- Ubuntu virtual machine networking
- DNS resolution
- Local router connectivity
- Internet connectivity
- HTTP or HTTPS communication
- Ubuntu package repository availability
- Virtual machine resource limitations

Rather than immediately restarting or reinstalling Ubuntu, I performed diagnostic tests to narrow down the possible causes.

This exercise provided practical experience with Linux networking, TCP/IP, command-line troubleshooting, and systematic problem isolation.

---

## 2. Home Lab Environment

### Physical and Virtual Infrastructure

| Component | Configuration |
|---|---|
| Physical host | Trigkey Mini PC |
| Hypervisor | Proxmox VE 9.2.26 |
| Proxmox hostname | pve |
| Proxmox IP | 192.168.1.200 |
| Ubuntu VM | VM 100 |
| VM hostname | ubuntu-server |
| Ubuntu IP | 192.168.1.17 |
| Network bridge | vmbr0 |
| Ubuntu interface | ens18 |
| Default gateway | 192.168.1.1 |
| Router | Netgear Nighthawk RAX70 |
| Subnet | 192.168.1.0/24 |

### Virtual Machine Resources

| Resource | Allocation |
|---|---|
| vCPUs | 2 |
| Memory | 2 GiB |
| Virtual disk | 32 GiB |
| Storage | local-lvm |
| Network adapter | VirtIO |

The Proxmox host and Ubuntu VM share the physical network connection, but each has its own network configuration.

This distinction became important during troubleshooting.

---

## 3. Initial Problem

During the Ubuntu installation, the installer stopped showing progress while downloading the Linux kernel.

The last visible package download was:

```text
Get:4 http://us.archive.ubuntu.com/ubuntu ...
linux-modules-7.0.0-43-generic [170 MB]
```

The installer remained on this download for an extended period.

The installation had already completed several other tasks, including configuring storage, networking, and package repositories.

### Initial Observations

Proxmox continued reporting that VM 100 was running.

At one point, the VM summary showed approximately:

| Metric | Observation |
|---|---|
| CPU utilization | 3.55% |
| Memory utilization | 88.58% |
| Allocated RAM | 2 GiB |
| Virtual disk | 32 GiB |
| VM status | Running |

Although memory utilization was relatively high, this did not establish that the virtual machine had run out of memory.

The next step was to investigate connectivity.

---

## 4. Troubleshooting Strategy

I used a layered troubleshooting approach.

The basic sequence was:

1. Verify DNS resolution from Proxmox.
2. Test Proxmox HTTP connectivity.
3. Access a second terminal inside the Ubuntu installer.
4. Test HTTP connectivity from Ubuntu.
5. Test communication with the local router.
6. Test connectivity to a public IP address.
7. Force HTTP traffic to IPv4.
8. Test connectivity over HTTPS.
9. Test the Ubuntu repository using HTTPS.
10. Recheck installation progress.

Testing from both Proxmox and Ubuntu was important because successful connectivity from the host does not guarantee successful connectivity from a guest VM.

---

## 5. Test 1 - DNS Resolution and ICMP Connectivity from Proxmox

Opened the Proxmox Shell:

`Datacenter → pve → Shell`

Executed:

```bash
ping -c 4 us.archive.ubuntu.com
```

### Command Explanation

| Component | Purpose |
|---|---|
| ping | Sends ICMP Echo Requests |
| -c | Specifies how many requests to send |
| 4 | Sends four requests |
| us.archive.ubuntu.com | Ubuntu package mirror hostname |

### Actual Result

```text
PING us.archive.ubuntu.com (91.189.91.83) 56(84) bytes of data.

4 packets transmitted, 0 received, 100% packet loss
```

### Analysis

The hostname successfully resolved to:

`91.189.91.83`

This demonstrated that DNS resolution was functioning on the Proxmox host.

However, the server did not return ICMP Echo Replies.

This did not necessarily mean the Ubuntu repository was unreachable.

Some servers and network devices filter or ignore ICMP traffic.

### Lesson Learned

**Successful DNS resolution does not guarantee successful network communication.**

Also, **a failed ping does not necessarily indicate a failed TCP or HTTP connection.**

---

## 6. Test 2 - HTTP Connectivity from Proxmox

Because the Ubuntu mirror did not respond to ping, I tested its actual HTTP service.

Executed on Proxmox:

```bash
curl -I --connect-timeout 10 --max-time 20 http://us.archive.ubuntu.com/ubuntu/
```

### Command Explanation

| Component | Purpose |
|---|---|
| curl | Transfers data using supported network protocols |
| -I | Requests HTTP response headers |
| --connect-timeout 10 | Limits connection establishment to 10 seconds |
| --max-time 20 | Limits the entire request to 20 seconds |
| http:// | Uses HTTP |
| us.archive.ubuntu.com | Ubuntu archive mirror |

### Actual Result

```text
HTTP/1.1 200 OK
Server: Apache/2.4.52 (Ubuntu)
Content-Type: text/html;charset=UTF-8
```

### Analysis

HTTP status code `200 OK` indicates that the server successfully handled the request.

This confirmed that the Proxmox host could communicate with the Ubuntu repository using HTTP.

### Important Finding

The earlier failed ping did not indicate that the repository's HTTP service was unavailable.

### Lesson Learned

When troubleshooting a specific service, testing its actual application protocol is generally more informative than relying exclusively on ping.

---

## 7. Test 3 - Access a Second Ubuntu Terminal

The Ubuntu installer was still running.

To avoid terminating the installation, I opened a second Linux virtual terminal.

### Command / Keyboard Shortcut

```text
Ctrl + Alt + F2
```

This switched from the installer interface to a separate terminal.

The Ubuntu environment displayed a shell prompt.

### Network Information

The terminal displayed:

```text
IPv4 address for ens18: 192.168.1.17
```

The interface had received an IP address through DHCP.

### Why This Was Useful

Linux supports multiple virtual terminals.

This allowed troubleshooting commands to be executed while leaving the installation process running in another terminal.

The installer could be accessed again using:

```text
Ctrl + Alt + F1
```

### Lesson Learned

A Linux installation environment may provide additional terminals for troubleshooting without requiring a restart.

The exact shortcut and available consoles can vary by distribution and installer environment.

---

## 8. Test 4 - Ubuntu HTTP Connectivity

The next step was to run the same HTTP test from inside the Ubuntu VM.

Executed:

```bash
curl -I --connect-timeout 10 --max-time 20 http://us.archive.ubuntu.com/ubuntu/
```

### Actual Result

```text
curl: (28) Operation timed out after 20002 milliseconds with 0 bytes received
```

### Analysis

This result was different from the Proxmox host.

Proxmox returned:

```text
HTTP/1.1 200 OK
```

Ubuntu returned:

```text
curl: (28) Operation timed out
```

This indicated that the VM could not complete the same HTTP request within the configured timeout.

### Important Finding

The host and VM had different HTTP connectivity results, despite being connected to the same home network.

The failure could have involved guest networking, packet filtering, routing, the remote server connection, or another network-path issue.

The precise cause was not yet established.

### Lesson Learned

**Always test connectivity from the system experiencing the problem.**

Testing only from the hypervisor can miss issues specific to a virtual machine.

---

## 9. Test 5 - Local Router Connectivity

I tested whether Ubuntu could communicate with the Netgear router.

Executed inside Ubuntu:

```bash
ping -c 4 192.168.1.1
```

### Actual Result

```text
4 packets transmitted, 4 received, 0% packet loss
```

Response times were approximately:

```text
min/avg/max = 3.670/4.903/6.762 ms
```

### Analysis

Ubuntu successfully communicated with the router.

This established that basic connectivity through the virtual network interface and home LAN was functioning.

### Network Path

```text
Ubuntu VM (192.168.1.17)
        |
        v
Proxmox Virtual Bridge (vmbr0)
        |
        v
Physical Network Connection
        |
        v
Netgear Router (192.168.1.1)
```

### Lesson Learned

Testing the default gateway is a useful early step in troubleshooting connectivity.

If a machine cannot communicate with its gateway, the problem may involve interface configuration, VLAN membership, local switching, routing, or filtering.

---

## 10. Test 6 - External Internet Connectivity

The next step was to determine whether Ubuntu could communicate beyond the local network.

Executed:

```bash
ping -c 4 1.1.1.1
```

### What is 1.1.1.1?

`1.1.1.1` is a public DNS service operated by Cloudflare.

Using a numerical IP address avoids requiring DNS resolution for the ping destination.

### Actual Result

```text
4 packets transmitted, 4 received, 0% packet loss
```

Response times were approximately:

```text
min/avg/max = 6.321/6.727/6.977 ms
```

### Analysis

Ubuntu successfully reached an external public IP address.

This confirmed basic outbound IPv4 connectivity.

The virtual NIC, gateway, and path to at least one external destination were functioning.

### Lesson Learned

A system may have working internet connectivity while still being unable to access a particular service.

ICMP connectivity alone does not confirm that TCP connections or package downloads will work.

---

## 11. Test 7 - Force HTTP Traffic to IPv4

One possible explanation was that the VM might have difficulty communicating over IPv6.

To investigate, I forced curl to use IPv4.

Executed:

```bash
curl -4 -I --connect-timeout 10 --max-time 20 http://us.archive.ubuntu.com/ubuntu/
```

### Command Explanation

The additional option:

```text
-4
```

Tells curl to use IPv4 only.

### Actual Result

```text
curl: (28) Operation timed out after 20002 milliseconds with 0 bytes received
```

### Analysis

The HTTP connection still timed out when IPv4 was explicitly selected.

This showed that simply forcing IPv4 did not resolve the problem.

### Lesson Learned

IPv4 and IPv6 can experience different connectivity problems.

Tools such as curl provide options for explicitly testing one address family.

However, a failed IPv4 test alone does not identify the exact cause of a connection problem.

---

## 12. Test 8 - HTTPS Connectivity to Google

I then tested HTTPS to a different internet destination.

Executed:

```bash
curl -4 -I --connect-timeout 10 --max-time 20 https://www.google.com
```

### Actual Result

```text
HTTP/2 301
location: https://www.google.com/
```

### Analysis

HTTP status code `301` indicates a permanent redirect.

Although it is not a `200 OK` response, it confirms that the remote HTTPS service responded.

This demonstrated that Ubuntu could establish HTTPS communication with Google.

### HTTP vs HTTPS

| Protocol | Typical Port | Encryption |
|---|---|---|
| HTTP | TCP 80 | No transport encryption |
| HTTPS | TCP 443 | TLS encryption |

### Lesson Learned

A system can successfully communicate using HTTPS while having difficulties with HTTP connections.

Testing different protocols, ports, and destinations helps isolate service-specific connectivity problems.

---

## 13. Test 9 - Ubuntu Mirror Over HTTPS

Since HTTPS connectivity to Google worked, I tested the Ubuntu repository using HTTPS.

Executed:

```bash
curl -4 -I --connect-timeout 10 --max-time 20 https://us.archive.ubuntu.com/ubuntu/
```

### Actual Result

```text
HTTP/1.1 200 OK
Server: Apache/2.4.52 (Ubuntu)
Content-Type: text/html;charset=UTF-8
```

### Analysis

Ubuntu successfully connected to the same repository over HTTPS.

The tests established that:

- The Proxmox host could reach the mirror over HTTP.
- The Ubuntu VM's HTTP request to the mirror timed out.
- The Ubuntu VM could reach Google over HTTPS.
- The Ubuntu VM could reach the Ubuntu mirror over HTTPS.

### Key Finding

The connectivity issue appeared to be related to HTTP requests from the Ubuntu VM to the Ubuntu archive during the installation.

However, the exact cause was not conclusively identified.

No packet capture or firewall log analysis was performed, so a specific firewall or network-filtering cause cannot be confirmed.

### Lesson Learned

Changing only one variable at a time—such as the protocol or destination—helps narrow down network problems.

---

## 14. Troubleshooting Results Summary

| Test | System | Result |
|---|---|---|
| DNS lookup through ping | Proxmox | Resolved |
| Ping Ubuntu mirror | Proxmox | 100% packet loss |
| HTTP Ubuntu mirror | Proxmox | 200 OK |
| HTTP Ubuntu mirror | Ubuntu VM | Timeout |
| Ping local router | Ubuntu VM | 0% packet loss |
| Ping 1.1.1.1 | Ubuntu VM | 0% packet loss |
| HTTP mirror using IPv4 | Ubuntu VM | Timeout |
| HTTPS Google | Ubuntu VM | 301 redirect |
| HTTPS Ubuntu mirror | Ubuntu VM | 200 OK |

### Conclusions

DNS resolution and basic network connectivity were functioning.

The Ubuntu VM could communicate with its default gateway and an external public IP address.

HTTPS requests succeeded.

HTTP requests to the Ubuntu mirror timed out during testing.

The observed failure was more specific than a complete loss of internet connectivity.

---

## 15. Installation Recovery

After completing the diagnostic tests, I returned to the Ubuntu installer using:

```text
Ctrl + Alt + F1
```

The installer had progressed beyond the previously stalled package download.

It had completed additional installation steps, including:

- Installing the Linux kernel
- Configuring the installed system
- Configuring APT
- Configuring network settings
- Installing GRUB
- Installing OpenSSH Server
- Configuring cloud-init
- Processing security updates

The installation screen eventually displayed:

```text
Reboot Now
```

### Resolution

The installation completed successfully without manually changing the package mirror, restarting the VM, or modifying Proxmox networking.

The exact reason the stalled download resumed was not established.

### Final Status

**Resolved: Ubuntu Server installed successfully.**

---

## 16. Verify Ubuntu After Reboot

Selected:

```text
Reboot Now
```

The VM rebooted into its installed Ubuntu Server operating system.

Successfully logged in with the user account created during installation.

The welcome screen confirmed:

```text
Ubuntu 26.04.1 LTS
Linux 7.0.0-43-generic
```

The network interface continued using:

```text
192.168.1.17
```

The operating system was functioning.

Subsequent package updates also completed successfully.

---

## 17. Commands Used During Troubleshooting

### Ping the Ubuntu Mirror

```bash
ping -c 4 us.archive.ubuntu.com
```

Tests ICMP connectivity while also showing whether the hostname resolves.

### Test HTTP Headers

```bash
curl -I --connect-timeout 10 --max-time 20 http://us.archive.ubuntu.com/ubuntu/
```

Tests whether the Ubuntu repository responds over HTTP.

### Test Local Router Connectivity

```bash
ping -c 4 192.168.1.1
```

Checks communication with the default gateway.

### Test External IPv4 Connectivity

```bash
ping -c 4 1.1.1.1
```

Checks basic connectivity to an external public address.

### Force IPv4 HTTP

```bash
curl -4 -I --connect-timeout 10 --max-time 20 http://us.archive.ubuntu.com/ubuntu/
```

Tests the repository using HTTP and IPv4.

### Test External HTTPS

```bash
curl -4 -I --connect-timeout 10 --max-time 20 https://www.google.com
```

Checks HTTPS connectivity to a different external service.

### Test Ubuntu Mirror Over HTTPS

```bash
curl -4 -I --connect-timeout 10 --max-time 20 https://us.archive.ubuntu.com/ubuntu/
```

Tests whether the Ubuntu repository responds over HTTPS.

---

## 18. Important Networking Concepts Learned

### DNS

DNS stands for Domain Name System.

DNS resolves hostnames, such as:

`us.archive.ubuntu.com`

Into IP addresses.

DNS resolution is separate from actually establishing a connection to the destination service.

### ICMP

ICMP stands for Internet Control Message Protocol.

The `ping` utility commonly uses ICMP Echo Request and Echo Reply messages.

Ping is useful for diagnosing connectivity but does not test whether a website or TCP service is available.

### TCP

TCP stands for Transmission Control Protocol.

TCP provides connection-oriented communication and supports reliable delivery of ordered data.

HTTP and HTTPS commonly operate over TCP, although HTTP/3 uses QUIC over UDP.

### HTTP

HTTP stands for Hypertext Transfer Protocol.

It is used to exchange requests and responses between clients and web servers.

Ubuntu software repositories may be accessed over HTTP or HTTPS, depending on configuration.

### HTTPS

HTTPS is HTTP protected using TLS.

TLS encrypts communications and provides authentication of the server when certificates are properly validated.

### Default Gateway

A default gateway forwards traffic destined for networks outside the local subnet.

In this home lab:

`192.168.1.1`

Is the Netgear router and default gateway.

### Network Bridge

A network bridge connects network interfaces at Layer 2.

The Proxmox bridge `vmbr0` connects the Ubuntu VM's network interface to the bridged home network.

### Virtual Network Interface

Ubuntu identifies its virtual Ethernet interface as:

`ens18`

The interface is separate from the physical host's network configuration.

---

## 19. Troubleshooting Lessons

### Lesson 1 - Do Not Assume Ping Failure Means Internet Failure

The Ubuntu mirror did not respond to ICMP requests from Proxmox.

However, an HTTP test from Proxmox succeeded.

This demonstrated the importance of understanding protocol differences.

### Lesson 2 - Test From the Affected System

Proxmox could communicate with the Ubuntu repository over HTTP.

Ubuntu could not complete the same request during testing.

Testing exclusively from Proxmox would have overlooked this difference.

### Lesson 3 - Verify Local Connectivity First

Successful communication with the router confirmed that the VM was connected to the local network.

### Lesson 4 - Test External IP Connectivity Separately

Successful communication with `1.1.1.1` established basic connectivity beyond the LAN.

### Lesson 5 - Different Services May Behave Differently

HTTP to the Ubuntu mirror timed out, while HTTPS succeeded.

Network troubleshooting should account for differences in protocols, ports, and destinations.

### Lesson 6 - Avoid Premature Reinstallation

The installer eventually recovered without being restarted.

Reinstalling immediately would have interrupted the existing installation and potentially hidden the original problem.

### Lesson 7 - Document Actual Evidence

It is important to distinguish between:

- An observed symptom
- A diagnostic result
- A possible explanation
- A confirmed root cause

The actual root cause of this incident was not conclusively determined.

### Lesson 8 - Troubleshooting Is a Repeatable Process

The diagnostic workflow can be reused for future Linux, cloud, and networking problems.

A useful troubleshooting sequence is:

**Interface → Gateway → External IP → DNS → Application Protocol → Logs**

---

## 20. Potential Future Troubleshooting Tools

As the home lab progresses, additional commands will be studied.

| Tool | Purpose |
|---|---|
| `ip addr` | Inspect network interfaces and addresses |
| `ip route` | Inspect routing tables |
| `ss` | Inspect sockets and network connections |
| `dig` | Query DNS records |
| `traceroute` | Investigate network paths |
| `tcpdump` | Capture and analyze packets |
| `journalctl` | Inspect systemd service logs |
| `curl -v` | Display more detailed connection information |
| `openssl s_client` | Investigate TLS connections |

These tools will be useful in later network administration and cybersecurity projects.

---

## 21. Resolution Checklist

- [x] Identified stalled Linux kernel download.
- [x] Confirmed VM remained running.
- [x] Verified Proxmox DNS resolution.
- [x] Tested Proxmox ICMP connectivity.
- [x] Verified Proxmox HTTP connectivity.
- [x] Opened a second Ubuntu terminal.
- [x] Tested Ubuntu HTTP connectivity.
- [x] Verified local gateway connectivity.
- [x] Verified external IPv4 connectivity.
- [x] Tested IPv4-only HTTP.
- [x] Verified external HTTPS connectivity.
- [x] Verified Ubuntu mirror HTTPS connectivity.
- [x] Returned to installer.
- [x] Confirmed installation progressed.
- [x] Completed Ubuntu installation.
- [x] Rebooted Ubuntu successfully.
- [x] Verified Ubuntu login and networking.

---

## 22. Final Outcome

The Ubuntu Server installation stalled during a Linux kernel package download.

Systematic testing established that the VM had working local networking, basic external IPv4 connectivity, and HTTPS connectivity.

HTTP connectivity to the Ubuntu archive timed out during testing, while the same destination responded successfully over HTTPS.

The installation ultimately recovered and completed without manual changes to repository configuration or virtual hardware.

**Final Status: Resolved — Ubuntu Server 26.04.1 LTS installed and operational.**

### Skills Demonstrated

- Linux command-line troubleshooting
- Virtual machine network diagnostics
- DNS troubleshooting
- ICMP testing
- HTTP and HTTPS connectivity testing
- TCP/IP fundamentals
- Hypervisor versus guest network isolation
- Evidence-based troubleshooting
- Technical documentation
- Root cause analysis with documented uncertainty

---

**Next Document:** `03-ubuntu-networking-and-updates.md`

This will cover Ubuntu networking commands, APT updates, Ubuntu Pro security services, and operating system verification.
