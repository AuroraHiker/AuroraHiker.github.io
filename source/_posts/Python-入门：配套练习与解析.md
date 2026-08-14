---
title: Python 入门：配套练习与解析
tags:
  - Python 入门
series:
  - Python 入门
categories:
  - Python 入门
author: AuroraHiker
---

# 0. 前言

这套练习题面向已经学完 Python 基础语法、准备进入数据分析入门阶段的读者。建议先阅读前置教程 [Python 入门：零基础编程速通指南]({% post_link Python-入门：零基础编程速通指南 %})，再开始本篇练习。

如果你在练习过程中频繁遇到类型相关报错，例如 `TypeError`、`unsupported operand type(s)`，或者经常把字符串和数字混在一起使用，可以阅读 [Python 入门：变量类型专题]({% post_link Python-入门：变量类型专题 %})，那篇文章系统梳理了常用变量类型、类型转换和常见报错原因，适合作为本篇练习的补充参考。

需要说明的是，这篇练习的重点不在“刷题”本身，而是为后续学习数据分析打底。实际做数据分析时，你会频繁遇到这些基础操作：判断数据是否合格、对一列数据进行批量计算、从文本中拆分字段、将字符串转换为数值、筛选有效数据、把重复逻辑封装成函数。通过这组练习，你不仅能巩固 Python 语法熟练度，也能提前培养数据分析所需要的基本思维和编码习惯。

本套练习题的设计遵循以下四个原则：

1. **强化语法记忆**：让 `if`、`for`、列表、类型转换和函数变成顺手动作。  
2. **训练编程逻辑**：逐步建立读题、拆解问题、设计步骤、写出代码的思维方式。
3. **定位薄弱环节**：不要只看题目做对还是做错，更要观察自己究竟卡在了哪里：是没读懂题意、变量类型混乱、分支条件没想全、循环边界写错，还是不知道该如何组织数据。找到具体问题，才能在后续学习中有意识地查漏补缺。
4. **训练自学能力**：少数题会出现教程里没有详细展开但很常用的写法，例如 `strip()`、`split()`、`replace()`。查资料、看示例、再回到代码中验证，是程序开发中非常常见的学习方式。

做题时建议先按下面五步拆解：

1. **输入是什么**：输入的是一个数、一行文本，还是一组数据？数据来自键盘输入、已有变量，还是文件？它目前是什么格式？
2. **处理过程是什么**：需要判断、转换、计算、筛选，还是重复执行？把大问题拆成几步小操作。
3. **输出是什么**：最终要得到一个结果、一组结果，还是新的数据？结果应该通过 print() 显示，保存到变量，还是写入文件？
4. **边界情况是什么**：如果遇到 0、负数、空字符串、超范围、缺失值、非法字符，程序应该怎么处理？
5. **报错原因是什么**：如果程序报错，先不要急着重写代码。先看报错信息提示的错误类型和具体内容，再定位是哪一行代码触发了报错。接着回头检查：到报错这一行为止，解题思路是否正确？相关变量是否已经正确赋值？调用变量时，变量类型、取值范围和边界情况是否符合预期？所使用的方法或函数语法是否写对？

---

# 1. 条件判断

条件判断是数据分析中“筛选”的基础。Pandas 里的布尔筛选，本质上也是先判断每一行是否满足条件。这里先从单个数据开始练。

## 1.1 判断成绩是否合格

输入一个百分制成绩，判断是否及格。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 先用 `float()` 把输入转成数字。
2. 先判断成绩是否超出 `0-100`。
3. 合法后再判断是否大于等于 `60`。

```python
score = float(input("请输入成绩："))

if score < 0 or score > 100:
    print("成绩不合法")
else:
    if score >= 60:
        print("成绩合格")
    else:
        print("成绩不合格")
```

**易错分析**

- 直接判断 `score >= 60` 会漏掉对输入合法性的校验，真实数据中录入错误并不少见。数据分析的常见思路是：先检查数据范围，再进行分类或统计。

</details>

## 1.2 判断一个值是否为缺失值

输入一个文本值，判断它是否表示缺失数据。这里约定空字符串、`NA`、`na`、`None`、`none` 都表示缺失。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 用 `strip()` 去掉输入前后的空格。
2. 用多个 `or` 判断它是否等于常见缺失标记。
3. 根据结果输出“缺失”或“非缺失”。

```python
value = input("请输入一个值：").strip()

if value == "" or value == "NA" or value == "na" or value == "None" or value == "none":
    print("这是缺失值")
else:
    print("这不是缺失值")
```

**易错分析**

- 漏掉 `strip()`。真实数据中，用户输入或从文件中读取的文本常带有前后空格，不处理会导致本应判为缺失的值被漏判。

</details>

## 1.3 判断鸢尾花花萼长度是否可疑

鸢尾花数据集常用于数据分析入门。输入一个花萼长度 `sepal_length`，如果小于 `4.0` 或大于 `8.0`，输出“疑似异常值”；否则输出“正常范围”。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 花萼长度是小数，所以使用 `float()` 转换数据类型。
2. 先判断是否小于等于 `0`。
3. 再判断是否落在可疑范围外。
4. 剩下的情况就是正常范围。

```python
sepal_length = float(input("请输入花萼长度："))

if sepal_length <= 0:
    print("不合法")
elif sepal_length < 4.0 or sepal_length > 8.0:
    print("疑似异常值")
else:
    print("正常范围")
```

**易错分析**

- 区间判断时混淆逻辑运算符：or 表示“或”，用于“两端之外”的场景；and 表示“且”，用于“介于两者之间”的场景，此处应使用 or。
- 合法性校验容易被忽略，花萼长度作为物理测量值，不可能小于或等于 0，需提前拦截。
- `input()` 返回的一定是字符串，必须先转换为数值类型。

</details>

## 1.4 根据 BMI 做简单分组

输入身高和体重，计算 BMI，并按下面规则输出分组：

| BMI 范围 | 分组 |
| --- | --- |
| `< 18.5` | 偏低 |
| `18.5-23.9` | 正常 |
| `24.0-27.9` | 偏高 |
| `>= 28.0` | 肥胖 |

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 身高和体重均可能为小数，用 `float()` 进行类型转换。
2. 身高或体重小于等于 `0` 时不计算，并给出提示。
3. BMI 公式是 `体重 / 身高²`。
4. 从小到大或从大到小的顺序，依次判断 BMI 区间，输出对应的分组结果。

```python
height = float(input("请输入身高，单位为米："))
weight = float(input("请输入体重，单位为千克："))

if height <= 0 or weight <= 0:
    print("身高和体重必须大于 0")
else:
    bmi = weight / (height ** 2)

    if bmi < 18.5:
        group = "偏低"
    elif bmi < 24.0:
        group = "正常"
    elif bmi < 28.0:
        group = "偏高"
    else:
        group = "肥胖"

    print(f"BMI = {bmi:.2f}，分组为：{group}")
```

**易错分析**

- 分支顺序写反 (例如 `bmi < 28.0` 被写在最前面) 会导致后面的区间被提前匹配，无法正确执行；这类区间判断建议统一按从小到大或从大到小的方向书写，保持逻辑一致。
- 合法性校验不可忽略，身高和体重必须为正数，否则应提前拦截并给出提示。
- `input()`返回的一定是字符串，必须先转换为数值类型。

</details>

---

# 2. 循环

循环是批量处理数据的第一步。数据分析不会局限于一个成绩或一个长度，而是分析一列、一批、一整张表。

## 2.1 计算 7 天步数总和

连续输入 7 天的步数，计算这一周的总步数。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 准备一个变量 `total` 保存总和。
2. 用 `for` 循环重复输入 7 次。
3. 每输入一天，就把步数加到 `total` 上。

```python
total = 0

for day in range(1, 8):
    steps = int(input(f"请输入第 {day} 天的步数："))
    total = total + steps

print(f"7 天总步数为：{total}")
```

**易错分析**

- 容易把 `total = 0` 写在循环内部，导致每轮循环都重新清零，累加失效。

    **提示** (以下两种方案二选一) 

    - 累加、计数、求和类操作遵循“循环外初始化 → 循环内更新 → 循环后输出”的固定结构，写代码时可以先确定循环外需要准备哪些变量，再填充循环体。
    - 在循环体内每次更新变量时，问自己一句“这个变量是每轮都要更新，还是累积到最后一并输出？”——想清楚再写。

- `input()` 返回的是字符串，直接与其他数值相加会报错，必须先用 `int()` 或 `float()` 转换为数值类型。

</details>

## 2.2 计算 5 次测量值的平均数

连续输入 5 次测量值，计算平均数。测量值可以是小数。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 输入值可能为小数，用 `float()`进行类型转换。
2. 初始化 `total`，在循环中依次读入 5 次测量值并累加。
3. 循环结束后，用总和除以 5 得到平均值，并保留两位小数输出。

```python
total = 0

for i in range(1, 6):
    value = float(input(f"请输入第 {i} 次测量值："))
    total = total + value

average = total / 5
print(f"平均值为：{average:.2f}")
```

**易错分析**

- `input()` 返回的一定是字符串，必须先通过 `int()` 或 `float()` 转换为数值类型。

</details>

## 2.3 统计高于阈值的数据个数

给定一组花萼长度，统计其中大于 `5.0` 的数据有多少个。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 用列表保存一组花萼长度。
2. 准备计数变量 `count`。
3. 遍历每一个长度，满足条件就加 1。
4. 输出统计结果。

```python
lengths = [5.1, 4.9, 4.7, 5.4, 6.3, 5.0, 7.1]
count = 0

for length in lengths:
    if length > 5.0:
        count = count + 1

print(f"大于 5.0 的数据有 {count} 个")
```

或使用列表推导式：

```python
lengths = [5.1, 4.9, 4.7, 5.4, 6.3, 5.0, 7.1]

print(f"大于 5.0 的数据有 {len([ x for x in lengths if x > 5 ])} 个")
```

**易错分析**

- 把 `>` 写成 `>=`，把刚好等于 `5.0` 的数据也统计进来。
- 处理阈值类问题时，先明确边界值是否包含在内，这是数据筛选中的一个关键决策点。

</details>

## 2.4 找出一组数据的最大值和最小值

给定一组数据，找出最大值和最小值。

**要求：**不能使用 `max()` 和 `min()`，用循环完成。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 先假设第一个数据同时是最大值和最小值。
2. 遍历列表中的每个数据。
3. 若当前值大于最大值则更新最大值，若小于最小值则更新最小值。

```python
datas = [210, 180, 260, 240, 300, -190, 275]

max_value = datas[0]
min_value = datas[0]

for value in datas:
    if value > max_value:
        max_value = value

    if value < min_value:
        min_value = value

print(f"最大值：{max_value}")
print(f"最小值：{min_value}")
```

**易错分析**

- 把最大值初始值写成 `0`，在数据全是负数时会出错。 `0` 不是万能初始值。
- 数据分析中，初始化值、数据清洗方式、异常处理策略等都应当根据具体问题具体分析，避免套用固定模板。例如，同样是求最大值，价格数据从 `0` 开始合理，但温度数据可能从负数开始，适用场景不同。
- 培养一种习惯：拿到数据后先大致浏览一下分布 (正负、范围、缺失情况) ，再决定如何处理，而不是凭经验直接写死某个值或方法。

</details>

---

# 3. 列表

列表可以理解成“没有表头的一列数据”。在正式学习 DataFrame 之前，先把列表的存储、切片、筛选、合并和排序练熟。

## 3.1 把一行分数转换成列表

输入一行由空格分隔的百分制分数，例如 `88 92 75 64 90`，把它们转换成数字列表，并输出人数、总分和平均分。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 用 `split()` 把一行文本拆成多个小字符串。
2. 创建一个空列表 `scores` 用于存放有效分数。
3. 遍历拆分后的每个字符串，转成 `int` 并校验是否在 `0–100` 范围内，合法则加入列表。
4. 基于列表计算人数、总分和平均分，输出结果。
   
```python
text = input("请输入一组分数，用空格分隔：")
items = text.split()

scores = []
for item in items:
    score = int(item)
    if score < 0 or score > 100:
        print(f"跳过不合法的分数：{score}")
    else:
        scores.append(score)

total = sum(scores)
count = len(scores)
average = total / count if count > 0 else 0

print(f"人数：{count}")
print(f"总分：{total}")
print(f"平均分：{average:.2f}")

```

**易错分析**

- `input()` 返回的一定是字符串，必须先转换为 `int` 或 `float`。
- 分数是百分制，输入超出 `0–100` 范围应提前拦截。
- 若所有分数都被跳过，`len(scores)` 为 `0`，直接除零会报错，因此平均分计算前需要判断列表是否为空。

</details>

## 3.2 查看数据的前 3 个和后 3 个

给定一组实验测量值，输出前 3 个和后 3 个数据。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 列表中从开头取数据，用 `values[:3]`。
2. 从结尾取数据，用 `values[-3:]`。
3. 输出结果，帮助自己快速观察数据。

```python
values = [5.1, 4.9, 4.7, 4.6, 5.0, 5.4, 4.6, 5.0]

first_three = values[:3]
last_three = values[-3:]

print(f"前 3 个数据：{first_three}")
print(f"后 3 个数据：{last_three}")
```

**易错分析**

- 切片操作是“左闭右开”，`values[:3]` 取索引 0、1、2 共三个元素，不要写成 `values[:2]` 或 `values[0:2]` (只取两个) ；
- 同理，`values[-3:]` 取最后三个，不要写成 `values[-2:]` (只取两个) 。记住：`[:n]` 取前 `n` 个，`[-n:]` 取后 `n` 个。

</details>

## 3.3 筛选有效成绩

给定一组成绩，其中小于 `0` 或大于 `100` 的值是录入错误。请把合法成绩筛选出来，放到新列表中。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 准备空列表 `valid_scores`，用于存放有效成绩。
2. 遍历每个成绩，合法就 `append()` 到新列表。

```python
raw_scores = [98, -1, 86, 105, 73, 60, 0]
valid_scores = []

for score in raw_scores:
    if score >= 0 and score <= 100:
        valid_scores.append(score)

print(f"有效成绩：{valid_scores}")
```

也可以使用列表推导式实现：

```python
raw_scores = [98, -1, 86, 105, 73, 60, 0]

valid_scores = [score for score in raw_scores if score >= 0 and score <= 100]

print(f"有效成绩：{valid_scores}")
```

**易错分析**

- 不要直接在原列表上删除异常值。边遍历边删除会改变列表长度，导致元素被跳过或索引错位。筛选场景下，更稳妥的做法是建立新列表保存有效数据。

</details>

## 3.4 合并两组实验数据

两个小组分别记录了一组实验数据，请把它们合并成一个列表，并计算合并后的数据量和平均值。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 两个列表可以用 `+` 合并。
2. 合并后再统一计算长度、总和和平均值。

```python
group_a = [5.1, 4.9, 5.0]
group_b = [5.4, 5.2, 4.8, 5.3]

all_values = group_a + group_b
average = sum(all_values) / len(all_values)

print(f"合并后的数据：{all_values}")
print(f"数据量：{len(all_values)}")
print(f"平均值：{average:.2f}")
```

**易错分析**

- 求总体平均值时，不能先将两组数据分别求平均，再将两个平均值做平均。当两组数据量不同时，这种做法会让数据量较小的一组权重被放大，结果与真实总体平均值有偏差。正确的做法是先合并所有原始数据，再统一计算平均值。

</details>

---

# 4. 数据清洗

变量类型的误判是数据分析初学者最容易掉入的陷阱之一。从外部文件读入的数据，其字段类型往往与预期不符：看似数值的列可能被解析为字符串，带有单位或百分号的字段无法直接参与运算，必须先经过规范化处理，才能用于后续分析。只有完成这些基础的类型校验与格式清理，数据才能真正“可用”。

## 4.1 清理带单位的长度数据

从键盘输入一个带单位的长度值 (单位为厘米，例如 5.1 cm) ，将其转换为纯数值后，加上 0.2，并输出处理前后的结果。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 用 `strip()` 去掉前后空格。
2. 用 `replace("cm", "")` 去掉单位。
3. 再次用 strip() 清理可能残留的空格，然后通过 `float()` 转成数字。
4. 执行加法运算，并按要求输出结果。

```python
text = input("请输入长度，例如 5.1 cm：").strip()

clean_text = text.replace("cm", "").strip()
length = float(clean_text)

new_length = length + 0.2
print(f"处理后的长度：{length}")
print(f"加 0.2 后：{new_length:.2f} cm")
```

**易错分析**

- 直接对 "`5.1 cm`" 调用 `float()` 会引发 `ValueError`，因为字符串中包含了非数字字符。
- 类似地，带有百分号 (%) 、千位分隔符 (逗号) 或其他非数值符号的数据，在参与数学运算前都必须先进行格式清洗和类型转换。养成“先清洗，后计算”的习惯，能有效避免许多隐蔽的数据类型错误。

</details>

## 4.2 把一行 CSV 风格文本拆成字段

输入一行学生记录，例如 `张三,85,90`，其中三列分别是姓名、数学成绩、英语成绩。请拆出字段，并计算两门课平均分。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 逗号分隔的数据用 `split(",")` 拆开。
2. 第 1 个字段是姓名，仍然是字符串。
3. 第 2、3 个字段是成绩，要转成 `float`。
4. 再计算平均分。

```python
row = input("请输入一行数据，例如 张三,85,90：")
parts = row.split(",")

# 1. 检查字段数量
if len(parts) != 3:
    print("输入格式错误：请确保包含姓名、数学成绩、英语成绩三个字段，并以逗号分隔。")
else:
    name = parts[0].strip()          # 去除姓名前后多余空格
    
    try:
        math_score = float(parts[1].strip())
        english_score = float(parts[2].strip())
    except ValueError:
        print("成绩必须为有效数字，请检查输入。")
    else:
        # 2. 检查成绩范围
        if not (0 <= math_score <= 100) or not (0 <= english_score <= 100):
            print("成绩应介于 0 到 100 之间，请重新输入。")
        else:
            average = (math_score + english_score) / 2
            print(f"{name} 的平均分是：{average:.2f}")
```

**易错分析**

- **直接拼接字符串**：拆分后直接计算 `parts[1] + parts[2]`，得到的是字符串拼接 (如 `"85"+"90"` → `"8590"`) ，而非数值相加。必须先转换为数值类型。
- **忽略异常**：输入可能包含非数字字符 (如 `"八十五"` 或 `"85分"`) ，直接 `float()` 会抛出异常。使用 try-except 捕获异常，能有效防止程序崩溃。
- **忽略范围校验**：即使成功转换为数字，成绩仍可能不合理 (如负数或超过 100) 。增加范围检查可尽早发现数据异常，避免后续分析结果失真。
- **字段数量与空格**：输入中可能包含多余空格 (如 `"张三 , 85 , 90"`) ，使用 `strip()` 清理可提高容错性。养成“先清洗，再转换，后校验”的习惯，能大大减少数据类错误。

</details>

## 4.3 百分数字符串转成小数

从键盘输入一个百分数 (例如 `86%`) ，将其转换为对应的小数 (如 `0.86`) 。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 使用 `strip()` 去除字符串首尾空白字符。  
2. 检查输入是否为空，若为空则提示并退出。  
3. 检查字符串是否以 `%` 结尾，若不是则提示格式错误。  
4. 去掉 `%` 符号，并尝试将剩余部分转换为浮点数，捕获可能出现的 `ValueError`。  
5. 检查转换后的数值是否在 0 ~ 100 之间 (百分数的合理范围) ，若超出则发出警告。  
6. 将数值除以 `100`，得到小数比例，并输出。

```python
text = input("请输入百分数，例如 86%：").strip()

# 1. 检查是否为空
if not text:
    print("输入不能为空，请重新运行程序。")
else:
    # 2. 检查是否以 % 结尾
    if not text.endswith("%"):
        print("输入格式错误：百分数应以 '%' 结尾。")
    else:
        # 3. 去掉 % 并转换为数字
        number_text = text[:-1].strip()   # 去掉最后一个字符 (%) 
        try:
            percent_value = float(number_text)
        except ValueError:
            print("百分号前的部分必须为有效数字，请检查输入。")
        else:
            # 4. 检查数值是否在合理范围 (0~100) 
            if not (0 <= percent_value <= 100):
                print("百分数应在 0 到 100 之间，当前值超出范围。")
            else:
                rate = percent_value / 100
                print(f"转换后的比例为：{rate}")
                # 可选：输出带格式的小数
                print(f" (保留两位小数：{rate:.2f}) ")
```

**易错分析**

- **忘记除以 100**：百分数 `86%` 在数学上等于 `86/100 = 0.86`，如果只去掉百分号而不除以 `100`，就会得到 `86`，导致后续比例计算严重偏差。  
- **直接替换百分号**：使用 `replace("%", "")` 虽然可行，但可能意外替换掉百分号以外的字符 (虽然本例中不会) 。更精确的做法是检查结尾并切片，避免误删。  
- **忽略格式校验**：输入可能不含百分号 (如 `86`) 或包含多余字符 (如 `86%abc`) ，未校验就直接转换会引发异常或得到错误结果。  
- **忽略数值范围**：百分比通常约定在 0% ~ 100% 之间，但实际数据中可能出现负值或大于 100 的情况，及时检测有助于发现数据异常。  
- **通用经验**：在处理百分号、单位、逗号等常见符号时，务必先清洗格式、校验合法性，再转换类型，最后进行计算。这一流程适用于绝大多数数据清洗场景。

</details>

---

# 5. 条件判断 + 循环综合

真实的数据处理很少只是单次判断或单次循环，更多时候是需要对一组数据逐条进行判断、清洗、计算，并最终汇总统计结果。将条件分支与循环结构结合使用，是编写稳定、准确的数据处理脚本的核心能力。

## 5.1 计算有效成绩的平均分

从键盘输入一组成绩，成绩之间用空格分隔，例如 `90 85 NA 110 78 -1 66`。其中 `NA` 表示缺失，小于 `0` 或大于 `100` 的成绩是异常值。请只使用有效成绩计算平均分。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 用 `split()` 输入字符串，得到成绩的字符串列表。
2. 遇到 `NA` 直接跳过。
3. 其他内容尝试转成 `float`，若转换失败则跳过并给出提示。
4. 检查转换后的分数在 `0-100` 内才累加。
5. 最后用有效数量计算平均值。

```python
text = input("请输入一组成绩，用空格分隔：").strip()

# 若输入为空，直接结束
if not text:
    print("输入为空，无法计算。")
else:
    items = text.split()
    total = 0
    count = 0

    for item in items:
        # 先处理明确标记的缺失值
        if item == "NA":
            continue

        # 再尝试转换为数值
        try:
            score = float(item)
        except ValueError:
            # 遇到其他非数字字符，可选择跳过或报错，此处按跳过处理
            print(f"警告：忽略非数字项 '{item}'")
            continue

        # 最后进行范围校验 (0～100) 
        if 0 <= score <= 100:
            total += score
            count += 1
        else:
            print(f"警告：忽略异常值 {score} (不在0~100范围内) ")

    if count == 0:
        print("没有有效成绩，无法计算平均分")
    else:
        average = total / count
        print(f"有效成绩数量：{count}")
        print(f"有效成绩平均分：{average:.2f}")
```

**易错分析**

- **类型转换前未过滤缺失值和非数值项：**若先执行 `float(item)`，遇到 `"NA"` 等非数值项会直接抛出 `ValueError` 导致程序崩溃。必须先将缺失值 (或其他非数字标记) 排除，再进行类型转换，这是数据处理中的顺序原则。

- **忽视输入为空**：直接对空字符串执行 `split()` 会得到空列表，后续循环不会执行，但 `count = 0` 时最终会提示“无有效成绩”。不过最好显式检查输入是否为空，提升用户体验。

- **异常值范围误判**：边界条件 `0 <= score <= 100` 包含了 `0` 和 `100`，需根据具体情况确认是否包含端点 (通常成绩为 0～100 是合理的) 。

</details>

## 5.2 统计鸢尾花类别数量

给定一组鸢尾花类别，统计 `setosa`、`versicolor`、`virginica` 三个类别各有多少条记录，同时识别并记录无法归类的未知类别或异常值。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 为每个类别准备一个计数变量。
2. 遍历类别列表。
3. 用 `if-elif-else` 判断当前类别属于哪一种，对应类别的计数加 1。未知类别计入 `unknown_count`。
4. 输出各类别计数结果。

```python
species_list = [
    "setosa", "setosa", "versicolor", "virginica",
    "setosa", "versicolor", "virginica", "virginica "
]

setosa_count = 0
versicolor_count = 0
virginica_count = 0
unknown_count = 0

for species in species_list:
    # 数据清洗：去除首尾空格，统一转换为小写
    cleaned = species.strip().lower()
    
    if cleaned == "setosa":
        setosa_count += 1
    elif cleaned == "versicolor":
        versicolor_count += 1
    elif cleaned == "virginica":
        virginica_count += 1
    else:
        unknown_count += 1

print(f"setosa: {setosa_count}")
print(f"versicolor: {versicolor_count}")
print(f"virginica: {virginica_count}")
print(f"未知类别: {unknown_count}")
```

字典版：

```python
species_list = [
    "setosa", "setosa", "versicolor", "virginica",
    "setosa", "versicolor", "virginica", "virginica "
]

# 用字典存储计数，key 为类别，value 为数量
counts = {}

for species in species_list:
    cleaned = species.strip().lower()
    counts[cleaned] = counts.get(cleaned, 0) + 1

# 按固定顺序输出已知类别，同时显示所有未知类别
known_species = ["setosa", "versicolor", "virginica"]
for name in known_species:
    print(f"{name}: {counts.get(name, 0)}")

# 输出除已知类别外的其他类别 (未知/异常) 
for name, cnt in counts.items():
    if name not in known_species:
        print(f"未知类别 '{name}': {cnt}")
```

**易错分析**

- **多个独立 `if` 代替 `if-elif-else`**：若用多个独立的 `if`，每条记录可能被重复统计 (例如先进入 `if` 再进入后续 `if`，虽然本例因值互斥不会发生，但逻辑上属于不良习惯) 。`if-elif-else` 能保证一个样本只进入一个分支，互斥分类必须使用。
- **忽略数据清洗**：实际数据中可能存在 `"Setosa"`、`"setosa "`、`"SETOSA"` 等变体，若不做清洗直接比较，会被误判为未知类别。统一使用 `.strip().lower()` 可有效提升容错性。
- **尽可能不要硬编码多个变量**：当类别数量较多时 (如 10 种以上) ，使用独立变量会导致代码臃肿且难以扩展。推荐使用字典 (见字典版) ，使统计逻辑更简洁、可维护性更强。


</details>

## 5.3 统计高于平均值的天数

给定 7 天销售额，统计有多少天高于平均值。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 数据清洗：遍历原始数据，只保留非负数值 (`>= 0`) ，存入有效列表。若存在负数，输出警告信息并排除。
2. 第一次遍历：对有效数据求和，计算平均值。若有效数据为空，则提示无法计算。
3. 第二次遍历：统计有效数据中高于平均值的天数。
4. 输出平均值和高于平均值的天数。

```python
datas = [210, 180, 260, 240, 300, 190, 275]

# 步骤1：过滤非法数据 (负数视为异常) 
valid_datas = []
for value in datas:
    if value < 0:
        print(f"警告：忽略非法销售额 {value} (负数) ")
    else:
        valid_datas.append(value)

# 步骤2：检查是否有有效数据
if not valid_datas:
    print("没有有效数据，无法计算")
else:
    # 步骤3：求和并计算平均值
    total = 0
    for value in valid_datas:
        total += value

    average = total / len(valid_datas)

    # 步骤4：统计高于平均值的天数
    count = 0
    for value in valid_datas:
        if value > average:
            count += 1

    print(f"有效数据条数：{len(valid_datas)}")
    print(f"平均销售额：{average:.2f}")
    print(f"高于平均值的天数：{count}")
```

列表推导式版：


```python
datas = [210, 180, 260, 240, 300, 190, 275]

# 步骤1：过滤非法数据 (负数视为异常) 
valid_datas = [x for x in datas if x >= 0]

# 步骤2：检查是否有有效数据
if not valid_datas:
    print("没有有效数据，无法计算")
else:
    # 步骤3：求和并计算平均值
    total = sum(valid_datas)          # 使用内置函数简化求和
    average = total / len(valid_datas)

    # 步骤4：统计高于平均值的天数 (列表推导式直接生成符合条件的列表) 
    above_avg = [x for x in valid_datas if x > average]

    print(f"有效数据条数：{len(valid_datas)}")
    print(f"平均销售额：{average:.2f}")
    print(f"高于平均值的天数：{len(above_avg)}")
```

**易错分析**

- **忽略数据合法性**：销售额为负数在是不合理的 (除非有退货退款等特殊情况，但此类业务需单独处理) 。若不过滤负数，平均值会被拉低，导致统计结果失真。清洗数据时，必须先剔除或修正异常值。
- **平均值未求出就提前判断**：初学者容易试图在一次循环中同时完成求和与比较，但此时平均值尚未可知，无法判断“高于平均值”。必须分两轮处理，先建立基准，再执行筛选。
- **忽视空数据**：若过滤后有效列表为空，`len(valid_datas) = 0`，`total / len(valid_datas)` 会触发 `ZeroDivisionError`，务必提前检查。
- **边界判断**：高于平均值使用 `>`，若需求是“不低于平均值”则应使用 `>=`。根据实际要求选择正确的比较符号。

</details>

## 5.4 判断多条打卡记录是否迟到

给定一组打卡时间，规定 `08:30` 之后为迟到。统计迟到人数。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 遍历每条打卡记录，检查是否为空、格式是否为 `HH:MM`、小时和分钟是否在合理范围内 (0~23 和 0~59) 。不合法的记录输出警告并跳过。
2. 将合法的时间字符串拆分为小时和分钟，换算为当天的总分钟数 (`小时 * 60 + 分钟`) ，便于数值比较。
3. 将打卡时间与标准时间 `08:30` (即 `8*60+30 = 510` 分钟) 比较，若大于标准时间则计为迟到。
5. 输出迟到总人数。

```python
times = ["08:20", "08:35", "09:05", "08:30", "08:55"]

# 标准时间：08:30 (分钟数) 
STANDARD = 8 * 60 + 30
late_count = 0
valid_count = 0

for time_text in times:
    # 1. 去除首尾空格，检查是否为空
    time_text = time_text.strip()
    if not time_text:
        print("警告：忽略空记录")
        continue

    # 2. 检查格式是否包含 ":"
    if ":" not in time_text:
        print(f"警告：忽略格式错误的时间 '{time_text}' (缺少冒号) ")
        continue

    # 3. 拆分并检查是否恰好两个部分
    parts = time_text.split(":")
    if len(parts) != 2:
        print(f"警告：忽略格式错误的时间 '{time_text}' (字段数量不对) ")
        continue

    # 4. 转换小时和分钟，捕获非数字异常
    try:
        hour = int(parts[0])
        minute = int(parts[1])
    except ValueError:
        print(f"警告：忽略格式错误的时间 '{time_text}' (包含非数字字符) ")
        continue

    # 5. 检查时间范围是否合法
    if not (0 <= hour <= 23):
        print(f"警告：忽略非法时间 '{time_text}' (小时应在 0~23 之间) ")
        continue
    if not (0 <= minute <= 59):
        print(f"警告：忽略非法时间 '{time_text}' (分钟应在 0~59 之间) ")
        continue

    # 6. 转换为分钟数并与标准时间比较
    total_minutes = hour * 60 + minute
    valid_count += 1

    if total_minutes > STANDARD:
        late_count += 1
        print(f"  {time_text} → 迟到")
    else:
        print(f"  {time_text} → 正常")

print(f"有效打卡记录数：{valid_count}")
print(f"迟到数：{late_count}")
```

**易错分析**

- **直接比较字符串：**初学者容易写成 `"09:05" > "08:30"`，认为这种写法等同于时间比较。字符串比较是按字典序逐字符进行的，在统一 `HH:MM` 且小时补零时可能碰巧有效，但遇到未补零时间 (如 `"9:05"`)、多余空格或跨天记录时就不可靠。必须转换为同一单位 (如分钟数) 再进行数值比较。
- **忽略输入格式校验**：如果数据来自用户输入或文件，无法保证每条都是合法的 HH:MM。缺少冒号、包含非数字字符、小时/分钟超限等情况都可能出现，必须逐一校验。
- **边界判断错误**：规定 `08:30` 之后为迟到，则 `08:30` 不算迟到，应使用 `>` 而不是 `>=`。务必根据业务规则选择正确的比较符号。

</details>

## 5.5 10 个数字的排序

从键盘输入 10 个数字 (用空格分隔) ，按从小到大顺序排序后输出。

**要求：禁止使用 Python 内置排序函数 (如 `sorted()` 或 `list.sort()`) ，必须手动实现排序算法。**

<details>
<summary><b>参考答案</b></summary>

**排序算法简介**

排序是编程中最基础、最重要的问题之一。在不能使用内置函数的情况下，我们需要自己实现排序逻辑。常见的排序算法包括：

- **冒泡排序 (Bubble Sort)** ：重复遍历列表，每次比较相邻两个元素，若顺序错误则交换，直到整个序列有序。其名称源于越大的元素会像气泡一样“冒”到序列末端。
- **选择排序 (Selection Sort)** ：每次从未排序部分选出最大或最小元素，放到已排序部分的开头或末尾。
- **插入排序 (Insertion Sort)** ：将未排序元素逐个插入到已排序部分的正确位置。

本任务以**冒泡排序**为例，因其逻辑直观、易于理解，是初学者掌握排序思想的最佳起点。

**冒泡排序的核心思想** (以升序为例) 

1. 比较相邻的两个元素。如果左边的比右边的大，就交换它们的位置。
2. 对每一对相邻元素重复上述比较，一轮结束后，最大的元素就被“冒泡”到了列表末尾。
3. 忽略已排好序的末尾元素，对剩余部分重复上述过程。
4. 每完成一轮，未排序部分就减少一个元素。共需执行 **n-1 轮** (`n` 为列表长度) 。

> 示例演示：`[5, 3, 8, 1]` → 第一轮后 `[3, 5, 1, 8]` → 第二轮后 `[3, 1, 5, 8]` → 第三轮后 `[1, 3, 5, 8]`

---

**拆解**

1. **输入与清洗**：读取一行输入，用 `split()` 按空格拆分为字符串列表。
2. **数据校验**：
   - 检查元素个数是否为 10，若非 10 则提示并退出。
   - 逐个尝试将字符串转换为浮点数，捕获异常并提示非法输入。
3. **冒泡排序** (双重循环) ：
   - 外层循环 `for i in range(n-1)`：控制排序轮数 (共 `n-1` 轮，每次确定一个最大数) 。
   - 内层循环 `for j in range(n-1-i)`：比较相邻元素 `numbers[j]` 和 `numbers[j+1]`，若前者大于后者则交换位置。
   - 注意内层循环的上界是 `n-1-i`，因为每轮结束后列表末尾 `i` 个元素已经有序，无需再比较。
4. **输出结果**：打印排序后的列表。

---

**版本一：冒泡排序**

```python
text = input("请输入 10 个数字，用空格分隔：")
items = text.split()

# 步骤1：校验输入数量
if len(items) != 10:
    print("错误：请恰好输入 10 个数字")
else:
    # 步骤2：转换为数字，并校验合法性
    numbers = []
    valid = True
    for item in items:
        try:
            numbers.append(float(item))
        except ValueError:
            print(f"错误：'{item}' 不是有效数字")
            valid = False
            break

    if valid:
        n = len(numbers)

        # 步骤3：冒泡排序
        for i in range(n - 1):                    # 外层循环：排序轮数
            for j in range(n - 1 - i):            # 内层循环：比较相邻元素
                if numbers[j] > numbers[j + 1]:
                    # 交换两个元素
                    temp = numbers[j]
                    numbers[j] = numbers[j + 1]
                    numbers[j + 1] = temp

        print(f"排序结果：{numbers}")
```

**版本二：选择排序**

作为对比，这里展示选择排序的核心实现，帮助读者理解不同算法的差异。以下片段应放在 `numbers` 和 `n` 已经准备好之后运行：：

```python
# 选择排序：每轮选出未排序部分的最小值，放到最前面
# 跳过数据检查和清洗部分
for i in range(n - 1):
    min_idx = i
    for j in range(i + 1, n):
        if numbers[j] < numbers[min_idx]:
            min_idx = j
    if min_idx != i:
        numbers[i], numbers[min_idx] = numbers[min_idx], numbers[i]
```

**易错分析**

- **内层循环下标越界**：内层循环写成 `range(n)` 时，当 `j = n-1` 时访问 `numbers[j+1]` 会越界。必须将上界设为 `n-1-i`，因为每轮末尾 `i` 个元素已有序，不需要再比较。**只要代码中出现 `j+1`，循环上界就要留出一个位置。**
- **输入数量不为 10**：若用户输入 9 个或 11 个数字，程序仍尝试排序可能导致逻辑错误。务必提前校验并给出明确提示。
- **未处理非数字输入**：用户可能输入字母或特殊符号，直接 `float()` 会抛出异常。应使用 `try-except` 捕获并引导用户修正。
- **交换变量时错误操作**：初学者常写 `a = b; b = a`，结果两个变量都变成了原 b 的值。正确做法是使用临时变量 `temp = a; a = b; b = temp`，或直接利用 Python 的并行赋值 `a, b = b, a`。
- **外层循环轮数理解错误**：n 个元素排序最多需要 `n-1` 轮，写成 `range(n)` 多跑一轮虽然不报错，但会造成不必要的计算，初学者应清楚其含义。

---

**延伸思考**

- **不同排序算法的适用场景**：冒泡排序时间复杂度为 O(n²)，适合数据量较小 (如本题 10 个) 的场景。数据量大时应考虑快速排序、归并排序等更高效的算法。
- **泛化能力**：本例虽为固定 10 个数字，但代码稍加修改即可支持任意数量，通用性更强。

</details>

---

# 6. 函数

函数的意义不是为了“显得高级”，而是把重复的清洗、判断和统计逻辑打包。后续做数据分析时，函数经常用于清洗列、计算指标和复用流程。

## 6.1 将冒泡算法写成排序函数

将冒泡排序算法封装为一个可复用的函数 `bubble_sort(arr, reverse=False)`，接收一个数字列表，对其按升序 (默认) 或降序排序，并返回排序后的列表。要求：
- 不能使用 Python 内置排序函数 (如 `sorted` 或 `list.sort`) 。
- 须处理输入为空列表、元素非数字等异常情况，保证函数健壮性。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. **函数定义**：`def bubble_sort(arr, reverse=False, inplace=True):`
   - `arr`：待排序列表。
   - `reverse`：布尔值，`False` 为升序 (默认) ，`True` 为降序。
2. **输入校验**：
   - 若 `arr` 不是列表，抛出 `TypeError`。
   - 遍历列表，确保每个元素都是数字 (整数或浮点数) ，否则抛出 `ValueError` 或打印警告并返回。
   - 将转换后的数字存入新列表 `result` (原始列表不被修改) 。
3. **排序实现** (冒泡算法) ：
   - 若 `result` 为空或长度为 `1`，直接返回。
   - 外层循环控制轮数 (`n-1` 轮) 。
   - 内层循环比较相邻元素，根据 `reverse` 决定交换条件 (升序：`>`，降序：`<`) 。
   - 增加优化：若某轮无交换，提前结束。
4. **返回值**：返回新列表。

```python
def bubble_sort(arr, reverse=False):
    """
    冒泡排序 (返回新列表，不修改原列表) 
    :param arr: 可迭代对象，元素应为数字 (int, float) 或数字字符串
    :param reverse: False 为升序，True 为降序
    :return: 排序后的新列表
    :raises TypeError: arr 不可迭代
    :raises ValueError: 元素无法转换为数字
    """
    # 1. 检查 arr 是否可迭代 (简单判断是否有 __iter__) 
    if not hasattr(arr, "__iter__"):
        raise TypeError("arr 必须是可迭代对象")

    # 2. 转换为数字列表，同时进行合法性校验
    result = []
    for item in arr:
        try:
            num = float(item)   # 兼容 int, float, 数字字符串
        except (ValueError, TypeError):
            raise ValueError(f"元素 '{item}' 无法转换为数字")
        result.append(num)

    n = len(result)
    # 3. 空列表或单元素列表，直接返回
    if n <= 1:
        return result

    # 4. 冒泡排序
    for i in range(n - 1):
        swapped = False
        # 每轮比较范围逐步缩小
        for j in range(n - 1 - i):
            # 升序：前 > 后 则交换；降序：前 < 后 则交换
            if (not reverse and result[j] > result[j + 1]) or (reverse and result[j] < result[j + 1]):
                result[j], result[j + 1] = result[j + 1], result[j]
                swapped = True
        if not swapped:   # 本轮无交换，已有序
            break
    return result


# 测试示例
original = [64, 34, 25, 12, 22, 11, 90]
sorted_asc = bubble_sort(original)
print("原列表:", original)                # 原列表不变
print("升序结果:", sorted_asc)            # [11, 12, 22, 25, 34, 64, 90]

sorted_desc = bubble_sort(original, reverse=True)
print("降序结果:", sorted_desc)           # [90, 64, 34, 25, 22, 12, 11]

# 测试数字字符串
print(bubble_sort(["3", "1", "4", "1", "5"]))  # [1.0, 1.0, 3.0, 4.0, 5.0]

# 测试空列表
print(bubble_sort([]))   # []

# 测试非法输入
try:
    bubble_sort([10, "abc", 30])
except ValueError as e:
    print("捕获异常:", e)   # 元素 'abc' 无法转换为数字
```

**易错分析**

- **把函数写成只会打印结果**：排序函数应该返回排序后的列表，方便后续继续计算或保存；`print()` 更适合放在测试代码中。
- **修改了原列表却没有提醒调用者**：本题代码先创建 `result`，再排序 `result`，因此原始列表不会被改变。写函数时要想清楚：到底返回新结果，还是直接修改传入的数据。
- **异常处理过度复杂**：函数版比 5.5 难一些，因为它不仅要排序，还要处理空列表、数字字符串和非法元素。读者如果还不熟，可以先只实现“数字列表升序排序”，再逐步加上这些校验。
- **复用前没有测试边界**：函数写完后至少测试普通列表、空列表、数字字符串和非法输入四类情况。函数一旦被复用，隐藏错误会传播得更远。

</details>


## 6.2 写函数计算平均值

编写函数 `mean(values)`，接收一个数字列表，返回平均值。如果列表为空，返回 `None`。若列表中含有非数字元素，也返回 `None` (并给出警告) ，以确保计算结果的可靠性。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 函数先判断列表是否为空。
2. 遍历列表中的每个元素，尝试将其转换为浮点数 (以便同时支持整数、浮点数及数字字符串) 。若转换失败 (抛出 `ValueError`) ，说明元素不是有效数字，打印警告并返回 `None`。
3. 对所有有效数字 (转换后的浮点数) 进行累加。
4. 用总和除以数字的个数 (即列表长度，因为已确保全部合法) ，返回结果。
5. 在函数外部调用并输出，注意处理 `None` 的情况。

```python
def mean(values):
    # 1. 判空
    if len(values) == 0:
        return None

    # 2. 数据合法性校验：检查每个元素是否为数字
    numbers = []
    for item in values:
        try:
            num = float(item)          # 尝试转换为浮点数 (兼容 int, float, 数字字符串) 
            numbers.append(num)
        except (ValueError, TypeError):
            print(f"警告：忽略非法数据 '{item}'，该元素不是有效数字")
            return None                # 一旦发现非法数据，直接返回 None

    # 3. 求和
    total = 0
    for num in numbers:
        total += num

    # 4. 计算平均值
    average = total / len(numbers)
    return average


# 测试
scores = [88, 92, 75, 64, 90]
result = mean(scores)

print(result)
```

**易错分析**

- **函数中只 `print` 不 `return`**：初学者容易将结果直接打印出来，但这样函数调用后无法将结果赋值给变量或参与后续运算。函数应当通过 `return` 将计算结果“交还”给调用者，`print()` 只负责显示，二者用途不同。
- **忽视空列表：**如果不处理空列表，`total / len(values)` 会触发 `ZeroDivisionError`，导致程序崩溃。务必在除法前检查列表长度。
- **类型转换的统一性**：使用 `float(item)` 可以将整数、浮点数和数字字符串统一转换为浮点数，便于求和与平均计算。对于 `int` 或 `float` 类型的元素，`float()` 不会改变其数值；对于 `"123"` 这样的字符串，也能正确转换。这是处理混合类型数据的常用技巧。
- **是否提前返回 None**：当检测到非法数据时，是直接返回 `None` 还是跳过该元素继续计算？本方案选择“严格模式”——一旦发现非法数据立即返回 `None`，因为这能帮助调用者尽早发现数据质量问题。若业务需求允许忽略非法值，可改为 `continue` 并记录跳过次数，但需在计算平均值时调整分母。

</details>

## 6.3 写函数清洗一批数值文本

编写函数 `clean_number(text)`，把 `"5.1 cm"` 转成 `5.1`，把 `"NA"` 或空字符串转成 `None`。然后用它处理一组长度文本。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 定义清洗函数：`clean_number(text)` 只负责处理一个文本值，职责单一，便于复用和测试。
2. 数据清洗：
   - 使用 `strip()` 去除首尾空白字符；
   - **先判断缺失值**：若文本为空字符串或等于 `"NA"` (可扩展为 `"N/A"`、`"null"` 等) ，直接返回 `None`；
   - **去除单位**：用 `replace()` 删除已知单位 (如 `"cm"`) ，再次 `strip()` 清理残留空格
3. **批量处理**：遍历原始数据列表，对每个元素调用 `clean_number()`，将结果存入新列表。
4. **输出结果**：打印清洗后的列表，便于检查

```python
def clean_number(text):
    """
    清洗单个长度文本，转换为数字或 None
    :param text: 字符串，如 "5.1 cm" 或 "NA"
    :return: 浮点数 (成功) 或 None (缺失/非法) 
    """
    # 1. 去除首尾空格
    text = text.strip()

    # 2. 判断缺失值 (可扩展更多标记) 
    if text == "" or text.upper() in ("NA", "N/A", "NULL"):
        return None

    # 3. 去除单位 (示例中只处理 cm，可扩展) 
    #   注意：必须先处理单位，再尝试转换
    text = text.replace("cm", "").strip()

    # 4. 处理空字符串 (单位被移除后可能为空) 
    if text == "":
        return None

    # 5. 尝试转换为浮点数
    try:
        number = float(text)
    except ValueError:
        # 转换失败，打印警告并返回 None
        print(f"警告：无法转换 '{text}'，将作为缺失值处理")
        return None

    return number


# 原始数据
raw_values = ["5.1 cm", "4.9 cm", "NA", " 5.4 cm ", "", "7.2cm", "abc"]

# 批量清洗
clean_values = []
for value in raw_values:
    number = clean_number(value)
    clean_values.append(number)

print("原始数据:", raw_values)
print("清洗结果:", clean_values)

# 过滤掉 None，得到纯数字列表
valid_numbers = [x for x in clean_values if x is not None]
print("有效数字:", valid_numbers)
```

**易错分析**

- **清洗顺序错误：**应先去掉空格并判断缺失值，再去单位，最后做 `float()` 转换。如果一开始就转换，遇到 `"NA"`、空字符串或 `"5.1 cm"` 都会报错。
- **单位替换后残留空格**：`"5.1 cm"`.`replace("cm", "")` 会得到 `"5.1 "`，末尾有空格。直接 `float("5.1 ")` 会成功 (`float()` 能处理两侧空格) ，但更安全的做法是再次 `strip()`。
- **未处理转换失败**：如果文本是 `"abc"`，`float()` 会抛出异常。务必用 `try-except` 捕获，避免整个程序崩溃。本例中转换失败时返回 `None` 并给出警告。
- **函数职责应保持单一**：`clean_number()` 只处理单个文本，不负责批量遍历或输出。这样函数更纯粹，易于测试和复用；批量处理交由调用者完成 (`for` 循环或列表推导式)。

</details>

## 6.4 写函数统计成绩概况

编写函数 `summarize_scores(scores)`，接收一个成绩列表，返回平均分、最高分、最低分和及格人数。为了便于调用方理解和使用，将统计结果封装在字典中返回。

<details>
<summary><b>参考答案</b></summary>

**拆解**

1. 先过滤非法数据：跳过非数字、负数和超过 `100` 的成绩。
2. 初始化统计变量：
   - `total` 用于累加总分
   - `max_score` 和 `min_score` 初始化为列表的第一个元素 (基于数据本身，而非固定值如 `0`) ，避免因数据范围未知而导致的初始化错误。
   - `pass_count` 初始为 `0`，用于累计及格人数。
3. 遍历列表：在一次循环中同时完成求和，更新最大、最小值，统计及格人数三项任务，减少不必要的多次遍历。
4. 计算平均值：用总分除以列表长度。
5. 封装结果：将四个统计值存入字典并返回。

```python
def summarize_scores(scores):
    # 1. 过滤非法数据 (非数字、负数、超过100) 
    valid_scores = []
    for s in scores:
        try:
            num = float(s)          # 兼容数字字符串
        except (ValueError, TypeError):
            print(f"警告：忽略非数字项 '{s}'")
            continue

        if 0 <= num <= 100:
            valid_scores.append(num)
        else:
            print(f"警告：忽略异常值 {num} (不在 0~100 范围内) ")

    # 2. 检查过滤后是否为空
    if len(valid_scores) == 0:
        print("没有有效成绩，无法统计")
        return None

    # 3. 初始化统计变量 (基于有效数据) 
    total = 0
    max_score = valid_scores[0]
    min_score = valid_scores[0]
    pass_count = 0

    for score in valid_scores:
        total += score

        if score > max_score:
            max_score = score

        if score < min_score:
            min_score = score

        if score >= 60:
            pass_count += 1

    average = total / len(valid_scores)

    return {
        "平均分": average,
        "最高分": max_score,
        "最低分": min_score,
        "及格人数": pass_count,
        "有效人数": len(valid_scores)   # 额外信息，便于调用方了解数据质量
    }


# 测试含异常值的数据
scores = [88, 92, -5, 75, "abc", 64, 101, 90]
summary = summarize_scores(scores)
if summary:
    print(f"平均分：{summary['平均分']:.2f}，有效人数：{summary['有效人数']}")
```

**易错分析**

- **未过滤空列表和非法数据**：如果列表为空，或者所有数据都被过滤掉，继续计算平均分会导致除零错误；如果非法数据混入统计结果，平均分、最高分和最低分都会失真。
- **进行多次遍历**：求和、求最值、统计及格人数可以在一次循环中完成，效率更高，代码也更紧凑。

</details>

---

# 7. 复盘

做完题目后，不要只统计“错了几题”。更有价值的复盘方式是追问自己：**反复出错的是同一类问题吗？** 如果是，在后续的代码开发中就需要有意识地主动关注这类问题，而非被动等待错误发生。

| 常见问题类型 | 典型表现 | 后续开发需要关注 |
| --- | --- | --- |
| 读题与拆解问题 | 不知道从哪里开始，或者写着写着偏离题目要求 | 写代码前先明确“输入、处理、输出、边界”四项，用一两句话概括自己要做什么。 |
| 输入校验问题 | 遇到 `0`、负数、空值、超范围数据就报错或崩溃 | 每个输入变量至少准备 2-3 个边界测试用例，例如 `0`、空字符串、最大值、负值等。 |
| 变量类型问题 | 字符串和数字混用，`input()` 后直接进行计算 | 看到外部输入，先问自己：“它现在的类型是什么？后续计算需要什么类型？”确认后再决定是否转换。 |
| 条件分支问题 | 区间判断重叠、遗漏边界条件、`and` / `or` 用反 | 用表格列出所有区间，按从小到大或从大到小的顺序依次编写分支，确保每个分支互不重叠、边界覆盖完整。 |
| 循环与索引问题 | `range()` 的边界多一轮或少一轮，列表下标越界 | 假设循环共执行 3 次，手动列出 `i` 的取值 (例如 `0, 1, 2`) ，基于这些值验证表达式是否正确，尤其注意出现 `i+1` 或 `i-1` 时的边界情况。 |
| 列表组织问题 | 不知道该新建列表、修改原列表还是只计数 | 优先采用“新建列表”策略，先将符合条件的元素逐个追加到新列表中，最后再输出新列表或通过 `len()` 统计数量，逻辑更清晰且不易出错。 |
| 函数封装问题 | 同一段代码复制粘贴多遍，改一处漏一处 | 重复逻辑出现第二次时，就应考虑封装为函数；函数应尽量通过 `return` 返回结果，而不是直接在函数内部 `print()`，以便复用。 |
| 调试习惯问题 | 一报错就推倒重写，或者只看最后一行错误信息 | 保留完整的报错信息，先看行号定位错误位置，再看异常类型判断问题性质，最后用 `print()` 输出中间变量及变量类型进行验证。 |

如果你的错误主要集中在**变量类型**和**输入校验**，后续学习和处理数据时要特别关注：**每进行一步计算，就主动检查这步计算前后的变量类型是否符合预期**。例如，在 Pandas 中要特别关注 `dtype`、缺失值、字符串列与数值列之间的转换，而在原生 Python 中则要习惯用 `type()` 或 `isinstance()` 验证关键变量的类型。

如果你的错误主要集中在**条件分支**，后续做数据筛选时要重点检查筛选条件是否覆盖了所有情况，特别注意前面的分支是否无意中“吞掉”了本应进入后面分支的数据。

如果你的错误主要集中在**循环和列表**，后续处理表格数据时要先放慢速度，想清楚当前操作的对象是“一行”、“一列”还是“整张表”，再动手写代码。

如果你的错误主要集中在**函数封装**，说明你已经开始接近真实开发场景了。下一步的重点不是写更多零散代码，而是学会把数据清洗、指标计算和结果展示等流程拆解为可复用的小函数，逐步建立起自己的工具库。

> 新手阶段最重要的不是少犯错，而是能**说清楚自己为什么错**。能说清楚，代码能力就已经在进步了。老手也并不是不犯错，而是搞清楚犯错原因的速度更快、解决错误的方式更系统。错误本身不是问题，真正的问题是把错误归因于“粗心”就翻篇了——每一次复盘，都是代码能力的一次升级。