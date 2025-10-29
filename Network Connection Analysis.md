
Attackers establish **network connections** (e.g., **reverse shells**) to bypass security controls, maintain command, and exfiltrate data. Incident response requires immediate identification and analysis of these connections and their associated processes.

### Ⅰ. Detecting Active Connections & Listening Ports

The **`netstat`** utility is the primary tool for real-time connection visibility, often available by default.

#### **A. Listing All Connections**

Identify established connections ($\rightarrow$ potential C2/reverse shell) and listening ports ($\rightarrow$ potential backdoor).

| Command | Purpose | Key Parameters |
| :--- | :--- | :--- |
| `netstat -ant` | Lists **all** connections, using **TCP**, with **numeric** IP/ports (faster, no DNS lookup). | `-a` (all), `-n` (numeric), `-t` (TCP) |
| `netstat -nlpt` | Lists all **listening** sockets (backdoors), with **numeric** IP/ports, showing **Process ID (PID)** and **Program name**. | `-n` (numeric), `-l` (listening), `-p` (process), `-t` (TCP) |

#### **B. Focus: Listening Ports (Backdoors)**

Attackers install backdoors that listen on unusual ports for future access.

* **Action:** Scrutinize all ports listening on $\mathbf{0.0.0.0}$ (all interfaces) or a specific non-default interface.
* **Suspicion:** Any port not tied to a known service (e.g., $\mathbf{80, 443, 22}$) or an unexpected process is highly suspicious.
* **Critical:** Use the **`-p`** flag to map suspicious connections back to a **PID/Process Name** for containment.

### Ⅱ. Firewall Rule Review (`IPTables`)

Attackers often modify the host firewall (**iptables**) to ensure their persistence/access rules are allowed or to redirect traffic.

* **Goal:** List and inspect current firewall rules for unexpected $\mathbf{ALLOW}$ or $\mathbf{REDIRECT}$ entries created by the attacker.

| Command | Purpose |
| :--- | :--- |
| `iptables -L` | Lists all rules across all chains (INPUT, OUTPUT, FORWARD) for the active firewall. |
