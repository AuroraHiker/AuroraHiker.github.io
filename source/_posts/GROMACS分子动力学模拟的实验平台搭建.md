---
title: GROMACS分子动力学模拟的实验平台搭建
tags:
  - 分子动力学模拟
  - 实验平台搭建
series:
  - GROMACS分子动力学模拟
  - 计算机基础与实验平台搭建
categories:
  - GROMACS分子动力学模拟
  - 计算机基础与实验平台搭建
date: 2026-02-24 22:40:18
---


分子动力学模拟是一种基于经典力学的计算机模拟方法，通过求解牛顿运动方程来追踪原子和分子的运动轨迹，从而在原子尺度上研究体系的动态演化过程。它已经成为生物学、化学、材料科学等领域不可或缺的研究工具——从蛋白质折叠、酶催化机制，到药物分子与靶点的相互作用，再到新型材料的设计与优化，分子动力学模拟都能够提供实验难以直接观测的微观细节和动态信息。

在众多分子动力学模拟软件中，GROMACS(GROningen MAchine for Chemical Simulations)因其开源免费、计算效率高、并行扩展性好、以及支持GPU加速等特点，被广泛应用于生物大分子体系的研究。它拥有丰富的力场支持、强大的分析工具以及活跃的社区生态，使得研究者能够快速搭建模拟体系、运行大规模并行计算，并对结果进行深入分析。

然而，要充分发挥GROMACS的性能，搭建一个稳定、高效的实验平台至关重要。这不仅涉及硬件层面的考量——例如CPU核心数、内存大小、GPU型号与数量、存储系统的I/O能力——还包含软件环境的配置：操作系统的选择、编译器的优化、MPI并行库的安装、CUDA及GPU驱动版本匹配、GROMACS本身的编译与调优等。一个精心搭建的平台能够大幅缩短模拟时间，提高资源利用率，并保证计算的准确性和可重复性。

本文旨在为初学者和有一定经验的研究者提供一个系统的GROMACS实验平台搭建指南。我们将从硬件选型建议开始，讲解Ubuntu系统(20.04，包括WSL2 Ubuntu)下GROMACS v2025.4的编译安装、环境变量设置，并补充常见问题的解决方法，这套方法同样可以迁移至Ubuntu 22.04/24.04环境下，GROMACS v2025.4或其他版本的编译安装。希望通过本文，读者能够掌握从零开始构建一套适用于自身研究需求的GROMACS模拟平台的方法，为后续的分子动力学模拟研究打下坚实的基础。

本文参考GROMACS的官方说明文档(版本2026.0)：https://manual.gromacs.org/documentation/current/index.html

---
## 1. 硬件选择

关于计算机硬件的简单介绍，详见这一篇博客：{% post_link 计算机基础：从零认识计算机硬件 %}

**GROMACS硬件配置速查表:**

| 部件 | 最低要求 | 推荐配置(入门) | 推荐配置(进阶) |
|------|----------|------------------|------------------|
| CPU 架构 | x86_64(Intel/AMD) | 同左 | 同左 |
| CPU 核心数 | 4 核 | 8 核 | 16 核或更多 |
| 内存 | 8 GB | 16 GB | 32 GB 或更多 |
| 硬盘 | 20 GB 空闲 | SSD + 100 GB | 大容量 SSD/NVMe |
| GPU(可选) | 无(纯 CPU 跑) | NVIDIA GTX 1050 或更新(需支持 CUDA，计算能力 ≥5.0) | NVIDIA RTX 3060 以上或专业卡 |
| 操作系统 | Linux(推荐)、Windows、macOS | Linux(Ubuntu) | 同左 |

> **对于零基础初学者**：即使只有一台普通台式机或笔记本(没有独立显卡)，也可以顺利安装 GROMACS 并运行小规模的例子。等你熟悉了基本操作，再考虑升级硬件或使用 GPU 加速。

GROMACS 在编译时会自动检测 CPU 支持哪些指令集(如 SSE4.1、AVX2 等)，这些指令集越新，计算越快。你不需要手动设置，但了解自己的 CPU 世代有助于判断性能。

- **2011 年以前的 CPU**：可能只支持 SSE2，可以运行但较慢。  
- **2011–2017 年主流 CPU**：支持 AVX/AVX2，速度不错。  
- **2017 年以后的新 CPU**：支持 AVX-512 或更新的指令集(Intel 第 8 代以后，AMD Zen 4 以后)，速度最快。

希望这些解释能帮你了解自己的硬件基础。接下来我们就可以开始安装软件环境了！

---
## 2. 依赖软件及其安装

### 2.1 系统环境配置

GROMACS 可以在三种主流操作系统上运行，但对初学者最推荐的是 Linux(特别是 Ubuntu)。因为：
- Linux 对科学计算支持最好，安装和配置最直接。
- 绝大多数高性能计算机（包括超算）都运行 Linux。
- 如果你用的是 Windows，也可以通过“适用于 Linux 的 Windows 子系统（WSL）”来安装 Ubuntu，这样既能保留 Windows 习惯，又能享受 Linux 环境。

关于系统环境的选择，在这篇博客中进行了对比和详细介绍：{% post_link 如何搭建适合自己的生信实验平台 %}

### 2.2 编译器配置

GROMACS 的源代码是用 C 和 C++ 编写的，需要编译器将其翻译成机器能执行的指令。  
最低版本要求：**GCC ≥ 11**（必须支持 C++17 标准）。

- 第一步：检查当前 GCC 版本
   
    ```bash
    gcc -v
    g++ -v
    ```
   
    如果输出的版本号高于或等于 11，说明已满足要求，可以跳过安装步骤；如果版本低于 11 或提示“命令未找到”，则需要安装 GCC 11。

- 第二步：安装 GCC 11
   
    ```bash
    # 添加 Ubuntu 工具链 PPA（Personal Package Archive）
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt update

    # 安装 GCC 11 和 G++ 11
    sudo apt install gcc-11 g++-11

    # 安装完成后，可以通过 update-alternatives 将系统默认的 gcc 指向 gcc-11
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 100 \
                            --slave /usr/bin/g++ g++ /usr/bin/g++-11

    # 如果需要同时保留旧版本，可以添加优先级
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 50 \
                            --slave /usr/bin/g++ g++ /usr/bin/g++-9

    ```

    安装完成后，再次检查gcc和g++版本是否符合要求。

### 2.3 CMake配置

CMake 是一个“编译管理工具”。它不会直接编译代码，而是检查你的系统环境、找到各种库的位置，然后生成能让编译器工作的“施工图纸”（Makefile）。
版本要求：**CMake ≥ 3.28**

- 第一步：检查当前 CMake 版本
    ```bash
    cmake --version 
    ```
    若版本 ≥ 3.28，则无需额外操作。若版本过低或未安装，请按以下步骤升级：

- 第二步：使用 Kitware 官方 APT 仓库安装最新 CMake
    ```bash
    # 安装依赖并添加 Kitware 的 GPG 密钥
    sudo apt update
    sudo apt install -y wget gnupg
    wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | sudo tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null

    # 添加 Kitware 仓库（自动适配当前 Ubuntu 版本）
    echo "deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/kitware.list

    # 更新并安装 cmake
    sudo apt update
    sudo apt install cmake

    # 验证安装
    cmake --version
    ```

---
## 3. GROMACS安装

### 3.1 源码包下载

前往 GROMACS 官网下载页面：https://manual.gromacs.org/documentation/current/download.html
或直接使用命令行下载 2025.4 版本（示例版本）：
```bash
# 进入存放软件的目录，例如 ~/software
cd ~/software
wget https://ftp.gromacs.org/gromacs/gromacs-2025.4.tar.gz
```

下载完成后，检查文件是否存在：
```bash
ls -l gromacs-2025.4.tar.gz
```

### 3.2 编译安装
    
依次执行以下命令，并留意输出信息。若出现错误，请参考 3.3 节 debug。
```bash
# 解压源码包
tar xfz gromacs-2025.4.tar.gz
    
# 进入解压后的目录
cd gromacs-2025.4
    
# 创建独立的构建目录（保持源码干净）
mkdir build
cd build
    
# 编译安装，并自动下载FFTW库和回归测试集
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON
    
# 编译（-j4 表示用 4 核并行，可根据 CPU 核心数调整，例如 -j8
make

# 运行测试
make check

sudo make install

# 将 GROMACS 环境配置永久写入 ~/.bashrc，避免每次手动 source
# zsh用户需要将下列命令中的~/.bashrc置换为~/.zshrc
echo 'source /usr/local/gromacs/bin/GMXRC' >> ~/.bashrc
source ~/.bashrc

# 验证安装
gmx --version
```
**编译选项解释：**
- `-DGMX_BUILD_OWN_FFTW=ON` 让 GROMACS 自动下载并编译 FFTW 库，这是官方推荐的简易方式，免去了手动安装 FFTW 的麻烦。
- `-DREGRESSIONTEST_DOWNLOAD=ON` 下载回归测试集，之后可以通过 `make check` 验证编译是否正确。如果不需要测试，可以去掉此选项以节省时间。

### 3.3 常见问题（debug）

**问题1：cmake .. 时报错提示编译器版本过低或找不到编译器**

- 如果运行`cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON`时报错提示编译器版本过低或找不到编译器，优先检查gcc & g++的版本是否大于11，CMake版本是否大于3.28，并升级不符合要求的软件。
- 升级完成后，直接运行`cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON`会产生bug，需要清理缓存
    ```bash
    # 回到build的上级目录，即gromacs-2025.4
    cd ../

    # 删除旧的 build 文件夹并创建新的（如果里面有重要文件，请备份）
    rm -rf build
    mkdir build
    cd build
    ```

**问题2：安装完成后输入 gmx 提示命令未找到**

- 检查是否正确执行了 `source /usr/local/gromacs/bin/GMXRC`。可以运行 `echo $GMXRC` 查看是否已设置。若未设置，重新执行 source 命令，或检查是否已将 source 命令加入 `~/.bashrc` 并重新加载。

---

至此，你已经成功搭建了一个基础的 GROMACS 模拟平台！从硬件认知到软件安装，每一步都经过了实践检验。接下来，你可以：

- 学习 GROMACS 的输入文件准备（拓扑、坐标、参数）。
- 尝试运行一个完整的模拟例子（如官方教程中的 Lysozyme）。
- 根据实际需求，进一步配置 MPI 并行或 GPU 加速。

如果在使用过程中遇到问题，欢迎向我的邮箱aurorahiker@163.com发送留言交流。祝你在分子动力学模拟的世界里探索愉快！