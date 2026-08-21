# 🔥 STP Comprehensive Lab — Labs 1–3

## 📌 Overview

This lab focuses on understanding the fundamentals of **Spanning Tree Protocol (STP)** using **GNS3 and Cisco IOSvL2 switches**.

The first three labs cover:

1. **STP Root Bridge Election**
2. **Changing the Root Bridge**
3. **Understanding STP Port Roles**

---

# 🧪 Lab 1 — STP Root Bridge Election

## 🎯 Objective

Build a Layer 2 redundant topology and observe how STP:

* Elects the Root Bridge
* Selects Root Ports
* Selects Designated Ports
* Places redundant ports into a blocking/non-forwarding state
* Prevents Layer 2 loops

## 🖥️ Topology

```text
                 SW1
                /   \
               /     \
             SW2-----SW3
```

### Devices

* 3 × Cisco IOSvL2 Switches
* GNS3
* Ethernet links between switches

## ⚙️ Configuration

Check the STP status:

```bash
show spanning-tree
```

For a specific VLAN:

```bash
show spanning-tree vlan 10
```

## 🔎 What to Analyze

Identify:

* Root Bridge
* Root Bridge ID
* Root Port
* Designated Port
* Alternate/Blocking Port
* Bridge Priority
* MAC Address
* Root Path Cost

### Key Concept

STP elects the switch with the **lowest Bridge ID** as the Root Bridge.

```text
Bridge ID = Priority + MAC Address
```

If priorities are equal, the switch with the **lower MAC address** wins.

---

# 🧪 Lab 2 — Change the Root Bridge

## 🎯 Objective

Manually control the Root Bridge election by changing the STP priority.

## 🔧 Make SW1 the Root Bridge

On SW1:

```bash
SW1(config)# spanning-tree vlan 10 priority 4096
```

Verify:

```bash
SW1# show spanning-tree vlan 10
```

SW1 should now become the Root Bridge.

## 🔄 Alternative Method

Cisco also provides:

```bash
SW1(config)# spanning-tree vlan 10 root primary
```

Verify:

```bash
show spanning-tree vlan 10
```

## 🧪 Experiment

After making SW1 the Root Bridge:

1. Check the Root Port on SW2.
2. Check the Root Port on SW3.
3. Find the blocked/alternate port.
4. Record the Root Path Cost.
5. Record the Bridge IDs.

Then make SW2 the Root Bridge:

```bash
SW2(config)# spanning-tree vlan 10 priority 4096
```

Check how the topology changes.

## 🧠 Important Concept

**Lower Bridge ID = Higher chance of becoming Root Bridge.**

Therefore:

```text
Lower Priority  → Lower Bridge ID → Better
Higher Priority → Higher Bridge ID → Worse
```

---

# 🧪 Lab 3 — Analyze STP Port Roles

## 🎯 Objective

Understand how STP assigns different roles to ports in a redundant topology.

## 🖥️ Topology

```text
                 SW1
                /   \
               /     \
             SW2-----SW3
```

Make SW1 the Root Bridge:

```bash
SW1(config)# spanning-tree vlan 10 priority 4096
```

Then check:

```bash
show spanning-tree vlan 10
```

## 🔍 Identify the Port Roles

### 1. Root Port

The **Root Port** is the best path from a non-root switch toward the Root Bridge.

Each non-root switch normally has **one Root Port per STP instance**.

---

### 2. Designated Port

A **Designated Port** is the forwarding port selected for a particular Layer 2 segment.

The Root Bridge's active ports are normally Designated Ports.

---

### 3. Alternate Port

An **Alternate Port** provides an alternative path toward the Root Bridge.

It remains non-forwarding so that the Layer 2 loop is prevented.

---

## 🧪 Experiment — Shut Down a Link

Take one of the redundant links and shut it down:

```bash
SW1(config)# interface e0/1
SW1(config-if)# shutdown
```

Then check:

```bash
show spanning-tree vlan 10
```

Observe:

* Which port changes?
* Which port becomes the new Root Port?
* Does the previously blocked path become active?
* Does connectivity remain available?

Bring the link back:

```bash
SW1(config-if)# no shutdown
```

Check STP again.

---

# 📊 Commands Used

```bash
show spanning-tree
```

```bash
show spanning-tree vlan 10
```

```bash
show spanning-tree summary
```

```bash
show spanning-tree inconsistentports
```

Useful interface commands:

```bash
show interfaces status
```

```bash
show interfaces trunk
```

---

# 🧠 Key Concepts Learned

| Concept         | Meaning                                 |
| --------------- | --------------------------------------- |
| Root Bridge     | Central reference point selected by STP |
| Bridge ID       | Priority + MAC-based identifier         |
| Root Port       | Best path toward Root Bridge            |
| Designated Port | Forwarding port for a segment           |
| Alternate Port  | Backup path that does not forward       |
| Root Path Cost  | Cost to reach Root Bridge               |
| STP Priority    | Used to influence Root Bridge election  |
| STP Convergence | Process of adapting to topology changes |

---

# 🏁 Final Challenge

After completing all three labs, try to predict the STP topology **before running `show spanning-tree`**.

For the topology:

```text
                 SW1
                /   \
               /     \
             SW2-----SW3
```

Answer these questions:

1. Which switch becomes Root Bridge?
2. Which ports become Root Ports?
3. Which ports become Designated Ports?
4. Which port becomes Alternate/non-forwarding?
5. What happens when the Root Bridge changes?
6. What happens when a link fails?
7. How does STP prevent the Layer 2 loop?

If you can answer these questions **before checking the CLI output**, you are starting to understand STP rather than simply memorizing STP commands.

---

## 📚 Lab Environment

* **Simulator:** GNS3
* **Switch:** Cisco IOSvL2
* **Protocol:** Spanning Tree Protocol (STP)
* **Topology:** 3-switch Layer 2 triangle
