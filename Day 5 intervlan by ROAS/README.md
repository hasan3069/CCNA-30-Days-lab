# Day 05 — 802.1Q Trunking + Inter-VLAN Routing (Router-on-a-Stick)

## Objective

Configure **802.1Q trunking** and **Inter-VLAN Routing using Router-on-a-Stick (ROAS)**.

The goal is to connect two VLANs through a router using:

* VLAN 10
* VLAN 20
* One trunk link between SW1 and R1
* Router subinterfaces
* 802.1Q VLAN tagging
* Wireshark packet analysis

---

## Topology

```text
PC1 ─── SW1 ═════════ R1
VLAN 10     802.1Q    Ethernet0/0
192.168.10.11  TRUNK       │
                            ├── Ethernet0/0.10 → VLAN 10
                            │
                            └── Ethernet0/0.20 → VLAN 20
PC2 ─── SW1
VLAN 20
192.168.20.11
```

---

## IP Addressing

| Device | Interface      | VLAN | IP Address    | Mask | Gateway      |
| ------ | -------------- | ---: | ------------- | ---- | ------------ |
| R1     | Ethernet0/0.10 |   10 | 192.168.10.1  | /24  | —            |
| R1     | Ethernet0/0.20 |   20 | 192.168.20.1  | /24  | —            |
| PC1    | eth0           |   10 | 192.168.10.11 | /24  | 192.168.10.1 |
| PC2    | eth0           |   20 | 192.168.20.11 | /24  | 192.168.20.1 |

---

## Configuration

### 1. Create VLANs on SW1

```cisco
enable
configure terminal

vlan 10
name VLAN10
exit

vlan 20
name VLAN20
exit
```

### 2. Configure PC1 Port — VLAN 10

```cisco
interface fa0/1
switchport mode access
switchport access vlan 10
exit
```

### 3. Configure PC2 Port — VLAN 20

```cisco
interface fa0/2
switchport mode access
switchport access vlan 20
exit
```

### 4. Configure SW1-R1 Link as Trunk

Assuming R1 is connected to `Fa0/24`:

```cisco
interface fa0/24
switchport mode trunk
exit
```

The trunk carries traffic for both VLAN 10 and VLAN 20.

---

## Router-on-a-Stick Configuration

### Physical Interface

The router uses an **Ethernet interface** in this lab.

```cisco
interface ethernet0/0
no shutdown
exit
```

No IP address is assigned directly to `Ethernet0/0`.

### VLAN 10 Subinterface

```cisco
interface ethernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit
```

### VLAN 20 Subinterface

```cisco
interface ethernet0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit
```

---

## PC Configuration

### PC1 — VLAN 10

```text
IP Address:      192.168.10.11
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1
```

### PC2 — VLAN 20

```text
IP Address:      192.168.20.11
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.20.1
```

---

## Verification

### Check VLANs

```cisco
show vlan brief
```

Verify that:

* PC1's port belongs to VLAN 10.
* PC2's port belongs to VLAN 20.

### Check Trunk

```cisco
show interfaces trunk
```

Verify that the SW1-R1 interface is operating as a trunk.

### Check Router Interfaces

```cisco
show ip interface brief
```

Expected:

```text
Ethernet0/0       unassigned
Ethernet0/0.10    192.168.10.1
Ethernet0/0.20    192.168.20.1
```

### Check Routing Table

```cisco
show ip route
```

Expected connected networks:

```text
192.168.10.0/24
192.168.20.0/24
```

### Test Connectivity

From PC1:

```text
ping 192.168.10.1
ping 192.168.20.1
ping 192.168.20.11
```

The final ping verifies **Inter-VLAN Routing**.

---



## Key Concepts Learned

### VLAN

Separates a Layer-2 network into different logical broadcast domains.

### Access Port

Carries traffic for a single VLAN and connects end devices such as PCs.

### Trunk Port

Carries traffic for multiple VLANs over a single physical link.

### 802.1Q

The IEEE standard used to tag Ethernet frames with VLAN information.

### Subinterface

A logical router interface created under a physical Ethernet interface.

Example:

```text
Ethernet0/0.10 → VLAN 10
Ethernet0/0.20 → VLAN 20
```

### Router-on-a-Stick

Uses one physical router interface with multiple subinterfaces to perform Inter-VLAN Routing.

```text
VLAN 10
   ↓
SW1
   ↓
802.1Q Trunk
   ↓
R1 Ethernet0/0
   ↓
Ethernet0/0.10 / Ethernet0/0.20
   ↓
Layer-3 Routing
   ↓
VLAN 20
```

---

## Important Notes

> **802.1Q tags are observed on the trunk link, not on normal access-port traffic.**

> **The physical router interface Ethernet0/0 does not have an IP address in this ROAS configuration. The subinterfaces hold the gateway IP addresses.**

> **VLAN 10 and VLAN 20 are different networks, so communication between them requires Layer-3 routing.**

> **The default gateway for each PC is the IP address of its corresponding router subinterface.**

---

## Final Verification

* [x] VLAN 10 configured.
* [x] VLAN 20 configured.
* [x] PC1 assigned to VLAN 10.
* [x] PC2 assigned to VLAN 20.
* [x] SW1-R1 link configured as a trunk.
* [x] R1 Ethernet0/0 enabled.
* [x] R1 Ethernet0/0.10 configured for VLAN 10.
* [x] R1 Ethernet0/0.20 configured for VLAN 20.
* [x] PC1 can ping its gateway.
* [x] PC2 can ping its gateway.
* [x] PC1 can ping PC2.
* [x] 802.1Q VLAN tags analyzed in Wireshark.
* [x] ICMP Echo Request/Reply traffic identified in Wireshark.

---

## Key Takeaway

**Router-on-a-Stick = One physical Ethernet interface + multiple subinterfaces + 802.1Q trunk = Inter-VLAN Routing.**
