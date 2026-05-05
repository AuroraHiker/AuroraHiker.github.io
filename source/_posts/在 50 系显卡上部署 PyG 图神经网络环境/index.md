---
title: 在 50 系显卡上部署 PyG 图神经网络环境
tags:
  - 图神经网络
  - 实验平台搭建
series:
  - 图神经网络
  - 计算机基础与实验平台搭建
categories:
  - 图神经网络
author: AuroraHiker
date: 2026-05-05 15:21:29
---

最近在使用 RTX 5060 Ti 训练图神经网络模型 (GNN) 时，发现在新硬件上配置环境比预想中要多花一些功夫。这篇博客记录了从零开始搭建 PyTorch Geometric (PyG) 环境的完整过程，适用于 WSL2或原生 Ubuntu 20.04 及以上版本的系统，可作为一份新手友好的 checklist。跟随这篇博客完成配置，你将得到：

- 一个在 50 系显卡 (以 RTX 5060 Ti 为代表) 上稳定运行 PyG 的 conda 环境
- 编译安装 torch-sparse 与 torch-scatter 的实操方法，以及遇到依赖缺失时的处理思路
- 一套完整、可复用的图神经网络环境配置方案，日后换机或重建环境可直接照做

---

## 1. 环境配置方案总览

这套方案基于 WSL2 (Ubuntu 20.04)，原生系统同样适用。

| 组件 | 版本/说明 |
|------|----------|
| GPU | NVIDIA GeForce RTX 5060 Ti |
| 驱动版本 | 581.57 |
| 环境管理器 | Conda 24.11.0 |
| Python | 3.10.16 |
| pip | 26.0.1 |
| CUDA Toolkit | 12.8.1 |
| PyTorch | 2.12.0.dev20260407+cu128 (Nightly) |
| torchvision | 0.27.0.dev20260407+cu128 |
| torchaudio | 2.11.0.dev20260407+cu128 |
| torch-scatter | 2.1.2 (手动编译) |
| torch-sparse | 0.6.18 (手动编译) |
| PyTorch Geometric | 2.8.0.dev20260504 (pyg-nightly) |
| triton | 3.7.0+git9c288bc5 |
| numpy | 2.2.6 |
| scipy | 1.15.3 |
| networkx | 3.4.2 |
| tqdm | 4.67.3 |
| jupyterlab | 4.5.7 |

> **为什么一定要用 Nightly 版 PyTorch？**  
>
> 在软件开发中，很多大型项目 (比如 PyTorch) 都会设置一套持续集成系统。每当有开发者向主分支提交代码后，这套系统就会在每天夜间自动拉取最新代码，编译、打包、跑测试，并生成可供下载的安装包，被称为 **Nightly Build (每夜构建版)**，简称 Nightly 版本，通俗而言就是抢先体验版。也正因包含了最新代码，它才能率先支持新硬件。
>
> RTX 5060 Ti 是 Blackwell 架构，计算能力 `sm_120`。截至本文写作时，稳定版 PyTorch 的预编译包并不支持 `sm_120`，无法识别新卡。要让 GPU 正常工作，必须从 Nightly 源安装并配合 CUDA 12.8 (截至本文写作时的推荐组合)。

---

## 2. 环境准备

先建一个新环境，把编译工具和常用库装好。

```bash
# 创建并激活环境
conda create -n GNN python=3.10 -y
conda activate GNN

# 常用科学计算与可视化
pip install pandas numpy scipy matplotlib seaborn tqdm jupyterlab

# 编译依赖 (后续编译 torch-sparse 等需要)
pip install ninja cmake
sudo apt install -y build-essential cmake ninja-build
sudo apt install -y cuda-toolkit-12-8
```

---

## 3. 安装 PyTorch Nightly

这一步最关键，必须指定 CUDA 12.8 的 Nightly 源。

```bash
pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128
```

装完马上验证，确保新 GPU 被正确支持：

```bash
# 检查编译架构列表
python -c "import torch; print(torch.cuda.get_arch_list());"

# [预期输出] 必须包含 sm_120。这是 Blackwell 架构 (如 RTX 5060 Ti) 的关键标识
# ['sm_50', 'sm_60', 'sm_70', 'sm_75', 'sm_80', 'sm_86', 'sm_90', 'sm_100', 'sm_120']
```

```bash
# 检查版本和 CUDA 可用性
python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available())"

# [预期输出] 版本号随 Nightly 更新，但 CUDA 须为 12.8，is_available 须为 True
# 2.12.0.dev20260407+cu128 12.8 True
```

如果 `get_arch_list()` 没有 `sm_120`，说明 Nightly 源不对或安装失败，此时应检查pip命令中的--index-url是否正确，或确认网络能正常访问PyTorch Nightly源后重试安装。

---

## 4. 手动编译 torch-sparse 与 torch-scatter

PyG 在稠密图计算时的性能高度依赖这两个底层库。因为我们的 PyTorch 版本特别新，截至本文写作时预编译的 whl 不兼容，自己编译相对稳妥。

### 4.1 获取源码

从 GitHub 下载 `pytorch_sparse` 和 `pytorch_scatter` 的 master 分支源码。推荐两种方式：

- **直接下载压缩包**(适合国内网络不稳定时使用)
- **git 克隆**：`git clone https://github.com/rusty1s/pytorch_sparse.git`(同理克隆 scatter 仓库)

> 国内访问 GitHub 偶尔不稳定。我在之前的博客中曾推荐过一款免费加速器 ({% post_link Windows效率神器推荐-Wox-Markdown编辑器与GitHub加速器配置指南 %})，但这款加速器无法加速 WSL 内部的网络访问。  
>
> 因此，如果你使用 WSL，建议在 Windows 下用浏览器或下载工具提前下载好项目的 zip 压缩包，下载后放置在任意目录，然后通过 WSL 访问 `/mnt/c/` 路径进行操作。

假设已将压缩包下载至 Windows 的 `Downloads` 文件夹：

```bash
cd /mnt/c/Users/你的用户名/Downloads
unzip pytorch_sparse-master.zip
unzip pytorch_scatter-master.zip
```

### 4.2 编译 torch-sparse

```bash
cd pytorch_sparse-master
export TORCH_CUDA_ARCH_LIST="12.0"
export MAX_JOBS=12
python setup.py clean --all
python setup.py install --verbose
```

编译过程中可能会遇到一个常见的错误信息：

```text
fatal error: parallel_hashmap/phmap.h: No such file or directory
```

**原因**：`torch-sparse` 依赖的第三方头文件库 `parallel-hashmap` 虽然文件夹存在，但里面缺少实际的 `.h` 文件，属于源码打包时的小疏忽。

**解决方法**：手动补全这 6 个头文件。由于 WSL 内直接 `wget` 也可能因 `raw.githubusercontent.com` 网络不稳定而失败，这里提供两种互补方案，请先执行共同步骤，再任选其一获取文件。

#### 共同步骤：创建目标文件夹
```bash
cd pytorch_sparse-master/third_party/parallel-hashmap
mkdir -p parallel_hashmap
cd parallel_hashmap
```

#### 方法一：WSL 内直接 wget (网络通畅时推荐)
```bash
wget https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap.h
wget https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_base.h
wget https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_bits.h
wget https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_config.h
wget https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_fwd_decl.h
wget https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_utils.h
```

#### 方法二：Windows 下浏览器手动下载 (备选，稳定可靠)
1. 在 Windows 浏览器中分别打开以下链接，每个页面打开后按 `Ctrl+S` 保存为对应的 `.h` 文件 (确保文件名和数量完全一致)：
  ```text
  https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap.h
  https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_base.h
  https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_bits.h
  https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_config.h
  https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_fwd_decl.h
  https://raw.githubusercontent.com/greg7mdp/parallel-hashmap/master/parallel_hashmap/phmap_utils.h
  ```

2. 将下载好的 6 个 `.h` 文件集中放到 Windows 的 `Downloads` 文件夹 (或其他方便的位置)。

3. 回到 WSL 终端，应该仍停留在 `parallel_hashmap` 目录 (请确保你已在该目录下，可输入`pwd`命令确认)，执行拷贝命令：
   ```bash
   cp /mnt/c/Users/你的Windows用户名/Downloads/phmap*.h .
   ```

#### 最后一步：重新编译
无论用哪种方法获得头文件，完成后都返回源码根目录并重新编译：
```bash
cd ../../..
python setup.py clean --all
python setup.py install --verbose
```

补全头文件后编译即可顺利通过，`torch-sparse` 安装完成。

### 4.3 编译 torch-scatter

同样的方式，进入 `pytorch_scatter-master` 目录执行编译：

```bash
cd ../pytorch_scatter-master
export TORCH_CUDA_ARCH_LIST="12.0"
export MAX_JOBS=12
python setup.py clean --all
python setup.py install --verbose
```

`torch-scatter` 一般不会缺文件，正常等待结束即可。

---

## 5. 安装 PyG 主库

依赖就绪后，安装 PyTorch Geometric 本体，也推荐 Nightly 版以保持兼容：

```bash
pip install pyg-nightly
```

不必再单独安装 `torch-cluster`、`torch-spline-conv` 等旧式 whl，它们可能与 Nightly PyTorch 产生依赖冲突。

---

## 6. 最终验证

依次执行以下检查，确保每个组件都正常工作。

### 6.1 GPU 硬件信息
```bash
nvidia-smi -q | grep -E "Product Name|CUDA Version|Driver Version"

# [预期输出] Product Name 应为你的显卡版本 (笔者的是RTX 5060 Ti)，驱动版本 ≥ 535，CUDA Version 需 ≥ 12.8
#     Driver Version                      : 581.57
#     CUDA Version                        : 13.0
#     Product Name                        : NVIDIA GeForce RTX 5060 Ti
```

### 6.2 PyTorch 支持的架构列表
```bash
python -c "import torch; print(torch.cuda.get_arch_list())"

# [预期输出] 必须包含 `sm_120`，这是 Blackwell 架构 RTX 5060 Ti 的关键标识
# ['sm_50', 'sm_60', 'sm_70', 'sm_75', 'sm_80', 'sm_86', 'sm_90', 'sm_100', 'sm_120']
```

### 6.3 PyTorch 基本状态
```bash
python -c "import torch; print(f'PyTorch {torch.__version__}'); print(f'CUDA {torch.version.cuda}'); print(f'Available: {torch.cuda.is_available()}')"

# [预期输出] 版本号随 Nightly 更新，但编译 CUDA 版本须为 12.8，且 is_available 为 True，否则无法为 RTX 5060 Ti 生成正确内核
# PyTorch 2.12.0.dev20260407+cu128
# CUDA 12.8
# Available: True
```

### 6.4 GNN 所需库状态
```bash
python -c "import torch_sparse, torch_scatter, torch_geometric; print('torch-sparse:', torch_sparse.__version__); print('torch-scatter:', torch_scatter.__version__); print('PyG:', torch_geometric.__version__)"

# [预期输出]
# torch-sparse: 0.6.18
# torch-scatter: 2.1.2
# PyG: 2.8.0.dev20260504
```

> ✅ 如果这四条命令都输出正常，你的 RTX 5060 Ti 就已完整支持图神经网络计算了！

---

## 7. 回顾：问题与处理

| 遇到的状况 | 原因分析 | 解决方法 |
|-----------|---------|----------|
| GPU 不工作，PyTorch 只调用 CPU | 稳定版 PyTorch 缺少 `sm_120` 目标架构 | 安装 PyTorch Nightly (CUDA 12.8) |
| `torch-sparse` 编译报错 `phmap.h: No such file or directory` | `parallel-hashmap` 头文件缺失 | 手动获取 6 个核心头文件后重新编译 |
| 旧式 whl 包与 Nightly PyTorch 冲突 | ABI 不兼容 | 统一使用 Nightly 源安装 PyG，不混用旧版子包 |

---

## 8. 写在最后

环境配置的本质，不过是一次常规磨合。遇到报错，就耐心地顺藤摸瓜定位原因、查阅官方文档、补全缺失的部分，这套思路适用于任何开发环境的装配。

希望这篇记录不仅能帮你节省搜索和尝试的时间，也能让你在跟练过程中慢慢建立起解决环境配置问题的 debug 思维。如果配置时有什么新发现，欢迎通过邮箱 aurorahiker@163.com 留言交流。说不定你的经验，正好能帮到下一个正在给 50 系显卡配环境的人。

---

**参考资料**
- [PyTorch 官方安装页](https://pytorch.org/get-started/locally/)
- [PyTorch Geometric 文档](https://pytorch-geometric.readthedocs.io)
- [NVIDIA CUDA 计算能力查询](https://developer.nvidia.com/cuda-gpus)
