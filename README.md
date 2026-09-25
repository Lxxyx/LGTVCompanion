# LGTV Companion — HDMI / G-SYNC controls

An experimental community fork of [JPersson77/LGTVCompanion](https://github.com/JPersson77/LGTVCompanion), sharing desktop changes for LG WebOS TVs used as PC displays.

The custom source is on **`hdmi-gsync-controls`**, based on upstream commit `f9561e2d67b5cda61a9041e96ea72cf5adeda812`. It has not been rebased onto the latest upstream release. This is a source snapshot, not a newly tested binary release or an official LG/NVIDIA firmware fix.

## Included features

- A **G-Sync** button for the selected TV in the main window.
- Console commands that query the TV's current G-SYNC setting and toggle it, including HDMI-specific variants.
- Service commands that request G-SYNC off, wait 800 ms, and request it on again.
- An opt-in startup setting, stored as `ResyncGsyncAfterBootPowerOn`.
- A **Find** button that looks for the configured MAC address using ARP on candidate local IPv4 `/24` subnets.

These controls may help recover from some HDMI/VRR display-state problems. They do not detect image corruption and cannot guarantee that green artifacts will stop recurring.

## Usage

Configure and pair your own TV first. Replace `Device1` below with your configured device ID or name. The examples target one TV explicitly.

### Query and toggle the current setting

Run from the directory containing the custom console build:

```powershell
& '.\LGTVcli.exe' -toggle_gsync Device1
& '.\LGTVcli.exe' -toggle_gsync_hdmi1 Device1
```

HDMI-specific variants are available for inputs 1 through 4.

### Request an off/on cycle

These commands are handled by the **custom Windows service**, through the GUI executable's command-line interface:

```powershell
& '.\LGTV Companion.exe' -resync_gsync Device1
& '.\LGTV Companion.exe' -resync_gsync_hdmi1 Device1
```

Use the matching custom service build; launching a custom GUI alongside an unmodified upstream service does not add these commands to that service. A brief loss of picture during a mode change is expected.

## Current limitations

- The main-window G-Sync button and startup option alternate a locally cached state. They do **not** query the TV's current state and do **not** perform a guaranteed off/on cycle. The startup checkbox's existing wording says "Re-sync", but its implementation uses this cached toggle.
- Those cached-toggle paths also request AMD FreeSync off, including the configured HDMI input when applicable.
- The startup option is armed by the system-boot event, is consumed on a subsequent eligible power-on operation, and has a five-minute window. It is not a continuous monitor or general resume/input-switch recovery mechanism.
- The console `-toggle_gsync` path queries the TV; it is separate from the cached GUI/service toggle paths.
- The service's 800 ms off/on sequence requests setting changes without verifying that the TV has completed the handshake.
- The IP finder assumes candidate `/24` ranges, uses the first configured valid MAC address, and is limited to ARP-reachable devices. A discovery attempt can take time.
- No fresh compilation or hardware regression test was performed when publishing this source snapshot. Existing behavior is preserved rather than presented as a finished fix.

## Building

Use the upstream Windows C++ setup: Visual Studio with the v143 C++ toolset, a Windows 10 SDK, and vcpkg dependencies from `vcpkg.json`. The upstream [build workflow](.github/workflows/continuous-integration.yml) documents manifest-enabled MSBuild commands and installer dependencies.

For local development, use matching UI, console, and service builds. Upstream installation instructions and release links refer to the original project and do not include these custom features. No prebuilt EXEs, PDBs, pairing credentials, device configurations, logs, or local backups are distributed by this branch.

## 中文说明

这是 LGTV Companion 的实验性 HDMI / G-SYNC 功能分支，包含桌面按钮、命令行切换、服务端关闭再开启命令，以及按 MAC 查找电视 IP 的功能。

它提供显示状态恢复的手段，不是 LG / NVIDIA 官方的花屏修复。当前开机选项虽然写着“Re-sync”，实际仍是根据缓存切换一次开关；需要明确关闭再开启时，应使用上面的 `-resync_gsync` 服务命令。自定义界面需要配套的自定义后台服务才能使用新增服务功能。

本次公开的是源码快照，保留原有行为，没有重新编译或进行硬件回归测试。电视配对信息、个人配置、旧编译文件和调试符号不包含在发布内容中。

## Upstream and license

Original project: [JPersson77/LGTVCompanion](https://github.com/JPersson77/LGTVCompanion).

The [original README](Docs/UPSTREAM-README.md), copyright notices, and [GPL-3.0 license](LICENSE) are retained. Upstream release signatures and provenance claims apply to upstream artifacts, not to this experimental source snapshot.
