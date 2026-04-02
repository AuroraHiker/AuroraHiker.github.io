---
title: LightDock分子对接的实验平台搭建
tags:
  - 分子对接
  - 实验平台搭建
  - LightDock
series:
  - 分子对接
  - 计算机基础与实验平台搭建
categories:
  - 分子对接
author: AuroraHiker
date: 2026-04-02 09:44:01
---
![](./LightDock_install/lightdock_logo.png)

本系列博客面向零基础学习者，基于 LightDock 官方文档 (https://lightdock.org/) 与 GitHub 仓库 (https://github.com/lightdock/lightdock) ，系统介绍最新版本 v0.9.4 分子对接实验平台的搭建与使用方法。关于旧版本的安装与使用细节，建议参考官方网站。

LightDock 基于萤火虫算法，支持已知或未知相互作用位点的蛋白-蛋白、膜受体-可溶性蛋白复合物的预测，以及蛋白质-DNA/RNA 的刚性或柔性对接，并可实现多个潜在互作结构的对接分析。与其他分子对接工具相比，LightDock 的优势在于开源、能够探索多个结合模式、支持柔性对接、并行效率高、支持多种打分模式；但其计算成本较高，参数敏感且缺乏自动调参机制，不适用于小分子-蛋白对接。笔者后续将在博客中发布详细的分子对接工具对比，帮助大家选择合适的工具。

本教程基于 LightDock 官方基础知识文档 (https://lightdock.org/tutorials/0.9.3/basics) ，详解 LightDock v0.9.4 所依赖的硬件与系统环境配置，供大家参考。欢迎各位读者结合实际使用情况提供补充。

---

## 1. 硬件选择

关于计算机硬件的简单介绍，可参考笔者的另一篇博客：{% post_link 计算机基础：从零认识计算机硬件 %}
LightDock官方未给出具体的硬件建议，以下为笔者的硬件配置与使用体验，供大家参考。是否启用柔性对接、萤火虫群数量、步数 (step) 以及 CPU 线程数等设置均会影响资源消耗，建议在运行过程中监控资源使用情况，并据此合理调整参数。

推荐使用 WSL Ubuntu 环境运行，该环境下系统会自动终止过载线程，避免死机。欢迎其他系统环境的读者补充不同系统下的运行表现。

笔者的硬件配置如下：
- CPU：AMD Ryzen 7 9700X
- 内存：64 GB RAM

基于该配置，对于分子量较小蛋白-蛋白对接，能够覆盖较大的构象空间，在初始种群数量较多、柔性处理充分、100 步搜索较彻底的条件下，可通过八核并行处理实现较快的运算。若其中一个蛋白分子量较大，则只能采用双核、刚性对接、较小种群数量与 50 步的配置。

---

## 2. 系统选择

LightDock v0.9.4 支持 macOS 与 GNU/Linux 系统，暂不支持 Windows 系统。开发者表示其主要功能在 Windows 上也可运行，有兴趣的用户可联系开发团队 (邮箱：lightdocking@gmail.com) 参与 Windows 版的测试与开发。

笔者使用的是 Windows 10 环境下的 WSL2 Ubuntu 20.04。建议使用 Windows 系统的读者部署 WSL Ubuntu 22.04 或更高版本，以便运行支持 Linux 系统的软件。后续笔者将在博客中发布详细的部署教程。

支持的具体系统版本如下：

| 系统 | 版本 |
|-------|------------------------|
| macOS | El Capitan, Sierra, High Sierra, Mojave, Catalina. |
| GNU/Linux | Ubuntu 16+, Debian Stretch+, Scientific Linux 6+, CentOS 6+. |

---

## 3. 环境依赖

LightDock v0.9.4 依赖以下软件，安装过程中通常会自动下载：

| 软件 | 官网 |
|-------|------------------------|
| NumPy  | http://www.numpy.org/ |
| Scipy | http://www.scipy.org/ |
| Cython | http://cython.org/ |
| ProDy | http://prody.csb.pitt.edu/ |
| Freesasa | http://freesasa.github.io/ |

---

## 4. 安装

LightDock 对依赖软件的版本无特殊要求，但建议在安装时为其单独创建虚拟环境，以便于管理和调试。编译安装的详细步骤可参考 LightDock 官网。

配置虚拟环境需要 Conda，后续笔者将在博客中发布详细的 Conda 安装与部署教程。

```{bash}
# 创建并激活虚拟环境
conda create -n lightdock python=3.8
conda activate lightdock
```

若不需配置虚拟环境，可直接运行以下命令安装最新版 LightDock：

```{bash}
pip install lightdock
```

---

欢迎各位读者结合实际使用情况提供补充，也欢迎通过邮件与笔者交流使用过程中遇到问题：aurorahiker@163.com。

若在学术研究或论文中使用了 LightDock 软件，请按照官方要求引用相关文献。具体的引用信息可查阅 LightDock 官方引用建议 (https://lightdock.org/reference/) 或 GitHub 仓库中的说明。