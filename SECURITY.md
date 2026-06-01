# Security Policy

## English

This skill performs local Windows disk-cleanup guidance. Treat all cleanup and junction operations as potentially destructive.

Safety expectations:

- Scan first with read-only commands.
- Ask before deleting files, disabling hibernation, moving folders, or creating junctions.
- Require separate approval for every junction migration.
- Never manually delete Windows system folders, installer caches, page files, or hibernation files.
- Skip locked files by default.

Please report unsafe instructions, unclear consent language, system-damage risks, or privacy leaks through GitHub issues. Sanitize private paths, usernames, file names, and logs before posting.

## 中文

这个 skill 用于本地 Windows 磁盘清理指导。删除文件和创建 junction 都可能有破坏性风险。

安全预期：

- 先只读扫描。
- 删除文件、关闭休眠、移动目录或创建 junction 前必须询问用户。
- 每个 junction 迁移都需要单独授权。
- 不手动删除 Windows 系统目录、安装器缓存、分页文件或休眠文件。
- 默认跳过被占用文件。

如发现不安全指令、授权不清、系统损坏风险或隐私泄露风险，请通过 GitHub issue 反馈。提交前请脱敏私人路径、用户名、文件名和日志。
