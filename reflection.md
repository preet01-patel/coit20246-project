# Task 4.4 — Project Reflection

## Group Members

| Student Name | Student ID |
|---|---|
| Preetkumar Navinbhai Patel | 12277687 |
| Galbokke Hewage Adheesha Jithendree Sarathkumara | 12328412 |

---

## 1. GitHub Commit History

The following screenshot shows the commit history from both team members across the project period.

**Figure 43 — GitHub commit history showing commits from both team members**

![Figure 43](images/Figure%2043%20-%20GitHub%20commits.png)

*GitHub repository commit history showing contributions from both Preetkumar and Adheesha across the project timeline.*

---

## 2. Task Split

The following table shows the actual division of tasks completed by each student:

| Task | Description | Completed By |
|---|---|---|
| VM Installation & Network Setup | VirtualBox, OpenWRT install, adapter configuration | Both |
| SSH Access Fix | Diagnosing and resolving host-only network routing issues | Preetkumar |
| Web Server Setup | uhttpd install, HTML page creation, browser verification | Both |
| Firewall Rule 1 — HTTP Block/Allow | nftables rules, before/after screenshots | Preetkumar |
| Firewall Rule 2 — SSH Port Change | dropbear UCI config, port 22 vs 2222 testing | Adheesha |
| Firewall Rule 3 — ICMP Block/Allow | nftables ICMP rules, ping testing | Preetkumar |
| Firewall Rule 4 — Port 81 Restriction | Management interface access control | Adheesha |
| Hardening — Password Change | passwd, shadow file examination | Adheesha |
| Hardening — SSH Key Authentication | Key generation, authorized_keys, passwordless login | Both |
| Hardening — Disable Services | odhcpd identification and disabling | Preetkumar |
| HTTP Traffic Capture | tcpdump, Wireshark HTTP analysis | Both |
| SSH Traffic Capture | tcpdump, Wireshark SSH analysis | Both |
| Network Diagrams | draw.io lab and production diagrams | Preetkumar |
| Risk Assessment — TVAMatrix | Threats, vulnerabilities, asset analysis | Both |
| Security Controls | Selecting and explaining three controls | Adheesha |
| Report Writing | All Markdown files | Both |
| Video Demonstration | Recording and editing | Both |

---

## 3. Git Commits vs Contributions

Preetkumar made a higher number of commits than Adheesha overall, which reflects his primary role in diagnosing the network configuration issues early in the project. The VirtualBox host-only network problem required many iterative attempts — each attempted fix and verification was committed separately, resulting in a larger commit count for that period. Adheesha's commits were fewer but covered larger units of work, particularly the hardening steps and security controls section which were researched and written in longer sessions. The number of commits is therefore not directly proportional to contribution — Adheesha's fewer commits still represent equal effort in terms of time and quality of work produced.

---

## 4. Weekly Commit Activity

Both students made commits during Weeks 5 through 12 of the term. The most active weeks were Weeks 5–6 (network setup and firewall rules) and Week 10 (report writing). There were two weeks — Weeks 3 and 4 — where only Preetkumar committed, as the initial VM setup was being done on his machine before SSH was available for collaborative access. From Week 5 onwards both students committed every week they worked on the project. We consider this sufficient given the nature of the tasks, though in hindsight earlier collaboration would have been more balanced.

---

## 5. What Worked Well and Lessons Learned

### What Worked Well

**Using SSH for all OpenWRT management** worked extremely well once it was set up. Being able to copy-paste commands directly into the SSH terminal from Windows eliminated the error-prone manual typing that caused problems during the initial VM console setup phase. We would strongly recommend any future group establish SSH access as the very first priority before attempting any other configuration.

**WhatsApp for quick coordination** was effective for sharing screenshots and quick status updates between working sessions. Messages were responded to within a few hours, keeping the project moving without needing to schedule a Zoom call for every small question.

### Issues Encountered

**VirtualBox network adapter configuration** was the most significant challenge. The host-only network routing conflict between eth1 and br-lan caused several hours of troubleshooting. The root cause was that OpenWRT's br-lan bridge claimed the entire 192.168.56.0/24 subnet route, preventing eth1 replies from reaching Windows. This was resolved using `ip route del` and made permanent via `/etc/rc.local`. Future groups should verify SSH connectivity before proceeding to any other task.

**Copy-paste not working in the VirtualBox VM console** was an unexpected difficulty. OpenWRT does not support VirtualBox Guest Additions, so the shared clipboard option does not work. All commands had to be typed manually in the console until SSH was available. Future groups should be aware of this limitation from the start.

### Techniques for Future Group Projects

**Technique 1 — Establish remote access first:** Before doing any practical configuration work, verify that SSH or some remote access method is working. This single step would have saved several hours of troubleshooting in this project, because all subsequent configuration could then be done via copy-paste rather than error-prone manual typing.

**Technique 2 — Commit after every discrete task:** We agreed to commit after every meaningful change, such as adding a screenshot or completing a section of the report. This created a clear record of progress and made it easy to revert if something went wrong. Future groups should define what constitutes a "commit-worthy" action at the start of the project.

**Technique 3 — Document assumptions explicitly before starting practical work:** Taking time in Week 3 to agree on the business scenario, IP addressing scheme, and network design before touching the VM meant we never had to redo practical work due to a planning oversight. This technique solves the common problem of rework caused by unclear requirements.
