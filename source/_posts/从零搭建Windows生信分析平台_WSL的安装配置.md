---
title: 从零搭建Windows生信分析平台——WSL的安装配置
tags:
  - 计算机基础
  - 实验平台搭建
series: 计算机基础与实验平台构建
categories:
  - 计算机基础与实验平台搭建
date: 2026-04-02 17:40:19
---

欢迎来到 AuroraHiker 的技术博客！本系列博客(计算机基础与实验平台构建)此前已详细讲解 Windows 系统的安装，本文面向零基础学习者，基于 Windows 环境，详细介绍通过 WSL2 搭建 Ubuntu 子系统的完整流程，为后续运行 wget、awk、grep 等 Linux 系统自带工具以及 Bedtools、LightDock 等生信软件奠定基础。本文涵盖 WSL2 安装、Ubuntu 系统配置、图形界面配置、Zsh 终端优化及 Anaconda 环境部署等内容，供大家参考。欢迎各位读者结合实际使用情况提供补充。

---

## 1. WSL2 ubuntu安装和迁移

**注意：**
- WSL2 Ubuntu 子系统默认安装在 C 盘，**安装时无法更改路径。**
- **磁盘空间要求：**
  - 仅安装基础 Ubuntu 系统约需 5~10 GB。
  - 若后续安装生信软件(如 Conda、LightDock 等)并存储数据，建议 C 盘剩余空间 ≥ 30 GB。
  - **当 C 盘剩余空间低于 20 GB 时，强烈建议将子系统迁移到其他非系统盘**，否则可能导致系统运行缓慢、更新失败甚至无法启动。
- **建议安装或迁移至固态硬盘(SSD)：** SSD 具有更快的读写速度，能显著提升 WSL 子系统的启动速度、软件安装效率及 I/O 密集型操作(如编译、数据处理)的运行性能。
- **反复装卸的风险：**
  - 多次卸载并重新安装 Ubuntu 子系统可能产生残留配置文件或注册表项，导致新安装的系统出现异常或安装失败。
  - 卸载时若误删除 WSL 相关的系统文件，可能影响其他 WSL 发行版的正常运行，甚至导致 Windows 系统文件损坏。
- **迁移的另一用途：拷贝系统到新电脑。** 通过 `wsl --export` 导出 tar 文件，将该文件复制到新电脑后，使用 `wsl --import` 导入，即可在新电脑上获得完全一致的 Ubuntu 环境(包括已安装的软件、配置等)，无需重新配置。非常适合在多台设备间同步生信分析环境。

### 1.1 WSL2 ubuntu安装

- **系统版本要求**
本安装教程仅适用于 Windows 10 版本 2004 及更高版本(含 Windows 11)。若您使用的是旧版 Windows 系统，或需要了解本文未提及的安装细节，请参考 WSL 官方教程：https://learn.microsoft.com/zh-cn/windows/wsl/install

- **启用“适用于 Linux 的 Windows 子系统”：**
以管理员身份运行 Windows PowerShell，执行以下命令：
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

- **安装 WSL 并升级至 WSL2：**
```powershell
wsl --install
wsl --update
```

- **下载 Ubuntu 系统：**
**在 Microsoft Store 中搜索“Ubuntu”，选择所需的系统版本(如 Ubuntu 22.04)并下载安装。**
![在 Microsoft Store 中搜索 Ubuntu](./从零搭建Windows生信分析平台/WSLubuntu_install.png)

### 1.2 Ubuntu子系统的迁移

- 查看wsl状态
```powershell
wsl -l -v
```

- 关掉正在运行的wsl
```powershell
wsl --shutdown
```

- 导出需要迁移的Ubuntu子系统
```powershell
wsl --export Ubuntu(版本号) D:\wsl_backup\Ubuntu.tar # 该文件也可作为备份
```

- 确认导出.tar文件后，注销原来的版本
```powershell
wsl --unregister Ubuntu(版本号)
# 查看是否注销
wsl -l -v
```

- 在目标盘符(如D盘)创建新目录并导入
```powershell
wsl --import Ubuntu-20.04(此处严格按照wsl -l -v的输出填写) D:\WSL\Ubuntu D:\wsl_backup\Ubuntu.tar --version 2
```

- 设置默认登录用户(可选，否则默认以 root 登录)
```powershell
ubuntu config --default-user 你的用户名
```

- 验证迁移是否成功
```powershell
wsl --list --verbose
wsl
```

迁移完成后，原 C 盘中的子系统文件已被清理，新系统位于 D 盘(或其他分区)。后续所有操作均在新位置进行，无需重新配置环境。

---

## 2. Ubuntu基础配置

安装完成后，启动 Ubuntu，按提示设置用户名和密码。
![设置用户名和密码](./从零搭建Windows生信分析平台/ubuntu_username.png)

### 2.1 更换国内镜像源并更新系统

- **备份原始配置文件**
```bash
cp /etc/apt/sources.list /etc/apt/sources_bk.list
```

- **编辑源文件：**
使用 vim 打开配置文件：
```bash
vim /etc/apt/sources.list
```
打开后，按 `ggdG`(依次按 g、g、d、G 键)可快速删除全部内容，然后按 `i` 进入插入模式，将以下清华镜像源内容粘贴进去(以 Ubuntu 22.04 为例)：
```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

粘贴完成后，按 `Esc` 键退出插入模式，输入 `:wq` 保存并退出 vim。

>提示：若编辑过程中误操作想放弃保存，可按 `Esc` 后输入 `:q!` 强制退出。

### 2.2 更新并安装基础软件

```bash
sudo apt update
sudo apt upgrade
sudo apt install aptitude #智能安装、升级、降级或卸载依赖的软件包
sudo aptitude install aria2 #多线程下载软件
```

---

## 3. 图形界面的安装和配置
> **说明：** 笔者个人感觉 Ubuntu 图形界面运行较慢且体验一般，不如在 Ubuntu 中安装 Jupyter Notebook 进行文件操作更方便。本部分仅供有图形界面需求的读者参考。

### 3.1 安装图形界面

```bash
sudo aptitude install xorg
sudo aptitude install xfce4
```

### 3.2 安装并配置xrdp

Xrdp 通过远程桌面的方式来访问另外一台主机

- **安装**
```bash
sudo aptitude install xrdp
```

- **设置使用3390端口**
```
sudo sed -i 's/port=3389/port=3390/g' /etc/xrdp/xrdp.ini
```

- **向xsession中写入xfce4-session**
```
sudo echo xfce4-session >~/.xsession
```

- **重启xrdp服务**
```
sudo service xrdp restart
```

### 3.3 Windows 端远程桌面连接

在 Windows 中打开“远程桌面连接”，输入以下信息：
- 计算机：localhost:3390
- 用户名：Ubuntu 用户名

![远程桌面连接配置](./从零搭建Windows生信分析平台/远程桌面连接配置.png)

**注意：** 若连接失败，可尝试在 Ubuntu 中重启 xrdp 服务后重试。

连接成功后显示 Ubuntu 桌面环境：
![远程桌面连接成功](./从零搭建Windows生信分析平台/远程桌面.png)

---

## 4. Zsh 与 Oh My Zsh 的安装与配置

**关于 Zsh 的说明：**
Zsh(Z shell)是一种功能更强大的 Shell，相较于系统默认的 Bash，它提供了更丰富的特性：智能命令补全、语法高亮、主题自定义、强大的插件机制等。结合 Oh My Zsh 框架，可以极大提升终端使用效率。
**若您不希望配置 Zsh**，也可以直接使用默认的 Bash 完成后续 Conda 等软件的安装与配置，跳过本章节即可。使用 Bash 时，配置文件为 `~/.bashrc`，后续操作中的 `~/.zshrc` 请相应替换为 `~/.bashrc`。

### 4.1 ZSH安装

```bash
sudo aptitude install zsh
zsh  # 运行一次以完成初始化配置
```

**将zsh设为默认终端：**

```bash
sudo chsh -s /bin/zsh
```
执行后需重新登录或重启终端使设置生效。

### 4.2 Oh My Zsh 安装

Oh My Zsh 是一个开源的 Zsh 配置管理框架，能大幅简化插件与主题的管理。本文使用清华镜像源进行下载安装。
> 注意：执行 git clone 时，若终端出现类似 `remote: Waiting in queue... (Position: 881)` 的提示，说明镜像服务器正在排队处理请求，请耐心等待。

```zsh
git clone https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
cd ohmyzsh/tools
REMOTE=https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git sh install.sh
```

安装完成后，Oh My Zsh 的配置文件位于 `~/.zshrc`，插件目录为 `~/.oh-my-zsh/custom/plugins/` 。

### 4.3 配置常用插件

#### 4.3.1 autojump(目录快速跳转)

autojump 可以根据历史访问记录快速跳转到指定目录。使用国内镜像源下载安装：

```zsh
git clone https://gitee.com/holyvan/autojump.git 
cd autojump
./install.py # 或 python3 install.py
```

>**注意：** `./install.py`脚本依赖 Python，建议在安装 Conda 后再执行此步骤。但 Zsh 需在 Conda 安装前设为默认 Shell，请根据实际情况安排顺序。

安装成功后，脚本会输出类似以下提示，请将中间三行代码添加到 `~/.zshrc` 中(路径以实际输出为准，通常位于 $HOME/.autojump 目录下)：

```zsh
Please manually add the following line(s) to ~/.zshrc:

    [[ -s /data4/jxs/.autojump/etc/profile.d/autojump.sh ]] && source /data4/jxs/.autojump/etc/profile.d/autojump.sh

    autoload -U compinit && compinit -u

Please restart terminal(s) before running autojump.

```

![autojump的输出](./从零搭建Windows生信分析平台/autojump输出.png)

根据提示把中间的三行代码加到`.zshrc`文件中，
添加方法：使用 `vim ~/.zshrc` 打开配置文件，将上述三行内容粘贴到文件末尾，保存并退出(按 `Esc`，输入 `:wq`)。


#### 4.3.2 自动补全与语法高亮插件

**功能与用法**

- **zsh-syntax-highlighting(命令语法高亮)：** 为输入的命令提供实时颜色高亮。合法命令显示为绿色，非法命令显示为红色，存在的路径显示为下划线。这可以帮助您在按回车前快速发现拼写或路径错误。

- **zsh-autosuggestions(命令自动补全建议)：** 根据历史命令记录，在输入时自动显示灰色提示，建议可能想要输入的完整命令。按 →(右箭头)键即可一键补全，按 End 键仅补全光标之后的部分，继续正常输入则忽略建议。

同时启用这两个插件后，能获得实时语法校验与智能补全提示，大幅提升终端输入效率。

**插件下载：**

```zsh
# 自动补全插件
git clone https://gitee.com/hailin_cool/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# 语法高亮插件
git clone https://gitee.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
>**说明：** `${ZSH_CUSTOM:-~/.oh-my-zsh/custom}` 表示优先使用环境变量 `ZSH_CUSTOM` 定义的目录，若未定义则使用默认的 `~/.oh-my-zsh/custom`。

**检查插件是否下载到正确位置**
执行以下命令查看插件目录内容：

```zsh
ls -l ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/
```

若插件未出现在该目录中，可手动将其移动至正确位置。例如，若 zsh-syntax-highlighting 下载到了当前目录，可执行：

```zsh
mv ./zsh-syntax-highlighting ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

若不确定插件下载到了哪里，可使用 `find` 命令搜索：

```zsh
find ~ -type d -name "zsh-syntax-highlighting" 2>/dev/null
```

**启用插件**
编辑 `~/.zshrc` 文件，找到 `plugins=(...)` 行，修改为：

```zsh
plugins=(git zsh-autosuggestions zsh-syntax-highlighting autojump)
```

保存后执行以下命令使配置生效：

```zsh
source ~/.zshrc
```

---

## 5. conda的安装与配置
### 5.1 conda简介

conda 是一个开源的 Python 和 R 语言发行版，专注于数据科学、机器学习和科学计算。它内置了包管理器，可以方便地创建隔离的虚拟环境、安装和管理软件包，尤其适合生信分析中不同工具对 Python 版本的依赖管理。**但笔者使用 Conda 管理 R 环境的体验不佳**，不建议用 Conda 安装或管理 R 及相关包。推荐使用 Rstudio 进行 R 环境管理，更稳定、兼容性更好。

**Miniconda 与 Anaconda 的区别**

- Anaconda：预装了超过 1500 个常用的数据科学包(如 NumPy、SciPy、Pandas 等)，安装包较大(约 3~5 GB)，适合不想手动安装基础包的用户。
- Miniconda：仅包含 conda 和 Python，安装包很小(约 50~80 MB)，用户需要自行安装所需包。它更轻量、灵活，适合磁盘空间有限或希望按需安装的用户。

本教程以 Anaconda 为例进行安装，若您偏好 Miniconda，可前往 Miniconda 官网下载对应版本，后续使用方式与 Anaconda 基本一致。

### 5.2 下载并安装 Anaconda

从 Anaconda 官网获取最新 Linux 版本的下载链接(建议使用稳定版)。本文以 `Anaconda3-2023.09-0-Linux-x86_64.sh` 为例：

```zsh
wget https://repo.anaconda.com/archive/Anaconda3-2023.09-0-Linux-x86_64.sh
bash Anaconda3-2023.09-0-Linux-x86_64.sh 
```

安装过程中，请仔细阅读许可协议，按提示确认安装位置,默认为 `~/anaconda3`。在最后选择是否初始化 `conda`，建议选择`yes`，这样 `conda` 会自动将自身添加到 Shell 配置文件中(如 `~/.zshrc` 或 `~/.bashrc`)。

### 5.2 配置环境变量

若安装时未自动添加，或需要手动配置，请将以下内容添加到对应的配置文件中：

- 使用 Zsh 的用户：编辑 `~/.zshrc`
- 使用 Bash 的用户：编辑 `~/.bashrc`

在文件末尾添加，请将 `/path/to/anaconda3` 替换为实际安装路径：

```zsh
export PATH="/path/to/anaconda3/bin:$PATH"
```

保存后执行以下命令使配置生效：

``` bash
source ~/.zshrc   # 若使用 Zsh
# 或
source ~/.bashrc  # 若使用 Bash
```

**验证安装**

重新打开终端，或执行 source 后，运行：

```bash
conda --version
```
若输出版本号(如 conda 23.10.0)，则表示安装成功。
>**提示：** 如果您已按照第 4 章配置了语法高亮插件，在终端输入 conda 后，命令会显示为绿色(表示合法命令)；若未配置高亮插件，直接运行上述命令即可。

如果在后续使用中需要创建虚拟环境，可使用：

```bash
conda create -n 环境名称 python=3.8
conda activate 环境名称
```

如在使用过程中遇到问题，欢迎向我的邮箱aurorahiker@163.com留言！下篇见。