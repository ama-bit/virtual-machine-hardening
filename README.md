# Virtual Machine Hardening 🛡️ - Currently In Progress... ⚠️

## Step 1: Verify Downloads 🚩

Before installing anything, like **VirtualBox** or **Ubuntu**, always verify the **integrity** and **authenticity** of your files. This ensures the software hasn’t been tampered with or corrupted. Verification is a core cybersecurity best practice recommended by Ubuntu, VirtualBox, NIST, and SANS.

---

> 💡 Note: When I first tried to find the SHA256 checksum for Ubuntu, it wasn’t immediately visible on the release page like it had been in the past. I had to click on the download link, which brought me to the page where the checksum was located.
>
> Keep in mind that Ubuntu may change the location of checksums again in the future. Always make sure you are obtaining the checksum from the [**official Ubuntu site**](https://ubuntu.com/download).

---

## Ubuntu ISO Verification

Ubuntu publishes official SHA256 checksums for all release images. Always download files from the **official Ubuntu releases page**:

- [Ubuntu Releases](https://releases.ubuntu.com/) – Look for `SHA256SUMS` and optionally `SHA256SUMS.gpg` for signature verification.

>
> ⚠️ Replace `ubuntu-XX.XX-desktop-amd64.iso` with your downloaded filename from the official page.

### Linux / MacOS

```bash

# Navigate to your download directory

cd ~/Downloads

# Linux: generate SHA256 hash

sha256sum ubuntu-XX.XX-desktop-amd64.iso

# macOS: generate SHA256 hash

shasum -a 256 ubuntu-XX.XX-desktop-amd64.iso

```

---

>
>🗒️ Compare the output to the official SHA256 checksum on the Ubuntu website.
> They must match exactly before you proceed. If the hashes do not match,
> re-download the ISO from the official page. Continue to the
> **Compare ISO Hash to Official Checksum** section below to learn how.

---

### Windows (PowerShell)

```powershell
# Navigate to your download directory

cd C:\Users\YourName\Downloads

# Generate SHA256 hash

Get-FileHash .\ubuntu-XX.XX-desktop-amd64.iso -Algorithm SHA256

```

**Optional: GPG Signature Verification**
Ubuntu also provides a GPG-signed checksum file (SHA256SUMS.gpg) for additional verification:

[Ubuntu Image Verification Guide](https://ubuntu.com/tutorials/how-to-verify-ubuntu#4-retrieve-the-correct-signature-key)

This adds cryptographic assurance that the file is authentic and has not been tampered with.

---

## Step 2: VirtualBox Installer Verification

VirtualBox publishes SHA256 checksums for all installers on their official downloads page.

⚠️ Replace `VirtualBox-6.X.X-xxxxxx-Linux.run`, `shasum -a 256 VirtualBox-6.X.X-xxxxxx-OSX.dmg`,
or `.\VirtualBox-6.X.X-xxxxxx-Win.exe` with the filename you downloaded from the official page.

### VirtualBox Downloads

### Linux / MacOS

```bash
# Navigate to your download directory
cd ~/Downloads

# Linux: generate SHA256 hash for VirtualBox installer
sha256sum VirtualBox-6.X.X-xxxxxx-Linux.run

# macOS: generate SHA256 hash for VirtualBox installer
shasum -a 256 VirtualBox-6.X.X-xxxxxx-OSX.dmg
```

---

> ⚠️ Compare the hash to the value listed on the VirtualBox site.
> Continue to the **Compare ISO Hash to Official Checksum** section
> below to learn how.

---

### Windows (PowerShell)
```powershell
# Navigate to your download directory

cd C:\Users\YourName\Downloads

# Generate SHA256 hash

Get-FileHash .\VirtualBox-6.X.X-xxxxxx-Win.exe -Algorithm SHA256
```

---

> 💡 On Windows, installers are also digitally signed.
> A valid OS signature provides extra assurance but
> does not replace hash verification.

---

## Step 3: Compare ISO Hash to Official Checksum

These commands verify that your downloaded ISO matches the official SHA256 checksum provided by the distributor.  

You must have both the ISO file and the official `SHA256SUMS` file in the same folder.

---

### Windows (PowerShell) 🪟
```powershell
    # Go to your Downloads folder
    cd $env:USERPROFILE\Downloads

    # Calculate SHA256 hash of the ISO file
    $isoHash = (Get-FileHash .\ubuntu-24.04.1-desktop-amd64.iso -Algorithm SHA256).Hash

    # Read the official hash for this ISO from SHA256SUMS
    $officialHash = (Select-String "ubuntu-24.04.1-desktop-amd64.iso" .\SHA256SUMS).ToString().Split(" ")[0]

    # Compare the two hashes
    if ($isoHash -eq $officialHash) {
        Write-Host "✔ Hash matches — ISO verified"
    } else {
        Write-Host "✖ Hash mismatch — DO NOT USE"
    }
```
---

### macOS 🍏
```powershell
    # Go to your Downloads folder
    cd ~/Downloads

    # Verify the ISO against the official SHA256SUMS file
    shasum -a 256 -c SHA256SUMS 2>&1 | grep ubuntu-24.04.1-desktop-amd64.iso

    # Expected output if valid:
    # ubuntu-24.04.1-desktop-amd64.iso: OK
```
---

## Linux 💻
```bash
    # Go to your Downloads folder
    cd ~/Downloads

    # Verify the ISO against the official SHA256SUMS file
    sha256sum -c SHA256SUMS 2>&1 | grep ubuntu-24.04.1-desktop-amd64.iso

    # Expected output if valid:
    # ubuntu-24.04.1-desktop-amd64.iso: OK
```
---

## Manual Hash Comparison (Alternative)

If you prefer to compare hashes visually instead of automatic verification.

## Windows
```powershell 🪟
    # Compute SHA256 hash of ISO
    Get-FileHash .\ubuntu-24.04.1-desktop-amd64.iso -Algorithm SHA256

    # Manually compare the displayed hash
    # with the value in SHA256SUMS
```
## macOS 🍏
```powershell
    # Compute SHA256 hash of ISO
    shasum -a 256 ubuntu-24.04.1-desktop-amd64.iso

    # Manually compare with SHA256SUMS
```
## Linux 💻
```powershell
    # Compute SHA256 hash of ISO
    sha256sum ubuntu-24.04.1-desktop-amd64.iso

    # Manually compare with SHA256SUMS
```
---

> ⚠️ The ISO is authentic only if the computed SHA256 hash exactly matches  
> the official value in the `SHA256SUMS` file.
>
> Any difference indicates corruption or tampering — DO NOT USE the file.

---

## Why This Matters

Verifying downloads protects you from:

- Corrupted or incomplete downloads

- Tampered files or supply-chain attacks

- File integrity issues introduced by mirrors or network errors

Completing this step before installing any virtual machines or system software sets the foundation for safe VM hardening.

> 🔐 **Tip:** SHA256 creates a unique fingerprint for each file. Even a single byte difference produces a completely different hash, making it a reliable way to verify authenticity.

---

## References

1. [Ubuntu Image Verification Guide](https://ubuntu.com/tutorials/how-to-verify-ubuntu#1-overview)

2. [Ubuntu Image Verification Guide for GPG](https://ubuntu.com/tutorials/how-to-verify-ubuntu#4-retrieve-the-correct-signature-key)

3. [VirtualBox Security & Verification](https://www.virtualbox.org/manual/topics/Security.html)

4. [Linux Security](https://linuxsecurity.com/features/what-are-checksums-why-should-you-be-using-them)

5. [Generate a Hash From CLI](https://www.geeksforgeeks.org/linux-unix/generating-an-sha-256-hash-from-the-command-line/)
