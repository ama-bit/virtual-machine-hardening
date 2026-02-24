# Virtual Machine Security Guide 🛡️
Secure Ubuntu VM setup in VirtualBox, from download verification, install, to hardened VM.

>
> This project emphasizes supply chain integrity and VM hardening.
> Always verify installation media before use.

---

## Scope

This repository provides a secure workflow:

Download -> Verify -> Install -> Harden -> Maintain

You will:

- Verify VirtualBox installer authenticity  
- Verify Ubuntu ISO integrity and signature  
- Install Ubuntu in VirtualBox securely  
- Apply baseline VM and OS hardening
- Create a maintainence checklist 

---

## Guide Structure

### Part 1: Download & Verification
Verify software authenticity before installation:

- VirtualBox installer SHA256 verification  
- Ubuntu ISO checksum verification  
- Optional GPG signature validation  

↪️ See: **verify.md**

---

### Part 2: VM Install & Hardening
Secure virtual machine deployment:

- VM creation with safe defaults  
- Ubuntu installation  
- OS hardening  
- VirtualBox isolation controls  
- Maintenance & verification  

↪️ See: **install-vm.md**

---

## Supported Host Platforms

- 💻 Linux  
- 🍏 macOS  
- 🪟 Windows (PowerShell)

---

## Security Principles

- Never install unverified software  
- Hash mismatch = possible compromise (delete file, download from official source, verify)  
- Trust cryptographic verification, not mirrors  
- Isolate VMs from host unless required  

---

## References

- https://www.virtualbox.org/manual/topics/Security.html  
- https://ubuntu.com/security  
- https://ubuntu.com/tutorials/how-to-verify-ubuntu  
- https://linuxsecurity.com/features/what-are-checksums-why-should-you-be-using-them  

---

## Result

Following this guide yields:

- Verified installation media  
- Secure Ubuntu VM  
- Hardened configuration baseline  
- Reproducible trusted setup

---

