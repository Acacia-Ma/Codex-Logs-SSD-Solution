# Codex 日志 SSD 解决方案（V9）

如有疑问，可以联系我：dlmu108014301@dlmu.edu.cn、dlmu108014301@qq.com。

返回根 README：[中文 / English](../README.md)。

## 解决什么问题

V9 让 Codex 的高频 SQLite 运行数据优先写入 512 MB ImDisk RAM；只有 RAM 不可用时才切换到预分配、固定 256 MB 的 SSD 回退镜像。所有 `logs_*.sqlite`、WAL、SHM 共同受 48 MB / 12,000 行硬上限约束，空闲 SSD 回退维护最多每 60 分钟一次。启动、登录、状态快照和容量维护计划任务会在重启后自动重新建立运行目录，因此不需要手动删除 `config.toml` 或 `active`。

最新 V9 同时修复了 Verify 在 Windows PowerShell 5.1 上因空集合 `.Sum` 失败的问题：容量按逐项累加计算。安装前遗留在 `.codex` 顶层的日志只会显示警告；安装后的新增写入才会被判定为失败。

## 兼容性

- 支持：Windows 10 22H2 x64、Windows 11 22H2 及更高版本 x64。
- 安装运行时：Windows PowerShell 5.1（`powershell.exe`）。PowerShell 7 可以共存，但不是 V9 安装与计划任务的受支持运行时。
- 必须：管理员权限、64 位 ImDisk、可导入标准库 `sqlite3` 的 Python，以及安装时完全退出 Codex/ChatGPT。

## 从零安装

1. 从 [最新 Release](https://github.com/Leclerc-Hamilton/Codex-Logs-SSD-Solution/releases/latest) 下载 `Codex-RAMLog-Guard-Windows-V9.zip` 和对应 `.sha256`。
2. 将 ZIP 解压到本地文件夹；不要在压缩包内直接运行。
3. 以管理员身份打开 **Windows PowerShell**。如果提示符是 `C:\Windows\System32>`，先切换到真实解压目录。每个代码块都要整块复制。

```powershell
Set-Location -LiteralPath "D:\实际解压目录\Codex-RAMLog-Guard-Windows-V9"
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

安装器会识别并迁移 V4–V8/V9 旧组件；失败时保留或恢复旧配置与计划任务。安装成功后请正常重启 Windows。

## 重启后的验证

重启后，以管理员身份打开 Windows PowerShell，再次切换到解压目录：

```powershell
Set-Location -LiteralPath "D:\实际解压目录\Codex-RAMLog-Guard-Windows-V9"
```

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force
```

```powershell
.\Verify-CodexRamLogGuardV9.ps1
```

正常结果是 `总体结果：PASS`。如果看到 `.codex` 顶层已有 `logs_*.sqlite` 的 `[WARN]`，那是安装前遗留文件，不代表 V9 安装后继续写入；只要最后是 PASS 就可以正常使用 Codex。Verify 会检查稳定 `active` 路径、RAM/SSD 模式、四个计划任务、维护间隔和聚合容量上限。

如果需要检查解压包自身的文件和哈希：

```powershell
.\Check-CodexRamLogGuardV9.ps1 -ArchivePath "D:\下载目录\Codex-RAMLog-Guard-Windows-V9.zip" -DownloadsPath "$env:USERPROFILE\Downloads"
```

## SSD 写入边界

- RAM 模式下，V9 的成功容量维护、状态快照和状态文件留在 RAM，不会按每轮对话向 SSD 追加日志。
- SSD fallback 镜像固定 256 MB，不会随对话增长。
- fallback 中所有日志库合计最多 48 MB / 12,000 行；目标为 32 MB / 8,000 行。
- 容量维护运行中每 10 分钟检查一次；空闲 SSD 回退维护最多每 60 分钟一次。
- 诊断日志采用轮换并限制单份大小；正常成功路径不持续追加诊断文本。

这限制的是 V9 自有 Codex 日志和状态写入，不声称 Windows 或 Codex 其他目录绝对零写入。

## 修复、卸载和问题反馈

修复已安装的任务和运行目录：

```powershell
.\Repair-CodexRamLogGuardV9.ps1
```

卸载 V9（会先保留可恢复备份）：

```powershell
.\Uninstall-CodexRamLogGuardV9.ps1
```

清理旧版本目录但保留迁移备份：

```powershell
.\Remove-LegacyCodexRamLogGuardV9.ps1
```

请不要直接删除 `config.toml` 的 `sqlite_home` 行，也不要把 `sqlite_home` 改回 `R:\CodexSQLite`；V9 的稳定入口是 `%USERPROFILE%\.codex\ramlog-v9\active`，由 Guard 任务在启动和登录时准备。

## 许可证

本项目采用 [MIT License](../LICENSE)。
