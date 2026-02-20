# Virtual Machine Security & Verification Guide 🛡️

⚠️ Draft — content under active refinement

Securely set up an Ubuntu virtual machine by verifying and installing VirtualBox,
then validating Ubuntu installation media using SHA256 checksums and GPG signatures.

---

### Verification Flow

```md
Download ➡️ Verify Hash ➡️ Verify Signature (optional) ➡️ Install
```
---

### Platform Specific Commands

This guide provides support for the following operating systems/platforms:
```md
• 💻 Linux  
• 🍏 macOS  
• 🪟 Windows (PowerShell)  
```
---

## 🌐 Step 1: Download from Official Sources

<details>
<summary><strong>Open Step 1</strong></summary>

### VirtualBox

Official downloads and SHA256 hashes:  
https://www.virtualbox.org/wiki/Downloads

```
- Download the installer under `VirtualBox Platform Packages`
- SHA256 hash is listed under `File Checksums → SHA256`
```
❗ Always obtain software and checksum data from official vendor-controlled pages.  
> Once downloaded, do **not** install until hash verification is complete (Step 2).

---

### Ubuntu

Official ISO, checksum, and GPG files:  
https://releases.ubuntu.com/
```md
- ubuntu-XX.XX-desktop-amd64.iso  
- SHA256SUMS  
- (optional) SHA256SUMS.gpg  
```
📓 In some releases, the checksum appears only after opening the ISO download page.

</details>

---

## 🔍 Step 2: Verify Downloads

<details>
<summary><strong>Open Step 2</strong></summary>

### Why Verify?
```markdown
- Verification ensures downloaded files exactly match the vendor’s official release
and have not been altered in transit.

- SHA256 hashes act as cryptographic fingerprints — even a 1-byte difference produces a different hash.
```
### Verification methods covered

```markdown
1. Manual — VirtualBox installer (website hash)  
2. Automatic — Ubuntu ISO (SHA256SUMS)  
3. Optional — Ubuntu checksum signature (GPG)  
```
---

### 📦 Step 2a: Manual Verification (VirtualBox Installer)

<details>
<summary><strong>Open Step 2a</strong></summary>

### Purpose

```md
Verify VirtualBox installer hash matches official checksum.

Official hashes: https://www.virtualbox.org/wiki/Downloads

Hash values must match exactly before installing.
```
### Process
```md
1. Compute installer hash locally  
2. Compare to website value  
3. Verify exact match  
```
### Choose Your System

<details>
<summary>💻 Linux</summary>

```bash
cd ~/Downloads
sha256sum VirtualBox-6.X.X-xxxxxx-Linux.run
```

</details> <details> <summary>🍏 macOS</summary>

```bash
cd ~/Downloads
shasum -a 256 VirtualBox-6.X.X-xxxxxx-OSX.dmg
```

</details> <details> <summary>🪟 Windows (PowerShell)</summary>

```powershell
cd $env:USERPROFILE\Downloads
Get-FileHash .\VirtualBox-6.X.X-xxxxxx-Win.exe -Algorithm SHA256
```

</details>

❗ Mismatch → delete installer and re-download from official site

</details>

---

### 🐧 Step 2b: Automatic Verification (Ubuntu ISO)

<details> 
<summary><strong>Open Step 2b</strong></summary>
    
## Purpose
Verify Ubuntu ISO matches official checksum list.

## Process

Run the checksum verification command to compare your ISO’s SHA256 hash
against the official values listed in `SHA256SUMS`.
```md
Place both files in same folder:

Ubuntu ISO

SHA256SUMS
```

Choose Your System

<details> <summary>💻 Linux</summary>

```bash
cd ~/Downloads
sha256sum -c SHA256SUMS 2>&1 | grep ubuntu-24.04.1-desktop-amd64.iso
```

Expected Output: `ubuntu-24.04.1-desktop-amd64.iso: OK`

</details> <details> <summary>🍏 macOS</summary>

```bash
cd ~/Downloads
shasum -a 256 -c SHA256SUMS 2>&1 | grep ubuntu-24.04.1-desktop-amd64.iso
```

Expected Output: `ubuntu-24.04.1-desktop-amd64.iso: OK`

</details> 
<details> <summary>🪟 Windows (PowerShell)</summary>

```powershell
cd $env:USERPROFILE\Downloads

$isoHash = (Get-FileHash .\ubuntu-24.04.1-desktop-amd64.iso -Algorithm SHA256).Hash
$officialHash = (Select-String "ubuntu-24.04.1-desktop-amd64.iso" .\SHA256SUMS).ToString().Split(" ")[0]

if ($isoHash -eq $officialHash) {
    Write-Host "✔ Hash matches — ISO verified"
} else {
    Write-Host "✖ Hash mismatch — DO NOT USE"
}
```
</details> </details>

---

### 🔏 Step 2c: Verify Ubuntu Checksum Signature (GPG)

<details> <summary><strong>Open Step 2c</strong></summary>

## **Purpose**
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
Choose Your System
<details> <summary>💻 Linux / 🍏 macOS</summary>

Expected Output: `Good signature from "Ubuntu CD Image Automatic Signing Key"`

</details> <details> <summary>🪟 Windows (PowerShell)</summary>

Expected Output: `Good signature from "Ubuntu CD Image Automatic Signing Key"`

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
</details> </details>

---

## ⏸️ Safe to Proceed?

<details>
<summary><strong>Open Check</strong></summary>

### An ISO is considered trustworthy only if:
```md
- Obtained directly from the official vendor distribution site  
- Cryptographic hash matches the official checksum list  
- Verification reports success with no warnings or errors  

✅ Result: authenticity and integrity verified  

❗ Failure of any check indicates possible corruption or malicious alteration  
Abort installation and obtain a fresh copy from the official source.
```
</details>

---

## 📥 Step 3: Install

<details>
<summary><strong>Open Step 3</strong></summary>

### Congrats 🎇

By now you have obtained and verified the following:
```md
- VirtualBox installer  
- Ubuntu ISO  
- GPG signature (optional)  
```
🛑 Only proceed if all hashes match official values.

---

### VirtualBox Installation
```md
- Windows → `.exe`  
- macOS → `.dmg`  
- Linux → `.run` or package manager  
```
Follow installer prompts.

---

### Ubuntu ISO Usage
```md
- ISO is not installed directly  
- Attach as boot media in VirtualBox  
- No extraction needed  
```
---

### Security Guidance
```md
- Never install unverified downloads  
- Never ignore hash mismatches  
- Delete & re-download from official page if verification fails  
```
</details>

---

## 🔖 Links
<details> <summary><strong>🔖 Open Links</strong></summary>    

    
1. [Ubuntu Image Verification Guide](https://ubuntu.com/tutorials/how-to-verify-ubuntu#1-overview)

2. [Ubuntu Image Verification Guide for GPG](https://ubuntu.com/tutorials/how-to-verify-ubuntu#4-retrieve-the-correct-signature-key)

3. [VirtualBox Security & Verification](https://www.virtualbox.org/manual/topics/Security.html)

4. [Linux Security](https://linuxsecurity.com/features/what-are-checksums-why-should-you-be-using-them)

5. [Generate a Hash From CLI](https://www.geeksforgeeks.org/linux-unix/generating-an-sha-256-hash-from-the-command-line/)

---

>
> 🔐 Security Tip: VirusTotal scans files, URLS, and hashes across multiple AV engines for known threats:
> https://www.virustotal.com/gui/home/url

</details>

---
