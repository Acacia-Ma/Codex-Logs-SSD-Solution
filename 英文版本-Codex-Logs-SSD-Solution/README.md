# Codex Logs SSD Solution (V9)

For questions, contact: dlmu108014301@dlmu.edu.cn or dlmu108014301@qq.com.

Back to the root README: [中文 / English](../README.md).

## What it solves

V9 puts Codex's high-frequency SQLite runtime data in a 512 MB ImDisk RAM disk first. It switches to a preallocated, fixed-size 256 MB SSD fallback image only when RAM is unavailable. All `logs_*.sqlite`, WAL, and SHM files share a hard aggregate cap of 48 MB / 12,000 rows, and idle SSD-fallback maintenance runs at most once every 60 minutes. Startup, logon, state-snapshot, and capacity-guard tasks rebuild the runtime directory after reboot, so the user does not need to delete `config.toml` or `active` manually.

The latest V9 also fixes the Windows PowerShell 5.1 empty-collection `.Sum` failure in Verify: capacity is accumulated item by item. Top-level `.codex` logs that predate installation are warnings; new writes after installation are failures.

## Compatibility

- Supported: Windows 10 22H2 x64 and Windows 11 22H2 or later x64.
- Installation runtime: Windows PowerShell 5.1 (`powershell.exe`). PowerShell 7 may coexist, but it is not the supported runtime for V9 installation or scheduled tasks.
- Required: administrator rights, 64-bit ImDisk, Python with the standard-library `sqlite3` module, and Codex/ChatGPT fully closed during installation.

## Install from a clean download

1. Download `Codex-RAMLog-Guard-Windows-V9.zip` and its `.sha256` file from the [latest Release](https://github.com/Leclerc-Hamilton/Codex-Logs-SSD-Solution/releases/latest).
2. Extract the ZIP to a local folder; do not run scripts inside the archive.
3. Open **Windows PowerShell** as Administrator. If the prompt is `C:\Windows\System32>`, change to the real extracted directory first. Copy each complete code block as a unit.

```powershell
Set-Location -LiteralPath "D:\actual\extraction\Codex-RAMLog-Guard-Windows-V9"
```

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force
```

```powershell
.\Preflight-CodexRamLogGuardV9.ps1
```

```powershell
.\Install-CodexRamLogGuardV9.ps1
```

The installer detects and migrates V4–V8/V9 legacy components. On failure it preserves or restores the previous Codex configuration and Guard tasks. Reboot Windows normally after a successful installation.

## Verify after reboot

After reboot, open Windows PowerShell as Administrator and return to the extracted directory:

```powershell
Set-Location -LiteralPath "D:\actual\extraction\Codex-RAMLog-Guard-Windows-V9"
```

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force
```

```powershell
.\Verify-CodexRamLogGuardV9.ps1
```

The expected result is `总体结果：PASS` (overall PASS). A `[WARN]` listing `logs_*.sqlite` at the top of `.codex` means those files predate V9; it is not evidence of a new V9 write. Continue using Codex when the final result is PASS. Verify checks the stable `active` path, RAM/SSD mode, all four scheduled tasks, maintenance intervals, and the aggregate capacity cap.

To check the extracted package and archive hashes:

```powershell
.\Check-CodexRamLogGuardV9.ps1 -ArchivePath "D:\Downloads\Codex-RAMLog-Guard-Windows-V9.zip" -DownloadsPath "$env:USERPROFILE\Downloads"
```

## SSD write boundaries

- In RAM mode, successful V9 capacity maintenance, state snapshots, and state files stay in RAM; V9 does not append a new SSD log record for every conversation turn.
- The SSD fallback image is fixed at 256 MB and does not grow with conversations.
- All fallback databases together are capped at 48 MB / 12,000 rows, with a 32 MB / 8,000-row target.
- Capacity checks wake every 10 minutes while active; idle SSD-fallback maintenance runs at most every 60 minutes.
- Diagnostic logs rotate with a per-file size limit; successful runs do not continuously append diagnostic text.

These limits cover V9's own Codex log and state writes; they do not claim that Windows or other Codex directories perform absolutely zero writes.

## Repair, uninstall, and legacy cleanup

Repair the installed tasks and runtime directory:

```powershell
.\Repair-CodexRamLogGuardV9.ps1
```

Uninstall V9 (a recoverable backup is kept first):

```powershell
.\Uninstall-CodexRamLogGuardV9.ps1
```

Clean legacy directories while retaining migration backups:

```powershell
.\Remove-LegacyCodexRamLogGuardV9.ps1
```

Do not delete the `sqlite_home` line from `config.toml`, and do not change it back to `R:\CodexSQLite`. V9's stable entry point is `%USERPROFILE%\.codex\ramlog-v9\active`, prepared by Guard tasks at startup and logon.

## License

This project is released under the [MIT License](../LICENSE).
