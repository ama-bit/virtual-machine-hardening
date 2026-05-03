# Virtual Machine Install & Harden 🛡️

Set up and harden an **Ubuntu** virtual machine (VM) in **VirtualBox** with secure
defaults, including VM creation, installation, and post-install security hardening.

> **Tested on:** Ubuntu 24.04 LTS · VirtualBox 7.x · Host platforms: Linux, macOS, Windows
>
> **Status key used throughout this guide:**
> - ✅ *Tested* — verified hands-on during authorship
> - 📖 *Researched* — sourced from official documentation; not personally verified on every platform

---

## Scope

**This Guide**
```
Create VM ➡️ Install Ubuntu ➡️ Harden OS ➡️ Harden VirtualBox ➡️ Verify & Maintain
```

**Previous Guide**
```
Download ➡️ Verify SHA256 (Integrity) ➡️ Verify GPG Signature (Authenticity)
```

---

## Tools

| Tool | Purpose |
|------|---------|
| Oracle VM VirtualBox | Creates and runs VMs |
| Ubuntu ISO | Verified installation media |
| VBoxManage | VirtualBox CLI for advanced configuration |
| Terminal / PowerShell | Runs hardening and verification commands |

💡 *Supported host platforms:*
```
💻 Linux  
🍏 macOS  
🪟 Windows (PowerShell)
```

---

## Threat Model

This guide addresses the following threats and the controls that mitigate them.

Understanding *why* a control exists matters as much as *how* to apply it — that's
what separates a hardened system from one that just ran a checklist.

| # | Threat | Controls That Address It |
|---|--------|--------------------------|
| 1 | Running unverified or compromised installation media | ISO verification (prior guide); Step 2 warning to not skip verification |
| 2 | Weak default OS configuration post-install | Step 3: UFW, root lock, auto-updates, AppArmor |
| 3 | Guest-to-host escape via shared resources | Step 4: Disable clipboard, drag-and-drop, shared folders, USB |
| 4 | Unauthorized access through open ports or services | Step 3b (UFW), Step 3f (disable unused services), Step 3i (SSH hardening) |
| 5 | Persistence of a compromised state without rollback | Step 4e: Snapshot strategy |

**Why each guest-isolation control matters (Threat 3 in depth):**

- **Shared clipboard** creates a bidirectional data channel between guest and host. If the guest is compromised, this becomes an exfiltration or injection path — malware in the guest can read from or write to the host's clipboard without any additional privilege escalation.
- **Drag-and-drop** operates similarly — it's a file transfer channel that bypasses network controls entirely. An attacker with guest access could use it to stage files on the host.
- **Shared folders** mount a host filesystem path inside the guest. Any process in the guest — including malware — that has filesystem access can read, modify, or delete files on the host through that mount.
- **USB passthrough** grants the guest direct access to physical USB devices. A compromised guest could interact with USB storage, firmware, or HID devices on the host.

This guide does **not** address:
- Network-level threats external to the VM host
- Physical access attacks
- Advanced persistent threats (APT)
- Full disk encryption of the host machine

> 📓 Hardening is a tradeoff between security and usability.
> Apply controls that match your actual threat model.
> Not every step is required for every use case.

---

## 🌐 Step 1: Create Virtual Machine

<details>
<summary><strong>Open Step 1</strong></summary>

### Purpose
Create a new virtual machine in VirtualBox and attach the verified Ubuntu ISO as boot media.

---

### Recommended Specifications

| Setting | Minimum | Recommended |
|---------|---------|-------------|
| CPU | 2 cores | 4 cores |
| RAM | 4 GB | 8 GB |
| Disk | 25 GB | 50 GB |
| Display | 16 MB VRAM | 32 MB VRAM |
| Network | NAT | Host-Only (see Step 4c) |

> 📓 These specs target Ubuntu 24.04 LTS desktop. Adjust based on your host machine's
> available resources. Server installs can run on significantly less RAM.

---

### GUI Steps (All Platforms)

<details>
<summary><strong>VirtualBox GUI</strong></summary>

✅ *Tested*

```
1. Open VirtualBox → Click "New"
2. Name your VM (e.g. "Ubuntu-24.04")
3. Set Type: Linux | Version: Ubuntu (64-bit)
4. Set RAM and CPU to recommended values above
5. Create a new virtual hard disk
   - Format: VDI (VirtualBox Disk Image)
   - Storage: Dynamically allocated
   - Size: 25 GB minimum, 50 GB recommended
6. Click Finish
```

<!-- 📸 SCREENSHOT NEEDED: VirtualBox "Create Virtual Machine" dialog
     Show: Name field, Type set to Linux, Version set to Ubuntu (64-bit)
     Show: RAM and CPU fields filled in -->

**Attach the verified Ubuntu ISO:**

```
1. Select your VM → Click "Settings"
2. Navigate to Storage
3. Under Controller: IDE → Click the empty disk icon
4. Click the disk icon on the right → "Choose a disk file"
5. Select your verified ubuntu-XX.XX-desktop-amd64.iso
6. Click OK
```

<!-- 📸 SCREENSHOT NEEDED: VirtualBox Storage settings dialog
     Show: Empty optical drive selected
     Show: File picker with Ubuntu ISO selected -->

> ❗ Only attach the ISO verified in vm-verify.md.
> Do not use ISOs from mirrors or unverified sources.

</details>

---

### CLI Steps (VBoxManage)

> Replace `Ubuntu-24.04` with your chosen VM name.
> Replace ISO path with the full path to your verified Ubuntu ISO.

<details>
<summary>💻 Linux</summary>

✅ *Tested*

```bash
# Create the VM directory first if it does not exist
mkdir -p ~/VirtualBox\ VMs/Ubuntu-24.04/

# Create the VM
VBoxManage createvm --name "Ubuntu-24.04" --ostype Ubuntu_64 --register

# Set RAM and CPU
VBoxManage modifyvm "Ubuntu-24.04" --memory 4096 --cpus 2

# Set display memory
VBoxManage modifyvm "Ubuntu-24.04" --vram 32

# Create virtual hard disk (size in MB — 25600 = 25 GB)
VBoxManage createhd --filename ~/VirtualBox\ VMs/Ubuntu-24.04/Ubuntu-24.04.vdi --size 25600

# Add SATA storage controller
VBoxManage storagectl "Ubuntu-24.04" --name "SATA Controller" --add sata --controller IntelAhci

# Attach the virtual hard disk
VBoxManage storageattach "Ubuntu-24.04" --storagectl "SATA Controller" \
  --port 0 --device 0 --type hdd \
  --medium ~/VirtualBox\ VMs/Ubuntu-24.04/Ubuntu-24.04.vdi

# Attach the verified Ubuntu ISO
VBoxManage storageattach "Ubuntu-24.04" --storagectl "SATA Controller" \
  --port 1 --device 0 --type dvddrive \
  --medium /path/to/ubuntu-XX.XX-desktop-amd64.iso

# Set boot order — DVD first for initial installation
VBoxManage modifyvm "Ubuntu-24.04" --boot1 dvd --boot2 disk --boot3 none --boot4 none
```

</details>

<details>
<summary>🍏 macOS</summary>

📖 *Researched — commands mirror Linux; paths differ*

```bash
# Create the VM directory first if it does not exist
mkdir -p ~/VirtualBox\ VMs/Ubuntu-24.04/

# Create the VM
VBoxManage createvm --name "Ubuntu-24.04" --ostype Ubuntu_64 --register

# Set RAM and CPU
VBoxManage modifyvm "Ubuntu-24.04" --memory 4096 --cpus 2

# Set display memory
VBoxManage modifyvm "Ubuntu-24.04" --vram 32

# Create virtual hard disk (size in MB — 25600 = 25 GB)
VBoxManage createhd --filename ~/VirtualBox\ VMs/Ubuntu-24.04/Ubuntu-24.04.vdi --size 25600

# Add SATA storage controller
VBoxManage storagectl "Ubuntu-24.04" --name "SATA Controller" --add sata --controller IntelAhci

# Attach the virtual hard disk
VBoxManage storageattach "Ubuntu-24.04" --storagectl "SATA Controller" \
  --port 0 --device 0 --type hdd \
  --medium ~/VirtualBox\ VMs/Ubuntu-24.04/Ubuntu-24.04.vdi

# Attach the verified Ubuntu ISO
VBoxManage storageattach "Ubuntu-24.04" --storagectl "SATA Controller" \
  --port 1 --device 0 --type dvddrive \
  --medium /path/to/ubuntu-XX.XX-desktop-amd64.iso

# Set boot order — DVD first for initial installation
VBoxManage modifyvm "Ubuntu-24.04" --boot1 dvd --boot2 disk --boot3 none --boot4 none
```

</details>

<details>
<summary>🪟 Windows (PowerShell)</summary>

📖 *Researched — see Known Limitations note below*

```powershell
# Create the VM
VBoxManage createvm --name "Ubuntu-24.04" --ostype Ubuntu_64 --register

# Set RAM and CPU
VBoxManage modifyvm "Ubuntu-24.04" --memory 4096 --cpus 2

# Set display memory
VBoxManage modifyvm "Ubuntu-24.04" --vram 32

# Create virtual hard disk (size in MB — 25600 = 25 GB)
VBoxManage createhd `
  --filename "$env:USERPROFILE\VirtualBox VMs\Ubuntu-24.04\Ubuntu-24.04.vdi" `
  --size 25600

# Add SATA storage controller
VBoxManage storagectl "Ubuntu-24.04" --name "SATA Controller" --add sata --controller IntelAhci

# Attach the virtual hard disk
VBoxManage storageattach "Ubuntu-24.04" --storagectl "SATA Controller" `
  --port 0 --device 0 --type hdd `
  --medium "$env:USERPROFILE\VirtualBox VMs\Ubuntu-24.04\Ubuntu-24.04.vdi"

# Attach the verified Ubuntu ISO
# Replace with the full path to your verified ISO
VBoxManage storageattach "Ubuntu-24.04" --storagectl "SATA Controller" `
  --port 1 --device 0 --type dvddrive `
  --medium "C:\path\to\ubuntu-XX.XX-desktop-amd64.iso"

# Set boot order — DVD first for initial installation
VBoxManage modifyvm "Ubuntu-24.04" --boot1 dvd --boot2 disk --boot3 none --boot4 none
```

> 📓 **Windows CLI — Known Limitations**
> VBoxManage is available on Windows but behavior may differ from Linux and macOS.
> If you encounter errors running these commands:
>
> - Confirm VBoxManage is in your system PATH:
>   ```powershell
>   Get-Command VBoxManage
>   ```
> - If not found, add the VirtualBox installation directory to PATH manually:
>   ```powershell
>   # Default VirtualBox install location
>   $env:PATH += ";C:\Program Files\Oracle\VirtualBox"
>   ```
> - Run PowerShell as Administrator if permission errors occur
> - Confirm your VirtualBox version supports the flags used — some flags
>   differ between VirtualBox versions
> - If a command fails silently, check the VirtualBox log:
>   ```powershell
>   # Logs are stored per-VM
>   Get-Content "$env:USERPROFILE\VirtualBox VMs\Ubuntu-24.04\Logs\VBox.log"
>   ```
> - For persistent PATH changes, use System Properties →
>   Environment Variables → add VirtualBox directory to System PATH

</details>

> ✅ VM created and ISO attached. Proceed to Step 2 to install Ubuntu.

</details>

---

## 🖥️ Step 2: Install Ubuntu

<details>
<summary><strong>Open Step 2</strong></summary>

### Purpose
Install Ubuntu inside the VM using the verified ISO attached in Step 1.

---

### Installation Steps (All Platforms)

<details>
<summary><strong>VirtualBox GUI</strong></summary>

✅ *Tested*

```
1. Select your VM in VirtualBox → Click "Start"
2. Ubuntu installer will boot from the attached ISO
3. Select "Try or Install Ubuntu"
```

<!-- 📸 SCREENSHOT NEEDED: Ubuntu installer welcome screen
     Show: "Try or Install Ubuntu" option highlighted -->

```
4. Choose your language and keyboard layout
5. Select "Install Ubuntu"
6. Installation type:
   - Select "Erase disk and install Ubuntu"
   - Click "Advanced features" to access:
     - LVM (flexible disk management — allows resizing volumes later without reinstalling)
     - LVM with encryption (LUKS) — encrypts the virtual disk at rest; recommended if
       the VM will hold sensitive data or the host machine is shared or portable
7. Set your timezone
8. Create your user account:
   - Use a strong password
   - Enable "Require password to log in"
   - Do not enable auto-login
       ↳ Auto-login bypasses authentication entirely. If the host is left unattended
         with the VM running, anyone with physical access can open the VM without a password.
9. Complete installation and restart when prompted
10. Remove ISO when prompted or via Settings → Storage
```

<!-- 📸 SCREENSHOT NEEDED: Installation type screen
     Show: "Erase disk and install Ubuntu" selected
     Show: "Advanced features" button location -->

<!-- 📸 SCREENSHOT NEEDED: Advanced features dialog
     Show: LVM option
     Show: LVM with encryption (LUKS) option -->

<!-- 📸 SCREENSHOT NEEDED: User account creation screen
     Show: Password field
     Show: "Require password to log in" checkbox enabled
     Show: Auto-login option disabled -->

<!-- 📸 SCREENSHOT NEEDED: VirtualBox Storage settings after install
     Show: Optical drive with ISO — highlight where to remove it -->

> ❗ Do not install from an unverified ISO.
> If you skipped vm-verify.md, complete verification before proceeding.
>
> Reason: An unverified ISO could be corrupted or tampered with. The verification
> step in the prior guide confirms both integrity (SHA256) and authenticity (GPG).
> Skipping it means you cannot trust the foundation everything else is built on.

</details>

---

### Post-Install Boot

After installation and restart:

```
1. VirtualBox will boot from the virtual hard disk
2. Log in with the credentials created during installation
3. Confirm Ubuntu loads correctly before proceeding to hardening
```

> 📓 If the VM boots back into the installer, the ISO was not removed.
> Go to Settings → Storage → remove the ISO from the optical drive.

</details>

---

## 🔐 Step 3: Harden OS

<details>
<summary><strong>Open Step 3</strong></summary>

### Purpose
Apply essential security configurations to Ubuntu after installation.

Ubuntu ships with reasonable defaults, but "reasonable defaults" are designed for broad
compatibility — not for security. Each step below closes a specific gap.

---

### 3a: Update System

✅ *Tested*

```bash
# Update package lists and upgrade all installed packages
sudo apt update && sudo apt upgrade -y

# Remove unused packages
sudo apt autoremove -y
```

> 📓 Many exploits target known vulnerabilities in unpatched software. Running updates
> immediately after install ensures you are not starting from an already-outdated baseline.

---

### 3b: Enable Firewall (UFW)

✅ *Tested*

```bash
# Install UFW if not already present
sudo apt install ufw -y

# Set default policies — deny all incoming, allow all outgoing
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH only if remote access is needed
# sudo ufw allow ssh

# Enable firewall
sudo ufw enable

# Confirm firewall status
sudo ufw status verbose
```

> 📓 UFW is a frontend for iptables. The default policy here is deny-by-default on inbound,
> which means no port is reachable unless you explicitly open it. This is the correct posture
> for a VM that isn't intentionally running services.
>
> Every open port is a potential entry point. Only open what you explicitly need —
> not what might be convenient later.

---

### 3c: Disable Root Login

✅ *Tested*

```bash
# Lock the root account — sudo user created during install is sufficient
sudo passwd -l root

# Confirm root is locked
sudo passwd -S root
# Expected output: root L (locked)
```

> 📓 Ubuntu creates a sudo-capable user during install and locks root by default,
> but it's worth explicitly confirming and enforcing this.
>
> Why it matters: a locked root account means that even if an attacker gains a
> foothold in the system, they cannot escalate by simply switching to root —
> they still need to know your sudo user's password and abuse a privilege
> escalation path. It narrows the attack surface on that escalation step.

---

### 3d: Automatic Security Updates

✅ *Tested*

```bash
# Install unattended-upgrades
sudo apt install unattended-upgrades -y

# Enable automatic security updates
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

> 📓 Security patches are only useful if they're applied. Unattended-upgrades handles
> security updates specifically (not major version upgrades) — it applies patches
> without requiring manual intervention. For a VM you might not log into frequently,
> this is especially important.

---

### 3e: Verify Sudo User

✅ *Tested*

```bash
# Confirm your user has sudo access
sudo -l

# Confirm root is not used for regular tasks
whoami
# Expected output: your username, not root
```

---

### 3f: Disable Unused Services — Optional

📖 *Researched — verify behavior for your specific setup before disabling*

Reducing running services reduces attack surface. Every running service is code that
could contain vulnerabilities, and code that isn't running can't be exploited.

**When to apply this step:** If the VM is long-lived, networked, or holds sensitive data.
Skip this step if the VM is short-lived or isolated and you don't want the maintenance overhead.

**Commonly safe to disable in a VM context:**

| Service | What it does | Disable if... |
|---------|-------------|---------------|
| bluetooth | Bluetooth support | No Bluetooth hardware or use — VMs typically have no BT hardware |
| cups | Printing service | No printing needed |
| avahi-daemon | Network discovery (mDNS) | No local network discovery needed; also reduces network fingerprint |
| ModemManager | Mobile broadband management | No modem attached |
| snapd | Snap package manager | Not using Snap packages — **verify no dependencies first on Ubuntu 24.04** |

```bash
# List all running services
systemctl list-units --type=service --state=running

# Check what a service does before disabling
systemctl status SERVICE

# Disable a service (replace SERVICE with service name from table above)
sudo systemctl disable --now SERVICE

# Example: disable Bluetooth
sudo systemctl disable --now bluetooth

# Verify service is stopped and disabled
systemctl is-enabled SERVICE
systemctl is-active SERVICE
```

> ⚠️ Do not disable services you do not recognize without researching them first.
> Disabling critical services can break system functionality or prevent boot.
> snapd in particular may have dependencies on Ubuntu 24.04 — verify before disabling.

---

### 3g: Verify AppArmor is Active

✅ *Tested*

AppArmor ships with Ubuntu and enforces mandatory access controls (MAC) on applications.
It limits what a given process can do — even if that process is exploited — by defining
a profile of allowed behaviors (files it can read, syscalls it can make, etc.).
This provides a layer of containment that operates independently of standard Unix permissions.

AppArmor should be active by default; this step confirms it.

```bash
# Check AppArmor status
# aa-status is the canonical command on modern Ubuntu
sudo aa-status

# Expected output: apparmor module is loaded
# Profiles should show enforced, not complain mode

# If inactive, enable and start it
sudo systemctl enable apparmor
sudo systemctl start apparmor
```

> 📓 "Complain mode" logs violations but does not block them — it's useful for
> developing new profiles but provides no actual protection. Ensure profiles
> show `enforce` mode, not `complain`.

---

### 3h: Kernel Hardening (sysctl) — Optional

📖 *Researched — parameters sourced from official kernel documentation and established hardening guides*

Apply kernel-level security parameters to reduce attack surface at the OS level.
These settings affect how the kernel handles networking, memory, and process information.

**When to apply this step:** If the VM is networked, exposed to untrusted input,
or is part of a higher-security environment. Safe to skip for an isolated dev VM
with no network exposure.

> ⚠️ Understand each parameter before applying.
> Some settings may affect VM functionality depending on your use case.

```bash
# Check the current value of a specific parameter before changing it
sysctl net.ipv4.ip_forward

# Apply hardening parameters
sudo tee /etc/sysctl.d/99-hardening.conf <<EOF
# Disable IP forwarding — not needed for a standard desktop VM
# Without this, a compromised VM could be used to route traffic between networks
net.ipv4.ip_forward = 0

# Disable ICMP redirects — prevents an attacker from manipulating routing tables
# via forged ICMP redirect messages
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Ignore ICMP broadcast requests — reduces amplification attack participation
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Enable SYN flood protection via SYN cookies
# Mitigates DoS attacks that exhaust the TCP connection table
net.ipv4.tcp_syncookies = 1

# Restrict kernel pointer exposure to root only
# Prevents unprivileged processes from reading kernel memory addresses,
# which are useful to attackers writing kernel exploits
kernel.kptr_restrict = 2

# Restrict dmesg access to root only
# dmesg can leak kernel addresses and hardware info useful for exploit development
kernel.dmesg_restrict = 1

# Disable core dumps for setuid programs
# Prevents sensitive memory from being written to disk if a setuid program crashes
fs.suid_dumpable = 0
EOF

# Apply specific file
sudo sysctl -p /etc/sysctl.d/99-hardening.conf

# Or apply all sysctl config files
sudo sysctl --system
```

> 📓 Changes persist across reboots via `/etc/sysctl.d/`.
> To revert, delete the file and reboot or re-apply original values.

---

### 3i: SSH Hardening — Optional

📖 *Researched — apply only if SSH access to the VM is actually needed*

**When to apply this step:** Only if you need remote access to the VM over SSH.
If SSH is not required, skip this step entirely and confirm port 22 is closed in UFW.

Opening SSH when you don't need it is unnecessary attack surface. If you do need it,
the defaults are insecure and must be hardened.

```bash
# Confirm SSH is installed
ssh -V

# Install if needed
sudo apt install openssh-server -y

# Back up default config before editing
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Edit SSH configuration
sudo nano /etc/ssh/sshd_config
```

**Key settings to configure in `/etc/ssh/sshd_config`:**

```
# Disable root login via SSH
# Even with a strong password, root login over SSH is a direct path to full
# system compromise if credentials are leaked or brute-forced
PermitRootLogin no

# Disable password authentication — use key-based auth only
# Password auth is vulnerable to brute force; key-based auth is not
# (assuming the private key is protected)
PasswordAuthentication no

# Limit SSH to specific user (replace USERNAME)
# Reduces attack surface to one account instead of any valid system user
AllowUsers USERNAME

# Set idle timeout — disconnect after 5 minutes of inactivity
# Prevents unattended authenticated sessions from remaining open
ClientAliveInterval 300
ClientAliveCountMax 0
```

```bash
# Restart SSH to apply changes
sudo systemctl restart ssh

# Confirm SSH is running
sudo systemctl status ssh

# Allow SSH through UFW if needed
sudo ufw allow ssh
```

> ⚠️ If you disable password authentication, ensure your SSH key is
> configured and tested before restarting SSH — locking yourself out
> requires console access to recover.

---

### 3j: Audit Logging — Optional

📖 *Researched*

**When to apply this step:** If you need a record of system events for security review,
compliance, or incident investigation. Adds log overhead — not necessary for a
short-lived or isolated VM.

```bash
# Install auditd
sudo apt install auditd -y

# Enable and start the audit daemon
sudo systemctl enable auditd
sudo systemctl start auditd

# Confirm auditd is running
sudo systemctl status auditd

# View recent login events
sudo ausearch -m LOGIN --start today

# View all recent events
sudo aureport --start today
```

> 📓 Audit logs are stored in `/var/log/audit/audit.log`.
> Log rotation is handled automatically by auditd.
> For advanced audit rules, see the official linux-audit project:
> https://github.com/linux-audit/audit-userspace

</details>

---

## 🛡️ Step 4: Harden VirtualBox

<details>
<summary><strong>Open Step 4</strong></summary>

### Purpose
Reduce the attack surface between the VM guest and the host machine.

VirtualBox provides a hypervisor boundary between guest and host, but that boundary
is only as strong as the features you leave disabled. Every shared resource —
clipboard, USB, folders, network — is a potential channel for data or code to
cross that boundary. This step closes those channels.

---

### 4a: Disable Shared Clipboard and Drag-and-Drop

✅ *Tested*

**Why:** Shared clipboard creates a bidirectional channel between guest and host.
Drag-and-drop is effectively a file transfer mechanism. Both bypass network controls
and can be abused by a compromised guest to exfiltrate data or stage files on the host.
If you don't need them, disable them — the risk is not theoretical.

**GUI:**
```
Settings → General → Advanced
- Shared Clipboard: Disabled
- Drag'n'Drop: Disabled
```

**CLI:**
```bash
# Disable shared clipboard
VBoxManage modifyvm "Ubuntu-24.04" --clipboard-mode disabled

# Disable drag and drop
VBoxManage modifyvm "Ubuntu-24.04" --drag-and-drop disabled
```

---

### 4b: Disable Shared Folders

✅ *Tested*

**Why:** A shared folder mounts a host path inside the guest. Any process running in the
guest — including malware — with access to the filesystem can read, modify, or delete
files on the host through that mount point. This is one of the most significant
guest-to-host attack surfaces in a VM setup.

**GUI:**
```
Settings → Shared Folders
- Remove any shared folder entries unless explicitly required
```

**CLI:**
```bash
# List existing shared folders
VBoxManage showvminfo "Ubuntu-24.04" | grep "Shared folders"

# Remove a shared folder (replace FOLDER-NAME)
VBoxManage sharedfolder remove "Ubuntu-24.04" --name "FOLDER-NAME"
```

---

### 4c: Configure Network Isolation

✅ *Tested (NAT, Host-Only, Not Attached)* · 📖 *Researched (NAT Network, Internal, Bridged)*

Choose the network mode that matches your use case. The principle is minimum necessary
access — don't give the VM more network exposure than the task requires.

**Network Mode Comparison**

| Mode | Internet | Host Access | VM-to-VM | Use Case | When to Choose |
|------|----------|-------------|----------|----------|----------------|
| NAT | ✅ | ❌ | ❌ | General use, outbound internet needed | Default safe choice when internet access is required |
| NAT Network | ✅ | ❌ | ✅ | Multi-VM setups needing internet | Multiple VMs that need to communicate and reach the internet |
| Host-Only | ❌ | ✅ | ✅ | Development, host communication needed | Local dev/testing where internet isn't needed but host access is |
| Internal Network | ❌ | ❌ | ✅ | Isolated lab, VM-to-VM only | Simulating a network without exposing anything to the host |
| Not Attached | ❌ | ❌ | ❌ | Full isolation | Analysis of untrusted software; no network needed at all |
| Bridged | ✅ | ✅ | ✅ | ⚠️ VM exposed to physical network | Avoid unless explicitly required — VM is treated as a full network peer |

> 📓 **For most hardened setups:** use **Host-Only** or **Not Attached**.
>
> NAT is the VirtualBox default and is a reasonable starting point, but it still
> provides internet access. If your use case doesn't require internet, Not Attached
> is the most secure option.
>
> Bridged mode places the VM directly on your physical network, where it is visible
> to other devices and subject to the same threats as any physical machine on that network.
> Avoid it for hardened or sensitive workloads.

**GUI (All Platforms):**
```
Settings → Network → Adapter 1 → Attached to: [choose mode]
```

**CLI:**

```bash
# NAT — outbound internet, no host or VM-to-VM access
VBoxManage modifyvm "Ubuntu-24.04" --nic1 nat

# NAT Network — outbound internet with VM-to-VM communication
# Requires a NAT network to exist — create one first if needed
VBoxManage natnetwork add --netname "SecureNet" --network "10.0.2.0/24" --enable
VBoxManage modifyvm "Ubuntu-24.04" --nic1 natnetwork --natnetwork1 "SecureNet"

# Host-Only — no internet, VM can communicate with host and other VMs on same adapter
# Replace "vboxnet0" with your host-only adapter name (see tip below)
VBoxManage modifyvm "Ubuntu-24.04" --nic1 hostonly --hostonlyadapter1 "vboxnet0"

# Internal Network — no internet, no host access, VM-to-VM only
# The name "intnet" is arbitrary — must match across all VMs that need to communicate
VBoxManage modifyvm "Ubuntu-24.04" --nic1 intnet --intnet1 "intnet"

# Not Attached — full network isolation
VBoxManage modifyvm "Ubuntu-24.04" --nic1 none

# Bridged — not recommended for hardened setups
# VM appears as a device on your physical network
# Replace "eth0" with your actual host network interface (run 'ip link show' to find it)
VBoxManage modifyvm "Ubuntu-24.04" --nic1 bridged --bridgeadapter1 "eth0"
```

> 📓 Check available host-only adapters:
> ```bash
> VBoxManage list hostonlyifs
> ```
> Check available NAT networks:
> ```bash
> VBoxManage list natnetworks
> ```

---

### 4d: Disable USB Access

✅ *Tested*

**Why:** USB passthrough gives the guest direct access to physical USB hardware on the host.
A compromised guest could interact with USB storage (data exfiltration), HID devices
(keyboard/mouse injection), or USB firmware. If the VM doesn't need USB, disable all controllers.

**GUI:**
```
Settings → USB
- Uncheck "Enable USB Controller"
```

**CLI:**
```bash
# Disable all USB controllers
# OHCI = USB 1.1 | EHCI = USB 2.0 | xHCI = USB 3.0
VBoxManage modifyvm "Ubuntu-24.04" --usbohci off
VBoxManage modifyvm "Ubuntu-24.04" --usbehci off
VBoxManage modifyvm "Ubuntu-24.04" --usbxhci off
```

---

### 4e: Take a Clean Snapshot

✅ *Tested*

**Why:** A snapshot captures the complete VM state at a point in time. If the VM is
later compromised or misconfigured, you can restore to a known-good baseline rather
than rebuilding from scratch. The post-harden snapshot is your recovery point —
treat it as permanent.

**GUI:**
```
Machine → Take Snapshot → Name: "Post-Harden Baseline"
```

**CLI:**
```bash
# Take a snapshot of the current VM state
VBoxManage snapshot "Ubuntu-24.04" take "Post-Harden Baseline" \
  --description "Clean hardened baseline — Ubuntu 24.04, VirtualBox 7.x"
```

> 📓 Snapshots are stored within the VM directory. They are not external backups.
> Take a new snapshot after any significant configuration change.

</details>

---

## ✅ Step 5: Verify & Maintain

<details>
<summary><strong>Open Step 5</strong></summary>

### Purpose
Confirm the VM is secure and establish a maintenance routine.

---

### 5a: Verification Checklist

**VirtualBox Settings**
- [ ] Shared clipboard disabled
- [ ] Drag-and-drop disabled
- [ ] Shared folders removed or restricted
- [ ] USB controller disabled
- [ ] Network mode configured per threat model (see Step 4c)
- [ ] Clean snapshot taken and labeled

**Ubuntu OS**
- [ ] System fully updated
- [ ] UFW enabled and active
- [ ] Root account locked
- [ ] Automatic security updates enabled
- [ ] No unnecessary services running
- [ ] Login requires password

**Ubuntu OS — Optional Steps Applied**
- [ ] AppArmor active and enforcing (not complain mode)
- [ ] sysctl hardening applied (if applicable)
- [ ] SSH hardened or port 22 confirmed closed (if applicable)
- [ ] Audit logging enabled (if applicable)

---

### 5b: Ongoing Maintenance

```bash
# Run regularly to keep system updated and secure
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y

# Check firewall status
sudo ufw status verbose

# Review running services periodically
systemctl list-units --type=service --state=running

# Check for failed services
systemctl --failed
```

---

### 5c: Snapshot Strategy

```
Snapshot Name          When to Take
──────────────────────────────────────────────────────
Post-Harden Baseline   After completing Step 4 — never delete this
Post-Update            After major system updates
Pre-Change             Before any configuration changes
```

> 📓 Label snapshots with dates or descriptions.
> VirtualBox snapshots do not replace external backups — they only protect
> against changes within the VM, not against host-level failures.

</details>

---

## 🔖 References

<details>
<summary><strong>Open References</strong></summary>

**Official Sources**
1. [VirtualBox — Official Documentation](https://www.virtualbox.org/manual/)
2. [VirtualBox — VBoxManage Reference](https://www.virtualbox.org/manual/topics/vboxmanage.html)
3. [VirtualBox — Security Documentation](https://www.virtualbox.org/manual/topics/Security.html)
4. [Ubuntu — Security Guide](https://ubuntu.com/security)
5. [Ubuntu — UFW Documentation](https://help.ubuntu.com/community/UFW)
6. [Ubuntu — Automatic Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates)
7. [Linux Kernel — sysctl documentation](https://www.kernel.org/doc/html/latest/admin-guide/sysctl/)
8. [linux-audit — auditd project](https://github.com/linux-audit/audit-userspace)

</details>

---

*Authored and maintained by SaltedBytes. Last reviewed: 2026. Tested on Ubuntu 24.04 LTS / VirtualBox 7.x.*
