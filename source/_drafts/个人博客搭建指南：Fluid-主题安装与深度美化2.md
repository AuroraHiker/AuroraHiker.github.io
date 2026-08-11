---
title: 个人博客搭建指南：Fluid-主题安装与深度美化2
tags:
---
## 4.4 配置 Hexo Fluid

将 Giscus 生成的配置填入 `_config.fluid.yml`：

```yaml
# Giscus 评论系统
giscus:
  repo: "用户名/仓库名"
  repo-id: "你的 repo-id"
  category: "Announcements"
  category-id: "你的 category-id"
  mapping: "pathname"
  strict: 1
  reactions-enabled: 1
  emit-metadata: 0
  input-position: "top"
  theme-light: "light"
  theme-dark: "dark"
  lang: "zh-CN"

# 启用评论
comments:
  enable: true
  type: giscus
```

保存配置后，执行 `hexo clean && hexo d -g` 部署，即可在博客文章底部看到评论区域。

---

# 5. 访问统计配置

Fluid 内置了网页访问统计功能。支持不蒜子（busuanzi）和 LeanCloud 两种统计源。

## 5.1 不蒜子统计（推荐）

不蒜子是国内常用的免费站点统计服务，配置简单，无需注册。在 `_config.fluid.yml` 中修改：

```yaml
footer:
  statistics:
    enable: true
    source: "busuanzi"
```

部署后，页脚会自动显示站点总访问量和总访客数。

## 5.2 LeanCloud 统计

如需更详细的文章阅读量统计，可配置 LeanCloud：

1. 注册 [LeanCloud](https://leancloud.cn/) 并创建应用。
2. 在 `_config.fluid.yml` 中配置：

```yaml
footer:
  statistics:
    enable: true
    source: "leancloud"

leancloud_visitors:
  enable: true
  app_id: "你的 App ID"
  app_key: "你的 App Key"
  server_url: "你的服务器地址"  # 国内用户需绑定自定义域名
```
---

# 7. 自定义 CSS 样式

Fluid 原生支持 `custom_css` 字段，允许无侵入地自定义样式。

## 7.1 使用方法

1. 在 `source/` 目录下创建 `css/` 文件夹（如不存在）。
2. 在 `source/css/` 中创建自定义 CSS 文件，例如 `custom.css`。
3. 在 `_config.fluid.yml` 中添加：

```yaml
custom_css:
  - /css/custom.css
```

> **注意**：路径是相对于 `source/` 的，开头**不能**带 `/`。

## 7.2 常见美化示例

**修改滚动条样式**：在 `source/css/` 下创建 `ScrollBar.css`，添加滚动条样式代码，然后在 `custom_css` 中引入。

**标题颜色渐变**：在 `source/css/` 下创建 `TitleGradient.css`，实现标题渐变效果。

**暗色模式下的字体样式**：通过 `custom_css` 调整暗色模式下的加粗字体和斜体样式。