# IT-4501 · NRSIP Lab Experiments

![Course](https://img.shields.io/badge/Course-IT--4501%20NRSIP-blue)
![Packet Tracer](https://img.shields.io/badge/Tool-Cisco%20Packet%20Tracer-1BA0D7)
![Experiments](https://img.shields.io/badge/Experiments-5-orange)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

## About

This repository contains the lab configurations for **IT-4501 — Network Routing, Switching
and Internet Protocols (NRSIP)**. All experiments were designed and verified in **Cisco Packet
Tracer**, covering VLANs, Spanning Tree Protocol, VLSM subnetting, IPv6/SLAAC, and static/default
routing.

Each experiment folder is self-contained and generally includes:

| File | Purpose |
|---|---|
| `README.md` | Experiment-specific write-up (aim, topology, procedure) |
| `*.cfg` | Cisco IOS CLI configuration commands for each device |
| `pc_addressing.txt` / `addressing.txt` | End-device / subnet IP addressing plan |
| `verification.txt` | Commands used to verify the configuration (`show` commands, `ping`, `tracert`, etc.) |
| `output.txt` | Expected output / result as per the lab manual |
| `*.pkt` | Cisco Packet Tracer topology file (where available) |

## Folder Structure

```
IT-4501-NRSIP-LAB/
├── README.md
│
├── Experiment_1_VLAN_InterVLAN/
│   ├── README.md
│   ├── router.cfg
│   ├── switch.cfg
│   ├── pc_addressing.txt
│   ├── verification.txt
│   ├── output.txt
│   └── first.pkt
│
├── Experiment_2_STP/
│   ├── README.md
│   ├── S1.cfg
│   ├── S2.cfg
│   ├── S3.cfg
│   ├── pc_addressing.txt
│   ├── verification.txt
│   ├── output.txt
│   └── second.pkt
│
├── Experiment_3_VLSM/
│   ├── README.md
│   ├── R1.cfg
│   ├── R2.cfg
│   ├── addressing.txt
│   ├── verification.txt
│   ├── output.txt
│   └── three.pkt
│
├── Experiment_4_IPv6_SLAAC/
│   ├── README.md
│   ├── R1.cfg
│   ├── R2.cfg
│   ├── pc_addressing.txt
│   ├── verification.txt
│   ├── output.txt
│   └── four.pkt
│
└── Experiment_5_Static_Default_Routing/
    ├── README.md
    ├── R1.cfg
    ├── R2.cfg
    ├── R3.cfg
    ├── pc_addressing.txt
    ├── verification.txt
    └── output.txt
```

## Experiments Overview

| # | Experiment | Key Concepts | Devices | Packet Tracer File |
|---|---|---|---|---|
| 1 | VLAN Configuration and Inter-VLAN Routing | VLANs, Trunk, Router-on-a-Stick, dot1Q | 1 Router (1941), 1 Switch (2960), 4 PCs | ✅ `first.pkt` |
| 2 | STP Verification and Root Bridge Election | STP, Root Bridge, Port Roles, Loop Prevention | 3 Switches (2960), 3 PCs | ✅ `second.pkt` |
| 3 | IPv4 Subnetting and VLSM Design | VLSM, Subnetting, Static Routing | 2 Routers, 3 Switches, 3 PCs | ✅ `three.pkt` |
| 4 | IPv6 Configuration and SLAAC | IPv6, SLAAC, RA Messages, Static IPv6 Routes | 2 Routers, 2 Switches, 2 PCs | ✅ `four.pkt` |
| 5 | Static and Default Route Configuration | Static Routes, Default Route, Inter-network Routing | 3 Routers, 3 Switches, 3 PCs | ❌ No `.pkt` |

---

## Experiment 1 — VLAN Configuration and Inter-VLAN Routing

**Aim:** Configure VLANs on a Layer 2 switch and enable inter-VLAN communication using the
Router-on-a-Stick method with 802.1Q trunking on a single router interface.

**Topology**

```
                         +------------------+
                         |   Router R1      |
                         |   (Cisco 1941)   |
                         |                  |
                         |  G0/0.10 --------|--- 192.168.10.1/24 (VLAN10)
                         |  G0/0.20 --------|--- 192.168.20.1/24 (VLAN20)
                         +--------+---------+
                                  | G0/0 (802.1Q Trunk)
                                  |
                         +--------+---------+
                         |   Switch SW1     |
                         |   (Cisco 2960)   |
                         +--+----+----+----+-+
                            |    |    |    |
                       Fa0/1  Fa0/2 Fa0/3 Fa0/4
                            |    |    |    |
                          PC0  PC1  PC2  PC3
                        VLAN10 VLAN10 VLAN20 VLAN20
```

**IP Addressing**

| Device | Interface / VLAN | IP Address | Subnet Mask | Gateway |
|---|---|---|---|---|
| R1 | G0/0.10 (VLAN 10) | 192.168.10.1 | 255.255.255.0 | — |
| R1 | G0/0.20 (VLAN 20) | 192.168.20.1 | 255.255.255.0 | — |
| PC0 | VLAN 10 | 192.168.10.2 | 255.255.255.0 | 192.168.10.1 |
| PC1 | VLAN 10 | 192.168.10.3 | 255.255.255.0 | 192.168.10.1 |
| PC2 | VLAN 20 | 192.168.20.2 | 255.255.255.0 | 192.168.20.1 |
| PC3 | VLAN 20 | 192.168.20.3 | 255.255.255.0 | 192.168.20.1 |

**Key Config Files:** [`router.cfg`](Experiment_1_VLAN_InterVLAN/router.cfg) (sub-interfaces
`G0/0.10` / `G0/0.20` with `encapsulation dot1Q`), [`switch.cfg`](Experiment_1_VLAN_InterVLAN/switch.cfg)
(VLAN creation + trunk/access ports)

**Result:** PCs within the same VLAN communicate directly at Layer 2. PCs in different VLANs
(VLAN10 ↔ VLAN20) successfully ping each other via inter-VLAN routing through R1's
sub-interfaces, confirming correct trunk encapsulation and routing.

---

## Experiment 2 — STP Verification and Root Bridge Election

**Aim:** Configure a redundant, loop-prone Layer 2 topology across three switches, force root
bridge election via priority manipulation, and verify Spanning Tree Protocol port roles and
loop prevention.

**Topology**

```
                    +-----------+           +-----------+
                    |    S1     |-----------|    S2     |
                    | ROOT      |           |           |
                    | Pri: 4096 |           |Pri: 32768 |
                    +-----+-----+           +-----+-----+
                          |                       |
                          |     +-----------+     |
                          +-----|    S3     |-----+
                                |           |
                                |Pri: 32768 |
                                +-----+-----+
                                      |
        PC1 --- S1        PC2 --- S2        PC3 --- S3
```

**IP Addressing**

| Device | IP Address | Subnet Mask | Gateway |
|---|---|---|---|
| PC1 | 192.168.1.2 | 255.255.255.0 | 192.168.1.1 |
| PC2 | 192.168.1.3 | 255.255.255.0 | 192.168.1.1 |
| PC3 | 192.168.1.4 | 255.255.255.0 | 192.168.1.1 |

**Key Config Files:** [`S1.cfg`](Experiment_2_STP/S1.cfg) (`spanning-tree vlan 1 priority 4096`
— forces S1 as Root Bridge), [`S2.cfg`](Experiment_2_STP/S2.cfg), [`S3.cfg`](Experiment_2_STP/S3.cfg)
(default priority 32768)

**Result:** S1 is elected Root Bridge due to its lower bridge priority (4096). STP places one
port on S2/S3 into a **Blocking** state to eliminate the physical loop while all PCs retain
full connectivity, confirmed via `show spanning-tree`.

---

## Experiment 3 — IPv4 Subnetting and VLSM Design

**Aim:** Subnet a single Class C network (192.168.10.0/24) using Variable Length Subnet Masking
to efficiently allocate address space to LANs of different sizes, and configure static routing
between two routers.

**Topology**

```
   LAN1 (/26)                                          LAN3 (/28)
192.168.10.0/26                                     192.168.10.96/28
      |                                                     |
   +--+--+        WAN Link (/30)                        +--+--+
   |  SW  |    192.168.10.112/30                         |  SW  |
   +--+--+   .113                    .114                +--+--+
      |        +------+          +------+                    |
     PC1 ------|  R1  |==========|  R2  |------------------- PC3
                +------+  Serial +--+---+
                   |  G0/0                 G0/0 |
                   |  .1 (LAN1)                  |
                   |                             |
              +----+----+                        |
              |   SW    |                        |
              +----+----+                        |
                   |                              LAN2 attaches to R1 G0/1
                  PC2 (LAN2, 192.168.10.64/27)
```

**VLSM Allocation**

| Subnet | Network | Mask | Usable Hosts | Broadcast |
|---|---|---|---|---|
| LAN1 | 192.168.10.0/26 | 255.255.255.192 | .1 – .62 | .63 |
| LAN2 | 192.168.10.64/27 | 255.255.255.224 | .65 – .94 | .95 |
| LAN3 | 192.168.10.96/28 | 255.255.255.240 | .97 – .110 | .111 |
| WAN (R1–R2) | 192.168.10.112/30 | 255.255.255.252 | .113 – .114 | .115 |

**IP Addressing**

| Device | Interface | IP Address | Mask | Gateway |
|---|---|---|---|---|
| R1 | G0/0 (LAN1) | 192.168.10.1 | /26 | — |
| R1 | G0/1 (LAN2) | 192.168.10.65 | /27 | — |
| R1 | Serial0/0/0 (WAN) | 192.168.10.113 | /30 | — |
| R2 | Serial0/0/0 (WAN) | 192.168.10.114 | /30 | — |
| R2 | G0/0 (LAN3) | 192.168.10.97 | /28 | — |
| PC1 | LAN1 | 192.168.10.2 | /26 | 192.168.10.1 |
| PC2 | LAN2 | 192.168.10.66 | /27 | 192.168.10.65 |
| PC3 | LAN3 | 192.168.10.98 | /28 | 192.168.10.97 |

**Key Config Files:** [`R1.cfg`](Experiment_3_VLSM/R1.cfg) (`ip route 192.168.10.96 255.255.255.240 192.168.10.114`),
[`R2.cfg`](Experiment_3_VLSM/R2.cfg) (`ip route` back to LAN1 and LAN2 via `192.168.10.113`)

**Result:** All three LANs successfully reach each other across the R1–R2 serial link. Static
routes on both routers correctly point to the remote VLSM subnets, verified with `ping` and
`show ip route`.

---

## Experiment 4 — IPv6 Configuration and SLAAC

**Aim:** Enable IPv6 unicast routing, assign IPv6 addresses to router interfaces, configure a
static IPv6 route between two routers, and allow end devices to auto-configure their IPv6
address via SLAAC using Router Advertisement (RA) messages.

**Topology**

```
                2001:DB8:1:1::/64            2001:DB8:1:12::/64        2001:DB8:1:2::/64
   PC1 ---- SW1 ---- G0/0            R1            G0/1 ======== G0/0            R2            G0/1 ---- SW2 ---- PC2
                     ::1                                          ::1      ::2                    ::1
                (SLAAC)                                                                        (SLAAC)
```

**IP Addressing**

| Device | Interface | IPv6 Address | Assignment |
|---|---|---|---|
| R1 | G0/0 | 2001:DB8:1:1::1/64 | Static |
| R1 | G0/1 | 2001:DB8:1:12::1/64 | Static |
| R2 | G0/0 | 2001:DB8:1:12::2/64 | Static |
| R2 | G0/1 | 2001:DB8:1:2::1/64 | Static |
| PC1 | — | Auto-configured | SLAAC (via R1 G0/0 RA) |
| PC2 | — | Auto-configured | SLAAC (via R2 G0/1 RA) |

**Key Config Files:** [`R1.cfg`](Experiment_4_IPv6_SLAAC/R1.cfg) / [`R2.cfg`](Experiment_4_IPv6_SLAAC/R2.cfg)
(`ipv6 unicast-routing`, interface `ipv6 address`, `ipv6 route 2001:DB8:1:X::/64 <next-hop>`)

**Result:** PC1 and PC2 automatically acquire an IPv6 address, prefix length, and default
gateway from Router Advertisement messages sent by their respective routers. End-to-end IPv6
connectivity across R1 ↔ R2 is confirmed via `ping` and `show ipv6 route`.

---

## Experiment 5 — Static and Default Route Configuration

**Aim:** Interconnect three routers using point-to-point links, configure static routes for
specific remote networks, and configure a default route (`0.0.0.0/0`) on the edge router to
reach unadvertised destinations.

**Topology**

```
 192.168.1.0/24                10.0.12.0/30              10.0.23.0/30            192.168.3.0/24
       |                              |                          |                       |
   +---+---+  G0/0        G0/1   +----+----+   G0/0        G0/1   +----+----+   G0/0      +---+---+
   |  SW1  |--------------| R1  |-------------|  R2  |-------------|  R3  |------------|  SW3  |
   +---+---+   .1          +----+   .1     .2  +--+--+  .1    .2   +--+--+   .1         +---+---+
       |                                          | G0/2                       .1        |
      PC1                                     192.168.2.0/24 --- SW2 --- PC2           PC3
   192.168.1.2                                     +---+---+
                                                     |
                                                    PC2
                                              192.168.2.2
```

**IP Addressing**

| Device | Interface | IP Address | Mask | Notes |
|---|---|---|---|---|
| R1 | G0/0 | 192.168.1.1 | /24 | LAN toward PC1 |
| R1 | G0/1 | 10.0.12.1 | /30 | Link to R2 |
| R2 | G0/0 | 10.0.12.2 | /30 | Link to R1 |
| R2 | G0/1 | 10.0.23.1 | /30 | Link to R3 |
| R2 | G0/2 | 192.168.2.1 | /24 | LAN toward PC2 |
| R3 | G0/0 | 10.0.23.2 | /30 | Link to R2 |
| R3 | G0/1 | 192.168.3.1 | /24 | LAN toward PC3 |
| PC1 | — | 192.168.1.2 | /24 | GW 192.168.1.1 |
| PC2 | — | 192.168.2.2 | /24 | GW 192.168.2.1 |
| PC3 | — | 192.168.3.2 | /24 | GW 192.168.3.1 |

**Routing:** R1 and R2 use explicit static routes for every remote LAN
(e.g. R1 → `ip route 192.168.2.0 255.255.255.0 10.0.12.2`). R3, being the edge router with only
one upstream path, uses a single **default route**: `ip route 0.0.0.0 0.0.0.0 10.0.23.1`.

**Key Config Files:** [`R1.cfg`](Experiment_5_Static_Default_Routing/R1.cfg),
[`R2.cfg`](Experiment_5_Static_Default_Routing/R2.cfg),
[`R3.cfg`](Experiment_5_Static_Default_Routing/R3.cfg) (`ip route 0.0.0.0 0.0.0.0 10.0.23.1`)

**Result:** All three LANs (192.168.1.0/24, 192.168.2.0/24, 192.168.3.0/24) achieve full
end-to-end reachability. R3's default route correctly forwards all non-local traffic toward
R2/R1, verified via `ping` and `tracert`.

---

## How to Use

**Experiments 1–4 (Packet Tracer file included):**
Simply open the provided `.pkt` file in Cisco Packet Tracer — the topology, device placement,
and cabling are pre-built. Device configurations are already applied; use the corresponding
`verification.txt` commands to inspect the running state.

- Experiment 1 → [`first.pkt`](Experiment_1_VLAN_InterVLAN/first.pkt)
- Experiment 2 → [`second.pkt`](Experiment_2_STP/second.pkt)
- Experiment 3 → [`three.pkt`](Experiment_3_VLSM/three.pkt)
- Experiment 4 → [`four.pkt`](Experiment_4_IPv6_SLAAC/four.pkt)

**Experiment 5 (no Packet Tracer file):**
Build the topology manually in Packet Tracer using the devices and cabling described above,
then paste the commands from each `R*.cfg` file into that device's CLI (in `enable` /
`configure terminal` mode) to reproduce the configuration.

---

## Notes

- `output.txt` in each experiment folder documents the **expected outcome as per the lab
  manual** (e.g. expected `ping` success, expected STP port states) — it is **not** a live
  packet/terminal capture from a running session.
- All IP addressing and routing values in this README are taken directly from each
  experiment's `pc_addressing.txt` / `addressing.txt` and `*.cfg` files; refer to those files
  for the authoritative configuration.

---

## Author

**Yaswanth**
IT-4501 — NRSIP Lab
B.Tech, Information Technology
