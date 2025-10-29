Understanding where system actions are recorded is fundamental to tracing attacker activity (TTPs).

| Log File / Artifact | Primary Location(s) | Key Information for IR |
| :--- | :--- | :--- |
| **`syslog`** | `/var/log/syslog`, `/var/log/messages` | Records **Cron job** executions, **Service** activations, and general system messages. |
| **`auth.log`** | `/var/log/auth.log`, `/var/log/secure` | **Logon/Authentication events** (SSH, SuDO), **User/Group creation** and changes. |
| **`access.log`** | `/var/log/apache2/access.log`, `/var/log/nginx/access.log` | **Web requests** made to the server (crucial for finding webshell access or exploitation attempts). |
| **`bash_history`** | `~/.bash_history` (in each user's home directory) | **Commands executed** by the user through the terminal (if the attacker didn't clear it). |
| **`lastlog`** | `/var/log/lastlog` | Record of each user's **last login** session. |
