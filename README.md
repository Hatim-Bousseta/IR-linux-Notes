# IR-linux-Notes
Linux Incident Response (IR) File Analysis Quick Reference  A concise cheat sheet and guide for security analysts and responders. Focuses on actionable steps for investigating attacker artifacts, webshells, and persistence mechanisms by leveraging native Linux tools.

💾 Filesystem Incident Response

Attacker Focus Areas

    Filesystem Artifacts: Attacker writes → Malware/Tools, Stolen Data, Persistence files, Config changes.

    Goal: Restore system OR Detect/Clean all attacker changes (if no snapshot).

Linux Directory Review (IR Context)

    /etc: Config files. Check for changes/new entries.

    /var: Log files (Critical!).

    /bin, /sbin: Executables. Check for trojanized binaries.

    /home, /root: User/Root home → Custom tools, history, download folders.

Finding Suspicious Files

Suspicious Directories (Priority Check)

    /tmp: Highest priority.

        Why: Easy Read/Write for all users.

        Urgency: Files are volatile (deleted automatically) → Move fast.

    Web-Facing Dirs (e.g., /var/www):

        Target: Webshells, backdoor scripts.
        Action: First, identify services open to the internet (netstat). Second, examine the associated application directories.


        Here is the second set of notes, maintaining the requested raw, direct, and actionable "note way" format:



### **Suspicious Extensions**

  * Goal: Find **malware**, **webshells**, and **runnable files** written by attacker.
  * Focus on standard malicious extensions: `.sh`, `.php`, `.php7`, `.elf` (executables).
  * **Command:** Search entire filesystem for these:
    ```bash
    find / -type f \( -iname \*.php -o -iname \*.php7 -o -iname \*.sh -o -iname \*.elf \)
    ```

### **Modification Time (Mtime)**

  * Use when **attack time window is known**.
  * Search for files **modified** during the incident timeframe.
  * **Example Command:** Files modified on Oct 25, 2021, in `/tmp`:
    ```bash
    find /tmp -newermt "2021-10-25 00:00:00" ! -newermt "2021-10-25 23:59:00"
    ```

### **Owner**

  * If **compromised user(s)** are known, analyze files owned by them.
  * **Note:** Cannot search by *modifier*, only by *owner*.
  * **Example Command:** Files owned by `www-data` in `/tmp`:
    ```bash
    find /tmp -user www-data
    ```

### **Change Date (Ctime)**

  * Change Date updates when **file metadata changes** (ownership, permissions/authorizations, or content).
  * Attackers often change file **permissions/ownership** for persistence.
  * **Command:** Find files whose metadata was changed $X$ days ago:
    ```bash
    find / -ctime +X
    ```

-----

## 🔬 **File Analysis (Initial Triage)**

  * Once suspicious files are ID'd, **gather initial info** before static/dynamic analysis.
  * **Tool:** `stat`
  * **Purpose:** Get **detailed file metadata** (timestamps, owner, permissions, etc.).
  * **Command:**
    ```bash
    stat FILENAME
    ```


Here is the final part of your notes, condensed and structured for a GitHub reference guide format.

-----

## ⏰ Linux IR: Crontab Persistence Analysis

**Cron** is the scheduling tool. **Crontab** is the configuration file listing the **Cron Jobs** (scheduled tasks). Attackers abuse this for **persistence**.

### Ⅰ. Crontab Structure (Review)

A crontab entry has $\mathbf{6}$ fields:

1.  **Minute** (0-59)
2.  **Hour** (0-23)
3.  **Day of Month** (1-31)
4.  **Month** (1-12)
5.  **Day of Week** (0-7, Sun=0 or 7)
6.  **Command** to execute

| Example | Execution Schedule |
| :--- | :--- |
| `0 * * * * /path/script.sh` | At the beginning of **every hour**. |
| `0 0 1 * * /path/script.sh` | At midnight on the **1st of every month**. |
| `*/15 * * * * /path/script.sh` | **Every 15 minutes**. |

-----

### Ⅱ. Incident Response: Listing Active Jobs

Check all potential locations for malicious or altered cron jobs.

| Location | Purpose | Command |
| :--- | :--- | :--- |
| **System Crontab** | Core system-wide jobs. Only authorized users can edit. | `cat /etc/crontab` |
| **System Drop-In Dir** | Directory for additional system job files. | `cat /etc/cron.d/*` |
| **User Crontabs** | Individual user scheduled jobs. High attacker target. | `cat /var/spool/cron/crontabs/*`<br>`cat /var/spool/cron/*` |

-----

### Ⅲ. Historical Data (Detecting Deleted Jobs)

Active jobs are insufficient; check logs for **deleted** or **temporarily run** jobs.

| Log Source | Purpose | Command(s) |
| :--- | :--- | :--- |
| **Syslog** | Quickly filter for any CRON executions. | `cat /var/log/syslog \| grep CRON` |
| **Systemd Journal** | Use the `cron` service unit for detailed, time-bound logs. | `journalctl -u cron` |
| **Time-Bounded Search** | Focus on the known attack time window. | `journalctl -u cron --since "YYYY-MM-DD 00:00" --until "YYYY-MM-DD 23:59"` |

-----

### Ⅳ. Analysis and Eradication

#### **Analysis Focus**

  * **Timing:** Understand *when* the job executes (hourly, monthly, etc.).
  * **Command/Script:** Examine the executed command string or script content. Look for:
      * **Suspicious locations** (e.g., executing a script from `/tmp`).
      * **Keywords** (`rev`, `shell`, `backdoor`, base64 strings, etc.).
      * **Reverse Shells** as the command.
  * **Verification:** Cross-reference malicious cron jobs with **historical logs** (Section III) to confirm if and when they ran.

#### **Eradication**

Delete attacker-added cron jobs and revert any changes to existing jobs.

| Action | Command |
| :--- | :--- |
| **Edit Current User's Crontab** | `crontab -e` |
| **Edit Another User's Crontab** | `crontab -u USERNAME -e` |



        Action: First, identify services open to the internet (netstat). Second, examine the associated application directories.
