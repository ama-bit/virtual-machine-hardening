# Secure VM Setup Guide🛡️
⚠️ Draft: In progress...
Secure Ubuntu Virtual Machine (VM) setup in VirtualBox, from download verification, install, to hardened VM.

> This project emphasizes supply chain integrity and VM hardening.
> It is meant as a basic guide to securely setup a VM in VirtualBox using an ISO like Ubuntu.
> Always verify installation media before use.

---
## Goals

**Verify**

1. **Official** Source Pages for all downloads before downloading
  
2. **VirtualBox installer** authenticity
   
3. **Ubuntu ISO** integrity and signature
   
**Download**

4. Securely install **Ubuntu in VirtualBox** after full verification is successful

**Harden/Secure**

5. Apply baseline **VM and OS hardening**
   
**Maintain**

6. Create a secure maintainence **checklist**

---

## Steps

### Step 1: Download & Verification
Verify software authenticity before installation:

1. VirtualBox installer **SHA256** verification
2. Ubuntu **ISO** checksum verification
3. **GPG** signature validation  

↪️ See: **vm-verify.md**

---

### Step 2: VM Install & Hardening

Secure VM deployment:

1. **Create VM**
2. **Install Ubuntu**
3. **Harden OS**
   - Disable shared clipboard, drag-and-drop, and host USB access
   - Disable or limit Guest Additions based on security needs
4. **Configure VirtualBox Isolation**
   - Set networking depending on setup needs
   - Disable shared folders unless required
5. **Verify**
   - Confirm isolation settings are applied and active
6. **Maintain**
   - Keep VirtualBox, Guest Additions (if used), and Ubuntu updated
   - Re-verify settings after any major update

↪️ See: **vm-install-harden.md**

---

## Supported Host Platforms for Guides 1 & 2

💻 Linux  
🍏 macOS  
🪟 Windows (PowerShell)

---

## Security Principles

- Never install unverified software  
- Hash mismatch = possible compromise (delete file, download from official source, verify)  
- Trust cryptographic verification, not mirrors  
- Isolate VMs from host unless required  

---

## Result

The outcome after following this guide is to have verified installation media, a secure Ubuntu VM with a hardened configuration baseline, and a reproducible trusted setup including checklists for future reference.

---

## References
1. https://www.virtualbox.org/manual/topics/Security.html
2. https://ubuntu.com/security
3. https://ubuntu.com/tutorials/how-to-verify-ubuntu
4. https://linuxsecurity.com/features/what-are-checksums-why-should-you-be-using-them
5. https://help.ubuntu.com/community/HowToSHA256SUM

---

## Good Luck!
