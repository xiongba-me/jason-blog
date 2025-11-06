
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# 项目概述

这是一个基于 Hugo 框架构建的博客项目，使用了 `PaperMod` 主题。

## 高层代码架构和结构

*   **hugo.yaml**: 项目的主要配置文件，定义了网站的全局设置、主题、菜单、参数等。
*   **content/**: 包含网站的所有内容，如博客文章 (`posts/`)、归档页面 (`archives.md`) 和搜索页面 (`search.md`)。
*   **static/**: 存放静态资源，例如图片 (`images/`)。
*   **themes/PaperMod/**: 包含 `PaperMod` 主题的模板、样式和脚本等文件。

## 常用的构建、lint、测试和运行命令

### 运行开发服务器
```bash
hugo server
```
此命令将在本地启动一个开发服务器，通常在 `http://localhost:1313` 端口运行，方便预览网站。

### 构建网站
```bash
hugo
```
此命令将生成静态网站文件到 `public/` 目录中，用于部署。

### 其他
*   **Linting**: 未发现明确的 linting 配置或命令。
*   **测试**: 未发现明确的测试配置或命令。

## 未找到的相关文件

*   在项目根目录未找到 `README.md` 文件（仅在主题目录找到）。
*   未找到 `.cursor/rules/` 目录。
*   未找到 `.cursorrules` 文件。
*   未找到 `.github/copilot-instructions.md` 文件。