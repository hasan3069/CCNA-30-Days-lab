# Day 03 — Static Routing Between Multiple Routers

## Objective

Build a **3-router chain** and configure a complete **static routing table** on each router.

The goal is to understand how routers forward packets between multiple networks using manually configured static routes, without using any dynamic routing protocol.

## Topology

```text
PC1 ── SW1 ── R1 ── R2 ── R3 ── SW2 ── PC2
          LAN1      R1-R2    R2-R3      LAN2
```

## IP Addressing Table

| Link  | Network          | R1                | R2            | R3                |
| ----- | ---------------- | ----------------- | ------------- | ----------------- |
| LAN1  | `172.16.1.0/24`  | `172.16.1.1` (GW) | —             | —                 |
| R1-R2 | `172.16.12.0/30` | `172.16.12.1`     | `172.16.12.2` | —                 |
| R2-R3 | `172.16.23.0/30` | —                 | `172.16.23.1` | `172.16.23.2`     |
| LAN2  | `172.16.2.0/24`  | —                 | —             | `172.16.2.1` (GW) |

### End Devices

| Device | IP Address    | Subnet Mask     | Default Gateway |
| ------ | ------------- | --------------- | --------------- |
| PC1    | `172.16.1.10` | `255.255.255.0` | `172.16.1.1`    |
| PC2    | `172.16.2.10` | `255.255.255.0` | `172.16.2.1`    |

## Static Routing Configuration

Each router is configured with static routes for every **non-directly-connected network**.

### R1

```bash
ip route 172.16.23.0 255.255.255.252 172.16.12.2
ip route 172.16.2.0 255.255.255.0 172.16.12.2
```

### R2

```bash
ip route 172.16.1.0 255.255.255.0 172.16.12.1
ip route 172.16.2.0 255.255.255.0 172.16.23.2
```

### R3

```bash
ip route 172.16.12.0 255.255.255.252 172.16.23.1
ip route 172.16.1.0 255.255.255.0 172.16.23.1
```

> **Important:** Static routing must work in both directions. A router needs a route to the destination network and a return path for the reply.

## Verification

### Check Interfaces

```bash
show ip interface brief
```

Make sure all required interfaces are **up/up** and have the correct IP addresses.

### Check Routing Table

```bash
show ip route
```

Static routes should appear with the **`S`** code.

### Test Connectivity

From PC1:

```bash
ping 172.16.2.10
```

From PC2:

```bash
ping 172.16.1.10
```

Successful replies confirm end-to-end connectivity.

## Wireshark Analysis

Capture traffic while running:

```bash
ping 172.16.2.10
```

Use the following Wireshark display filter:

```text
icmp
```

Add the **IP TTL** field as a column and observe the TTL value as the packet passes through the routers.

```text
PC1 → R1 → R2 → R3 → PC2
       ↓     ↓     ↓
      TTL   TTL   TTL
```

The IPv4 TTL decreases by **1 at every router hop**.

## Verification Checklist

* [ ] PC1 IP, subnet mask, and gateway configured
* [ ] PC2 IP, subnet mask, and gateway configured
* [ ] R1 interfaces configured
* [ ] R2 interfaces configured
* [ ] R3 interfaces configured
* [ ] Static routes configured on R1
* [ ] Static routes configured on R2
* [ ] Static routes configured on R3
* [ ] Router-to-router connectivity verified
* [ ] PC1 → PC2 connectivity verified
* [ ] PC2 → PC1 connectivity verified
* [ ] Routing tables checked with `show ip route`
* [ ] ICMP traffic captured in Wireshark
* [ ] TTL decrement observed

