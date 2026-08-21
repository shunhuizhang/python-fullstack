

# Python全栈入门到实战【基础篇 17】循环进阶：推导式大全（列表/字典/集合）

哈喽各位小伙伴！上一节咱们吃透了函数的核心用法，能通过封装逻辑实现模块化开发——但在实际编程中，批量生成或处理数据时，你可能会写出这样的代码：
- 用3行循环生成一个偶数列表；
- 用5行代码筛选字典中的符合条件的键值对；
- 用嵌套循环去重并处理数据，代码层级繁琐。

这些场景下，Python的**推导式（Comprehension）** 能帮你用一行代码替代多行循环，既简洁又高效。推导式本质是“循环+条件判断”的语法糖，支持列表、字典、集合三种核心类型，是处理批量数据的“高效工具”。

这节咱们系统讲解推导式的核心用法，覆盖所有实用场景：
- 列表推导式：批量生成/筛选列表，替代`for+append`；
- 字典推导式：批量构建/转换字典，简化键值对操作；
- 集合推导式：批量去重+处理数据，结合集合特性；
- 推导式进阶：带条件判断、嵌套推导式、与普通循环的性能对比；
- 避坑要点：可读性边界、嵌套层级限制、生成器表达式区别。

掌握推导式后，你处理批量数据的代码会更简洁、执行效率更高，还能提升代码的可读性（合理使用前提下）～

@[TOC](文章目录)


# 一、前置引入：为什么需要推导式？
在推导式出现之前，批量生成或筛选数据需要写完整的循环结构，代码冗余且不够直观。比如：
```python
# 普通循环：生成1-10的偶数列表（3行代码）
even_nums = []
for num in range(1, 11):
    if num % 2 == 0:
        even_nums.append(num)
print(even_nums)  # [2, 4, 6, 8, 10]

# 推导式：一行代码实现相同功能
even_nums = [num for num in range(1, 11) if num % 2 == 0]
print(even_nums)  # [2, 4, 6, 8, 10]
```

推导式的核心价值的是：
1. **代码简洁**：用一行代码替代多行循环+条件判断，减少冗余；
2. **执行高效**：推导式的底层实现比普通循环更优化，执行速度更快（尤其是数据量较大时）；
3. **可读性强**：逻辑集中在一行，直观体现“输入→处理→输出”的流程（合理使用时）。

简单说：推导式是“批量数据处理”的最优解之一，适合场景：生成列表/字典/集合、筛选数据、格式转换等。


# 二、核心基础：列表推导式（List Comprehension）
列表推导式是最常用的推导式，用于快速生成或筛选列表，语法简洁且灵活。

## 1. 基础语法
```python
# 无条件判断：生成列表
[表达式 for 变量 in 可迭代对象]

# 带条件判断：筛选列表
[表达式 for 变量 in 可迭代对象 if 条件表达式]
```
### 语法说明
- 表达式：对变量的处理逻辑（如`num*2`、`str(num)`等）；
- 变量：遍历可迭代对象时的临时变量；
- 可迭代对象：列表、字符串、range、字典等可遍历的对象；
- 条件表达式：筛选条件，结果为`True`时才保留该元素。

## 2. 基础示例（无条件判断）
### 示例1：生成1-10的平方列表
```python
# 普通循环
squares = []
for num in range(1, 11):
    squares.append(num * num)

# 列表推导式
squares = [num * num for num in range(1, 11)]
print(squares)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

### 示例2：将字符串列表转为小写
```python
words = ["PYTHON", "JAVA", "C++", "GO"]
lower_words = [word.lower() for word in words]
print(lower_words)  # ['python', 'java', 'c++', 'go']
```

## 3. 进阶示例（带条件判断）
### 示例1：筛选1-20的奇数并乘以2
```python
# 普通循环
result = []
for num in range(1, 21):
    if num % 2 == 1:
        result.append(num * 2)

# 列表推导式
result = [num * 2 for num in range(1, 21) if num % 2 == 1]
print(result)  # [2, 6, 10, 14, 18, 22, 26, 30, 34, 38]
```

### 示例2：筛选字符串列表中长度大于3的元素
```python
fruits = ["apple", "banana", "pear", "orange", "grape"]
long_fruits = [fruit for fruit in fruits if len(fruit) > 3]
print(long_fruits)  # ['apple', 'banana', 'orange', 'grape']
```

## 4. 高级示例：嵌套列表推导式
用于处理二维列表（列表中的列表），语法：
```python
[表达式 for 外层变量 in 外层可迭代对象 for 内层变量 in 内层可迭代对象]
```
### 示例1：二维列表转一维列表
```python
# 普通循环
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flatten = []
for row in matrix:
    for num in row:
        flatten.append(num)

# 嵌套列表推导式
flatten = [num for row in matrix for num in row]
print(flatten)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 示例2：筛选二维列表中的偶数
```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
even_nums = [num for row in matrix for num in row if num % 2 == 0]
print(even_nums)  # [2, 4, 6, 8]
```


# 三、核心重点：字典推导式（Dict Comprehension）
字典推导式用于快速构建或转换字典，语法与列表推导式类似，但需指定“键-值”对。

## 1. 基础语法
```python
# 无条件判断：构建字典
{键表达式: 值表达式 for 变量 in 可迭代对象}

# 带条件判断：筛选键值对
{键表达式: 值表达式 for 变量 in 可迭代对象 if 条件表达式}

# 遍历字典：转换键值对
{新键表达式: 新值表达式 for 键, 值 in 原字典.items() if 条件表达式}
```

## 2. 基础示例（构建字典）
### 示例1：生成“数字-平方”字典
```python
# 普通循环
square_dict = {}
for num in range(1, 6):
    square_dict[num] = num * num

# 字典推导式
square_dict = {num: num * num for num in range(1, 6)}
print(square_dict)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

### 示例2：用两个列表构建字典（键值对应）
```python
keys = ["name", "age", "phone"]
values = ["张三", 20, "13812345678"]
user_dict = {key: value for key, value in zip(keys, values)}
print(user_dict)  # {'name': '张三', 'age': 20, 'phone': '13812345678'}
```
**说明：`zip(keys, values)` 用于将两个列表按索引配对，返回元组迭代器。**

## 3. 进阶示例（转换/筛选字典）
### 示例1：交换字典的键和值
```python
original_dict = {"a": 1, "b": 2, "c": 3}
swapped_dict = {value: key for key, value in original_dict.items()}
print(swapped_dict)  # {1: 'a', 2: 'b', 3: 'c'}
```

### 示例2：筛选字典中值大于90的键值对
```python
score_dict = {"math": 85, "english": 92, "python": 78, "java": 95}
high_score = {subject: score for subject, score in score_dict.items() if score > 90}
print(high_score)  # {'english': 92, 'java': 95}
```

### 示例3：字典值格式转换（数字转字符串）
```python
data_dict = {"id": 1001, "age": 25, "score": 90}
str_dict = {key: str(value) for key, value in data_dict.items()}
print(str_dict)  # {'id': '1001', 'age': '25', 'score': '90'}
```


# 四、补充重点：集合推导式（Set Comprehension）
集合推导式用于快速生成集合（自动去重），语法与列表推导式类似，用`{}`包裹。

## 1. 基础语法
```python
# 无条件判断：生成集合
{表达式 for 变量 in 可迭代对象}

# 带条件判断：筛选集合
{表达式 for 变量 in 可迭代对象 if 条件表达式}
```

## 2. 核心示例（去重+筛选）
### 示例1：生成1-10的偶数集合（自动去重）
```python
# 普通循环
even_set = set()
for num in range(1, 11):
    if num % 2 == 0:
        even_set.add(num)

# 集合推导式
even_set = {num for num in range(1, 11) if num % 2 == 0}
print(even_set)  # {2, 4, 6, 8, 10}（集合无序）
```

### 示例2：字符串去重并转为大写
```python
text = "abacbd"
unique_chars = {char.upper() for char in text}
print(unique_chars)  # {'A', 'B', 'C', 'D'}（去重+大写）
```

### 示例3：筛选列表中大于5的元素并去重
```python
nums = [1, 5, 3, 5, 7, 9, 7, 10]
unique_gt5 = {num for num in nums if num > 5}
print(unique_gt5)  # {7, 9, 10}（去重+筛选）
```


# 五、推导式进阶：嵌套推导与性能对比
## 1. 嵌套推导式（列表/字典/集合）
嵌套推导式支持多层循环，适合处理复杂数据结构，但嵌套层级建议不超过2层（否则可读性下降）。

### 示例1：嵌套列表推导式（生成二维列表）
```python
# 生成3x3的矩阵（1-9）
matrix = [[row * 3 + col + 1 for col in range(3)] for row in range(3)]
print(matrix)  # [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

### 示例2：嵌套字典推导式（处理嵌套字典）
```python
# 原始数据：用户成绩字典
user_scores = {
    "张三": {"math": 85, "english": 92},
    "李四": {"math": 78, "english": 88},
    "王五": {"math": 95, "english": 90}
}
# 筛选数学成绩大于80的用户，构建新字典
high_math = {name: scores for name, scores in user_scores.items() if scores["math"] > 80}
print(high_math)  # {'张三': {'math': 85, 'english': 92}, '王五': {'math': 95, 'english': 90}}
```

## 2. 推导式 vs 普通循环：性能对比
推导式的执行效率高于普通循环，原因是推导式的底层是C语言实现，减少了Python层面的循环开销。

### 性能测试示例（生成10万条数据）
```python
import time

# 普通循环
start = time.time()
result = []
for num in range(100000):
    if num % 2 == 0:
        result.append(num * 2)
end = time.time()
print(f"普通循环耗时：{end - start:.4f}秒")

# 列表推导式
start = time.time()
result = [num * 2 for num in range(100000) if num % 2 == 0]
end = time.time()
print(f"列表推导式耗时：{end - start:.4f}秒")
```
### 运行结果（参考）
```
普通循环耗时：0.0082秒
列表推导式耗时：0.0045秒
```
**结论：推导式的执行速度约为普通循环的2倍（数据量越大，差距越明显）。**


# 六、核心避坑要点
## 1. 可读性优先：避免过度复杂的推导式
推导式虽简洁，但过度嵌套或逻辑复杂会导致可读性下降，比如：
```python
# 不推荐：逻辑复杂，难以理解
complex_expr = [x*y for x in range(5) if x > 2 for y in range(3) if y < 2]

# 推荐：拆分为普通循环（逻辑更清晰）
complex_expr = []
for x in range(5):
    if x > 2:
        for y in range(3):
            if y < 2:
                complex_expr.append(x*y)
```
**原则：推导式逻辑不超过1个条件+1层嵌套，复杂逻辑用普通循环。**

## 2. 集合推导式 vs 列表推导式：去重差异
- 列表推导式保留重复元素，集合推导式自动去重，需根据需求选择：
```python
nums = [1, 2, 2, 3, 3, 3]
list_expr = [num for num in nums]
set_expr = {num for num in nums}
print(list_expr)  # [1, 2, 2, 3, 3, 3]（保留重复）
print(set_expr)  # {1, 2, 3}（自动去重）
```

## 3. 字典推导式：键必须唯一
字典的键是唯一的，推导式中若出现重复键，后定义的会覆盖前定义的：
```python
# 重复键，后值覆盖前值
dict_expr = {num: num*2 for num in [1, 2, 2, 3]}
print(dict_expr)  # {1: 2, 2: 4, 3: 6}（重复的2只保留最后一个）
```

## 4. 推导式与生成器表达式的区别
- 推导式：立即生成完整的列表/字典/集合，占用内存（如`[x for x in range(1000000)]`）；
- 生成器表达式：用`()`包裹，延迟生成数据，占用内存少（如`(x for x in range(1000000))`）；
### 示例：生成器表达式
```python
gen = (num * 2 for num in range(10))
print(gen)  # <generator object <genexpr> at 0x0000021F7D8A3C10>（不立即生成数据）
# 遍历生成器
for value in gen:
    print(value, end=" ")  # 0 2 4 6 8 10 12 14 16 18
```
**适用场景：**数据量极大时用生成器表达式，避免内存溢出。


# 七、实战案例：推导式批量处理数据
需求：基于推导式实现“学生成绩数据清洗与统计”，完成以下功能：
1. 筛选数学成绩大于80的学生；
2. 计算所有学生的平均成绩；
3. 构建“学生姓名-平均成绩”字典；
4. 统计各成绩段（优秀/良好/及格/不及格）的学生人数。

```python
# 原始数据：学生成绩列表（嵌套字典）
students = [
    {"name": "张三", "math": 85, "english": 92, "python": 78},
    {"name": "李四", "math": 78, "english": 88, "python": 90},
    {"name": "王五", "math": 95, "english": 90, "python": 92},
    {"name": "赵六", "math": 65, "english": 72, "python": 68},
    {"name": "孙七", "math": 58, "english": 63, "python": 70}
]

# 1. 筛选数学成绩大于80的学生（列表推导式）
math_gt80 = [student for student in students if student["math"] > 80]
print("数学成绩大于80的学生：")
for student in math_gt80:
    print(f"  {student['name']}：{student['math']}分")

# 2. 计算所有学生的平均成绩（列表推导式+内置函数）
# 先获取所有学生的平均成绩列表
avg_scores = [
    (student["math"] + student["english"] + student["python"]) / 3
    for student in students
]
# 计算总体平均成绩
total_avg = sum(avg_scores) / len(avg_scores)
print(f"\n所有学生的平均成绩：{total_avg:.1f}分")

# 3. 构建“学生姓名-平均成绩”字典（字典推导式）
name_avg_dict = {
    student["name"]: (student["math"] + student["english"] + student["python"]) / 3
    for student in students
}
print("\n学生姓名-平均成绩：")
for name, avg in name_avg_dict.items():
    print(f"  {name}：{avg:.1f}分")

# 4. 统计各成绩段的学生人数（字典推导式+条件判断）
# 定义成绩段判断函数
def get_grade(score):
    if score >= 90:
        return "优秀"
    elif score >= 80:
        return "良好"
    elif score >= 60:
        return "及格"
    else:
        return "不及格"

# 统计人数（先生成成绩段列表，再用字典推导式统计）
grade_list = [get_grade(avg) for avg in avg_scores]
grade_count = {grade: grade_list.count(grade) for grade in grade_list}
print("\n各成绩段学生人数：")
for grade, count in grade_count.items():
    print(f"  {grade}：{count}人")
```

**运行结果**：
```
数学成绩大于80的学生：
  张三：85分
  王五：95分

所有学生的平均成绩：80.1分

学生姓名-平均成绩：
  张三：85.0分
  李四：85.3分
  王五：92.3分
  赵六：68.3分
  孙七：63.7分

各成绩段学生人数：
  良好：2人
  优秀：1人
  及格：2人
```


# 八、总结
本节系统讲解了Python推导式的核心用法，核心要点梳理如下：
1. **推导式类型**：
   - 列表推导式：`[表达式 for 变量 in 可迭代对象 if 条件]`，批量生成/筛选列表；
   - 字典推导式：`{键: 值 for 变量 in 可迭代对象 if 条件}`，批量构建/转换字典；
   - 集合推导式：`{表达式 for 变量 in 可迭代对象 if 条件}`，批量去重+筛选。
2. **核心优势**：代码简洁、执行高效，适合批量数据处理场景；
3. **进阶用法**：支持条件判断、嵌套推导（建议不超过2层）；
4. **避坑要点**：
   - 可读性优先，避免复杂推导式；
   - 集合推导式自动去重，字典推导式键唯一；
   - 数据量极大时用生成器表达式替代推导式；
5. **适用场景**：数据生成、筛选、格式转换、统计分析等批量操作。

推导式是Python的“高效语法糖”，合理使用能大幅提升开发效率，但需注意“简洁与可读性的平衡”——复杂逻辑仍建议用普通循环，避免过度追求一行代码而降低可维护性。

下一节，咱们会学习**Python异常处理（try-except-finally）**：解决程序运行中的报错问题，让程序更健壮，避免因意外错误导致崩溃～


# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后，专栏内所有文章可永久免费查看，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/157911088>
