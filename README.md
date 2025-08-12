# VelociKAPE

Two PowerShell scripts to **parse Velociraptor offline collections with KAPE** and produce **combined, enterprise-wide CSVs** ready for analysis.

- `VelociKAPE-Pro.ps1` — feature-rich, parallel, resume-safe, optional 7-Zip extraction.
- `Mass-VelociKAPE.ps1` — lightweight runner (simple, dependable).

Both scripts:
- Discover ZIPs (`Collection-<Server>-YYYY-MM-DDThh_mm_ssZ.zip`) or work from pre-exported folders.
- Run KAPE modules per server and write outputs under `C:\Output\<Server>\…`.
- **Combine** all per-artifact CSVs across hosts into a **single** `combined_<Artifact>.csv` (normalized: timestamps/log prefixes removed).
- Tag each row with `ServerName` and `SourcePath`.

> **Why this matters (DFIR)**  
> Automates repetitive parsing + merges results, so responders can pivot by host, time, or artifact instantly in Excel, SIEM, or **Timeline Explorer (Eric Zimmerman)**.

---

## Requirements

- Windows 10/11 or Server 2016+
- [KAPE](https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kroll-artifact-parser-extractor-kape) executable
- Velociraptor offline collections in ZIP format
- PowerShell 5.1 (works) or PowerShell 7+ (recommended for parallel)
- **Strongly recommended AV exclusions** for:
  - `kape.exe` folder
  - `C:\Export`, `C:\Output`, `C:\Combined`

---

## Default Modules (SigCheck excluded)

!EZParser,BMC-Tools_RDPBitmapCacheParser,BitsParser,Hayabusa,WMI-Parser,LogParser,RECmd_AllBatchFiles,PowerShell_ParseScheduledTasks


- **SysInternals_SigCheck** is **CPU/IO-heavy**.  
  **Recommendation:** run your main modules first, then re-run with:
-SkipExtract -Modules "SysInternals_SigCheck"

and combine afterward.

---

## Quick Start (Pro script)

### Full pipeline
Discovers ZIPs → extracts → parses with KAPE → auto-combines (if >1 server).
```powershell`
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
-KapePath "C:\Tools\KAPE\kape.exe" `
-CollectionPath "C:\Collection" `
-ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined"

### Parse-only (skip extraction)

powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" -SkipExtract `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Combine:$false

Combine-only (later)
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -OutputPath "C:\Output" -CombinedPath "C:\Combined" -CombineOnly

Parallel servers (PS7 recommended)

pwsh -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" -CollectionPath "C:\Collection" `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Parallel 3

Run only SigCheck later (incremental)
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" -SkipExtract `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Modules "SysInternals_SigCheck"
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -OutputPath "C:\Output" -CombinedPath "C:\Combined" -CombineOnly

Target specific servers
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" -CollectionPath "C:\Collection" `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Servers "DC01","WEB03"

Resume / Force re-parse / Pruning
-Resume → skip servers already parsed for this module set (per-server marker files).

-ForceReparse → ignore markers and re-run anyway.

-PruneOutput → delete existing CSVs in each server’s output folder before parsing.

-PruneCombined → delete existing combined_*.csv before combine.

Module Profiles (Pro script)
Use -ModulesProfile instead of -Modules to pick from baked-in sets:

Triage (default)
!EZParser,BMC-Tools_RDPBitmapCacheParser,BitsParser,Hayabusa,WMI-Parser,LogParser,RECmd_AllBatchFiles,PowerShell_ParseScheduledTasks

Full
Triage + EvtxECmd + LECmd (heavier; adjust to taste)

SignatureOnly
SysInternals_SigCheck

You can still override anytime with -Modules or -ModuleListFile.


Output & Combining
Per server: C:\Output\<Server>\YYYYMMDDhhmmss_Artifact_Output.csv

Combined: C:\Combined\combined_<Artifact>.csv
(artifact key normalized: timestamp prefixes removed; _Output suffix removed)

Each row gains:

ServerName (folder name under Output)

SourcePath (original per-server CSV)

Performance Tips
Keep KAPE, Export, Output, Combined on the same SSD.

Use PowerShell 7 and -Parallel for multi-host speed-ups.

SkipExtract when you already have exports—unzipping is often the slowest step.

Exclude paths from AV (and kape.exe) to avoid scanning overhead/quarantines.

Analysis Tips
Open combined_*.csv in Timeline Explorer (Eric Zimmerman) for fast filtering & pivots by time, host, and artifact.

Filter by ServerName, ImagePath, CommandLine, User, etc., then export filtered CSVs for sharing.

License
MIT — use at your own risk. Contributions welcome
