## Group Members and Roles

| No. | Full Name | Assigned Role | Tasks Completed |
|-----|-----------|---------------|-----------------|
| 1 | *(Member 1)* | Script Writer A | `script1_sysinfo.sh`, `script2_diskcheck.sh` |
| 2 | *(Member 2)* | Script Writer B | `script3_userreport.sh`, `script4_cleanup.sh` |
| 3 | *(Member 3)* | Log Rotation & Backup Dev | `log_rotate.sh`, `backup_auto.sh` |
| 4 | *(Member 4)* | Scheduler & Tester | Cron configuration, all scripts run, execution table, group report |

### STEP 1 - Working Directory (All Members)
PDF: "Create the working directory: mkdir ~/lab1a_bash && cd ~/lab1a_bash. All six scripts are written in this directory."

```bash
mkdir ~/lab1a_bash && cd ~/lab1a_bash
```
### STEP 2 - script1_sysinfo.sh (Member 1)
"Store the hostname, uptime, current user, date, and kernel version in variables, obtaining each through command substitution. Print every value in ANSI colour `(echo -e "\e[32m$value\e[0m")`. Test with bash script1_sysinfo.sh and capture the output."

```bash
nano script1_sysinfo.sh
```
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
### OUTPUT:
```bash
bash script1_sysinfo.sh
```
<img width="581" height="118" alt="image" src="https://github.com/user-attachments/assets/06f2e73d-71c3-47fc-8b75-bab7cb4aea55" />

### STEP 3 - script2_diskcheck.sh (Member 1)
Read the usage of / with df. Test the thresholds in descending order, since an ascending test can never reach the higher branch: print CRITICAL above 90 per cent, WARNING above 70 per cent, and OK otherwise. Use if-elif-else with integer comparison, and test the script against all three outcomes.
```bash
nano script2_diskcheck.sh
```
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
### OUTPUT:
```bash
bash script2_diskcheck.sh
```
### If the usage is less than 70:
```bash
OK: Disk usage is 1%
```
### If the usage is greater than 70:
```bash
WARNING: Disk usage is 80%
```
### If the usage is greater than 90:
```bash
CRITICAL: Disk usage is 100%
```

### STEP 4 - script3_userreport.sh (Member 2)
Declare USERS=(sysadmin root nobody). Write a function check_user() that takes the account name as its positional argument $1 and prints either the output of id or NOT FOUND. Loop over the array calling the function for each element, and report the element count with ${#USERS[@]}.

```bash
nano script3_userreport.sh
```
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
### OUTPUT:
```bash
bash script3_userreport.sh
```
<img width="607" height="138" alt="image" src="https://github.com/user-attachments/assets/ede163a5-81d6-47fd-a364-77c004696bc6" />

### STEP 5 - script4_cleanup.sh (Member 2)
Begin with set -euo pipefail, then register trap 'echo "ERROR at line $LINENO" >> ~/lab1a_bash/error.log; exit 1' ERR. Create a disposable test tree first (mkdir -p ~/lab1a_bash/tmp/test, then create files and age them with touch -d '10 days ago'), and run find ~/lab1a_bash/tmp/test -mtime +7 -delete against it. Never point this command at the live /tmp... Exit 0 on success. Trigger an intentional error and confirm that error log records it.

```bash
nano script4_cleanup.sh
```
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

#ls /this/path/does/not/exist
exit 0
```

### OUTPUT:
<img width="592" height="258" alt="image" src="https://github.com/user-attachments/assets/55ee3ba9-6eb6-4bd4-acfb-2e7587e33231" />

Trigger an intentional error and confirm that error log records it
```bash
cat error.log
```
```bash
ERROR at line 19
ERROR at line 19
```

### STEP - 6 log_rotate.sh (Member 3)
Write rotate_log(), taking the log path as $1. Delete version .5, then shift .4 to .5, .3 to .4, .2 to .3, .1 to .2, and the current file to .1. Build the numbered names with parameter expansion... Create ~/lab1a_bash/logs/app.log and append a line to it before each rotation. Run rotate_log seven times, listing the directory after each run, and confirm that the numbered versions never exceed five (.1 to .5) and that the oldest is discarded on every run after the fifth

```bash
nano log_rotate.sh
```
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
### OUTPUT:
```bash
bash log_rotate.sh
```

```bash
---------- Rotation 1 -----------
total 12
drwxr-xr-x 2 bonneljhon bonneljhon 4096 Sep 23 00:32 .
drwxr-xr-x 7 bonneljhon bonneljhon 4096 Sep 23 00:32 ..
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.1.log

---------- Rotation 2 -----------
total 16
drwxr-xr-x 2 bonneljhon bonneljhon 4096 Sep 23 00:32 .
drwxr-xr-x 7 bonneljhon bonneljhon 4096 Sep 23 00:32 ..
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.1.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.2.log

---------- Rotation 3 -----------
total 20
drwxr-xr-x 2 bonneljhon bonneljhon 4096 Sep 23 00:32 .
drwxr-xr-x 7 bonneljhon bonneljhon 4096 Sep 23 00:32 ..
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.1.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.2.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.3.log

---------- Rotation 4 -----------
total 24
drwxr-xr-x 2 bonneljhon bonneljhon 4096 Sep 23 00:32 .
drwxr-xr-x 7 bonneljhon bonneljhon 4096 Sep 23 00:32 ..
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.1.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.2.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.3.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.4.log

---------- Rotation 5 -----------
total 28
drwxr-xr-x 2 bonneljhon bonneljhon 4096 Sep 23 00:32 .
drwxr-xr-x 7 bonneljhon bonneljhon 4096 Sep 23 00:32 ..
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.1.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.2.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.3.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.4.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.5.log

---------- Rotation 6 -----------
total 28
drwxr-xr-x 2 bonneljhon bonneljhon 4096 Sep 23 00:32 .
drwxr-xr-x 7 bonneljhon bonneljhon 4096 Sep 23 00:32 ..
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.1.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.2.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.3.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.4.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.5.log

---------- Rotation 7 -----------
total 28
drwxr-xr-x 2 bonneljhon bonneljhon 4096 Sep 23 00:32 .
drwxr-xr-x 7 bonneljhon bonneljhon 4096 Sep 23 00:32 ..
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.1.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.2.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.3.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.4.log
-rw-r--r-- 1 bonneljhon bonneljhon   44 Sep 23 00:32 app.5.log
```
### STEP 7 - backup_auto.sh (Member 3)
Begin with set -euo pipefail. Set `DEST=~/lab1a_bash/backup_$(date +%Y%m%d_%H%M%S)` and run `mkdir -p $DEST`, then `rsync -avz ~/lab1a_bash/logs/ $DEST >> backup.log 2>&1`. Register trap `'echo "FAILED $(date -Is)" >> backup.log; exit 1' ERR`, and echo the start and end timestamps. Run the script twice and confirm two separate timestamped directories. Finally restore one file from the first directory.

```bash
nano backup_auto.sh
```

```
#!/bin/bash

set -euo pipefail

trap 'echo "FAILED $(date -Is)" >> ~/lab1a_bash/backup.log; exit 1' ERR

DEST=~/lab1a_bash/backup_$(date +%Y%m%d_%H%M%S)

mkdir -p "$DEST"

echo "Backup started: $(date -Is)"

rsync -avz ~/lab1a_bash/logs/ "$DEST" >> ~/lab1a_bash/backup.log 2>&1

echo "Backup completed: $(date -Is)"
```

### OUTPUT:
```bash
bash backup_auto.sh
```
Run it twice
```bash
Backup started: 2026-09-23T00:45:09+08:00
Backup completed: 2026-09-23T00:45:09+08:00
```
```bash
ls backup_*
```
<img width="492" height="188" alt="image" src="https://github.com/user-attachments/assets/bf2519b4-b6d0-43e7-8ce6-44f4d2680d92" />

Finally restore one file from the first directory, since a backup that has never been restored is an assumption rather than a backup.

```bash
cp ~/lab1a_bash/backup_YYYYMMDD_HHMMSS/app.log ~/lab1a_bash/restore_test/
```
Check if the restoration is successful.
```bash
ls -la ~/lab1a_bash/restore_test
```

### STEP 8 - Cron Configuration
Edit crontab:
```bash
crontab -e
```
Add (use absolute paths — tilde is not reliably expanded by every cron):
```bash
5 * * * * /bin/bash /home/bonneljhon/lab1a_bash/script2_diskcheck.sh >> /home/bonneljhon/lab1a_bash/cron_disk.log 2>&1
```
For quick testing, temporarily use * * * * * then restore 5 * * * *.
```bash
* * * * * /bin/bash /home/bonneljhon/lab1a_bash/script2_diskcheck.sh >> /home/bonneljhon/lab1a_bash/cron_disk.log 2>&1
```
Too Verify:
```bash
crontab -l
```
You should see the `5 * * * * /bin/bash /home/bonneljhon/lab1a_bash/script2_diskcheck.sh >> /home/bonneljhon/lab1a_bash/cron_disk.log 2>&1`
or `* * * * * /bin/bash /home/bonneljhon/lab1a_bash/script2_diskcheck.sh >> /home/bonneljhon/lab1a_bash/cron_disk.log 2>&1`

```bash
cat ~/lab1a_bash/cron_disk.log
```
Displays the number of logs

```bash
journalctl -u cron --since "20 min ago"
```

Use journalctl, not grep CRON /var/log/syslog — the minimum Ubuntu Server install used in this course does not include rsyslog.







