---
title: 'Nginx 多平台配置实战指南'
date: '2024-07-22'
tags: ['ImmortalWrt', 'Synology', 'nginx', '运维']
categories: ["运维", "技术"]
description: "详细记录 ImmortalWrt 和 Synology DSM 平台上的 Nginx 配置差异与实践经验"
draft: false
---

> 💡 **实战指南**：本文详细对比分析 ImmortalWrt 和 Synology DSM 两大平台的 Nginx 配置差异，提供完整的配置路径、命令参考和最佳实践。

## 平台配置对比总览

不同平台的 Nginx 配置存在显著差异，了解这些差异有助于快速定位问题和优化配置。以下是两大主流平台的详细对比：

### ImmortalWrt vs. Synology DSM 配置差异表

| 特性 (Feature) | ImmortalWrt (OpenWrt/标准 Linux) | Synology DSM (系统/Web Station) |
| :--- | :--- | :--- |
| **主配置文件** | `/etc/nginx/nginx.conf` | `/usr/syno/share/nginx/nginx.mustache` |
| **自定义配置目录** | `/etc/nginx/conf.d/` 或 `/etc/nginx/sites-enabled/` | `/etc/nginx/` |
| **DSM/反向代理日志** | `/var/log/nginx/access.log` | **`/var/log/nginx/access.log`** |
| **Web Station 站点日志** | N/A (需手动配置) | `/var/log/nginx/webstation/<站点名称>/access.log` |
| **错误日志** | `/var/log/nginx/error.log` | `/var/log/nginx/error.log` |
| **服务重启命令** | `service nginx restart` 或 `/etc/init.d/nginx restart` | `synoservicectl --restart nginx` |
| **服务重载配置** | `nginx -s reload` | `synoservicectl --reload nginx` |
| **配置文件校验** | `nginx -t` | `nginx -t` |
| **Root 权限获取** | `sudo` 或 直接以 `root` 用户登录 | 必须使用 `sudo -i` 切换到 `root` 才能访问核心目录 |

## 配置要点说明

### ImmortalWrt 平台特点
- **标准 Linux 环境**：遵循传统的 Nginx 配置规范
- **配置灵活性高**：可直接修改主配置文件和自定义配置
- **日志管理简单**：统一的日志路径，便于维护

### Synology DSM 平台特点
- **双重配置体系**：系统级配置与 Web Station 配置分离
- **权限管理严格**：需要通过特定命令获取 root 权限
- **日志分级管理**：DSM 系统日志与 Web Station 站点日志分开存储

## 常用操作命令

### 配置文件编辑
```bash
# ImmortalWrt
vim /etc/nginx/nginx.conf
vim /etc/nginx/conf.d/custom.conf

# Synology DSM
sudo -i
vim /usr/syno/share/nginx/nginx.conf
vim /usr/local/etc/nginx/conf.d/custom.conf
```

### 服务管理
```bash
# ImmortalWrt
service nginx restart
service nginx reload
nginx -t

# Synology DSM
synoservicectl --restart nginx
synoservicectl --reload nginx
nginx -t
```

## 最佳实践建议

1. **配置备份**：修改配置前务必备份原始文件
2. **语法检查**：使用 `nginx -t` 验证配置语法
3. **日志监控**：定期检查错误日志，及时发现问题
4. **权限管理**：Synology 平台需特别注意权限切换
