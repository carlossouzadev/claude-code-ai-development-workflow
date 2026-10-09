---
name: blue-team-operations
description: "Run hands-on incident response triage: live Windows/Linux collection, memory and packet analysis, and PICERL execution with analyst decision discipline."
model: opus
metadata:
  version: 1.0.0
  category: blue-team
  source: "Blue Team Handbook: Incident Response Edition (Don Murdoch)"
---

# Blue Team Operations: Hands-On Incident Response Playbook

## Goal
Give an analyst or incident handler an execution-ready playbook for working a live
incident: what to collect first, which commands to run on Windows and Linux hosts,
how to triage memory and packet captures, and the decision discipline (OODA,
Alexiou Principle, bias controls) that keeps an investigation from derailing. This is
the hands-on analyst companion to the `incident-response` knowledge skill, which
covers lifecycle, evidence law, and chain-of-custody governance; load that skill for
scope/authorization and this one for the actual keystrokes.

## When to Use
- An alert has been escalated to an incident handler and triage must start now.
- You need to collect volatile data (memory, process, network state) from a live
  Windows or Linux host before it is contained or reimaged.
- A packet capture (PCAP) needs triage with tcpdump/tshark/Wireshark to confirm
  scanning, beaconing, DNS manipulation, or data exfiltration.
- You are deciding whether an alert is a false positive, explainable activity, or a
  real incident that must be scoped, contained, and escalated.
- You are writing or exercising IR playbooks, metrics, or the Lessons Learned report.

## When NOT to Use
- Formal evidence acquisition for litigation or law enforcement handoff requiring
  strict NIST SP 800-86 chain of custody; use `incident-response` and
  `disk-triage-hunter` for forensic-grade, write-blocked acquisition.
- Deep memory image analysis beyond triage (full Volatility plugin sweep, malware
  reverse engineering); use `memory-forensics-hunter`.
- Offensive validation of a vulnerability found during IR; that is out of scope here
  and belongs to the appropriate hunter skill under its own authorization gate.

## Authorization Check
- Confirm you are acting under the organization's IR policy and "right to monitor"
  notice; defensive triage on company-owned systems during a declared incident
  does not require separate pentest-style scope approval, but verify with the
  incident commander before touching user endpoints, cloud consoles, or third-party
  (SaaS) tenants.
- Confirm evidence handling requirements (regulatory, legal hold) before you run
  any command that changes system state; if full chain of custody is required,
  stop and hand off to `incident-response` / `disk-triage-hunter` instead.
- Confirm you have authority (or sign-off from the incident commander) before
  resetting credentials, isolating hosts, or blocking network paths.

## Methodology

1. **Orient with PICERL and the OODA loop before touching a keyboard.**
   - Work the SANS phases: Preparation, Identification, Containment, Eradication,
     Recovery, Lessons Learned (PICERL). Treat it as a feedback loop, not a line:
     containment actions and new IoCs can rescope the incident at any point.
   - Run Boyd's OODA loop (observe, orient, decide, act) inside each phase:
     observe raw data, orient using context and prior cases, decide on the next
     action, act, then re-observe. Avoid analysis paralysis by timeboxing the
     observe/orient steps.
   - Apply the Alexiou Principle to every open question: (1) what question are
     you answering, (2) what data answers it, (3) how do you extract that data,
     (4) what does the data actually say. Do not force data to fit a theory.
   - Watch for confirmation bias: cherry-picked evidence, a single attack path
     considered, containment chosen without testing it works, or a checklist
     followed so rigidly that contrary evidence gets ignored. If an investigation
     hasn't changed theory despite new data, pause and challenge the premise.
   - Externalize: keep timestamped notes in UTC, noting both event time and
     discovery time. If you are moving too fast to write notes, you are moving
     too fast.

2. **Identification: scope the alert before you act on it.**
   - Pull the risk object (hostname, username, internal IP, public IP) and check
     for other recent alerts tied to it; one analyst should own a risk object to
     avoid duplicated or conflicting work.
   - Establish the incident start date/time; an alert that trips today may reflect
     an initial compromise weeks earlier, which immediately rescopes the window
     of systems/users to review.
   - Cross-check internal vs external views of the same host: local netstat vs
     external Nmap scan vs on-wire tcpdump. A mismatch (ports reported locally
     that do not appear on the wire, or vice versa) is strong evidence of a
     rootkit or hidden process.
   - Decide: explainable activity, confirmed incident, or needs more data? Record
     severity (low/medium/high/critical) and whether to "watch and learn" (with a
     timebox) or contain immediately.

3. **Windows live response (run from a trusted, write-protected kit; collect in
   order of volatility).**
   - Prepare a collection target: sanitized USB, a SMB share (disable signing
     enforcement first with `Set-SmbClientConfiguration -RequireSecuritySignature
     $false` if needed), or a quick HTTP upload endpoint
     (`python3 -m pip install uploadserver && python3 -m uploadserver`).
   - Capture physical memory first (volatile, decays fastest): WinPMem
     (`winpmem_mini_x64.exe YYYYMMDD.HHMM.sys.dmp`) or an EDR's memory module.
   - Triage the image with Volatility 3 in this order: `windows.info` (validate
     OS/build), then `windows.pslist` / `psscan` / `pstree` / `cmdline` /
     `psxview` (process plus hidden-process check), then `windows.netstat` /
     `netscan` (network state), then `windows.malfind` / `hollowprocesses` /
     `yarascan.YaraScan` (malware check), then filesystem (`filescan`,
     `dumpfiles`), registry (`registry.hivelist`, `registry.printkey`), timeline
     (`timeliner.Timeliner`, `mftscan`), users (`sessions`, `getsids`), and
     services (`svcscan`).
   - Ask process-indicator questions for anything suspicious: is the image name
     spelled correctly (svch0st.exe vs svchost.exe)? Is it running from a normal
     path (not AppData\Local\Temp, not %SystemRoot% root, not Recycle Bin)? Does
     the parent-child relationship make sense? Is the command line obfuscated or
     encoded? Is the binary signed?
   - Collect live system state from the trusted kit, not local binaries:
     `whoami /all`, `whoami /groups /fo csv`, `ipconfig /all`, processes,
     services, ASEPs (autostart locations, scheduled tasks, registry Run keys),
     active network connections, installed hotfixes/apps, USB history, Volume
     Shadow Copy state, DNS cache.
   - Pull Windows Event Logs: 4624 (logon; use the positional or XML overlay
     method to extract logon type/source), 4625 (logon failure), 4688 (process
     creation; requires "Include command line in process creation events"
     enabled), 4771/4772 (Kerberos failures). Cross-reference with Sysmon Event
     ID 1 (process creation with hashes) where deployed.
   - For automated triage at scale, use KAPE (artifact collection) and
     DeepBlueCLI (automated Windows Event Log hunting) instead of doing every
     step by hand.

4. **Linux live response.**
   - Prepare storage (mount a USB or SAMBA share), then dump memory via AVML,
     LiME, or `netcat` to a remote host before anything else; analyze with
     Volatility 3.
   - Collect live state: processes (`ps auxef`), connections (`ss -tulpn`),
     kernel modules (`lsmod`), cron/systemd timers, `/etc/passwd` and
     `/etc/shadow` changes, `last`/`lastb`/`w`.
   - Use `lsof` to tie open files, sockets, and deleted-but-still-mapped binaries
     back to running processes; a process holding an open handle to a deleted
     executable is a strong persistence/rootkit indicator.
   - Pull filesystem and mount info (`cat /proc/mounts`, `lsblk -f`, `df -T`),
     USB history (`lsusb -v`, `dmesg | grep -i usb`, `udevadm info`), and file
     share exposure (NFS: `cat /etc/exports`, `showmount -e`; SAMBA:
     `smbstatus`, `cat /etc/samba/smb.conf`).
   - Collect logs: `journalctl --since="7 days ago"`, `journalctl -u ssh --since
     "24 hours ago" | grep -E "(Failed|Invalid|Refused)"`, `journalctl _COMM=sudo`
     / `_COMM=su` for privilege escalation, plus `/var/log` copies. Check
     package install history (`dpkg -l`, `/var/log/dpkg.log*`,
     `/var/log/apt/history.log*`) for unexplained installs in the compromise
     window.
   - Investigate commonly abused directories: `/tmp`, `/var/tmp`, `/dev/shm`,
     `/var/run`, user home directories, and recent changes under `/etc`.
   - For containment on Linux hosts, use `iptables`/`nft` rules to isolate a
     host at the network layer without a full power-down (preserves volatile
     state collected earlier).

5. **Network/packet triage (tcpdump, tshark, Wireshark).**
   - Capture close to the choke point (SPAN/mirror port, network tap, NGFW TLS
     break-and-inspect export) rather than only on the endpoint.
   - Find connection initiators vs responders: SYN-only count
     (`tcpdump -n -r <pcap> 'tcp[13]=0x02'`) vs SYN/ACK count
     (`tcpdump -n -r <pcap> '(tcp[13] & 0x12 == 0x12)'`). A large SYN-only
     excess over SYN/ACK indicates scanning or a network problem.
   - Extract unique source/destination/port tuples for data reduction:
     `tshark -r <pcap> -T fields -e ip.src -e tcp.srcport -e ip.dst -e
     tcp.dstport -E separator=,`.
   - Profile top talkers and least talkers: `tshark -r <pcap> -T fields -e
     ip.src | sort | uniq -c | sort -nr | head -10` (reverse sort for least
     talkers); isolate the top talker into its own PCAP to shrink the analysis
     surface for the rest.
   - Inspect DNS for manipulation: short TTLs, unexpected CNAME chains, or
     responses inconsistent with the query
     (`tshark -r <pcap> -T fields -e ip.src -e dns.qry.name -e dns.resp.ttl -Y
     "(udp.port==53||tcp.port==53) && dns.flags.response==1"`).
   - Check certificates for self-signed or sparsely populated fields (telltale
     sign of adversary infrastructure) and verify MAC-to-IP pairing stability
     (`tshark -r <pcap> -T fields -e eth.src -e ip.src -e eth.dst -e ip.dst |
     sort | uniq`); a MAC address binding to multiple IPs over time suggests
     spoofing or pivoting.
   - For HTTP/HTTP2/HTTP3, pull request method, host, full URI, and user agent
     to spot beaconing (regular pulse traffic to a low-reputation or newly
     registered domain) and credential-harvesting patterns in cleartext
     protocols (HTTP, SMTP, IMAP, POP3).

6. **Containment and eradication: stop the adversary without tipping your hand.**
   - Favor EDR-based host containment, account disable/credential reset, and
     targeted ACL/firewall/security-group changes over "pull the plug," which
     destroys volatile data and can damage media. Collect memory and a triage
     image first if time allows.
   - If Kerberos ticket abuse (Golden Ticket, DCSync) is suspected, rotate the
     krbtgt account password twice (it keeps two password history entries); plan
     around the default 10-hour max ticket lifetime.
   - Treat every containment action as a potential OpSec tip-off to a hands-on
     adversary; use out-of-band communications and avoid broadcasting response
     actions through channels the adversary may read.
   - Before declaring eradication complete, confirm root cause, sweep the
     enterprise for the full IoC set, and validate that no rootkit/backdoor
     remains (if a rootkit is confirmed, the system is not trustable: wipe,
     reformat, and restore from a known-clean backup rather than trying to
     clean in place).

7. **Recovery and Lessons Learned.**
   - Before returning a system to production, validate logging is active, the
     application/database stack is healthy, backups are verified, and no
     repeat-event indicators appear.
   - Add enhanced, incident-specific monitoring (new Suricata/Snort rules, extra
     SIEM alert logic, file integrity checks) tuned to the confirmed attack
     pattern before closing the case.
   - Run a blameless Lessons Learned session: write the follow-up report,
     capture what worked, update playbooks/detections against the MITRE ATT&CK
     techniques actually observed (checking for a missed initial detection),
     and track countermeasure adoption through an SBAR-style one-pager if
     executive sign-off is needed.

8. **Track metrics that drive improvement, not busywork.**
   - Use the Golden Rule: only track a metric you can tie to an improvement
     action. Avoid metrics that incentivize closing tickets fast over closing
     them right.
   - Core timers: time to data availability, MTTD (detect), MTTA (acknowledge),
     MTTI (identify/confirm as incident), MTTR (respond/initiate containment),
     MTTC (fully contained), MTTE (eradicate), MTRS (recover service). Measure
     both event time and discovery time in UTC for every milestone.

## Output Format
- A timestamped (UTC) incident working log following the Alexiou Principle:
  question, data source, extraction method, and finding for each investigative
  step.
- Collected artifacts: memory image, live-state command output, relevant Event
  Log/journal exports, and any PCAP extracts, stored in a case directory with
  hashes recorded at collection time.
- A PICERL-phase status summary (current phase, exit criteria met/open, next
  decision point) suitable for the incident commander's bridge-line update.
- A Lessons Learned report and SBAR-format countermeasure proposal where a
  control change requires approval.

## Quality Check
- Every claim of compromise or clean status is backed by at least one concrete
  data source and extraction command, not inference alone.
- Internal/external consistency checks (netstat vs Nmap vs tcpdump) were run for
  any host where rootkit or hidden-process activity is suspected.
- Confirmation bias check: was at least one alternative hypothesis considered and
  explicitly ruled out with data, not assumption?
- Containment actions were validated to actually work (tested, not assumed) before
  being reported as closing the exposure.

## Common Issues
- Collecting disk images before memory: always capture memory first, since it
  decays and is lost on power-down or reimage.
- Running collection tools from the (possibly compromised) local system instead
  of a trusted, write-protected kit; this risks tainted output and evidence
  spoliation.
- Treating a single data source as proof of compromise; it is usually easier to
  disprove malicious activity with one source than to prove it, so seek a second
  corroborating source before escalating severity.
- Following a checklist so rigidly that it suppresses a better investigative
  question; checklists guide, they do not replace analyst judgment.
- Forgetting that containment can rescope the incident; new IoCs discovered while
  isolating one host often expand (or shrink) the blast radius and must loop back
  into Identification.
