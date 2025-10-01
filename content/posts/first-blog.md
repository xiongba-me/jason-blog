---
title: '使用Hugo + Cloudflare 搭建博客'
date: '2024-04-22'
tags: ['blog', 'hugo', 'PaperMod', 'Cloudflare']
categories: ["blog"]
description: "自动更新的简单博客系统"
draft: false
---
## 目录
1. [系统要求](#系统要求)
2. [Hugo 安装与配置](#hugo-安装与配置)
3. [PaperMod 主题配置（可选）](#papermod-主题配置)
4. [创建和管理博客内容](#创建和管理博客内容)
5. [Cloudflare Pages 部署](#cloudflare-pages-部署)
6. [高级配置](#高级配置)
7. [常见问题与解决方案](#常见问题与解决方案)
---

## 系统要求

### 准备工作
- **安装Hugo** 
- **GitHub 账户** (用于代码托管，giscus评论系统)
- **Cloudflare 账户** (用于部署博客)
- **托管在Cloudflare的域名** (用于访问博客)

---

## Hugo 安装与配置

### 1. 安装 Hugo

#### macOS (使用 Homebrew)
```bash
# 安装 Homebrew (如果未安装)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装 Hugo Extended
brew install hugo
```

### 2. 验证安装
```bash
hugo version
```

### 3. 初始化新应用
```bash
# 初始化新应用
hugo new site jason-blog
cd jason-blog

# 初始化 Git 仓库
git init
```

---

## PaperMod 主题配置

### 1. 安装 PaperMod 主题

#### 直接下载，放入jason-blog的themes目录下
```bash
# 下载并解压
wget https://github.com/adityatelange/hugo-PaperMod/archive/refs/heads/master.zip
unzip master.zip -d themes/
mv themes/hugo-PaperMod-master themes/PaperMod
# 还有种方法是采用git命令下载为子模块， 为什么要直接下载源码，因为有几个地方要改代码。
```
- 修改 [PaperMod/layouts/partials/comments.html](https://github.com/xiongba-me/jason-blog/blob/master/themes/PaperMod/layouts/partials/comments.html) 文件，填入[giscus](https://giscus.app/zh-CN)生成的内容

```javascript
<script src="https://giscus.app/client.js"
        data-repo="[在此输入仓库]"
        data-repo-id="[在此输入仓库 ID]"
        data-category="[在此输入分类名]"
        data-category-id="[在此输入分类 ID]"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
```
### 2. 基础配置文件 (hugo.yaml)

创建或修改根目录下的 `hugo.yaml` 文件：
- 参考[hugo.yaml](https://github.com/xiongba-me/jason-blog/blob/master/hugo.yaml)

---

## 创建和管理博客内容

### 1. 创建文章

```bash
# 创建新文章
hugo new posts/my-first-post.md
```

### 2. 文章模板 (Front Matter)

```markdown
---
title: "我的第一篇博客"
date: 2024-01-01T12:00:00+08:00
draft: false
tags: ["Hugo", "博客"]
categories: ["技术"]
author: "您的名字"
description: "这是我的第一篇博客文章"
---

# 文章内容

这里是您的文章内容...
```

### 3. 创建必要页面

#### 搜索页面
创建 `content/search.md`：
```markdown
---
title: "搜索"
layout: "search"
---
```

#### 归档页面
创建 `content/archives.md`：
```markdown
---
title: "归档"
layout: "archives"
url: "/archives/"
summary: archives
---
```

### 4. 本地预览

```bash
# 启动开发服务器
hugo server -D

# 访问 http://localhost:1313 查看效果
```

---

## Cloudflare Pages 部署

### 1. 准备代码仓库

#### 推送代码到 GitHub
```bash
# 添加远程仓库
git remote add origin https://github.com/yourusername/your-blog.git

# 提交代码
git add .
git commit -m "Initial commit"
git push -u origin main
```

### 2. 连接 Cloudflare Pages

1. **登录 Cloudflare Dashboard**
   - 访问 [dash.cloudflare.com](https://dash.cloudflare.com)
   - 登录您的账户

2. **创建 Pages 项目**
   - 点击侧边栏的 "Workers 和 Pages"
   - 点击 "创建应用"
   - 选择 "导入存储库(从导入现有 Git 存储库开始。)"

3. **连接 Git 仓库**
   - 选择 GitHub
   - 授权 Cloudflare 访问您的 GitHub 账户
   - 选择您的博客仓库


### 3. 构建设置

以下是在 Cloudflare Pages 中配置 Hugo 项目的截图示例：

![Cloudflare Pages 构建配置](/images/post/gjpz.png)

配置说明：
- **生产分支**: 通常选择 `main` 或 `master`
- **框架预设**: Hugo
- **构建命令**: `hugo`
- **构建输出目录**: `public`
- **自动部署**: `已启用`


## 高级配置

### 1. 自定义域名

1. **在 Cloudflare Pages 中设置**
   - 进入您的 Pages 项目
   - 点击 "Custom domains"
   - 添加您的域名

2. **DNS 配置**
   - 添加 CNAME 记录指向 `your-project.pages.dev`
---

## 常见问题与解决方案

### 1. 主题相关问题

**Q: 搜索功能不工作**
- 确保在 `hugo.yaml` 中配置了正确的输出格式
- 检查 `content/search.md` 是否存在

### 2. 部署相关问题

**Q: Cloudflare Pages 构建失败**
- 查看构建日志


### 3. 内容管理

**Q: 文章不显示**
- 检查 `draft: false`
- 确认文件路径正确
- 验证 Front Matter 格式

**Q: 图片不显示**
- 将图片放在 `static/images/` 目录
- 使用相对路径引用

---

## 总结

通过以上步骤，您已经成功搭建了一个基于 Hugo + PaperMod + Cloudflare Pages 的博客系统。这个配置具有以下优势：

- ✅ **快速加载**: Hugo 生成静态网站，Cloudflare 全球 CDN 加速
- ✅ **SEO 友好**: 完整的 SEO 配置和优化
- ✅ **移动端适配**: PaperMod 主题响应式设计
- ✅ **免费托管**: Cloudflare Pages 提供免费托管服务
- ✅ **易于维护**: Git 版本控制，自动部署

如果您在搭建过程中遇到任何问题，请参考官方文档或在相关社区寻求帮助。

---
## 相关链接

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [PaperMod 主题文档](https://github.com/adityatelange/hugo-PaperMod)
- [Cloudflare Pages 文档](https://developers.cloudflare.com/pages/)

---
