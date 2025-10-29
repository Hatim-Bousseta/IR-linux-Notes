

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


