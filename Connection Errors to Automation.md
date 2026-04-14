# 🚀 Jenkins to Ubuntu VM: The Automation Journey

This document summarizes the technical evolution of setting up a Jenkins CI/CD pipeline to automate file transfers from a local system to a remote Ubuntu VirtualBox VM.

---

## 🏗️ Phase 1: Establishing Connectivity
**The Goal:** Make the Windows host and Ubuntu VM talk to each other.

* **Initial Problem:** `ssh: connect to host 192.168.1.6: No route to host`.
* **The Diagnosis:** The IP address had changed, or the VirtualBox network adapter was misconfigured.
* **The Solution:**
    * Verified IPs using `ipconfig` (Windows) and `ip a` (Ubuntu).
    * Ensured VirtualBox was using a **Bridged Adapter**.
    * Confirmed the **OpenSSH Server** service was running on Windows.

---

## 🔑 Phase 2: SSH Key Authentication
**The Goal:** Log in securely without a password prompt.

* **The Problem:** `ssh-copy-id` failed because Windows doesn't interpret Linux `exec` commands correctly.
* **The Solution:** * Generated an **ED25519** key pair on Windows using `ssh-keygen`.
    * Manually appended the public key (`id_ed25519.pub`) to the Ubuntu VM's `~/.ssh/authorized_keys`.
    * Applied strict Linux permissions: `chmod 700 ~/.ssh` and `chmod 600 ~/.ssh/authorized_keys`.

---

## ⚙️ Phase 3: Jenkins Environment Alignment
**The Goal:** Use Jenkins to automate the transfer.

* **The Problem (The "Container" Trap):** Jenkins was running in a Linux Docker container. Commands like `Execute Windows batch command` failed because the environment had no `cmd.exe`.
* **The Solution:** * Switched to **Execute Shell**.
    * Used `echo "content" > file.txt` to create the file directly in the Jenkins Linux workspace instead of trying to reach a Windows `D:\` drive.

---

## 🛠️ Phase 4: Solving "Auth Fail" in Jenkins
**The Goal:** Authenticate the "Publish Over SSH" plugin.

* **The Problem:** Even with the correct password, Jenkins returned `Auth fail for methods 'publickey,password'`.
* **The Solution:** * Configured the **Global System Settings** in Jenkins.
    * Moved the **Private Key** (the contents of `id_ed25519`) into the **Key** field.
    * Cleared the **Passphrase** field (since the key was created without one).
    * Restarted the SSH service on the VM: `sudo systemctl restart ssh`.

---

## ✅ Phase 5: Final Transfer & Verification
**The Goal:** Confirm the file arrived and manage it.

* **The Success:** The build log finally showed: `SSH: Transferred 1 file(s)`.
* **The Final Problem:** The file seemed "missing" after transfer.
* **The Solution:** * Used `find ~ -name "Virutal_VM.txt"` to locate the file.
    * Used `mv` to move the file to the desired directory.
    * Corrected the **Remote Directory** path in Jenkins job settings to `/home/kamal173`.

---

## 📝 Troubleshooting Summary Table

| Error Message | Meaning | Fix |
| :--- | :--- | :--- |
| `No route to host` | Network path broken | Check IP and Bridged Adapter. |
| `Permission denied` | Wrong password or key | Use Microsoft Password (not PIN) or fix `authorized_keys`. |
| `Cannot run program "cmd"` | OS Mismatch | Use **Execute Shell** for Linux/Docker Jenkins. |
| `Auth fail` | Credential Mismatch | Paste **Private Key** into Jenkins, not Public Key. |
| `Status [2]` | Path not found | Use absolute paths like `/home/user/file`. |

---
**Build Result:** SUCCESS 🟢
