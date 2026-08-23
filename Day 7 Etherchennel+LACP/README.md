# Day 7 — LACP EtherChannel Lab

## Objective

Bundle two physical links between two switches into **one logical LACP EtherChannel** instead of relying on STP to block one of the redundant links.

In this lab, we will:

* Configure LACP EtherChannel.
* Bundle two physical links into `Port-channel 1`.
* Verify the EtherChannel using `show etherchannel summary`.
* Verify that both physical interfaces are successfully bundled.
* Confirm that STP sees the EtherChannel as **one logical link**.
* Test connectivity between the PCs.

---

## Topology

```text
                  LACP ETHERCHANNEL
              ┌─────────────────────┐
              │                     │
           E0/1│=====================│E0/1
              │=====================│
           E0/2│                     │E0/2
              │                     │
           ┌──┴──┐               ┌──┴──┐
           │ SW1 │               │ SW2 │
           └──┬──┘               └──┬──┘
              │                     │
            E0/0                  E0/0
              │                     │
             PC1                   PC2
```

### Physical Connections

```text
PC1 ───── SW1 E0/0

SW1 E0/1 ───────── SW2 E0/1
SW1 E0/2 ───────── SW2 E0/2

PC2 ───── SW2 E0/0
```

The two links between SW1 and SW2 will be bundled into:

```text
Port-channel 1
```

---

## IP Addressing

| Device | Interface | IP Address    | Subnet Mask   | VLAN |
| ------ | --------- | ------------- | ------------- | ---- |
| PC1    | eth0      | 192.168.10.10 | 255.255.255.0 | 10   |
| PC2    | eth0      | 192.168.10.20 | 255.255.255.0 | 10   |

> **Note:** A default gateway is not required because both PCs are in the same subnet.

---

# Configuration

## Step 1 — Create VLAN 10

### SW1

```cisco
vlan 10
 name USERS
```

### SW2

```cisco
vlan 10
 name USERS
```

---

## Step 2 — Configure PC-facing interfaces

### SW1

```cisco
interface e0/0
 switchport mode access
 switchport access vlan 10
```

### SW2

```cisco
interface e0/0
 switchport mode access
 switchport access vlan 10
```

---

## Step 3 — Configure LACP EtherChannel

### SW1

```cisco
interface range e0/1 - 2
 channel-group 1 mode active
```

### SW2

```cisco
interface range e0/1 - 2
 channel-group 1 mode active
```

`active` enables **LACP negotiation**.

Both switches use LACP Active mode, so they actively negotiate the EtherChannel.

---

## Step 4 — Configure the Port-Channel as a Trunk

### SW1

```cisco
interface port-channel 1
switchport trunk encapsulation dot1q
 switchport mode trunk
 
```

### SW2

```cisco
interface port-channel 1
switchport trunk encapsulation dot1q
 switchport mode trunk
 
```

The two physical links now operate as members of:

```text
Port-channel 1
```

---

# Verification

## 1. Verify EtherChannel

Run on both switches:

```cisco
show etherchannel summary
```

Expected output should look similar to:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------
1      Po1(SU)       LACP        E0/1(P)
                                  E0/2(P)
```

### Important symbols

```text
Po1(SU)
```

* `S` = Layer 2 EtherChannel
* `U` = Port-Channel is in use/up

```text
E0/1(P)
E0/2(P)
```

* `P` = Port is successfully bundled in the EtherChannel

Both physical interfaces should show `(P)`.

---

## 2. Verify LACP Neighbor

```cisco
show lacp neighbor
```

This verifies that the switches have successfully formed an LACP relationship.

---

## 3. Verify STP

```cisco
show spanning-tree
```

STP should now see:

```text
Port-channel 1
```

as the logical connection between the switches rather than treating the two physical links as two separate paths.

You can also check:

```cisco
show spanning-tree interface port-channel 1
```

### Key Concept

Before EtherChannel:

```text
SW1 ===== SW2
       Link 1

SW1 ===== SW2
       Link 2

STP may block one link.
```

After EtherChannel:

```text
SW1 ================= SW2
       Port-channel 1
```

STP sees the bundle as **one logical link**.

---

# Connectivity Test

From PC1:

```bash
ping 192.168.10.20
```

Expected result:

```text
PC1 → SW1 → Port-channel 1 → SW2 → PC2
```

The ping should be successful.

You can also test from PC2:

```bash
ping 192.168.10.10
```

---

# Important Concepts Learned

### EtherChannel

EtherChannel combines multiple physical links into one logical link.

### LACP

LACP stands for **Link Aggregation Control Protocol** and is used to negotiate the EtherChannel.

### Port-Channel

`Port-channel 1` is the logical interface representing the bundled physical links.

### LACP Active

```cisco
channel-group 1 mode active
```

means the switch actively participates in LACP negotiation.

### STP and EtherChannel

STP sees the EtherChannel as **one logical link**, rather than seeing each physical member as a separate redundant path.

### Redundancy

If one physical member fails, the remaining member can continue carrying traffic through the EtherChannel, although the available aggregate capacity is reduced.

---

# Useful Commands

```cisco
show etherchannel summary
show etherchannel detail
show lacp neighbor
show lacp internal
show spanning-tree
show spanning-tree interface port-channel 1
show interfaces trunk
```
