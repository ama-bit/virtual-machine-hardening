# Virtual Machine Install & Harden 🛡️

Set up and harden an **Ubuntu** virtual machine (VM) in **VirtualBox** with secure 
defaults, including VM creation, installation, and basic post-install security.

---

## Scope

**This Guide**
```md
Create VM ➡️ Install Ubuntu ➡️ Harden OS ➡️ Harden VirtualBox ➡️ Verify & Maintain
```

**Previous Guide**
```md
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
```md
💻 Linux  
🍏 macOS  
🪟 Windows (PowerShell)
```

---

## Threat Model

This guide addresses the following threats:

1. Running unverified or compromised installation media
2. Weak default OS configuration post-install
3. Guest-to-host escape via shared resources
4. Unauthorized access through open ports or services
5. Persistence of a compromised state without rollback capability

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
| Network | NAT | Host-Only (see Step 4) |

> 📓 These specs are for Ubuntu 24.04 LTS desktop. Adjust based on your 
> host machine's available resources.

---

### GUI Steps (All Platforms)

<details>
<summary><strong>VirtualBox GUI</strong></summary>

```md
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

```md
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

```md
1. Select your VM in VirtualBox → Click "Start"
2. Ubuntu installer will boot from the attached ISO
3. Select "Try or Install Ubuntu"
```

<!-- 📸 SCREENSHOT NEEDED: Ubuntu installer welcome screen
     Show: "Try or Install Ubuntu" option highlighted -->

```md
4. Choose your language and keyboard layout
5. Select "Install Ubuntu"
6. Installation type:
   - Select "Erase disk and install Ubuntu"
   - Click "Advanced features" to access:
     - LVM (flexible disk management)
     - LVM with encryption (LUKS) — recommended if your threat model requires it
7. Set your timezone
8. Create your user account:
   - Use a strong password
   - Enable "Require password to log in"
   - Do not enable auto-login
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

</details>

---

### Post-Install Boot

After installation and restart:

```md
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

---

### 3a: Update System

```bash
# Update package lists and upgrade all installed packages
sudo apt update && sudo apt upgrade -y

# Remove unused packages
sudo apt autoremove -y
```

---

### 3b: Enable Firewall (UFW)

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

> 📓 Only open ports you explicitly need.
> Every open port increases attack surface.

---

### 3c: Disable Root Login

```bash
# Lock the root account — sudo user created during install is sufficient
sudo passwd -l root

# Confirm root is locked
sudo passwd -S root
# Expected output: root L (locked)
```

---

### 3d: Automatic Security Updates

```bash
# Install unattended-upgrades
sudo apt install unattended-upgrades -y

# Enable automatic security updates
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

---

### 3e: Verify Sudo User

```bash
# Confirm your user has sudo access
sudo -l

# Confirm root is not used for regular tasks
whoami
# Expected output: your username, not root
```

---

### 3f: Disable Unused Services — Optional

Reducing running services reduces attack surface.
Only disable services you do not need.

**Commonly safe to disable in a VM context:**

| Service | What it does | Disable if... |
|---------|-------------|---------------|
| bluetooth | Bluetooth support | No Bluetooth hardware or use |
| cups | Printing service | No printing needed |
| avahi-daemon | Network discovery (mDNS) | No local network discovery needed |
| ModemManager | Mobile broadband management | No modem attached |
| snapd | Snap package manager | Not using Snap packages — verify no dependencies first |

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

AppArmor ships with Ubuntu and enforces mandatory access controls on applications.
It should be active by default — this step confirms it.

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

---

### 3h: Kernel Hardening (sysctl) — Optional

Apply kernel-level security parameters to reduce attack surface.

> ⚠️ Understand each parameter before applying.
> Some settings may affect VM functionality depending on your use case.

```bash
# Check the current value of a specific parameter before changing it
sysctl net.ipv4.ip_forward

# Apply hardening parameters
sudo tee /etc/sysctl.d/99-hardening.conf <<EOF
# Disable IP forwarding — not needed for a standard desktop VM
net.ipv4.ip_forward = 0

# Disable ICMP redirects — prevents route manipulation
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Ignore ICMP broadcast requests
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Enable SYN flood protection
net.ipv4.tcp_syncookies = 1

# Restrict kernel pointer exposure
kernel.kptr_restrict = 2

# Restrict dmesg access to root only
kernel.dmesg_restrict = 1

# Disable core dumps for setuid programs
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

Apply only if SSH access to the VM is needed.
If SSH is not required, skip this step and ensure port 22 remains closed in UFW.

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
PermitRootLogin no

# Disable password authentication — use key-based auth only
PasswordAuthentication no

# Limit SSH to specific user (replace USERNAME)
AllowUsers USERNAME

# Set idle timeout — disconnect after 5 minutes of inactivity
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
> configured before restarting SSH or you will lose access.

---

### 3j: Audit Logging — Optional

Enable basic audit logging to track system events and detect suspicious activity.

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

---

### 4a: Disable Shared Clipboard and Drag-and-Drop

**GUI:**
```md
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

**GUI:**
```md
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

Choose the network mode that matches your use case and threat model.

**Network Mode Overview**

| Mode | Internet | Host Access | VM-to-VM | Use Case |
|------|----------|-------------|----------|----------|
| NAT | ✅ | ❌ | ❌ | General use, outbound internet needed |
| NAT Network | ✅ | ❌ | ✅ | Multi-VM setups needing internet |
| Host-Only | ❌ | ✅ | ✅ | Development, host communication needed |
| Internal Network | ❌ | ❌ | ✅ | Isolated lab, VM-to-VM only |
| Not Attached | ❌ | ❌ | ❌ | Full isolation, no network needed |
| Bridged | ✅ | ✅ | ✅ | ⚠️ VM exposed to physical network — not recommended for hardened setups |

> 📓 For most hardened setups, **Host-Only** or **Not Attached** are preferred.
> Use the minimum network access your use case requires.
> Bridged mode exposes the VM directly to your physical network and
> should be avoided unless explicitly required.

---

**GUI (All Platforms):**
```md
Settings → Network → Adapter 1 → Attached to:
- NAT
- NAT Network
- Host-Only Adapter
- Internal Network
- Not Attached
- Bridged Adapter (not recommended)
```

**CLI:**

```bash
# NAT — outbound internet, no host or VM-to-VM access
VBoxManage modifyvm "Ubuntu-24.04" --nic1 nat

# NAT Network — outbound internet with VM-to-VM communication
# Requires a NAT network to exist — create one first if needed
VBoxManage natnetwork add --netname "SecureNet" --network "10.0.2.0/24" --enable
VBoxManage modifyvm "Ubuntu-24.04" --nic1 natnetwork --natnetwork1 "SecureNet"

# Host-Only — no internet, VM can communicate with host and other VMs
# Replace "vboxnet0" with your host-only adapter name
VBoxManage modifyvm "Ubuntu-24.04" --nic1 hostonly --hostonlyadapter1 "vboxnet0"

# Internal Network — no internet, no host access, VM-to-VM only
# Replace "intnet" with your chosen internal network name
# This name must match across all VMs that need to communicate
VBoxManage modifyvm "Ubuntu-24.04" --nic1 intnet --intnet1 "intnet"

# Not Attached — full network isolation
VBoxManage modifyvm "Ubuntu-24.04" --nic1 none

# Bridged — not recommended for hardened setups
# VM is exposed directly to your physical network
# Replace "eth0" with your actual host network interface name
# Run 'ip link show' on Linux or 'ipconfig' on Windows to find it
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

**GUI:**
```md
Settings → USB
- Uncheck "Enable USB Controller"
```

**CLI:**
```bash
# Disable all USB controllers (OHCI = USB 1.1, EHCI = USB 2.0, xHCI = USB 3.0)
VBoxManage modifyvm "Ubuntu-24.04" --usbohci off
VBoxManage modifyvm "Ubuntu-24.04" --usbehci off
VBoxManage modifyvm "Ubuntu-24.04" --usbxhci off
```

---

### 4e: Take a Clean Snapshot

**GUI:**
```md
Machine → Take Snapshot → Name: "Post-Harden Baseline"
```

**CLI:**
```bash
# Take a snapshot of the current VM state
VBoxManage snapshot "Ubuntu-24.04" take "Post-Harden Baseline" --description "Clean hardened baseline"
```

> 📓 Snapshots allow rollback if the VM is later compromised or misconfigured.
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
- [ ] Clean snapshot taken

**Ubuntu OS**
- [ ] System fully updated
- [ ] UFW enabled and active
- [ ] Root account locked
- [ ] Automatic security updates enabled
- [ ] No unnecessary services running
- [ ] Login requires password

**Ubuntu OS — Optional Steps**
- [ ] AppArmor active and enforcing
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

```md
- Post-Harden Baseline  → taken after Step 4 (never delete)
- Post-Update           → taken after major system updates
- Pre-Change            → taken before any configuration changes
```

> 📓 Label snapshots clearly with dates or descriptions.
> VirtualBox snapshots do not replace external backups.

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

</details>

---

## Good Luck!
