# Task 4.1 — Network Setup

## 4.1.1 Assumptions

Before starting any of the practical work, we needed to define what the business actually looks like since the scenario was fairly open-ended. These are the assumptions we made, and everything in the project is based on them.

**1. Location — Rockhampton, Queensland**
We chose Rockhampton because it's a regional city in Central Queensland which fits the "small business serving local clients" description well. It's not a major metro area, so the business would be dealing with local individuals and small businesses rather than large corporations.

**2. Business Type — Accounting and Financial Services**
The business, which we named Patel & Associates, provides accounting services including tax returns, payroll management, financial planning, and general business accounting. We chose this because accounting firms handle genuinely sensitive data (tax file numbers, financial statements, payroll records) which makes the risk assessment more realistic and meaningful.

**3. Staff — 8 people total**
We assumed 1 IT support officer, 2 admin staff who handle reception and scheduling, and 5 accountants who work directly with client files. All staff use Windows workstations on the office network. This felt like a realistic size for a small regional accounting firm.

**4. Website Content**
The website hosted on the router is a basic static HTML page with the business name, a list of services, office location, and contact details. There's no client portal, no login system, nothing dynamic — it's purely a public information page so clients can find out what the business does.

---

## 4.1.2 Lab Network Setup

### Overview

The lab runs OpenWRT 23.05.3 as a virtual machine inside Oracle VirtualBox on a Windows 10 host. Two network adapters are used — one NAT adapter so OpenWRT can reach the internet through the host machine, and one host-only adapter so the Windows machine can communicate with the OpenWRT VM directly. This simulates how a real router would have a WAN interface facing the internet and a LAN interface facing the internal network.

Getting this working took a bit of troubleshooting. The main issue was that OpenWRT's br-lan bridge was taking over the 192.168.56.0/24 subnet route which meant packets from Windows were arriving at eth1 but replies were going out through br-lan instead, so nothing got back to Windows. Once we sorted the routing it worked properly.

---

### Booting OpenWRT

**Figure 1 — OpenWRT VM booted successfully in VirtualBox showing root shell**

![Figure 1](images/Figure%201%20-%20OpenWRT%20VM%20booted%20successfully%20in%20VirtualBox%20showing%20root%20shell.png)

*OpenWRT 23.05.3 booted in VirtualBox showing the root shell prompt.*

---

### Network Interfaces

```bash
ip addr
```

**Figure 2 — OpenWRT network interfaces showing IP address assignments**

![Figure 2](images/Figure%202%20-%20OpenWRT%20network%20interfaces%20showing%20IP%20address%20assignments.png)

*The ip addr output showing eth0, eth1 and br-lan with their assigned addresses.*

Here's what each interface does in our setup:

| Interface | Adapter Type | IP Address | Purpose |
|---|---|---|---|
| eth0 | NAT | 10.0.2.15/24 | WAN — internet access via host machine |
| eth1 | Host-only | 192.168.56.101/24 | LAN — connects to Windows host |
| br-lan | Bridge (eth1) | 192.168.56.2/24 | LAN bridge interface |
| Windows Host-Only Adapter | Host-only | 192.168.56.1/24 | Windows side of the virtual network |

The Windows host connects to OpenWRT through the VirtualBox host-only network. VirtualBox creates a virtual network adapter on the Windows machine (at 192.168.56.1) and OpenWRT's eth1 sits on the same subnet (192.168.56.101). This lets us SSH into the router and access the web server from the Windows browser without going through the internet.

---

### Internet Connectivity

**Figure 3 — OpenWRT confirming internet connectivity via NAT adapter**

![Figure 3](images/Figure%203%20-%20OpenWRT%20confirming%20internet%20connectivity%20via%20NAT%20adapter.png)

*Ping to 8.8.8.8 confirming the NAT adapter gives OpenWRT internet access.*

---

### br-lan Reconfiguration

**Figure 4 — OpenWRT br-lan reconfigured to align with VirtualBox Host-Only network**

![Figure 4](images/Figure%204%20-%20OpenWRT%20br-lan%20reconfigured%20to%20192.168.56.2%20to%20align%20with%20VirtualBox%20Host-Only%20network.png)

*br-lan showing 192.168.56.2/24 after reconfiguring to match the host-only subnet.*

---

### Web Server Setup

We installed uhttpd which is OpenWRT's built-in lightweight web server:

```bash
opkg update
opkg install uhttpd
uci set uhttpd.main.listen_http='0.0.0.0:80'
uci commit uhttpd
service uhttpd restart
```

The HTML page was created at `/www/index.html` with the business name, our names and student IDs, and the date.

**Figure 5 — uhttpd web server running and listening on port 80**

![Figure 5](images/Figure%205%20-%20uhttpd%20web%20server%20running%20and%20listening%20on%20port%2080.png)

*uhttpd confirmed running and listening on ports 80 and 443.*

**Figure 6 — Business website accessible from Windows host browser showing student names and IDs**

![Figure 6](images/Figure%206%20-%20Business%20website%20accessible%20from%20Windows%20host%20browser%20showing%20student%20names%20and%20ID.png)

*The Patel & Associates website loaded in the Windows browser at http://192.168.56.101.*

**Figure 7 — Business website HTML file content on OpenWRT**

![Figure 7](images/Figure%207%20-%20Business%20website%20HTML%20file%20content%20on%20OpenWRT.png)

*/www/index.html contents showing the HTML source with student names and IDs.*

---

## 4.1.3 Firewall Rules

OpenWRT 23.05.3 uses nftables rather than iptables. All rules below were written using nft commands. For each rule we show what the behaviour was before, the rule configuration, and then what changed after.

---

### Rule 1 — Block and Allow HTTP (Port 80)

This demonstrates the ability to control access to the web server. In a real deployment you might block HTTP entirely and only allow HTTPS to force encrypted connections. Blocking port 80 at the firewall level means even if the web server is running, nobody outside can reach it.

**Figure 8 — Firewall rule added to block HTTP traffic on port 80**

![Figure 8](images/Figure%208%20-%20Firewall%20rule%20added%20to%20block%20HTTP%20traffic%20on%20port%2080.png)

*nft rule dropping all TCP traffic to port 80, shown with nft list ruleset.*

```bash
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; policy accept \; }
nft add rule inet filter input tcp dport 80 drop
```

**Figure 9 — AFTER BLOCK: Website inaccessible after port 80 drop rule applied**

![Figure 9](images/Figure%209%20-%20AFTER%20BLOCK%20-%20Website%20inaccessible%20after%20port%2080%20drop%20rule%20applied.png)

*Browser showing connection failed after the drop rule was applied.*

Then we changed it to allow:

```bash
nft flush table inet filter
nft add rule inet filter input tcp dport 80 accept
```

**Figure 10 — Firewall rule changed to allow HTTP traffic on port 80**

![Figure 10](images/Figure%2010%20-%20Firewall%20rule%20changed%20to%20allow%20HTTP%20traffic%20on%20port%2080.png)

*Updated ruleset showing accept rule for port 80.*

**Figure 11 — AFTER ALLOW: Website accessible again after port 80 accept rule applied**

![Figure 11](images/Figure%2011%20-%20AFTER%20ALLOW%20-%20Website%20accessible%20again%20after%20port%2080%20accept%20rule%20applied.png)

*Website loading normally again after switching to the accept rule.*

---

### Rule 2 — SSH Port Change (22 to 2222)

Running SSH on port 22 is the default and it means automated scanners constantly try to brute-force it. Changing to a non-standard port like 2222 won't stop a determined attacker but it does significantly reduce the noise from automated bots and scanning tools.

**Figure 12 — BEFORE: Successful SSH connection on default port 22**

![Figure 12](images/Figure%2012%20-%20BEFORE%20-%20Successful%20SSH%20connection%20on%20default%20port%2022.png)

*SSH connecting successfully on port 22 before the change.*

```bash
uci set dropbear.@dropbear[0].Port='2222'
uci commit dropbear
service dropbear restart
```

**Figure 13 — SSH port changed from 22 to 2222 via UCI configuration**

![Figure 13](images/Figure%2013%20-%20SSH%20port%20changed%20from%2022%20to%202222%20via%20UCI%20configuration.png)

*UCI config showing dropbear port set to 2222.*

**Figure 14 — AFTER: SSH on port 22 refused after port change**

![Figure 14](images/Figure%2014%20-%20AFTER%20-%20SSH%20on%20port%2022%20refused%20after%20port%20change.png)

*Connection refused when trying port 22 — confirming it's no longer active.*

**Figure 15 — AFTER: Successful SSH connection on new port 2222**

![Figure 15](images/Figure%2015%20-%20AFTER%20-%20Successful%20SSH%20connection%20on%20new%20port%202222.png)

*SSH working on port 2222 using ssh -p 2222 root@192.168.56.101.*

We changed it back to port 22 after testing so we didn't lose access.

---

### Rule 3 — Block and Allow ICMP (Ping)

Blocking ICMP stops ping from working, which is useful because ping is one of the first tools an attacker uses to check if a host is up. If the router doesn't respond to ping it's harder to map the network. Of course it also makes troubleshooting harder, so this is a trade-off.

**Figure 16 — BEFORE: Ping from Windows to OpenWRT succeeds before ICMP block**

![Figure 16](images/Figure%2016%20-%20BEFORE%20-%20Ping%20from%20Windows%20to%20OpenWRT%20succeeds%20before%20ICMP%20block.png)

*Ping showing replies before any ICMP rule is in place.*

```bash
nft add rule inet filter input icmp type echo-request drop
```

**Figure 17 — Firewall rule added to block ICMP echo-request (ping)**

![Figure 17](images/Figure%2017%20-%20Firewall%20rule%20added%20to%20block%20ICMP%20echo-request%20(ping).png)

*nftables ruleset with the ICMP drop rule added.*

**Figure 18 — AFTER BLOCK: Ping from Windows times out after ICMP drop rule applied**

![Figure 18](images/Figure%2018%20-%20AFTER%20BLOCK%20-%20Ping%20from%20Windows%20times%20out%20after%20ICMP%20drop%20rule%20applied.png)

*Ping showing "Request timed out" after the rule was applied.*

```bash
nft flush table inet filter
```

**Figure 19 — ICMP block rule removed, ping re-enabled**

![Figure 19](images/Figure%2019%20-%20ICMP%20block%20rule%20removed%2C%20ping%20re-enabled.png)

*Ruleset flushed, ICMP no longer blocked.*

**Figure 20 — AFTER RE-ENABLE: Ping from Windows to OpenWRT succeeds again**

![Figure 20](images/Figure%2020%20-%20AFTER%20RE-ENABLE%20-%20Ping%20from%20Windows%20to%20OpenWRT%20succeeds%20again.png)

*Ping working again after removing the block.*

---

### Rule 4 — Restrict Management Interface (Port 81)

The management web interface gives full control over the router — firewall rules, network config, everything. Leaving it open to any IP on the network is a risk, especially if a client laptop or guest device ever got on the network. Restricting it to only the IT officer's workstation IP makes sense.

**Figure 21 — BEFORE: Management web interface accessible on port 81**

![Figure 21](images/Figure%2021%20-%20BEFORE%20-%20Management%20web%20interface%20accessible%20on%20port%2081.png)

*Management interface loading in browser at http://192.168.56.101:81 before restriction.*

```bash
nft add rule inet filter input ip saddr != 192.168.56.1 tcp dport 81 drop
```

**Figure 22 — Firewall rule restricting management interface access on port 81**

![Figure 22](images/Figure%2022%20-%20Firewall%20rule%20restricting%20management%20interface%20access%20on%20port%2081.png)

*nft rule dropping port 81 traffic from any IP that isn't 192.168.56.1.*

**Figure 23 — AFTER: Management interface access restricted to authorised IP only**

![Figure 23](images/Figure%2023%20-%20AFTER%20-%20Management%20interface%20access%20restricted%20to%20authorised%20IP%20only.png)

*Access to port 81 blocked from unauthorised sources.*

---

## 4.1.4 Network Diagrams

### Lab Network

**Figure 42 — Lab Network Diagram**

![Lab Network Diagram](images/Figure%2042%20-%20Lab%20Network.png)

*Lab setup showing the OpenWRT VM, Windows host, and the two VirtualBox adapters.*

#### IP Address Table — Lab Network

| Device | Interface | Adapter | IP Address | Subnet | Role |
|---|---|---|---|---|---|
| OpenWRT VM | eth0 | NAT | 10.0.2.15 | /24 | WAN — internet |
| OpenWRT VM | eth1 | Host-only | 192.168.56.101 | /24 | LAN link |
| OpenWRT VM | br-lan | Bridge | 192.168.56.2 | /24 | LAN bridge |
| VirtualBox NAT | — | NAT | 10.0.2.2 | /24 | WAN gateway |
| Windows Host | Host-Only Adapter | Host-only | 192.168.56.1 | /24 | Host machine |

---

### Production Network Design

This is what the actual Patel & Associates office network would look like if it were deployed in the real world. The lab setup only simulates the router and one connected device — the full production design has proper network segmentation with a DMZ for the web server.

**Figure — Production Network Diagram**

![Production Network Diagram](diagrams/production-network.png)

*Full production network design for Patel & Associates with WAN, LAN, and DMZ zones.*

#### IP Address Table — Production Network

IP addresses use **87** as the first octet, from the last two digits of Student 1's ID (1227768**7**), as per Section 4.1.5 of the spec.

| Device | Interface | IP Address | Subnet | Zone | Role |
|---|---|---|---|---|---|
| OpenWRT Router | WAN (eth0) | 87.10.1.1 | /24 | WAN | Internet-facing |
| OpenWRT Router | LAN (eth1) | 87.10.2.1 | /24 | LAN | Staff network gateway |
| OpenWRT Router | DMZ (eth2) | 87.10.3.1 | /24 | DMZ | Server network gateway |
| IT Support WS | NIC | 87.10.2.2 | /24 | LAN | IT officer workstation |
| Admin WS 1 | NIC | 87.10.2.3 | /24 | LAN | Admin staff |
| Admin WS 2 | NIC | 87.10.2.4 | /24 | LAN | Admin staff |
| Accountant WS 1–5 | NIC | 87.10.2.5–9 | /24 | LAN | Accountant workstations |
| Web Server | NIC | 87.10.3.2 | /24 | DMZ | Public business website |
| File Server | NIC | 87.10.3.3 | /24 | DMZ | Client records storage |
| ISP Modem | WAN | 87.10.1.254 | /24 | WAN | ISP gateway |

The reason for putting the web server and file server in a DMZ rather than on the main LAN is separation of risk. If the web server gets compromised — which is more likely since it's publicly accessible — the attacker is in the DMZ network (87.10.3.0/24) and not directly on the same network as the staff workstations and the sensitive file server. The firewall controls what traffic can pass between zones.
