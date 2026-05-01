# Virtual Machine Security & Verification Guide 🛡️

:construction: ***Draft — content under active development***

Securely set up an Ubuntu VM by verifying and installing **VirtualBox**,
then validating **Ubuntu** installation media using **SHA256** checksums and **GPG** signatures.

---

## Threat Model

Checksum and signature verification protect against:

1. Corrupted downloads
2. Compromised mirrors
3. Man-in-the-middle attacks (MITM)
4. Supply chain tampering

Verification confirms both **integrity** (file is unaltered) 
and **authenticity** (file came from the official source).

---

## Scope

**This Guide**
```md
Download ➡️ Verify SHA256 (Integrity) ➡️ Verify GPG Signature (Authenticity)
```

**Next Guide**
```md
➡️ Install ➡️ Configure ➡️ Harden VM
```
---

## Tools

| Tool | Purpose |
|------|---------|
| Oracle VM VirtualBox | Creates and runs VMs |
| Ubuntu ISO | Installation media for the guest OS |
| SHA256 hashing | Verifies file integrity |
| GnuPG (GPG) | Verifies checksum authenticity |
| Terminal / PowerShell | Runs verification commands |

💡*This guide provides support for the following operating systems/platforms:*
```md
💻 Linux  
🍏 macOS  
🪟 Windows (PowerShell)  
```
---

## 🌐 Step 1: Download from Official Sources

<details>
<summary><strong>Open Step 1</strong></summary>

❗ Always download software and checksums directly from 
official vendor-controlled pages.

---

### VirtualBox

**Official page**: https://www.virtualbox.org/wiki/Downloads

```md
1. Download installer under: VirtualBox Platform Packages
2. Note the SHA256 hash listed under: File Checksums → SHA256
```

> VirtualBox publishes individual hashes on its download page rather 
> than a checksum file. The hash must be compared manually in Step 2a.

**Installer file by platform:**
```md
Windows → .exe  
macOS   → .dmg  
Linux   → .run or package manager
```

> Do **not** install until SHA256 verification is complete in Step 2.

---

### Ubuntu

**Official page**: https://releases.ubuntu.com/

Download all three files:
```md
- ubuntu-XX.XX-desktop-amd64.iso  
- SHA256SUMS  
- SHA256SUMS.gpg  
```

> 📓 In some releases the checksum appears only after opening the ISO 
> download page. The location has changed in the past and may change again — always check the official 
> page directly.

> Do **not** install until all verification steps in Step 2 are complete.

</details>

---

## 🔍 Step 2: Verify Downloads

<details>
<summary><strong>Open Step 2</strong></summary>

### Why Verify?

- Ensures downloaded files exactly match the vendor's official release
- SHA256 hashes act as fingerprints — any alteration produces a 
  different hash
- GPG signatures confirm the checksum itself came from Ubuntu, 
  not a malicious mirror

### Verification Methods

| # | Method | Target | Type |
|---|--------|--------|------|
| 2a | SHA256 | VirtualBox installer | Manual |
| 2b | SHA256 | Ubuntu ISO (SHA256SUMS) | Automatic |
| 2c | GPG | Ubuntu SHA256SUMS signature | Cryptographic |

---

### 📦 Step 2a: VirtualBox Installer — SHA256 (Manual)

<details>
<summary><strong>Open Step 2a</strong></summary>

**Source**: https://www.virtualbox.org/wiki/Downloads

**Purpose**: Confirm the VirtualBox installer matches the official hash 
listed on the download page.

**Process**:
```md
1. Compute installer hash locally
2. Compare to hash listed on the VirtualBox download page
3. Both values must match exactly
```

> Replace filenames below with your downloaded VirtualBox file.

<details>
<summary>💻 Linux</summary>

```bash
cd ~/Downloads
sha256sum YOUR-FILENAME.run
```

</details>

<details>
<summary>🍏 macOS</summary>

```bash
cd ~/Downloads
shasum -a 256 YOUR-FILENAME.dmg
```

</details>

<details>
<summary>🪟 Windows (PowerShell)</summary>

```powershell
cd $env:USERPROFILE\Downloads
Get-FileHash .\YOUR-FILENAME.exe -Algorithm SHA256
```

</details>

> ⚠️ If hashes do not match — delete the installer and re-download 
> from the official page. Do not proceed.

</details>

---

### 🐧 Step 2b: Ubuntu ISO — SHA256 (Automatic)

<details>
<summary><strong>Open Step 2b</strong></summary>

**Source**: https://releases.ubuntu.com/

**Purpose**: Confirm the Ubuntu ISO matches the official SHA256SUMS file.

**Before running commands, place both files in the same folder**:
```md
- ubuntu-XX.XX-desktop-amd64.iso
- SHA256SUMS
```

<details>
<summary>💻 Linux</summary>

```bash
cd ~/Downloads
sha256sum -c SHA256SUMS 2>&1 | grep OK
```

Expected output:
ubuntu-24.04.1-desktop-amd64.iso: OK

</details>

<details>
<summary>🍏 macOS</summary>

```bash
cd ~/Downloads
shasum -a 256 ubuntu-24.04.1-desktop-amd64.iso
```

Compare the output hash against the matching entry in `SHA256SUMS`.

</details>

<details>
<summary>🪟 Windows (PowerShell)</summary>

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

Expected output:
✔ Hash matches — ISO verified

</details>

> ⚠️ If hashes do not match — delete the ISO and re-download 
> from the official page. Do not proceed.

</details>

---

### 🔏 Step 2c: Ubuntu SHA256SUMS — GPG Signature

<details>
<summary><strong>Open Step 2c</strong></summary>

**Source**: https://releases.ubuntu.com/

**Purpose**: Confirm the SHA256SUMS file was signed by Ubuntu — 
not replaced by a malicious mirror.

**Why this matters**:
```md
- A compromised mirror could replace both the ISO and its checksum
- GPG verification confirms the checksums were published by Ubuntu developers
- SHA256 alone cannot detect this — GPG is the final trust anchor
```

**Files needed** (both downloaded in Step 1):
```md
- SHA256SUMS
- SHA256SUMS.gpg
```

<details>
<summary>💻 Linux / 🍏 macOS</summary>

```bash
gpg --keyid-format long --verify SHA256SUMS.gpg SHA256SUMS
```

Expected output:
Good signature from "Ubuntu CD Image Automatic Signing Key"

</details>

<details>
<summary>🪟 Windows (PowerShell)</summary>

Requires **Gpg4win**: https://www.gpg4win.org/

```bash
gpg --verify SHA256SUMS.gpg SHA256SUMS
```

Expected output:
Good signature from "Ubuntu CD Image Automatic Signing Key"

</details>

---

### ⚠️ First-Time Trust Warning

You may see:
WARNING: This key is not certified with a trusted signature

This is **normal** on first use and does **not** indicate a bad signature.

```md
GPG distinguishes between:

- Signature validity  ➡️ cryptographically correct (this is what matters)
- Key trust          ➡️ whether you have personally verified the signer
```

The signature is valid as long as the output reads **"Good signature"**.

> ⚠️ If the output reads "BAD signature" — do not proceed. 
> Delete all files and re-download from the official source.

</details>

</details>

---

## ✅ Step 3: Verification Checklist

<details>
<summary><strong>Open Checklist</strong></summary>

**VirtualBox**
- [ ] Installer downloaded from https://www.virtualbox.org/wiki/Downloads
- [ ] SHA256 hash matches official value on download page

**Ubuntu**
- [ ] ISO downloaded from https://releases.ubuntu.com/
- [ ] SHA256SUMS downloaded from https://releases.ubuntu.com/
- [ ] SHA256SUMS.gpg downloaded from https://releases.ubuntu.com/
- [ ] ISO SHA256 hash verified against SHA256SUMS
- [ ] GPG signature reports "Good signature"

**Final Check**
- [ ] No hash mismatches
- [ ] No unexpected errors or warnings

> 🛑 If any item is unchecked — do not proceed. 
> Delete affected files and re-download from the official source.

</details>

---

## ⏸️ Safe to Proceed?

<details>
<summary><strong>Open Check</strong></summary>

All of the following must be true before installing:

```md
✅ VirtualBox installer hash matches the official download page value
✅ Ubuntu ISO hash matches the SHA256SUMS file
✅ GPG signature reports "Good signature"
✅ All downloads sourced directly from official vendor domains over HTTPS

❗ Failure of any check = possible corruption or compromise
   Delete affected files and re-download from the official source.
```

---

### Ubuntu ISO — Usage Note
```md
- The ISO is not installed directly
- Attach it as boot media inside VirtualBox
- No extraction needed
```

---

### Security Reminders 💭
```md
1. Never install unverified software
2. Never ignore hash mismatches
3. Delete and re-download from the official page if verification fails
```

> 🔐 **VirusTotal** can scan files and hashes across multiple AV engines 
> as a supplementary check: https://www.virustotal.com  
> Note: VirusTotal does not replace GPG verification and cannot detect 
> zero-day or unknown threats.

</details>

---

## 🔖 References

<details>
<summary><strong>Open References</strong></summary>

**Official Sources**
1. [Ubuntu — How to Verify Your Ubuntu Download](https://ubuntu.com/tutorials/how-to-verify-ubuntu#1-overview)
2. [Ubuntu — GPG Signature Verification](https://ubuntu.com/tutorials/how-to-verify-ubuntu#4-retrieve-the-correct-signature-key)
3. [Ubuntu — SHA256SUMS](https://help.ubuntu.com/community/HowToSHA256SUM)
4. [VirtualBox — Security Documentation](https://www.virtualbox.org/manual/topics/Security.html)
5. [VirtualBox — Downloads](https://www.virtualbox.org/wiki/Downloads)


</details>

---

## Good Luck!
