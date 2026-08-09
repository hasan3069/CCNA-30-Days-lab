# Day 02 — Subnetting Design Lab (VLSM)

## Objective

Take one network block, `10.10.0.0/24`, and subnet it using **Variable Length Subnet Masking (VLSM)** for four unequal-sized networks.

The goal is to design the subnets, configure the routers, and verify connectivity using `ping`, `traceroute`, and Wireshark.

---

## Topology

```text
                  WAN Link
        ┌────────────────────────┐
        │                        │
      R1 │                        │ R2
   ┌─────┴─────┐              ┌───┴───┐
   │           │              │       │
 LAN A       LAN B          LAN C
 50 Hosts    20 Hosts       10 Hosts
   │           │              │
  PC1         PC2            PC3
```

### Network Design

* **R1** handles LAN A and LAN B.
* **R1 ↔ R2** uses a point-to-point WAN link.
* **R2** handles LAN C.
* One PC is used to represent each LAN.

---

## VLSM Addressing Plan

Starting network:

```text
10.10.0.0/24
```

The networks are allocated from largest to smallest based on the number of required hosts.

| Segment  | Hosts Needed | Network          | Subnet Mask       | Usable Range                | Gateway      | Broadcast     |
| -------- | -----------: | ---------------- | ----------------- | --------------------------- | ------------ | ------------- |
| LAN A    |           50 | `10.10.0.0/26`   | `255.255.255.192` | `10.10.0.1 – 10.10.0.62`    | `10.10.0.1`  | `10.10.0.63`  |
| LAN B    |           20 | `10.10.0.64/27`  | `255.255.255.224` | `10.10.0.65 – 10.10.0.94`   | `10.10.0.65` | `10.10.0.95`  |
| LAN C    |           10 | `10.10.0.96/28`  | `255.255.255.240` | `10.10.0.97 – 10.10.0.110`  | `10.10.0.97` | `10.10.0.111` |
| WAN Link |            2 | `10.10.0.112/30` | `255.255.255.252` | `10.10.0.113 – 10.10.0.114` | N/A          | `10.10.0.115` |

---

## IP Address Assignment

### R1

| Interface | IP Address       | Purpose       |
| --------- | ---------------- | ------------- |
| LAN A     | `10.10.0.1/26`   | LAN A Gateway |
| LAN B     | `10.10.0.65/27`  | LAN B Gateway |
| WAN       | `10.10.0.113/30` | R1–R2 Link    |

### R2

| Interface | IP Address       | Purpose       |
| --------- | ---------------- | ------------- |
| LAN C     | `10.10.0.97/28`  | LAN C Gateway |
| WAN       | `10.10.0.114/30` | R1–R2 Link    |

### PCs

| Device      | IP Address    | Subnet Mask       | Default Gateway |
| ----------- | ------------- | ----------------- | --------------- |
| PC1 — LAN A | `10.10.0.10`  | `255.255.255.192` | `10.10.0.1`     |
| PC2 — LAN B | `10.10.0.70`  | `255.255.255.224` | `10.10.0.65`    |
| PC3 — LAN C | `10.10.0.100` | `255.255.255.240` | `10.10.0.97`    |

---

## VLSM Calculation

The original network is:

```text
10.10.0.0/24
```

### LAN A — 50 Hosts

50 hosts require at least 62 usable addresses.

```text
/26
255.255.255.192
62 usable hosts
```

Allocated:

```text
10.10.0.0/26
```

### LAN B — 20 Hosts

20 hosts require at least 30 usable addresses.

```text
/27
255.255.255.224
30 usable hosts
```

Allocated:

```text
10.10.0.64/27
```

### LAN C — 10 Hosts

10 hosts require at least 14 usable addresses.

```text
/28
255.255.255.240
14 usable hosts
```

Allocated:

```text
10.10.0.96/28
```

### WAN Link — 2 Hosts

A point-to-point link requires 2 usable addresses.

```text
/30
255.255.255.252
2 usable hosts
```

Allocated:

```text
10.10.0.112/30
```

---

## Configuration Steps

### 1. Configure R1

Configure the interfaces connected to LAN A, LAN B, and the WAN link according to the addressing table.

Example:

```bash
R1(config)# interface <LAN-A-interface>
R1(config-if)# ip address 10.10.0.1 255.255.255.192
R1(config-if)# no shutdown

R1(config)# interface <LAN-B-interface>
R1(config-if)# ip address 10.10.0.65 255.255.255.224
R1(config-if)# no shutdown

R1(config)# interface <WAN-interface>
R1(config-if)# ip address 10.10.0.113 255.255.255.252
R1(config-if)# no shutdown
```

---

### 2. Configure R2

```bash
R2(config)# interface <LAN-C-interface>
R2(config-if)# ip address 10.10.0.97 255.255.255.240
R2(config-if)# no shutdown

R2(config)# interface <WAN-interface>
R2(config-if)# ip address 10.10.0.114 255.255.255.252
R2(config-if)# no shutdown
```

---

## Static Routing

R1 needs a route to LAN C:

```bash
R1(config)# ip route 10.10.0.96 255.255.255.240 10.10.0.114
```

R2 needs routes to LAN A and LAN B:

```bash
R2(config)# ip route 10.10.0.0 255.255.255.192 10.10.0.113
R2(config)# ip route 10.10.0.64 255.255.255.224 10.10.0.113
```

---

## Verification

### Check Interface Status

```bash
show ip interface brief
```

All required interfaces should be **up/up**.

### Check Routing Table

On R1:

```bash
show ip route
```

On R2:

```bash
show ip route
```

Verify that the remote LAN networks appear as static routes.

### Test Local Connectivity

From PC1:

```bash
ping 10.10.0.1
```

From PC2:

```bash
ping 10.10.0.65
```

From PC3:

```bash
ping 10.10.0.97
```

### Test End-to-End Connectivity

From PC1:

```bash
ping 10.10.0.100
```

From PC3:

```bash
ping 10.10.0.10
```

Successful replies confirm that routing between the LANs is working.

