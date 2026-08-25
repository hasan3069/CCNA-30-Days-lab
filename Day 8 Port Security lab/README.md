# 🔐 Port Security Lab — Port Security, Violation Modes

##  Objective

The objective of this lab is to understand and practice **Cisco Port Security** using GNS3.

In this lab, I configured:

* Port Security on access ports
* Maximum allowed secure MAC addresses
* **Shutdown** violation mode
* **Restrict** violation mode
* MAC address aging
* Sticky MAC learning
* Port-security violation testing
* Verification of secure MAC addresses
* An SVI on SW1 to observe how additional MAC addresses affect the maximum-MAC limit

> **Note:** Sticky learning is not enabled on SW1. It is enabled only on the SW2 uplink.

---

## 🖥️ Topology

```text
                         PC1
                          |
                       Ethernet0/0
                          |
                     +----------+
                     |   SW1    |
                     +----------+
                      |    |    |
             Ethernet0/1  |  Ethernet0/3
                      |    |    |
                     PC2  PC3  Ethernet0/0
                               |
                              SW2
                               |
                         Ethernet0/1
                               |
                              R1
```

---

## 🌐 IP Addressing

All devices are in the same network:

**Network:** `10.0.0.0/24`
**Subnet Mask:** `255.255.255.0`

| Device | Interface   | IP Address   | Subnet Mask     |
| ------ | ----------- | ------------ | --------------- |
| PC1    | Ethernet    | `10.0.0.1`   | `255.255.255.0` |
| PC2    | Ethernet    | `10.0.0.2`   | `255.255.255.0` |
| PC3    | Ethernet    | `10.0.0.3`   | `255.255.255.0` |
| SW1    | SVI         | `10.0.0.4`   | `255.255.255.0` |
| R1     | Ethernet0/0 | `10.0.0.254` | `255.255.255.0` |

---

# 🔐 Port Security Requirements

### SW1

Interfaces:

```text
Ethernet0/0
Ethernet0/1
Ethernet0/2
```

Requirements:

```text
Violation Mode: Shutdown
Maximum Addresses: 1
Sticky Learning: Disabled
Aging Time: 1 hour
```

### SW2

Interface:

```text
Ethernet0/0
```

Requirements:

```text
Violation Mode: Restrict
Maximum Addresses: 4
Sticky Learning: Enabled
```

---

# 1️⃣ Configure Port Security on SW1

Configure the three PC-facing interfaces as access ports.

```cisco
SW1(config)# interface range ethernet0/0 - 2
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport port-security
SW1(config-if-range)# switchport port-security maximum 1
SW1(config-if-range)# switchport port-security violation shutdown
SW1(config-if-range)# switchport port-security aging time 1
```

Sticky learning is **not enabled** on SW1.

### Verify

```cisco
SW1# show port-security
```

```cisco
SW1# show port-security interface ethernet0/0
```

---

# 2️⃣ Test Port Security on SW1

Because the maximum is:

```text
Maximum MAC addresses = 1
```

each port can have only one secure MAC address.

When PC1 sends traffic, SW1 learns PC1's MAC address.

If another device is connected to the same port, SW1 detects a different MAC address.

This produces a **port-security violation**.

Because the violation mode is:

```text
shutdown
```

the interface is placed into an **err-disabled state**.

### Verify

```cisco
SW1# show port-security interface ethernet0/0
```

You can also check:

```cisco
SW1# show interfaces status
```

---

# 3️⃣ Configure SVI on SW1

For additional testing, I configured an SVI on SW1.

```cisco
SW1(config)# interface vlan 1
SW1(config-if)# ip address 10.0.0.10 255.255.255.0
SW1(config-if)# no shutdown
```

Now there are five IP addresses in the same network:

```text
PC1  → 10.0.0.1
PC2  → 10.0.0.2
PC3  → 10.0.0.3
SW1  → 10.0.0.10
R1   → 10.0.0.254
```

---

# 4️⃣ Configure Port Security on SW2

The connection between SW1 and SW2 uses Ethernet interfaces.

Configure the SW2 interface connected toward SW1:

```cisco
SW2(config)# interface ethernet0/0
SW2(config-if)# switchport mode access
SW2(config-if)# switchport port-security
SW2(config-if)# switchport port-security maximum 4
SW2(config-if)# switchport port-security violation restrict
SW2(config-if)# switchport port-security mac-address sticky
```

Here:

```text
Maximum = 4
Violation mode = Restrict
Sticky learning = Enabled
```

---

# 5️⃣ Understanding Sticky Learning on SW2

With sticky learning enabled, SW2 automatically learns MAC addresses arriving on the protected interface and converts them into **sticky secure MAC addresses**.

For example, SW2 can learn MAC addresses belonging to:

```text
1. PC1
2. PC2
3. PC3
4. SW1 SVI
```

Verify them using:

```cisco
SW2# show port-security address
```

and:

```cisco
SW2# show running-config
```

---

# 6️⃣ Important Observation — Maximum 4 on SW2

The SW2 interface is configured with:

```text
Maximum secure MAC addresses = 4
```

Because SW1 has three PCs and an SVI, SW2 can learn multiple different source MAC addresses through the same interface.

The four learned MAC addresses can represent:

```text
1. PC1
2. PC2
3. PC3
4. SW1 SVI
```

Therefore, the maximum of four can be reached.

---

# 7️⃣ Effect of the MAC Address Limit

This was an important observation during the lab.

When SW2 has already learned four secure MAC addresses:

```text
SW2 Ethernet0/0
       |
       +---- MAC 1
       +---- MAC 2
       +---- MAC 3
       +---- MAC 4
              ↓
        Maximum reached
```

SW2 cannot learn another MAC address as an additional secure MAC.

As a result, traffic involving another MAC address can be restricted, which can affect connectivity toward R1.

This demonstrates an important concept:

> **Port Security works with MAC addresses, not IP addresses.**

---

# 8️⃣ Restrict Violation Mode

SW2 uses:

```cisco
switchport port-security violation restrict
```

With **restrict** mode:

* Unauthorized frames are dropped.
* The port does not become err-disabled.
* The security violation counter increases.
* The interface remains operational.

Verify with:

```cisco
SW2# show port-security interface ethernet0/0
```

Pay attention to:

```text
Security Violation Count
```

---

# 9️⃣ Shutdown vs Restrict

| Feature                      | Shutdown | Restrict |
| ---------------------------- | -------- | -------- |
| Unauthorized traffic dropped | ✅        | ✅        |
| Violation counter increases  | ✅        | ✅        |
| Port becomes err-disabled    | ✅        | ❌        |
| Port continues operating     | ❌        | ✅        |
| Used on                      | SW1      | SW2      |

### SW1

```text
Violation mode = shutdown
```

A violation can cause:

```text
PORT → ERR-DISABLED
```

### SW2

```text
Violation mode = restrict
```

A violation causes:

```text
Unauthorized traffic → Dropped
Violation counter → Increased
Port → Remains operational
```

---

# 🔍 Verification Commands

### Check Port Security

```cisco
show port-security
```

### Check a Specific Interface

```cisco
show port-security interface ethernet0/0
```

### Display Secure MAC Addresses

```cisco
show port-security address
```

### Check MAC Address Table

```cisco
show mac address-table
```

### Check Sticky MAC Addresses

```cisco
show running-config
```

Look for entries similar to:

```text
switchport port-security mac-address sticky xxxx.xxxx.xxxx
```

### Check Interface Status

```cisco
show interfaces status
```

### Check SVI

```cisco
show ip interface brief
```

---

# 🧪 Lab Testing Process

## Test 1 — SW1 Port Security

```text
Configure Ethernet0/0
        ↓
Connect PC1
        ↓
Generate traffic
        ↓
MAC learned
        ↓
Connect/change to another device
        ↓
Violation
        ↓
Shutdown mode
        ↓
Port becomes err-disabled
```

## Test 2 — SW2 Sticky Learning

```text
Configure Ethernet0/0
        ↓
Maximum = 4
        ↓
Sticky enabled
        ↓
Generate traffic from different devices
        ↓
MAC addresses learned
        ↓
Secure MAC count increases
```

## Test 3 — Maximum MAC Limit

```text
PC1 MAC
PC2 MAC
PC3 MAC
SW1 SVI MAC
       ↓
4 MAC addresses
       ↓
Maximum = 4
       ↓
Limit reached
       ↓
Additional MAC → Port Security Violation
```

---

# 🧠 Key Concepts Learned

### 1. Port Security

Port Security controls which MAC addresses are allowed to use a switch port.

### 2. Maximum MAC Address

```text
maximum 4
```

means the port can have up to **four secure MAC addresses**.

### 3. Sticky Learning

```text
switchport port-security mac-address sticky
```

allows the switch to automatically learn MAC addresses and treat them as sticky secure MAC addresses.

### 4. Sticky ≠ Maximum

These are two different settings:

```text
maximum 4
      ↓
How many MAC addresses?

sticky
      ↓
How learned MAC addresses are secured/stored?
```

### 5. Shutdown

A violation can place the interface into an **err-disabled** state.

### 6. Restrict

Unauthorized traffic is dropped, while the interface remains operational and the violation count increases.

### 7. Port Security Uses MAC Addresses

Port Security does not count IP addresses.

For example:

```text
5 IP addresses
```

does not automatically mean five secure addresses.

The switch is concerned with the **source MAC addresses** it receives.

---

# 📚 Conclusion

This lab demonstrated how Cisco Port Security can be used to control access to switch ports based on MAC addresses.

The main practical difference observed was between **shutdown** and **restrict** violation modes. SW1 used shutdown, causing the port to become err-disabled after a violation, while SW2 used restrict, which dropped unauthorized traffic and increased the violation counter without shutting down the port.

The additional SVI configured on SW1 was useful for understanding that **multiple MAC addresses can be learned through a single switch port**. Because SW2 was configured with a maximum of four secure MAC addresses, reaching that limit affected the ability to accept traffic from additional MAC addresses.

Overall, the lab provided practical experience with **maximum secure MAC addresses, sticky learning, MAC address violations, violation modes, MAC address tables, and port-security verification**.
