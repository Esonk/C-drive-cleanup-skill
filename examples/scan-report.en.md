# Example Scan Report

A sanitized example of the workflow this skill is designed to produce.

## Initial State

| Metric | Value |
|---|---:|
| C drive total | 120 GB |
| C drive free | 15.84 GB |
| C drive used | 104.16 GB |
| Free percentage | 13.2% |

## Main Findings

| Area | Size | Guidance |
|---|---:|---|
| `C:\Windows` | 34.56 GB | System-managed. Do not manually delete. |
| `C:\Users` | 32.21 GB | Main cleanup and migration target. |
| Root system files | 14.71 GB | Includes hibernation and page files. |
| `C:\ProgramData` | 7.10 GB | Mixed vendor logs and caches. Review carefully. |

## Safe Cleanup Candidates

| Target | Approx. Size |
|---|---:|
| `%LOCALAPPDATA%\Temp` | 2.21 GB |
| `%LOCALAPPDATA%\npm-cache` | 1.24 GB |
| `%LOCALAPPDATA%\pip\cache` | 0.81 GB |
| Chrome `OptGuideOnDeviceModel` | 2.67 GB |
| App updater caches | 1.4 GB |

## Junction Migration Candidates

| Source | Approx. Size |
|---|---:|
| `%USERPROFILE%\.vscode\extensions` | 2.01 GB |
| `%USERPROFILE%\.antigravity\extensions` | 0.84 GB |
| `%USERPROFILE%\.cursor\extensions` | 0.77 GB |
| `%USERPROFILE%\.p2` | 0.54 GB |

## Approved Results

| Step | Free Space |
|---|---:|
| Initial scan | 15.84 GB |
| After user disabled hibernation and approved cache cleanup | 35.23 GB |
| After approved junction migrations | 39.40 GB |

Each deletion or migration was separately approved by the user.
