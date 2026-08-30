OSPF Multi-Router Lab

Objective

Configure a multi-router OSPF Area 0 topology and practice:

OSPF on router interfaces

Loopback interfaces

Passive interfaces

OSPF neighbors and routes

Default static route

OSPF default-route advertisement

ISP1 static return route

End-to-end connectivity testing

Topology

                         203.0.113.0/30
                   .2                 .1
              +---------+        +---------+
              |  ISP1   |--------|   R1    |
              +---------+        +---------+
                                    /                                       /                               10.0.12.0/30    10.0.13.0/30
                                 /                                          /                                          R2              R3
                              |                |
                       10.0.24.0/30     10.0.34.0/30
                              |                |
                              +------ R4 ------+
                                     |
                              192.168.4.0/24
                                     |
                                    SW1
                                     |
                                    PC1
                                  .1/24

IP Addressing

Link/Device

Address

ISP1-R1

ISP1 203.0.113.2/30, R1 203.0.113.1/30

R1-R2

R1 10.0.12.1/30, R2 10.0.12.2/30

R1-R3

R1 10.0.13.1/30, R3 10.0.13.2/30

R2-R4

R2 10.0.24.1/30, R4 10.0.24.2/30

R3-R4

R3 10.0.34.1/30, R4 10.0.34.2/30

R1 Loopback0

1.1.1.1/32

R2 Loopback0

2.2.2.2/32

R3 Loopback0

3.3.3.3/32

R4 Loopback0

4.4.4.4/32

R4 LAN

192.168.4.254/24

PC1

192.168.4.1/24

PC1 Gateway

192.168.4.254

Use the actual interface names on your Packet Tracer router. They may
be FastEthernet or GigabitEthernet.

/30 Wildcard Mask

A /30 has:

Subnet mask: 255.255.255.252
Wildcard:    0.0.0.3
Usable IPs:  2

Example:

10.0.12.0/30
R1 = 10.0.12.1
R2 = 10.0.12.2

1. Configure Router Interfaces

R1

enable
configure terminal
hostname R1

interface <R1-to-R2>
 ip address 10.0.12.1 255.255.255.252
 no shutdown

interface <R1-to-R3>
 ip address 10.0.13.1 255.255.255.252
 no shutdown

interface <R1-to-ISP1>
 ip address 203.0.113.1 255.255.255.252
 no shutdown

interface loopback0
 ip address 1.1.1.1 255.255.255.255
end

R2

enable
configure terminal
hostname R2

interface <R2-to-R1>
 ip address 10.0.12.2 255.255.255.252
 no shutdown

interface <R2-to-R4>
 ip address 10.0.24.1 255.255.255.252
 no shutdown

interface loopback0
 ip address 2.2.2.2 255.255.255.255
end

R3

enable
configure terminal
hostname R3

interface <R3-to-R1>
 ip address 10.0.13.2 255.255.255.252
 no shutdown

interface <R3-to-R4>
 ip address 10.0.34.1 255.255.255.252
 no shutdown

interface loopback0
 ip address 3.3.3.3 255.255.255.255
end

R4

enable
configure terminal
hostname R4

interface <R4-to-R2>
 ip address 10.0.24.2 255.255.255.252
 no shutdown

interface <R4-to-R3>
 ip address 10.0.34.2 255.255.255.252
 no shutdown

interface <R4-to-SW1>
 ip address 192.168.4.254 255.255.255.0
 no shutdown

interface loopback0
 ip address 4.4.4.4 255.255.255.255
end

ISP1

enable
configure terminal
hostname ISP1

interface <ISP1-to-R1>
 ip address 203.0.113.2 255.255.255.252
 no shutdown
end

2. Configure PC1

IP address:      192.168.4.1
Subnet mask:     255.255.255.0
Default gateway: 192.168.4.254

Test:

ping 192.168.4.254

3. Configure OSPF Area 0

R1

router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0

R2

router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.24.0 0.0.0.3 area 0
 network 2.2.2.2 0.0.0.0 area 0

R3

router ospf 1
 network 10.0.13.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
 network 3.3.3.3 0.0.0.0 area 0

R4

router ospf 1
 network 10.0.24.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
 network 192.168.4.0 0.0.0.255 area 0
 network 4.4.4.4 0.0.0.0 area 0

4. Configure Passive Interface

R4’s LAN interface connects to a PC, so it should be passive.

R4(config)# router ospf 1
R4(config-router)# passive-interface <R4-to-SW1>

Passive interface means:

No OSPF Hello packets are sent on that interface.

No OSPF neighbor forms there.

The LAN network can still be advertised through OSPF.

So passive does not mean “do not advertise the network.”

5. Verify OSPF

Neighbors

show ip ospf neighbor

Expected relationships:

R1 <-> R2
R1 <-> R3
R2 <-> R4
R3 <-> R4

Neighbors should reach:

FULL

Routing table

show ip route

OSPF routes begin with:

O

Useful commands:

show ip route ospf
show ip ospf
show ip ospf interface
show ip protocols

6. Configure R1 Default Route Toward ISP1

R1 is the ASBR connecting OSPF to ISP1.

R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2

Verify:

show ip route

Expected:

S* 0.0.0.0/0 [1/0] via 203.0.113.2

Advertise the default route into OSPF:

R1(config)# router ospf 1
R1(config-router)# default-information originate

Check R2/R3/R4:

show ip route

You should see an OSPF external default route similar to:

O*E2 0.0.0.0/0

7. Configure ISP1 Return Route

This is required for PC1 to communicate with ISP1.

ISP1 must know how to return traffic to the internal LAN:

192.168.4.0/24

On ISP1:

configure terminal
ip route 192.168.4.0 255.255.255.0 203.0.113.1
end

Verify:

show ip route 192.168.4.0

Expected:

S 192.168.4.0/24 [1/0] via 203.0.113.1

Why the return route matters

The forward path is:

PC1 -> R4 -> R2/R3 -> R1 -> ISP1

The reply must return:

ISP1 -> R1 -> OSPF -> R4 -> PC1

Without a route on ISP1 back to 192.168.4.0/24, the ping can fail even
when the forward path is correct.


9. Troubleshooting

Check interfaces:

show ip interface brief

Check OSPF neighbors:

show ip ospf neighbor

Check OSPF configuration:

show running-config | section router ospf

Check routing:

show ip route
show ip route ospf
show ip route 192.168.4.0
show ip route 0.0.0.0

Trace the path:

traceroute 203.0.113.2

From a PC:

tracert 203.0.113.2

10. Save Configuration

On every Cisco device:

copy running-config startup-config

or:

write memory

Key Concepts

OSPF

OSPF dynamically exchanges routes between routers and selects paths
using OSPF cost.

Loopback

A loopback is a logical interface independent of a physical cable. It is
commonly used for stable router identification, testing, and OSPF router
IDs.

Passive Interface

A passive interface does not form an OSPF neighbor relationship or send
OSPF Hellos, but its connected network can still be advertised.

/30

A /30 subnet provides two usable addresses, making it useful for
point-to-point router links.

Mask:      255.255.255.252
Wildcard:  0.0.0.3
