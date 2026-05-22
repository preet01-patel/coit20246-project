# Task 4.3 — Risk Assessment and Security Controls

## 4.3.1 Risk Assessment

We used the TVAMatrix framework provided in the unit to do a mini risk assessment for Patel & Associates. The full spreadsheet is in the risk-assessment folder. Below is a summary of how we approached it.

### Scope

The assessment covers the network we built — the OpenWRT router, staff workstations, the web server, and the data the business handles. The most significant assets from a security perspective are the client financial records and personal information, since an accounting firm is legally required under the Australian Privacy Act to protect that data.

We covered 10 of the 12 threat categories and included 14 assets across all four asset types (Data, Hardware, Software, People). The four data assets are: client financial records, client personal information, business financial data, and staff credentials.

### TVAMatrix

![Figure 44](images/Figure%2044%20-%20TVAMatrix%20Risk%20Assessment.png)

*Completed TVAMatrix showing all assets, threats, vulnerabilities, likelihood, impact and risk ratings.*

[Full spreadsheet: risk-assessment.xlsx](risk-assessment/risk-assessment.xlsx)

### Key Findings

The highest-rated risks were around client financial records being exposed through software attacks (particularly ransomware) and through data interception on the network. The HTTP traffic capture in Section 4.2.2 directly demonstrated that unencrypted traffic exposes content to anyone on the network — that's not a theoretical risk, we showed it happening.

Staff behaviour also came up as a significant risk area. Phishing and accidental data disclosure (like emailing a client file to the wrong address) ranked moderately because they're fairly likely in a small office where staff may not have had formal security training.

---

## 4.3.2 Security Controls

The asset with the highest risk rating is **client financial records** (tax returns, financial statements, client data stored on the file server). We've selected three controls that would most directly reduce the risk to this asset.

---

### Control 1 — Access Control (NIST SP800-53: AC-3)

The idea here is that not everyone in the office should be able to access all client files. An admin staff member has no legitimate reason to open a client's tax return — that's an accountant's job. Role-based access control means each person can only access what they actually need for their role.

In practice for Patel & Associates this would mean:
- Each accountant gets read/write access only to their assigned client folders on the file server (87.10.3.3 in the production network)
- Admin staff get access to shared scheduling and correspondence folders only
- The IT officer gets system-level access to manage the server but not necessarily read access to client documents
- Access is tied to individual Windows user accounts, not shared passwords

This directly links to the firewall work we did — the firewall rules in Section 4.1.3 already restrict which IPs can reach which services. Access control at the file system level adds another layer on top of that. Even if someone gets onto the network, they still can't open files they don't have permissions for.

The downside is maintenance overhead. When an accountant takes on a new client, someone has to update the permissions. When a staff member leaves, their account needs to be deactivated promptly. In a small business with no dedicated IT staff this can slip — especially during busy periods like tax season.

---

### Control 2 — Transmission Encryption (NIST SP800-53: SC-8)

The HTTP capture showed exactly why this matters. Everything sent over HTTP is readable to anyone who can see the traffic. For a business that handles financial data, running HTTP is genuinely risky — not just for the website but for any internal system that uses unencrypted connections.

For this business the control would involve:
- Replacing the current HTTP web server with HTTPS by adding a TLS certificate to uhttpd. The firewall rule we demonstrated in Section 4.1.3 that blocks port 80 could be made permanent, forcing all connections to port 443
- Using encrypted protocols for internal file transfers (SFTP rather than FTP if files are transferred between machines)
- Making sure accounting software communicates with any cloud services over HTTPS

This is one of the more impactful controls because it addresses the vulnerability we literally demonstrated in this project. The capture showed our names and page content in plaintext — if that had been a login form or a file transfer it would have been much worse.

The trade-off is certificate management. TLS certificates expire and need to be renewed. Let's Encrypt automates this for free but someone needs to set it up and make sure the renewal process works. For a small business without dedicated IT support, this could become a problem if the certificate expires and nobody notices until clients report the site is showing security warnings.

---

### Control 3 — Audit Logging (NIST SP800-53: AU-2)

Without logs, you can't tell if someone accessed files they shouldn't have, or whether an attack happened at all. For an accounting firm, logs also matter for compliance — if there's ever a data breach, the business needs to report it under the Notifiable Data Breaches scheme and demonstrate what happened and when.

Implementation would look like:
- Enable logging on the OpenWRT firewall so every connection attempt is recorded. This builds on the nftables rules from Section 4.1.3 — you can add a log action before the drop/accept to record what was blocked and allowed
- Enable Windows file auditing on the file server so access to client folders is logged with username, timestamp and action
- Store logs somewhere other than the router itself — ideally send them to the IT officer's workstation (87.10.2.2) using syslog so they can't be tampered with if the router is compromised
- Review logs regularly, at minimum weekly, looking for failed logins, after-hours access, or large file downloads

The practical challenge is that logs generate a lot of data and reviewing them takes time. A small business might set up logging and then never look at it because there's always something more urgent to deal with. Without some kind of alerting on suspicious events, logs only become useful after something has already gone wrong. That said, having them at all is significantly better than not — forensic investigation after a breach is much harder without a log trail.
