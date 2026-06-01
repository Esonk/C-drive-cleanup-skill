# Windows C 盘安全清理 Skill

[English README](README.md)

这是一个用于 Codex 的 Windows C 盘安全清理 skill，面向“C 盘爆满但不敢乱删文件”的场景。

它不会让 Codex 上来就删文件，而是先扫描、再分类、解释风险，并且在删除缓存或用 junction 迁移目录前要求用户逐项明确同意。

## 为什么需要它

Windows C 盘清理的风险很高，因为占空间最大的目录往往是系统托管目录。误删文件可能破坏 Windows 更新、软件卸载/修复、开发环境、浏览器资料或安全软件。

这个 skill 把清理流程变成一套保守、可复查的步骤：

- 先执行只读扫描
- 区分系统文件、缓存、日志、可迁移目录
- 没有明确授权就不删除
- junction 迁移必须一个目录一个目录确认
- 迁移前先复制到目标盘并校验，再创建 junction
- 遇到被占用文件默认跳过，不强制关闭程序

## 它能做什么

- 检查 C 盘剩余空间和顶层目录占用
- 识别真实 C 盘占用，避免把已有 junction 指向 D 盘的内容重复计入 C 盘
- 解释哪些 Windows 文件和目录不能手动删除
- 找出低风险清理目标，例如 Temp、pip cache、npm cache、浏览器组件缓存、updater 缓存
- 规划 VS Code、Cursor、Antigravity、Eclipse 等开发工具目录的 junction 迁移
- 每次授权操作后汇报清理前后空间变化

## 它不会做什么

- 不会手动删除 `C:\Windows\WinSxS`、`C:\Windows\Installer`、`System32`、`SysWOW64`、`pagefile.sys`、`swapfile.sys`、`hiberfil.sys`。
- 不会未经确认批量删除厂商软件或安全软件数据。
- 不会一次授权后连续迁移多个目录。
- 不会未经授权强制关闭应用程序。

## 实际效果示例

一次真实清理流程中，C 盘空间变化如下：

| 步骤 | C 盘剩余空间 |
|---|---:|
| 初始扫描 | 15.84 GB |
| 用户手动关闭休眠，并执行已授权缓存清理后 | 35.23 GB |
| 执行已授权 junction 迁移后 | 39.40 GB |

可查看[脱敏扫描示例](examples/scan-report.zh-CN.md)。

## 安装方式

把仓库克隆到 Codex skills 目录：

```powershell
git clone https://github.com/Esonk/C-drive-cleanup-skill `
  "$env:USERPROFILE\.codex\skills\windows-c-drive-cleanup"
```

如有需要，重启 Codex 让 skill 被重新发现。

## 示例提示词

```text
我的 C 盘快满了，请使用 Windows C Drive Cleanup skill 扫描并给出安全清理计划。没有我的确认不要修改任何文件。
```

```text
帮我扫描 C 盘，告诉我哪些是系统文件不能动，哪些缓存可以删，哪些目录可以通过 junction 移到 D 盘。
```

```text
帮我把一个已确认的编辑器扩展目录从 C 盘迁移到 D 盘，并创建 junction。每迁移一个目录都要先问我。
```

## 安全模型

这个 skill 的核心是逐项授权：

1. 先用只读命令扫描。
2. 按风险分类结果。
3. 给出分阶段计划。
4. 每个清理阶段或每个 junction 迁移目录都单独请求授权。
5. junction 迁移时先复制、校验文件数量和字节数，再创建 junction。
6. 每次操作后汇报清理前后的具体数字。

## 推荐 GitHub Topics

建议给仓库添加这些 topics，方便更多用户搜索到：

`codex-skill`, `windows`, `disk-cleanup`, `powershell`, `junction`, `storage-management`, `c-drive`, `developer-tools`

## 许可证

MIT License，见 [LICENSE](LICENSE)。
