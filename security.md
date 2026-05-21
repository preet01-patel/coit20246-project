# Task 4.2 — Security Hardening and Traffic Analysis

## 4.2.1 Hardening the OpenWRT System

System hardening reduces the attack surface of the router by removing default credentials, strengthening authentication, and disabling unnecessary services. Each step below includes before and after evidence.

---

### Hardening Step 1 — Change Default Credentials

**Security Risk Addressed:** OpenWRT ships with no root password by default. Any user who gains access to the router console or SSH can take full administrative control without any authentication. Setting a strong root password is the most fundamental security measure.

#### BEFORE — Password Status

The current password hash was inspected before making any changes:

```bash
grep root /etc/shadow
```

**Figure 24 — BEFORE: Root password hash status on OpenWRT**

![Figure 24](images/Figure%2024%20-%20BEFORE%20-%20Root%20password%20hash%20status%20on%20OpenWRT.png)

*The /etc/shadow file showing the existing root password hash before the change.*

#### Password Changed

```bash
passwd root
```

**Figure 25 — Root password successfully changed on OpenWRT**

![Figure 25](images/Figure%2025%20-%20Root%20password%20successfully%20changed%20on%20OpenWRT.png)

*passwd command confirming "Password for root changed by root" after entering the new password.*

#### AFTER — Hash Updated

```bash
grep root /etc/shadow
```

**Figure 26 — AFTER: Root password hash status confirmed updated**

![Figure 26](images/Figure%2026%20-%20AFTER%20-%20Root%20password%20hash%20status%20confirmed%20updated.png)

*The hash value in /etc/shadow is now different from the BEFORE screenshot, confirming the password was changed.*

---

### Hardening Step 2 — Examine Password Storage

**Security Risk Addressed:** Storing passwords in plaintext means anyone reading the file instantly knows all passwords. Hashing is a one-way transformation that makes passwords unreadable — even administrators cannot retrieve the original password from the hash.

The full shadow file was examined and the hash algorithm identified:

```bash
cat /etc/shadow
awk -F: '/root/{print $2}' /etc/shadow
```

**Figure 27 — /etc/shadow file showing root password stored as hash**

![Figure 27](images/Figure%2027%20-%20etc%20shadow%20file%20showing%20root%20password%20stored%20as%20hash.png)

*Full /etc/shadow file showing all system accounts. Only root has a real password hash; service accounts show asterisks.*

**Figure 28 — Root password hash showing hashing algorithm identifier**

![Figure 28](images/Figure%2028%20-%20Root%20password%20hash%20showing%20hashing%20algorithm%20identifier.png)

*The extracted hash beginning with `$1$` identifies the MD5 hashing algorithm used by this OpenWRT version.*

**Hash Algorithm Identification:**

| Prefix | Algorithm |
|---|---|
| `$1$` | MD5 (used by this OpenWRT system) |
| `$5$` | SHA-256 |
| `$6$` | SHA-512 (recommended for modern systems) |

Passwords are stored as hashes rather than plaintext because hashing is a one-way process — the hash cannot be reversed to recover the original password. If the shadow file is compromised, an attacker only obtains the hash, not the actual password. They would need to perform computationally expensive brute-force or dictionary attacks to crack it. MD5 is considered weak by modern standards; a production system should use SHA-512 (`$6$`).

---

### Hardening Step 3 — SSH Key-Based Authentication

**Security Risk Addressed:** Password-only SSH authentication is vulnerable to brute-force attacks where an attacker systematically tries thousands of password combinations. Key-based authentication uses a cryptographic key pair — the private key stays on the user's machine and the public key is stored on the router. Without the private key file, authentication is impossible regardless of what passwords are tried.

#### Key Pair Generation on Windows

```bash
ssh-keygen -t rsa -b 4096 -f C:\Users\LENOVO\.ssh\openwrt_key
```

**Figure 29 — SSH RSA key pair generated on Windows host**

![Figure 29](images/Figure%2029%20-%20SSH%20RSA%20key%20pair%20generated%20on%20Windows%20host.png)

*4096-bit RSA key pair generated on the Windows host, creating openwrt_key (private) and openwrt_key.pub (public).*

#### Public Key Copied to OpenWRT

```bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh
echo "PUBLIC_KEY_CONTENT" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

**Figure 30 — SSH public key added to OpenWRT authorized_keys file**

![Figure 30](images/Figure%2030%20-%20SSH%20public%20key%20added%20to%20OpenWRT%20authorized_keys%20file.png)

*authorized_keys file on OpenWRT containing the public key from the Windows host.*

#### Successful Passwordless Login

```bash
ssh -i C:\Users\LENOVO\.ssh\openwrt_key root@192.168.56.101
```

**Figure 31 — Successful SSH login using key-based authentication without password**

![Figure 31](images/Figure%2031%20-%20Successful%20SSH%20login%20using%20key-based%20authentication%20without%20password.png)

*SSH connects to OpenWRT without prompting for a password, using only the private key for authentication.*

Key-based authentication is more secure than password authentication because: the private key is a 4096-bit cryptographic file that cannot be guessed or brute-forced; authentication requires physical possession of the private key file; and even if an attacker intercepts network traffic, they cannot extract the key from the SSH handshake.

---

### Hardening Step 4 — Disable Unnecessary Services

**Security Risk Addressed:** Every running service is a potential attack vector. Services that are not required for the router's core functions increase the attack surface unnecessarily. Disabling them reduces the number of potential vulnerabilities an attacker could exploit.

#### BEFORE — Services Running

```bash
ls /etc/init.d/
```

**Figure 32 — BEFORE: List of all services available on OpenWRT**

![Figure 32](images/Figure%2032%20-%20BEFORE%20-%20List%20of%20all%20services%20available%20on%20OpenWRT.png)

*All available OpenWRT services before any were disabled.*

The `odhcpd` service (OpenWRT DHCPv6/RA daemon) was identified as unnecessary. This service manages IPv6 address assignment and router advertisements. Since the business network does not require IPv6, this service presents an unnecessary attack surface.

```bash
/etc/init.d/odhcpd status
```

**Figure 33 — BEFORE: odhcpd service running status before disabling**

![Figure 33](images/Figure%2033%20-%20BEFORE%20-%20odhcpd%20service%20running%20status%20before%20disabling.png)

*odhcpd service shown as running before being disabled.*

#### Service Disabled

```bash
/etc/init.d/odhcpd stop
/etc/init.d/odhcpd disable
/etc/init.d/odhcpd status
```

**Figure 34 — AFTER: odhcpd service disabled — reduces IPv6 attack surface**

![Figure 34](images/Figure%2034%20-%20AFTER%20-%20odhcpd%20service%20disabled%20-%20reduces%20IPv6%20attack%20surface.png)

*odhcpd service stopped and disabled, confirming it will not restart on reboot.*

Disabling odhcpd means the router no longer broadcasts IPv6 router advertisements or responds to DHCPv6 requests. This eliminates potential IPv6-based attacks such as rogue router advertisement injection and reduces the daemon count running on the system.

---

## 4.2.2 Network Traffic Capture and Analysis

Network traffic was captured using `tcpdump` on OpenWRT and analysed using Wireshark on the Windows host. Two captures were performed to compare unencrypted (HTTP) and encrypted (SSH) protocols.

---

### Capture 1 — HTTP Traffic

#### Capture Command

Traffic was captured on eth1 (the interface connected to the Windows host) while the business website was accessed from the Windows browser:

```bash
tcpdump -i eth1 -w /tmp/http_capture.pcap port 80 &
```

**Figure 35 — tcpdump capturing HTTP traffic on eth1 interface**

![Figure 35](images/Figure%2035%20-%20tcpdump%20capturing%20HTTP%20traffic%20on%20eth1%20interface.png)

*tcpdump running on eth1, capturing all traffic on port 80 to the file http_capture.pcap.*

The website at `http://192.168.56.101` was loaded from the Windows browser to generate HTTP traffic. The capture was then stopped and the file verified:

**Figure 36 — HTTP packet capture file saved to /tmp/http_capture.pcap**

![Figure 36](images/Figure%2036%20-%20HTTP%20packet%20capture%20file%20saved%20to%20tmp%20http_capture.pcap.png)

*http_capture.pcap file confirmed saved on OpenWRT, then copied to Windows using SCP for Wireshark analysis.*

#### Wireshark Analysis

**Figure 37 — Wireshark analysis showing HTTP GET request with source/destination IPs**

![Figure 37](images/Figure%2037%20-%20Wireshark%20analysis%20showing%20HTTP%20GET%20request%20with%20source%20destination%20IP.png)

*Wireshark showing HTTP GET request packet with clearly visible source IP (192.168.56.1 — Windows host), destination IP (192.168.56.101 — OpenWRT), and the requested URL path.*

**Figure 38 — Wireshark showing HTTP response payload containing student names and IDs in plaintext**

![Figure 38](images/Figure%2038%20-%20Wireshark%20showing%20HTTP%20response%20payload%20containing%20student%20names%20and%20IDs%20in%20plaintext.png)

*Wireshark showing the HTTP response body. The full HTML content of the web page — including student names, IDs, and all business information — is visible in plaintext in the packet payload.*

#### Security Implications

An attacker intercepting HTTP traffic on the network could observe: the full URL and page content being accessed, all data sent between the browser and web server including any form submissions, source and destination IP addresses revealing the internal network structure, and HTTP headers revealing server software details (useful for targeted attacks). For Patel & Associates, if any client financial information were ever transmitted over HTTP, it would be completely exposed to anyone able to capture network traffic. This demonstrates why HTTPS encryption is essential for any business web presence.

---

### Capture 2 — SSH Traffic

#### Capture Command

```bash
tcpdump -i eth1 -w /tmp/ssh_capture.pcap port 22 &
```

**Figure 39 — tcpdump capturing SSH traffic on port 22**

![Figure 39](images/Figure%2039%20-%20tcpdump%20capturing%20SSH%20traffic%20on%20port%2022.png)

*tcpdump capturing SSH session traffic on port 22.*

An SSH session was established from Windows to OpenWRT, several commands were run, then the session was closed. The capture file was copied to Windows for analysis.

#### Wireshark Analysis

**Figure 40 — Wireshark showing SSH traffic with encrypted payload — content not readable**

![Figure 40](images/Figure%2040%20-%20Wireshark%20showing%20SSH%20traffic%20with%20encrypted%20payload%20-%20content%20not%20readable.png)

*Wireshark showing SSH packets. While source/destination IPs and port numbers are visible, the payload of every packet shows only "Encrypted Packet" — the actual commands and responses are completely unreadable.*

#### Comparison — HTTP vs SSH

**Figure 41 — Contrast: SSH encrypted payload vs HTTP plaintext**

![Figure 41](images/Figure%2041%20-%20Contrast%20-%20SSH%20encrypted%20payload%20vs%20HTTP%20plaintext%20-%20demonstrating%20importance%20of%20encryption.png)

*Side-by-side comparison demonstrating that HTTP exposes full content in plaintext while SSH shows only encrypted data.*

| Feature | HTTP (Unencrypted) | SSH (Encrypted) |
|---|---|---|
| Payload visible? | Yes — full HTML content readable | No — shows "Encrypted Packet" only |
| Credentials exposed? | Yes — if login forms used | No — encrypted before transmission |
| Commands visible? | N/A | No — all commands encrypted |
| Source/Dest IPs | Visible | Visible (metadata always visible) |
| Risk to business | High — client data could be intercepted | Low — encrypted end-to-end |

Encrypted protocols like SSH protect data in transit by making the payload unreadable to anyone who intercepts the network traffic. Even if an attacker captures every SSH packet, they cannot determine what commands were run or what data was transferred. This is why businesses must use encrypted protocols (HTTPS, SSH, SFTP) for all sensitive communications.
