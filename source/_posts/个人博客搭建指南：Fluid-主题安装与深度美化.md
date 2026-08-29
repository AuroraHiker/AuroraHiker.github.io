---
title: 个人博客搭建指南：Fluid 主题安装与深度美化
tags:
  - 个人博客搭建指南
series:
  - 个人博客搭建指南
categories:
  - 个人博客搭建指南
author: AuroraHiker
date: 2026-07-14 03:41:32
---

在{% post_link 个人博客搭建指南：基于-GitHub-Pages-的完整流程 %}中，我们完成了 Hexo 博客从零到部署的全过程。博客能正常发布和浏览了，但外观还是默认的 landscape 主题，功能不够全面，也略显单调。在这篇博客，笔者将带领大家一起来给博客“装修”。

Fluid 是一款基于 Material Design 风格的 Hexo 主题，由 Fluid-dev 团队开发维护。它设计简洁、配置灵活，内置了暗色模式、访问统计、本地搜索等常用功能。截至本文撰写时，Fluid 最新版本为 1.9.9。

> **说明**：本文基于上一篇搭建好的 Hexo 博客继续操作，默认您已完成 Hexo 安装、GitHub Pages 配置及本地仓库初始化。若尚未完成，请先参考上一篇指南。

---

# 1. 安装 Fluid 主题

## 1.1 环境要求

Fluid 主题要求 Hexo 版本不低于 5.0.0，Node.js 版本不低于 10.13.0。您可以在终端中执行以下命令确认版本：

```bash
hexo -v
node -v
```

若 Hexo 版本低于 5.0.0，可通过以下命令升级：

```bash
npm install hexo@latest --save
```

## 1.2 安装主题

Fluid 提供两种安装方式 (npm 与手动)，推荐使用 npm 方式。

**npm 安装 (推荐)**

进入博客根目录，执行：

```bash
npm install --save hexo-theme-fluid
```

> 即使此前在另一个博客中已安装过 Fluid 主题，在新站点根目录下仍需重新执行安装指令。Hexo 基于当前工作目录加载依赖，Node.js 会从当前博客目录下的 node_modules 或 themes 文件夹中查找 hexo-theme-fluid 包，无法跨目录复用。

## 1.3 启用主题


1. 编辑 Hexo 根目录下的 `_config.yml`，修改以下配置：

```yaml
theme: fluid
```

2. 在博客目录下创建 `_config.fluid.yml` 文件，[官方配置指南](https://hexo.fluid-dev.com/docs/guide/#%E5%85%B3%E4%BA%8E%E6%8C%87%E5%8D%97)中说明需要将主题默认配置的全部或部分内容复制到该文件中。但笔者在 Hexo 8.1.2 环境下测试发现，无需复制任何内容，主题会自动读取默认配置。由于笔者不清楚哪些版本的组合支持此特性，欢迎读者尝试并反馈。

至此，Fluid 主题已完成安装与启用。接下来开始进行各项配置与美化。

---

# 2 配置文件说明

理解 Fluid 的配置机制是后续所有操作的基础。

- **主题默认配置**：`themes/fluid/_config.yml` —— 存放主题默认设置，**不建议直接修改**。
- **用户自定义配置**：`_config.fluid.yml` —— 位于博客根目录，优先级更高，更新主题时不会被覆盖。仅当该项在自定义配置中未定义时，才回退使用默认值。

**所有后续修改均在 `_config.fluid.yml` 中进行**。

> **提示**：
> 
> 1.只要配置存在于 `_config.fluid.yml` 中，就具有高优先级，修改原 `_config.yml` 是无效的。每次更新主题后，需留意更新说明，可能需要对 `_config.fluid.yml` 同步修改。
>
> 2.所有配置中引用的图片或文件，其根目录均为 source 文件夹。

---

# 3. 关于页配置

关于页是博客中展示作者、博客信息的独立页面，需要单独创建并配置。

## 3.1 创建关于页

Fluid 的关于页需手动创建：

```bash
hexo new page about
```
然后编辑 `/source/about/index.md`，添加 `layout: about` 属性：

```markdown
---
title: about
layout: about
---
这里写关于页的正文，支持 Markdown 和 HTML。
```

> 注意：layout: about 必须存在且不能修改为其他值，否则无法显示头像、简介等主题样式。

## 3.2 关于页美化

以下是与关于页相关的所有配置项，在 `_config.fluid.yml` 中修改：

```yaml
# 根目录位于 source 文件夹
about:
  enable: true
  banner_img: /img/default.png        # 关于页的背景图
  banner_img_height: 60
  banner_mask_alpha: 0.3
  avatar: /img/avatar.png             # 作者头像
  name: "Fluid"
  intro: "An elegant theme for Hexo"  # 简介
  # 更多图标可从 https://hexo.fluid-dev.com/docs/icon/ 查找
  # `class` 代表图标的 css class；添加 `qrcode` 后，图标变为悬浮二维码
  icons:
    - { class: "iconfont icon-github-fill", link: "https://github.com", tip: "GitHub" }
    - { class: "iconfont icon-douban-fill", link: "https://douban.com", tip: "豆瓣" }
    - { class: "iconfont icon-wechat-fill", qrcode: "/img/favicon.png" }
```

---

# 4. 基础配置

## 4.1 网站基本信息

```yaml
# 根目录位于 source 文件夹
# 网站图标 (浏览器标签页)
favicon: /images/logo.png
# 苹果设备图标
apple_touch_icon: /images/logo.png
# 标签页标题分隔符
tab_title_separator: " - "
```

## 4.2 导航栏配置

```yaml
navbar:
  # 导航栏左侧标题
  blog_title: "你的博客名称"
  # 导航菜单
  menu:
    - { key: "home", link: "/", icon: "iconfont icon-home-fill" }
    - { key: "archive", link: "/archives/", icon: "iconfont icon-archive-fill" }
    - { key: "category", link: "/categories/", icon: "iconfont icon-category-fill" }
    - { key: "tag", link: "/tags/", icon: "iconfont icon-tags-fill" }
    - { key: "about", link: "/about/", icon: "iconfont icon-user-fill" }
```

其中 `key` 用于关联多语言配置，`icon` 对应图标库中的图标类名。Fluid 使用 Iconfont 提供图标支持，可以通过修改 `icon` 值更换菜单图标。

## 4.3 首页配置

```yaml
index:
  # 首页 Banner 图片
  banner_img: /img/default.png
  banner_img_height: 100
  # Banner 遮罩透明度
  banner_mask_alpha: 0.3
  # 首页副标题
  slogan:
    enable: true
    text: "副标题文本" # 为空则使用默认值 (原配置文件的副标题文本)
  # 自动截取文章摘要
  auto_excerpt:
    enable: true
  # 文章元信息显示
  post_meta:
    date: true
    category: true
    tag: true
```

> 建议 Banner 图片尺寸为 1920×1080。

## 4.4 页脚配置

```yaml
footer:
  # 页脚内容（支持 HTML）
  content: '
    <a href="https://hexo.io" target="_blank">Hexo</a> |
    <a href="https://github.com/fluid-dev/hexo-theme-fluid" target="_blank">Fluid</a>
  '
  # 访问统计
  statistics:
    enable: true
    source: "busuanzi"  # 可选：busuanzi / leancloud
```

`source` 可选用不蒜子 (busuanzi，不需要额外配置) 或 LeanCloud (需额外配置) 作为统计来源。

---

# 5. 本地搜索配置

Fluid 已集成 `hexo-generator-search` 插件，默认在根目录生成并使用 `local-search.xml`。若已安装其他搜索插件，需关闭以避免生成多余的索引文件。

如未安装，执行：

```bash
npm install hexo-generator-search --save
```

然后在 `_config.fluid.yml` 中启用：

```yaml
local_search:
  enable: true
```

---

# 6. 高级美化技巧

## 6.1 Banner 图片与视频背景

Fluid 支持为不同页面配置独立的 Banner 图片：

```yaml
# 根目录位于 source 文件夹
index:
  banner_img: /img/index_banner.jpg

archive:
  banner_img: /img/archive_banner.jpg

category:
  banner_img: /img/category_banner.jpg

tag:
  banner_img: /img/tag_banner.jpg
```

如需更动态的效果，可参考社区方案为首页添加随机视频背景。

## 6.2 友链配置

Fluid 原生支持友链功能。在 `_config.fluid.yml` 中配置：

```yaml
links:
  enable: true

  # 友链分组
  groups:
    - group_name: "好友博客"
      items:
        - title: "博客名称"
          intro: "博客简介"
          link: "https://example.com"
          avatar: "/img/avatar.jpg" # 根目录位于 source 文件夹
```

然后在导航菜单中添加友链入口：

```yaml
navbar:
  menu:
    - { key: "links", link: "/links/", icon: "iconfont icon-link-fill" }
```

---

# 7. 更新主题

Fluid 主题持续迭代，定期更新可获得新功能和 bug 修复。

**npm 安装方式**：

```bash
npm update --save hexo-theme-fluid
```

**Release 压缩包方式**：

先将原文件夹重命名为 `fluid-bkp` 作为备份，然后下载最新 release 解压并重命名为 `fluid`。

> **注意**：每次更新后，需留意 release 说明中的配置变更，手动同步修改 `_config.fluid.yml`。

---

从安装主题到调完每一个细节，博客从“能用”变成了“好用”。这个过程其实没有太多高深的技术——翻文档、改配置、刷新页面看效果，反复几轮，那些陌生的配置项就变成了顺手的东西。

Fluid 的配置文档写得非常详细，几乎每个配置项都有注释说明。遇到拿不准的，先去[官方配置指南](https://hexo.fluid-dev.com/docs/guide/)翻一翻，八成能找到答案。如果发现了更优雅的美化方案，或者踩了文档里没写到的坑，欢迎通过邮箱 aurorahiker@163.com 和我交流。

博客的外观只是第一步，后面还可以折腾的内容还有很多——比如接入评论、图床、优化加载速度、配置 SEO等 ，后续我会继续研究并记录，慢慢把这一整套流程补齐。

---

## 参考资料

- [Fluid 官方文档](https://fluid-dev.github.io/hexo-fluid-docs)
- [Fluid 配置指南](https://hexo.fluid-dev.com/docs/guide/)
- [Fluid GitHub 仓库](https://github.com/fluid-dev/hexo-theme-fluid)