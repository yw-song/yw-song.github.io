---
layout: article
title: 宿主机 USB 设备连接到 WSL 中 
mathjax: true
tags: WSL Ubuntu
---

> 转载自 [链接 USB 设备](https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb) 为整理方便写到博客中。

# 宿主机 `USB` 设备连接到 `WSL` 中 

## 先决条件

- 运行 Windows 11（内部版本 22000 或更高版本）。 （可提供 Windows 10 支持，请参见下面的注释）
- 需要具有 x64 处理器的计算机。 （x86 和 Arm64 目前不支持 usbipd win）。
- WSL [已安装并使用最新版本进行设置](https://learn.microsoft.com/zh-cn/windows/wsl/install)。
- Linux 发行版已安装并[设置为 WSL 2](https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands#set-wsl-version-to-1-or-2)。

我使用的是：Windows 10 专业版 22H2 | WSL2 | Ubuntu - 20.04

> [!NOTE]
>
> 若要检查 Windows 版本及内部版本号，选择 Windows 徽标键 + R，然后键入“winver”，选择“确定”。 可通过选择“开始”>“设置”>“Windows 更新”>“[检查更新](ms-settings:windowsupdate)”来更新到最新的 Windows 版本。 若要检查 Linux 内核版本，请打开 Linux 发行版并输入命令：`uname -a`。 若要手动更新到最新内核，请打开 PowerShell 并输入以下命令：`wsl --update`。

## 安装 USBIPD-WIN 项目

WSL 本身并不支持连接 USB 设备，因此你需要安装开源 usbipd-win 项目。

**内核要求**

若要将 USBIPD 与适用于 Linux 的 Windows 子系统 (WSL) 配合使用，则需要具有 [Linux 内核版本 5.10.60.1 或更高版本](https://github.com/dorssel/usbipd-win/wiki/WSL-support/6befeedd4c8e2a49468e4b03532c9a20478f8677)。 如果已安装的内核版本低于 5.10.60.1，则可以通过使用 `wsl --shutdown` 先关闭 WSL 的任何正在运行的实例，然后运行以下命令来更新它：`wsl --update`。

**在 WSL 上安装 USBIPD**

1. 转到 [usbipd-win 项目的最新发布页](https://github.com/dorssel/usbipd-win/releases)。
2. 选择 .msi 文件，该文件将下载安装程序。 （你可能会收到一条警告，要求你确认你信任此下载）。
3. 运行下载的 usbipd-win_x.msi 安装程序文件。

**这将安装：**

- 名为 `usbipd` 的服务（显示名称：USBIP 设备主机）。 可使用 Windows 中的“服务”应用检查此服务的状态。
- 命令行工具 `usbipd`。 此工具的位置将添加到 PATH 环境变量。
- 名为 `usbipd` 的防火墙规则，用于允许所有本地子网连接到服务。 可修改此防火墙规则以微调访问控制。

## 插入USB设备

在附加 USB 设备之前，请确保 WSL 命令行已打开。 这将使 WSL 2 轻型 VM 保持活动状态。

> [!NOTE]
>
> 此文档假定已安装 [`usbipd-win 4.0.0` 或更高版本](https://github.com/dorssel/usbipd-win/releases/latest)

1. 通过以*管理员* 模式打开 PowerShell 并输入以下命令，列出所有连接到 Windows 的 USB 设备。 列出设备后，选择并复制要附加到 WSL 的设备总线 ID。

   ```powershell
   usbipd list
   ```

2. 在附加 USB 设备之前，必须使用命令 `usbipd bind` 来共享设备，从而允许它附加到 WSL。 这需要管理员权限。 选择要在 WSL 中使用的设备总线 ID，然后运行以下命令。 运行命令后，请再次使用命令 `usbipd list` 验证设备是否已共享。

   ```powershell
   usbipd bind --busid 4-4
   ```

3. 若要附加 USB 设备，请运行以下命令。 （不再需要使用提升的管理员提示。）确保 WSL 命令提示符处于打开状态，以使 WSL 2 轻型 VM 保持活动状态。 请注意，只要 USB 设备连接到 WSL，Windows 将无法使用它。 附加到 WSL 后，任何作为 WSL 2 运行的分发版本都可以使用 USB 设备。 使用 `usbipd list` 验证设备是否已附加。 在 WSL 提示符下，运行 `lsusb` 以验证 USB 设备是否已列出，并且可以使用 Linux 工具与之交互。

   ```powershell
   usbipd attach --wsl --busid <busid>
   ```

4. 打开 Ubuntu（或首选的 WSL 命令行），使用以下命令列出附加的 USB 设备：

   Bash

   ```bash
   lsusb
   ```

   你应会看到刚刚附加的设备，并且能够使用常规 Linux 工具与之交互。 根据你的应用程序，你可能需要配置 udev 规则以允许非根用户访问设备。

5. 在 WSL 中完成设备使用后，可物理断开 USB 设备，或者从 PowerShell 运行此命令：

   PowerShell

   ```powershell
   usbipd detach --busid <busid>
   ```

~~粘贴复制爽~~