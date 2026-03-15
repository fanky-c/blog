# MyBlog 技术指南

## 1. 项目概览

本项目是一个基于 `Hexo + aircloud 主题` 的个人博客。

- 源码分支：`dev/blog`
- 产物分支：`release/blog`
- 默认分支：`master`

核心流程如下：

1. 在 `source/` 中编写内容
2. 将静态文件生成到 `public/`
3. 通过 `hexo-deployer-git` 把生成结果部署到 GitHub Pages 对应分支

## 2. 技术栈

- 运行环境：`Node.js >= 14`
- 静态站点生成器：`hexo@7`
- 主题：本地主题 `themes/aircloud`
- 模板引擎：`ejs`
- Markdown 渲染器：`marked`
- 搜索索引：`hexo-generator-search`（生成 `search.json`）

## 3. 目录结构

```txt
.
├── _config.yml                  # Hexo 主配置文件
├── package.json                 # 脚本与依赖定义
├── scaffolds/                   # 文章 / 页面 / 草稿模板
├── source/                      # 站点源文件
│   ├── _posts/                  # 博客文章
│   ├── about/index.md           # About 页面
│   ├── tags/index.md            # 标签页
│   └── img/                     # 图片资源
├── themes/aircloud/             # 本地主题源码
│   ├── _config.yml              # 主题配置
│   ├── layout/                  # EJS 模板
│   └── source/                  # 主题的 CSS / JS / 静态资源
├── public/                      # 生成后的静态产物（不要手动修改）
└── .deploy_git/                 # Hexo 生成的部署临时仓库
```

## 4. 本地开发

### 4.1 安装依赖

```bash
npm install
```

### 4.2 启动本地服务

```bash
npm run dev
```

默认本地访问地址：`http://localhost:5001`

### 4.3 构建产物

```bash
npm run build
```

当前 `build` 脚本实际执行：

```bash
hexo clean && hexo generate
```

这样可以避免旧缓存带来的问题，并保证每次构建都是全量重新生成。

## 5. 内容开发

### 5.1 新建文章

```bash
npx hexo new "文章标题"
```

生成文件位置：`source/_posts/<title>.md`

### 5.2 文章 front-matter 示例

```yaml
---
title: 示例文章
date: 2026-03-08 10:00:00
tags:
  - js
  - node
---
```

### 5.3 新建自定义页面

```bash
npx hexo new page "about"
```

然后在文件的 front-matter 中设置布局，例如：

```yaml
---
layout: "about"
title: "About"
---
```

## 6. 主题开发

主要文件如下：

- 布局入口：`themes/aircloud/layout/layout.ejs`
- 首页列表：`themes/aircloud/layout/index.ejs`
- 文章页：`themes/aircloud/layout/post.ejs`
- 全局样式：`themes/aircloud/source/css/aircloud.css`
- 全局脚本：`themes/aircloud/source/js/index.js`

如果你修改了主题文件，建议执行 `npm run build`，并检查 `public/` 中生成的文件是否符合预期。

## 7. 关键配置

### 7.1 Hexo 主配置（`_config.yml`）

- 站点元信息：`title`、`description`、`keywords`、`author`
- 路由配置：
  - `url`
  - `root`
  - `permalink`
- 分页配置：`per_page`
- 主题配置：`theme: aircloud`
- 部署目标：
  - `deploy.type: git`
  - `deploy.repo`
  - `deploy.branch: release/blog`

### 7.2 搜索配置

```yml
search:
  path: search.json
  field: post
```

前端搜索功能依赖构建时生成的 `public/search.json`。

## 8. 构建与部署

### 8.1 仅构建

```bash
npm run build
```

### 8.2 部署

```bash
npm run deploy
```

部署命令内部实际执行：

```bash
hexo g -d
```

它会先生成静态文件，再把部署产物推送到配置好的 Git 分支。

## 9. 常见问题

### 9.1 生成的 HTML / 静态文件为空（0 字节）

常见现象：

- `public/*.html` 或 `public/js/*` 文件大小为 `0`
- 页面在浏览器中显示为空白

解决方式：

1. 使用兼容的现代依赖版本（如 `hexo@7` 及其匹配插件）
2. 确保 Node 版本为 `>=14`
3. 始终执行一次干净构建：

```bash
npm run build
```

4. 检查生成文件大小：

```bash
wc -c public/index.html public/js/index.js public/css/aircloud.css public/search.json
```

### 9.2 搜索面板没有结果

请依次检查：

1. 是否已安装 `hexo-generator-search`
2. `_config.yml` 中是否存在 `search` 配置
3. `public/search.json` 是否存在且内容非空

## 10. 协作约定

- 不要手动修改 `public/` 中的文件
- 文章资源统一放在 `source/img/` 下
- 文章 front-matter 应保持完整（`title/date/tags`）
- 部署前至少完成以下本地验证：
  - `npm run build`
  - `npm run dev`

## 11. 建议发布检查清单

1. 安装依赖：`npm install`
2. 检查配置变更（`_config.yml`、主题配置）
3. 执行构建：`npm run build`
4. 冒烟验证：首页、文章页、标签页、About、搜索功能
5. 执行部署：`npm run deploy`
