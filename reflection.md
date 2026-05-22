# Task 4.4 — Project Reflection

| Name | Student ID |
|---|---|
| Preetkumar Navinbhai Patel | 12277687 |
| Galbokke Hewage Adheesha Jithendree Sarathkumara | 12328412 |

---

## 1. Commit History

**Figure 43 — GitHub commit history showing commits from both team members**

![Figure 43](images/Figure%2043%20-%20GitHub%20commits.png)

*GitHub commit history for the coit20246-project repository.*

---

## 2. How We Split the Work

| Task | Who Did It |
|---|---|
| VirtualBox and OpenWRT installation | Preetkumar |
| Network adapter configuration and SSH troubleshooting | Preetkumar |
| Web server setup and HTML page | Both |
| Firewall rules 1 and 3 (HTTP and ICMP) | Preetkumar |
| Firewall rules 2 and 4 (SSH port and port 81) | Adheesha |
| Password change and shadow file analysis | Adheesha |
| SSH key generation and setup | Both |
| Disabling odhcpd service | Preetkumar |
| HTTP and SSH traffic captures | Both |
| Wireshark analysis | Both |
| draw.io network diagrams | Preetkumar |
| TVAMatrix risk assessment | Both |
| Security controls write-up | Adheesha |
| Report writing (all .md files) | Both |
| Video recording | Both |

---

## 3. Commits vs Actual Contributions

Preetkumar has more commits than Adheesha, but that's mostly because of the network setup phase early in the project. Getting the host-only adapter to work properly took a lot of trial and error — each attempt to fix the routing was a separate commit. That period generated a high commit count for Preetkumar that doesn't really reflect more contribution, just more iterations on a tricky problem.

Adheesha's commits tend to cover bigger chunks of work done in fewer sessions. The security controls section and the risk assessment write-up were both done in longer focused sessions with fewer but larger commits. So the raw commit numbers don't tell the whole story about who did what.

From about Week 6 onward both of us were committing regularly each week. The early weeks were mostly Preetkumar because the setup work was happening on his machine.

---

## 4. Weekly Commit Activity

Both of us were active in the repository from Week 5 through to Week 12. The weeks with the most activity were Week 6 (firewall rules), Week 8 (traffic captures), and Week 10 (report writing). There were a couple of weeks early on where only Preetkumar committed because the VM setup was on his machine and we hadn't sorted out a workflow for both of us to contribute yet. From Week 5 onwards it was more consistent. Looking back we probably should have started committing together earlier, but once we had SSH working it became much easier to collaborate.

---

## 5. What Worked, What Didn't, and What We'd Do Differently

### Things that went well

WhatsApp worked really well for day-to-day coordination. Being able to send a quick message like "SSH is working now, your turn to test it" or share a screenshot of something that didn't work was much faster than scheduling a call every time. We didn't have many situations where one of us was blocked waiting for the other.

Splitting the firewall rules between us so we each did two of them was a good call. It meant both of us had to actually understand how nftables works rather than one person doing all of it while the other watched.

### Problems we ran into

The network setup at the beginning took much longer than expected. The issue with the br-lan bridge claiming the 192.168.56.0/24 route and blocking replies wasn't obvious at first. We spent a fair amount of time on it before figuring out that the problem was conflicting routes rather than a firewall issue. In hindsight, running `ip route show` earlier would have pointed us to it faster.

Another thing that caught us off guard was that VirtualBox clipboard sharing doesn't work with OpenWRT. The Guest Additions that enable clipboard sharing aren't compatible with OpenWRT's minimal Linux environment. That made the early console work much slower because every command had to be typed manually. Once we got SSH working that problem went away entirely.

### What we'd recommend for future group projects

**Sort out remote access first.** Before doing any configuration work, make sure you have a reliable way to send commands without having to type them manually in the VM console. In our case that meant getting SSH working as the first priority. It's tempting to jump straight into the actual tasks but fixing the workflow early saves a lot of time later.

**Define what a commit is before you start.** We agreed in general terms to commit regularly, but we didn't define what that meant precisely. Some sessions one person would make five small commits and the other would make one big one covering the same amount of work. If we'd agreed upfront on something like "commit after each completed sub-task" it would have been more consistent and the commit history would be a more accurate reflection of progress.

**When something isn't working, check the basics before going deep.** We spent time troubleshooting SSH connectivity when the actual problem was a routing conflict that would have taken one command to spot. Checking `ip route show` and `ip addr` systematically before trying more complex fixes would have saved time. It's easy to assume the problem is in the thing you were just configuring when it's actually somewhere completely different.
