# Cyber-ScareCrow Development Guide

This guide covers the reproducible build architecture for compiling the Cyber-ScareCrow modules natively on Windows.

## 1. Windows Client (Rust)
The primary client module orchestrates the Anti-RAT screen capturing and Registry Self-Healing.

### Prerequisites
1. Install [Rustup](https://rustup.rs/) (Ensure you install the MSVC toolchain).
2. Install [Visual Studio Build Tools 2022](https://visualstudio.microsoft.com/visual-cpp-build-tools/) with the "Desktop development with C++" workload.

### Compilation
1. Navigate to the client directory.
2. Build for release:
   ```powershell
   cargo build --release
   ```
   *Note: This will output the highly optimized binary to `target/release/cyber-scarecrow.exe`.*

---

## 2. Ring-0 Driver (C / WDK)
The kernel driver handles absolute process isolation and un-bypassable deep hooks.

### Prerequisites
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
   sc create ScareCrowDriver type= kernel binPath= "C:\\path\\to\\driver.sys"
   sc start ScareCrowDriver
   ```
