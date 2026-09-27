# IK2215 – ISP108 Test Plan

This plan follows the teacher's recommended order.

- **Part A (own AS):** Step 0 checks our configuration against the course rules. Steps 1–4
  test internal routing, services, clients and internal link failures.
- **Part B (external networks):** steps 5–8 test BGP peerings, BGP policies, external link
  failures, and access to our services from other ASes.

For every test you get: **where** to run it, **how** (the exact commands), and the
**expected** result. Tick the box and add a short note when you run it.

> The teacher's key point: *it is not enough that the network works in normal operation.
> Everything must keep working when a link fails in any part of the network.* Steps 4, 7 and 8
> test exactly this.

---

## Progress overview

| Step | Description | Done by | Status |
|---|---|---|---|
| 0 | Compliance with course rules | | ☐ |
| 1 | Internal routing (OSPF) | | ☐ |
| 2 | Services work locally | | ☐ |
| 3 | Clients: DHCP, services, reachability | | ☐ |
| 4 | Internal link failures | | ☐ |
| 5 | BGP peerings (eBGP, iBGP) | | ☐ |
| 6 | BGP policies, normal operation | | ☐ |
| 7 | External link failures, router offline | | ☐ |
| 8 | Other ASes can use our DNS and WWW | | ☐ |

---

## How to use this plan

**Start the lab** (from `~/IK2215/IK2215-project-isp108/project`):

```
kathara lclean
kathara lstart --noterminals
```

Wait about **60 seconds** for OSPF, BGP and DHCP to settle. Then:

```
kathara connect <device>        # e.g. kathara connect as108r1
vtysh -c "<command>"            # FRR commands, on routers only
```

Start every test session from a fresh `lclean` / `lstart`. Resubmit to the course verification
script whenever a step is finished.

---

## Reference

### Our devices

| Device | Interface | IP address | Connected to |
|---|---|---|---|
| as108r1 | eth0 | 1.0.0.5/31 | AS1 (as1r1 eth4, 1.0.0.4) |
| as108r1 | eth1 | 1.108.0.7/31 | r4 |
| as108r1 | eth2 | 1.108.0.0/31 | r2 |
| as108r1 | eth3 | 1.108.0.8/31 | r3 |
| as108r1 | dummy0 | 1.108.3.1/32 | – |
| as108r2 | eth0 | 2.21.0.1/31 | AS21 (as21r1 eth0, 2.21.0.0) |
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
| as108s1 | eth0 | 1.108.1.2/24 | DNS, ns.isp108.lab |
| as108s2 | eth0 | 1.108.1.4/24 | Web, www.isp108.lab |
| as108s3 | eth0 | 1.108.1.3/24 | DHCP, dhcpd.isp108.lab |
| as108c1, as108c2 | eth0 | 1.108.2.x/24 (DHCP) | dhcp-x.clients.isp108.lab |

### Links and OSPF costs

| Link | Interfaces | OSPF cost | Type |
|---|---|---|---|
| r1–r4 | r1 eth1 ↔ r4 eth1 | 10 | internal |
| r1–r2 ("iBGP link") | r1 eth2 ↔ r2 eth2 | 30 | internal |
| r1–r3 | r1 eth3 ↔ r3 eth3 | 10 | internal |
| r2–r3 | r2 eth1 ↔ r3 eth1 | 25 | internal |
| r3–r4 | r3 eth2 ↔ r4 eth2 | 10 | internal |
| Primary link | r1 eth0 ↔ as1r1 eth4 | – | eBGP to AS1 |
| Private link | r2 eth0 ↔ as21r1 eth0 | – | eBGP to AS21 |
| AS21's primary | as21r1 eth1 ↔ as2r1 eth4 | – | not ours |

All internal links use `ip ospf network point-to-point`. r3 eth0 and r4 eth0 are passive.

### External test hosts

| AS | Device | IP | Name |
|---|---|---|---|
| AS1 | as1h2 | 1.0.1.3 | ns.isp1.lab |
| AS2 | as2h2 | 2.0.1.3 | ns.isp2.lab |
| AS3 | as3h2 | 3.0.1.3 | ns.isp3.lab |
| AS12 | as12h1 | 1.12.1.2 | ns.isp12.lab |
| AS21 | as21h1 | 2.21.1.2 | ns.isp21.lab |
| AS22 | as22h1 | 2.22.1.2 | ns.isp22.lab |

### Traceroute hop decoder

| Router | IP addresses you may see in a traceroute |
|---|---|
| our r1 | 1.108.0.7, 1.108.0.0, 1.108.0.8, 1.0.0.5, 1.108.3.1 |
| our r2 | 1.108.0.2, 1.108.0.1, 2.21.0.1, 1.108.3.2 |
| our r3 | 1.108.1.1, 1.108.0.3, 1.108.0.4, 1.108.0.9, 1.108.3.3 |
| our r4 | 1.108.2.1, 1.108.0.6, 1.108.0.5, 1.108.3.4 |
| AS1 | 1.0.0.4, 1.0.0.0, 1.0.0.2, 3.0.0.2, 1.0.1.1 |
| AS2 | 1.0.0.1, 2.0.0.2, 2.0.0.4, 3.0.0.6, 2.0.1.1 |
| AS3 | 3.0.0.1, 3.0.0.5, 3.0.1.1 |
| AS12 | 1.0.0.3, 1.12.1.1 |
| AS21 | 2.21.0.0, 2.0.0.5, 2.21.1.1 |
| AS22 | 2.0.0.3, 2.22.1.1 |

Tip: inside our network, run `traceroute` without `-n`. Our hops then show names like
`eth2.r3.isp108.lab`, which also tests reverse DNS.

### Expected primary and secondary internal paths

The secondary path applies when the "Link to fail" is down.

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

### Reusable check blocks

We refer to these blocks by name throughout the plan.

**[PING-ROUTERS]**: ping every router interface (run in bash):

```
for ip in 1.108.0.{0..9} 1.108.1.1 1.108.2.1 1.108.3.{1..4} 1.0.0.5 2.21.0.1; do
  ping -c1 -W1 $ip >/dev/null && echo "OK   $ip" || echo "FAIL $ip"; done
```

- 1.0.0.5 is r1's link to AS1 and 2.21.0.1 is r2's link to AS21. They are unreachable when
  that router or link is down.
- The two addresses of a failed internal link are also expected to fail.

**[PING-ASES]**: ping one host in every AS:

```
for ip in 1.0.1.3 2.0.1.3 3.0.1.3 1.12.1.2 2.21.1.2 2.22.1.2; do
  ping -c1 -W2 $ip >/dev/null && echo "OK   $ip" || echo "FAIL $ip"; done
```

**[SERVICES-IN]**: services as seen from a client or server inside our AS:

```
nslookup www.isp108.lab          # expect 1.108.1.4
nslookup ns.isp1.lab             # expect 1.0.1.3 (external name, tests recursion)
nslookup 1.12.1.2                # expect ns.isp12.lab (external reverse)
curl -s http://www.isp108.lab    # expect our page (fallback: wget -qO- http://www.isp108.lab)
```

**[SERVICES-OUT]**: our services as seen from another AS:

```
nslookup www.isp108.lab          # expect 1.108.1.4
nslookup ns.isp108.lab           # expect 1.108.1.2
nslookup 1.108.1.4               # expect www.isp108.lab
curl -s http://www.isp108.lab    # expect our page
```

**[DHCP-RENEW]**: on as108c2:

```
dhclient -r eth0; dhclient -v eth0      # expect DHCPACK and an address in 1.108.2.0/24
```

### How to fail and restore things

**Fail a link:** take down **both ends**, with one terminal per router:

```
ip link set <iface> down        # restore with: ip link set <iface> up
```

If only one end is downed, the other side does not notice, because Kathará links are virtual
bridges. OSPF then needs up to **40 s** (dead interval) and an eBGP neighbor up to **180 s**
(hold timer). That is a valid test too, but wait these times before checking.

**Router offline:**

```
systemctl stop frr
for i in eth0 eth1 eth2 eth3; do ip link set $i down 2>/dev/null; done
```

**Router back online:**

```
for i in eth0 eth1 eth2 eth3; do ip link set $i up 2>/dev/null; done
systemctl start frr
```

**After any restore:** wait about 60 s and check `vtysh -c "show bgp summary"` on r1 and r2.
If the iBGP session is stuck in `Active`, run `vtysh -c "clear bgp 1.108.3.2"` on r1.

---

# PART A – OUR OWN AS

## Step 1 – Internal routing (OSPF)

**Goal:** all routers run the IGP, and every router can reach every internal address with
deterministic paths.

### 1.1 OSPF neighbors

**Where:** as108r1 to as108r4.

**How:**

```
vtysh -c "show ip ospf neighbor"
```

**Expected:** every neighbor is `Full/-` (point-to-point, no DR).

| Router | Neighbors |
|---|---|
| r1 | 1.108.3.2, 1.108.3.3, 1.108.3.4 |
| r2 | 1.108.3.1, 1.108.3.3 |
| r3 | 1.108.3.1, 1.108.3.2, 1.108.3.4 |
| r4 | 1.108.3.1, 1.108.3.3 |

- [ ] Result — notes:

### 1.2 Costs, network type and passive interfaces

**Where:** as108r1 to as108r4.

**How:**

```
vtysh -c "show ip ospf interface" | grep -E "^[a-z]|Cost|Network Type|Passive"
```

**Expected:**

- Costs match the links table on **both ends** of every link.
- `POINTOPOINT` on internal links.
- r3 eth0 and r4 eth0 show `No Hellos (Passive interface)`.

- [ ] Result — notes:

### 1.3 Routing tables and no ECMP

**Where:** as108r1 to as108r4.

**How:**

```
vtysh -c "show ip route ospf"
ip route | grep -c nexthop
```

**Expected:**

- Routes exist for 1.108.1.0/24, 1.108.2.0/24, all /31 links and all four dummy0 /32s.
- Each route has **one** next hop, so `grep -c nexthop` prints `0`.
- **Known open issue:** see the end of this document. Check it with
  `vtysh -c "show ip route 1.108.0.4/31"` on r1.

- [ ] Result — notes:

### 1.4 Every router reaches every internal address

**Where:** as108r1 to as108r4.

**How:** run **[PING-ROUTERS]**.

**Expected:** every `1.108.x.x` address is OK. 1.0.0.5 and 2.21.0.1 are also OK, via r1 and
r2.

- [ ] Result — notes:

### 1.5 Primary paths

**Where:** as108r1 and as108r2 (to clients and servers), as108c1 and as108s2 (to r1 and r2),
as108c1 (to servers).

**How:**

```
# r1 and r2
traceroute -n 1.108.2.<c1>     # to clients
traceroute -n 1.108.1.4        # to servers
# c1 and s2
traceroute -n 1.108.3.1        # to r1
traceroute -n 1.108.3.2        # to r2
# c1
traceroute -n 1.108.1.4        # clients → servers
```

**Expected:** the hops match the **Primary** column of the paths table. Use the hop decoder
to map IPs to routers.

- [ ] Result — notes:

### 1.6 Default route and the only redistributed prefix

**Where:** as108r3, as108r4.

**How:**

```
ip route | grep default
vtysh -c "show ip ospf database external"
```

**Expected:**

- r3's default is via 1.108.0.8 (r1) and r4's default is via 1.108.0.7 (r1).
- The only external LSAs are `0.0.0.0` twice (from 1.108.3.1 with metric 10, and from
  1.108.3.2 with metric 20) and `2.21.0.0` (from 1.108.3.2).

- [ ] Result — notes:

---

## Step 2 – Services work locally

**Goal:** each server's service works on the server itself, before we involve the network.

### 2.1 DNS on s1

**Where:** as108s1.

**How:**

```
systemctl status named | head -3
named-checkconf -z /etc/bind/named.conf
dig @127.0.0.1 www.isp108.lab +short          # expect 1.108.1.4
dig @127.0.0.1 dhcp-5.clients.isp108.lab +short   # expect 1.108.2.5
dig @127.0.0.1 -x 1.108.1.2 +short            # expect ns.isp108.lab.
dig @127.0.0.1 ns.isp1.lab +short             # expect 1.0.1.3 (recursion; may need a 2nd try)
```

**Expected:**

- `named` is `active (running)`.
- All zones load without errors: isp108.lab, 108.1.in-addr.arpa and the localhost zones.

- [ ] Result — notes:

### 2.2 Web on s2

**Where:** as108s2.

**How:**

```
systemctl status apache2 | head -3
curl -s http://127.0.0.1
```

**Expected:** `apache2` is `active (running)`, and the page shows ASN 108, NETWORK
1.108.0.0/20, and both names and emails.

- [ ] Result — notes:

### 2.3 DHCP server on s3

**Where:** as108s3.

**How:**

```
dhcpd -t -cf /etc/dhcp/dhcpd.conf       # config syntax test
ps aux | grep dhcpd | grep -v grep
systemctl status isc-dhcp-server | head -5
```

**Expected:**

- The syntax test prints no errors.
- A `dhcpd` process is running.
- **Known open issue:** the status may show `failed (dead)` while `dhcpd` is running. See the
  end of this document.

- [ ] Result — notes:

---

## Step 3 – Clients

**Goal:** clients get their configuration through the relay, can use every service, and can
ping every interface of all routers.

### 3.1 Clients get an address at boot

**Where:** as108c1, as108c2, and as108s3 for the log.

**How:**

```
# c1 and c2, right after lstart
ip -4 addr show eth0
# s3
grep -E "DHCPDISCOVER|DHCPOFFER|DHCPREQUEST|DHCPACK" /var/log/syslog
```

**Expected:**

- Each client has an address in 1.108.2.2–254 within seconds of starting.
- The log shows DISCOVER → OFFER → REQUEST → ACK "via 1.108.2.1" for both clients.

- [ ] Result — notes:

### 3.2 Options received

**Where:** as108c1, as108c2.

**How:**

```
ip route | grep default           # expect: default via 1.108.2.1
cat /etc/resolv.conf              # expect: nameserver 1.108.1.2, domain/search isp108.lab
cat /var/lib/dhcp/dhclient.leases
```

- [ ] Result — notes:

### 3.3 Relay path (tcpdump)

**Where:** as108r4 (capture), as108c2 (trigger).

**How:**

```
# r4
tcpdump -ni any 'port 67 or port 68'
# c2, in a second terminal
dhclient -r eth0; dhclient -v eth0
```

**Expected in the capture:**

1. On eth0: `0.0.0.0.68 > 255.255.255.255.67` (client broadcast).
2. On eth2: `1.108.2.1.67 > 1.108.1.3.67` (relayed to the server).
3. Back again: `1.108.1.3.67 > 1.108.2.1.67` (server reply).
4. On eth0: r4 → the client.

- [ ] Result — notes:

### 3.4 Clients can use all services

**Where:** as108c1, as108c2.

**How:** run **[SERVICES-IN]**, plus the lookups below. Use your own client's address in place
of `<c1>`.

```
nslookup www                          # short name via the search domain → 1.108.1.4
nslookup dhcpd.isp108.lab             # → 1.108.1.3
nslookup eth2.r3.isp108.lab           # → 1.108.0.4
nslookup 1.108.2.<c1>                 # → dhcp-<c1>.clients.isp108.lab
nslookup dhcp-<c1>.clients.isp108.lab # → 1.108.2.<c1>
```

**Expected:** every answer is as shown.

- [ ] Result — notes:

### 3.5 Clients can ping every router interface

**Where:** as108c1, as108c2.

**How:** run **[PING-ROUTERS]**.

**Expected:** every address is OK.

- [ ] Result — notes:

---

## Step 4 – Internal link failures

**Goal:** break each internal link, **one at a time**. All services must keep working, and all
clients must still reach them.

### Procedure (repeat for each of the 5 internal links)

1. **Continuous ping.** On c1, start `ping 1.108.1.4` and keep it running.
2. **Fail the link.** Take down both ends of the link, as in the table below.
3. **Wait.** Give OSPF about 10 s to converge.
4. **Check the secondary paths.** Repeat the traceroutes from 1.5 that involve this link. They
   must follow the **Secondary** column.
5. **Check services from the client.** On c1, run **[SERVICES-IN]** and **[PING-ROUTERS]**.
   The only expected failures are the two addresses of the failed link.
6. **Check DHCP.** On c2, run **[DHCP-RENEW]**.
7. **Check iBGP.** On r1, run `vtysh -c "show bgp summary"`. The iBGP session must stay up,
   with its `Up/Down` timer not reset.
8. **Restore and record.** Restore the link, stop the ping, and write down how many packets
   were lost.

| Link | Commands to fail it | Traceroutes to repeat | Extra check |
|---|---|---|---|
| r1–r4 | r1: `ip link set eth1 down`, r4: `ip link set eth1 down` | r1 → clients, clients → r1 | c1 → Internet now goes r4 → r3 → r1 |
| r1–r3 | r1: `ip link set eth3 down`, r3: `ip link set eth3 down` | r1 → servers, servers → r1 | s2 → Internet now goes r3 → r4 → r1 |
| r3–r4 | r3: `ip link set eth2 down`, r4: `ip link set eth2 down` | clients ↔ servers, clients → r2 | DHCP relay now travels r4 → r1 → r3 |
| r2–r3 | r2: `ip link set eth1 down`, r3: `ip link set eth1 down` | r2 → clients, r2 ↔ servers | c1 → AS21 now goes r4 → r1 → r2 |
| r1–r2 | r1: `ip link set eth2 down`, r2: `ip link set eth2 down` | r1 → r2 (`traceroute 1.108.3.2` on r1) | r1 → r2 now goes via r3; iBGP stays up |

**Results:**

- [ ] r1–r4 — lost packets: / services OK: / DHCP OK: / iBGP up:
- [ ] r1–r3 — lost packets: / services OK: / DHCP OK: / iBGP up:
- [ ] r3–r4 — lost packets: / services OK: / DHCP OK: / iBGP up:
- [ ] r2–r3 — lost packets: / services OK: / DHCP OK: / iBGP up:
- [ ] r1–r2 — lost packets: / services OK: / DHCP OK: / iBGP up:

---

# PART B – INCORPORATING EXTERNAL NETWORKS

## Step 5 – BGP peerings

**Goal:** eBGP learns all routes, and the iBGP speakers exchange routes that are valid and
installed.

### 5.1 eBGP sessions and routes learned

**Where:** as108r1, as108r2.

**How:**

```
vtysh -c "show bgp summary"
# r1
vtysh -c "show bgp ipv4 unicast neighbors 1.0.0.4 routes"
# r2
vtysh -c "show bgp ipv4 unicast neighbors 2.21.0.0 routes"
```

**Expected:**

- r1 has 1.0.0.4 (AS1) established; r2 has 2.21.0.0 (AS21) established.
- Each receives about **6 prefixes**: 1.0.0.0/20, 1.12.0.0/20, 2.0.0.0/20, 2.21.0.0/20,
  2.22.0.0/20 and 3.0.0.0/20.

- [ ] Result — notes:

### 5.2 iBGP session and exchanged routes

**Where:** as108r1, as108r2.

**How:**

```
# r1
vtysh -c "show bgp ipv4 unicast neighbors 1.108.3.2 routes"
# r2
vtysh -c "show bgp ipv4 unicast neighbors 1.108.3.1 routes"
```

**Expected:**

- The session is established between the dummy0 addresses, 1.108.3.1 ↔ 1.108.3.2.
- r1 receives 2 prefixes from r2: 1.108.0.0/20 and 2.21.0.0/20.
- r2 receives about 6 prefixes from r1: the 5 learned from AS1 plus 1.108.0.0/20.

- [ ] Result — notes:

### 5.3 iBGP routes are valid and installed

**Where:** as108r1, as108r2.

**How:**

```
vtysh -c "show bgp nexthop"
vtysh -c "show bgp ipv4 unicast 2.21.0.0/20"    # on r1
vtysh -c "show bgp ipv4 unicast 1.0.0.0/20"     # on r2
vtysh -c "show ip route bgp"
```

**Expected:**

- The next hops (1.108.3.2 on r1, 1.108.3.1 on r2) are shown as **valid**, resolved through
  OSPF. This works because of `next-hop-self`.
- The routes show `valid, internal, best`.
- They appear as `B>*` in the routing table.

- [ ] Result — notes:

---

## Step 6 – BGP policies in normal operation

### 6.1 Local preference

**Where:** as108r1, as108r2.

**How:**

```
vtysh -c "show bgp ipv4 unicast"
```

**Expected:**

- **r1:** routes from AS1 have LocPrf **200**. The best path to 2.21.0.0/20 is via iBGP from
  r2, with LocPrf **300**.
- **r2:** 2.21.0.0/20 from AS21 has **300**. Everything else is best via r1 (iBGP, 200). The
  copies learned from AS21 have 100 and are not best.

- [ ] Result — notes:

### 6.2 Outbound traffic

**Where:** as108c1 and as108s2.

**How:** run **[PING-ASES]**, then:

```
traceroute -n 1.0.1.3; traceroute -n 2.0.1.3; traceroute -n 3.0.1.3
traceroute -n 1.12.1.2; traceroute -n 2.21.1.2; traceroute -n 2.22.1.2
```

**Expected:**

- Everything exits through **r1 → AS1**, except AS21.
- AS21 traffic goes **r3 → r2 → AS21** over the direct link.

- [ ] Result — notes:

### 6.3 Inbound traffic

**Where:** as1h2, as2h2, as3h2, as12h1, as22h1, as21h1.

**How:**

```
traceroute -n 1.108.2.<c1>
traceroute -n 1.108.1.4
```

**Expected:**

- From every AS except AS21: traffic enters through **AS1 → r1 (1.0.0.5)**.
- From AS21: traffic enters over the **direct link to r2 (2.21.0.1)**.

- [ ] Result — notes:

### 6.4 What r1 advertises to AS1

**Where:** as108r1, then as1r1 to see what AS1 actually received.

**How:**

```
# r1
vtysh -c "show bgp ipv4 unicast neighbors 1.0.0.4 advertised-routes"
# as1r1
vtysh -c "show bgp ipv4 unicast neighbors 1.0.0.5 received-routes"
```

**Expected:** exactly two prefixes.

- 1.108.0.0/20 with path `108`.
- 2.21.0.0/20 with path `108 108 108 108 21`.

- [ ] Result — notes:

### 6.5 What r2 advertises to AS21

**Where:** as108r2, then as21r1.

**How:**

```
# r2
vtysh -c "show bgp ipv4 unicast neighbors 2.21.0.0 advertised-routes"
# as21r1
vtysh -c "show bgp ipv4 unicast neighbors 2.21.0.1 received-routes"
```

**Expected:**

- 1.108.0.0/20 with path `108 108`.
- All other prefixes with `108 108 108 108 …`.
- No more-specific 1.108.x.x prefixes.

- [ ] Result — notes:

### 6.6 No internal prefixes leak

**Where:** as1r1, as21r1, as2r1, as12r1.

**How:**

```
vtysh -c "show bgp ipv4 unicast" | grep 1.108
```

**Expected:** only `1.108.0.0/20`, with no /24, /31 or /32 from 1.108.x.x.

- [ ] Result — notes:

### 6.7 AS1's view of our prefix

**Where:** as1r1.

**How:**

```
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"
```

**Expected:**

- The best path is `108`, with `localpref 200`.
- There is **no** `Community:` line, because AS1 removes 1:200 after using it.

- [ ] Result — notes:

### 6.8 Other ASes reach us via AS1

**Where:** as2r1, as3r1, as12r1, as22r1.

**How:**

```
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"
```

**Expected best path:**

| Router | Best path |
|---|---|
| AS2 | `1 108` |
| AS3 | `1 108` |
| AS12 | `1 108` |
| AS22 | `2 1 108` |

None of them should use the path through AS21 (`21 108 108`).

- [ ] Result — notes:

### 6.9 The AS21 relationship

**Where:** as21r1, as1r1, as21h1, as1h2.

**How:**

```
# as21r1
vtysh -c "show bgp ipv4 unicast 1.108.0.0/20"   # expect best: 108 108 (direct)
vtysh -c "show bgp ipv4 unicast 1.0.0.0/20"     # expect best: 2 1 (not via 108)
# as1r1
vtysh -c "show bgp ipv4 unicast 2.21.0.0/20"    # expect best: 2 21 (not via 108)
# as21h1
traceroute -n 1.0.1.3; traceroute -n 1.12.1.2; traceroute -n 2.22.1.2   # via AS2, no 1.108/2.21.0.1 hop
# as1h2
traceroute -n 2.21.1.2                          # via AS2, no 1.0.0.5 hop
```

**Expected:** as shown in the comments. AS21 uses the direct link only to reach us, and
neither AS1 nor AS21 uses us to reach the other.

- [ ] Result — notes:

### 6.10 No transit for anyone else

**Where:** as1r1, as2r1, as3r1, as12r1, as22r1.

**How:**

```
vtysh -c "show bgp ipv4 unicast" | grep "^\*>" | grep " 108 "
```

**Expected:**

- The only best route that contains 108 is `1.108.0.0/20`.
- Traceroutes between other ASes (for example as12h1 → 2.22.1.2) never show a 1.108.x.x or
  2.21.0.1 hop.

- [ ] Result — notes:

---

## Step 7 – External link failures and router offline

The teacher requires breaking the **primary link**, the **private link** and the **iBGP link**.
We also test AS21's own primary link, and each border router going completely offline.

### Checks to run in every scenario

1. **BGP state.** Run `vtysh -c "show bgp summary"` on r1 and r2, and check the best path to
   our prefix on as1r1 and as21r1.
2. **Outbound.** On c1, run **[PING-ASES]**, then `traceroute -n 1.12.1.2` and
   `traceroute -n 2.21.1.2`.
3. **Inbound.** On as12h1 and as21h1, run `traceroute -n 1.108.2.<c1>`.
4. **Services inside.** On c1, run **[SERVICES-IN]**.
5. **Restore and wait.** Restore everything, wait about 60 s, and confirm that the Step 6
   results are back.

### 7.1 Primary link down (AS1)

**Fail:** r1: `ip link set eth0 down`, and as1r1: `ip link set eth4 down`.

**Expected:**

- r1's session to AS1 goes down, and as1r1's best path to us becomes `2 21 108 108`.
- **c1 → AS12:** r4 → r1 → r2 → AS21 → AS2 → AS1 → AS12. The first hop is still r1, because
  r1 originates its default route with `always`.
- **c1 → AS21:** r4 → r3 → r2 → AS21, unchanged.
- **AS12 → c1:** AS1 → AS2 → AS21 → r2 → r3 → r4.
- All services work.

- [ ] Result — notes:

### 7.2 Private link down (AS21)

**Fail:** r2: `ip link set eth0 down`, and as21r1: `ip link set eth0 down`.

**Expected:**

- as21r1's best path to us is `2 1 108`, and r1's best path to AS21 is `1 2 21`.
- **c1 → AS21:** … → r1 → AS1 → AS2 → AS21. This may detour through r2, because r2 still
  redistributes its iBGP copy of 2.21.0.0/20.
- **AS21 → c1:** AS2 → AS1 → r1 → r4.
- All services work.

- [ ] Result — notes:

### 7.3 iBGP link down (r1–r2)

**Fail:** r1: `ip link set eth2 down`, and r2: `ip link set eth2 down`.

**Expected:**

- The iBGP session **stays up**, with its `Up/Down` timer not reset. r1 reaches 1.108.3.2
  via r3 (`vtysh -c "show ip route 1.108.3.2"` shows it via 1.108.0.9).
- External paths are unchanged: c1 → AS12 via r1, and c1 → AS21 via r3 → r2.
- r2's own traffic to AS1 goes r2 → r3 → r1 → AS1 (`traceroute -n 1.0.1.3` on r2).
- All services work.

- [ ] Result — notes:

### 7.4 AS21's link to AS2 down

**Fail:** as21r1: `ip link set eth1 down`, and as2r1: `ip link set eth4 down`.

**Expected:**

- as21r1's best path to 1.0.0.0/20 is `108 108 108 108 1` (through us).
- as1r1's best path to 2.21.0.0/20 is `108 108 108 108 21`.
- **as21h1 → AS1 / AS22:** AS21 → r2 → r1 → AS1 (→ AS2 → AS22).
- Our own traffic is unaffected.

- [ ] Result — notes:

### 7.5 r1 offline

**Fail:** take as108r1 offline, as described in the reference section.

**Expected:**

- **Internal (OSPF):** r3 and r4 lose neighbor 1.108.3.1. r3's default is via 1.108.0.2 and
  r4's default is via 1.108.0.4.
- **External (BGP):** as1r1's best path to us is `2 21 108 108`.
- **c1 → AS12:** r4 → r3 → r2 → AS21 → AS2 → AS1 → AS12, and back the same way.
- All services work.

- [ ] Result — notes:

### 7.6 r2 offline

**Fail:** take as108r2 offline.

**Expected:**

- **Internal (OSPF):** the OSPF route to 2.21.0.0/20 disappears on r3 and r4, which now use
  the default via r1.
- **External (BGP):** as21r1's best path to us is `2 1 108`.
- **c1 → AS21:** r4 → r1 → AS1 → AS2 → AS21, and back the same way.
- All services work.

- [ ] Result — notes:

---

## Step 8 – Other ASes can use our DNS and WWW

**Goal:** hosts in other ASes can resolve our names and load our web page, both in normal
operation and during every failure.

### 8.1 Normal operation

**Where:** as1h2, as2h2, as3h2, as12h1, as21h1, as22h1.

**How:** run **[SERVICES-OUT]** on each.

**Expected:** every lookup returns the right answer, and the page loads. This works through
the other AS's own DNS server, via the `.lab` → `ns.isp108.lab` delegation.

- [ ] Result — notes:

### 8.2 During failures

**How:** for each scenario, fail it as described earlier. Then run **[SERVICES-OUT]** on
**as12h1** (reaches us via AS1) and **as21h1** (reaches us via the direct link). Restore
before the next scenario.

| Scenario | Fail commands | as12h1 OK? | as21h1 OK? |
|---|---|---|---|
| Primary link down | see 7.1 | ☐ | ☐ |
| Private link down | see 7.2 | ☐ | ☐ |
| iBGP link (r1–r2) down | see 7.3 | ☐ | ☐ |
| r1–r3 down (servers' primary path to r1) | see Step 4 | ☐ | ☐ |
| r2–r3 down (servers' primary path to r2) | see Step 4 | ☐ | ☐ |
| r1 offline | see 7.5 | ☐ | ☐ |
| r2 offline | see 7.6 | ☐ | ☐ |

**Expected:** all ☐ become ✔. When one side of an external link is left up, wait for BGP to
converge (up to 180 s) before judging.

---

## Known issues and expected quirks

1. **ECMP on the /31 transit subnets (1.3).** The r1–r3–r4 triangle has equal costs of 10, so
   each router in it has two equal paths to the link it is not on. Still open.
2. **`isc-dhcp-server` shows `failed (dead)` (2.3)** even though a `dhcpd` process serves
   leases. Probably the IPv6 part of the service script. To be checked.
3. **Extra startup line on the servers (0.4).** `ip link set up dev eth0` is not part of the
   teacher's three-line pattern. Consider removing it.
4. **Test 7.1:** outbound traffic still goes via r1 first, because of
   `default-information originate always`.
5. **Test 7.2:** traffic may detour through r2, because r2 redistributes its iBGP-learned
   2.21.0.0/20 into OSPF.
6. **Timers:** OSPF needs up to 40 s and eBGP up to 180 s when only one end of a link is
   downed.
