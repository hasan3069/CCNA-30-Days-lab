# Static NAT Lab – GNS3

## Objective

Configure and verify **Static Network Address Translation (Static NAT)** using Cisco routers in GNS3.

The main goal is to understand how a private inside-local IP address is permanently mapped to a public inside-global IP address.

---

## Topology

```text
PC1 ── SW1 ── R1 ───────── R2 ── SW2 ── PC2
             │              │
          NAT Router      Outside
```

### Devices

* 2 PCs
* 2 Layer 2 switches
* 2 Cisco routers
* GNS3

---

## IP Addressing

| Device | Interface | IP Address         | Default Gateway |
| ------ | --------- | ------------------ | --------------- |
| PC1    | Ethernet  | `192.168.10.10/24` | `192.168.10.1`  |
| R1     | E0/0      | `192.168.10.1/24`  | —               |
| R1     | E0/1      | `10.0.0.1/30`      | —               |
| R2     | E0/1      | `10.0.0.2/30`      | —               |
| R2     | E0/0      | `203.0.113.1/24`   | —               |
| PC2    | Ethernet  | `203.0.113.10/24`  | `203.0.113.1`   |

### Static NAT Mapping

```text
Inside Local     →     Inside Global
192.168.10.10   →     203.0.113.100
```

* **Inside Local:** `192.168.10.10`
* **Inside Global:** `203.0.113.100`

---

## Step 1 – Configure PC1

Configure PC1:

```text
IP Address:      192.168.10.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1
```

---

## Step 2 – Configure R1

### Inside interface

```cisco
enable
configure terminal

interface e0/0
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
 no shutdown
exit
```

### Outside interface

```cisco
interface e0/1
 ip address 10.0.0.1 255.255.255.252
 ip nat outside
 no shutdown
exit
```

### Static NAT

```cisco
ip nat inside source static 192.168.10.10 203.0.113.100
```

---

## Step 3 – Configure R2

```cisco
enable
configure terminal

interface e0/1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

interface e0/0
 ip address 203.0.113.1 255.255.255.0
 no shutdown
exit
```

---

## Step 4 – Configure Routing

### R1

R1 needs a route to the outside network:

```cisco
ip route 203.0.113.0 255.255.255.0 10.0.0.2
```

### R2

R2 needs a route back to the inside network:

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

---

## Step 5 – Configure PC2

```text
IP Address:      203.0.113.10
Subnet Mask:     255.255.255.0
Default Gateway: 203.0.113.1
```

---

## Step 6 – Verify Basic Connectivity

Before analyzing NAT, verify that the network is working normally.

From PC1:

```text
ping 192.168.10.1
ping 10.0.0.2
ping 203.0.113.10
```

From PC2:

```text
ping 203.0.113.1
```

If these tests work, basic Layer 2/Layer 3 connectivity and routing are functioning.

---

## Step 7 – Verify Static NAT

On R1:

```cisco
show ip nat translations
```

You should see the static mapping:

```text
Inside Global       Inside Local
203.0.113.100       192.168.10.10
```

Also use:

```cisco
show ip nat statistics
```

This displays NAT statistics and helps confirm that NAT is configured correctly.

---

## Important Testing Concept

The public/global address:

```text
203.0.113.100
```

represents PC1's private address:

```text
192.168.10.10
```

Therefore:

```text
192.168.10.10  ↔  203.0.113.100
```

is the permanent Static NAT mapping.

### Inside-to-Outside Traffic

When PC1 communicates with PC2:

```text
PC1 → R1 → R2 → PC2
```

R1 can translate PC1's source address from:

```text
192.168.10.10
```

to:

```text
203.0.113.100
```

Verify this with:

```cisco
show ip nat translations
```

---

## Important Note About the Outside Test

If PC2 and the Static NAT address `203.0.113.100` are configured in the **same subnet**, PC2 may try to reach `203.0.113.100` directly using ARP instead of sending the traffic toward R1.

For a clean **outside-to-inside Static NAT** demonstration, the outside host should be placed on a different subnet from the NAT global address.

The desired traffic flow is:

```text
PC2
  |
  | ping 203.0.113.100
  ↓
R2
  |
  ↓
R1
  |
  | Static NAT
  ↓
192.168.10.10
  |
PC1
```

This allows R1 to receive the traffic and translate:

```text
203.0.113.100 → 192.168.10.10
```

---

## Verification Commands

### Check interfaces

```cisco
show ip interface brief
```

### Check routing table

```cisco
show ip route
```

### Check NAT translations

```cisco
show ip nat translations
```

### Check NAT statistics

```cisco
show ip nat statistics
```





## Conclusion

This lab demonstrated the configuration and verification of **Static NAT in GNS3** using Cisco routers, switches, and PCs.

The lab showed how:

```text
Inside Local
192.168.10.10
      ↓
Static NAT
      ↓
Inside Global
203.0.113.100
```

provides a permanent one-to-one address translation.
