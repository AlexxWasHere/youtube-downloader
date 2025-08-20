# Spectra - YouTube Downloader (Open Source)

**Short description**

Spectra is a lightweight Python-based YouTube video & playlist downloader that uses `yt_dlp`. It is distributed as source code and as a pre-built Windows `.exe`. This repository focuses on transparency and reproducible builds so you (and security-conscious users) can verify the binary.

---

# Table of contents

1. Installation & running (recommended: from source)
2. Build instructions (reproducible `.exe` using PyInstaller)
3. Verify the release (SHA-256 checksum + how to compute it)
4. Why some antivirus engines may flag the `.exe` (false positives)
5. Submitting false-positive reports & VirusTotal guidance
6. Security & privacy notes (legal disclaimer)
7. Contributing
8. License

---

## 1) Installation & run (recommended: run from source)

**Why run from source?** Rebuilding from the provided source code proves the distributed `.exe` contains only the code you can read. This is the most transparent option.

### Steps (Windows / Linux / macOS)

```bash
# 1. Clone the repo
git clone https://github.com/AlexxWasHere/youtube-downloader
cd AlexxWasHere

# 2. Create a virtualenv (recommended)
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt
pip install -r dev-requirements.txt

# 4. Run
python main.py
```

Replace `main.py` with the actual entrypoint if different (e.g. `spectra.py`).

---

## 2) Build a reproducible `.exe` (PyInstaller)

We recommend building the `.exe` yourself instead of downloading the prebuilt binary.

### Example build steps (Windows, PowerShell)

```powershell
# Activate virtualenv first
.venv\Scripts\activate

# Ensure you have the same versions used to produce the release
pip install -r requirements.txt
pip freeze > requirements.lock
pip install -r dev-requirements.txt
pip freeze > dev-requirements.lock

# Clean build directory and build one-file exe
pyinstaller --onefile --clean --name SpectraDownloader --noconfirm main.py

# After building, the executable will be in `dist\SpectraDownloader.exe`
```

**Notes & tips**
- Do **not** use UPX compression if you want to reduce false positives.
- Add a `--version-file` metadata file for proper product/version info (reduces SmartScreen warnings).
- If you plan to distribute widely, consider purchasing a Windows code-signing certificate and signing the binary (see section below).

---

## 3) Verify the release (checksum)

A checksum is a cryptographic fingerprint of a file. We publish the SHA-256 for each release so users can verify the file they downloaded is *exactly* the one we produced.

### Example published checksum (this repo)

```
SHA-256: 81f1a05eeb85d24f1b3fc6b24e6b694bbd8703f5708497d3d4b27f23e91942fc
```

### How to compute SHA-256 locally

- **Windows (PowerShell)**
```powershell
Get-FileHash .\SpectraDownloader.exe -Algorithm SHA256
```

- **Windows (cmd)**
```cmd
certutil -hashfile SpectraDownloader.exe SHA256
```

- **macOS / Linux**
```bash
sha256sum SpectraDownloader.exe
# or
shasum -a 256 SpectraDownloader.exe
```

If the computed SHA-256 matches the published checksum, the file is byte-for-byte identical.

---

## 4) Why a few antivirus engines may flag the `.exe` (false positives)

If you uploaded the `.exe` to VirusTotal you may see a handful of detections from some engines (ML/heuristic detections like `BehavesLike.Win64.Generic` or `W64.AIDetectMalware`). This is a common and expected outcome for many Python apps packaged into a single-file executable (PyInstaller / cx_Freeze), because:

- The binary bundles an embedded Python interpreter, libraries, and compressed code — this makes the file large and "unusual" compared to standard compiled native apps.
- Many antivirus vendors rely on machine-learning heuristics that flag packed or uncommon binary layouts.
- The executable may lack a code-signing certificate and product metadata, which increases suspicion by SmartScreen/heuristic detectors.

**Important:** A small number of detections does **not** automatically mean the software contains malware. The source code is provided so anyone can inspect and rebuild the binary themselves.

---

## 5) Submitting false-positive reports & VirusTotal guidance

If you or users encounter false positives:

1. Rebuild the exe locally and verify the checksum.
2. Submit the binary to the specific antivirus vendor through their false-positive submission page (links available on their site). Include a short explanation and a link to this repo.
3. Optionally upload the binary to VirusTotal and use the "Request reanalysis" / vendor submission features there (or copy the VirusTotal URL into the vendor form).

We are happy to help submit vendor reports — open an issue with the detection details and we can provide reproducible steps and the exact build metadata.

---

## 6) Digital signing (optional, recommended for broad distribution)

Code-signing the executable with a trusted certificate prevents many warnings (SmartScreen and some AV heuristics) and helps users trust the publisher.

- **Windows**: use a code signing (EV) certificate and `signtool.exe` to sign the executable.
- **Alternate**: PGP-sign the published SHA-256 checksum so users can verify you (the publisher) published that checksum.

If you want guidance on signing, open an issue and we’ll add step-by-step instructions for your platform.

---

## 7) Security & privacy notes / legal disclaimer

- **Use only for content you own or are permitted to download.** Do not use this tool to violate YouTube’s Terms of Service or local law.
- **Privacy:** this project does **not** phone home. Logs are saved locally to `spectra_downloader.log` for troubleshooting. We don’t collect or transmit user data.
- **No warranty:** use at your own risk. The project is provided "as-is".

---

## 8) Contributing

Contributions welcome. Please:

1. Fork the repo.
2. Create a branch for your change.
3. Open a pull request with a clear description and test steps.

If you find an AV false positive or want help producing a signed release, open an issue and label it `security`.

---

## 9) License

This project is released under the MIT License. See `LICENSE` for details.

---
