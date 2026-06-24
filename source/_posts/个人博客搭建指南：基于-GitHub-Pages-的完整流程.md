---
title: 个人博客搭建指南：基于 GitHub Pages 的完整流程
tags:
  - 个人博客搭建指南
series:
  - 个人博客搭建指南
categories:
  - 个人博客搭建指南
author: AuroraHiker
date: 2026-06-24 19:53:28
---


本文记录从零开始，基于 GitHub Pages + Hexo 搭建个人博客/网站的完整过程。笔者使用的是 Windows 10 系统，因此本教程完全适用于 Windows 10/11 用户。其他操作系统在操作细节上可能略有差异，但整体部署流程一致。

---

# 1.基础软件安装

下表列出了搭建过程中必需的软件，后续会详细介绍其安装与配置方法。表格中同时给出了笔者所使用的版本，供您参考。

| 软件 | 版本 | 官网 |
| - | - | - |
| Node.js | 24.16.0.0 | https://nodejs.org/zh-cn |
| git | 2.50.1.windows.1 | https://git-scm.com/install/ |

## 1.1 git的安装

- 打开[Git官网](https://git-scm.com/install/)
- 下载适合您系统的安装包。
    ![Git安装包下载](./个人博客搭建指南/git安装包下载.png)
- 双击安装包，按默认选项执行即可完成安装。
- 安装完成后，会出现Git Bash。
    ![Git bash](./个人博客搭建指南/git安装完成后.png)
- 启动 Git Bash 并输入命令 `git --version`，若输出类似 `git version 2.50.1.windows.1` 的版本信息，则说明 Git 已成功安装。

## 1.2 Node.js的安装

- 打开 [Node.js 官网](https://nodejs.org/zh-cn) 。
    ![Node.js 官网](./个人博客搭建指南/Nods.js安装包下载1.png)
- 选择安装包对应的系统和版本，支持命令行方式和安装包方式安装。
    ![Node.js 安装](./个人博客搭建指南/Nods.js安装.png)
  - **命令行方式**：复制页面中提供的命令，在终端中运行即可。  
    **注意**：若采用命令行方式，建议优先选择带有 LTS (长期支持) 标识的版本，该版本通常更稳定。
  - **安装包方式**：下载对应安装包，双击并按默认选项完成安装。

---

# 2. GitHub 配置

## 2.1 GitHub 账户注册

> **注意**：国内访问 GitHub 可能不稳定，建议借助加速工具。此前博客中推荐的免费工具 Watt Toolkit 可实现长期稳定连接，详见：[Windows效率神器推荐——Wox、Markdown编辑器与GitHub加速器配置指南]({% post_link Windows效率神器推荐-Wox-Markdown编辑器与GitHub加速器配置指南 %})。

- 打开[GitHub官网](https://github.com/)
    ![GitHub官网](./个人博客搭建指南/GitHub注册.png)
- 填写注册信息 (用户名需独一无二)，并完成邮箱验证。请务必使用本人长期使用的邮箱，这不仅用于注册验证，后续博客部署出现问题时，GitHub 也会通过该邮箱发送通知。

## 2.2 SSH 密钥配置

建议使用 SSH 协议连接 GitHub，以简化推送与拉取操作，避免反复输入密码。配置步骤如下：

### 2.2.1 设置本地 Git 用户信息

在 Git Bash 中执行以下命令，将用户名和邮箱设置为 GitHub 注册信息：

```bush
git config --global user.name "GitHub 用户名"
git config --global user.email "GitHub 邮箱"
```

### 2.2.2 生成密钥对

执行以下命令生成密钥：
  
```bush
    ssh-keygen -t rsa -C "your_email@example.com"
```

 **参数说明**

| 参数 | 含义解释 |
| :--- |:--- |
| `-t` | 指定密钥的**加密算法**。 |
| `-C` | 给密钥文件添加一条**注释标签**。这个字符串会附加在公钥文件的末尾，**仅用于识别用途**（例如区分不同设备或用途），不影响加密验证过程。通常填写 GitHub 邮箱，方便在账户管理页面辨识。 |
   
**交互过程**
    
按下回车后，终端会依次提示：

1. **保存路径**  
    ```text
    Enter file in which to save the key (/home/user/.ssh/id_rsa):
    ```

    - 直接按 **回车**，则使用默认路径：`~/.ssh/id_rsa`（私钥）和 `~/.ssh/id_rsa.pub`（公钥）。  
    - 若想自定义路径（例如多密钥管理），可输入路径后回车。

 2. **密码短语**  
    ```text
    Enter passphrase (empty for no passphrase):
    ```
    
    - 可输入密码保护私钥（更安全，但每次使用需输入）。
    - 直接按 **回车** 则为空密码（便于自动化，但需妥善保管私钥文件）。

**生成的文件说明**

| 文件 | 路径（默认） | 作用 |
| :--- | :--- | :--- |
| **私钥** | `~/.ssh/id_rsa` | **绝不可泄露**。用于解密和签名，认证时证明你的身份。 |
| **公钥** | `~/.ssh/id_rsa.pub` | **可以公开**。内容需添加到 GitHub（或服务器）的 `~/.ssh/authorized_keys` 中，用于验证持有对应私钥的你。 |

> 公钥内容以 ssh-rsa AAAA... 开头，末尾附有注释（邮箱）。

### 2.2.3 将公钥添加到 GitHub

生成密钥后，必须将公钥添加到 GitHub 账户，才能使用 SSH 协议 (避免每次输入密码或处理 SSL 证书问题) 。

- 用记事本打开公钥文件，复制全部文本
- 登录 GitHub → 右上角头像 → **Settings** → **SSH and GPG keys** → **New SSH Key**。
  ![GitHub SSH配置](./个人博客搭建指南/GitHubSSH密钥配置.png)

> **注意事项**
>
> - **权限保护**：私钥文件 (id_rsa) 默认权限为 600 (仅本人读写) ，切勿修改为更宽松的权限，否则 SSH 会拒绝使用。
> 
> - **密码短语**：若设置了密码短语，后续 `git push` 等操作会提示输入，可使用 `ssh-agent` 缓存密码以减轻输入负担。
> - **公钥可以随意展示，但私钥必须像密码一样保管好**

## 2.3 创建 GitHub Pages 仓库
### 2.3.1 初始化仓库
- 登录 GitHub 账户。
- 点击页面右上角的 **+** 号，选择 **New repository**。

- **填写仓库信息**
    | 字段 | 填写内容 | 说明 |
    | :--- | :--- | :--- |
    | **Repository name** | 例如 `用户名.github.io` | **必须**符合 Pages 命名规则（项目站点可任意名称，但建议包含 `.github.io` 后缀以表明用途）。 |
    | **Description** (可选) | 简短描述，如 “我的 Hexo 博客” | 仅用于展示，不影响功能。 |
    | **Public / Private** | 选择 **Public** | GitHub Pages 仅支持公共仓库 (免费计划下) ，私有仓库需付费。 |
    | **Initialize this repository with** | 建议勾选 **Add a README file** | 便于初始克隆和说明；若留空，空仓库将暂无法启用 Pages 服务。也可按需添加 .gitignore (Node 模板) 和许可证。 |

- 点击 **Create repository** 完成创建。

### 2.3.2 启用 GitHub Pages 服务

- 进入仓库主页，点击上方 Settings 选项卡。
- 左侧导航栏中选择 **Pages**。
- 在 **Branch** 下拉菜单中，选择 main 分支，并保留根目录 / (或 /docs，但通常用根目录) 。
- 点击 **Save** 保存。

等待片刻，页面刷新后会显示绿色提示框，其中包含你的博客访问地址，例如：
> Your site is published at `https://aurorahiker.github.io/`

## 2.4 将仓库克隆到本地

打开终端，进入你存放项目的目录 (如 workspace) ，执行克隆命令。推荐使用 SSH 方式 (无需每次输入密码) ：

- HTTPS 
    ```bash
    git clone https://github.com/你的用户名/你的仓库名.git my-blog
    ```

- SSH
    ```bash
    git clone git@github.com:你的用户名/你的仓库名.git my-blog
    ```

其中`my-blog` 是本地文件夹名，可自定义。

---

# 3. Hexo 的配置

## 3.1 Hexo安装

运行下列代码安装hexo：
```
npm install -g hexo-cli
```
详细的、其他系统的安装说明见[Hexo官网](https://hexo.io/zh-cn/docs/)

## 3.2 本地仓库初始化

- 进入本地仓库

    ```bash
    cd my-blog
    ```

- **注意**：因为仓库已有 README 和 `.git`，而Hexo初始化不允许目录中有其他文件，所以需要先把README 和 `.git`移动至其他文件夹中，初始化完成后再移回来。

- 初始化
  
    ```bash
    hexo init # 初始化
    npm install    # 安装组件
    ```

- 安装部署插件：

    ```bash
    npm install hexo-deployer-git --save
    ```

## 3.3 配置 Hexo 部署信息

编辑博客根目录下的 `_config.yml` 文件(注意不是主题内的) ，修改以下配置：

- 网站的基本信息
    
    ```yaml
    title: 你的博客标题          
    subtitle: '你的副标题'        
    description: '博客简短描述'   # 用于 SEO
    keywords:                    # 网站关键词
    - 关键词1
    - 关键词2
    author:                      # 默认作者名，会显示在文章作者处
    language: zh-CN              # 网站语言，中文设为 zh-CN
    timezone: 'Asia/Shanghai'    # 时区，确保时间正确
    ```

- URL 网址设置 (最重要)
  
    这是确保所有页面链接正确的基础，必须准确修改。
    ```yaml
    # URL
    ## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
    url: https://用户名.github.io   
    root: /                                 # 因为这是用户/组织站点，根目录就是 /
    permalink: :year/:month/:day/:title/    # 文章链接格式，保持默认即可
    ```

- 部署设置
  
    ```yaml
    # Deployment
    deploy:
    type: git
    repo: 你的仓库地址  
    branch: main        # GitHub Pages 默认使用 main 分支
    ```
    > 如果你使用 SSH 地址 (推荐) ，则 `repo` 填写 `git@github.com:用户名/仓库名.git`。

### 3.4 创建第一篇博文

- 创建新页面
    ```bash
    hexo new page "页面名称"
    ```
- 打开新建的md文件，写入内容
    ```md
    Hello World!
    ```
- 生成、清理、部署
    ```bash
    hexo clean && hexo d -g
    hexo s # 本地预览
    ```
- 提交更改并推送到远程仓库
- 打开你的博客网址，就能看见新的更改啦！

---

# 5. 写在最后

从注册 GitHub 到在浏览器里看到自己的站点，整个过程看似步骤繁多，但拆解开其实就是“工具安装 → 密钥配对 → 仓库创建 → 本地部署”这条主线。每完成一步，离那个属于自己的小空间就近了一点，这种逐步推进的踏实感，大概是搭建博客独有的乐趣。

更重要的是，这一套流程并不只是为博客服务——熟悉了 Git 和 Hexo 的工作流，往后任何静态站点的管理都会顺手很多。遇到报错也别急，顺着提示去查文档、改配置，多试几次，那些红字就变成你的经验值了。

如果你在跟着操作时发现了更省力的办法，发现了本文未曾设想的bug，欢迎通过邮箱 aurorahiker@163.com 和笔者分享。你的一个小提示，说不定就能让下一位读者少绕一大段路。

至于博客外观和交互细节，笔者打算在下一篇里专门聊 **Fluid 主题的完整配置**——从导航栏定制、评论系统接入，到响应式优化，都会一步步写清楚。感兴趣的话，记得来看。

---

**参考资源**

- [GitHub Pages 官方文档](https://docs.github.com/zh/pages)
- [Hexo 官方部署文档](https://hexo.io/zh-cn/docs/github-pages)
- [Git 官方教程](https://git-scm.com/book/zh/v2)