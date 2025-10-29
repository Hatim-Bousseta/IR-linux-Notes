Attackers often use SSH public key authentication to maintain reliable, credential-free access (**persistence**). They insert their own public key into the target user's `authorized_keys` file.

### Ⅰ. Key Persistence Mechanism

  * **Standard Method:** A user generates a public/private key pair. The **public key** is placed in the `~/.ssh/authorized_keys` file on the remote server.
  * **Attacker Use:** The attacker adds their own public key to the victim user's `authorized_keys` file, granting them immediate SSH access anytime, bypassing username/password authentication.

### Ⅱ. Incident Response Steps

#### A. Locate All `authorized_keys` Files

Since every user may have their own file, search the entire filesystem.

| Location | Command |
| :--- | :--- |
| **All User Directories** | `find / -name 'authorized_keys'` |

#### B. Analyze and Verify Content

1.  **Examine the content** of every located `authorized_keys` file.
    ```bash
    cat /home/SUSPECT_USER/.ssh/authorized_keys
    ```
2.  **Scrutinize** any key that is unfamiliar, unusually long, or tied to an unknown entity.
3.  **Check Metadata:** Use `stat` on the file to determine when it was last changed (Ctime/Mtime) and verify this against the known intrusion timeline.
    ```bash
    stat /home/SUSPECT_USER/.ssh/authorized_keys
    ```

#### C. Eradication

**Immediate Action:** Delete the attacker's key from the file to revoke their access.

1.  **Edit the file** and manually remove the suspicious line(s).
    ```bash
    nano /home/SUSPECT_USER/.ssh/authorized_keys
    ```
2.  Alternatively, if the entire file is malicious, **delete the file** (if access is not required by legitimate users).
    ```bash
    rm /home/SUSPECT_USER/.ssh/authorized_keys
    ```

