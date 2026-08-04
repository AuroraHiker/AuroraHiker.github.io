---
title: 将 Codex 智能体纳入麾下：Windows 10 本地化部署实战
tags:
  - 智能体
  - 开发者工具箱
  - Codex
series: 开发者工具箱
categories:
  - 开发者工具箱
author: AuroraHiker
---

欢迎来到 AuroraHiker 的技术博客！本文面向**从未接触过编程智能体的读者**，以 Codex 为例，介绍如何在 Windows 10 中部署一个能够协助我们处理本地文件和项目的智能体助手。

本文使用的 API 来自[西农er's GPT](https://gpt.nwafu-ai.cn/)。截至本文完成时，平台公布的 API 地址为 `https://api.nwafu-ai.cn`，并为相关模型提供了 `Codex` 令牌分组。该平台需要使用教育邮箱注册，平台规则、模型名称和费用可能变化，请以实际页面为准。

> **“本地化部署”说明：** 本文所说的本地化，是指 Codex 程序安装在自己的电脑上，并在获得许可后操作本地项目文件。负责推理的 AI 模型仍然位于远程服务器，Codex 需要通过 API 联网调用，并不是将大模型完整下载到电脑中。

智能体可以理解为“接入双手的 AI 模型”。它不仅能读取文件、浏览网页、思考和回答问题，还能把模型的判断转换成文件操作和命令。能力更强也意味着风险更高，因此使用时务必注意：

1. 看清 Codex 准备执行的操作，再决定是否允许。
2. 提前备份需要处理的项目。初次使用时，建议复制一份项目到新文件夹，并以新文件夹作为工作目录。

---

## 0. 准备

### 0.1 模型的 API 是什么

AI 模型通常运行在远程服务器中，我们安装的 Codex 则运行在自己的电脑上。两者需要通过 API 进行通信：

```text
你向 Codex 下达任务
        ↓
Codex 通过 API 地址发送请求
        ↓
远程 AI 模型分析任务并返回判断
        ↓
Codex 在获得许可后操作本地文件
```

其中有三个概念需要认识：

| 名称 | 作用 | 可以怎样理解 |
| :--- | :--- | :--- |
| API | 规定软件与模型怎样传递请求和回答 | Codex 与模型之间的通信通道 |
| API 地址 | 告诉 Codex 应该连接哪台服务器 | 通信通道的网络地址 |
| API Key | 证明账户身份并记录调用费用 | 使用 API 的个人通行证 |

API Key 与账户余额直接相关，任何得到它的人都可能消耗你的额度，因此不能发到群聊、写进公开博客或上传到 GitHub。

### 0.2 注册账户并创建 API Key

打开[西农er's GPT](https://gpt.nwafu-ai.cn/)，使用教育邮箱注册并完成验证 (官方使用指南 [https://blog.nwafu-ai.cn/archives/1705731294486])。截至本文完成时，每个新账户提供 2 元试用额度，具体活动规则以平台实时页面为准。

登录后从右上角的用户头像点击‘个人资料’或‘钱包’进入个人主页，点击‘API密钥’，创建一个新令牌：

![API密钥创建界面](智能体部署/创建API密钥.png)

1. 令牌名称可以填写 `codex-win10`；
2. 根据需要设置有效期和额度上限；
3. 创建后复制生成的 API Key，并妥善保存。

平台生成的 API Key 通常以 `sk-` 开头。下文中的 `sk-请替换为你自己的API-Key` 只是占位符，实际操作时应替换为自己刚刚创建的密钥。

> **安全提醒：** 如果 API Key 曾经完整出现在聊天记录、截图或公开网页中，应立即在平台删除该密钥并重新生成。仅仅把已经发布的内容打码，并不能保证旧密钥没有被他人复制。

## 1. Codex 安装配置

### 1.1 安装

Codex 主要有两种本地使用方式：

| 版本 | 操作方式 | 适合人群 |
| :--- | :--- | :--- |
| Codex CLI | 在命令行中输入文字，没有传统图形界面 | 熟悉 PowerShell、CMD 或终端的读者 |
| Codex 应用版 | 提供图形化窗口，可视化程度更高 | 第一次接触智能体的读者 |

本文以 **Codex 应用版**为主，CLI 只用于补全本地组件和检查版本。

#### 1.1.1 下载 Codex 应用版

打开 [Codex 中文官网](https://openai.com/zh-Hans-CN/codex/)，下载 Windows 安装包。双击安装文件会出现下图所示的界面，等待安装完成即可。

![Codex 安装界面](智能体部署/codex安装.png)

第一次启动 Codex 时，在登录页面选择 **Sign in another way**，再选择使用 API Key 的方式，将自己在准备阶段创建的新密钥粘贴进去。

![Codex 登录界面](智能体部署/codex首次打开登陆.png)

#### 1.1.2 安装或更新 Codex CLI

打开 CMD 或 PowerShell，先检查 Node.js 和 npm：

```powershell
node --version
npm --version
```

如果两条命令都能显示版本号，继续执行：

```powershell
npm install -g @openai/codex@latest
codex --version
```

`npm install -g` 表示把 Codex CLI 安装到当前电脑，`@latest` 表示安装最新稳定版本。最后一条命令用于检查安装结果，能够看到版本号就说明 CLI 已经可以使用。

如果系统提示找不到 `node` 或 `npm`，请先从 [Node.js 官网](https://nodejs.org/zh-cn)安装带有 **LTS** 标识的版本。下载 Node.js 或安装 Codex 较慢时，可以在遵守所在网络规定的前提下更换更稳定的网络环境。

### 1.2 配置

Codex 的个人设置保存在当前 Windows 用户目录下的 `.codex` 文件夹中 (路径中的 `usersname` 请替换为你的实际用户名)：

```text
C:\Users\usersname\.codex
```

该目录下有两个关键的配置文件：

```text
.codex
├── auth.json      # 保存登录方式和 API Key
└── config.toml    # 保存模型、API 地址和运行设置
```

![.codex 目录下的配置文件](智能体部署/codex配置文件.png)

配置时请注意两者的不同用途：

- config.toml (**必须配置**)：无论你使用应用版还是 CLI，都必须手动修改此文件。这是因为应用版的图形界面没有提供修改 API 地址 (base_url) 和模型名称 (model) 的选项，而我们需要连接西农er's GPT 的第三方接口，因此必须通过该文件指定。

- auth.json (**按需配置**)：

  - 如果你**只使用应用版**，且在 1.1.1 节已通过图形界面登录 (粘贴 API Key)，则**无需**手动配置 auth.json，应用版会自动处理身份验证。
  - 如果你使用 CLI (或希望 CLI 也能正常工作)，则必须手动编辑 auth.json，写入你的 API Key (具体操作见 1.2.1)。

#### 1.2.1 配置 `auth.json`

找到 `auth.json` 并复制到其他位置备份，使用记事本打开 `auth.json`，删除原有内容，再粘贴下面的配置：

```json
{
  "auth_mode": "apikey",
  "OPENAI_API_KEY": "sk-请替换为你自己的API-Key"
}
```

将占位符替换成准备阶段创建的新 API Key，然后保存文件。

这两项配置的含义是：

| 配置项 | 含义 |
| :--- | :--- |
| `auth_mode` | 指定使用 API Key 进行身份验证 |
| `OPENAI_API_KEY` | 保存实际调用平台时使用的密钥 |

> **重要：** `auth.json` 会以明文形式保存 API Key。不要把这个文件上传到 GitHub、网盘公开目录或发送给他人，截图时也必须将密钥完全遮挡。

#### 1.2.2 配置 `config.toml`

找到 `config.toml` 并复制到其他位置备份，使用记事本打开 `config.toml`，删除原有内容，再粘贴：

```toml
model_provider = "OpenAI"
model = "gpt-5.5"
model_reasoning_effort = "xhigh"
network_access = "enabled"
disable_response_storage = true

[model_providers.OpenAI]
name = "OpenAI"
base_url = "https://api.nwafu-ai.cn/v1"
wire_api = "responses"
requires_openai_auth = true
```

主要配置项的作用如下：

| 配置项 | 作用 |
| :--- | :--- |
| `model_provider` | 选择下方名为 `OpenAI` 的服务商配置 |
| `model` | 指定调用的模型，本文示例为 `gpt-5.5` |
| `model_reasoning_effort` | 设置推理强度，`xhigh` 表示较高的推理投入 |
| `network_access` | 在当前环境和授权允许时启用网络访问能力 |
| `disable_response_storage` | 要求兼容接口不要保存响应，但第三方平台实际处理规则仍以其说明为准 |
| `base_url` | 指定西农er's GPT 的 API 地址，Codex 配置中需要包含 `/v1` |
| `wire_api` | 使用 Codex 所需的 Responses API 通信格式 |
| `requires_openai_auth` | 从前面的 `auth.json` 中读取 API Key |

模型名称可能随平台调整。如果 `gpt-5.5` 已不可用，请登录平台查看当前 Codex 分组支持的模型，并将完整模型 ID 替换到 `model = "..."` 中。

配置完成后，彻底退出 Codex 应用并重新打开，使新设置生效。

#### 1.2.3 验证是否部署成功

提前复制一个用于测试的项目文件夹，在 Codex 应用中将它设置为工作目录，然后输入：

```text
请先只读取当前文件夹，告诉我其中有哪些文件。不要执行命令，也不要修改任何内容。
```

如果 Codex 能正确列出测试目录中的文件，并且没有出现身份验证或模型错误，就说明应用、API Key、API 地址和模型已经连接成功。

---

至此，Codex 智能体的本地化部署已全部完成。我们先后完成了应用安装、CLI 环境更新、API Key 写入以及模型接口配置，为后续的使用打下了基础。

在下一期内容中，我们将聚焦 Codex 的基础操作，探讨如何在保障文件安全的前提下，让它高效协助我们处理实际项目。

若在部署过程中遇到任何问题，欢迎通过邮箱 aurorahiker@163.com 与笔者交流。**求助时请务必提供已对敏感信息打码的错误截图，切勿发送 auth.json 或明文 API Key。**

---

**参考资源**

- [Codex 中文官网](https://openai.com/zh-Hans-CN/codex/)
- [Codex CLI 官方仓库](https://github.com/openai/codex)
- [Node.js 官网](https://nodejs.org/zh-cn)
- [西农er's GPT](https://gpt.nwafu-ai.cn/)