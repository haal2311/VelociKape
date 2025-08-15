# VelociKAPE-Pro & Mass-VelociKAPE

PowerShell automation for DFIR analysts using [KAPE](https://www.kroll.com/en/insights/publications/cyber/kroll-artifact-parser-extractor-kape) to:
- Parse collected forensic artifacts at scale (single or multiple servers)
- Automate extraction → parsing → CSV export → combined analysis
- Handle custom modules, profiles, and batch parsing
- Reduce repetitive DFIR processing steps

This toolkit contains:
1. **VelociKAPE-Pro.ps1** – Flexible parsing pipeline for single/multi-host exports
2. **Mass-VelociKAPE.ps1** – Bulk orchestration across many servers

---

## 🛠 Setup

1. Install **KAPE**:
   - Download from: https://www.kroll.com/en/insights/publications/cyber/kroll-artifact-parser-extractor-kape
   - Place in `C:\Tools\KAPE` (or your preferred directory)

2. Ensure PowerShell execution policy allows script execution:
   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force
   ```

3. Clone or download this repo containing:
   - `VelociKAPE-Pro.ps1`
   - `Mass-VelociKAPE.ps1`
   - This README

4. Prepare your input folders:
   - **ExportPath**: Directory where collected artifacts are stored (`serverX\uploads`)
   - **OutputPath**: Per-server parsed CSV outputs
   - **CombinedPath**: Optional merged CSVs for multi-host analysis

---

## ⚙ Default Modules

- Default module list **excludes** `SysInternals_SigCheck` for performance reasons.
- `SysInternals_SigCheck` should be run **last**, after initial parsing is complete, due to high CPU and I/O overhead.
- Use `-Modules` to specify your own list or add `SysInternals_SigCheck` later.

---

## 🚀 Quick Start Examples

### 1. Full Parse (Extraction + Parsing)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -CollectionPath "C:\Collection" `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined"
```

### 2. Parse Only (Skip Extraction)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -SkipExtract `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -CombinedPath "C:\Combined"
```

### 3. Dry Run (Show commands without executing)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -SkipExtract `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -DryRun
```

### 4. Run with Custom Modules
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -SkipExtract `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -Modules "!EZParser,MyCustomModule"
```

### 5. Add Heavy Module Later (SigCheck)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "VelociKAPE-Pro.ps1" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -SkipExtract `
  -ExportPath "C:\Export" `
  -OutputPath "C:\Output" `
  -Modules "SysInternals_SigCheck"
```

### 6. Mass Parsing Across Many Servers
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "Mass-VelociKAPE.ps1" `
  -ServerList "servers.txt" `
  -KapePath "C:\Tools\KAPE\kape.exe" `
  -ExportBase "C:\Export" `
  -OutputBase "C:\Output"
```

---

## 🧠 DFIR Workflow Scenarios

### Scenario A: Live Incident Triage
- Run with default profile excluding `SigCheck`
- Output goes to per-host folders
- Combine results for cross-host correlation

### Scenario B: Post-Incident Deep Dive
- Rerun only `SysInternals_SigCheck` on already parsed servers
- Merge into existing combined CSV
- Use [Timeline Explorer](https://ericzimmerman.github.io/) for pivoting

### Scenario C: IOC Hunting
1. Run `VelociKAPE-Pro.ps1` to generate combined CSV
2. Open in Timeline Explorer
3. Filter by:
   - File names
   - Hash matches
   - Registry key patterns
   - Command-line arguments
4. Export hits for reporting

### Scenario D: Custom Artifact Modules
- Build `.tkape` targeting your org’s log locations
- Pass it to `-Modules` for targeted parsing

---

## ⚡ Performance Tips

- Run from SSD for speed
- Avoid network shares for OutputPath when possible
- Exclude heavy modules until final pass
- Use `-DryRun` to validate before long jobs
- Run in parallel (Mass-VelociKAPE) for large-scale IR

---

## 🛠 Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| **`Cannot convert value "System.String" to type "System.Boolean"`** | Incorrect Boolean syntax | Use `-Param:$true` or `-Param:$false` |
| **No output generated** | Wrong `ExportPath` structure | Ensure `serverX\uploads` exists |
| **KAPE not found** | Wrong `-KapePath` | Point to `kape.exe` directly |
| **Slow parsing** | Heavy modules or HDD | Remove `SysInternals_SigCheck`, use SSD |

---

## 📌 Notes
- **SysInternals_SigCheck**: Expensive, run last.
- **Timeline Explorer**: Highly recommended for browsing combined CSVs.
- Ensure **`-ExportPath`** matches your KAPE export structure exactly.

---

## 📄 License
MIT License
