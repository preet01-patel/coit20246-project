# Task 4.2 — Security Hardening and Traffic Analysis

## 4.2.1 Hardening OpenWRT

We went through four hardening steps on the router. Each one targets a specific weakness that would exist in a default OpenWRT installation. The before and after screenshots show the actual change that was made.

---

### Step 1 — Changing the Default Password

Out of the box, OpenWRT has no root password. That means anyone who gets to the console or SSH can log straight in with full admin access — no credentials needed. It's one of those things that seems obvious but is easy to overlook, especially in a small business where the IT person might set up the router and not think about it.

We ran `grep root /etc/shadow` to see what the password hash looked like before making any changes.

**Figure 24 — BEFORE: Root password hash status on OpenWRT**

![Figure 24](images/Figure%2024%20-%20BEFORE%20-%20Root%20password%20hash%20status%20on%20OpenWRT.png)

*The /etc/shadow file showing the root entry before the password was changed.*

```bash
passwd root
```

**Figure 25 — Root password successfully changed on OpenWRT**

![Figure 25](images/Figure%2025%20-%20Root%20password%20successfully%20changed%20on%20OpenWRT.png)

*Confirmation that the password was changed successfully.*

After changing it, we ran the same command again to confirm the hash had changed.

**Figure 26 — AFTER: Root password hash status confirmed updated**

![Figure 26](images/Figure%2026%20-%20AFTER%20-%20Root%20password%20hash%20status%20confirmed%20updated.png)

*The hash in /etc/shadow is visibly different from the BEFORE screenshot.*

---

### Step 2 — Looking at How Passwords Are Stored

This step is more about understanding than changing something. We looked at the full shadow file and identified what hashing algorithm is being used.

```bash
cat /etc/shadow
awk -F: '/root/{print $2}' /etc/shadow
```

**Figure 27 — /etc/shadow file showing root password stored as hash**

![Figure 27](images/Figure%2027%20-%20etc%20shadow%20file%20showing%20root%20password%20stored%20as%20hash.png)

*The full /etc/shadow file. Service accounts like dnsmasq show asterisks meaning they have no login password.*

**Figure 28 — Root password hash showing hashing algorithm identifier**

![Figure 28](images/Figure%2028%20-%20Root%20password%20hash%20showing%20hashing%20algorithm%20identifier.png)

*Extracted hash starting with $1$ which tells us it's MD5.*

The `$1$` at the start of the hash identifies MD5 as the algorithm. MD5 is considered weak by today's standards — modern systems use SHA-512 (`$6$`) which is much harder to crack if someone gets hold of the hash. This is worth noting as something to improve in a production deployment.

Passwords are stored as hashes rather than plaintext because hashing is a one-way operation. You can't reverse a hash to get the original password. If someone gets the shadow file, they'd still need to run dictionary or brute-force attacks to crack it, which takes time and resources. With plaintext passwords, it's game over instantly.

| Hash Prefix | Algorithm | Status |
|---|---|---|
| $1$ | MD5 | Weak — used on this system |
| $5$ | SHA-256 | Acceptable |
| $6$ | SHA-512 | Recommended |

---

### Step 3 — Setting Up SSH Key Authentication

Password-based SSH means the router will accept login attempts from anyone who can reach port 22 and keep guessing passwords. Key-based authentication removes that entirely — you need the private key file to authenticate, and guessing doesn't work because you're not authenticating with a password at all.

We generated a 4096-bit RSA key pair on the Windows machine:

```bash
ssh-keygen -t rsa -b 4096 -f C:\Users\LENOVO\.ssh\openwrt_key
```

**Figure 29 — SSH RSA key pair generated on Windows host**

![Figure 29](images/Figure%2029%20-%20SSH%20RSA%20key%20pair%20generated%20on%20Windows%20host.png)

*Key generation output on Windows CMD showing the key fingerprint and randomart.*

Then we copied the public key to OpenWRT:

```bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh
echo "PUBLIC_KEY" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

**Figure 30 — SSH public key added to OpenWRT authorized_keys file**

![Figure 30](images/Figure%2030%20-%20SSH%20public%20key%20added%20to%20OpenWRT%20authorized_keys%20file.png)

*authorized_keys file on OpenWRT with the public key added.*

To test it, we connected using the private key:

```bash
ssh -i C:\Users\LENOVO\.ssh\openwrt_key root@192.168.56.101
```

**Figure 31 — Successful SSH login using key-based authentication without password**

![Figure 31](images/Figure%2031%20-%20Successful%20SSH%20login%20using%20key-based%20authentication%20without%20password.png)

*SSH connected without asking for a password — key authentication working.*

The reason this is more secure is straightforward: a 4096-bit RSA key can't be guessed. Even if someone runs a brute-force tool against SSH for years, they're not going to stumble on the correct key. With passwords, people often pick weak ones, reuse them, or get them phished — none of those problems exist with key authentication.

---

### Step 4 — Disabling an Unnecessary Service

Every service running on the router is an additional thing that could potentially be exploited. The odhcpd service handles IPv6 address assignment and router advertisements. Since the Patel & Associates network doesn't use IPv6, there's no reason to have it running.

**Figure 32 — BEFORE: List of all services available on OpenWRT**

![Figure 32](images/Figure%2032%20-%20BEFORE%20-%20List%20of%20all%20services%20available%20on%20OpenWRT.png)

*All services visible in /etc/init.d/ before any changes.*

**Figure 33 — BEFORE: odhcpd service running status before disabling**

![Figure 33](images/Figure%2033%20-%20BEFORE%20-%20odhcpd%20service%20running%20status%20before%20disabling.png)

*odhcpd shown as active and running.*

```bash
/etc/init.d/odhcpd stop
/etc/init.d/odhcpd disable
```

**Figure 34 — AFTER: odhcpd service disabled — reduces IPv6 attack surface**

![Figure 34](images/Figure%2034%20-%20AFTER%20-%20odhcpd%20service%20disabled%20-%20reduces%20IPv6%20attack%20surface.png)

*odhcpd stopped and disabled, won't start on reboot.*

With odhcpd off, the router no longer broadcasts IPv6 router advertisements or responds to DHCPv6 requests. This removes a potential vector for IPv6-based attacks like rogue RA injection. It's a small change but it's part of the broader principle of reducing attack surface — don't run things you don't need.

---

## 4.2.2 Traffic Captures and Analysis

We captured traffic using tcpdump on OpenWRT and then analysed the pcap files in Wireshark on Windows. The two captures compare HTTP (unencrypted) and SSH (encrypted) to show the difference in what's visible to someone who intercepts the traffic.

---

### Capture 1 — HTTP

We started the capture on eth1 (the interface the Windows host connects through) then loaded the website from the Windows browser a few times to generate traffic.

```bash
tcpdump -i eth1 -w /tmp/http_capture.pcap port 80 &
```

**Figure 35 — tcpdump capturing HTTP traffic on eth1 interface**

![Figure 35](images/Figure%2035%20-%20tcpdump%20capturing%20HTTP%20traffic%20on%20eth1%20interface.png)

*tcpdump running on eth1 capturing to http_capture.pcap.*

After stopping the capture we checked the file was there and copied it to Windows using SCP.

**Figure 36 — HTTP packet capture file saved to /tmp/http_capture.pcap**

![Figure 36](images/Figure%2036%20-%20HTTP%20packet%20capture%20file%20saved%20to%20tmp%20http_capture.pcap.png)

*http_capture.pcap confirmed saved on OpenWRT.*

Opening the pcap in Wireshark with an `http` filter shows the GET request clearly:

**Figure 37 — Wireshark analysis showing HTTP GET request with source/destination IPs**

![Figure 37](images/Figure%2037%20-%20Wireshark%20analysis%20showing%20HTTP%20GET%20request%20with%20source%20destination%20IP.png)

*HTTP GET request showing source IP 192.168.56.1 (Windows) and destination 192.168.56.101 (OpenWRT).*

The response packet is where it gets interesting from a security standpoint:

**Figure 38 — Wireshark showing HTTP response payload containing student names and IDs in plaintext**

![Figure 38](images/Figure%2038%20-%20Wireshark%20showing%20HTTP%20response%20payload%20containing%20student%20names%20and%20IDs%20in%20plaintext.png)

*The HTTP response body showing the full HTML — including our names and student IDs — completely readable.*

This is the core problem with HTTP. Everything is in plaintext. An attacker on the same network — or anyone who can intercept the traffic between the client and server — can read the full content of every page. For a static website with public information that might not seem like a big deal, but if the business ever sent client documents or login credentials over HTTP, they'd be completely exposed. The fix is HTTPS, which encrypts the connection so the payload can't be read even if captured.

---

### Capture 2 — SSH

Same process, but this time capturing SSH traffic while we ran an active SSH session.

```bash
tcpdump -i eth1 -w /tmp/ssh_capture.pcap port 22 &
```

**Figure 39 — tcpdump capturing SSH traffic on port 22**

![Figure 39](images/Figure%2039%20-%20tcpdump%20capturing%20SSH%20traffic%20on%20port%2022.png)

*tcpdump capturing SSH session traffic.*

We connected from Windows, ran a few commands, then closed the session and stopped the capture.

**Figure 40 — Wireshark showing SSH traffic with encrypted payload — content not readable**

![Figure 40](images/Figure%2040%20-%20Wireshark%20showing%20SSH%20traffic%20with%20encrypted%20payload%20-%20content%20not%20readable.png)

*SSH packets in Wireshark showing "Encrypted Packet" in the payload — no readable content.*

**Figure 41 — Contrast: SSH encrypted payload vs HTTP plaintext**

![Figure 41](images/Figure%2041%20-%20Contrast%20-%20SSH%20encrypted%20payload%20vs%20HTTP%20plaintext%20-%20demonstrating%20importance%20of%20encryption.png)

*Comparison showing HTTP payload is readable text while SSH payload is encrypted binary data.*

The difference is obvious when you look at both captures side by side. With HTTP you can read the exact content being transferred. With SSH you can see that packets are being exchanged and you can see the IP addresses, but the actual content of the session — every command typed and every response — is encrypted and unreadable.

| | HTTP | SSH |
|---|---|---|
| Payload readable? | Yes — full HTML visible | No — shows "Encrypted Packet" |
| Credentials exposed? | Yes, if forms are used | No |
| IPs visible? | Yes | Yes |
| Commands visible? | N/A | No |
| Risk level | High | Low |

IP addresses are always visible because they're in the packet headers — that's how routing works. But the content is what matters most for confidentiality, and SSH protects that completely.
