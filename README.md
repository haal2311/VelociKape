# VelociKAPE-Pro & Mass-VelociKAPE

Automated PowerShell pipelines for forensic evidence processing with [Eric Zimmerman's KAPE](https://ericzimmerman.github.io/).  
These scripts help extract, parse, and combine results from multiple systems efficiently.

---

## 📌 Overview

### VelociKAPE-Pro.ps1
A **single-server or multi-server** forensic pipeline:
1. **Extracts** artifacts from ZIPs (optional).
2. **Parses** them with KAPE using selected modules.
3. **Combines** CSV outputs into unified files per artifact type.

### Mass-VelociKAPE.ps1
Batch execution of VelociKAPE-Pro **across multiple servers** automatically.

---

## 🚀 Key Features

- **Automatic discovery** of servers from ExportPath.
- **Module profiles** with sensible defaults.
- **Custom module selection** support.
- **Combining CSV outputs** by artifact type across servers.
- **Dry-run mode** for testing commands without execution.
- **SysInternals_SigCheck excluded by default** (high performance cost).

---

## ⚠️ Recommendations

- **SysInternals_SigCheck** is expensive and should be run **only after** all other parsing is complete.
- Use [TimelineExplorer](https://ericzimmerman.github.io/) for advanced timeline analysis of CSV output.
- Keep KAPE and all modules up to date.
- Run from a fast SSD to improve parsing speed.

---

## 🛠 Default Modules (Triage Profile)

By default, the scripts run the following modules (excluding SysInternals_SigCheck):

```
!EZParser,
BMC-Tools_RDPBitmapCacheParser,
BitsParser,
Hayabusa,
WMI-Parser,
LogParser,
RECmd_AllBatchFiles,
PowerShell_ParseScheduledTasks
```

You can override these with `-Modules`.

---

## ⚡ Quick Start

### Full pipeline (extract + parse + combine)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -CollectionPath "C:\Collection" `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined"
```

### Parse-only (skip extraction)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -SkipExtract $true `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined"
```

### Dry-run (no execution, just shows planned commands)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -SkipExtract $true `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined" `
  -DryRun $true
```

---

## 🔄 Mass-VelociKAPE

### Process all servers in bulk
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\Mass-VelociKAPE.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -ExportBase "C:\Export" `
  -OutputBase "C:\Output" `
  -CombinedBase "C:\Combined"
```

---

## 📜 Parameters

### VelociKAPE-Pro.ps1
| Parameter | Description | Default |
|-----------|-------------|---------|
| `-KapePath` | Path to `kape.exe`. | **Required** |
| `-CollectionPath` | Path to extracted ZIP contents. | None |
| `-ExportPath` | Path to extracted artifacts. | **Required** |
| `-OutputPath` | Directory to store parsed CSVs. | **Required** |
| `-CombinedPath` | Directory for combined CSV outputs. | **Required** |
| `-Modules` | Comma-separated list of modules. | Default triage list |
| `-SkipExtract` | Skip the extraction step. | `$false` |
| `-DryRun` | Show commands without running. | `$false` |
| `-Combine` | Combine CSV outputs. | `$true` |

### Mass-VelociKAPE.ps1
| Parameter | Description |
|-----------|-------------|
| `-KapePath` | Path to `kape.exe`. |
| `-ExportBase` | Base path where each server’s export folder is located. |
| `-OutputBase` | Base path for parsed output per server. |
| `-CombinedBase` | Base path for combined CSVs. |

---

## 🧪 Examples

**1. Run defaults, skip extraction:**
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -SkipExtract $true `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined"
```

**2. Custom modules only:**
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -Modules "RECmd_Batch_InstalledSoftware,PrefetchParser" `
  -SkipExtract $true `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined"
```

**3. Parse only and no combine:**
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -SkipExtract $true `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined" `
  -Combine $false
```

---

## 📌 Notes
- Output directories are per-server. Running again with additional modules will store new results in the same server folder without overwriting unrelated results.
- Combining will merge outputs **by artifact type**, ignoring run timestamps, so `combined_RECmd_Batch_InstalledSoftware.csv` contains **all** matching outputs.
- SysInternals_SigCheck is excluded from defaults because it’s **slow** and should be run separately.
