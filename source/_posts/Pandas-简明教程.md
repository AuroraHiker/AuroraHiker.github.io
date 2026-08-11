---
title: Pandas 简明教程
tags:
  - Python 入门
series:
  - Python 入门
categories:
  - Python 入门
author: AuroraHiker
---

Pandas 是利用 Python 进行数据分析的基石，内置多种强大数据类型，在日常分析中，**DataFrame (数据框)** 的使用频率一骑绝尘，足以应对 80% 以上的实战需求。作为一份专为新手设计的入门级简明教程，本文将手把手带您走通 DataFrame 从读取、清洗、运算到导出的全流程，轻松上手最核心的数据分析技能。所有示例数据均采用经典的 Iris (鸢尾花) 数据集。

**建议尚未接触过 Python 语法的读者先完成 {% post_link Python-入门：零基础编程速通指南 %} 的学习，再回到这里。**

---

# 0. 准备工作：数据导入与查看

## 0.1 安装所需的库

> 小技巧：在 Jupyter Notebook 中，行首的感叹号 `!` 可以让 Python 识别并执行终端命令。因此无需切换窗口，直接在单元格中运行 `!pip install ...` 即可安装库。例如：

```python
!pip install pandas seaborn openpyxl
```

若在终端/命令行中执行 (不加感叹号) ，则使用：

```python
pip install pandas seaborn openpyxl
```

## 0.2 初识数据：载入并查看 Iris 数据集

拿到数据的第一步，不是急着处理，而是先 "看一眼"。我们使用 Seaborn 库内置的 Iris 数据。

```python
import pandas as pd

# 加载数据集
import seaborn as sns
iris = sns.load_dataset('iris')

# ---- 查看数据概览 ----
print(iris.head())      # 查看前 5 行 (可指定行数，如 head(10)) 
print(iris.info())      # 查看列名、非空数量及数据类型
print(iris.describe())  # 查看数值列的统计摘要 (均值、标准差、分位数)
print(iris.shape)       # 数据形状 (几行几列)
print(iris.columns)     # 查看列名
print(iris.index)       # 查看索引范围
print(iris.count())     # 快速统计非空值个数 (与 info 互补)
```

<details>
<summary><b>Debug: <code>sns.load_dataset('iris')</code> 失败 (Timeout Error) 怎么办？</b></summary>


**Bug 成因**

该错误通常由 Seaborn 无法从 GitHub 自动下载数据集引发。

**解决方案 (手动下载并放置到本地目录)**

1. **找到 Seaborn 的数据存放目录**  
   Windows 默认路径为：  
   `C:\Users\你的用户名\seaborn-data`  
    (若不存在，可手动创建该文件夹) 

2. **从 GitHub 获取 `iris.csv` 文件**  
   访问 [Seaborn 官方数据仓库](https://github.com/mwaskom/seaborn-data/)，找到 `iris.csv` 文件并下载。  
   若 GitHub 访问不稳定，推荐使用 **Watt Toolkit (加速器)** 改善连接，具体配置可参考 {% post_link Windows效率神器推荐-Wox-Markdown编辑器与GitHub加速器配置指南 %}。

   另外，出于交流学习的目的，本教程也提供该数据集的直接下载：  
   [iris.csv](/dataset/iris.csv)

3. **将下载的 `iris.csv` 放入 `seaborn-data` 目录**，之后重新运行 `sns.load_dataset('iris')` 即可正常加载。

</details>

---

## 0.3 导出本次教程所需的数据

为了便于后续演示从文件读取，我们先利用导出功能将数据保存为本地文件 (导出功能的详细用法将在最后一节介绍)。这里先执行保存操作：

```python
iris.to_csv('iris.csv', index=False)          # 带表头的标准 CSV
iris.to_csv('iris_no_header.csv', index=False, header=False)  # 不带表头的版本 (用于演示 header=None)
iris.to_csv('iris.tsv', sep='\t', index=False)  # 保存为 tsv
iris.to_excel('iris.xlsx', index=False)  # 保存为 excel
iris.to_csv('iris_index.csv')  # 有行索引的版本 (用于演示 指定行索引)
```

勘误：当 iris数据集导入失败时产生timeout error报错的解决方案：
bug原因：联网下载数据集失败
解决思路：手动下载数据集，存到seaborn包相应的目录中。
解决步骤：1.找到本地seaborn包存放数据的目录：C:\Users\用户名\seaborn-data
2.从github下载数据集：打开[seaborn官方项目](https://github.com/mwaskom/seaborn-data/),找到`iris.csv`文件打开，快捷键`ctrl+s`保存。使用Watt Toolkit加速器可以稳定访问GitHub，详见{% post_link Windows效率神器推荐-Wox-Markdown编辑器与GitHub加速器配置指南 %}。出于交流学习的目的，本教程也提供iris数据集的下载，供有需要的读者使用：iris.csv(插入下载索引)

3.下载后保存到`seaborn-data`目录即可。


---

# 1. 读取 / 创建 DataFrame

## 1.1 读取文件

Pandas 对常见表格格式的支持非常友好，只需一行代码就能导入。但现实中拿到的文件往往不规整，学会灵活配置参数，才是真正“会用”的开始。

先看三种基本读法：

```python
# CSV (逗号分隔，最常用) —— 读取带表头的 iris.csv
df_csv = pd.read_csv('iris.csv')

# TSV (制表符分隔，指定 sep 参数)
tsv = pd.read_csv('iris.tsv', sep='\t')

# Excel (需安装 openpyxl) 
df_excel = pd.read_excel('iris.xlsx', sheet_name='Sheet1')
```

其中 read_csv 是使用率最高的函数，下面介绍的是常用参数。read_excel 的用法也大致相同，区别主要在于多了 sheet_name 参数 (用于指定读取哪个工作表，可以传工作表名称、位置序号或 None 读取所有表)。掌握这些参数后, 就能轻松完成 CSV 和 Excel 文件的导入。更详细的参数说明，可查阅官方文档。

**列名处理**

1. **文件自带表头**。当首行就是列名时，保持默认即可 (`header=0`)，Pandas 会自动将第一行识别为列名。当第 x 行是列名，则 `header=x-1` (注意python是从0开始计数的)。
2. **文件没有表头 (数据从第一行就开始)**。务必加上 `header=None`，否则第一行数据会被“错认”成列名，导致数据少一行。
   ```python
   df = pd.read_csv('iris_no_header.csv', header=None)  # 列名变为默认值 0, 1, 2, 3, 4
   ```
3. **自定义列名 (覆盖原表头或为无表头文件命名)**。用 `names` 参数传入一个列表，它会强行指定数据框的列名。若文件有表头，需与 `header=0` 搭配, 它会直接覆盖原来的列名。
    ```python
    # 为无表头文件指定列名
    df = pd.read_csv('iris_no_header.csv', header=None,
                 names=['sepal_len', 'sepal_wid', 'petal_len', 'petal_wid', 'species'])
    # 覆盖有表头文件的列名 (如改为中文，方便阅读)
    df = pd.read_csv('iris.csv', names=['花萼长度', '花萼宽度', '花瓣长度', '花瓣宽度', '品种'], header=0)
    ```

**其他实用参数**

- **`index_col` (指定行索引)**：若文件中第 x 列可作为行的唯一标识 (如 `id` 列)，设置 `index_col=x-1` 即可将其设为行索引，后续按行取值会更方便。默认情况下，Pandas 会使用从 0 开始的数字序号作为行索引。
  ```python
  df = pd.read_csv('iris_index.csv', index_col=0)  # 将第一列 (id) 设为行索引，id 是唯一值，适合做索引
  ```
- **`usecols` (只读取部分列)**：当文件有几十列而你只需要 3 列时，用该参数能大幅减少内存占用。比如 `usecols=['sepal_length','species']` (按列名) 或 `usecols=[0, 4]` (按位置)。
    ```python
    # 只读取花萼长度和品种两列 (按列名)
    df = pd.read_csv('iris.csv', usecols=['sepal_length', 'species'])
    # 或按位置
    df = pd.read_csv('iris.csv', usecols=[0, 4])
    ```
- **`dtype` (预指定数据类型)**：防止 Pandas 自作聪明地把 "001" 读成数值 1，或把身份证号读成浮点数。传入字典即可，如 `dtype={'id': str, 'score': float}`。
  ```python
  df = pd.read_csv('iris.csv', dtype={'sepal_length': float, 'species': str})
  ```
- **`encoding` (编码格式)**：读取中文文件时容易遇到乱码，优先试 `encoding='utf-8'`，若报错或乱码依旧，换 `encoding='gbk'` 或 `encoding='utf-8-sig'` (后者对 Excel 导出的 CSV 更友好)。
- **`skiprows` (跳过开头若干行)**：如果文件前N行是注释或说明，用 `skiprows=N` 直接跳过。

下面是一个综合示例，把上面几个参数串起来，展示如何用 Iris 文件做一些定制化读取:

```python
# 综合实战：读取 iris.csv，自定义列名，将第一列设为行索引，
# 只读取花萼长度和品种两列，并强制指定数据类型
df_custom = pd.read_csv(
    'iris_index.csv',
    usecols=[1, 5],          # 筛选出第 2 列 (sepal_length) 和第 6 列 (species)
    header=0,                  # 指定表头行 (默认值为 0)
    names=['SL', 'SP'],   # 为筛选出的列依次命名 (必须与 usecols 的列数一致)
    index_col=0,                # 将第一列 (id) 设为行索引
    dtype={'SL': float, 'SP': str}   # 指定类型
)
print(df_custom.head())
```

学会这些参数，无论面对的是干净的标准数据集，还是从各处收集来的格式未统一的文件，你都能顺利导入 Pandas 中，为后续分析铺平道路。

## 1.2 从头构建

当手头有少量测试数据时，可以直接在代码里创建：

```python
# 方式一：字典 (键为列名，值为列表)
data_dict = {
    'sepal_length': [5.1, 4.9, 4.7],
    'sepal_width': [3.5, 3.0, 3.2],
    'species': ['setosa', 'setosa', 'setosa']
}

df1 = pd.DataFrame(data_dict)

# 方式二：列表嵌套 (每行作为一个列表)
data_list = [[5.1, 3.5, 'setosa'], [4.9, 3.0, 'setosa']]
df2 = pd.DataFrame(data_list, columns=['sepal_length', 'sepal_width', 'species'])
```
---

# 2. 选择与筛选数据

这是数据分析中最频繁的动作，请务必熟练掌握。

## 2.1 选择列

- 按列索引调用
    ```python
    # 单列 → 返回 Series (带索引的一维数组)
    sepal = iris['sepal_length']  
    # 快捷方式 (列名无空格时)
    sepal = iris.sepal_length  

    # 多列 → 返回 DataFrame (注意双中括号)
    sub_df = iris[['sepal_length', 'species']]
    ```
<details>
  <summary><strong>思考1：为什么提取多列需要用双中括号？最里层中括号代表什么意思？</strong> (点击展开参考答案) </summary>

**1. 外层中括号 (`df[...]`) ** 是 Pandas 的**列索引运算符**，它的功能是“从 DataFrame 中取出指定的列”。

**2. 最里层的中括号 `[ ]`** 代表 **Python 原生的列表 (List) **。

- **为什么必须用双层？**
  因为 Pandas 的 `df[ ]` 语法规定：**括号内只能传入一个参数**。
  - 如果你想取**单列**，传入一个**字符串** (`df['列名']`) ，返回 `Series`。
  - 如果你想取**多列**，你必须把多个列名**打包成一个整体**传入。而 Python 中最直接的“打包容器”就是**列表**。
  - 所以 `df[ ['列1', '列2'] ]` 的逻辑是：**外层的索引器**接收到了**一个列表对象**，Pandas 识别出这个列表里装了多个字符串，于是解析并返回多列组成的 `DataFrame`。

- **如果少写一层会怎样？**
  如果你写成 `df['列1', '列2']`，Python 解释器会把 `'列1', '列2'` 解释为一个**元组 (Tuple) **。此时 Pandas 会去 DataFrame 中查找名为 `('列1', '列2')` 的**单一列名** (这在多层索引 MultiIndex 中才用得到) ，绝大多数情况下会直接报错 `KeyError`。

## 代码验证 (你可以在 Jupyter 中试一下) 
```python
import pandas as pd
df = pd.DataFrame({'A': [1,2], 'B': [3,4], 'C': [5,6]})

# 正确：传入一个列表 (双括号) 
print(df[['A', 'B']])  

# 错误：传入一个元组 (单括号内加逗号) ，会报错 KeyError
# print(df['A', 'B'])  
```
</details> 

- 按列位置调用
    ```python
    col_0 = iris.iloc[:, 0]          # 冒号表示所有行，0 表示第一列

    # 选择前 3 列,类似于列表切片 (位置 0, 1, 2) → sepal_length, sepal_width, petal_length
    first_3_cols = iris.iloc[:, 0:3]
    ```

## 2.2 选择行 (loc 按标签 / iloc 按位置)

```python
# 按位置 (前 3 行)
row_first3 = iris.iloc[0:3]  

# 按索引名 (假设索引是 0,1,2...，此处同 iloc)
row_5 = iris.loc[4]  
```

## 2.3 条件筛选 (布尔索引)

通过逻辑判断快速过滤出符合条件的子集：

```python
# 筛选出所有 "setosa" 品种
setosa_df = iris[iris['species'] == 'setosa']

# 筛选出 "sepal_length > 6.0" 且品种为 "virginica" 的数据
# 注意：多个条件用 & (且) 或 | (或)，每个条件必须加括号
virginica_large = iris[(iris['sepal_length'] > 6.0) & (iris['species'] == 'virginica')]

# 筛选出花瓣长度在 [3.0, 5.0] 区间内的数据
between_df = iris[iris['petal_length'].between(3.0, 5.0)]
```

## 2.4 排序 (sort_values)

查看最大值、最小值或排名时，排序是极为常用的操作。

`sort_values` 的核心参数是 `ascending：ascending=True` (默认) 表示升序 (从小到大)，`ascending=False` 表示降序 (从大到小)。当按多列排序时，`ascending` 可以传入一个与列名顺序对应的布尔列表，分别控制每列的升降序。

```python
# 按花萼长度升序排列
sorted_asc = iris.sort_values('sepal_length')

# 按花萼长度降序，若相同则按花瓣宽度升序
sorted_mix = iris.sort_values(['sepal_length', 'petal_width'], ascending=[False, True])
```

---

# 3. 数据清洗

现实数据往往有异常值，或数据本身缺失、类型错误，清洗是绕不开的步骤。

## 3.1 缺失值检测与处理

```python
# 检测缺失值 (Iris 数据很干净，此处仅为演示方法)
print(iris.isnull().sum())  # 统计每列缺失个数

# 方法一：粗暴删除含 NA 的行 (慎用)
df_dropped = iris.dropna()

# 方法二：填充缺失值 (更推荐)
# 用该列均值填充 (数值列)
# iris['sepal_length'].fillna(iris['sepal_length'].mean(), inplace=True)
# 或用特定值填充 (字符串列)
# iris['species'].fillna('unknown', inplace=True)
```

## 3.2 类型转换 (astype)

确保数据类型符合预期，避免计算时报错：

- 将物种列转为 category 类型：就是将 species 列从普通的字符串列 (object 类型)，转换为分类类型。
    ```python 
    iris['species'] = iris['species'].astype('category')
    ```
    > 如果查看 `iris['species'].cat.categories`，会看到底层存储的三个类别值：`['setosa', 'versicolor', 'virginica']`。
- 将浮点型转为整型 (注意精度丢失)：
    ```python
    iris['sepal_length'] = iris['sepal_length'].astype('int')
    ```

## 3.3 列名重命名 (rename)

列名过长、含空格或大小写不规范时，需要修改：

```python
# 将列名改为更简洁的英文 (或中文)
iris.rename(columns={
    'sepal_length': 'Sepal_Len',
    'sepal_width': 'Sepal_Wid'
}, inplace=True)  # inplace=True 表示直接修改原数据框

# 试验完记得再调整回来
# iris = sns.load_dataset('iris')
```

---

# 4. 常用计算与变换

## 4.1 行列求和、均值、计数

```python
# 对列求均值 (axis=0，默认)
print(iris[['sepal_length', 'sepal_width']].mean())

# 对行求均值 (axis=1)
iris['avg_feature'] = iris.iloc[:, :4].mean(axis=1)

# 计数 (非空值数量)
print(iris.count())
```

## 4.2 对列进行算术运算

Pandas 支持将列视为变量进行直接的算术运算，包括列与列之间的运算，以及列与常数之间的运算。

```python
# 创建新列：花萼长宽比
iris['sepal_ratio'] = iris['Sepal_length'] / iris['sepal_width']

# 所有花瓣长度 + 2
iris['petal_length_plus'] = iris['petal_length'] + 2
```

## 4.3 应用函数 (apply) 与字符串处理

Pandas 内置了大量常用函数 (如求和、均值、排序等)，但实际分析中总会遇到一些个性化需求——比如按某个条件给数据打标签、把两列数据按某种规则组合成一个新值，或者对字符串做拆分提取。这时候 `apply` 函数就派上了用场。

- **apply 是什么？**

    `apply` 是 Pandas 中最灵活的变换工具。它的作用简单说就是：**把函数 (包括你的自定义函数) 应用到 DataFrame 的每一行或每一列上**，然后返回一个新的 Series 或 DataFrame。

    通俗地理解，`apply` 就像一条“数据加工流水线”，你只需要告诉它“加工规则 (函数)”和“加工方向 (按行还是按列)”，它就会自动帮你处理完所有数据。

- **axis 参数：控制加工方向**

    `axis` 是 `apply` 最核心的参数，决定了函数是逐列执行还是逐行执行：

    | axis 取值 | 含义 | 函数接收的参数 |
    |-----------|------|----------------|
    | `axis=0` (默认)  | 按列遍历，对每一列执行函数 | 函数接收一列数据 (Series) |
    | `axis=1` | 按行遍历，对每一行执行函数 | 函数接收一行数据 (Series) |

- **示例**

    1. 对单列应用函数
        ```python
        # 自定义函数：判断花萼长度是否大于 5
        def mark_length(x):
            if x > 5:
                return 'Long'
            else:
                return 'Short'

        # apply 会把 mark_length 应用到 sepal_length 列的每一个值上
        iris['len_tag'] = iris['sepal_length'].apply(mark_length)
        print(iris[['sepal_length', 'len_tag']].head())

        # 也可以直接用 lambda 匿名函数，更简洁
        iris['len_tag'] = iris['sepal_length'].apply(lambda x: 'Long' if x > 5 else 'Short')
        ```
        > lambda 是 Python 中的匿名函数，即“没有名字的临时函数”，专为一次性使用而设计。它的语法非常简洁：`lambda 参数: 表达式`
        > - 左侧 (lambda 参数:)：定义传入的参数
        > - 右侧 (表达式)：对参数执行操作，并自动返回结果
        >
        > 对比普通函数 (def)，lambda 不用写 return，也不需要起函数名，特别适合作为参数传递给 apply 这类“用完即弃”的场景。
        >
        >**练习:** 判断sepal_width列度是否大于 3。

    2. 对整行应用函数

        ```python
        # 自定义函数：接收一行数据，返回花瓣长度 / 花萼长度
        def calc_ratio(row):
            # row 是一行数据 (Series) ，可以通过列名取到每个字段的值
            if row['sepal_length'] != 0:   # 防止除以零
                return row['petal_length'] / row['sepal_length']
            else:
                return 0

        # axis=1 表示按行遍历，每一行都会传入 calc_ratio 函数
        iris['petal_sepal_ratio'] = iris.apply(calc_ratio, axis=1)
        print(iris[['sepal_length', 'petal_length', 'petal_sepal_ratio']].head())
        ```

    3. 结合条件判断，实现复杂的逻辑映射

        ```python
        # 根据花瓣长度给出等级 (多分支逻辑) 
        def grade_petal(x):
            if x < 1.5:
                return 'S'
            elif x < 4.0:
                return 'M'
            else:
                return 'L'

        iris['petal_grade'] = iris['petal_length'].apply(grade_petal)
        print(iris[['petal_length', 'petal_grade']].head())
        # lambda 版本 (嵌套三元运算符) 
        iris['petal_grade'] = iris['petal_length'].apply(
            lambda x: 'S' if x < 1.5 else ('M' if x < 4.0 else 'L')
        )
        ```

## 4.4 字符串处理

Pandas 为字符串类型的列提供了专门的 `.str` 接口，方便进行拆分、提取、替换等操作，无需手动写循环。

```python
# 假设我们有列 'species_info' 格式为 'setosa_1' (品种_编号)
# 先创建一个示例列
iris['species_info'] = iris['species'] + '_' + iris.index.astype(str)

# 使用 .str.split() 拆分为两列
iris[['species_clean', 'id_num']] = iris['species_info'].str.split('_', expand=True)
print(iris[['species_info', 'species_clean', 'id_num']].head())

# 其他常用的 str 方法
iris['species_upper'] = iris['species'].str.upper()          # 转大写
iris['contains_setosa'] = iris['species'].str.contains('setosa')  # 判断是否包含
iris['species_len'] = iris['species'].str.len()              # 计算字符串长度
```

当内置函数无法满足需求时，`apply` 能帮你快速实现个性化的数据处理逻辑。配合 `axis` 参数的灵活切换，无论是逐列还是逐行操作都能轻松应对。而 `.str` 方法则专门用于处理文本数据，与 `apply` 互为补充。掌握这两种方法，你就掌握了数据清晰的诀窍。

---

# 5. 分组聚合

功能与 Excel 数据透视表相似，其基本逻辑如下。
1. 按照某一列 (或多列) 将数据划分为若干组。
2. 对每组数据独立执行某种运算 (如求和、均值、计数，或自定义函数)。
3. 将各组运算结果合并为一个新的数据结构。

调用模式为:`df.groupby('分组列')['待计算列'].聚合方法()`

```python
# 按 species 分组，计算各数值列的均值 (默认)
grouped_mean = iris.groupby('species')[['sepal_length', 'petal_length']].mean()
print(grouped_mean)
```

当需要**对不同列执行不同的聚合运算**时，agg() 方法提供了更大的灵活性。调用模式为：
`df.groupby('分组列').agg({'列名1': ['统计量1', '统计量2'], '列名2': '统计量3'})`

```python
grouped_agg = iris.groupby('species').agg({
    'sepal_length': ['mean', 'std', 'min'],
    'petal_length': ['max', 'count']
})
print(grouped_agg)

```

如果需要计算每个品种内部的花萼长度标准化值 (z-score),即：(每个值 - 该品种均值) / 该品种标准差,则需要 `.transform()`。这种方法会将计算结果广播回原始数据的每一行，保持行数不变。这在需要为每一行添加分组统计特征时非常有用。

```python
iris['sepal_length_z'] = iris.groupby('species')['sepal_length'].transform(
    lambda x: (x - x.mean()) / x.std()
)
print(iris[['species', 'sepal_length', 'sepal_length_z']].head())
```

---

# 6. 合并与拼接 (DataFrame 间的运算)

## 6.1 左右合并

`merge` 是 Pandas 中最常用的表连接方法，其核心逻辑为: **根据一个或多个共同的列，将两张表横向拼接到一起**。

**核心调用模式：** `pd.merge(left, right, on='键列', how='连接方式')`,或等价于：`left.merge(right, on='键列', how='连接方式')`

**参数解析**
| 参数 | 含义 | 常用取值 |
|------|------|----------|
| `left` | 左侧 DataFrame | — |
| `right` | 右侧 DataFrame | — |
| `on` | 用于连接的列名 (左右表必须同名)  | 列名字符串，或多个列名组成的列表 |
| `left_on` / `right_on` | 左右表连接列名不同时分别指定 | `left_on='左表列名', right_on='右表列名'` |
| `how` | 连接方式，决定哪些行保留 | `'inner'` (默认) 、`'left'`、`'right'`、`'outer'` |
> **注意**：如果左右表的键列**列名不同**，需要分别指定 `left_on` 和 `right_on`。例如左表叫 `species`，右表叫 `species_name`，则写法为：`left.merge(right, left_on='species', right_on='species_name', how='left')`。

**连接方式**
| how 取值 | 结果 |
|----------|------|
| `'inner'` (默认)  | 只保留两张表**键列都匹配**的行 (交集)  |
| `'left'` | 保留**左表所有行**，右表无匹配则填充 `NaN` |
| `'right'` | 保留**右表所有行**，左表无匹配则填充 `NaN` |
| `'outer'` | 保留**两张表所有行**，无匹配则填充 `NaN` (并集)  |

```python
# 构建一个汇总表：每个品种的平均花萼长度
avg_by_species = iris.groupby('species')['sepal_length'].mean().reset_index()
avg_by_species.columns = ['species', 'avg_sepal_len']

# 左连接，将平均长度匹配回原表的每一行
iris_merged = iris.merge(avg_by_species, on='species', how='left')
print(iris_merged[['species', 'sepal_length', 'avg_sepal_len']].head())
```

## 6.2 上下拼接 

```python
# 拆分两个子集再拼回去 (模拟追加数据)
part1 = iris.iloc[:50]
part2 = iris.iloc[50:100]
concat_df = pd.concat([part1, part2], axis=0)  # axis=0 纵向堆叠
```

## 6.3 逐行读取 (for 循环与 iterrows)

虽然强烈建议优先使用向量化操作，但某些特殊逻辑下循环仍有价值：

```python
# 逐行遍历 (注意：大数据集下速度较慢，慎用)
for idx, row in iris.iterrows():
    if row['sepal_length'] > 7.0:
        print(f"第 {idx} 行是大型鸢尾花，品种: {row['species']}")

# 提示 (若非必要，请用 apply 或向量化替代循环，速度更快)
```

---

# 7. 数据导出

## 7.1 导出为列表格式

```python
# 将某列转为 Python 列表 (便于与其他库交互)
sepal_list = iris['sepal_length'].tolist()
print(sepal_list[:5])  # 输出前 5 个
```

## 7.2 导出到文件

```python
# 导出为 CSV (最通用，index=False 避免多出索引列)
iris.to_csv('iris_final.csv', index=False)

# 导出为 TSV (通过sep指定分隔符)
iris.to_csv('iris_final.tsv', sep='\t', index=False)

# 导出为 Excel (需 openpyxl 库)
# iris.to_excel('iris_final.xlsx', index=False)
```

---

以上便是 Pandas 数据框最核心、最常用的功能。从数据探查、条件筛选，到清洗转换，再到分组聚合与合并导出，这套组合拳打下来，足以让你轻松应对绝大多数数据分析场景。记住，遇到报错别慌张，多查文档 `print(help(pd.DataFrame))` 或利用搜索引擎拆解错误信息，是成长最快的方式。

本博客的初衷是与各位读者共同探讨技术、共同进步，如果你在实战中遇到卡点，或有更好的技巧想分享，亦或发现教程有待优化之处，都非常欢迎你发邮件至 **aurorahiker@163.com** 与我交流。你的每一次提问和建议，都会让这份教程变得更加实用和鲜活。期待与你一起，把数据分析这条路走得既扎实又有趣！ 