# 🐚 Nushell for Android

[![Build Nushell Module](https://github.com/shanyouli/nu-magisk/actions/workflows/numagisk.yml/badge.svg)](https://github.com/shanyouli/nu-magisk/actions/workflows/numagisk.yml)
[![Latest Release](https://img.shields.io/github/v/release/shanyouli/nu-magisk?logo=github)](https://github.com/shanyouli/nu-magisk/releases/latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

一个 [KernelSU](https://kernelsu.org/guide/module.html) / [Magisk](https://github.com/topjohnwu/Magisk) 通用模块，以 **systemless** 方式将现代 Shell —— [Nushell](https://www.nushell.sh/) 安装到 Android 设备的 `/system/bin/nu`，无需修改系统分区。

> 模块由 GitHub Actions 每周自动跟踪 [nushell/nushell](https://github.com/nushell/nushell/releases) 上游 Release 并构建，二进制采用 `aarch64-unknown-linux-musl`（静态链接，无 glibc 依赖）。

## ✨ 特性

- 🔁 **全自动跟进上游**：每周一自动检查 Nushell 新版本，发布后自动构建并推送 Release
- 🏷️ **手动构建可控**：支持指定任意版本构建、强制重建
- 🧩 **遵循 KernelSU 模块规范**：标准 `module.prop` + `customize.sh`（架构校验、规范权限）
- 🛡️ **systemless**：不触碰系统分区，卸载即恢复原状
- 📦 **静态链接**：musl 二进制，兼容任意 Android 版本的 arm64 设备

## 📲 安装

### 前置要求

| 项目 | 要求 |
| --- | --- |
| 架构 | **arm64** (aarch64) |
| Root 方案 | KernelSU 或 Magisk |

### 步骤

1. 从 [Releases](https://github.com/shanyouli/nu-magisk/releases/latest) 下载 `nushell-magisk-vX.Y.Z.zip`
2. 在 **KernelSU 管理器** 或 **Magisk App** 中刷入该 zip
3. 重启设备
4. 打开终端，输入 `nu` 🎉

> [!WARNING]
> **KernelSU 用户注意**：本模块通过 `system/` 目录修改 `/system/bin/nu`，
> 需要安装 metamodule（如 [meta-overlayfs](https://github.com/tiann/KernelSU/releases)）才能挂载生效，详见 [KernelSU 模块文档](https://kernelsu.org/guide/module.html)。
> Magisk 用户无需额外操作。

## 🚀 使用示例

```sh
# 进入交互式 Shell
nu

# 单命令执行：列出当前目录中大于 1MB 的文件
nu -c "ls | where size > 1mb"

# 管道与结构化数据
nu -c "open Cargo.toml | get package.version"
```

> 💡 建议仅作为增强工具使用，**不要**替换系统默认 Shell（`/system/bin/sh`）。

## 🌳 模块结构

```txt
nushell_magisk/
├── module.prop          # 模块元数据（id / version / versionCode …）
├── customize.sh         # 安装脚本（架构校验、权限设置、KernelSU 提示）
└── system/
    └── bin/
        └── nu           # Nushell 主程序 → /system/bin/nu
```

| 字段 | 值 |
| --- | --- |
| 模块 ID | `nushell_magisk` |
| versionCode | 构建日期（`YYYYMMDD`） |
| 安装位置 | `/system/bin/nu` |

## 🤖 自动构建

Workflow：[`.github/workflows/numagisk.yml`](.github/workflows/numagisk.yml)

| 触发方式 | 行为 |
| --- | --- |
| ⏰ 定时（每周一 00:00 UTC） | 获取上游最新稳定版；**已发布过则自动跳过**，有新版本才构建 |
| 🖱️ 手动触发（默认） | 同上，等价于定时触发 |
| 🖱️ 手动触发 + 填写 `tag` | 构建指定版本（如 `0.115.0`，自动补全 `v` 前缀） |
| 🖱️ 手动触发 + 勾选 `force` | 强制重建，即使该版本已发布过 |

构建产物：

- 📦 模块 zip（Release 附件 + Actions Artifact）
- 🔐 SHA256 校验和（见 Release 说明与构建摘要）

## 🛠️ 本地验证模块包

```sh
# 1. 校验 zip 结构：module.prop 必须位于根目录
unzip -l nushell-magisk-vX.Y.Z.zip

# 2. 校验 sha256（与 Release 说明对照）
sha256sum nushell-magisk-vX.Y.Z.zip

# 3. 确认二进制架构
unzip -p nushell-magisk-vX.Y.Z.zip system/bin/nu | file -
# 期望输出: ELF 64-bit LSB ... ARM aarch64 ... statically linked
```

## 📄 许可

- 本仓库模块打包脚本：[MIT](LICENSE)
- [Nushell](https://github.com/nushell/nushell)：MIT © The Nushell Project Developers
