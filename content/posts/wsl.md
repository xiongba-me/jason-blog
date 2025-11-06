---
date: '2025-11-06T10:40:06+08:00'
draft: false
title: '局域网 SSH 连接到 WSL 完整配置指南'
tags: ['内网', 'wsl']
categories: ["内网"]
---

> 💡 **WSL（Windows Subsystem for Linux）** 是一个在 Windows 10 和 Windows 11 上能够原生运行 Linux 二进制可执行文件（ELF格式）的兼容层。它能让开发者在 Windows 系统上直接运行一个完整的 Linux 发行版（例如 Ubuntu、Debian、Kali 等），而无需创建虚拟机或设置双系统。
---

### 🔧 一、前提条件
| 项目 | 要求 | 检查方法 |
|------|------|----------|
| **WSL 版本** | WSL2 + Windows 10/11 22H2+ | `wsl -l -v` → 查看 `VERSION` |
| **发行版** | Ubuntu（或其他 Debian 系） | `wsl -l` → 确认名称（如 `Ubuntu`） |
| **SSH 服务** | 已安装 `openssh-server` | 在 WSL 中运行 `which sshd` |

> ⚠️ 如果未安装 SSH：
> ```bash
> sudo apt update && sudo apt install -y openssh-server
> ```

---

### ✅ 二、核心配置步骤（按顺序执行）

#### **步骤 1：启用 WSL 的 systemd（关键！）**
**验证**：
   ```powershell
   wsl -d Ubuntu -e ps -p 1 -o comm=
   # 应输出：systemd
   ```

> 💡 为什么需要？  
> 使 `sudo systemctl enable ssh` 生效（否则服务不会开机启动）。

---

#### **步骤 2：配置 SSH 服务**
在 **WSL 终端** 中执行：
```bash
# 1. 启用 SSH 服务（开机自启）
sudo systemctl enable ssh

# 2. 确保服务运行（测试）
sudo systemctl start ssh
sudo systemctl is-active ssh  # 应返回：active
```

> 📌 重要：  
> 无需修改 `/etc/ssh/sshd_config`，默认配置已足够。

---

#### **步骤 3：自动更新端口转发（IP 变化时自动适配）**
1. **创建更新脚本**（在 Windows 中）：
   ```powershell
   @'
    $wslIp = (wsl hostname -I).Trim()
    if ($wslIp -eq "") {
        Write-Host "WSL is not running. Please start WSL first."
        exit 1
    }
    # Remove existing rule
    netsh interface portproxy delete v4tov4 listenport=2222 listenaddress=0.0.0.0 >$null 2>&1

    # Add new rule: forward Windows :2222 to WSL :22
    netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=22 connectaddress=$wslIp

    # Open firewall (only if not already open)
    $rule = Get-NetFirewallRule -DisplayName "WSL SSH Forward" -ErrorAction SilentlyContinue
    if (-not $rule) {
        New-NetFirewallRule -DisplayName "WSL SSH Forward" -Direction Inbound -LocalPort 2222 -Protocol TCP -Action Allow | Out-Null
    }
    Write-Host "Successfully forwarded Windows port 2222 to WSL ($wslIp):22"
   '@ | Out-File -FilePath "$env:USERPROFILE\wsl-port-forward.ps1" -Encoding UTF8
   ```

2. **在 WSL `~/.bashrc` 中添加自动执行**：
   ```bash
   # Auto-update Windows port forwarding when WSL starts
    if [ -z "$WSL_PORT_FORWARD_RUN" ]; then
      export WSL_PORT_FORWARD_RUN=1
     # Only run in interactive shells
        if [ -t 0 ]; then
          cmd.exe /c "powershell.exe -ExecutionPolicy Bypass -File \"%USERPROFILE%\\wsl-port-forward.ps1\"" >/dev/null 2>&1
        fi
    fi
   ```
   >    ✅ 作用：**每次打开 WSL 终端时自动更新 IP**

---

#### **步骤 4：设置开机自动启动 WSL（无窗口干扰）**
1. **打开 Windows 启动文件夹**：
    - 按 `Win + R` → 输入 `shell:startup` → 回车
2. **创建快捷方式**：
    - 在文件夹中 **右键 → 新建 → 快捷方式**
    - 位置：`C:\Windows\System32\wsl.exe`
    - 名称：`WSL Auto Start`
3. **设置最小化启动**：
    - 右键快捷方式 → **属性** → “运行方式” → 选 **“最小化”**
    - 点击“确定”

> ✅ 效果：  
> Windows 登录后，自动启动 WSL（终端窗口最小化到任务栏），SSH 服务立即可用。

---

### 🔍 三、验证流程（开机后 30 秒内完成）

| 操作 | 命令/步骤                                     | 预期结果                     |
|------|-------------------------------------------|--------------------------|
| **1. 检查 WSL 是否运行** | `wsl uname`                               | 返回uname（如 `Linux`）       |
| **2. 检查 SSH 服务** | `wsl systemctl is-active ssh`             | `active`                 |
| **3. 检查端口转发** | `Test-NetConnection 127.0.0.1 -Port 2222` | `TcpTestSucceeded: True` |
| **4. 从其他设备 SSH 连接** | `ssh -p 2222 <你的WindowsIP>`               | 成功登录 WSL                 |

> 💡 你的 Windows IP：在 PowerShell 中运行 `ipconfig | findstr "IPv4"` → 找到 `IPv4 地址`。

---

### ⚙️ 四、常见问题解决

| 问题 | 解决方案 |
|------|----------|
| **SSH 连接超时** | 1. 检查 Windows 防火墙：允许端口 `2222`<br>2. 确认 `wsl-port-forward` 已执行（打开 WSL 终端手动运行 `~/.local/bin/wsl-port-forward`） |
| **WSL 退出** | 1. 检查启动文件夹快捷方式是否设为“最小化”<br>2. 确保未删除 `~/.bashrc` 中的自动更新脚本 |
| **端口转发失效** | 1. 重启 WSL：`wsl --shutdown`<br>2. 重新打开 WSL 终端（触发 `.bashrc` 自动更新） |

---

### ✅ 最终效果
| 项目 | 状态 |
|------|------|
| WSL 开机自动启动 | ✅ 无窗口弹出（最小化） |
| SSH 服务自动运行 | ✅ `sudo systemctl enable ssh` |
| 端口转发自动更新 | ✅ `.bashrc` 触发 `wsl-port-forward` |
| 局域网 SSH 连接 | ✅ `ssh -p 2222 <WindowsIP>` |

> 🎯 **配置完成！**  
> 从此你的 WSL 就像一台真正的 Linux 服务器：  
> **开机即用、服务常驻、IP 自适应、无干扰运行**
---
