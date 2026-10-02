---
name: security-ops-bash
description: "Collect, parse, baseline, and triage host and log data from the command line using core Unix text tools, then automate the result into bash scripts."
model: sonnet
metadata:
  version: 1.0.0
  category: blue-team
  source: "Cybersecurity Ops with bash (O'Reilly)"
---

# Security Ops with bash

## Goal
Use core command-line tools (grep, awk, sed, sort, uniq, cut, tr, join, find, xxd) to collect
system and log data, parse it into structured fields, baseline normal behavior, detect anomalies
through frequency analysis, triage suspicious files statically, and automate the whole workflow
into reusable bash scripts and dashboards, without installing heavyweight tooling.

## When to Use
- Need to pull logs, running processes, registry, or filesystem state from a Linux or Windows
  (Git Bash) host for review, with no agent or SIEM available.
- Need to turn a raw web server, auth, or system log into counts, sums, or a histogram to spot
  outliers (spikes in 404s, data volume, unusual user agents, off-hours activity).
- Need a lightweight log-based IDS (tail -f plus a pattern file of indicators of compromise).
- Need to baseline a filesystem (hash every file) and later detect added, removed, changed, or
  relocated files, or baseline open ports and alert on drift.
- Need to statically triage an unknown binary or JSON/XML blob (magic numbers, strings, hex,
  VirusTotal hash lookup) before deciding on deeper reverse engineering.
- Need to turn any of the above into a cron/schtasks job, an HTML report, or a live dashboard.

## When NOT to Use
- Exploit development, payload crafting, or active offensive tooling (out of scope; this skill is
  defensive data collection and analysis only).
- Large-scale or high-volume log analytics better served by a SIEM, ELK, or a real database;
  bash pipelines are for rapid triage, not petabyte-scale correlation.
- Dynamic malware analysis (sandbox detonation); this skill covers static triage only.

## Authorization Check
- Confirm you have permission/legal authority to collect data from the target host before running
  any collection script (see Chapter 5 principle: gather broadly, but only where authorized).
- For anything touching production or another team's systems, confirm scope in
  `.claude/security-scope.yaml` or get explicit owner sign-off before running baselines, port
  scans, or remote SSH collection against it.
- Malware samples: analyze only on an isolated, non-networked system; never upload files with
  sensitive or privileged content to third-party services like VirusTotal.

## Methodology

1. **Data collection**: gather logs, system state, and files before anything else; you can delete
   later but cannot analyze what you never captured.
   - Logfiles: Linux under `/var/log/` (`auth.log`, `syslog`, `kern.log`, app-specific); Windows
     via `wevtutil el` (list logs) and `wevtutil qe "<log>" //c:1 //rd:true` (query recent, use
     `//` in Git Bash). Archive with `tar -czf ${HOSTNAME}_logs.tar.gz /var/log/`.
   - System state: `uname -a`, `ps -e` / `tasklist`, `netstat -a`, `ifconfig` / `ipconfig`, `mount`
     / `net share`; wrap output in simple XML tags so later tooling can parse it reliably.
   - Filesystem search: `find <path> -name '*pattern*'`, `-mtime`/`-atime` for recent activity,
     `-size` for outliers, `file` command plus known magic-number patterns to find a file type
     regardless of its extension, `sha1sum`/`md5sum` to search by exact content hash.
   - Remote collection: `ssh host <cmd>`, or stream a local script to a remote shell with
     `ssh host bash < script.sh` to avoid a two-step copy-then-run. Transfer results back with
     `scp`, never with plaintext credentials embedded in scripts; use SSH keys.

2. **Data processing**: normalize raw text into clean fields before analysis.
   - Delimited data: `cut -d',' -f2` for whole columns, `cut -c2-13` for fixed-width character
     ranges. Note `cut` does not collapse repeated delimiters; use `awk` instead when spacing is
     irregular.
   - Line-by-line field logic: `awk -F',' '{print $4}'`, `awk '$9 == 404 {print $1}'` to filter and
     extract in one pass.
   - Substitution and cleanup: `sed 's/old/new/g'` for text replacement, `tr -d '"'` to strip
     stray characters, `tr '\r' ''` or `sed -i 's/$/\r/'` to fix Windows/Linux line-ending
     mismatches.
   - Merge two related files on a shared key with `join` (both inputs must be pre-sorted on the
     join field: `join -1 3 -2 1 -t, file_a file_b`).
   - XML/JSON without a parser: `grep -o '<tag>.*</tag>'` then `sed 's/<[^>]*>//g'` to strip tags;
     prefer `jq '.path.to.field'` when available.

3. **Data analysis (frequency and outlier detection)**: start broad, narrow as insight is gained.
   - Counting: `cut -d' ' -f1 access.log | sort | uniq -c | sort -rn` (or a `declare -A` bash
     associative array / `awk '{cnt[$1]++}'` for a single-pass count on huge files).
   - Totals: sum a numeric field per key the same way, replacing `cnt[$id]++` with
     `cnt[$id]+=$value` (e.g. total bytes transferred per host, to spot exfiltration).
   - Visualize counts as a horizontal bar histogram (scale each count to a fixed max width of `#`
     characters) to make spikes in time-bucketed or per-host data visually obvious.
   - Anomaly detection by exception: maintain a small allowlist (known user agents, known
     processes) and flag anything that does not match via `=~` regex comparison; this surfaces web
     crawlers, scanners, and spoofed clients that don't match any expected signature.
   - Interesting signals from web/app logs: repeated 404s from one source (probing), repeated 401s
     (brute force), one source hitting nearly every page exactly once (site cloning / crawler),
     byte-count or request-count outliers, and activity outside normal business hours.

4. **Real-time log monitoring and lightweight IDS**:
   - `tail -f logfile | egrep --line-buffered -i -f ioc.txt` to alert on a file of regex indicators
     of compromise (directory traversal `\.\./`, `etc/passwd`, `etc/shadow`, `cmd\.exe`, shell
     paths) as they appear; always use `--line-buffered` or output is batched and delayed.
   - Pipe matches into `tee -a alerts.log` to display and persist simultaneously.
   - For Windows logs without a native tail, poll `wevtutil qe` in a loop and diff against the
     last-seen entry.
   - Build rolling counts on streamed data using a signal-based design: a counting loop that resets
     on `SIGUSR1` (via `trap`), driven by a second script that sleeps N seconds and sends the signal;
     use `shopt -s lastpipe` so the counting loop shares the parent's PID instead of running in a
     subshell.

5. **Baselining for intrusion / integrity detection**:
   - Filesystem: `find / -type f | xargs -d '\n' sha1sum > baseline.txt` on a known-good system.
     Re-verify later with `sha1sum -c --quiet baseline.txt` to catch changed files; use `find` plus
     `join` (both files sorted on path) to catch newly added files that have no baseline entry;
     `sdiff -s` gives a quick side-by-side diff of two sorted file lists.
   - Network: scan each host's open TCP ports with bash's `/dev/tcp/<host>/<port>` pseudo-device
     (no nc/nmap required), save dated results, and diff successive scans to flag newly opened or
     closed ports, which can indicate a backdoor or unauthorized service.
   - Automate both with `crontab -e` (Linux, standard 5-field schedule) or `schtasks //Create //SC
     DAILY //ST HH:MM //TR "<cmd>"` (Windows), and email or alert on any detected delta.

6. **Malware static triage**: never on a networked or production system; assume the analysis host
   is burned afterward.
   - Identify file type by magic number, not extension: `file somefile`; magic numbers include
     `FF D8 FF DB` (JPEG), `4D 5A` (DOS/PE exe), `7F 45 4C 46` (ELF), `50 4B 03 04` (zip).
   - Inspect raw bytes with `xxd -s <offset> -l <len> file` (hex) or add `-b` for binary; convert
     between hex/decimal/ASCII with `printf "%d" 0x41` / `printf "%x" 65` / `xxd -r`.
   - Extract readable strings without the `strings` binary:
     `egrep -a -o '\b[[:print:]]{2,}\b' file | awk '{print length(), $0}' | sort -rnu` (longest,
     most interesting strings first).
   - Check reputation via VirusTotal's REST API with `curl` (hash lookup first to avoid uploading
     sensitive files; upload only as a last resort, and never upload files containing sensitive or
     privileged data).

7. **Formatting and reporting**: make the output consumable by someone other than you.
   - Wrap extracted fields in HTML tags (a small `tagit()` helper: `printf '<%s>%s</%s>\n'`) to turn
     a raw log into a browser-viewable, printable table.
   - Build a live terminal dashboard with `tput` (cup, el, rev, sgr0, smul/rmul) to redraw sections
     in place, each ending in an erase-to-end-of-line so stale output never lingers; refresh on a
     `sleep N` loop and clean up background jobs with a `trap cleanup EXIT`.

8. **Automation and packaging**: once a pipeline proves useful, wrap it in a script with
   `getopts` for options, meaningful defaults (`${VAR:-default}`), and a documented usage header;
   schedule it (cron/schtasks) and route alerts by email or file drop so detection runs continuously
   without manual re-invocation.

## Output Format
- Dated, hostname-prefixed collection archives (`${HOSTNAME}_logs.tar.gz`, `scan_YYYY-MM-DD`,
  `baseline.txt`) for timelining across multiple runs.
- XML- or HTML-wrapped structured output from raw command text for later machine or human
  consumption.
- Histogram/bar-chart text output for quick visual triage of counts or sums.
- A findings list of anomalies (new/changed/removed files, new open ports, IOC matches, unknown
  user agents) with enough context (host, path, timestamp) to act on each one.
- Scheduled scripts (cron entries or scheduled tasks) plus the alerting mechanism (email, log file)
  that keeps the detection running unattended.

## Quality Check
- Every collection script declares where it reads from and where it writes to, and uses `sudo`
  only where genuinely required to reach protected files.
- Counting/summing pipelines are validated against a `sort | uniq -c` cross-check before trusting
  a custom awk/bash tally on a new dataset.
- Baseline comparisons are tested against a deliberately modified/added/removed file before being
  trusted on a live host, to confirm the three cases (changed, new, removed) are all detected.
- IOC pattern files are anchored and specific enough to avoid matching benign traffic (test
  against a known-clean log sample before deploying to a live tail).
- Any script processing untrusted input (log lines, file content) quotes variables and handles
  the no-match / empty-field case without crashing the pipeline.

## Common Issues
- Forgetting `--line-buffered` on grep/egrep in a `tail -f` pipeline, so matches appear in bursts
  or not at all instead of in real time.
- Using `cut` on a file with inconsistent whitespace; `cut` treats every delimiter as a field
  break and does not collapse repeats, so columns silently shift. Use `awk` instead, which
  collapses repeated whitespace when splitting fields by default.
- Running `join` on unsorted input; `join` silently drops or mismatches rows if both files are not
  sorted on the join key first.
- Trusting the `file` command's magic-number database on a potentially compromised host; a
  malicious user can tamper with `/usr/share/misc/magic`. Mount the suspect drive on a known-good
  system instead.
- Scaling a histogram off a single sample without allowing for rescaling; a later spike that
  exceeds the original max silently overflows the display width.
- Uploading a sensitive or privileged file to VirusTotal to "just check" it; search by hash first,
  since VirusTotal retains everything uploaded.
- Leaving background helper processes (tail -f, counters) running after a dashboard or monitor
  script exits; always pair a background launch with a `trap cleanup EXIT` that kills the job.
