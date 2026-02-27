# Virtual Machine Install & Harden 🛡️

Set up and harden an **Ubuntu** virtual machine (VM) in **VirtualBox** (VBox) with secure defaults, including VM creation, installation, and basic post-install security.

---

### Verification Flow

Create VM ➡️ Install Ubuntu ➡️ Basic Hardening ➡️ Post-Install Checks

---

### Platform Specific Commands

Supported platforms for VirtualBox host:

• 💻 Linux  
• 🍏 macOS  
• 🪟 Windows (PowerShell)  

Each platform has its own dropdown section for commands where applicable.

---

## 🌐 Step 1: Create Virtual Machine

<details>
<summary><strong>Open Step 1</strong></summary>

### Purpose

Set up a new virtual machine with proper resources and attach the verified Ubuntu ISO.

### Recommended Settings

- CPU: X cores  
- RAM: X GB  
- Disk: X GB  
- Attach ISO as boot media  

### Host-specific Notes

<details><summary>💻 Linux / 🍏 macOS / 🪟 Windows</summary>

- Instructions or commands for creating VM on each platform

</details>

</details>

---

## 🖥️ Step 2: Install Ubuntu

<details>
<summary><strong>Open Step 2</strong></summary>

### Purpose

Install Ubuntu safely following verified ISO.

### Notes

- Partitioning recommendations  
- User setup  
- Enable encryption if desired  

### Host-specific Notes

<details><summary>💻 Linux / 🍏 macOS / 🪟 Windows</summary>

- Installation steps, screenshots or CLI commands if needed

</details>

</details>

---

## 🔐 Step 3: Basic Hardening

<details>
<summary><strong>Open Step 3</strong></summary>

### Purpose

Apply essential security configurations after install.

### Process

- Enable firewall (UFW)  
- Disable root login  
- Create limited sudo user  
- Automatic security updates  
- Optional: disable unused services or obfuscate ports

</details>

---

## 🛡️ Step 4: VirtualBox Hardening

<details>
<summary><strong>Open Step 4</strong></summary>

### Purpose

Secure the virtual machine environment.

### Suggested Actions

- Snapshots for rollback  
- Disable unnecessary shared folders and clipboard  
- Restrict USB / network access  
- Adjust VM settings for maximum isolation

</details>

---

## ✅ Step 5: Verification & Maintenance

<details>
<summary><strong>Open Step 5</strong></summary>

### Purpose

Ensure VM is secure and properly maintained.

### Suggested Actions

- Confirm system integrity  
- Update regularly  
- Optional monitoring or antivirus  
- Backup important snapshots  

</details>

---

## 🔖 References / Links

<details>
<summary><strong>Open Links</strong></summary>    

1. [VirtualBox Official Documentation](https://www.virtualbox.org/manual/)  
2. [Ubuntu Security Guide](https://ubuntu.com/security)  
3. [Linux Hardening Guides](https://linuxsecurity.com/)  

</details>

---
