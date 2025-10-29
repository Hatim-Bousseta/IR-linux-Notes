# IR-linux-Notes
Linux Incident Response (IR) File Analysis Quick Reference  A concise cheat sheet and guide for security analysts and responders. Focuses on actionable steps for investigating attacker artifacts, webshells, and persistence mechanisms by leveraging native Linux tools.

I apologize. My goal was to create clear, standard notes using bullet points and tables. It seems my structure might have been too "formatted" or "polished."

I will now focus on a style that is even more raw, direct, and focused solely on the critical action or data point, which is often what "notes" look like when they are written quickly for reference.

Here is the revised, bare-bones version of your first section:

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
        

        Action: First, identify services open to the internet (netstat). Second, examine the associated application directories.
