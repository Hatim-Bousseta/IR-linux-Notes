Shell Profile Persistence (`.bashrc` / `.bash_profile`)

Attackers often modify shell initialization files (`.bashrc` and `.bash_profile`) to execute malicious commands every time a user logs in or opens a new terminal, ensuring highly effective **persistence**.

### Ⅰ. File Definitions & Execution

| File | Execution Trigger | Attacker Goal |
| :--- | :--- | :--- |
| **`.bashrc`** | Runs for **non-login shells** (e.g., opening a new terminal window). | Executes malicious commands **frequently** (aliases, functions, reverse shells). |
| **`.bash_profile`** | Runs **only once** for a **login shell** (e.g., initial SSH session or text console login). | Sets environment variables, can launch a **single persistent backdoor**. |

### Ⅱ. Incident Response & Triage

1.  **Locate All Files:** Because every user has their own files, all user home directories must be checked.

    ```bash
    find /home -name '.bashrc'
    find /home -name '.bash_profile'
    ```

    *(Note: Also check `/root/` and any service user home directories if applicable.)*

2.  **Examine Content:** Review the contents of each file for suspicious entries.

    ```bash
    cat ~/.bashrc
    ```

    *Look for:*

      * **Reverse Shell Commands:** (e.g., `nc -e`, Python/Perl one-liners, base64-encoded strings).
      * **Hidden Execution:** Commands run in the background (using `&`).
      * **Unexpected `source` commands:** Loading scripts from suspicious directories (e.g., `/tmp/`).

3.  **Check Metadata:** Use `stat` to verify the **modification time (Mtime)** of the file against the incident timeline.

### Ⅲ. Eradication

  * **Action:** Manually **remove the malicious line(s)** from the file(s) using a text editor (`nano`, `vi`).
  * **Do Not Simply Delete:** If the entire file is deleted, the legitimate user's configuration may break, impacting system stability or usability. Only remove the attacker's line(s).
