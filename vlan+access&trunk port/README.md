# Day 04 — VLAN Segmentation, Broadcast Isolation & Trunking

## Objective

Segment switches into multiple VLANs and understand how VLANs create separate broadcast domains.

This lab demonstrates:

* Creating VLAN 10 and VLAN 20.
* Assigning PCs to VLANs.
* Verifying communication within the same VLAN.
* Verifying isolation between different VLANs.
* Connecting two switches using an 802.1Q trunk.
* Understanding how the same VLAN can span multiple switches.
* Understanding the difference between **VLAN membership** and **IP subnet membership**.
* Observing that **same VLAN + different IP networks still requires Layer-3 routing**.

---

## Topology

```text
                    802.1Q TRUNK
             ┌──────────────────────┐
             │                      │
           SW1 ==================== SW2
          /   \                    / | \
        PC1   PC3                PC4 PC5 PC6
        V10   V10                V10 V20 V20
```

### VLAN Distribution

**SW1**

* PC1 → VLAN 10
* PC3 → VLAN 10
* PC2 → VLAN 20

**SW2**

* PC4 → VLAN 10
* PC5 → VLAN 20
* PC6 → VLAN 20

---

## IP Addressing Table

| Device | Switch | VLAN | IP Address      | Subnet Mask | Gateway |
| ------ | ------ | ---: | --------------- | ----------- | ------- |
| PC1    | SW1    |   10 | `192.168.10.11` | `/24`       | None    |
| PC3    | SW1    |   10 | `192.168.10.12` | `/24`       | None    |
| PC2    | SW1    |   20 | `192.168.20.11` | `/24`       | None    |
| PC4    | SW2    |   10 | `192.168.10.13` | `/24`       | None    |
| PC5    | SW2    |   20 | `192.168.20.12` | `/24`       | None    |
| PC6    | SW2    |   20 | `192.168.20.13` | `/24`       | None    |

> **Note:** No default gateway is configured because this lab does not contain a router or Layer-3 switch. Inter-VLAN routing will be covered in a later lab.

---

## VLAN Configuration

Create VLANs on **both switches**:

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

Verify:

```cisco
show vlan brief
```

---

## Access Port Configuration

### SW1

PC1 → VLAN 10:

```cisco
interface e0/0
switchport mode access
switchport access vlan 10
exit
```

PC2 → VLAN 20:

```cisco
interface e0/1
switchport mode access
switchport access vlan 20
exit
```

PC3 → VLAN 10:

```cisco
interface e0/2
switchport mode access
switchport access vlan 10
exit
```

### SW2

Assign the three PCs according to the topology:

* PC4 → VLAN 10
* PC5 → VLAN 20
* PC6 → VLAN 20

Example:

```cisco
interface e0/0
switchport mode access
switchport access vlan 10
exit

interface e0/1
switchport mode access
switchport access vlan 20
exit

interface e0/2
switchport mode access
switchport access vlan 20
exit
```

---

## Trunk Configuration

The link between SW1 and SW2 must be configured as a trunk.

On the IOSvL2 switch, first specify the trunk encapsulation:

```cisco
interface e0/3
switchport trunk encapsulation dot1q
switchport mode trunk
exit
```

Configure the same trunk settings on the corresponding interface of the other switch.

Verify:

```cisco
show interfaces trunk
```

The trunk should show:

```text
Status: trunking
Encapsulation: 802.1q
```

The trunk carries VLAN 10 and VLAN 20 traffic between SW1 and SW2.

---

## Connectivity Tests

### Test 1 — Same VLAN, Same Switch

From PC1:

```bash
ping 192.168.10.12
```

Expected:

```text
SUCCESS
```

PC1 and PC3 are both:

* VLAN 10
* `192.168.10.0/24`

Therefore, they can communicate directly.

---

### Test 2 — Same VLAN, Different Switch

From PC1:

```bash
ping 192.168.10.13
```

Expected:

```text
SUCCESS
```

PC1 and PC4 are connected to different switches, but both belong to **VLAN 10**.

The traffic travels through the trunk:

```text
PC1 → SW1 → 802.1Q Trunk → SW2 → PC4
```

This proves that a VLAN can span multiple switches through a trunk.

---

### Test 3 — Different VLANs

From PC1:

```bash
ping 192.168.20.11
```

Expected:

```text
FAIL
```

PC1 is in VLAN 10, while PC2 is in VLAN 20.

There is no router or Layer-3 switch configured, so there is no inter-VLAN routing.

---

### Test 4 — VLAN 20 Across the Trunk

From PC2:

```bash
ping 192.168.20.12
```

Expected:

```text
SUCCESS
```

PC2 and PC5 are both in VLAN 20 and the same IP subnet.

Their traffic crosses the trunk between SW1 and SW2.

---

## Important Note — Same VLAN but Different IP Network

> **⚠️ IMPORTANT CONCEPT: Same VLAN ≠ Same IP Network**
>
> Two devices can be placed in the **same VLAN** but still belong to **different IP networks**.
>
> Example:
>
> ```text
> PC1 → VLAN 10 → 192.168.10.11/24
> PC3 → VLAN 10 → 192.168.20.11/24
> ```
>
> Although both devices are in **VLAN 10**, they belong to different IP networks:
>
> ```text
> 192.168.10.0/24
> 192.168.20.0/24
> ```
>
> Therefore, they cannot communicate directly without a **default gateway/router**.
>
> For direct communication without routing, hosts normally need to be in:
>
> **Same VLAN + Same IP subnet**

---

## Broadcast Domain Isolation

A VLAN represents a separate Layer-2 broadcast domain.

For example, when PC1 sends an ARP broadcast:

```text
PC1 → ARP Broadcast
        ↓
       SW1
        ↓
     VLAN 10
        ↓
       PC3
```

The broadcast remains inside VLAN 10.

PC2, which belongs to VLAN 20, should not receive the broadcast.

This demonstrates:

```text
VLAN 10 = Broadcast Domain 1
VLAN 20 = Broadcast Domain 2
```

---

## MAC Address Verification

To see which MAC addresses the switch has learned:

```cisco
show mac address-table
```

To check a specific VLAN:

```cisco
show mac address-table vlan 10
```

```cisco
show mac address-table vlan 20
```

To see connected ports:

```cisco
show interfaces status
```

---

## Wireshark Analysis

### Access Ports

When capturing traffic from a PC connected to an access port, VLAN tags normally will not be visible.

Access ports normally send/receive untagged Ethernet frames toward the connected host.

```text
PC ───── Access Port ───── SW
          Untagged
```

### Trunk Port

The link between SW1 and SW2 is a trunk.

802.1Q VLAN tags can be observed on trunk traffic:

```text
SW1 ───── 802.1Q Trunk ───── SW2
             VLAN Tags
```

The trunk allows multiple VLANs to travel over a single physical link.

---

## Verification Commands

### Check VLANs

```cisco
show vlan brief
```

### Check trunk

```cisco
show interfaces trunk
```

### Check MAC address table

```cisco
show mac address-table
```

### Check interface status

```cisco
show interfaces status
```

### Check a specific interface

```cisco
show interfaces e0/3 switchport
```

---

## Expected Results

| Test                        | Expected Result | Reason                                      |
| --------------------------- | --------------- | ------------------------------------------- |
| PC1 → PC3                   | ✅ Works         | Same VLAN + same subnet                     |
| PC1 → PC4                   | ✅ Works         | Same VLAN + same subnet, different switches |
| PC2 → PC5                   | ✅ Works         | Same VLAN + same subnet, different switches |
| PC1 → PC2                   | ❌ Fails         | Different VLANs + no routing                |
| PC1 → PC5                   | ❌ Fails         | Different VLANs + no routing                |
| VLAN 10 broadcast → VLAN 20 | ❌ Not received  | Separate broadcast domains                  |

---

## Key Takeaways

1. **VLANs provide Layer-2 segmentation.**
2. Each VLAN is a separate **broadcast domain**.
3. Devices in the same VLAN can communicate at Layer 2, but their IP addressing must also allow direct Layer-3 communication.
4. **Same VLAN + Same IP subnet → direct communication.**
5. **Same VLAN + Different IP subnet → requires a router/default gateway.**
6. Different VLANs require **inter-VLAN routing** to communicate.
7. A trunk allows multiple VLANs to travel between switches.
8. **802.1Q** is used to identify VLANs across trunk links.
9. Access ports normally carry untagged frames toward end devices.
10. Trunk links can carry traffic from multiple VLANs.

---

## Troubleshooting

### Problem: Trunk command rejected

If you receive:

```text
An interface whose trunk encapsulation is "Auto"
can not be configured to "trunk" mode.
```

Configure 802.1Q first:

```cisco
switchport trunk encapsulation dot1q
switchport mode trunk
```

### Problem: VLAN 20 is not appearing in the MAC table

Generate traffic from a device in VLAN 20 and then check:

```cisco
show mac address-table vlan 20
```

The switch learns MAC addresses dynamically when it receives frames from the devices.

---

## Save Configuration

After completing the lab, save the configuration on both switches:

```cisco
copy running-config startup-config
```

Press **Enter** when asked for the destination filename.

---

## Resources

* [Practical Networking — VLANs](https://www.practicalnetworking.net/series/vlans/vlans/)

## Lab Conclusion

This lab demonstrated how VLANs divide a network into separate broadcast domains and how trunk links allow the same VLAN to extend across multiple switches.

The most important concept learned from this lab is:

> **Same VLAN does not automatically mean same IP network.**

A device needs to be in the same VLAN **and** the same IP subnet for direct communication without a router.
