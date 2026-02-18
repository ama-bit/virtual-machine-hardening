# Virtual Machine Hardening Guide 🛡️

*Work in progress* ⚠️

---

# Step 1: Verify Downloads 🚩

Before installing software such as **VirtualBox** or **Ubuntu**, always verify file **integrity** and **authenticity**.

This ensures the download has not been corrupted or tampered. Verifying the integrity of files is recommended by Ubuntu, VirtualBox, NIST, and SANS.

---

# Step 1a: Verify VirtualBox Installer 📦

VirtualBox publishes SHA256 checksums on its official downloads page:
https://www.virtualbox.org/wiki/Downloads  

⚠️ Replace filenames below with the installer you downloaded.

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

```bash
cd $env:USERPROFILE\Downloads
Get-FileHash .\VirtualBox-6.X.X-xxxxxx-Win.exe -Algorithm SHA256
```

</details>

- Compare the output hash with the value listed on the VirtualBox website.
    - Comparing hash outputs in the CLI is covered in later sections.
- Hash values must match exactly before installing. Delete the file and re-download from the official site if hash values dont match.

💡 On Windows, installers are also digitally signed.
Signature validation adds assurance but does not replace hash verification.

---

# Step 1b: Verify Ubuntu ISO 🐧

## Download from the official Ubuntu releases page

Ubuntu publishes official SHA256 checksums for every release image.
Always obtain both the ISO and checksum files from the official releases site:

- https://releases.ubuntu.com/

    - Download:

        - `ubuntu-XX.XX-desktop-amd64.iso`

        - `SHA256SUMS`

        - *(optional)* `SHA256SUMS.gpg`

---

> 🛑 Replace `ubuntu-XX.XX-desktop-amd64.iso` with your actual downloaded filename.

---

# Step 1c: Prepare
## Files required for Hash Comparison 🗃️

You should have these files in the same folder:

1. **Ubuntu installation image**: `ubuntu-XX.XX-desktop-amd64.iso`

2. **Official checksum list**: `SHA256SUMS`

3. **Signed checksum file**: `SHA256SUMS.gpg` (optional)
   - Ubuntu signs the checksum file to prove it was published by Ubuntu and not altered.
   - Guide: https://ubuntu.com/tutorials/how-to-verify-ubuntu#4-retrieve-the-correct-signature-key

---

> 📔 The SHA256 checksum may not be directly visible on the main release page.
> In some Ubuntu releases, you must click the ISO download link to get to where `SHA256SUMS` is listed.

---

# Step 2: Automatic ISO Verification (Recommended) 🔎

Place both files in the same folder:

```bash
ISO file
SHA256SUMS
```

<details> <summary>💻 Linux</summary>

```bash
cd ~/Downloads
sha256sum -c SHA256SUMS 2>&1 | grep ubuntu-24.04.1-desktop-amd64.iso
```

Expected:

ubuntu-24.04.1-desktop-amd64.iso: OK

### Explanation of Linux ISO Hash Verification

- `sha256sum -c SHA256SUMS`  
  Reads the downloaded **SHA256SUMS** file (from Ubuntu) and recomputes the SHA256 hash of each listed file on your system to check for a match.

- `2>&1`  
  Redirects error messages into standard output so they appear in the terminal output stream.

- `grep ubuntu-24.04.1-desktop-amd64.iso`  
  Filters the results to show only the line for your specific ISO file.

🛑 Replace `ubuntu-24.04.1-desktop-amd64.iso` with the exact filename you downloaded.  
The result must show `OK` and anything else means the file should **not** be used.

---

</details> <details> <summary>🍏 macOS</summary>

```bash
cd ~/Downloads
shasum -a 256 -c SHA256SUMS 2>&1 | grep ubuntu-24.04.1-desktop-amd64.iso
```

Expected:

ubuntu-24.04.1-desktop-amd64.iso: OK

### Explanation of macOS ISO Hash Verification

- `shasum -a 256 -c SHA256SUMS`  
  Reads the local **SHA256SUMS** file (downloaded from Ubuntu) and recomputes the SHA256 hash of each listed file to verify it matches the official value.

- `2>&1`  
  Redirects error messages into standard output so they appear in the terminal output.

- `grep ubuntu-24.04.1-desktop-amd64.iso`  
  Filters the results to show only the verification line for your ISO file.

- `SHA256SUMS` must already exist in the same folder.  
  This is the official checksum list downloaded from Ubuntu alongside the ISO.

---

</details> <details> <summary>🪟 Windows (PowerShell)</summary>

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
### Explanation of Windows ISO Hash Verification

- `Get-FileHash`  
  Computes the SHA256 hash of your downloaded ISO file.

- `Select-String`  
  Searches the local **SHA256SUMS** file and extracts the official hash for your specific ISO filename.

- `.Split(" ")[0]`  
  The SHA256SUMS file stores the hash followed by the filename.  
  Splitting on spaces returns only the hash portion for comparison.

- `-eq` comparison  
  Checks whether the locally calculated hash exactly matches the official hash.

- Output  
  - ✅ **Hash matches — ISO verified** → file integrity confirmed  
  - ❌ **Hash mismatch — DO NOT USE** → file may be corrupted or tampered with
    
---

</details>

---

## Why Verification Matters ❓

> SHA256 creates a unique fingerprint for each file. Even a single byte difference produces a completely different hash, making it a reliable way to verify authenticity.

Verifying downloads protects against:

- Corrupted downloads

- Tampered or malicious files

- Supply-chain attacks

- Mirror/network errors

---

## Before You Go, Stay Safe!

> 🔐
> [VirusTotal](https://www.virustotal.com/gui/home/url) scans URLs, files, and hashes using multiple
> antivirus engines.
> It provides community safety reports, but does **not** guarantee 100% protection. Always exercise caution.

---

## References

1. [Ubuntu Image Verification Guide](https://ubuntu.com/tutorials/how-to-verify-ubuntu#1-overview)

2. [Ubuntu Image Verification Guide for GPG](https://ubuntu.com/tutorials/how-to-verify-ubuntu#4-retrieve-the-correct-signature-key)

3. [VirtualBox Security & Verification](https://www.virtualbox.org/manual/topics/Security.html)

4. [Linux Security](https://linuxsecurity.com/features/what-are-checksums-why-should-you-be-using-them)

5. [Generate a Hash From CLI](https://www.geeksforgeeks.org/linux-unix/generating-an-sha-256-hash-from-the-command-line/)
