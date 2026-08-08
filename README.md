# Day 01 — Basic Routing, ARP & ICMP Analysis

## Objective

The objective of this lab is to build a small routed network and understand how devices communicate across different IP networks.

In this lab, we will:

* Configure IP addresses, subnet masks, and default gateways on two Linux PCs.
* Configure two Gigabit interfaces on the router.
* Connect two LANs through the router.
* Verify connectivity between PC1 and PC2 using ICMP.
* Observe ARP request/reply messages in Wireshark.
* Analyze ICMP Echo Request and Echo Reply packets.

## Topology

The lab uses the smallest network required to demonstrate communication between two different networks.

```text
PC1 ─── SW1 ─── R1 ─── SW2 ─── PC2
```

### Devices

* 2 × IOSvL2 Switches
* 1 × Router
* 2 × Linux Hosts

### IP Addressing

| Device | Interface | IP Address    | Subnet Mask         | Default Gateway |
| ------ | --------- | ------------- | ------------------- | --------------- |
| PC1    | eth0      | 192.168.10.10 | 255.255.255.0 (/24) | 192.168.10.1    |
| R1     | e0/0      | 192.168.10.1  | 255.255.255.0 (/24) | —               |
| R1     | e0/1      | 192.168.20.1  | 255.255.255.0 (/24) | —               |
| PC2    | eth0      | 192.168.20.10 | 255.255.255.0 (/24) | 192.168.20.1    |

## Steps

### 1. Configure PC1

Assign the IP address to PC1's `eth0` interface:

```bash
ip addr add 192.168.10.10/24 dev eth0
```

Configure the default gateway:

```bash
ip route add default via 192.168.10.1
```

Verify the configuration:

```bash
ip addr show eth0
ip route
```

PC1 is now part of the `192.168.10.0/24` network.

---

### 2. Configure PC2

Assign the IP address to PC2:

```bash
ip addr add 192.168.20.10/24 dev eth0
```

Configure the default gateway:

```bash
ip route add default via 192.168.20.1
```

Verify the configuration:

```bash
ip addr show eth0
ip route
```

PC2 is now part of the `192.168.20.0/24` network.

---

### 3. Configure R1 Gi0/0

Configure the router interface connected to SW1:

```text
enable
configure terminal

interface GigabitEthernet0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
```

This interface acts as the default gateway for PC1.

---

### 4. Configure R1 Gi0/1

Configure the router interface connected to SW2:

```text
interface GigabitEthernet0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
```

This interface acts as the default gateway for PC2.

Save the configuration:

```text
end
write memory
```

---

### 5. Verify Router Interfaces

Check the status and IP addresses of the router interfaces:

```text
show ip interface brief
```

The expected result is that both `Gi0/0` and `Gi0/1` are **up/up**.

The router should have two directly connected networks:

```text
192.168.10.0/24
192.168.20.0/24
```

---

### 6. Test Local Connectivity

From PC1, first test its local gateway:

```bash
ping 192.168.10.1
```

From PC2, test its local gateway:

```bash
ping 192.168.20.1
```

If both pings are successful, the PCs can communicate with their respective router interfaces.

---

### 7. Test PC1 to PC2 Connectivity

From PC1, ping PC2:

```bash
ping 192.168.20.10
```

The expected result is successful ICMP Echo Replies from:

```text
192.168.20.10
```

This proves that traffic is successfully traveling:

```text
PC1 → SW1 → R1 → SW2 → PC2
```

The return traffic follows the reverse path:

```text
PC2 → SW2 → R1 → SW1 → PC1
```

## Wireshark Analysis

Wireshark is used to observe the actual packets exchanged between the devices.

Capture traffic on both **PC–SW links**.

### ARP Analysis

Before the first successful ping, ARP is used to discover the MAC address of the next-hop device.

Use the Wireshark display filter:

```text
arp
```

You should observe an ARP Request followed by an ARP Reply.

The ARP Request contains a message similar to:

```text
Who has 192.168.10.1?
```

The router responds with an ARP Reply:

```text
192.168.10.1 is-at <MAC address>
```

The same process occurs on the PC2 side for its gateway:

```text
Who has 192.168.20.1?
```

This demonstrates that ARP resolves an IP address to a MAC address on the local Ethernet network.

### ICMP Analysis

Use the following Wireshark filter:

```text
icmp
```

After ARP resolution, ICMP packets can be observed during the ping.

The main ICMP messages are:

```text
Echo Request
Echo Reply
```

PC1 sends an **ICMP Echo Request** toward PC2, and PC2 responds with an **ICMP Echo Reply**.

The Echo Request and Echo Reply can be matched using their:

* Identifier
* Sequence number

This allows us to confirm that the reply corresponds to the original request.

### Key Takeaways

* Devices use **ARP** to discover MAC addresses on the local network.
* A **default gateway** is required when a host communicates with another IP network.
* A router forwards packets between different networks.
* **ICMP Echo Request and Echo Reply** are used by `ping` to test IP connectivity.
