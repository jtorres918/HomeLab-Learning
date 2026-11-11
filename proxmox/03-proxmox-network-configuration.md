# 03 - Proxmox Network Configuration

**Project:** HomeLab-Learning  
**Platform:** Proxmox Virtual Environment  
**Physical Server:** TRIGKEY Mini PC  
**Router:** NETGEAR Nighthawk RAX70  
**Status:** Initial network configuration completed

---

## 1. Objective

Configure the Proxmox host to communicate with the existing home network while maintaining a consistent management IP address.

The goals were to:

- Connect Proxmox to the home LAN.
- Understand basic network addressing.
- Configure a static IP address for the Proxmox host.
- Understand the purpose of the default gateway and DNS server.
- Configure the router's DHCP address pool.
- Reserve an IP address range for manually configured devices.
- Understand the Proxmox virtual network bridge.
- Verify remote access to the Proxmox web interface.
- Avoid disrupting the existing home network.

The Proxmox installation was intended to operate alongside existing household devices without replacing the NETGEAR router or interfering with its internet connection.

---

## 2. Physical Network Environment

The home network uses a NETGEAR Nighthawk RAX70 router.

The TRIGKEY mini PC runs Proxmox VE directly on its internal SSD.

Proxmox connects to the home network using an Ethernet interface.

### Network Diagram

```text
                  INTERNET
                     |
                     |
             NETGEAR RAX70
               192.168.1.1
                     |
                Home LAN
                     |
              Ethernet Cable
                     |
             TRIGKEY Mini PC
              PROXMOX VE
             192.168.1.200
                     |
               vmbr0 Bridge
                     |
              Virtual Machines
                     |
               Ubuntu VM 100
              192.168.1.17
```

The Ubuntu VM was added after the Proxmox network was configured.

The diagram represents network connectivity. Devices on the same LAN can exchange traffic without being routed through the default gateway.

---

## 3. IP Address Planning

The existing home network uses the following subnet:

`192.168.1.0/24`

### Network Addressing

| Setting | Value |
|---|---|
| Network address | 192.168.1.0 |
| Subnet mask | 255.255.255.0 |
| CIDR prefix | /24 |
| Default gateway | 192.168.1.1 |
| Router | NETGEAR RAX70 |
| Proxmox management IP | 192.168.1.200 |
| Broadcast address | 192.168.1.255 |

### Understanding CIDR

CIDR stands for Classless Inter-Domain Routing.

The `/24` prefix indicates that the first 24 bits of the IPv4 address represent the network portion.

A `/24` subnet provides 256 total IPv4 addresses, including the network and broadcast addresses.

For this subnet, the usable host address range is:

`192.168.1.1 - 192.168.1.254`

The router uses `192.168.1.1`.

The Proxmox server uses `192.168.1.200`.

### Why This Matters

Devices within the same subnet can normally communicate directly across the local network.

Devices communicating with destinations outside the subnet generally use their default gateway.

---

## 4. Configure the NETGEAR DHCP Address Range

The NETGEAR router provides DHCP services to devices on the home LAN.

### What is DHCP?

DHCP stands for Dynamic Host Configuration Protocol.

It automatically provides network settings, including:

- IP address
- Subnet mask
- Default gateway
- DNS server information
- Lease duration

### Original DHCP Range

The original router DHCP pool was:

`192.168.1.2 - 192.168.1.254`

This allowed the router to dynamically assign addresses across nearly the entire usable subnet.

### Modified DHCP Range

The DHCP pool was changed to:

`192.168.1.2 - 192.168.1.199`

This leaves the following range outside the DHCP pool:

`192.168.1.200 - 192.168.1.254`

### Address Planning

| IP Address Range | Intended Purpose |
|---|---|
| 192.168.1.1 | NETGEAR router |
| 192.168.1.2 - 192.168.1.199 | DHCP-assigned client devices |
| 192.168.1.200 - 192.168.1.254 | Manually managed static addresses |

The Proxmox host was assigned `192.168.1.200`.

### Why Change the DHCP Pool?

Manually configuring a static IP address within an active DHCP pool can create a risk of address conflicts.

Separating the dynamic DHCP pool from the manually assigned address range makes network administration easier.

### Important Considerations

Removing an address range from the DHCP pool does not automatically guarantee that every address in that range is unused.

Existing reservations, static assignments, and active devices should be checked before assigning new static addresses.

A DHCP reservation is also different from manually configuring a static IP on a device.

---

## 5. Proxmox Static IP Configuration

The Proxmox host was configured with a static management IP during installation.

### Configuration Values

| Setting | Value |
|---|---|
| Hostname / FQDN | pve.homelab.internal |
| Static IPv4 address | 192.168.1.200 |
| Prefix length | /24 |
| Subnet mask | 255.255.255.0 |
| Default gateway | 192.168.1.1 |
| DNS server | 192.168.1.1 |
| Primary network bridge | vmbr0 |
| Physical Ethernet interface | nic0 |

### What is a Static IP?

A static IP address is manually configured instead of being obtained dynamically from DHCP.

It normally remains consistent until an administrator changes the configuration.

### Why Use a Static IP for Proxmox?

Proxmox is a management server.

A consistent address is useful for:

- Opening the Proxmox web interface.
- Connecting through SSH.
- Managing virtual machines.
- Configuring monitoring tools.
- Planning future network services.
- Maintaining consistent documentation.

The selected management address is:

`192.168.1.200`

---

## 6. Understanding the Default Gateway

The Proxmox default gateway is:

`192.168.1.1`

This is the NETGEAR RAX70 router.

### What Does a Default Gateway Do?

The default gateway is used when the computer needs to communicate with a destination for which it has no more-specific route.

For example, when Proxmox needs to access an internet software repository, the traffic is normally forwarded through the NETGEAR router.

### Local vs Remote Traffic

**Local destination:**

Proxmox `192.168.1.200` communicates with Ubuntu `192.168.1.17`.

Both are on `192.168.1.0/24`.

No default gateway is required for the IP routing decision.

**Remote destination:**

Proxmox communicates with an internet server.

The destination is outside `192.168.1.0/24`.

Proxmox sends the traffic toward its default gateway at `192.168.1.1`.

### Key Concept

A default gateway is a network role, not necessarily a particular hardware model.

In this home lab, the NETGEAR router performs that role.

---

## 7. Understanding DNS

The Proxmox DNS server was configured as:

`192.168.1.1`

### What is DNS?

DNS stands for Domain Name System.

DNS allows devices to resolve hostnames into IP addresses.

For example:

`us.archive.ubuntu.com`

Can be resolved to an IP address so the system can communicate with the remote service.

### Why Use the Router as DNS?

The NETGEAR router provides a DNS resolution or forwarding service for devices on the home LAN.

This allows Proxmox to use the router as its configured DNS resolver.

### Important Distinction

DNS resolution and internet connectivity are separate concepts.

A hostname may successfully resolve even when the destination does not respond to a network request.

DNS success does not guarantee that HTTP, HTTPS, or ICMP communication will succeed.

---

## 8. Understanding the Proxmox Network Bridge

The Proxmox host uses a virtual network bridge named:

`vmbr0`

### What is a Network Bridge?

A network bridge connects network interfaces at Layer 2 of the OSI model.

It can forward Ethernet frames between connected interfaces based on MAC addresses.

### Physical vs Virtual Interfaces

**nic0**

The physical Ethernet interface detected by Proxmox.

**vmbr0**

A Linux virtual bridge connecting the Proxmox host and virtual machine network adapters to the LAN.

### Example

```text
         Home Network
              |
              |
       Physical NIC (nic0)
              |
              |
       Proxmox Bridge
            vmbr0
              |
              |
       Virtual NIC (VirtIO)
              |
              |
       Ubuntu VM 100
```

The Ubuntu VM uses its own network interface and IP address even though it shares the host's physical Ethernet connectivity.

### Why a Bridge?

Using `vmbr0` allows virtual machines to participate in the home LAN.

The NETGEAR router can assign DHCP addresses to bridged virtual machines.

This is why Ubuntu later received:

`192.168.1.17`

While the Proxmox host uses:

`192.168.1.200`

---

## 9. Access Proxmox from Windows

After completing the installation and network configuration, the Proxmox web management interface was accessed from a Windows computer.

### Management URL

```text
https://192.168.1.200:8006
```

### URL Components

| Component | Meaning |
|---|---|
| https | Encrypted HTTP connection |
| 192.168.1.200 | Proxmox management IP |
| 8006 | Proxmox web management TCP port |

### Initial Browser Certificate Warning

The browser displayed a certificate warning when first connecting to Proxmox.

This is expected when Proxmox uses a self-signed certificate that the browser does not already trust.

The warning does not necessarily indicate an attack, but the certificate identity should be verified before trusting a connection.

A trusted certificate can be configured later.

### Result

The Proxmox management interface was accessible from another computer on the LAN.

---

## 10. Verify Network Connectivity

Networking was validated through access to the Proxmox web interface and by testing the Ubuntu repository during a later troubleshooting exercise.

### DNS and ICMP Test

From the Proxmox shell:

```bash
ping -c 4 us.archive.ubuntu.com
```

Result:

- The hostname resolved to `91.189.91.83`.
- Four ICMP requests were sent.
- No replies were received.

This demonstrated successful hostname resolution but not successful ICMP communication.

### HTTP Test

From the Proxmox shell:

```bash
curl -I --connect-timeout 10 --max-time 20 http://us.archive.ubuntu.com/ubuntu/
```

Result:

`HTTP/1.1 200 OK`

This confirmed that the Proxmox host could communicate with the Ubuntu repository over HTTP during that test.

### Lessons

- Ping uses ICMP.
- HTTP commonly uses TCP port 80.
- HTTPS commonly uses TCP port 443.
- Some services do not respond to ICMP.
- DNS resolution is not the same as application connectivity.
- A failed ping does not automatically mean a server is unreachable.

---

## 11. Useful Proxmox Networking Commands

These commands can be used for future inspection and troubleshooting.

### Display IP Addresses

```bash
ip -4 addr
```

Displays IPv4 addresses assigned to the system's network interfaces.

### Display Routing Table

```bash
ip route
```

Displays the IPv4 routes Linux uses to determine where packets should be forwarded.

### Display Interfaces

```bash
ip link show
```

Displays network interface information, including interface state and MAC addresses.

### Test the Default Gateway

```bash
ping -c 4 192.168.1.1
```

Tests ICMP communication with the NETGEAR router.

### Test External IPv4 Connectivity

```bash
ping -c 4 1.1.1.1
```

Tests ICMP connectivity to an external public IP address without requiring a DNS lookup.

### Inspect Network Configuration

```bash
cat /etc/network/interfaces
```

Displays the Proxmox network configuration file.

**Important:** This is a read-only command.

Incorrectly changing Proxmox network configuration can interrupt access to the server.

### Understanding `/etc/network/interfaces`

On this Proxmox installation, networking is managed using the Linux interfaces configuration system.

The file defines network settings such as:

- Physical interfaces
- Virtual bridges
- Static addresses
- Default gateway
- Bridge ports

The exact contents should be captured and verified before any configuration changes.

---

## 12. Protecting the Existing Home Network

An important requirement of this lab was avoiding disruptions to household internet connectivity.

The following decisions helped reduce that risk:

- Kept the NETGEAR router as the existing home gateway.
- Did not replace the router with Proxmox.
- Did not configure a separate DHCP server on Proxmox.
- Assigned Proxmox a static address outside the revised DHCP pool.
- Used an existing LAN subnet rather than introducing new routing immediately.
- Connected virtual machines through the existing `vmbr0` bridge.
- Did not configure VLANs or additional firewall routing during initial setup.

### Why This Matters

Misconfigured DHCP servers, duplicate addresses, or incorrect gateway settings can disrupt normal network communication.

The initial lab configuration prioritized reliable access and basic networking knowledge.

More advanced isolation and segmentation can be introduced in later projects.

---

## 13. Networking Concepts Learned

### LAN

Local Area Network.

A network connecting devices within a limited area, such as a home or office.

### WAN

Wide Area Network.

A network spanning a larger geographic area.

A home router typically connects the home LAN to an internet service provider's network through its WAN connection.

### IP Address

A logical address used to identify a network interface and support network-layer communication.

### Subnet

A defined portion of an IP network.

### Subnet Mask

Identifies which bits of an IP address represent the network prefix.

### DHCP

Automatically provides network configuration to clients.

### Static IP

An address configured manually rather than obtained through DHCP.

### Default Gateway

A router used to reach destinations without a more-specific route.

### DNS

Resolves domain names into IP addresses and supports other naming records.

### Network Bridge

A Layer 2 mechanism used to connect network interfaces.

### MAC Address

A link-layer address associated with a network interface.

### Routing

The process of selecting where IP packets should be forwarded.

---

## 14. Verification Checklist

- [x] Proxmox installed on TRIGKEY.
- [x] Physical Ethernet networking established.
- [x] NETGEAR RAX70 identified as default gateway.
- [x] Home subnet identified as `192.168.1.0/24`.
- [x] Router DHCP pool adjusted to `.2 - .199`.
- [x] Static address range planned for `.200 - .254`.
- [x] Proxmox configured with static IP `192.168.1.200`.
- [x] Default gateway configured as `192.168.1.1`.
- [x] DNS server configured as `192.168.1.1`.
- [x] `vmbr0` virtual bridge configured.
- [x] Proxmox web interface accessed over LAN.
- [x] Proxmox DNS resolution tested.
- [x] HTTP connectivity to Ubuntu mirror verified.
- [ ] DHCP reservation for Ubuntu VM.
- [ ] VLAN configuration and segmentation.
- [ ] Dedicated firewall lab.
- [ ] Review and document complete `/etc/network/interfaces` configuration.

---

## 15. Future Networking Plans

The current setup uses the existing home LAN.

Future home lab projects may include:

1. Reserving a consistent IP address for Ubuntu VM 100.
2. Learning ARP and MAC address resolution.
3. Configuring VLANs and separate virtual networks.
4. Deploying a virtual firewall such as OPNsense.
5. Learning NAT and routing between networks.
6. Configuring firewall rules and access controls.
7. Adding network monitoring and logging.
8. Developing a more isolated network security testing environment.

Any changes affecting the physical router or primary LAN will be planned carefully to avoid disrupting the household network.

---

## 16. Final Outcome

Proxmox was successfully integrated into the existing home network.

The host uses a static management IP of `192.168.1.200/24` and connects through the NETGEAR Nighthawk RAX70 router at `192.168.1.1`.

The router's DHCP pool was reduced to `192.168.1.2 - 192.168.1.199`, leaving an address range for manually configured static devices.

The Proxmox bridge `vmbr0` allows virtual machines to participate in the existing LAN.

The Proxmox web management interface is accessible from a Windows computer using HTTPS on TCP port 8006.

**Status: Initial Proxmox network configuration successfully completed.**

---

**Next Phase:** Ubuntu Server deployment, networking, SSH administration, and security hardening.
