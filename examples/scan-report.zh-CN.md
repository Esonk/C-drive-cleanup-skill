# 扫描报告示例

这是一个脱敏示例，用于展示该 skill 期望生成的报告结构。

## 初始状态

| 指标 | 数值 |
|---|---:|
| C 盘总容量 | 120 GB |
| C 盘剩余空间 | 15.84 GB |
| C 盘已用空间 | 104.16 GB |
| 空闲比例 | 13.2% |

## 主要发现

| 区域 | 大小 | 建议 |
|---|---:|---|
| `C:\Windows` | 34.56 GB | 系统托管，不要手动删除。 |
| `C:\Users` | 32.21 GB | 主要清理和迁移目标。 |
| 根目录系统文件 | 14.71 GB | 包含休眠文件和分页文件。 |
| `C:\ProgramData` | 7.10 GB | 混合厂商日志和缓存，需要谨慎复查。 |

## 低风险清理候选

| 目标 | 估算大小 |
|---|---:|
| `%LOCALAPPDATA%\Temp` | 2.21 GB |
| `%LOCALAPPDATA%\npm-cache` | 1.24 GB |
| `%LOCALAPPDATA%\pip\cache` | 0.81 GB |
| Chrome `OptGuideOnDeviceModel` | 2.67 GB |
| 应用 updater 缓存 | 1.4 GB |

## Junction 迁移候选

| 源目录 | 估算大小 |
|---|---:|
| `%USERPROFILE%\.vscode\extensions` | 2.01 GB |
| `%USERPROFILE%\.antigravity\extensions` | 0.84 GB |
| `%USERPROFILE%\.cursor\extensions` | 0.77 GB |
| `%USERPROFILE%\.p2` | 0.54 GB |

## 已授权操作结果

| 步骤 | C 盘剩余空间 |
|---|---:|
| 初始扫描 | 15.84 GB |
| 用户手动关闭休眠，并授权缓存清理后 | 35.23 GB |
| 授权 junction 迁移后 | 39.40 GB |

每一次删除或迁移都需要用户单独确认。
