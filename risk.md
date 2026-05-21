# Task 4.3 — Risk Assessment and Security Controls

## 4.3.1 Cyber Security Risk Assessment

A mini cyber security risk assessment was conducted for Patel & Associates using the TVAMatrix framework provided in the unit. The assessment covers the network built in Tasks 4.1 and 4.2, and references specific configurations such as firewall rules, hardening steps, and traffic analysis findings where relevant.

### Scope

The risk assessment covers the small business office network including: the OpenWRT router/firewall, staff Windows workstations, the business web server, client financial records, and staff data. The assessment considers the business context — a professional accounting firm handling sensitive client financial data in Rockhampton, QLD.

### TVAMatrix Risk Assessment

The completed risk assessment is available in the spreadsheet:
[risk-assessment.xlsx](risk-assessment/risk-assessment.xlsx)

The TVAMatrix considers vulnerabilities across the following threat categories covered in the assessment:

1. Unauthorised Access
2. Malware / Ransomware
3. Phishing / Social Engineering
4. Denial of Service (DoS)
5. Data Interception (Man-in-the-Middle)
6. Physical Security Breach
7. Insider Threat
8. Weak Authentication
9. Unpatched Software / Vulnerabilities
10. Misconfigured Firewall
11. Data Loss / Accidental Deletion
12. Third-Party / Supply Chain Risk

**TVAMatrix Screenshot:**

![TVAMatrix Risk Assessment](images/Figure%2044%20-%20TVAMatrix%20Risk%20Assessment.png)

*Completed TVAMatrix showing assets, threats, vulnerabilities, likelihood, impact, and risk ratings for Patel & Associates.*

---

## 4.3.2 Recommended Security Controls

### Highest Risk Asset

Based on the TVAMatrix, **client financial records** (tax returns, financial statements, and payroll data stored on the file server) received the highest risk rating. This asset is highly sensitive under Australian Privacy Act obligations, is a prime target for ransomware and data theft, and its compromise would cause severe reputational and legal consequences for the business.

The primary threat identified for this asset is **unauthorised access combined with data interception**, which directly relates to the HTTP traffic capture finding in Section 4.2.2 — data transmitted over unencrypted HTTP is fully visible to any network attacker.

---

### Control 1 — Access Control (AC-3: Access Enforcement) — NIST SP800-53

**How it reduces risk:** Access control ensures that only authorised staff can access client financial records. By implementing role-based access, accountants can only access their own clients' files, administrative staff cannot access financial data, and the IT officer has system-level access without necessarily accessing client content. This directly reduces the risk of insider threats and unauthorised access.

**Implementation in this scenario:**
- On the file server (87.10.3.3 in production), configure Windows NTFS folder permissions so each accountant has read/write access only to their assigned client folders
- The IT support officer account is granted system administration rights but not read access to client data folders
- Administrative staff accounts are restricted to shared reception folders only
- The OpenWRT firewall rules configured in Section 4.1.3 already restrict which internal IPs can reach the file server on port 445 (SMB)

**Disadvantages:** Role-based access requires ongoing maintenance — when staff change roles or new clients are assigned, permissions must be updated manually. If the IT officer is unavailable, there is no one to grant emergency access, which could interrupt business operations during urgent situations.

---

### Control 2 — System and Communications Protection (SC-8: Transmission Confidentiality) — NIST SP800-53

**How it reduces risk:** As demonstrated in the HTTP traffic capture (Section 4.2.2), unencrypted data is fully visible to any attacker on the network. Enforcing encrypted communications (HTTPS for the web server, SFTP/SCP for file transfers, encrypted email) ensures that even if traffic is intercepted, it cannot be read. This directly mitigates the data interception risk for client financial records.

**Implementation in this scenario:**
- Replace the current HTTP web server with HTTPS by generating an SSL/TLS certificate for the OpenWRT uhttpd server (Let's Encrypt for production, self-signed for internal use)
- The existing firewall rule demonstrated in Section 4.1.3 that blocks port 80 can be permanently applied, forcing all web traffic to port 443 (HTTPS)
- Configure Windows workstations to use encrypted file sharing protocols (SMB over TLS or SFTP) when accessing the file server
- Implement encrypted email (S/MIME or PGP) for sending client financial documents

**Disadvantages:** SSL/TLS certificates require renewal (typically every 90 days for Let's Encrypt), adding an ongoing maintenance task. Older client browsers or systems may not support modern TLS versions, potentially causing access issues. Staff training is required to understand why HTTPS warnings should not be bypassed.

---

### Control 3 — Audit and Accountability (AU-2: Event Logging) — NIST SP800-53

**How it reduces risk:** Comprehensive logging of access to client financial records means that any unauthorised access or unusual activity can be detected and investigated. Without logs, a data breach may go undetected for weeks or months. Logs also provide forensic evidence if a breach occurs and are required for Australian Privacy Act compliance when reporting notifiable data breaches.

**Implementation in this scenario:**
- Enable OpenWRT firewall logging for all inbound and outbound connections — this builds on the nftables rules configured in Section 4.1.3 by adding a log action before the drop/accept action
- Enable Windows file auditing on the file server so that every access to client folders is recorded with timestamp, username, and action performed
- Configure log forwarding to a central syslog server (can be the IT support workstation at 87.10.2.2) so logs are stored off the router and cannot be tampered with
- Review logs weekly — the IT support officer should check for failed login attempts, after-hours access, and large file downloads

**Disadvantages:** Logging generates large volumes of data that must be stored and managed. Without automated alerting tools, manual log review is time-consuming and may be deprioritised during busy periods. Log storage on a small business server may become a cost and capacity issue over time.
