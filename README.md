# VelociKAPE

**VelociKAPE** streamlines enterprise-scale DFIR by parsing **Velociraptor offline collections** with **KAPE** and merging results into ready-to-analyze CSVs.

This repo includes two scripts:

- **VelociKAPE-Pro.ps1** — feature-rich: parallel runs, resume mode, skip-extract, profiles, combine-only, normalized combining, optional 7-Zip extraction, pruning.
- **Mass-VelociKAPE.ps1** — minimal runner with the same normalized combining logic.

Both scripts:
- Discover Velociraptor ZIPs (`Collection-<Server>-YYYY-MM-DDThh_mm_ssZ.zip`) **or** work from pre-extracted folders.
- Run KAPE modules per server into `C:\Output\<Server>\…`.
- **Combine** per-artifact CSVs across all servers into `C:\Combined\combined_<Artifact>.csv`, tagging rows with `ServerName` and `SourcePath`.

> **Why this matters (DFIR):**  
> You avoid repetitive, manual parsing and get **single consolidated views** per artifact (Amcache, MFTECmd, EVTX parsers, RECmd, etc.). These drop straight into Excel, SIEM, or **Timeline Explorer (Eric Zimmerman)** for fast pivots.

---

## Requirements

- Windows 10/11 or Windows Server 2016+
- [KAPE](https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kroll-artifact-parser-extractor-kape) executable
- Velociraptor offline collections (ZIPs) or pre-extracted collections
- PowerShell 5.1 (works) or **PowerShell 7+** (recommended for parallel)
- **AV exclusions recommended** for maximum speed and fewer quarantines:
  - `kape.exe` folder
  - `C:\Export`, `C:\Output`, `C:\Combined`

---

## Default Modules (SigCheck excluded)

```text
!EZParser,BMC-Tools_RDPBitmapCacheParser,BitsParser,Hayabusa,WMI-Parser,LogParser,RECmd_AllBatchFiles,PowerShell_ParseScheduledTasks
```

> ⚠️ **SysInternals_SigCheck is resource-heavy.**  
> Run it **after** your main parsing as a separate pass (see examples below). This keeps the primary pipeline fast.

---

## Quick Start (Pro script)

### Full pipeline  
Discovers ZIPs → extracts → parses with KAPE → auto-combines (if >1 server).

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -CollectionPath "C:\Collection" `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined"
```

### Parse-only (Skip extraction)
Use this if you already extracted the collections to per-server folders under `C:\Export\<Server>`.

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" -SkipExtract:$true `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Combine:$false -DryRun:$true
```

### Combine-only (Later/after parsing completes)
Build `combined_*.csv` from existing per-server outputs.

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined" `
  -CombineOnly:$true
```

### Parallel servers (fast on PS7+)
Run multiple servers at once (watch CPU/IO; start with 2–3).

```powershell
pwsh -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -CollectionPath "C:\Collection" `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Parallel 3
```

### Target specific servers
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -CollectionPath "C:\Collection" `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Servers "DC01","WEB03"
```

### Run **only** SysInternals_SigCheck (second pass)
Recommended after the main modules finish.

```powershell
# Parse only with SigCheck
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" -SkipExtract:$true `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Modules "SysInternals_SigCheck"

# Combine afterwards
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -OutputPath "C:\Output" -CombinedPath "C:\Combined" -CombineOnly:$true
```

---

## Quick Start (Mass script)

The lightweight runner with normalized combining.

### Full pipeline
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\Mass-VelociKAPE.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -CollectionPath "C:\Collection" `
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined"
```

### Combine-only
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\Mass-VelociKAPE.ps1" `
  -OutputPath "C:\Output" -CombinedPath "C:\Combined" -CombineOnly:$true
```

---

## Normalized Combining (Important)

- Per-server outputs look like: `C:\Output\<Server>\YYYYMMDDhhmmss_<Artifact>_Output.csv`
- Combined outputs look like: `C:\Combined\combined_<Artifact>.csv`

**Normalization rules** (so you get one file per artifact):
- Leading timestamps like `20250811153133_` or `2025-08-11T10_24_42Z_` are ignored
- A trailing `_Output` suffix is stripped
- Each row is tagged with **ServerName** and **SourcePath**

Examples of single combined files created:
- `combined_Amcache.csv`
- `combined_RECmd_Batch_InstalledSoftware.csv`
- `combined_EvtxECmd.csv`
- `combined_MFTECmd_$MFT.csv`
- `combined_Hayabusa.csv`

---

## Resume, Prune & Re-Runs

- `-Resume` — skip servers already parsed for the **same module set** (per-server marker files).
- `-ForceReparse` — ignore markers; re-run anyway.
- `-PruneOutput` — delete existing per-server CSVs before parsing (clean slate).
- `-PruneCombined` — delete `combined_*.csv` before combining.

> **Incremental runs supported:**  
> You can run defaults first, then run **only** a new module later (e.g., `SysInternals_SigCheck`) and outputs will land in the **same server folder**. Combine will merge everything by artifact name.

---

## Best Practices (DFIR at scale)

- **Skip extraction** (`-SkipExtract:$true`) when you already have exports — unzipping is usually the slowest step.
- Use **PowerShell 7** and `-Parallel` (2–3 to start); monitor disk/CPU/AV.
- Put **KAPE, Export, Output, Combined** on the **same SSD**.
- Add **AV exclusions** for these paths and `kape.exe`.
- **Run SigCheck last** (separate pass). It’s **CPU/IO-heavy** and can slow the main pipeline.
- Combine **after** multiple servers finish (or run `-CombineOnly:$true` at the end).
- Use **Timeline Explorer (Eric Zimmerman)** to quickly filter and pivot the large `combined_*.csv` results by `ServerName`, `ImagePath`, `CommandLine`, timestamps, etc.

---

## Troubleshooting

- **Script is blocked:**  
  ```powershell
  powershell -NoProfile -ExecutionPolicy Bypass -File .\VelociKAPE-Pro.ps1 -Help
  ```
- **ZIP names don’t match pattern:** Ensure `Collection-<Server>-YYYY-MM-DDThh_mm_ssZ.zip`.
- **No results combined:** Confirm per-server CSVs exist and are **not** under `\raw\` subfolders (these are ignored).
- **Slow parsing:** Add AV exclusions; use `-SkipExtract:$true`; enable `-Parallel` on PS7; run SigCheck later.

---

## License

MIT — use at your own risk. Contributions welcome.
