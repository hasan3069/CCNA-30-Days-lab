Day 03 — Static Routing Between Multiple Routers
Objective

Build a 3-router chain and configure a complete static routing table on each router.

The goal is to understand how routers forward packets between multiple networks using manually configured static routes, without using any dynamic routing protocol.

Topology
PC1 ── SW1 ── R1 ── R2 ── R3 ── SW2 ── PC2
          LAN1      R1-R2    R2-R3      LAN2
Networks
LAN1: 172.16.1.0/24
R1-R2: 172.16.12.0/30
R2-R3: 172.16.23.0/30
LAN2: 172.16.2.0/24
IP Addressing
Device	Interface	IP Address	Subnet Mask	Purpose
PC1	NIC	172.16.1.10	/24	LAN1 Host
R1	LAN Interface	172.16.1.1	/24	PC1 Gateway
R1	R1-R2	172.16.12.1	/30	Router Link
R2	R1-R2	172.16.12.2	/30	Router Link
R2	R2-R3	172.16.23.1	/30	Router Link
R3	R2-R3	172.16.23.2	/30	Router Link
R3	LAN Interface	172.16.2.1	/24	PC2 Gateway
PC2	NIC	172.16.2.10	/24	LAN2 Host
Static Routing

Each router must have static routes for every non-directly-connected network.

R1
ip route 172.16.23.0 255.255.255.252 172.16.12.2
ip route 172.16.2.0 255.255.255.0 172.16.12.2
R2
ip route 172.16.1.0 255.255.255.0 172.16.12.1
ip route 172.16.2.0 255.255.255.0 172.16.23.2
R3
ip route 172.16.12.0 255.255.255.252 172.16.23.1
ip route 172.16.1.0 255.255.255.0 172.16.23.1

The key idea is both directions: R1 must know how to reach LAN2, and R3 must know how to reach LAN1.

Verification
1. Check Router Interfaces
show ip interface brief

Make sure all required interfaces are up/up and have the correct IP addresses.

2. Check Routing Tables
show ip route

Look for routes marked with:

S

S indicates a static route.

3. Test Router-to-Router Connectivity

From R1:

ping 172.16.12.2

From R2:

ping 172.16.23.2
4. Test End-to-End Connectivity

From PC1:

ping 172.16.2.10

From PC2:

ping 172.16.1.10

Successful replies confirm that static routing is working in both directions.

Wireshark Verification

Capture traffic while running:

ping 172.16.2.10

In Wireshark, add the IP TTL field as a column.

Observe how the TTL decreases as the packet passes through each router:

PC1 → R1 → R2 → R3 → PC2
       ↓     ↓     ↓
      TTL   TTL   TTL

Each router decrements the IPv4 TTL by 1 before forwarding the packet.

You can also use:

icmp

as a Wireshark display filter to focus on the ICMP echo request and reply packets.

Verification Checklist
 PC1 configured with correct IP, mask, and gateway
 PC2 configured with correct IP, mask, and gateway
 R1 interfaces configured
 R2 interfaces configured
 R3 interfaces configured
 R1 static routes configured
 R2 static routes configured
 R3 static routes configured
 Router-to-router pings successful
 PC1 → PC2 ping successful
 PC2 → PC1 ping successful
 Routing tables verified with show ip route
 ICMP traffic observed in Wireshark
 TTL decrement observed at each router<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3149f75e-2d54-4bac-bb70-3806e5c8e090" />
