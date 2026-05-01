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

**Attach the verified Ubuntu ISO:**
```md
1. Select your VM → Click "Settings"
2. Navigate to Storage
3. Under Controller: IDE → Click the empty disk icon
4. Click the disk icon on the right → "Choose a disk file"
5. Select your verified ubuntu-XX.XX-desktop-amd64.iso
6. Click OK
```

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
4. Choose your language and keyboard layout
5. Select "Install Ubuntu"
6. Installation type:
   - Erase disk and install Ubuntu (recommended for VM)
   - Enable LVM if you want flexible disk management
   - Enable disk encryption (LUKS) if required by your threat model
7. Set your timezone
8. Create your user account:
   - Use a strong password
   - Enable "Require password to log in"
   - Do not enable auto-login
9. Complete installation and restart when prompted
10. Remove ISO when prompted or via Settings → Storage
```

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

### 3f: Optional — Disable Unused Services

```bash
# List running services
systemctl list-units --type=service --state=running

# Disable a service (replace SERVICE with the service name)
sudo systemctl disable --now SERVICE

# Example: disable Bluetooth if not needed
sudo systemctl disable --now bluetooth
```

> 📓 Only disable services you understand.
> Disabling critical services can break system functionality.

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

**GUI:**
```md
Settings → Network → Adapter 1
- For internet access:     NAT
- For host isolation:      Host-Only
- For full isolation:      Not Attached
```

**CLI:**
```bash
# Set network to Host-Only (replace "vboxnet0" with your host-only adapter name)
VBoxManage modifyvm "Ubuntu-24.04" --nic1 hostonly --hostonlyadapter1 "vboxnet0"

# Or disable network entirely for full isolation
VBoxManage modifyvm "Ubuntu-24.04" --nic1 none
```

> 📓 Check available host-only adapters:
> ```bash
> VBoxManage list hostonlyifs
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
# Disable USB controller
VBoxManage modifyvm "Ubuntu-24.04" --usbehci off
VBoxManage modifyvm "Ubuntu-24.04" --usbxhci off
```

---

### 4e: Take a Clean Snapshot

```md
After hardening is complete, take a snapshot to preserve the secure baseline.

GUI:
Machine → Take Snapshot → Name: "Post-Harden Baseline"

CLI:
```
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
- [ ] Network set to Host-Only or None
- [ ] Clean snapshot taken

**Ubuntu OS**
- [ ] System fully updated
- [ ] UFW enabled and active
- [ ] Root account locked
- [ ] Automatic security updates enabled
- [ ] No unnecessary services running
- [ ] Login requires password

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
