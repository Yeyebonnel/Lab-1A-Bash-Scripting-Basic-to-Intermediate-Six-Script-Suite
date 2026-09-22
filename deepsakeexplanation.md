# ITP 141 — Lab 1A: Bash Scripting (Basic to Intermediate), Six-Script Suite

**Course:** ITP 141 — Systems Administration and Maintenance  
**Program:** BS Information Technology  
**Institution:** Davao Oriental State University — Faculty of Computing, Engineering and Technology  
**Semester:** First Semester, AY 2026–2027  
**Instructor:** Ar-Jay R. Sacay, MIT  
**Points:** 50 pts | **CLOs:** CLO1, CLO2  
**Module:** Module 1, Topic 5 — Intermediate Bash Scripting  
**Weeks:** 6–7  
**Platform:** Ubuntu Server 24.04 LTS | Bash 5.2+ | nano / vim

---

## Scenario

The Information Technology Services Unit (ITSU) of Davao Oriental State University maintains a small group of Ubuntu servers that carry the BSIT Program's internal file shares, the student attendance database, and several faculty project sites. Because only two technicians support the whole program, routine housekeeping is done by hand: oversized log files are rotated, the shared drive is copied to a second disk, and the accounts consuming the most storage are reviewed.

During enrolment these tasks are quietly skipped, which is precisely when the servers are under the heaviest load. ITSU has asked us to remove this work from the technicians by writing a suite of Bash scripts that run without supervision.

---

## Objectives

1. Write six Bash scripts that apply variables, command substitution, ANSI colour output, conditionals with test operators, loops, functions with positional arguments, arrays, and parameter expansion.
2. Implement `log_rotate.sh`, using a function that takes the log path as a positional argument, so that exactly the five most recent versions of a log file are retained.
3. Build `backup_auto.sh` with strict mode (`set -euo pipefail`), a trap on `ERR`, rsync, timestamped destination directories, and an append-only log file, then verify the backup by restoring one file from it.
4. Schedule `script2_diskcheck.sh` with cron and verify execution from both the redirected log file and the cron service journal.

---

## Group Members and Roles

| No. | Full Name | Assigned Role | Tasks Completed |
|-----|-----------|---------------|-----------------|
| 1 | *(Member 1)* | Script Writer A | `script1_sysinfo.sh`, `script2_diskcheck.sh` |
| 2 | *(Member 2)* | Script Writer B | `script3_userreport.sh`, `script4_cleanup.sh` |
| 3 | *(Member 3)* | Log Rotation & Backup Dev | `log_rotate.sh`, `backup_auto.sh` |
| 4 | *(Member 4)* | Scheduler & Tester | Cron, execution table, group report |

---

## Repository Structure

```
lab1a_bash/
├── script1_sysinfo.sh
├── script2_diskcheck.sh
├── script3_userreport.sh
├── script4_cleanup.sh
├── log_rotate.sh
├── backup_auto.sh
├── backup.log
├── error.log
├── cron_disk.log
├── README.md
└── .gitignore
```

---

## Overview

The lab is divided into four roles across four group members. Each script teaches a different Bash concept and together they automate the routine housekeeping tasks ITSU would otherwise do by hand. The scripts are scheduled to run unsupervised through cron so that they continue working even when the technicians are busy.

---

## Script 1 — `script1_sysinfo.sh` (Member 1)

### What It Does
Prints five facts about the machine — hostname, uptime, current user, date, kernel version — each value wrapped in green ANSI colour.

### The Code
```bash
#!bin/bash

hostname=$(hostname)
uptime=$(uptime -p)
current_user=$(whoami)
current_date=$(date)
kernel_version=$(uname -r)

echo -e "\e[32mHostname: $hostname\e[0m"
echo -e "\e[32mUpime: $uptime\e[0m"
echo -e "\e[32mCurrent User: $current_user\e[0m"
echo -e "\e[32mDate: $current_date\e[0m"
echo -e "\e[32mKernel Version: $kernel_version\e[0m"
```

### How It Works
- `$(command)` is **command substitution** — runs the command and stores the output.
- `hostname=$(hostname)` stores the machine's hostname in a variable.
- Same pattern for `uptime -p`, `whoami`, `date`, and `uname -r`.
- `echo -e "...\e[32m$VAR\e[0m"` prints the value in green. `\e[32m` starts green; `\e[0m` resets it. The `-e` flag tells bash to interpret the escape codes.

### Why It Matters
A quick "what machine am I on and what's it running" report — useful as a first check when logging into a server, or as a line in a daily status log.

### Bash Concepts Demonstrated
Variables, command substitution, ANSI colour output.

### Notes
The shebang line at the top reads `#!bin/bash` — this is a typo. It should be `#!/bin/bash` (with a slash). Without the slash, the kernel will not know which interpreter to use and running `./script1_sysinfo.sh` directly would fail. Running it with `bash script1_sysinfo.sh` still works because bash is invoked explicitly. Also, the "Uptime" label reads "Upime" — a minor typo, but the value still prints correctly.

---

## Script 2 — `script2_diskcheck.sh` (Member 1)

### What It Does
Reads the disk usage of the root filesystem and reports **CRITICAL** (above 90%), **WARNING** (above 70%), or **OK**.

### The Code
```bash
#!/bin/bash

#usage=80
usage=$(df / | awk 'NR==2 {print $5}' | tr -d '%')

if [ "$usage" -gt 90 ]; then
	echo "CRITICAL: Disk usage is ${usage}%"
elif [ "$usage" -gt 70 ]; then
	echo "WARNING: Disk usage is ${usage}%"
else
	echo "OK: Disk usage is ${usage}%"
fi
```

### How It Works
- `df /` shows disk usage of the root filesystem.
- `awk 'NR==2 {print $5}'` grabs the fifth column of the second line — the usage percentage.
- `tr -d '%'` removes the percent sign so the number can be compared.
- The `if`/`elif`/`else` block tests the number in descending order: 90 first, then 70, then OK.
- The `#usage=80` line is a commented-out hard-coded value used for testing all three outcomes.

### Why Descending Order Matters
If the `> 70` test came first, a disk at 95% usage would match the WARNING branch and never reach CRITICAL. Testing the highest threshold first ensures the most urgent message wins.

### Why It Matters
A full disk is the most common cause of "the server suddenly stopped working." This script provides early warning before the disk is completely full.

### Bash Concepts Demonstrated
Conditionals (`if`/`elif`/`else`), integer comparison (`-gt`), pipes, `awk`, `tr`.

### Testing All Three Outcomes
Uncomment `#usage=80` and re-run the script with different values to test the WARNING and CRITICAL branches.

---

## Script 3 — `script3_userreport.sh` (Member 2)

### What It Does
Checks three accounts (`sysadmin`, `root`, `nobody`) and reports each one's `id` info or "NOT FOUND," then prints how many accounts were checked.

### The Code
```bash
#!/bin/bash

USERS=(sysadmin root nobody)

check_user(){
if id "$1" &>/dev/null; then
   echo "User: $1"
   id "$1"
else
    echo "$1: NOT FOUND"
fi

}
for user in "${USERS[@]}"; do
	check_user "$user"
done

echo "Total users checked: ${#USERS[@]}"
```

### How It Works
- `USERS=(sysadmin root nobody)` — an **array** holding three account names.
- `check_user()` — a **function** that takes the account name as its positional argument `$1`.
- Inside the function, `id "$1"` looks up the account. Success → print info. Failure → print "NOT FOUND."
- `${USERS[@]}` expands to all elements of the array.
- `${#USERS[@]}` returns the number of elements in the array.

### Why It Matters
ITSU uses this to review which accounts exist. Knowing an account is missing is as important as knowing it's present — unexpected accounts can mean security problems.

### Bash Concepts Demonstrated
Arrays, functions with positional arguments, loops, `${#array[@]}`.

---

## Script 4 — `script4_cleanup.sh` (Member 2)

### What It Does
Deletes files older than 7 days in a **disposable test directory** — never on live data. Uses strict mode and a trap to log any errors.

### The Code
```bash
#!/bin/bash

set -euo pipefail

trap 'echo "ERROR at line $LINENO" >> ~/lab1a_bash/error.log; exit 1' ERR

mkdir -p ~/lab1a_bash/tmptest

touch ~/lab1a_bash/tmptest/old_file.txt
touch -d '10 days ago' ~/lab1a_bash/tmptest/old_file.txt

touch ~/lab1a_bash/tmptest/new_file.txt

echo "Before"
ls -la ~/lab1a_bash/tmptest/

find ~/lab1a_bash/tmptest -mtime +7 -delete

echo "After"

ls -la ~/lab1a_bash/tmptest/

echo "Cleanup completed successfully"

ls /this/path/does/not/exist

exit 0
```

### How It Works
- `set -euo pipefail` — **strict mode:**
  - `-e`: exit immediately if any command fails.
  - `-u`: exit if any undefined variable is used.
  - `-o pipefail`: if any command in a pipe fails, the whole pipe fails.
- `trap '...' ERR` — if any command fails, the trap runs. It writes `"ERROR at line $LINENO"` to `error.log` and exits with code 1.
- `mkdir -p` creates the test directory.
- `touch` creates files; `touch -d '10 days ago'` backdates a file by 10 days.
- `find ~/lab1a_bash/tmptest -mtime +7 -delete` finds and deletes files older than 7 days.
- The final line `ls /this/path/does/not/exist` deliberately fails — this triggers the trap and proves the error logging works.

### The Error Log
After running the script, `error.log` contains:
```
ERROR at line 26
ERROR at line 26
```
Line 26 is the `ls /this/path/does/not/exist` command. This confirms the trap fires correctly on failure.

### Why It Matters
ITSU's servers accumulate old files. This cleans them up — but on a **test directory**, never on live `/tmp`. Files in `/tmp` may be in use by running services; deleting them would break those services.

### Bash Concepts Demonstrated
`set -euo pipefail`, `trap ... ERR`, `find -mtime`, `$LINENO`, exit codes.

---

## Script 5 — `log_rotate.sh` (Member 3)

### What It Does
Keeps only the **five most recent versions** of a log file, deleting the oldest each time a new rotation happens.

### The Code
```bash
#!/bin/bash

rotate_log() {
   local logfile="$1"
   local base="${logfile%.log}"

   rm -f "${base}.5.log"

   [ -f "${base}.4.log" ] && mv "${base}.4.log" "${base}.5.log"
   [ -f "${base}.3.log" ] && mv "${base}.3.log" "${base}.4.log"
   [ -f "${base}.2.log" ] && mv "${base}.2.log" "${base}.3.log"
   [ -f "${base}.1.log" ] && mv "${base}.1.log" "${base}.2.log"

   [ -f "$logfile" ] && mv "$logfile" "${base}.1.log"

}

mkdir -p ~/lab1a_bash/logs
rm -f ~/lab1a_bash/logs/app.*.log

touch ~/lab1a_bash/logs/app.log

for i in $(seq 1 7); do
    echo "Log entry $i at $(date)" >> ~/lab1a_bash/logs/app.log
    echo "---------- Rotation $i -----------"
    rotate_log ~/lab1a_bash/logs/app.log
    ls -la ~/lab1a_bash/logs
    echo
done
```

### How It Works
- `rotate_log()` takes the log file path as its positional argument `$1`.
- `base="${logfile%.log}"` — **parameter expansion** strips the `.log` suffix. So `app.log` becomes `app`.
- `rm -f "${base}.5.log"` deletes the oldest version (`.5`).
- Four `mv` lines shift each version up by one: `.4→.5`, `.3→.4`, `.2→.3`, `.1→.2`.
- `[ -f "$logfile" ] && mv "$logfile" "${base}.1.log"` moves the current log to `.1`.
- The loop runs **seven times**, appending a line before each rotation and listing the directory afterwards.
- After 5 rotations, the count caps at 5 — the oldest version is deleted each time.

### Why the Cleanup Lines Exist
Before the loop runs, the script deletes any `app.*.log` files left over from a previous run. Without this, a second run would start with 5 stale files and the buildup would not be visible. This ensures a clean 1→2→3→4→5 progression every time.

### Why It Matters
Log files grow daily. Left unrotated, they fill the disk. Real Linux systems use `logrotate` for this; the script shows how it works under the hood.

### Bash Concepts Demonstrated
Functions with parameters, parameter expansion (`${var%.log}`), file tests (`-f`), loops, `rm`, `mv`, `touch`.

---

## Script 6 — `backup_auto.sh` (Member 3)

### What It Does
Copies the logs directory into a **timestamped backup folder** and appends the rsync result to a log file.

### The Code
```bash
#!/bin/bash

set -euo pipefail

trap 'echo "FAILED $(date -Is)" >> ~/lab1a_bash/backup.log; exit 1' ERR

DEST=~/lab1a_bash/backup_$(date +%Y%m%d_%H%M%S)

mkdir -p "$DEST"

echo "Backup started: $(date -Is)"
rsync -avz ~/lab1a_bash/logs/ "$DEST" >> ~/lab1a_bash/backup.log 2>&1
echo "Backup completed: $(date -Is)"
```

### How It Works
- `set -euo pipefail` — strict mode.
- `trap '...' ERR` — logs `"FAILED <timestamp>"` and exits if any command fails.
- `DEST=~/lab1a_bash/backup_$(date +%Y%m%d_%H%M%S)` — builds a folder name from the current date and time, e.g. `backup_20260923_143001`.
- `mkdir -p "$DEST"` — creates the destination directory.
- `rsync -avz ~/lab1a_bash/logs/ "$DEST"` — copies the logs into the destination. `-a` preserves attributes, `-v` prints verbose output, `-z` compresses during transfer.
- `>> ~/lab1a_bash/backup.log 2>&1` — appends both stdout and stderr to the log.

### The Backup Log
After running twice, `backup.log` contains two rsync transfer summaries:
```
sending incremental file list
./
app.1.log
app.2.log
app.3.log
app.4.log
app.5.log

sent 455 bytes  received 114 bytes  1,138.00 bytes/sec
total size is 220  speedup is 0.39
...
```
Each block corresponds to one run of the script.

### Why It Matters
A backup that has never been restored is only an assumption. The lab requires running the script twice (to prove timestamped dirs work) and then restoring a file from the first directory — proving the backup is valid.

### Bash Concepts Demonstrated
`set -euo pipefail`, `trap ... ERR`, `$(date +...)`, `rsync`, `mkdir -p`, `>> ... 2>&1`.

---

## Cron Configuration (Member 4)

### What It Does
Tells the system to run `script2_diskcheck.sh` automatically at **minute 5 of every hour**.

### Setup
```bash
crontab -e
```

Add:
```cron
5 * * * * /bin/bash /home/bonneljhon/lab1a_bash/script2_diskcheck.sh >> /home/bonneljhon/lab1a_bash/cron_disk.log 2>&1
```

### How the Schedule Works
- `5` — run at minute 5
- `*` — every hour
- `*` — every day of month
- `*` — every month
- `*` — every day of week

So the job runs at 00:05, 01:05, 02:05, and so on.

### Why Absolute Paths
Cron does not expand `~`. Every path in the crontab entry must be absolute.

### Verification
```bash
crontab -l
cat ~/lab1a_bash/cron_disk.log
journalctl -u cron --since "20 min ago"
```

### Why journalctl, Not syslog
The minimum Ubuntu Server install used in this course does not include rsyslog, so `/var/log/syslog` may not exist. `journalctl` reads from systemd's journal, which is always present.

### Why It Matters
Cron is how the ITSU scripts run without a technician watching. Verifying from **two sources** (redirected log + journal) proves the job actually ran.

---

## Section 4 — Bash vs Python

*Analytical only; no Python code is written or submitted in ITP 141.*

| Criterion | Bash | Python | Preferred (B/P) |
|-----------|------|--------|-----------------|
| **Execution speed** | Starts fast, no interpreter boot. Best for short scripts and cron jobs. | Slower start, faster for heavy computation. | **B** |
| **Code readability / maintainability** | Short and terse. Hard to read when long. | Clear structure. Easy to read and edit. | **P** |
| **System command integration** | Native. Pipes and commands work directly. | Needs extra code to run commands. | **B** |
| **Error handling capability** | Basic. Uses exit codes and traps. | Strong. Uses try/except and logging. | **P** |
| **Available libraries** | Uses system tools like awk, grep, sed. | Huge library collection. | **P** |

**Conclusion:** Bash wins on speed and system integration (2/5). Python wins on readability, error handling, and libraries (3/5). For short scripts that call system tools — like the six in this lab — Bash is the right choice.

---

## Section 7 — Group Reflection

> We would automate **log rotation first** on the ITSU production server. Log files grow every day, and if they are not rotated, the disk will fill up and services will stop working. This is exactly what happens during enrolment, when the two ITSU technicians are too busy to do housekeeping and the server is under the heaviest load. Log rotation is also the safest task to automate first because it only creates numbered versions of the log and deletes the oldest one — it never touches live data. Once rotation is running reliably on cron, we would add the timestamped rsync backup, and then the disk usage report. This order matches the ITSU scenario: remove the most frequent and highest-risk manual task first, then build from there. The cron schedule makes sure these tasks run without anyone watching, which is what the ITSU team actually needs.

---

## How Everything Fits Together

| Step | Script | Purpose |
|------|--------|---------|
| 1 | `script1_sysinfo.sh` | Reports what machine you're on |
| 2 | `script2_diskcheck.sh` | Warns before the disk fills |
| 3 | `script3_userreport.sh` | Audits user accounts |
| 4 | `script4_cleanup.sh` | Cleans old files safely |
| 5 | `log_rotate.sh` | Keeps logs from growing forever |
| 6 | `backup_auto.sh` | Backs up logs to a timestamped folder |
| 7 | Cron | Runs `script2_diskcheck.sh` automatically |
| 8 | Worksheet | Documents what was built, compared, and learned |

Together, they turn ITSU's manual housekeeping into an automated system — exactly what the scenario asks for.

---

## Key Bash Concepts Demonstrated

| Concept | Where |
|---------|-------|
| Variables + command substitution | Script 1 |
| ANSI colour output | Scripts 1, 2 |
| Conditionals with integer comparison | Script 2 |
| Arrays | Script 3 |
| Functions with positional arguments | Scripts 3, 5 |
| Strict mode (`set -euo pipefail`) | Scripts 4, 6 |
| Trap on ERR | Scripts 4, 6 |
| Parameter expansion | Script 5 |
| rsync + timestamped directories | Script 6 |
| Cron scheduling | Step 8 |

---

## Grading Rubric (50 pts)

| Criterion | Pts |
|-----------|-----|
| Scripts 1 & 2: Variables, ANSI Colour & Conditionals | 20 |
| Scripts 3 & 4: Function + Array & trap Error Handling | 14 |
| Scripts 5 & 6: Log Rotation + rsync Backup | 10 |
| Cron Schedule, Execution Table & Reflection | 6 |
| **TOTAL** | **50** |

---

## References

- Robbins, A., & Beebe, N. H. F. (2022). *Classic shell scripting* (2nd ed.). O'Reilly Media.
- Nugraha, L. (2024, June 25). *Bash scripting tutorial for beginners*. Hostinger Tutorials.
- NIST SP 800-40 Rev. 4 — Guide to Enterprise Patch Management Planning.

---

## License

Coursework for ITP 141 — Systems Administration and Maintenance.  
Davao Oriental State University — Faculty of Computing, Engineering and Technology.
