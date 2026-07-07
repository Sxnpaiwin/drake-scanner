# Drake Scanner

Minecraft cheat-client detector — scans running `javaw.exe` / `java.exe` process memory and the DNS cache for signatures of ~388 known cheat clients.

**This is the obfuscated public build.** The 388 client signatures and 407 generic module flags are GZip-compressed + Base64-encoded into a single blob, and the C# scanner engine is Base64-encoded. This makes it significantly harder for cheat developers to extract the signature list and develop bypasses.

## Download

**[DrakeScanner.ps1](https://github.com/Sxnpaiwin/drake-scanner/releases/latest/download/DrakeScanner.ps1)** (~50 KB)

## Quick start

**Option 1 — One-liner (easiest):**

Copy and paste this into an elevated PowerShell window (right-click PowerShell → Run as administrator):

```powershell
& ([scriptblock]::Create((iwr "https://github.com/Sxnpaiwin/drake-scanner/releases/latest/download/DrakeScanner.ps1" -UseBasicParsing).Content))
```

**Option 2 — Download then run:**

```powershell
# Download
iwr "https://github.com/Sxnpaiwin/drake-scanner/releases/latest/download/DrakeScanner.ps1" -UseBasicParsing -OutFile DrakeScanner.ps1

# Unblock (Windows marks downloaded scripts as untrusted)
Unblock-File .\DrakeScanner.ps1

# Run as administrator
Start-Process powershell -Verb RunAs -ArgumentList "-NoExit -ExecutionPolicy Bypass -File .\DrakeScanner.ps1"
```

**Option 3 — Right-click:**

Download `DrakeScanner.ps1`, right-click → **Run with PowerShell** (then accept the UAC prompt).

## Parameters

```powershell
.\DrakeScanner.ps1                    # Full scan
.\DrakeScanner.ps1 -JournalTrace      # Scan + launch JournalTrace after
.\DrakeScanner.ps1 -NoScan -JournalTrace  # Skip scan, just launch JournalTrace
```

## Requirements

- Windows 10 or later
- PowerShell 5.1+ (built into Windows 10) or PowerShell 7+ (pwsh)
- No admin privileges required (works without elevation)
- `Set-ExecutionPolicy` may need to be `RemoteSigned` or `Bypass` to run .ps1 files

## How it works

1. Checks for admin privileges
2. Decodes embedded data (GZip + Base64) containing 388 client signatures + 407 generic module flags
3. Decodes embedded C# scanner engine (Base64 UTF-16LE) and loads it via `Add-Type`
4. The C# engine uses P/Invoke for `OpenProcess`, `ReadProcessMemory`, `VirtualQueryEx` + Aho-Corasick multi-pattern matcher
5. Finds `javaw.exe` / `java.exe` processes
6. Scans DNS cache via `ipconfig /displaydns`
7. Scans each process's readable memory for all signatures
8. Prints results in colored console output
9. Optional: `-JournalTrace` flag downloads and launches JournalTrace.exe

## Output

```
  +-------------------------------------------------+
  |         ____                       _              |
  |        |  _ \ __ _  ___ __ _ _ __ __| |             |
  |        | | | |/ _` |/ __/ _` | '__/ _` |            |
  |        | |_| | (_| | (_| (_| | | | (_| |            |
  |        |____/ \__,_|\___\__,_|_|  \__,_|            |
  |                                                 |
  |         Minecraft Cheat-Client Detector         |
  |              v1.0.0 - obfuscated                |
  +-------------------------------------------------+

  [*] Initializing... done

  === SCAN ===

  [*] Found 1 Minecraft process(es): 12345

  [DNS Cache Scan]
    [!] Vape Client (x3)

  [Memory Scan]
    Scanning PID 12345... 100%

  === CHEAT CLIENTS DETECTED ===
    [!] Vape Client (x8)
    [!] Prestige Client (x3)

  === GENERIC MODULE FLAGS ===
    [~] Kill Aura (x5)
    [~] AutoTotem (x12)

  >>> CHEATING DETECTED (31 matches) <<<

  [*] Done. Press ENTER to exit...
```

## Attribution

Original scanner logic and 388 client signature database (c) Tonynoh (https://github.com/MeowTonynoh).
PowerShell port + obfuscation by Sxnpaiwin.
