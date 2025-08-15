# VelociKAPE Toolkit

## Overview
The **VelociKAPE Toolkit** is a set of PowerShell automation scripts designed for Digital Forensics & Incident Response (DFIR) workflows using [KAPE](https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kroll-artifact-parser-extractor-kape).  
It simplifies **collection, parsing, and aggregation** of forensic artifacts across multiple systems.

Two scripts are included:
1. **VelociKAPE-Basic.ps1** – Single-server collection and parsing.
2. **VelociKAPE-Pro.ps1** – Multi-server automation with advanced features.

---

## Features
- **Automated ZIP discovery & extraction** (Basic + Pro).
- **Direct parsing from existing exports** (skip extraction).
- **Multi-server processing with auto-combination of results** (Pro).
- **Modular parsing** – Specify KAPE module sets.
- **Dry-run mode** – Preview commands before execution.
- **Customizable output paths**.
- **Server list support** – Process only specific hosts.
- **Combine control** – Enable or disable merged output.

---

## Quick Start

### Basic Script
Run full pipeline on a single system:
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Basic.ps1" `
-CollectionPath "C:\Collection" `
-ExportPath "C:\Export" `
-OutputPath "C:\Output"
```

Parse only (skip extraction):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Basic.ps1" `
-SkipExtract `
-ExportPath "C:\Export" `
-OutputPath "C:\Output"
```

---

### Pro Script – Multi-Server

Full pipeline:
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
-KapePath "C:\Tools\KAPE\kape.exe" `
-CollectionPath "C:\Collection" `
-ExportPath "C:\Export" `
-OutputPath "C:\Output" `
-CombinedPath "C:\Combined"
```

Parse-only (skip extraction):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
-KapePath "C:\Tools\KAPE\kape.exe" `
-SkipExtract `
-ExportPath "C:\Export" `
-OutputPath "C:\Output" `
-CombinedPath "C:\Combined"
```

Dry-run (no execution, preview commands):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
-KapePath "C:\Tools\KAPE\kape.exe" `
-ExportPath "C:\Export" `
-OutputPath "C:\Output" `
-CombinedPath "C:\Combined" `
-DryRun
```

---

## Advanced Usage

### Specify Servers to Process
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
-KapePath "C:\Tools\KAPE\kape.exe" `
-ExportPath "C:\Export" `
-OutputPath "C:\Output" `
-CombinedPath "C:\Combined" `
-Servers @("DC01", "WEB01")
```

### Disable Combination of Outputs
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
-KapePath "C:\Tools\KAPE\kape.exe" `
-ExportPath "C:\Export" `
-OutputPath "C:\Output" `
-CombinedPath "C:\Combined" `
-Combine:$false
```

### Enable Combination of Outputs (Default)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
-KapePath "C:\Tools\KAPE\kape.exe" `
-ExportPath "C:\Export" `
-OutputPath "C:\Output" `
-CombinedPath "C:\Combined" `
-Combine:$true
```

### Run with Custom KAPE Modules
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
-KapePath "C:\Tools\KAPE\kape.exe" `
-ExportPath "C:\Export" `
-OutputPath "C:\Output" `
-Modules "LECmd,MFTECmd" `
-CombinedPath "C:\Combined"
```

---

## Common DFIR Scenarios

### 1. Ransomware Incident – Triage
- Run Pro script with predefined triage modules.
- Skip extraction if you already have raw exports.
- Combine outputs for faster analysis.

### 2. Insider Threat Investigation
- Limit processing to suspect’s workstation(s).
- Use targeted modules (browser history, USB device logs).
- Disable combining if each analyst should get separate datasets.

### 3. Compliance Data Collection
- Run across all servers.
- Use a compliance-specific module set.
- Store combined results in an audit folder.

### 4. Large Environment Parsing
- Use `-Servers` to batch process in subsets.
- Run multiple instances of the script in parallel on different sets.

---

## Recommendations
- Always **dry-run** in large environments before executing live to confirm targets.
- Keep **KAPE** updated for the latest parsing modules.
- Store combined CSVs in a central location for easy ingestion into ELK/Splunk.
- Validate **path structures** (ensure `uploads` directories match your export layout).

---

## License
This project is distributed under the MIT License.
