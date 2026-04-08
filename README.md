# Virtual Machine Security Guide 🛡️
⚠️ Draft: In progress...
Secure Ubuntu VM setup in VirtualBox, from download verification, install, to hardened VM.

> This project emphasizes supply chain integrity and VM hardening.
> It is meant as a basic guide for anyone looking to setup a virtual machine in VirtualBox using an ISO like Ubuntu.
> Always verify installation media before use.

---

## Goals

> **Verify** Source --> **Download** Files --> **Verify** Integrity --> **Install** Files --> **Harden** VM --> **Maintain** > **Secure** VM

1. Verify **Official** Source Pages for all downloads before downloading
  
2. Verify **VirtualBox installer** authenticity
   
3. Verify **Ubuntu ISO** integrity and signature
   
4. Securely install **Ubuntu in VirtualBox**
   
5. Apply baseline **VM and OS hardening**
   
6. Create a secure maintainence **checklist**

---

## Steps

### Step 1: Download & Verification
Verify software authenticity before installation:

1. VirtualBox installer **SHA256** verification
2. Ubuntu ISO checksum verification
3. Optional *but* reccomended **GPG** signature validation  

↪️ See: **vm-verify.md**

---

### Step 2: VM Install & Hardening

Secure VM deployment:

1. Create VM
2. Install Ubuntu
3. Harden OS
4. Configure VirtualBox isolation controls
5. Verify and maintain  

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
6. https://www.geeksforgeeks.org/linux-unix/generating-an-sha-256-hash-from-the-command-line/
7. https://www.linux.org/docs/man1/sha256sum.html

---

## Thank you!
