# IK2215 – ISP108 Test Plan

This document lists every test we run on the AS108 implementation, where to run it, which
commands to use, and what the correct result looks like. Tick the box and write a short note
next to each test when you run it.

---

## 0. Before you start

**Start the lab** (from `~/IK2215/IK2215-project-isp108/project`):

```
kathara lclean
kathara lstart --noterminals
```

Wait about **60 seconds** for OSPF, BGP and DHCP to settle. Open a shell on any device with:

```
kathara connect <device>        # e.g. kathara connect as108r1
vtysh -c "<command>"            # FRR commands on routers
```

**Baseline:** run the course verification script first. Every run of this plan should start
from a clean `lclean` / `lstart` and a passing verification script.

- [ ] Verification script passes — notes:

---

## Reference tables

### Our devices

| Device | Interface | IP address | Connected to |
|---|---|---|---|
| as108r1 | eth0 | 1.0.0.5/31 | AS1 (as1r1, 1.0.0.4) |
| as108r1 | eth1 | 1.108.0.7/31 | r4 |
| as108r1 | eth2 | 1.108.0.0/31 | r2 |
| as108r1 | eth3 | 1.108.0.8/31 | r3 |
| as108r1 | dummy0 | 1.108.3.1/32 | – |
| as108r2 | eth0 | 2.21.0.1/31 | AS21 (as21r1, 2.21.0.0) |
| as108r2 | eth1 | 1.108.0.2/31 | r3 |
| as108r2 | eth2 | 1.108.0.1/31 | r1 |
| as108r2 | dummy0 | 1.108.3.2/32 | – |
| as108r3 | eth0 | 1.108.1.1/24 | server LAN |
| as108r3 | eth1 | 1.108.0.3/31 | r2 |
| as108r3 | eth2 | 1.108.0.4/31 | r4 |
| as108r3 | eth3 | 1.108.0.9/31 | r1 |
| as108r3 | dummy0 | 1.108.3.3/32 | – |
| as108r4 | eth0 | 1.108.2.1/24 | client LAN |
| as108r4 | eth1 | 1.108.0.6/31 | r1 |
| as108r4 | eth2 | 1.108.0.5/31 | r3 |
| as108r4 | dummy0 | 1.108.3.4/32 | – |
| as108s1 | eth0 | 1.108.1.2/24 | DNS (ns.isp108.lab) |
| as108s2 | eth0 | 1.108.1.4/24 | Web (www.isp108.lab) |
| as108s3 | eth0 | 1.108.1.3/24 | DHCP (dhcpd.isp108.lab) |
| as108c1, as108c2 | eth0 | 1.108.2.x/24 (DHCP) | dhcp-x.clients.isp108.lab |

### Internal links, interfaces and OSPF costs

| Link | Interfaces | OSPF cost |
|---|---|---|
| r1–r4 | r1 eth1 ↔ r4 eth1 | 10 |
| r1–r2 | r1 eth2 ↔ r2 eth2 | 30 |
| r1–r3 | r1 eth3 ↔ r3 eth3 | 10 |
| r2–r3 | r2 eth1 ↔ r3 eth1 | 25 |
| r3–r4 | r3 eth2 ↔ r4 eth2 | 10 |

All internal links use `ip ospf network point-to-point`. r3 eth0 and r4 eth0 are passive.

### External links

| Link | Our side | Their side |
|---|---|---|
| AS1 (primary) | as108r1 eth0 | as1r1 eth4 |
| AS21 (backup + direct) | as108r2 eth0 | as21r1 eth0 |
| AS21 ↔ AS2 (AS21's primary) | – | as21r1 eth1 ↔ as2r1 eth4 |

### External test targets

| AS | Kathará device | IP | Name |
|---|---|---|---|
| AS1 | as1h2 | 1.0.1.3 | ns.isp1.lab |
| AS1 (root DNS) | as1h1 | 1.0.1.2 | – |
| AS2 | as2h2 | 2.0.1.3 | ns.isp2.lab |
| AS2 (.lab DNS) | as2h1 | 2.0.1.2 | ns.lab |
| AS3 | as3h2 | 3.0.1.3 | ns.isp3.lab |
| AS12 | as12h1 | 1.12.1.2 | ns.isp12.lab |
| AS21 | as21h1 | 2.21.1.2 | ns.isp21.lab |
| AS22 | as22h1 | 2.22.1.2 | ns.isp22.lab |

### Traceroute hop decoder (which router owns which IP)

| Router | IP addresses you may see in a traceroute |
|---|---|
| our r1 | 1.108.0.7, 1.108.0.0, 1.108.0.8, 1.0.0.5, 1.108.3.1 |
| our r2 | 1.108.0.2, 1.108.0.1, 2.21.0.1, 1.108.3.2 |
| our r3 | 1.108.1.1, 1.108.0.3, 1.108.0.4, 1.108.0.9, 1.108.3.3 |
| our r4 | 1.108.2.1, 1.108.0.6, 1.108.0.5, 1.108.3.4 |
| AS1 router | 1.0.0.4, 1.0.0.0, 1.0.0.2, 3.0.0.2, 1.0.1.1 |
| AS2 router | 1.0.0.1, 2.0.0.2, 2.0.0.4, 3.0.0.6, 2.0.1.1 |
| AS3 router | 3.0.0.1, 3.0.0.5, 3.0.1.1 |
| AS12 router | 1.0.0.3, 1.12.1.1 |
| AS21 router | 2.21.0.0, 2.0.0.5, 2.21.1.1 |
| AS22 router | 2.0.0.3, 2.22.1.1 |

Tip: run `traceroute` without `-n` from inside our network. Our hops then show DNS names such
as `eth2.r3.isp108.lab`, which also tests reverse DNS.

### Expected primary and secondary paths (from the design report)

The secondary path is what should happen when the **first link of the primary path** fails.

| From → To | Primary | Secondary | Link to fail |
|---|---|---|---|
| r1 → clients | r1 → r4 | r1 → r3 → r4 | r1–r4 |
| clients → r1 | r4 → r1 | r4 → r3 → r1 | r1–r4 |
| r1 → servers | r1 → r3 | r1 → r4 → r3 | r1–r3 |
| servers → r1 | r3 → r1 | r3 → r4 → r1 | r1–r3 |
| r2 → clients | r2 → r3 → r4 | r2 → r1 → r4 | r2–r3 |
| clients → r2 | r4 → r3 → r2 | r4 → r1 → r2 | r3–r4 |
| r2 → servers | r2 → r3 | r2 → r1 → r3 | r2–r3 |
| servers → r2 | r3 → r2 | r3 → r1 → r2 | r2–r3 |
| clients → servers | r4 → r3 | r4 → r1 → r3 | r3–r4 |
| servers → clients | r3 → r4 | r3 → r1 → r4 | r3–r4 |

---

## How to fail and restore things

**Fail a link.** Take down **both ends** (two terminals, one per router):

```
ip link set <iface> down        # restore with: ip link set <iface> up
```

**Why both ends:** in Kathará the link is a virtual bridge. If you down only one side, the
other side does not notice:

- **OSPF:** the other router waits for the dead interval, up to **40 s**.
- **BGP:** the external neighbor keeps the session until its hold timer expires, up to
  **180 s**.

Downing one side only is a valid test, but you must wait these times before checking.

**Take a whole router offline:**

```
systemctl stop frr
for i in eth0 eth1 eth2 eth3; do ip link set $i down 2>/dev/null; done
```

**Bring it back:**

```
for i in eth0 eth1 eth2 eth3; do ip link set $i up 2>/dev/null; done
systemctl start frr
```

**After restoring anything**, wait about 60 s and re-check `show bgp summary` on r1 and r2.
If the iBGP session is stuck in `Active`, run `vtysh -c "clear bgp 1.108.3.2"` on r1.

---

## 1. Intra-domain routing (OSPF) and link failures

### 1.1 Connectivity inside our ISP (primary and secondary paths)

**Where:** as108r1, as108r2, as108c1, as108s2.

**Commands** (use c1's current IP, from `ip -4 addr show eth0` on c1):

```
# on r1
traceroute -n 1.108.2.<c1>      # r1 → clients
traceroute -n 1.108.1.4         # r1 → servers
# on r2
traceroute -n 1.108.2.<c1>      # r2 → clients
traceroute -n 1.108.1.4         # r2 → servers
# on c1
traceroute -n 1.108.3.1         # clients → r1
traceroute -n 1.108.3.2         # clients → r2
traceroute -n 1.108.1.4         # clients → servers
# on s2
traceroute -n 1.108.3.1         # servers → r1
traceroute -n 1.108.3.2         # servers → r2
traceroute -n 1.108.2.<c1>      # servers → clients
```

Then fail the "Link to fail" from the paths table above and repeat the matching traceroute.

**Expected:** hops match the **Primary** column normally and the **Secondary** column during
the failure. Use the hop decoder to map IPs to routers.

- [ ] All primary paths correct — notes:
- [ ] All secondary paths correct — notes:

### 1.2 OSPF neighbors

**Where:** as108r1 to as108r4.

**Command:**

```
vtysh -c "show ip ospf neighbor"
```

**Expected:** every neighbor is in state `Full/-` (the `-` means no DR on point-to-point
links). Neighbor IDs are the dummy0 addresses.

| Router | Neighbors |
|---|---|
| r1 | 1.108.3.2, 1.108.3.3, 1.108.3.4 |
| r2 | 1.108.3.1, 1.108.3.3 |
| r3 | 1.108.3.1, 1.108.3.2, 1.108.3.4 |
| r4 | 1.108.3.1, 1.108.3.3 |

- [ ] Result — notes:

### 1.3 OSPF costs, network type and passive interfaces

**Where:** as108r1 to as108r4.

**Commands:**

```
vtysh -c "show ip ospf interface" | grep -E "^[a-z]|Cost|Network Type|Passive"
```

**Expected:**

- Costs match the internal links table on both ends of every link.
- `Network Type POINTOPOINT` on all internal links.
- On r3 eth0 and r4 eth0: `No Hellos (Passive interface)`.
- OSPF is not enabled on r1 eth0 or r2 eth0.

- [ ] Result — notes:

### 1.4 Routing tables (no ECMP)

**Where:** as108r1 to as108r4.

**Commands:**

```
vtysh -c "show ip route ospf"
ip route | grep -c nexthop
```

**Expected:** every OSPF route has exactly **one** next hop, and `grep -c nexthop` prints `0`.

**Known open issue:** with the current costs, r1, r3 and r4 still have two next hops for the
/31 of the link they are not on (for example r1 → 1.108.0.4/31). Check with
`vtysh -c "show ip route 1.108.0.4/31"`.

- [ ] Result — notes:

### 1.5 Continuous ping during each link failure

**Where:** a client or server as the pinger, plus the two routers of the link being failed.

**Commands:** keep a ping running, fail the link on both ends, watch the ping, then restore.

```
ping 1.108.1.4                  # on c1 (or the pinger listed below)
```

| Link to fail | Ping from → to | Should switch to |
|---|---|---|
| r1–r4 | as108r1: `ping 1.108.2.<c1>` | r1 → r3 → r4 |
| r1–r3 | as108r1: `ping 1.108.1.4` | r1 → r4 → r3 |
| r3–r4 | c1: `ping 1.108.1.4` | r4 → r1 → r3 |
| r2–r3 | as108r2: `ping 1.108.1.4` | r2 → r1 → r3 |
| r1–r2 | as108r1: `ping 1.108.3.2` | r1 → r3 → r2 |

**Expected:** the ping recovers after a short outage. It should take a few seconds with both
ends down, and up to about 40 s with only one end down. After restoring the link, traffic
returns to the primary path. Record the number of lost packets (`Ctrl+C` prints the summary).

- [ ] r1–r4 — lost:
- [ ] r1–r3 — lost:
- [ ] r3–r4 — lost:
- [ ] r2–r3 — lost:
- [ ] r1–r2 — lost:

### 1.6 Default route on r3 and r4

**Where:** as108r3, as108r4.

**Commands:**

```
ip route | grep default
vtysh -c "show ip ospf database external 0.0.0.0"
```

**Expected, normal operation:**

- r3: `default via 1.108.0.8 dev eth3` (towards r1).
- r4: `default via 1.108.0.7 dev eth1` (towards r1).
- The database shows two default LSAs: 1.108.3.1 with metric 10 and 1.108.3.2 with metric 20.

**Expected, r1 offline** (take it offline as described above):

- r3: `default via 1.108.0.2 dev eth1` (towards r2).
- r4: `default via 1.108.0.4 dev eth2` (via r3 to r2).
- Only the 1.108.3.2 default LSA remains.

- [ ] Normal — notes:
- [ ] r1 offline — notes:

### 1.7 AS21 prefix in OSPF

**Where:** as108r3 or as108r4.

**Commands:**

```
vtysh -c "show ip ospf database external"
vtysh -c "show ip route ospf" | grep -v " 1.108."
```

**Expected:** the external LSAs are only `0.0.0.0` (two, from r1 and r2) and `2.21.0.0`
(from 1.108.3.2). No other external prefix such as 1.0.0.0/20 or 2.0.0.0/20 appears.

- [ ] Result — notes:

---

## 2. Inter-domain routing (BGP) and external connectivity

### 2.1 BGP sessions

**Where:** as108r1, as108r2.

**Command:**

```
vtysh -c "show bgp summary"
```

**Expected:** every neighbor shows a number (prefixes received) in `State/PfxRcd`, not
`Active` or `Idle`.

- r1: 1.0.0.4 (AS1) and 1.108.3.2 (iBGP r2).
- r2: 2.21.0.0 (AS21) and 1.108.3.1 (iBGP r1).

- [ ] Result — notes:

### 2.2 iBGP stability during internal link failures

**Where:** as108r1 (watch), plus the routers of the failed link.

**Commands:** note the `Up/Down` time of neighbor 1.108.3.2, then fail the r1–r2 link on both
ends and wait about 30 s.

```
vtysh -c "show bgp summary"
vtysh -c "show ip route 1.108.3.2"
```

Repeat with the r2–r3 link.

**Expected:**

- The iBGP session stays up, and its `Up/Down` timer keeps counting without resetting.
- During the r1–r2 failure, r1 reaches 1.108.3.2 via r3 (1.108.0.9).

- [ ] r1–r2 failed — notes:
- [ ] r2–r3 failed — notes:

### 2.3 Local preference on r1 and r2

**Where:** as108r1, as108r2.

**Commands:**

```
vtysh -c "show bgp ipv4 unicast"
vtysh -c "show bgp ipv4 unicast 2.21.0.0/20"
vtysh -c "show bgp ipv4 unicast 1.0.0.0/20"
```

**Expected on r1:**

- Routes from AS1 have `LocPrf 200`.
- The best path to 2.21.0.0/20 is the iBGP route via 1.108.3.2 with local pref 300.

**Expected on r2:**

- 2.21.0.0/20 from 2.21.0.0 has local pref 300.
- All other prefixes are best via 1.108.3.1 (iBGP, local pref 200).
- The copies learned from AS21 have local pref 100 and are not best.

- [ ] Result — notes:

### 2.4 Ping and traceroute from c1 to every AS

**Where:** as108c1 (repeat from as108s2).

**Commands:**

```
for ip in 1.0.1.3 2.0.1.3 3.0.1.3 1.12.1.2 2.21.1.2 2.22.1.2; do ping -c1 -W2 $ip; done
traceroute -n 1.0.1.3     # AS1
traceroute -n 2.0.1.3     # AS2
traceroute -n 3.0.1.3     # AS3
traceroute -n 1.12.1.2    # AS12
traceroute -n 2.21.1.2    # AS21
traceroute -n 2.22.1.2    # AS22
```

**Expected:** all pings succeed.

| From | Destination | Path |
|---|---|---|
| c1 | everything except AS21 | r4 → r1 → AS1 → … |
| c1 | AS21 | r4 → r3 → r2 → AS21 (direct link) |
| s2 | everything except AS21 | r3 → r1 → AS1 → … |
| s2 | AS21 | r3 → r2 → AS21 |

- [ ] Result — notes:

### 2.5 Reverse direction (other ASes → us)

**Where:** as1h2, as2h2, as3h2, as12h1, as22h1, as21h1.

**Commands:**

```
traceroute -n 1.108.2.<c1>
traceroute -n 1.108.1.4
```

**Expected:**

- From AS1, AS2, AS3, AS12 and AS22: traffic enters through AS1 → r1 (1.0.0.5), then reaches
  r4 (clients) or r3 (servers).
- From AS21: AS21 → r2 (2.21.0.1) → r3 → … over the direct link, never through AS2 or AS1.

- [ ] Result — notes:

### 2.6 AS1 link offline

**Fail:** `ip link set eth0 down` on as108r1 **and** `ip link set eth4 down` on as1r1. If you
only down r1's side, wait up to 180 s for AS1's hold timer.

**Commands:**

```
# as1r1
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"
# as108c1
traceroute -n 1.12.1.2
# as12h1
traceroute -n 1.108.2.<c1>
```

**Expected:**

- AS1's best path to 1.108.0.0/20 becomes `2 21 108 108`.
- **Outbound** from c1: r4 → r1 → r2 → AS21 → AS2 → AS1 → AS12. The first hop is still r1,
  because r1 originates its default route with `always`.
- **Inbound** from AS12: AS1 → AS2 → AS21 → r2 → r3 → r4.

**Restore:** bring both interfaces back up.

- [ ] Result — notes:

### 2.7 AS21 link offline

**Fail:** `ip link set eth0 down` on as108r2 **and** on as21r1.

**Commands:**

```
# as21r1
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"
# as108r1
vtysh -c "show bgp ipv4 unicast 2.21.0.0/20"
# as108c1
traceroute -n 2.21.1.2
# as21h1
traceroute -n 1.108.2.<c1>
```

**Expected:**

- AS21's best path to us is `2 1 108`.
- r1's best path to AS21 is `1 2 21`.
- c1 → AS21: … → r1 → AS1 → AS2 → AS21. The path may detour r4 → r3 → r2 → r1, because r2
  still redistributes its iBGP-learned 2.21.0.0/20 into OSPF.
- AS21 → c1: AS2 → AS1 → r1 → r4.

**Restore:** bring both interfaces back up.

- [ ] Result — notes:

### 2.8 AS21's link to AS2 offline

**Fail:** `ip link set eth1 down` on as21r1 **and** `ip link set eth4 down` on as2r1.

**Commands:**

```
# as21r1
vtysh -c "show bgp ipv4 unicast 1.0.0.0/20"
# as1r1
vtysh -c "show bgp ipv4 unicast 2.21.0.0/20"
# as21h1
traceroute -n 1.0.1.3
traceroute -n 2.22.1.2
# as22h1
traceroute -n 2.21.1.2
```

**Expected:**

- AS21's best path to AS1 is `108 108 108 108 1` (through us, because it is the only path).
- AS1's best path to AS21 is `108 108 108 108 21`.
- AS21's traffic to AS1 goes AS21 → r2 → r1 → AS1.
- AS21's traffic to AS22 goes AS21 → r2 → r1 → AS1 → AS2 → AS22.
- AS22 reaches AS21 through us.

**Restore:** bring both interfaces back up.

- [ ] Result — notes:

### 2.9 Restore all links

**Commands:** after restoring, wait about 60 s and re-run 2.1, 2.4 and 3.9.

**Expected:** everything is back to the normal-operation results.

- [ ] Result — notes:

---

## 3. BGP prefix filtering and transit rules

### 3.1 AS21 → our network uses the direct link

**Where:** as21h1.

**Commands:**

```
traceroute -n 1.108.1.4
traceroute -n 1.108.2.<c1>
```

**Expected:** the first hop after AS21's router is our r2 (2.21.0.1).

- [ ] Result — notes:

### 3.2 AS21 → AS1, AS12 and AS22 do not go through us

**Where:** as21h1, with as21r1 for the BGP check.

**Commands:**

```
traceroute -n 1.0.1.3
traceroute -n 1.12.1.2
traceroute -n 2.22.1.2
# as21r1
vtysh -c "show bgp ipv4 unicast 1.0.0.0/20"
```

**Expected:**

- All three traceroutes go via AS2 (2.0.0.4), with no 1.108.x.x or 2.21.0.1 hop.
- The BGP best path is `2 1`. The path through us, `108 108 108 108 1`, is present but not
  best.

- [ ] Result — notes:

### 3.3 AS1 → our network uses the direct link

**Where:** as1h2.

**Command:**

```
traceroute -n 1.108.1.4
```

**Expected:** AS1 → r1 (1.0.0.5) → r3 → s2.

- [ ] Result — notes:

### 3.4 AS1 → AS21 goes via AS2, not through us

**Where:** as1h2, with as1r1 for the BGP check.

**Commands:**

```
traceroute -n 2.21.1.2
# as1r1
vtysh -c "show bgp ipv4 unicast 2.21.0.0/20"
```

**Expected:**

- The traceroute goes AS1 → AS2 → AS21, with no 1.0.0.5 hop.
- The BGP best path is `2 21`. The path through us, `108 108 108 108 21`, is present but not
  best.

- [ ] Result — notes:

### 3.5 Routes advertised by r1 to AS1

**Where:** as108r1, with as1r1 to see what AS1 actually received.

**Commands:**

```
# as108r1
vtysh -c "show bgp ipv4 unicast neighbors 1.0.0.4 advertised-routes"
# as1r1
vtysh -c "show bgp ipv4 unicast neighbors 1.0.0.5 received-routes"
```

**Expected:** exactly two prefixes and nothing else.

- 1.108.0.0/20 with path `108`.
- 2.21.0.0/20 with path `108 108 108 108 21`.

- [ ] Result — notes:

### 3.6 Routes advertised by r2 to AS21

**Where:** as108r2, with as21r1.

**Commands:**

```
# as108r2
vtysh -c "show bgp ipv4 unicast neighbors 2.21.0.0 advertised-routes"
# as21r1
vtysh -c "show bgp ipv4 unicast neighbors 2.21.0.1 received-routes"
```

**Expected:**

- 1.108.0.0/20 with path `108 108` (one prepend).
- All other Internet prefixes with `108 108 108 108 …` (three prepends). This is backup
  transit for AS21 only.
- No 1.108.x.x more-specifics.

- [ ] Result — notes:

### 3.7 No internal prefixes leak

**Where:** as1r1, as21r1 (also as2r1, as12r1).

**Commands:**

```
vtysh -c "show bgp ipv4 unicast" | grep 1.108
vtysh -c "show ip route" | grep 1.108
```

**Expected:** only `1.108.0.0/20`. No /24, /31 or /32 from 1.108.x.x.

- [ ] Result — notes:

### 3.8 Local pref and community on AS1

**Where:** as1r1.

**Command:**

```
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"
```

**Expected:**

- The best path is `108`, from 1.0.0.5, with `localpref 200`.
- There is **no** `Community: 1:200` line, because AS1 deletes the community after using it.

- [ ] Result — notes:

### 3.9 Best path to us from AS2, AS3, AS12 and AS22

**Where:** as2r1, as3r1, as12r1, as22r1.

**Command:**

```
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"
```

**Expected best path** (the line marked `best`):

| Router | Best path |
|---|---|
| AS2 | `1 108` |
| AS3 | `1 108` |
| AS12 | `1 108` |
| AS22 | `2 1 108` |

None of them should use the path through AS21 (`… 21 108 108`).

- [ ] Result — notes:

### 3.10 No transit for anyone else

**Where:** as2r1, as3r1, as12r1, as22r1, as1r1.

**Command:**

```
vtysh -c "show bgp ipv4 unicast" | grep "^\*>" | grep " 108 "
```

**Expected:**

- On AS2, AS3, AS12 and AS22: the only best route that contains 108 is `1.108.0.0/20`.
- On AS1: the same, and 2.21.0.0/20 is **not** best via 108.
- Traceroutes between other ASes (for example as12h1 → 2.22.1.2) never show a 1.108.x.x hop.

- [ ] Result — notes:

---

## 4. DNS service

### 4.1 Our names from inside

**Where:** as108s2, as108s3, as108c1, as108c2.

**Commands:**

```
nslookup www.isp108.lab
nslookup ns.isp108.lab
nslookup dhcpd.isp108.lab
nslookup eth2.r3.isp108.lab
nslookup dummy0.r1.isp108.lab
nslookup www            # short name, uses the search domain isp108.lab
```

**Expected:** 1.108.1.4, 1.108.1.2, 1.108.1.3, 1.108.0.4, 1.108.3.1 and 1.108.1.4. The
server shown is 1.108.1.2.

- [ ] Result — notes:

### 4.2 Our names from other ASes

**Where:** as12h1, as21h1, as1h2, as22h1.

**Commands:**

```
nslookup www.isp108.lab
nslookup ns.isp108.lab
```

**Expected:** 1.108.1.4 and 1.108.1.2, resolved through the other AS's own DNS server. This
proves the delegation `.lab` → ns.isp108.lab works.

- [ ] Result — notes:

### 4.3 External names from inside

**Where:** as108c1, as108s2.

**Commands:**

```
nslookup ns.isp1.lab
nslookup ns.isp12.lab
nslookup ns.isp21.lab
nslookup ns.lab
```

**Expected:** 1.0.1.3, 1.12.1.2, 2.21.1.2 and 2.0.1.2.

- [ ] Result — notes:

### 4.4 Reverse lookups of external IPs from inside

**Where:** as108c1, as108s2.

**Commands:**

```
nslookup 1.0.1.3
nslookup 1.12.1.2
nslookup 2.21.1.2
nslookup 3.0.1.3
```

**Expected:** a name under the owning AS's domain, for example `ns.isp12.lab`.

- [ ] Result — notes:

### 4.5 Reverse lookups of our own IPs

**Where:** inside (as108c1), and outside (as12h1).

**Commands:**

```
nslookup 1.108.1.2
nslookup 1.108.1.4
nslookup 1.108.2.<c1>
nslookup 1.108.0.4
```

**Expected:**

- `ns.isp108.lab`, `www.isp108.lab` and `dhcp-<c1>.clients.isp108.lab`.
- Router interfaces such as 1.108.0.4 resolve only if PTR records for them were added. Reverse
  lookups for our own zone are optional in the guideline.

- [ ] Result — notes:

### 4.6 Client names

**Where:** as108c1, as108c2.

**Commands:**

```
ip -4 addr show eth0                        # note the address 1.108.2.N
nslookup 1.108.2.N                          # expect dhcp-N.clients.isp108.lab
nslookup dhcp-N.clients.isp108.lab          # expect 1.108.2.N
```

- [ ] Result — notes:

### 4.7 DNS during a failure (r1 offline)

**Where:** take as108r1 offline, then test from as12h1, as21h1 and as108c1.

**Commands:**

```
# as12h1 and as21h1
nslookup www.isp108.lab
# as108c1
nslookup ns.isp1.lab
```

**Expected:** all lookups still work, because traffic uses the AS21 path. Wait for BGP to
converge first; check with `show bgp ipv4 unicast 1.108.0.0/20` on as1r1.

- [ ] Result — notes:

---

## 5. Web service

### 5.1 Website from everywhere

**Where:** as108c1, as108c2, as108s1, as108s3, as108r1 to as108r4, as1h2, as12h1, as21h1,
as22h1.

**Command:**

```
curl -s http://www.isp108.lab      # if curl is missing: wget -qO- http://www.isp108.lab
```

**Expected:** the page shows:

```
ASN: 108
NETWORK: 1.108.0.0/20
NAME1: Malte Berg
EMAIL1: maltebe@kth.se
NAME2: Georgios Georgakopoulos
EMAIL2: gege@kth.se
```

- [ ] Result — notes:

### 5.2 Website during failures

**Commands:** repeat 5.1 from as108c1 and as12h1 in each of these situations:

- the r3–r4 link down;
- the AS1 link down (2.6);
- r1 offline.

**Expected:** the page loads in every case, possibly after the convergence times described
earlier.

- [ ] r3–r4 down — notes:
- [ ] AS1 link down — notes:
- [ ] r1 offline — notes:

---

## 6. DHCP allocation and relay

### 6.1 Clients get an address at boot

**Where:** as108c1, as108c2, and as108s3 for the log.

**Commands:**

```
# as108c1 and as108c2, right after lstart
ip -4 addr show eth0
# as108s3
grep -E "DHCPDISCOVER|DHCPOFFER|DHCPREQUEST|DHCPACK" /var/log/syslog
systemctl status isc-dhcp-server | head -5
ps aux | grep dhcpd
```

**Expected:**

- Each client has an address in 1.108.2.2–254 within seconds of starting.
- The log shows DISCOVER → OFFER → REQUEST → ACK "via 1.108.2.1" for both clients.
- A `dhcpd` process is running.

- [ ] Result — notes:

### 6.2 Options received

**Where:** as108c1.

**Commands:**

```
ip route | grep default                   # expect: default via 1.108.2.1
cat /etc/resolv.conf                      # expect: nameserver 1.108.1.2, domain/search isp108.lab
cat /var/lib/dhcp/dhclient.leases         # shows routers, domain-name-servers, domain-name
```

- [ ] Result — notes:

### 6.3 Relay on r4 (tcpdump)

**Where:** as108r4 (capture), as108c2 (trigger).

**Commands:**

```
# as108r4
tcpdump -ni any 'port 67 or port 68'
# as108c2, in another terminal
dhclient -r eth0; dhclient -v eth0
```

**Expected in the capture:**

1. On eth0: `0.0.0.0.68 > 255.255.255.255.67` (the client's DISCOVER and REQUEST).
2. On eth2 (towards r3): `1.108.2.1.67 > 1.108.1.3.67` (r4 relaying to the server).
3. Back again: `1.108.1.3.67 > 1.108.2.1.67` (the server's OFFER and ACK).
4. On eth0: r4 → the client (the relayed reply).

- [ ] Result — notes:

### 6.4 DHCP renew with the r3–r4 link down

**Where:** as108r3 and as108r4 (fail the link), as108r1 (capture), as108c2 (trigger).

**Commands:** fail the r3–r4 link on both ends and wait about 10 s.

```
# as108r4
ip route get 1.108.1.3          # expect: via 1.108.0.7 dev eth1 (through r1)
# as108r1
tcpdump -ni any port 67
# as108c2
dhclient -r eth0; dhclient -v eth0
```

**Expected:** c2 gets an address again, and the relayed packets are visible on r1.

**Restore** the link afterwards.

- [ ] Result — notes:

---

## 7. Whole router offline

### 7.1 r1 offline

**Take as108r1 offline**, as described in "How to fail and restore things".

**Commands:**

```
# as108r3
vtysh -c "show ip ospf neighbor"
ip route | grep default
# as108r4
ip route | grep default
# as1r1
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"
# as108c1
traceroute -n 1.12.1.2
# as12h1
traceroute -n 1.108.2.<c1>
```

**Expected:**

- **Internal (OSPF):** r3 and r4 lose neighbor 1.108.3.1. r3's default is via 1.108.0.2 (r2),
  and r4's default is via 1.108.0.4 (r3).
- **External (BGP):** AS1's best path is `2 21 108 108`.
- **Traffic:** c1 → r4 → r3 → r2 → AS21 → AS2 → AS1 → AS12. The reverse path goes through
  AS21 → r2.

- [ ] Result — notes:

### 7.2 r2 offline

**Take as108r2 offline.**

**Commands:**

```
# as108r3 and as108r4
ip route | grep 2.21
# as21r1
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"
# as108c1
traceroute -n 2.21.1.2
# as21h1
traceroute -n 1.108.2.<c1>
```

**Expected:**

- **Internal (OSPF):** the OSPF route to 2.21.0.0/20 disappears, and r3 and r4 use the default
  via r1.
- **External (BGP):** AS21's best path is `2 1 108`.
- **Traffic:** c1 → r4 → r1 → AS1 → AS2 → AS21, and the reverse path goes via AS2 → AS1 → r1.

- [ ] Result — notes:

### 7.3 Recovery

**Commands:** bring the router back, wait about 60 s, and re-run 1.2, 2.1, 2.4 and 3.9.

**Expected:** everything is back to normal.

- [ ] Result — notes:

---

## Known issues and expected quirks

1. **ECMP on /31 transit subnets** (see 1.4). The r1–r3–r4 triangle has equal costs, so each
   router in it has two paths to the link it is not on. Still open.
2. **`isc-dhcp-server` shows `failed (dead)`** even though a `dhcpd` process serves leases.
   Probably the IPv6 part of the service script. To be checked (6.1).
3. **Test 2.6:** outbound traffic still goes via r1 first, because of
   `default-information originate always` on r1.
4. **Test 2.7:** traffic may detour through r2, because r2 redistributes its iBGP-learned
   2.21.0.0/20 into OSPF.
5. **Timers:** OSPF needs up to 40 s and eBGP up to 180 s when only one end of a link goes
   down. Wait before judging a failover test.
