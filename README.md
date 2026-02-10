# GCP Cloud Workstation Breakout PoC

<img width="1470" height="798" alt="Screenshot 2026-02-09 at 4 40 55 PM" src="https://github.com/user-attachments/assets/a534af91-98c6-49a7-83f4-af6cb0dd69a9" />


**A Proof-of-Concept tool to demonstrate container breakout and privilege escalation in default Google Cloud Workstation configurations.**

> ⚠️ **DISCLAIMER**: Tested on us-central1-docker.pkg.dev/cloud-workstations-images/predefined/code-oss:latest. Should be applicable to almost every current predefined image. Removing root access and docker cli does not resolve the issue.

## 🚀 Overview

This Python script automates the "Docker Socket Breakout" technique to escalate privileges from a standard developer user (inside a Cloud Workstation container) to **Root** on the underlying Compute Engine Host VM.

It exploits a common misconfiguration where `/var/run/docker.sock` is mounted inside the workstation container to allow Docker-in-Docker (DinD) workflows.

### Features

* **Phase 1: Recon & Loot**
    * Dumps `/etc/shadow` and `/etc/passwd` from the Host OS.
    * Extracts the **GCP Service Account Identity Token** from the Host Metadata Service (IMDS).
* **Phase 2: Interactive Root Shell**
    * Launches a privileged container with the Host Root Filesystem mounted at `/mnt/host`.
    * Establishes a reverse shell connection back to the script.
    * Provides instructions to `chroot` and fully compromise the host.

## 📋 Prerequisites

* **Target Environment:** Google Cloud Workstation (or any container environment with an exposed Docker socket).
* **Vulnerability:** The workstation must have `/var/run/docker.sock` mounted and writeable by the user.
* **Dependencies:** Python 3 (Standard Library only - no `pip install` required).

## 🛠️ Usage

1. **Upload the script** to your Cloud Workstation (e.g., via `git clone` or copy-paste).

2. **Run the exploit:**

```bash
python3 exploit.py
```

3. **Follow the prompts:**
   * Phase 1 will print the sensitive host files and token to the screen.
   * Phase 2 will drop you into a shell.
   * Once connected, run the following command to enter the Host OS:

```bash
chroot /mnt/host /bin/bash
```

## 🛡️ Mitigation

To prevent this attack, Administrators should update the Workstation Configuration (Terraform/Console):

1. **Disable Privileged Mode:** Ensure the workstation container does not run with `--privileged`.
2. **Remove Docker Socket:** Do not mount `/var/run/docker.sock` into the workstation. Use Cloud Build or remote builders for container tasks.
3. **Identity Isolation:** Assign a dedicated Service Account with minimal scopes (e.g., `source.reader`) rather than the Default Compute Engine account.

## 📝 License

MIT License. See LICENSE for details.

---

**Remember:** Always practice responsible disclosure and only test systems you're authorized to assess.
