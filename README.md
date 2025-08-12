# VelociKAPE

**Update:** Scripts now use **[switch]** parameters for all toggles. Examples: `-SkipExtract -DryRun -CombineOnly`. No `$true/$false` needed.

## Quick Start (Pro)

Full pipeline (discover ZIPs → extract → parse → combine if >1 server):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" ^
  -KapePath "C:\Tools\KAPE\kape.exe" ^
  -CollectionPath "C:\Collection" ^
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined"
```

Parse-only (skip extraction, work from C:\Export\<Server>\uploads):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" ^
  -KapePath "C:\Tools\KAPE\kape.exe" -SkipExtract ^
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" -DryRun
```

Combine-only later:
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" ^
  -OutputPath "C:\Output" -CombinedPath "C:\Combined" -CombineOnly
```

### Defaults
Modules (SigCheck excluded):
```
!EZParser,BMC-Tools_RDPBitmapCacheParser,BitsParser,Hayabusa,WMI-Parser,LogParser,RECmd_AllBatchFiles,PowerShell_ParseScheduledTasks
```

Run **SysInternals_SigCheck** as a **second pass**:
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\VelociKAPE-Pro.ps1" ^
  -KapePath "C:\Tools\KAPE\kape.exe" -SkipExtract ^
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined" `
  -Modules "SysInternals_SigCheck"
```

## Mass (lite) runner
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\Mass-VelociKAPE.ps1" ^
  -KapePath "C:\Tools\KAPE\kape.exe" ^
  -CollectionPath "C:\Collection" ^
  -ExportPath "C:\Export" -OutputPath "C:\Output" -CombinedPath "C:\Combined"
```

Combine-only:
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\Mass-VelociKAPE.ps1" ^
  -OutputPath "C:\Output" -CombinedPath "C:\Combined" -CombineOnly
```

## Notes
- Works with `C:\Export\<Server>\uploads` **or** `C:\Export\<Server>\data\uploads`.
- Normalized combining: one `combined_<Artifact>.csv` per artifact (date prefix + `_Output` suffix removed). Rows include `ServerName` and `SourcePath`.
- Use **PowerShell 7** + `-Parallel 2..3` (Pro script) for speed. Add AV exclusions for `kape.exe`, `C:\Export`, `C:\Output`, `C:\Combined`.
- Use **Timeline Explorer** (Eric Zimmerman) to slice large combined CSVs quickly.
