---
title: 从零开始学 Python 绘图：用鸢尾花数据集掌握柱状图美化全流程
tags:
  - Python 入门
  - 数据可视化
  - Matplotlib
  - Seaborn
series:
  - Python 入门
categories:
  - Python 入门
author: AuroraHiker
---

很多人第一次接触 Python 绘图时，最容易卡住的地方并不是“怎么画一张图”，而是画出来之后总觉得不够像论文里的图：字体不统一、边框太重、图例位置尴尬、几个子图挤在一起、保存出来又不够清晰。

这篇博客就用经典的 Iris (鸢尾花) 数据集，带大家从最基础的一张柱状图开始，逐步调整成更适合论文或汇报使用的组合柱状图。最终效果是：**按鸢尾花品种分面，每个品种单独占一行，x 轴展示花萼长度、花萼宽度、花瓣长度、花瓣宽度四个统计项目，柱高表示对应项目的均值**。

本文不追求一次写出“最强绘图代码”，而是按新手真正学习绘图时的顺序来走：先认识全局样式，再画单个品种，再调整颜色和图例，最后把所有品种组合到同一张图中。

**建议先完成 {% post_link Python-入门：零基础编程速通指南 %} 和 {% post_link Pandas-简明教程 %} 教程的学习后，再进行这方面的学习。**

---

# 0. 准备工作：安装包与加载数据

本文需要用到 4 个常见包：

```shell
pip install pandas matplotlib seaborn scikit-learn
```

| 包名 | 作用 |
|------|------|
| `pandas` | 整理表格数据、计算均值 |
| `matplotlib` | 负责真正绘图 |
| `seaborn` | 提供更好看的配色方案 |
| `scikit-learn` | 提供内置的 Iris 鸢尾花数据集 |

先把数据加载进来，并整理成适合画图的长表格式。

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from matplotlib.patches import Patch
from sklearn.datasets import load_iris
from pathlib import Path

# Path 用于建立图片导出目录，避免手动拼接路径字符串
output_dir = Path("./") # './'表示当前工作目录

# 加载 Iris 数据集
iris = load_iris(as_frame=True)
df = iris.frame.copy()
print(df.head())
```

此时，表示鸢尾花品种的是target列，以数值标签 (0、1、2) 标识，需转换为对应的鸢尾花品种名称 (setosa、versicolor、virginica) 

```python
# 转换后删除原始数值列，仅保留品种名称列以便后续分析
species_map = dict(enumerate(iris.target_names))  # {0: 'setosa', 1: 'versicolor', 2: 'virginica'}
df["species"] = df["target"].map(species_map)     # 根据映射替换数值
df = df.drop(columns="target")                    # 移除已无用的数值标签列

print(df.head())
```

此时的 `df` 大致长这样：

| sepal length (cm) | sepal width (cm) | petal length (cm) | petal width (cm) | species |
|------|------|------|------|------|
| 5.1 | 3.5 | 1.4 | 0.2 | setosa |
| 4.9 | 3.0 | 1.4 | 0.2 | setosa |
| 4.7 | 3.2 | 1.3 | 0.2 | setosa |

为了后续绘图方便，我们把 4 个数值列整理成一列。这个操作叫做“宽表转长表”：原来每个特征各占一列，现在改成“特征名称一列、数值一列”。

```python
feature_order = [
    "Sepal length",
    "Sepal width",
    "Petal length",
    "Petal width"
]

feature_name_map = {
    "sepal length (cm)": "Sepal length",
    "sepal width (cm)": "Sepal width",
    "petal length (cm)": "Petal length",
    "petal width (cm)": "Petal width"
}

long_df = df.melt(
    id_vars="species",
    value_vars=list(feature_name_map.keys()),
    var_name="feature",
    value_name="value"
)

long_df["feature"] = long_df["feature"].map(feature_name_map)

summary_df = (
    long_df
    .groupby(["species", "feature"], as_index=False)
    .agg(mean_value=("value", "mean"))
)

print(summary_df.head())
```

`summary_df` 就是本文真正用来画柱状图的数据。它的含义很直接：每一行代表“某一个品种在某一个特征上的平均值”。

这里出现了两个后面经常会见到的 Pandas 操作：

- `melt()`：把“宽表”转换成“长表”。原来 4 个特征各占一列，转换后变成一列 `feature` 记录特征名称、一列 `value` 记录数值。
- `groupby().agg()`：先按 `species` 和 `feature` 分组，再用 `mean` 计算每组均值。这里得到的就是后续柱状图的柱高。

---

# 1. 先认识全局样式设置

## 1. 为什么需要全局样式？

在 Matplotlib 中，`plt.rcParams.update()` 可以理解为“全局绘图设置”。它的作用是：在后续所有图中，统一字体、字号、刻度线、边框、数学公式样式等基础参数。

如果不使用全局样式，每画一张图都需要单独指定字体、字号、边框等细节，代码会变得非常冗余且难以维护。全局样式的核心价值在于：**先定好一套基础审美，后面只对个别图表做局部微调**。

在引入全局样式之前，我们先用一段基础绘图代码生成一张原始图表，作为后续对比的参照 (绘图参数将在下一章详细介绍)。

```python
one_species = "setosa"

plot_df = (
    summary_df[summary_df["species"] == one_species]
    .set_index("feature")
    .loc[feature_order]
    .reset_index()
)

fig, ax = plt.subplots(figsize=(6, 4))

bars = ax.bar(
    plot_df["feature"],
    plot_df["mean_value"],
    color="#4C72B0",
    edgecolor="black",
    linewidth=0.6
)

ax.set_title("Setosa Feature Statistics", pad=12)
ax.set_xlabel("Feature", labelpad=10)
ax.set_ylabel("Mean value (cm)", labelpad=10)

ax.tick_params(axis="x", labelsize=12)
ax.tick_params(axis="y", labelsize=14)

plt.tight_layout()
plt.savefig(output_dir / "01_setosa_before_global_style.png", dpi=300, bbox_inches="tight")
plt.show()
```

![Setosa品种特征均值柱状图 (优化前) ](/python绘图/01_setosa_before_global_style.png)

从输出结果来看，图表整体结构完整，但字体、字重、刻度线、边框等元素还没有形成统一风格。如果后续要绘制多张图，每一张都手动设置这些细节，会很容易漏掉参数。

## 1.2 设置论文风格的全局参数

下面这套配置面向学术论文风格设计：采用 Times New Roman 字体、全局加粗、关闭网格线、保留坐标轴边框并统一刻度线粗细。为便于读者复制，代码本身保持简洁，每个参数的具体含义放在后面的表格里解释。

```python
plt.rcParams.update({
    "font.family": "Times New Roman",
    "font.weight": "bold",
    "axes.labelweight": "bold",
    "axes.titleweight": "bold",

    "font.size": 12,
    "axes.labelsize": 16,
    "xtick.labelsize": 14,
    "ytick.labelsize": 14,
    "legend.fontsize": 14,

    "xtick.minor.size": 0,
    "ytick.minor.size": 3,
    "xtick.major.size": 6,
    "ytick.major.size": 6,
    "xtick.major.width": 1,
    "ytick.major.width": 1,
    "xtick.minor.width": 1,
    "ytick.minor.width": 1,

    "axes.grid": False,
    "axes.edgecolor": "gray",
    "axes.linewidth": 1,
    "axes.spines.left": True,
    "axes.spines.right": True,
    "axes.spines.top": True,
    "axes.spines.bottom": True,

    "mathtext.default": "it",
    "mathtext.fontset": "stix",
    "axes.unicode_minus": False
})
```

上述参数可归纳为三大类别，读者在实际调整时，可直接参照下方表格查阅。

- **第一类：字体**

    | 参数 | 取值 | 示例 | 功能 |
    | :--- | :--- | :--- | :--- |
    | `font.family` | `'sans-serif'`、`'serif'`、`'monospace'`、`'cursive'`、`'fantasy'` 或具体字体名 (如 `'SimHei'`、`'Arial'`)  | `rcParams['font.family'] = 'sans-serif'` | 控制全局基础字体类型 |
    | `font.weight` | `'normal'`、`'bold'`、`'heavy'`、`'light'` 或数字权重 (100-900)  | `rcParams['font.weight'] = 'bold'` | 控制全局默认字体粗细 |
    | `axes.labelweight` | `'normal'`、`'bold'` | `rcParams['axes.labelweight'] = 'bold'` | 控制坐标轴标签 (xlabel/ylabel) 的加粗效果 |
    | `axes.titleweight` | `'normal'`、`'bold'` | `rcParams['axes.titleweight'] = 'normal'` | 控制坐标轴标题 (title) 的加粗效果 |
    | `font.size` | 浮点数 (如 `10.0`、`12.0`) 或相对值 (`'xx-small'`、`'x-small'`、`'small'`、`'medium'`、`'large'`、`'x-large'`、`'xx-large'`)  | `rcParams['font.size'] = 10` | 控制全局基础字号 (其他大小多以此为基准缩放)  |
    | `axes.labelsize` | 浮点数或相对值 (如 `'medium'`、`'large'`)  | `rcParams['axes.labelsize'] = 'large'` | 控制坐标轴标签 (xlabel/ylabel) 的字号大小 |
    | `xtick.labelsize` | 浮点数或相对值 | `rcParams['xtick.labelsize'] = 'medium'` | 控制 X 轴刻度标签的字号大小 |
    | `ytick.labelsize` | 浮点数或相对值 | `rcParams['ytick.labelsize'] = 'medium'` | 控制 Y 轴刻度标签的字号大小 |
    | `legend.fontsize` | 浮点数或相对值 (如 `'small'`、`'x-large'`)  | `rcParams['legend.fontsize'] = 'small'` | 控制图例 (legend) 的字号大小 |

- **第二类：边框与刻度线**

    | 参数 | 取值 | 参数示意 | 功能 |
    | :--- | :--- | :--- | :--- |
    | `xtick.major.size` | 浮点数 (长度，单位：磅)  | `rcParams['xtick.major.size'] = 3.5` | 控制 X 轴主刻度线的长度 |
    | `ytick.major.size` | 浮点数 (长度，单位：磅)  | `rcParams['ytick.major.size'] = 3.5` | 控制 Y 轴主刻度线的长度 |
    | `xtick.major.width` | 浮点数 (线宽，单位：磅)  | `rcParams['xtick.major.width'] = 0.8` | 控制 X 轴主刻度线的粗细 |
    | `ytick.major.width` | 浮点数 (线宽，单位：磅)  | `rcParams['ytick.major.width'] = 0.8` | 控制 Y 轴主刻度线的粗细 |
    | `axes.grid` | 布尔值 `True` / `False`，或字典 (如 `{'color': 'gray', 'linestyle': '--'}`)  | `rcParams['axes.grid'] = True` | 控制是否显示背景网格线 |
    | `axes.edgecolor` | 颜色字符串 (如 `'black'`、`'white'`、`'#FF0000'`)  | `rcParams['axes.edgecolor'] = 'black'` | 控制坐标轴边框 (四周边框线) 的颜色 |
    | `axes.linewidth` | 浮点数 (线宽，单位：磅)  | `rcParams['axes.linewidth'] = 0.8` | 控制坐标轴边框 (四周边框线) 的粗细 |
    | `axes.spines.*`  (含 `top`、`bottom`、`left`、`right`)  | 布尔值 (显示/隐藏) 或颜色值 / 线宽 (结合字典设置)  | `rcParams['axes.spines.top'] = False` | 分别控制上、下、左、右四条边框脊线的显示状态与样式 |

- **第三类：数学符号**

    | 参数 | 取值 | 参数示意 | 功能 |
    | :--- | :--- | :--- | :--- |
    | `mathtext.default` | `'it'` (斜体) 、`'rm'` (罗马体) 、`'bf'` (粗体) 、`'cal'` (书法体) 、`'tt'` (打字机体)  | `rcParams['mathtext.default'] = 'it'` | 控制数学公式中字母的默认字体样式 (如变量斜体)  |
    | `mathtext.fontset` | `'cm'` (Computer Modern) 、`'stix'`、`'stixsans'`、`'custom'` | `rcParams['mathtext.fontset'] = 'stix'` | 控制数学公式渲染所使用的整套字体集 |
    | `axes.unicode_minus` | 布尔值 `True` / `False` | `rcParams['axes.unicode_minus'] = False` | 控制负号显示 (设为 `False` 时使用标准 ASCII 减号，可解决负号显示为方块乱码的问题)  |

---

# 2. 先画一个柱状图

设置完全局样式后，我们使用相同的数据与代码，以 `setosa` 品种为例，对常用的单个子图绘图语法进行逐步讲解。后续的优化步骤均将在此基础上进行叠加与完善，以便清晰呈现每一处调整对图表效果的影响。

```python
# ---------- 数据准备阶段 ----------
# 指定要绘制的品种
one_species = "setosa"

# 从汇总表中筛选该品种的数据，并调整行索引顺序
plot_df = (
    summary_df[summary_df["species"] == one_species]  # 筛选出 setosa 品种的行
    .set_index("feature")                             # 将 "feature" 列设为行索引
    .loc[feature_order]                               # 按预设的特征顺序 (花萼长/宽、花瓣长/宽) 重新排序行
    .reset_index()                                    # 将行索引重置为普通列，还原数据框格式
)

# ---------- 创建画布 ----------
# fig: 整个图形窗口；ax: 单个坐标系 (axes) 对象
fig, ax = plt.subplots(figsize=(6, 4))  # 宽6英寸，高4英寸

# ---------- 绘制柱状图 ----------
bar_color = "#4C72B0"  # 定义统一柱子颜色 (十六进制蓝色) 

# ax.bar() 返回的是一个 BarContainer 对象，包含所有柱子的集合
bars = ax.bar(
    plot_df["feature"],      # x轴数据：四个特征的名称
    plot_df["mean_value"],   # y轴数据：对应的平均值
    color=bar_color,         # 柱子填充色
    edgecolor="black",       # 柱子边框颜色
    linewidth=0.6            # 柱子边框粗细
)

# ---------- 设置标题与轴标签 ----------
ax.set_title("Setosa Feature Statistics", pad=12,size=18)  # pad=12 表示标题与绘图区域顶部之间的间距;size=18 表示标题字号
ax.set_xlabel("Feature", labelpad=10)              # labelpad 控制轴标签与刻度标签的距离
ax.set_ylabel("Mean value (cm)", labelpad=10)

# ---------- 调整刻度样式 ----------
# tick_params() 专门控制刻度线和刻度标签的外观
ax.tick_params(axis="x", labelsize=12)             # x轴：刻度标签字号12
ax.tick_params(axis="y", labelsize=14)             # y轴：刻度标签字号14 (刻度线沿用全局默认长度) 

# ---------- 自动调整布局并显示 ----------
plt.tight_layout()  # 自动调整子图参数，使标签和标题不被裁切
plt.savefig(output_dir / "02_setosa_global_style.png", dpi=300, bbox_inches="tight") # 保存图片并自动识别保存格式，bbox_inches="tight" 使标签和标题不被裁切
plt.show()
```

这段代码运行的结果图如下所示，通过设定全局风格，已经能较为美观地展示 setosa 的基本信息了，但还可以进行进一步的调整。

![Setosa品种特征均值柱状图](/python绘图/02_setosa_global_style.png)

<details>
<summary style="font-weight: 600; color: #2c3e50; cursor: pointer; font-size: 1.05em; border-bottom: 1px dashed #bdc3c7; padding-bottom: 4px;">点此展开：十六进制颜色代码 <code>#RRGGBB</code> 的构成原理</summary>

<div style="margin-top: 16px; padding-left: 8px;">

<p>总的来说，这是一种能够精确指定颜色的方式，每个代码对应的颜色都是唯一的。上述代码中的 <code>"#4C72B0"</code> 是一种以十六进制数编码的 RGB 颜色值。其结构可拆解为三个通道：</p>

<ul>
    <li><strong><code>#4C</code></strong> —— 红色 (Red) 通道，十六进制值 0x4C = 十进制 76；</li>
    <li><strong><code>#72</code></strong> —— 绿色 (Green) 通道，十六进制值 0x72 = 十进制 114；</li>
    <li><strong><code>#B0</code></strong> —— 蓝色 (Blue) 通道，十六进制值 0xB0 = 十进制 176。</li>
</ul>

<p>每个通道的取值范围为 <code>00</code> 至 <code>FF</code> (十进制 0–255) ，分别对应显示器中该原色子像素的亮度等级。<code>#000000</code> 表示纯黑 (三通道全关) ，<code>#FFFFFF</code> 表示纯白 (三通道全开) 。</p>

<p><strong>获取途径</strong>：多数专业配色工具 (如 Adobe Color、Coolors) 及图像处理软件 (如 Photoshop、GIMP) 均以色十六进制码为主要输出格式，可直接复制使用。另需注意，Matplotlib 亦接受 <code>(R, G, B)</code> 元组形式，其中 R/G/B 为 <code>0–1</code> 浮点数，例如 <code>(0.298, 0.447, 0.690)</code> 与 <code>#4C72B0</code> 等价。</p>

<p><strong>扩展</strong>：部分场景下会见到 <code>#RRGGBBAA</code> 八位写法，末尾两位 <code>AA</code> 表示 Alpha 透明度通道，00 为全透明，FF 为完全不透明。Matplotlib 的 <code>color</code> 参数支持该格式。</p>

</div>
</details>

## 2.1 调整柱状图的统一颜色

若仅需为所有柱子赋予统一的填充色，最直接的途径是修改 `bar_color` 变量的赋值。请将上一节绘图代码中的该变量依次替换为以下四种取值，并重新运行代码，以观察不同色值对图表视觉风格的直接影响。

```python
bar_color = "#4C72B0"  # 蓝色
bar_color = "#55A868"  # 绿色
bar_color = "#C44E52"  # 红色
bar_color = "gray"     # 灰色
```

下面的代码将同一份 `setosa` 数据绘制成 2×2 分面图，只用于预览不同颜色的视觉效果，本处仅作为展示，不涉及组合图的完整绘制逻辑，组合图的具体构建方法将在第四节系统讲解。

建议读者以 setosa 单品种柱状图代码为基础模板，自行尝试修改柱状图的填充颜色、边框色或透明度等参数。通过反复练习颜色调整，有助于加深对 Matplotlib 颜色体系与绘图接口的理解，也为后续组合图的个性化配色打下基础。

```python
# 四种颜色方案预览
fig, axes = plt.subplots(2, 2, figsize=(10, 8))
axes = axes.flatten()

colors = ["#4C72B0", "#55A868", "#C44E52", "gray"]

for idx, ax in enumerate(axes):
    bars = ax.bar(
        plot_df["feature"],
        plot_df["mean_value"],
        color=colors[idx],
        edgecolor="black",
        linewidth=0.6
    )
    ax.set_title('', pad=10, fontsize=12)
    ax.set_ylabel("Mean value (cm)" if idx == 0 or idx == 2 else "")
    ax.tick_params(axis="x", length=0, labelsize=11)
    ax.tick_params(axis="y", labelsize=12)

plt.tight_layout()
plt.savefig(output_dir / "03_setosa_color_options.png", dpi=300, bbox_inches="tight")
plt.show()
```

![四种不同配色的Setosa品种特征均值柱状图](/python绘图/03_setosa_color_options.png)

如果所有柱子使用同色填充，柱子间的区分度会比较弱。第 3 节会在此基础上继续引入分组配色，为不同特征分配独立颜色。

## 2.2 刻度线调整

刻度文字和刻度线是分开调整的。例如，x 轴下方的 `Sepal length` 属于刻度标签 (tick label) ，而标签下方的小短线才是刻度线 (tick mark) 。两者的控制参数相互独立，可以根据需要分别调整。

**核心方法**：`ax.tick_params()` 用于控制当前坐标轴的刻度和标签样式。

| 参数 | 作用 | 示例 |
| :--- | :--- | :--- |
| `axis` | 指定操作哪个轴 | `"x"`、`"y"`、`"both"` |
| `length` | 刻度线长度 (单位：pt)  | `length=0` 表示隐藏刻度线 |
| `width` | 刻度线粗细 (单位：pt)  | `width=1` |
| `labelsize` | 刻度标签的字号 | `labelsize=12` |

实际应用如下：

```python
# 保留 x 轴刻度文字，但隐藏刻度线
ax.tick_params(axis="x", length=0)

# y 轴刻度线加长并加粗
ax.tick_params(axis="y", length=6, width=1)
```

这与全局设置中的 `xtick.major.size`、`ytick.major.size` 属于同一类控制项，区别在于作用范围：

- `plt.rcParams.update()` 是**全局设置**，一旦设定，影响后续所有图表。
- `ax.tick_params()` 是**局部设置**，仅对当前 `ax` 对象生效，优先级高于全局设置。

这种分层设计允许我们在全局统一风格的基础上，对特定图表进行精细微调。下面是将刻度调整应用到初始柱状图后的完整代码。

```python
fig, ax = plt.subplots(figsize=(6, 4))

bars = ax.bar(
    plot_df["feature"],
    plot_df["mean_value"],
    color="#4C72B0",
    edgecolor="black",
    linewidth=0.6
)


ax.set_title("Setosa Feature Statistics", pad=12, size=18)
ax.set_xlabel("Feature", labelpad=10)
ax.set_ylabel("Mean value (cm)", labelpad=10)

ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=14)

plt.tight_layout()
plt.savefig(output_dir / "04_setosa_tick_params.png", dpi=300, bbox_inches="tight") # 图片自动调整布局并导出
plt.show()
```
![刻度线调整后的Setosa品种特征均值柱状图](/python绘图/04_setosa_tick_params.png)

各位读者可以多尝试对上述 `length`、`width` 等参数进行微调，观察不同取值对图表视觉风格的影响，从而加深对刻度控制的理解。

## 2.3 坐标轴范围调整

柱状图最常见的问题之一是柱顶太贴近图的上边界。可以用 `set_ylim()` 给上方留一点空间。

```python
ax.set_ylim(0, 8)
```

这里的意思是：y 轴从 0 到 8。Iris 数据集的测量值单位是 cm，最大均值大约在 6 到 7 左右，所以设到 8 比较舒服。同理，这种操作也可以截断过长的柱子，优化数据的展示效果。

x 轴也可以调整边界：

```python
ax.set_xlim(-0.6, len(feature_order) - 0.4)
```

柱状图的 x 轴内部其实是 0、1、2、3 这样的数字位置。`-0.6` 表示最左边多留一点空白，`len(feature_order) - 0.4` 表示最右边多留一点空白。

如果只想固定坐标轴的起点，而让终点自动根据数据决定，可以写成：

```python
ax.set_ylim(0, None)
ax.set_xlim(0, None)
```
下面是将坐标轴范围调整应用到初始柱状图后的完整代码。

```python
fig, ax = plt.subplots(figsize=(6, 4))

bars = ax.bar(
    plot_df["feature"],
    plot_df["mean_value"],
    color="#4C72B0",
    edgecolor="black",
    linewidth=0.6
)

ax.set_title("Setosa Feature Statistics", pad=12, size=18)
ax.set_xlabel("Feature", labelpad=10)
ax.set_ylabel("Mean value (cm)", labelpad=10)

ax.set_ylim(0, 8)
ax.set_xlim(-0.6, len(feature_order) - 0.4)

ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=14)

plt.tight_layout()
plt.savefig(output_dir / "05_setosa_axis_limits.png", dpi=300, bbox_inches="tight")
plt.show()
```

![坐标轴范围调整后的Setosa品种特征均值柱状图](/python绘图/05_setosa_axis_limits.png)

## 2.4 调整标题和轴标题

标题和轴标题常用 3 个函数控制：

```python
ax.set_title("Setosa Feature Statistics", pad=12)
ax.set_xlabel("Feature", labelpad=10)
ax.set_ylabel("Mean value (cm)", labelpad=10)
```

其中：

- `set_title()` 控制主标题。
- `set_xlabel()` 控制 x 轴标题。
- `set_ylabel()` 控制 y 轴标题。
- `loc` 控制标题位置，可选 `"left"`、`"center"`、`"right"`。
- `pad` 控制标题和绘图区之间的距离。
- `labelpad` 控制轴标题和刻度文字之间的距离。

下面是将标题和轴标题调整应用到初始柱状图后的完整代码。

```python
fig, ax = plt.subplots(figsize=(6, 4))

bars = ax.bar(
    plot_df["feature"],
    plot_df["mean_value"],
    color="#4C72B0",
    edgecolor="black",
    linewidth=0.6
)

ax.set_title("Setosa Iris Feature Statistics", loc="center", pad=16, size=18)
ax.set_xlabel("Iris feature", labelpad=12)
ax.set_ylabel("Mean value (cm)", labelpad=12)

ax.set_ylim(0, 8)
ax.set_xlim(-0.6, len(feature_order) - 0.4)

ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=14)

plt.tight_layout()
plt.savefig(output_dir / "06_setosa_title_labels.png", dpi=300, bbox_inches="tight")
plt.show()
```

![标题和轴标题调整后的Setosa品种特征均值柱状图](/python绘图/06_setosa_title_labels.png)


## 2.5 调整边框

有些期刊风格喜欢保留完整边框，有些喜欢只保留左边和下边。我们可以通过 `spines` 控制四条边框。

如果去掉上边和右边：

```python
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)
```

如果四条边框都去掉：

```python
for spine in ax.spines.values():
    spine.set_visible(False)
```

如果你希望保留边框，但边框不能太显眼，可以只把边框颜色设为灰色、线宽设为 1。

```python
for spine in ax.spines.values():
    spine.set_color("gray")
    spine.set_linewidth(1)
```

下面分别展示这些调整。

```python
# ---------- 创建 1 行 3 列的分面图 ----------
fig, axes = plt.subplots(1, 3, figsize=(15, 4.5))

# ===== 子图 1：去掉上边和右边 =====
ax = axes[0]
ax.bar(plot_df["feature"], plot_df["mean_value"], color="#4C72B0", edgecolor="black", linewidth=0.6)
ax.set_title("Top & Right Off", fontsize=14, pad=12)
ax.set_xlabel("Iris feature", labelpad=10)
ax.set_ylabel("Mean value (cm)", labelpad=10)
ax.set_ylim(0, 8)
ax.set_xlim(-0.6, len(feature_order) - 0.4)
ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=12)
ax.spines["top"].set_visible(False)      # 去掉上边
ax.spines["right"].set_visible(False)    # 去掉右边

# ===== 子图 2：全部去掉 =====
ax = axes[1]
ax.bar(plot_df["feature"], plot_df["mean_value"], color="#4C72B0", edgecolor="black", linewidth=0.6)
ax.set_title("All Off", fontsize=14, pad=12)
ax.set_xlabel("Iris feature", labelpad=10)
ax.set_ylabel("Mean value (cm)", labelpad=10)
ax.set_ylim(0, 8)
ax.set_xlim(-0.6, len(feature_order) - 0.4)
ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=12)
for spine in ax.spines.values():
    spine.set_visible(False)              # 四条边全隐藏

# ===== 子图 3：保留全部边框，但设为灰色、线宽 1 =====
ax = axes[2]
ax.bar(plot_df["feature"], plot_df["mean_value"], color="#4C72B0", edgecolor="black", linewidth=0.6)
ax.set_title("Gray Full Border", fontsize=14, pad=12)
ax.set_xlabel("Iris feature", labelpad=10)
ax.set_ylabel("Mean value (cm)", labelpad=10)
ax.set_ylim(0, 8)
ax.set_xlim(-0.6, len(feature_order) - 0.4)
ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=12)
for spine in ax.spines.values():
    spine.set_color("gray")               # 颜色变灰
    spine.set_linewidth(1)                # 线宽调细

# ---------- 自动调整布局并显示 ----------
plt.tight_layout()
plt.savefig(output_dir / "07_border_styles_comparison.png", dpi=300, bbox_inches="tight")
plt.show()
```

![不同边框类型的Setosa品种特征均值柱状图](/python绘图/07_border_styles_comparison.png)

## 2.6 添加柱顶数值

`ax.text()` 可以在图中任意位置添加文字，常用于为柱状图标注柱顶数值。具体做法分为三步：

1. 用 `bar.get_height()` 获取当前柱子的高度 (即数值) ；

2. 用 `bar.get_x() + bar.get_width() / 2` 计算柱子的水平中心坐标；

3. 将文字放置在 (中心横坐标, 高度 + 适当偏移) 处，即可实现柱顶数值标注。

```python
fig, ax = plt.subplots(figsize=(6, 4))

bars = ax.bar(
    plot_df["feature"],
    plot_df["mean_value"],
    color="#4C72B0",
    edgecolor="black",
    linewidth=0.6
)

# 在每个柱子的正上方添加数值标签，显示该柱的精确高度 (保留两位小数) 
for bar in bars:
    height = bar.get_height()
    ax.text(
        # 水平居中：柱子左边缘 + 半宽
        bar.get_x() + bar.get_width() / 2,  
        # 垂直位置：顶部上方留出 0.08 的间距，避免与柱顶重叠
        height + 0.08,                      

        # 生成柱顶显示的文字：开头的 f 表示“这是一个能填变量的文本”，
        # 里面的 {height} 会被替换成真实的高度数值，
        # 冒号后面的 .2f 表示“固定保留两位小数” (f 代表小数，.2 代表两位) 。
        f"{height:.2f}",                    

        # 水平对齐：居中
        ha="center", 
        # 垂直对齐：文本底部对齐指定坐标，确保文字从该点向上绘制                       
        va="bottom",                        
        fontsize=11,
        fontweight="bold"
    )

ax.set_title("Setosa Iris Feature Statistics", loc="center", pad=16, size=18)
ax.set_xlabel("Iris feature", labelpad=12)
ax.set_ylabel("Mean value (cm)", labelpad=12)

ax.set_ylim(0, 8)
ax.set_xlim(-0.6, len(feature_order) - 0.4)

ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=14)

for spine in ax.spines.values():
    spine.set_color("gray")
    spine.set_linewidth(1)

plt.tight_layout()
plt.savefig(output_dir / "07_setosa_single_color_optimized.png", dpi=300, bbox_inches="tight")
plt.show()
```

![添加柱顶数值后的Setosa品种特征均值柱状图](/python绘图/07_setosa_single_color_optimized.png)

---

# 3. 按特征区分柱子颜色，并创建图例

在上一章的柱状图中，所有柱子都是同一种颜色，这种绘图方式对不同类别数据的区分度有限。这一章给每个特征分配不同颜色，并生成图例，生成图例的方式包含自动和手动两种。

## 3.1 颜色分配

Seaborn 的 `deep` 调色板比较适合入门使用，颜色区分明显，也不会太刺眼。

```python
palette = sns.color_palette("deep", n_colors=len(feature_order))
color_map = dict(zip(feature_order, palette))
bar_colors = [color_map[feature] for feature in plot_df["feature"]]
```

这三行代码做了三件事：

1. `sns.color_palette()` 从 Seaborn 中取出 4 个颜色。
2. `dict(zip(...))` 把 4 个特征名称和 4 个颜色一一配对，生成颜色字典。
3. `bar_colors = [...]` 按照当前数据中的特征顺序，为每根柱子取出对应颜色。

其中 `zip()` 可以简单理解为“并排配对”。例如第一个特征 `Sepal length` 会配到第一个颜色，第二个特征 `Sepal width` 会配到第二个颜色。

如果你希望颜色完全固定，也可以不用调色板，而是直接写十六进制颜色：

```python
color_map = {
    "Sepal length": "#4C72B0",
    "Sepal width": "#55A868",
    "Petal length": "#C44E52",
    "Petal width": "#8172B3"
}

bar_colors = [color_map[feature] for feature in plot_df["feature"]]
```

十六进制颜色的好处是稳定、可复制。只要颜色代码不变，每次出图的颜色就不会变化。这里为了延续 Seaborn 的整体风格，下面的完整代码仍然使用 `sns.color_palette("deep")`，各位读者可以自行修改后面的绘图参数，体验不同的配色方案。

## 3.2 基础版：自动图例

调用画图函数 `ax.bar()` 的 `label` 参数可以传入指定绘图数据的分组，例如：`label=feature_order`。这个参数不是给整张图一个名字，而是把 4 个特征名按顺序分别贴到 4 根柱子上。再调用函数 `ax.legend()`，Matplotlib 就会自动收集这些标签并生成图例。

```python
palette = sns.color_palette("deep", n_colors=len(feature_order))
color_map = dict(zip(feature_order, palette))
bar_colors = [color_map[feature] for feature in plot_df["feature"]]

fig, ax = plt.subplots(figsize=(7, 4))

bars = ax.bar(
    plot_df["feature"],
    plot_df["mean_value"],
    color=bar_colors,
    edgecolor="black",
    linewidth=0.6,
    label=feature_order  # 让每根柱子按顺序带上自己的特征名
)

for bar in bars:
    height = bar.get_height()
    ax.text(
        bar.get_x() + bar.get_width() / 2,
        height + 0.08,
        f"{height:.2f}",
        ha="center",
        va="bottom",
        fontsize=11,
        fontweight="bold"
    )

ax.set_title("Setosa Iris Feature Statistics", loc="center", pad=16, size=18)
ax.set_xlabel("Iris feature", labelpad=12)
ax.set_ylabel("Mean value (cm)", labelpad=12)

ax.set_ylim(0, 8)
ax.set_xlim(-0.6, len(feature_order) - 0.4)

ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=14)

for spine in ax.spines.values():
    spine.set_color("gray")
    spine.set_linewidth(1)

# 自动图例：不传 handles，Matplotlib 会从带 label 的柱子中收集图例条目
# 图例绘制函数的细节详见3.3.2节，此处仅用作展示

ax.legend(
    title="Feature",
    frameon=False,
    bbox_to_anchor=(1.02, 1),
    loc="upper left",
    title_fontsize=14
)

plt.tight_layout()
plt.savefig(output_dir / "08_setosa_feature_colors.png", dpi=300, bbox_inches="tight")
plt.show()
```

![自动图例版本的Setosa品种特征均值柱状图](/python绘图/08_setosa_feature_colors.png)


## 3.3 进阶版：手动创建图例

尽管用 `ax.bar()` 也可以自动生成图例，但自动图例的内容完全来自已经画出的柱子。如果你想控制图例顺序、色块边框，或者只显示部分特征，就需要手动创建图例。

### 3.3.1 创建图例

这里采用手动创建图例的方式：先用 `Patch` 创建色块，再把这些色块作为 `handles` 传给 `ax.legend()`。`Patch` 已经在文章开头导入，如果单独运行本节代码，需要补上：

```python
from matplotlib.patches import Patch
```

手动创建图例色块的写法如下：

```python
# Patch 不会出现在数据区，只负责在图例中显示颜色和标签
legend_handles = [
    Patch(
        facecolor=color_map[feature],  # 图例色块填充色，与柱子颜色保持一致
        edgecolor="black",             # 图例色块边框，提升辨识度
        label=feature                  # 图例中显示的标签文字
    )
    for feature in feature_order       # 按 feature_order 的顺序生成图例
]
```

这里的 `legend_handles` 可以理解为“图例条目列表”。`facecolor` 控制色块颜色，`edgecolor` 控制色块边框颜色，`label` 控制图例文字。因为图例色块和标签列表均由 `feature_order` 生成，所以顺序也会和 `feature_order` 保持一致，而不会受到绘图顺序的影响。

至此，用于绘制图例的列表已经创建完毕，我们需要将创建好的图例画到画布上。

### 3.3.2 绘制图例

图例位置主要由两个参数决定：

```python
loc="upper left"
bbox_to_anchor=(1.02, 1)
```

可以把 `bbox_to_anchor` 理解为“图例要贴到的坐标”，把 `loc` 理解为“图例自己的哪个位置去贴这个点”。

例如：

```python
bbox_to_anchor=(1.02, 1)
loc="upper left"
```

这表示把图例的左上角，贴到坐标轴右上方稍微偏外的位置。这里的 `1.02` 比 `1` 稍大，所以图例会被放到绘图区右侧外面；`1` 表示靠近绘图区顶部。

常见位置可以这样调整：

| 写法 | 效果 |
|------|------|
| `bbox_to_anchor=(1.02, 1)` | 放在图的右上方外侧 |
| `bbox_to_anchor=(1.02, 0.5)` | 放在图的右侧中间 |
| `bbox_to_anchor=(0.5, -0.15)` | 放在图的下方中间 |

总而言之，图例常用参数如下：

| 参数 | 作用 | 示例 |
|------|------|------|
| `handles` | 指定手动创建的图例条目 | `handles=legend_handles` |
| `title` | 设置图例标题 | `title="Feature"` |
| `frameon` | 是否显示图例外框 | `frameon=False` |
| `loc` | 控制图例自身的对齐位置 | `loc="upper left"` |
| `bbox_to_anchor` | 控制图例锚点位置 | `bbox_to_anchor=(1.02, 1)` |
| `ncol` | 设置图例列数 | `ncol=2` |
| `handlelength` | 调整色块长度 | `handlelength=1.2` |
| `labelspacing` | 调整图例条目间距 | `labelspacing=0.5` |

如果图例放在图外面，保存图片时建议加上：

```python
plt.savefig("figure.png", dpi=300, bbox_inches="tight")
```

其中 `bbox_inches="tight"` 可以避免图例被裁掉。尤其是图例放在右侧或下方时，这个参数非常实用。

下面是按特征配色并绘制图例后的完整代码。

```python
palette = sns.color_palette("deep", n_colors=len(feature_order))
color_map = dict(zip(feature_order, palette))
bar_colors = [color_map[feature] for feature in plot_df["feature"]]

fig, ax = plt.subplots(figsize=(7, 4))

bars = ax.bar(
    plot_df["feature"],
    plot_df["mean_value"],
    color=bar_colors,
    edgecolor="black",
    linewidth=0.6
)

for bar in bars:
    height = bar.get_height()
    ax.text(
        bar.get_x() + bar.get_width() / 2,
        height + 0.08,
        f"{height:.2f}",
        ha="center",
        va="bottom",
        fontsize=11,
        fontweight="bold"
    )

ax.set_title("Setosa Iris Feature Statistics", loc="center", pad=16, size=18)
ax.set_xlabel("Iris feature", labelpad=12)
ax.set_ylabel("Mean value (cm)", labelpad=12)

ax.set_ylim(0, 8)
ax.set_xlim(-0.6, len(feature_order) - 0.4)

ax.tick_params(axis="x", length=0, labelsize=12)
ax.tick_params(axis="y", length=6, width=1, labelsize=14)

for spine in ax.spines.values():
    spine.set_color("gray")
    spine.set_linewidth(1)

# 创建图例
legend_handles = [
    Patch(
        facecolor=color_map[feature],
        edgecolor="black",
        label=feature
    )
    for feature in feature_order
]

# 绘制图例
ax.legend(
    handles=legend_handles,      # 使用手动创建的图例条目
    title="Feature",             # 图例标题
    frameon=False,               # 不显示图例外框
    bbox_to_anchor=(1.02, 1),     # 将图例放到坐标轴右侧
    loc="upper left",            # 图例左上角对齐到锚点
    title_fontsize=14,
    handlelength=1.2,            # 控制色块长度
    labelspacing=0.5             # 控制图例条目之间的距离
)

plt.tight_layout()
plt.savefig(output_dir / "08_setosa_feature_colors2.png", dpi=300, bbox_inches="tight")
plt.show()
```

![按特征配色后的Setosa品种特征均值柱状图](/python绘图/08_setosa_feature_colors2.png)

---

# 4. 绘制所有品种的组合图

现在我们已经能画单个品种了，下一步就是把三个品种组合起来。

组合图的核心思想是：**先创建多个子图，再画到一个大图里**。

## 4.1 组合图的基本写法

组合图相比单图，新增的核心只有一行——plt.subplots()：

```python
fig, axes = plt.subplots(nrows=3, ncols=1)
```

这行代码会一次性创建 3 个坐标系。可以简单理解为：

- `fig`：整张大图。
- `axes`：多个小图组成的容器。
- `nrows`：子图行数。
- `ncols`：子图列数。

创建好画布后，下一步是将“每个品种”与“每个子图”一一对应。对于结构相同的多个子图，推荐用 zip() 配合循环来处理：

```python
for ax, species in zip(axes, species_order):
    # ax 是当前子图
    # species 是当前品种
    # 循环每运行一次，就把一个品种画到一个子图里
```

因此，多品种组合图的基本骨架可以写成：

```python
species_order = ["setosa", "versicolor", "virginica"]

fig, axes = plt.subplots(nrows=3, ncols=1, figsize=(7, 8))

for ax, species in zip(axes, species_order):
    sub_df = summary_df[summary_df["species"] == species]
    ax.bar(sub_df["feature"], sub_df["mean_value"])
```

上述代码仅展示核心结构，尚未加入排序、配色、图例、数值标签等细节。一旦掌握了这个骨架，后续只需在循环内部“填空”，即可逐步扩展成完整的可视化图表。

若不同子图之间结构差异较大 (例如第一个画折线图、第二个画散点图) ，则不宜使用循环，而是直接通过 `axes[0]`、`axes[1]` 等方式逐个调用并分别绘制，在此不做展开。

> **进阶建议**
> 当多个子图使用同质数据时，纵向组合图建议统一所有子图的 x 轴标签和刻度范围，横向组合图则建议统一 y 轴范围。否则各子图可能因数据范围不同而自动缩放坐标轴，导致同一数值在不同子图中柱高或位置不一致，造成视觉上的误判。可通过 sharex / sharey 参数或在循环后调用 set_xlim() / set_ylim() 来规避。

## 4.2 纵向组合图

掌握了上一节的基本骨架后，你可能已经跃跃欲试想自己动手写一遍了——非常推荐你先试着自己画一画，把整个流程跑通，这比光看代码印象深刻得多。

当然，从骨架到一张真正美观、信息完整的组合图，还需要补充不少细节：按数值排序、自定义配色、添加数值标签、设置图例和坐标轴标题等等。为了让你有一个完整的参照，在此附上纵向组合图的全部代码：

> 非常建议各位读者先独立完成一遍，遇到卡住的地方再对照这份完整代码找差异，这样学习效果最好。如果直接看完整代码也没问题，重点观察每个细节是如何在循环内部“嵌入”骨架的。


```python
species_order = ["setosa", "versicolor", "virginica"]

palette = sns.color_palette("deep", n_colors=len(feature_order))
color_map = dict(zip(feature_order, palette))
legend_handles = [
    Patch(facecolor=color_map[feature], edgecolor="black", label=feature)  # Patch 用于创建图例色块
    for feature in feature_order
]

fig, axes = plt.subplots(
    nrows=3,
    ncols=1,
    figsize=(7, 8),
    sharex=True,  # 所有子图共用 x 轴，避免重复显示不必要的信息
    sharey=True   # 所有子图共用 y 轴，方便直接比较柱高
)

for ax, species in zip(axes, species_order):  # zip 将每个子图 ax 与一个品种名称配对
    sub_df = (
        summary_df[summary_df["species"] == species]
        .set_index("feature")
        .loc[feature_order]
        .reset_index()
    )

    bar_colors = [color_map[feature] for feature in sub_df["feature"]]

    bars = ax.bar(
        sub_df["feature"],
        sub_df["mean_value"],
        color=bar_colors,
        edgecolor="black",
        linewidth=0.6,
        width=0.65
    )

    for bar in bars:
        height = bar.get_height()
        ax.text(
            bar.get_x() + bar.get_width() / 2,
            height + 0.08,
            f"{height:.2f}",
            ha="center",
            va="bottom",
            fontsize=10,
            fontweight="bold"
        )

    ax.set_title(species.capitalize(), loc="left", pad=8)
    ax.set_ylabel("Mean (cm)", labelpad=10)
    ax.set_ylim(0, 8)
    ax.set_xlim(-0.6, len(feature_order) - 0.4)

    ax.tick_params(axis="x", length=0, labelsize=12)
    ax.tick_params(axis="y", length=6, width=1, labelsize=14)

    for spine in ax.spines.values():
        spine.set_color("gray")
        spine.set_linewidth(1)

axes[-1].set_xlabel("Iris feature", labelpad=10)  # axes[-1] 表示最后一个子图，只在底部显示 x 轴标题

fig.suptitle(
    "Iris Feature Statistics by Species",
    y=0.98,  # y 控制总标题在整张图中的纵向位置
    fontsize=16,
    fontweight="bold"
)

fig.legend(
    handles=legend_handles,
    title="Feature",
    frameon=False,
    loc="center right",        # 图例自身的锚定位置
    bbox_to_anchor=(1.08, 0.5), # 图例相对于整张图的位置
    title_fontsize=14
)

fig.subplots_adjust(
    left=0.12,    # 左边界
    right=0.78,   # 右边界，给右侧图例留空间
    top=0.92,     # 顶部边界，给总标题留空间
    bottom=0.10,  # 底部边界，给 x 轴标题留空间
    hspace=0.35   # 子图之间的纵向间距
)

plt.savefig(output_dir / "09_iris_species_vertical_facets.png", dpi=300, bbox_inches="tight")
plt.show()
```

![鸢尾花品种纵向分面柱状图](/python绘图/09_iris_species_vertical_facets.png)

## 4.3 横向组合图

### 4.3 横向组合图

纵向组合图适合博客、论文等长条排版，而横向组合图则更适合 PPT 或宽屏页面——一左右一上下，核心思路完全相通，只是排列方向变了。

有了纵向组合图的基础，横向组合图就很好上手了，只需把子图的排列方向从“上下”改为“左右”。在动手写代码之前，先理清三个问题：

**1. 数据怎么放？**  
横向组合图通常用于对比不同类别在**同一指标**上的表现，比如三个品种的某种特征均值并排展示。你需要确保数据已按品种分组，且每个子图绘制的内容一致 (例如都是柱状图) 。

**2. 和纵向图有什么不同？**  
核心差异仅在 `plt.subplots()` 这一步：纵向是 `nrows=3, ncols=1`，横向则是 `nrows=1, ncols=3`——**行列互换**，子图从纵向堆叠变为横向并排，再适当调整图的大小。其余绘图逻辑 (循环、筛选数据、绘制柱状图) 完全一致。

**3. 需要注意什么？**  
横向组合图子图左右排列，空间比纵向更紧张，因此要注意：
- **y 轴标签** (如品种名) 通常只保留在最左侧的子图上，其余子图的 y 轴标签可以省略，否则重复的标签会显得拥挤冗余。
- **y 轴范围**建议统一，避免各子图自动缩放，导致同样的数值在不同子图中柱高不一，产生视觉误导。

建议各位读者先根据 4.1 节的骨架，把 `nrows` 和 `ncols` 对调，自己动手写一遍试试。遇到卡顿的地方，再回来对照下面的完整代码找差异。亲手跑通一遍，比只看代码印象深得多。

```python
fig, axes = plt.subplots(
    nrows=1,
    ncols=3,
    figsize=(12, 3.8),
    sharex=True,
    sharey=True
)

for ax, species in zip(axes, species_order):
    sub_df = (
        summary_df[summary_df["species"] == species]
        .set_index("feature")
        .loc[feature_order]
        .reset_index()
    )

    bar_colors = [color_map[feature] for feature in sub_df["feature"]]

    bars = ax.bar(
        sub_df["feature"],
        sub_df["mean_value"],
        color=bar_colors,
        edgecolor="black",
        linewidth=0.6,
        width=0.65
    )

    for bar in bars:
        height = bar.get_height()
        ax.text(
            bar.get_x() + bar.get_width() / 2,
            height + 0.08,
            f"{height:.2f}",
            ha="center",
            va="bottom",
            fontsize=9,
            fontweight="bold"
        )

    ax.set_title(species.capitalize(), pad=8)
    ax.set_ylim(0, 8)
    ax.tick_params(axis="x", length=0, labelsize=10, rotation=35) # rotation = 35 轴标签旋转35°
    ax.tick_params(axis="y", length=6, width=1, labelsize=12)

    for spine in ax.spines.values():
        spine.set_color("gray")
        spine.set_linewidth(1)

axes[0].set_ylabel("Mean (cm)", labelpad=10)  # 横向排布时，只给最左侧子图添加 y 轴标题

fig.suptitle(
    "Iris Feature Statistics by Species",
    y=1.02,
    fontsize=16,
    fontweight="bold"
)

fig.legend(
    handles=legend_handles,
    title="Feature",
    frameon=False,
    loc="center right",
    bbox_to_anchor=(1.02, 0.5),
    title_fontsize=12
)

fig.subplots_adjust(
    left=0.08,
    right=0.86,
    bottom=0.28,
    top=0.84,
    wspace=0.20  # 子图之间的横向间距
)

plt.savefig(output_dir / "10_iris_species_horizontal_facets.png", dpi=300, bbox_inches="tight")
plt.show()
```

![鸢尾花品种横向分面柱状图](/python绘图/10_iris_species_horizontal_facets.png)

### 4.4 控制子图间距：`hspace` 与 `wspace`

组合图最让人头疼的往往是子图之间的间距——太挤了标签重叠，太疏了又显得松散。`matplotlib` 提供了两个关键参数来解决这个问题：

- **纵向间距** (上下子图之间) 由 `hspace` 控制：
  ```python
  fig.subplots_adjust(hspace=0.35)
  ```
  `hspace` 值越大，上下子图之间的空白区域就越大。当子图有自己的标题或 x 轴标签时，适当增大 `hspace` 可以避免文字重叠。

- **横向间距** (左右子图之间) 由 `wspace` 控制：
  ```python
  fig.subplots_adjust(wspace=0.20)
  ```
  `wspace` 越大，左右子图之间的间隔越宽，适合放置共享图例或避免 y 轴标签相互遮挡。

需要特别说明的是：**这两个参数没有“最佳值”**，具体取值取决于多个因素——子图数量、标题字体大小、坐标轴标签长度、图例位置、整个画布的尺寸等。因此，最务实的做法是**先设一个初始值 (如 0.3) ，运行代码查看效果，再根据实际情况反复微调**，直到视觉上舒适为止。

如果你希望更直观地调整，也可以直接在 `plt.subplots()` 中通过 `gridspec_kw` 设置：
```python
fig, axes = plt.subplots(2, 2, gridspec_kw={'hspace': 0.3, 'wspace': 0.2})
```
这种方式与 `subplots_adjust` 等效，只是写法更集中。无论哪种方式，核心都是“试错+微调”，因为每个人的图表内容和尺寸都不一样，没有捷径可走。

---

# 写在最后

本文以柱状图为例进行演示，但核心并不在于柱状图本身，而在于背后通用的绘图思路：

1. **数据准备**：首先将原始数据整理成适合绘图的格式，这是所有可视化的基础。
2. **最小可行原型**：先绘制子图的最简版本 (如仅包含数据和坐标轴) ，不要在一开始就追求完美子图甚至复杂组合图。
3. **迭代优化**：逐步调整颜色、标题、坐标轴、边框、图例等参数。每次修改一项参数后，重新运行完整代码并观察效果，通过反复尝试逐步确定主题风格。
4. **扩展与组合**：将最终确定的风格推广到多个子图，再组合成完整的复合图表。
5. **全局布局**：利用 `hspace`、`wspace`、`left`、`right`、`top`、`bottom` 等参数精细控制子图之间的间距和整体边距。

在 Matplotlib 的学习中，有两道坎常常让新手卡住，它们的难度和应对策略其实完全不同：

- **数据处理**：依赖扎实的 pandas 基础，涉及清洗、聚合、重塑等操作，需要系统学习和持续积累，没有太多捷径可走。

- **可视化调整**：相比之下要直观得多——你改一个参数，图就变一个样，正反馈来得很快。针对这部分，通常有两种高效策略：

  - **模板复用**：找到一份相对接近需求的示例代码 (不一定是完整的组合图模板，也可以是某个子图的写法，甚至是多张图片拼凑的灵感) 。掌握关键参数后，将自己的数据套入模板，再按需微调细节，这是性价比最高的方式。

  - **AI 辅助**：如果找不到合适的现成模板，不妨先绘制一个最简版本 (只包含数据和基本坐标轴) ，然后把期望的效果用自然语言描述给 AI。在 AI 的引导下逐步添加参数、优化呈现。

掌握上述思路和工具，Matplotlib 将不再令人望而生畏，而是成为你表达数据故事的有力伙伴。

—— Aurora Hiker

---

# 附录

## 参考资料

- **Matplotlib 官方文档**：https://matplotlib.org/stable/contents.html  
  最权威的参考来源，涵盖所有绘图函数、参数说明及示例。

- **Pandas 官方文档**：https://pandas.pydata.org/docs/  
  数据处理与清洗的核心工具，本教程中所有数据预处理均依赖 Pandas。

- **NumPy 官方文档**：https://numpy.org/doc/  
  数值计算基础库，常用于生成示例数据或进行简单统计运算。


## 本教程的绘图脚本

本教程所有绘图代码均整理在以下脚本文件中，可逐段运行。建议将脚本与教程对照阅读，边看边改，以加深对图表调整与参数含义的理解。

[python绘图脚本.ipynb](/python绘图/python绘图脚本.ipynb)