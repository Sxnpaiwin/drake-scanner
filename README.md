# Drake Scanner

Minecraft cheat-client detector — scans running `javaw.exe` / `java.exe` process memory and the DNS cache for signatures of ~388 known cheat clients.

## Downloads

| File | Size | Description |
|---|---|---|
| **[DrakeScanner.ps1](https://github.com/Sxnpaiwin/drake-scanner/releases/latest/download/DrakeScanner.ps1)** | ~50 KB | **Public/obfuscated build** — signatures compressed + encoded, harder to reverse-engineer for bypass development |
| **[OrbitalScanner.ps1](https://github.com/Sxnpaiwin/drake-scanner/releases/latest/download/OrbitalScanner.ps1)** | ~100 KB | Source/audit build — plaintext signatures, fully readable |

## Quick start

```powershell
# Option 1: Download and run
iwr "https://github.com/Sxnpaiwin/drake-scanner/releases/latest/download/DrakeScanner.ps1" | iex

# Option 2: Right-click the .ps1 -> Run with PowerShell (accept UAC)

# Option 3: From elevated PowerShell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\DrakeScanner.ps1
```

## Parameters

```powershell
.\DrakeScanner.ps1                    # Full scan
.\DrakeScanner.ps1 -JournalTrace      # Scan + launch JournalTrace after
.\DrakeScanner.ps1 -NoScan -JournalTrace  # Skip scan, just launch JournalTrace
```

## Why two versions?

**DrakeScanner.ps1** (obfuscated) is for public distribution. The 388 client signatures and 407 generic module flags are GZip-compressed + Base64-encoded into a single blob, and the C# scanner engine is Base64-encoded. This makes it significantly harder for cheat developers to extract the signature list and develop bypasses.

**OrbitalScanner.ps1** (plaintext) is for auditing. You can open it in any text editor and read the full source, verify what the scanner does, and confirm it's not doing anything malicious. Use this if you want to inspect the code before running.

Both versions use the same scanner engine and detect the same clients.

## Requirements

- Windows 10 or later
- PowerShell 5.1+ (built into Windows 10) or PowerShell 7+ (pwsh)
- Administrator privileges recommended (script will prompt to continue without if not elevated)
- `Set-ExecutionPolicy` may need to be `RemoteSigned` or `Bypass` to run .ps1 files

## How it works

1. Checks for admin privileges
2. Loads a C# helper class via `Add-Type` (P/Invoke for `OpenProcess`, `ReadProcessMemory`, `VirtualQueryEx` + Aho-Corasick multi-pattern matcher)
3. Finds `javaw.exe` / `java.exe` processes
4. Scans DNS cache via `ipconfig /displaydns`
5. Scans each process's readable memory for 388 cheat-client signatures + 407 generic module flags
6. Prints results in colored console output
7. Optional: `-JournalTrace` flag downloads and launches JournalTrace.exe

The scanner uses the Aho-Corasick algorithm to search for all ~1,200 patterns (388 clients × UTF-8 + UTF-16 encodings + 407 generic flags) in a single pass through memory.

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

  >>> CHEATING DETECTED (14 matches) <<<

  [*] Done. Press any key to exit...
```

## Attribution

Original scanner logic and 388 client signature database (c) Tonynoh (https://github.com/MeowTonynoh).
PowerShell port + obfuscation by Sxnpaiwin.
