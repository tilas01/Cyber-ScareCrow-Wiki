# Cyber-ScareCrow Development Guide

This guide covers the reproducible build architecture for compiling the Cyber-ScareCrow modules across Windows (Client/Driver) and Linux (Central Server).

## 1. Central Server (Linux)
The Central Authority is built in Python (Flask) for lightweight, high-speed telemetry ingestion and license verification.

### Prerequisites (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### Setup & Execution
1. Clone the repository and navigate to the `Cyber-ScareCrow-Server` directory.
2. Set up the virtual environment:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```
3. Install dependencies:
   ```bash
   pip install Flask gunicorn
   ```
4. Run in production using Gunicorn:
   ```bash
   gunicorn -w 4 -b 0.0.0.0:5000 central_authority:app
   ```

---

## 2. Windows Client (Rust)
The primary client module orchestrates the Anti-RAT screen capturing and Registry Self-Healing.

### Prerequisites (Windows)
1. Install [Rustup](https://rustup.rs/) (Ensure you install the MSVC toolchain).
2. Install [Visual Studio Build Tools 2022](https://visualstudio.microsoft.com/visual-cpp-build-tools/) with the "Desktop development with C++" workload.

### Compilation
1. Navigate to the client directory.
2. Build for release:
   ```powershell
   cargo build --release
   ```
   *Note: This will output the highly optimized binary to `target/release/cyber_scarecrow_client.exe`.*

---

## 3. Ring-0 Driver (C / WDK)
The kernel driver handles absolute process isolation and un-bypassable deep hooks.

### Prerequisites (Windows)
1. Install the [Windows Driver Kit (WDK)](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk).
2. Install the matching Windows SDK.

### Compilation & Test-Signing
1. Open the **"x64 Native Tools Command Prompt for VS 2022"** as Administrator.
2. Navigate to the driver source directory.
3. Build the driver:
   ```powershell
   msbuild /p:Configuration=Release /p:Platform=x64
   ```
4. Enable Test-Signing mode on your development machine to load the custom driver:
   ```powershell
   bcdedit /set testsigning on
   ```
   *(Requires a reboot. A watermark will appear in the bottom right corner of your screen indicating Test Mode is active).*
5. Load the driver via the Service Control Manager (SCM):
   ```powershell
   sc create ScareCrowDriver type= kernel binPath= "C:\path\to\driver.sys"
   sc start ScareCrowDriver
   ```

## Security Warning
Ensure you **never** commit `.env` files, actual Stripe/PayPal API tokens, or your master cryptographic keys to the public repositories. The CI/CD pipelines will handle injecting secrets securely via GitHub Secrets during automated builds.
