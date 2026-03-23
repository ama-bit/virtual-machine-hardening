# Virtual Machine Security & Verification Guide 🛡️

:construction: ***Draft — content under active development***

Securely set up an Ubuntu VM by verifying and installing **VirtualBox**,
then validating **Ubuntu** installation media using **SHA256** checksums and **GPG** signatures.

---

## Table of Contents

- [Threat Model](#threat-model)
- [Scope](#scope)
- [Platform Specific Commands](#platform-specific-commands)
- [Step 1: Download from Official Sources](#-step-1-download-from-official-sources)
- [Step 2: Verify Downloads](#-step-2-verify-downloads)
  - [Step 2a: Manual Verification (VirtualBox)](#-step-2a-manual-verification-virtualbox-installer)
  - [Step 2b: Automatic Verification (Ubuntu ISO)](#-step-2b-automatic-verification-ubuntu-iso)
  - [Step 2c: Verify Ubuntu Checksum Signature (GPG)](#-step-2c-verify-ubuntu-checksum-signature-gpg)
- [Safe to Proceed?](#️-safe-to-proceed)
- [Verification Checklist](#-step-3-verification-checklist)
- [Links](#-links)

---

## Threat Model

Checksum and signature verification protect against:

1. Corrupted downloads
2. Compromised mirrors
3. Man-in-the-middle attacks (MITM)
4. Supply chain tampering

Verification uses hashes and signatures to confirm both integrity and authenticity.

---

## Tools

The following tools are used to verify software authenticity and integrity before installation.

1. **Oracle VM VirtualBox**: For creating VMs
2. **Ubuntu ISO**: Installation media
3. **GnuPG**: Verify checksum signatures
4. **SHA256 hashing**: Verifying file integrity
5. **Powershell/Terminal**: Running verification commands

---

## Scope

**This Guide**
```md
Download ISO ➡️ Verify GPG Signature (Authenticity) ➡️ Verify ISO Hash (Integrity)
```

**Next Guide**
```md
➡️ Install ➡️ Configure ➡️ Harden VM
```
---

### Platform Specific Commands

This guide provides support for the following operating systems/platforms:
```md
💻 Linux  
🍏 macOS  
🪟 Windows (PowerShell)  
```
---

## 🌐 Step 1: Download from Official Sources

<details>
<summary><strong>Open Step 1</strong></summary>

### VirtualBox

VirtualBox publishes individual hashes on its website rather than a checksum file for batch verification so the installer hash must be compared manually. 
- Oracle VM VirtualBox
VirtualBox Download Page:  
https://www.virtualbox.org/wiki/Downloads

```
- Download the installer under `VirtualBox Platform Packages`
- SHA256 hash is listed under `File Checksums -> SHA256`
```
❗ Always obtain software and checksum data from official vendor-controlled pages.  
> Once downloaded, do **not** install until hash verification is complete, which is covered in **Step 2**.

---

### Ubuntu

Official ISO, checksum, and GPG files:  
https://releases.ubuntu.com/
```md
- ubuntu-XX.XX-desktop-amd64.iso  
- SHA256SUMS  
- (optional) SHA256SUMS.gpg  
```
📓 In some releases, I've noticed the checksum now appears only after opening the ISO download page.

</details>

---

## 🔍 Step 2: Verify Downloads

<details>
<summary><strong>Open Step 2</strong></summary>

### Why Verify?
```markdown
- Verification ensures downloaded files exactly match the vendor’s official release
and have not been altered.

- SHA256 hashes are similar to fingerprints, any difference produces a different print, or hash.

- Peace of mind knowing you've done your due-diligence in mitigating risk of installing tampered or corrupt files.

```
### Verification methods covered

```markdown
1. Manual - VirtualBox installer (website hash)  

2. Automatic - Ubuntu ISO (SHA256SUMS)  

3. Optional - Ubuntu checksum signature (GPG)  
```
---

### 📦 Step 2a: Manual Verification (VirtualBox Installer)

<details>
<summary><strong>Open Step 2a</strong></summary>

### Official Page

**VBox Download & Checksums**: https://www.virtualbox.org/wiki/Downloads

### Purpose

```md
Verify VirtualBox (VBox) installer hash matches official checksum.
Hash values must match exactly before installing.
```

### Process
```md
1. Compute installer hash locally  
2. Compare to website value  
3. Verify exact match  
```
### Choose Your System

Click on your specific OS below to see tailored commands. Replace the VirtualBox example filename with your downloaded VBox file.

<details>
<summary>💻 Linux</summary>

```bash
cd ~/Downloads
sha256sum YOUR-FILENAME.run
```

</details> <details> <summary>🍏 macOS</summary>

```bash
cd ~/Downloads
shasum -a 256 YOUR-FILENAME.dmg
```

</details> <details> <summary>🪟 Windows (PowerShell)</summary>

```powershell
cd $env:USERPROFILE\Downloads
Get-FileHash .\YOUR-FILENAME.exe -Algorithm SHA256
```
> :warning: Never bypass hash verification.
> :warning: A mismatched hash or signature could indicate a malicious or corrupt file.
> :warning: Only proceed when all verification steps pass.

</details>

If hashes do NOT match ➡️ delete installer, investigate and re-download from official site

</details>

---

### 🐧 Step 2b: Automatic Verification (Ubuntu ISO)

<details> 
<summary><strong>Open Step 2b</strong></summary>
    
### Purpose
Verify Ubuntu ISO matches official checksum list.

## Process

Run the checksum verification command to compare your ISO’s SHA256 hash
against the official values listed in `SHA256SUMS`.

**Place both files in same folder**:
```md
Ubuntu ISO

SHA256SUMS
```

## Choose Your System

<details> <summary>💻 Linux</summary>

**Verify**

```bash
cd ~/Downloads
sha256sum -c SHA256SUMS 2>&1 | grep OK
```

Expected Output: `ubuntu-24.04.1-desktop-amd64.iso: OK`

References:

Ubuntu Community: [HowToSHA256SUM](https://help.ubuntu.com/community/HowToSHA256SUM)

GeeksforGeeks: [SHA-256 on Linux/Unix](https://www.geeksforgeeks.org/linux-unix/generating-an-sha-256-hash-from-the-command-line/)

To generate a SHA-256 checksum: sha256sum filename
To verify: sha256sum -c checksumfile
The checksumfile should contain the expected hash followed by the filename.

Linux man page: [sha256sum](https://www.linux.org/docs/man1/sha256sum.html)

-c, --check -> check SHA256 checksums from a file


</details> <details> <summary>🍏 macOS</summary>

```bash
cd ~/Downloads
shasum -a 256 ubuntu-24.04.1-desktop-amd64.iso
# Compare resulting hash with SHA256SUMS entry
```

Compare the resulting hash with the value listed for the ISO in 'SHA256SUMS'.

</details> 
<details> <summary>🪟 Windows (PowerShell)</summary>

```powershell
cd $env:USERPROFILE\Downloads

$iso = "ubuntu-24.04.1-desktop-amd64.iso"

$isoHash = (Get-FileHash $iso -Algorithm SHA256).Hash
$officialHash = (Select-String $iso SHA256SUMS).Line.Split(" ")[0]

if ($isoHash -eq $officialHash) {
    "✔ Hash matches — ISO verified"
} else {
    "✖ Hash mismatch — DO NOT USE"
}
```

Expected Output: `✔ Hash matches — ISO verified`

> :warning: Never bypass hash verification.
> :warning: A mismatched hash or signature could indicate a malicious or corrupt file.
> :warning: Only proceed when all verification steps pass.

</details> </details>

---

### 🔏 Step 2c: Verify Ubuntu Checksum Signature (GPG)

<details> <summary><strong>Open Step 2c</strong></summary>

### **Purpose**
- Verify SHA256SUMS file is authentic and signed by Ubuntu.

## Why this matters
```markdown
- Prevents a malicious mirror from replacing both the ISO and checksum  
- Confirms the checksums were published by Ubuntu developers  
```
## Files needed:
```md
- `SHA256SUMS`

- `SHA256SUMS.gpg`
```
### Choose Your System
<details> <summary>💻 Linux / 🍏 macOS</summary>

```bash
gpg --keyid-format long --verify SHA256SUMS.gpg SHA256SUMS
```

Expected Output: `"Good signature from "Ubuntu CD Image Automatic Signing Key"`

</details> <details> <summary>🪟 Windows (PowerShell)</summary>

```bash
# Windows requires Gpg4win
gpg --verify SHA256SUMS.gpg SHA256SUMS
```
You may need to install Gpg4win which provides the gpg command, if you dont have it installed already.

Expected Output:
```bash
Good signature from "Ubuntu CD Image Automatic Signing Key"
```
</details>

```md
⚠️ First-time trust warning is normal.

You may see:

`WARNING: This key is not certified with a trusted signature`

This does **not** indicate a bad signature.

It means your local GPG keyring has not yet assigned trust to the Ubuntu signing key.

GPG separates:

- **Signature validity** ➡️ cryptographically correct  
- **Key trust** ➡️ whether you personally trust the signer
```
> :warning: Never bypass hash verification.
> :warning: A mismatched hash or signature could indicate a malicious or corrupt file.
> :warning: Only proceed when all verification steps pass.

</details>

---

## ⏸️ Safe to Proceed?

<details>
<summary><strong>Open Check</strong></summary>

### An ISO is considered trustworthy only if:

```md
- Downloaded directly from the official distribution site  
- Cryptographic hash matches the official checksum list  
- Verification reports success with no warnings or errors
- Download performed over HTTPS from the official vendor domain
- GPG signature verification reports "Good Signature" 

✅ Result: authenticity and integrity verified  

❗ Failure of any check indicates possible corruption or malicious alteration  
Abort installation and obtain a fresh copy from the official source.
```

---

### By now you have obtained and verified the following:
```md
1. VirtualBox installer  
2. Ubuntu ISO  
3. GPG signature (optional)  
```
🛑 Only proceed if all hashes match official values.

---

### VirtualBox Installer Files per Platform/OS
```md
Windows → `.exe`  
macOS → `.dmg`  
Linux → `.run` or package manager  
```

---

### Ubuntu ISO File Use
```md
- ISO is not installed directly  
- Attach as boot media in VirtualBox  
- No extraction needed  
```

---

### Security Thoughts 💭
```md
1. Never install unverified downloads  
2. Never ignore hash mismatches  
3. Delete & re-download from official page if verification fails  
```

</details>

---

## ✅ Step 3: Verification Checklist
<details> <summary><strong>✅ Open Checklist</strong></summary>   
  
- [ ] VirtualBox installer downloaded  
- [ ] VirtualBox SHA256 verified  
- [ ] Ubuntu ISO downloaded  
- [ ] Ubuntu ISO SHA256 verified  
- [ ] SHA256SUMS signature verified (optional)  
- [ ] All files match official checksums  
- [ ] GPG signature valid  
- [ ] No warnings or errors

> 🛑 If any item is unchecked, do not proceed. Delete affected files and re-download. 🛑

</details>

---

## 🔖 Links
<details> <summary><strong>🔖 Open Links</strong></summary>    

    
1. [Ubuntu Image Verification Guide](https://ubuntu.com/tutorials/how-to-verify-ubuntu#1-overview)

2. [Ubuntu Image Verification Guide for GPG](https://ubuntu.com/tutorials/how-to-verify-ubuntu#4-retrieve-the-correct-signature-key)

3. [VirtualBox Security & Verification](https://www.virtualbox.org/manual/topics/Security.html)

4. [Linux Security](https://linuxsecurity.com/features/what-are-checksums-why-should-you-be-using-them)

5. [Generate a Hash From CLI](https://www.geeksforgeeks.org/linux-unix/generating-an-sha-256-hash-from-the-command-line/)


>
> 🔐 VirusTotal scans files, URLS, and hashes across multiple AV engines for known threats:
> https://www.virustotal.com/gui/home/url
> VT can not detect zero-day attacks or unknown threats.
> Always use caution when installing files, clicking links, or verifying hashes.


</details>

---

## Thank You!

---
