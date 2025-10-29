Attackers frequently use system services (especially `systemd` units) for persistence because they run reliably in the background, can auto-restart, and activate on boot.

### Ⅰ. Service Status and Purpose

Services are primarily managed by **`systemctl`** and have different states:

| Status | Meaning | IR Note |
| :--- | :--- | :--- |
| **Active (running)** | Service daemon is operating. | **Check PID and associated binary location.** |
| **Active (exited)** | Ran successfully, but no continuous daemon to monitor. | *Suspicious:* Could be a malicious **single-run** command. |
| **Inactive** | Service is stopped. | |
| **Enabled** | Service is configured to start automatically at boot. | **HIGH RISK:** Primary persistence method. |
| **Disabled** | Service will *not* start at boot. | |

### Ⅱ. Service Configuration Structure (`.service` Unit File)

Configuration files have a `.service` extension (e.g., `/lib/systemd/system/malware.service`).

| Section | Key Directive | IR Focus |
| :--- | :--- | :--- |
| **`[Unit]`** | `Description` | Check for vague or misleading descriptions. |
| **`[Service]`** | **`ExecStart`** | **CRITICAL:** The exact command run when the service starts. **Attacker's code is here.** |
| | `ExecStartPre` | Commands run *before* the main command (e.g., creating files). |
| | `Restart` | If set to `always` or `on-failure`, ensures persistence. |
| **`[Install]`**| `WantedBy` | Defines *when* it runs (e.g., `multi-user.target` = normal system boot). |

### Ⅲ. Listing Active Services

Retrieve a current list to identify new or suspicious entries.

| Tool | Command |
| :--- | :--- |
| **systemctl** (Recommended) | `systemctl list-units --type=service` |
| **service** (Legacy/Alternative) | `service --status-all` |

### Ⅳ. Service Analysis and Triage

Once a suspicious service is identified, collect all related data.

1.  **Get Status & Config Location:**

    ```bash
    systemctl status SERVICE_NAME
    ```

    *Output provides:* Active status, PID, and the path to the **Unit File** (e.g., `/lib/systemd/system/`).

2.  **Inspect Configuration File:**

    ```bash
    cat /path/to/unit/file.service
    ```

    *Focus on:* The `ExecStart` command and the `Restart` setting.

3.  **Check Configuration File Timestamps:**

    ```bash
    stat /path/to/unit/file.service
    ```

    *Goal:* Verify the **creation/modification date (Ctime/Mtime)** against the known attack window.

4.  **Review Service-Specific Logs:**
    Check what the service has done in the past, even if it is currently inactive.

    ```bash
    journalctl -u SERVICE_NAME.service
    ```

    *Time Window Filter:*

    ```bash
    sudo journalctl -u SERVICE_NAME --since "YYYY-MM-DD 00:00" --until "YYYY-MM-DD 23:59"
    ```

5.  **Historical Log Search (Generic):**
    Search the entire journal for suspicious activity during the attack window (if service name is unknown).

    ```bash
    sudo journalctl --since "2021-05-07 00:00" --until "2021-05-07 23:59"
    ```

### Ⅴ. Eradication Steps

To fully remove a malicious service and its persistence:

1.  **Stop the Service:**

    ```bash
    sudo systemctl stop SERVICE_NAME
    ```

2.  **Disable Auto-Start (Persistence Removal):**

    ```bash
    sudo systemctl disable SERVICE_NAME
    ```

3.  **Remove Configuration/Binary Files:**

    ```bash
    rm /path/to/unit/file.service
    rm /path/to/malicious/binary
    ```

4.  **Reload the Daemon (Apply Changes):**
    *Crucial step:* Forces `systemd` to recognize the file removal/changes.

    ```bash
    sudo systemctl daemon-reload
    ```
