---
date: '2025-11-06T10:40:06+08:00'
draft: false
title: '如何SSH远程连接到WSL'
tags: ['内网', 'wsl']
categories: ["内网"]
---

> 💡 **WSL（Windows Subsystem for Linux）** 是一个在 Windows 10 和 Windows 11 上能够原生运行 Linux 二进制可执行文件（ELF格式）的兼容层。它能让开发者在 Windows 系统上直接运行一个完整的 Linux 发行版（例如 Ubuntu、Debian、Kali 等），而无需创建虚拟机或设置双系统。

### 前提条件：开启镜像模式网络

为了让外部设备能够直接访问 WSL，需要启用 WSL 的镜像模式网络（mirrored networking mode）。这会将 Windows 的网络接口直接镜像到 Linux 中，使 WSL 共享与 Windows 主机相同的 IP 地址。

在您的 Windows 用户目录（通常是 `C:\Users\<YourUsername>`）下找到或创建 `.wslconfig` 文件，并添加以下内容：

```ini
[wsl2]
networkingMode=Mirrored
```
**注意**：修改配置后，需要重启 WSL 才能生效。可以在 PowerShell 中运行 `wsl --shutdown` 来关闭 WSL，下次启动时配置就会应用。

### 步骤：

1. **以管理员身份打开PowerShell**（右键点击PowerShell → 以管理员身份运行）

2. **检查并安装OpenSSH Server**：
   ```powershell
   # 检查是否已安装
   Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
   
   # 如果显示"State : NotPresent"，则执行安装
   Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
   ```

3. **启动SSH服务并设置为自动启动**：
   ```powershell
   Start-Service sshd
   Set-Service -Name sshd -StartupType 'Automatic'
   ```

4. **设置默认Shell为WSL的Bash**：
   ```powershell
   New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Windows\System32\bash.exe" -PropertyType String -Force
   ```

5. **配置Windows防火墙允许SSH连接**：
   ```powershell
   netsh advfirewall firewall add rule name="SSH" dir=in action=allow protocol=TCP localport=22
   ```

6. **从远程机器连接**：
   ```bash
   ssh username@windows_host_ip
   ```
   （使用Windows的用户名和密码登录，登录后直接进入WSL的Bash界面）

