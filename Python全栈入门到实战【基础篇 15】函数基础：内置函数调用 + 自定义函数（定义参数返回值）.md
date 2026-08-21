

# Python全栈入门到实战【基础篇 15】函数基础：内置函数调用 + 自定义函数（定义/参数/返回值）

哈喽各位小伙伴！前面咱们吃透了循环、条件判断、复合数据类型，能写出处理单一任务的代码——但实际开发中常会遇到这样的问题：
- 验证手机号格式的逻辑，在“用户注册”和“数据清洗”场景都需用到，重复编写不仅冗余，修改时还需同步更新多处；
- 计算成绩等级的规则调整后，所有涉及该逻辑的代码都要逐一修改；
- 代码规模扩大后，循环与判断语句交织，排查问题时需逐行检索，维护成本极高。

这些问题的核心，是缺少“代码封装复用”的能力——而Python的**函数（Function）** 正是解决该问题的核心方案。函数能将“完成特定功能的代码段”封装为可调用模块，使用时仅需一行代码调用，无需重复编写逻辑，如同将常用工具收纳于工具箱，取用便捷且结构清晰。

本节将系统讲解函数的核心用法，覆盖内置函数与自定义函数的全场景应用：
- 内置函数：Python自带的基础工具集（`len()`/`range()`/`enumerate()`等）；
- 自定义函数：封装专属业务逻辑（定义语法、参数设计、返回值处理）；
- 函数参数：位置参数、关键字参数、默认参数、可变参数（覆盖绝大多数开发场景）；
- 核心特性：作用域规则、返回值机制、函数调用流程；
- 常见问题：参数传递错误、返回值遗漏、作用域混淆等问题的解决方案。

掌握函数的使用，能让代码从“零散执行”升级为“模块化复用”，大幅提升代码的简洁性与可维护性。

@[TOC](文章目录)


# 一、前置引入：为什么需要函数？
没有函数的编程模式，如同使用散装零件完成任务——每次实现特定功能都需重新组合代码片段；而函数则是将零件组装成的标准化工具，一次封装即可反复使用。

用一个生活例子帮你理解：你经常需要 “切水果”，如果没有工具，每次都要找刀、洗刀、切水果、洗刀，步骤重复又麻烦；但你买了一个 “切水果机”（函数），以后要切水果时，只需把水果（参数）放进去，按一下开关（调用函数），机器就会自动完成切水果的步骤，最后给你切好的水果（返回值）。

函数的核心价值体现在三个维度：
1. **代码复用**：相同逻辑仅需编写一次，通过函数调用在多场景复用，避免重复编码（如手机号验证逻辑封装后，注册、数据清洗场景可直接调用）；
2. **模块化开发**：将复杂任务拆解为多个单一功能的函数（如用户管理系统拆分为“添加/查询/删除”函数），代码结构清晰，便于分工协作与问题定位；
3. **易维护性**：逻辑修改仅需更新函数内部实现，所有调用处自动生效（如调整成绩等级判定规则时，仅需修改`get_score_level()`函数）。

通过以下对比可直观感受函数的优势：
```python
# 无函数：重复编写成绩等级判断逻辑（冗余且维护成本高）
score1 = 85
if score1 >= 90:
    print("优秀")
elif score1 >= 80:
    print("良好")

score2 = 75
if score2 >= 90:
    print("优秀")
elif score2 >= 80:
    print("良好")
elif score2 >= 60:
    print("及格")

# 有函数：封装逻辑，调用即可（简洁且易维护）
def get_score_level(score):
    if score >= 90:
        return "优秀"
    elif score >= 80:
        return "良好"
    elif score >= 60:
        return "及格"
    else:
        return "不及格"

print(get_score_level(85))  # 良好
print(get_score_level(75))  # 及格
```


# 二、基础中的基础：内置函数（Python自带的“工具集”）
Python内置了大量开箱即用的函数，无需手动定义即可直接调用，覆盖日常开发80%的基础需求。以下按功能分类梳理核心内置函数，配套示例帮助快速理解：

## 1. 基础操作类（高频使用）
| 函数                      | 功能                                         | 示例代码                                  | 运行结果                         |
| ------------------------- | -------------------------------------------- | ----------------------------------------- | -------------------------------- |
| `len(obj)`                | 获取对象长度（列表元素个数、字符串字符数等） | `len([1,2,3])`、`len("Python")`           | 3、6                             |
| `type(obj)`               | 获取对象的数据类型                           | `type("Python")`、`type(123)`             | `<class 'str'>`、`<class 'int'>` |
| `print(*args)`            | 输出内容至控制台                             | `print("Hello Python!")`                  | Hello Python!                    |
| `input(prompt)`           | 接收用户输入，返回字符串类型                 | `name = input("请输入姓名：")`            | 读取用户输入并赋值给name         |
| `range(start, end, step)` | 生成指定范围的数字序列（左闭右开）           | `list(range(1,5))`、`list(range(2,10,2))` | `[1,2,3,4]`、`[2,4,6,8]`         |

### 示例：基于内置函数的简单交互
```python
# 1. 接收用户输入
name = input("请输入你的姓名：")
age = int(input("请输入你的年龄："))
# 2. 计算姓名长度
name_length = len(name)
# 3. 输出结果
print(f"你好，{name}！你的姓名包含{name_length}个字符，明年你将年满{age+1}岁。")
```
**运行示例**：
```
请输入你的姓名：张三
请输入你的年龄：25
你好，张三！你的姓名包含2个字符，明年你将年满26岁。
```

## 2. 序列操作类（处理列表/字符串常用）
| 函数                          | 功能                             | 示例代码                                           | 运行结果                     |
| ----------------------------- | -------------------------------- | -------------------------------------------------- | ---------------------------- |
| `enumerate(iter, start=0)`    | 遍历序列时返回（索引，元素）组合 | `list(enumerate(["苹果","香蕉"]))`                 | `[(0, '苹果'), (1, '香蕉')]` |
| `max(iter)`                   | 获取序列中的最大值               | `max([1,5,3,9])`                                   | 9                            |
| `min(iter)`                   | 获取序列中的最小值               | `min([1,5,3,9])`                                   | 1                            |
| `sum(iter)`                   | 计算序列元素的累加和             | `sum([1,2,3,4])`                                   | 10                           |
| `sorted(iter, reverse=False)` | 对序列进行排序（默认升序）       | `sorted([3,1,2])`、`sorted([3,1,2], reverse=True)` | `[1,2,3]`、`[3,2,1]`         |

### 示例：基于内置函数的成绩分析
```python
# 学生成绩列表
scores = [85, 92, 78, 95, 60]
# 1. 计算统计指标
highest = max(scores)
lowest = min(scores)
average = sum(scores) / len(scores)
sorted_scores = sorted(scores)
# 2. 遍历并输出详情
print("===== 成绩分析结果 =====")
print(f"最高分：{highest}")
print(f"最低分：{lowest}")
print(f"平均分：{average:.1f}")
print("排序后的成绩：", sorted_scores)
print("\n成绩详情：")
for idx, score in enumerate(scores, start=1):
    print(f"第{idx}名：{score}分")
```
**运行结果**：
```
===== 成绩分析结果 =====
最高分：95
最低分：60
平均分：82.0
排序后的成绩： [60, 78, 85, 92, 95]

成绩详情：
第1名：85分
第2名：92分
第3名：78分
第4名：95分
第5名：60分
```

## 3. 类型转换类（解决类型不匹配问题）
| 函数        | 功能                                       | 示例代码                              | 运行结果                   |
| ----------- | ------------------------------------------ | ------------------------------------- | -------------------------- |
| `int(obj)`  | 转换为整数类型（支持纯数字字符串、浮点数） | `int("123")`、`int(3.9)`              | 123、3                     |
| `str(obj)`  | 转换为字符串类型                           | `str(123)`、`str([1,2])`              | "123"、"[1,2]"             |
| `list(obj)` | 转换为列表类型                             | `list((1,2,3))`、`list("abc")`        | `[1,2,3]`、`['a','b','c']` |
| `dict(obj)` | 转换为字典类型（需传入键值对结构数据）     | `dict([("name","张三"), ("age",20)])` | `{"name":"张三","age":20}` |
| `set(obj)`  | 转换为集合类型（自动去重）                 | `set([1,2,2,3])`                      | `{1,2,3}`                  |

### 示例：解决类型不匹配问题
```python
# 错误示例：整数与字符串无法直接拼接
age = 18
# print("我今年" + age + "岁")  # TypeError: can only concatenate str (not "int") to str

# 正确示例：使用str()进行类型转换
print("我今年" + str(age) + "岁")  # 我今年18岁

# 接收用户输入并转换为整数
score = int(input("请输入你的成绩："))
print(f"你的成绩等级为：{get_score_level(score)}")
```

## 4. 内置函数调用规则
调用内置函数需遵循以下规范：
1. 语法格式：`函数名(参数1, 参数2, ...)`，参数需与函数定义的要求一致；
2. 参数传递：必填参数必须按要求传递，可选参数可根据需求省略；
3. 返回值处理：多数函数会返回处理结果（可赋值给变量后续使用），部分函数（如`print()`）无返回值。


# 三、核心重点：自定义函数（封装专属逻辑）
内置函数仅能满足基础需求，实际开发中需封装专属业务逻辑（如数据验证、业务计算等），这就需要自定义函数。自定义函数的核心是“定义→调用”的闭环，以下系统讲解其实现与使用。

## 1. 自定义函数的定义语法
```python
def 函数名(参数列表):
    """函数文档字符串（可选，用于说明函数功能、参数及返回值）"""
    函数体代码（需缩进4个空格）
    return 返回值（可选，无返回值时默认返回None）
```
### 各部分说明
- `def`：定义函数的关键字，必须位于函数名前；
- 函数名：遵循Python命名规范（小写字母+下划线），需见名知意（如`check_phone`表示手机号验证）；
- 参数列表：函数的输入数据，可根据需求定义多个参数或无参数；
- 文档字符串：用`""" """`包裹，用于说明函数功能，可通过`help(函数名)`查看；
- 函数体：实现核心逻辑的代码块，必须缩进；
- `return`：用于返回函数处理结果，执行后函数立即终止，后续代码不再执行。

### 完整示例：成绩等级判断函数
```python
def get_score_level(score):
    """
    根据分数判断成绩等级
    参数：
        score: 整数类型，0-100之间的分数
    返回值：
        字符串类型，返回"优秀/良好/及格/不及格"
    """
    if score >= 90:
        return "优秀"
    elif score >= 80:
        return "良好"
    elif score >= 60:
        return "及格"
    else:
        return "不及格"
```

## 2. 函数的核心要素：参数（输入数据）
参数是函数的输入接口，决定了函数可处理的数据类型与范围。以下讲解4种常用参数类型，覆盖绝大多数开发场景：

### （1）位置参数（基础类型）
位置参数需按定义顺序传递，数量与函数定义必须一致，是最常用的参数类型。

#### 示例：两数求和函数
```python
def add(a, b):
    """计算两个数的和（位置参数示例）"""
    return a + b

# 按顺序传递参数
result1 = add(1, 2)
print(f"1+2={result1}")  # 1+2=3

# 交换参数顺序（结果改变）
result2 = add(2, 1)
print(f"2+1={result2}")  # 2+1=3

# 错误示例：参数数量不匹配
# add(1)  # TypeError: add() missing 1 required positional argument: 'b'
```

### （2）关键字参数（增强可读性）
关键字参数通过“参数名=值”的形式传递，顺序可任意调整，适合参数较多的场景，能提升代码可读性。

#### 示例：用户信息打印函数
```python
def print_user_info(name, age, phone):
    """打印用户信息（关键字参数示例）"""
    print(f"姓名：{name}，年龄：{age}，手机号：{phone}")

# 纯关键字参数（顺序任意）
print_user_info(age=20, name="张三", phone="13812345678")
# 混合位置参数与关键字参数（位置参数需在前）
print_user_info("李四", phone="13987654321", age=22)
```
**运行结果**：
```
姓名：张三，年龄：20，手机号：13812345678
姓名：李四，年龄：22，手机号：13987654321
```

### （3）默认参数（可选输入）
默认参数在定义时指定默认值，调用时可省略（使用默认值），也可传递新值覆盖默认值，适合可选配置类场景。

#### 示例：带默认值的成绩等级判断
```python
def get_score_level(score, full_score=100):
    """
    判断成绩等级（默认参数示例）
    参数：
        score: 分数（必填）
        full_score: 满分（可选，默认100分）
    返回值：
        等级字符串
    """
    ratio = score / full_score  # 得分率
    if ratio >= 0.9:
        return "优秀"
    elif ratio >= 0.8:
        return "良好"
    elif ratio >= 0.6:
        return "及格"
    else:
        return "不及格"

# 省略默认参数（使用100分制）
print(get_score_level(85))  # 良好

# 覆盖默认参数（使用150分制）
print(get_score_level(120, full_score=150))  # 良好
```

### （4）可变参数（灵活接收多参数）
当参数数量不确定时，可使用可变参数，分为两种类型：
- `*args`：接收任意数量的位置参数，打包为元组；
- `**kwargs`：接收任意数量的关键字参数，打包为字典。

#### 示例1：`*args`实现任意个数求和
```python
def add_all(*args):
    """计算任意个数的累加和（*args示例）"""
    total = 0
    for num in args:
        total += num
    return total

# 传递不同数量的参数
print(add_all(1, 2))  # 3
print(add_all(1, 2, 3, 4))  # 10
print(add_all())  # 0（无参数时返回0）
```

#### 示例2：`**kwargs`实现任意用户信息打印
```python
def print_any_info(**kwargs):
    """打印任意用户信息（**kwargs示例）"""
    for key, value in kwargs.items():
        print(f"{key}：{value}")

# 传递不同数量的关键字参数
print_any_info(name="张三", age=20)
print_any_info(name="李四", phone="13987654321", address="北京市")
```
**运行结果**：
```
name：张三
age：20
name：李四
phone：13987654321
address：北京市
```

## 3. 函数的核心要素：返回值（输出结果）
`return`语句用于返回函数处理结果，同时终止函数执行。根据需求不同，返回值分为以下三种情况：

### （1）无返回值（默认返回None）
函数无`return`语句时，默认返回`None`，适合仅执行操作（如打印、修改全局变量）的场景。

#### 示例：无返回值函数
```python
def print_hello():
    """仅打印信息，无返回值"""
    print("Hello Python!")

result = print_hello()
print(f"函数返回值：{result}")  # 函数返回值：None
```

### （2）返回单个值
最常用场景，`return`后接单个数据（如数字、字符串、字典等）。

#### 示例：计算圆的面积
```python
def get_circle_area(radius):
    """计算圆的面积（πr²）"""
    pi = 3.1415926
    area = pi * radius * radius
    return area

area = get_circle_area(5)
print(f"半径为5的圆面积：{area:.2f}")  # 半径为5的圆面积：78.54
```

### （3）返回多个值
`return`后用逗号分隔多个值，Python会自动打包为元组返回，调用时可通过解包获取单个值。

#### 示例：计算矩形的周长与面积
```python
def get_rect_info(width, height):
    """计算矩形的周长和面积，返回多个值"""
    perimeter = 2 * (width + height)
    area = width * height
    return perimeter, area  # 打包为元组返回

# 解包获取多个返回值
perimeter, area = get_rect_info(3, 4)
print(f"矩形周长：{perimeter}，面积：{area}")  # 矩形周长：14，面积：12

# 直接接收为元组
info = get_rect_info(3, 4)
print(f"矩形信息：{info}")  # 矩形信息：(14, 12)
```

### （4）`return`终止函数执行
```python
def check_positive(num):
    """判断数字是否为正数"""
    if num > 0:
        return True  # 执行后函数终止，后续代码不执行
    print(f"数字{num}不是正数")
    return False

print(check_positive(5))  # True（无打印输出）
print(check_positive(-3))  # 打印"数字-3不是正数"，返回False
```

## 4. 函数的调用
定义函数后，需通过调用触发执行，语法格式为：
```python
函数名(参数1, 参数2, ...)
```

### 完整流程示例：手机号验证函数
```python
# 定义函数
def check_phone(phone):
    """
    验证手机号格式（11位数字）
    参数：
        phone: 手机号字符串
    返回值：
        合法返回True，不合法返回False
    """
    if not isinstance(phone, str):
        return False
    return phone.isdigit() and len(phone) == 11

# 调用函数
phone1 = "13812345678"
phone2 = "1398765432"
phone3 = "137abc12345"

print(f"{phone1}是否合法：{check_phone(phone1)}")  # True
print(f"{phone2}是否合法：{check_phone(phone2)}")  # False
print(f"{phone3}是否合法：{check_phone(phone3)}")  # False
```


# 四、函数的核心特性：作用域（变量生效范围）
作用域定义了变量的生效范围，即变量在代码的哪些区域可被访问。Python中函数相关的作用域分为两种：

## 1. 局部作用域（函数内部）
- 函数内定义的变量（包括参数）称为**局部变量**；
- 仅在函数内部生效，外部无法访问。

#### 示例：局部变量的作用域
```python
def test_local():
    local_var = "局部变量"
    print(f"函数内访问：{local_var}")  # 正常访问

test_local()
# 错误：外部无法访问局部变量
# print(f"函数外访问：{local_var}")  # NameError: name 'local_var' is not defined
```

## 2. 全局作用域（函数外部）
- 函数外定义的变量称为**全局变量**；
- 函数内部可直接读取全局变量，但默认无法修改（需通过`global`关键字声明）。

### （1）读取全局变量
```python
global_var = "全局变量"

def test_read_global():
    print(f"函数内读取：{global_var}")  # 直接读取全局变量

test_read_global()
print(f"函数外读取：{global_var}")  # 正常读取
```

### （2）修改全局变量（需`global`声明）
```python
global_var = 10

def test_modify_global():
    global global_var  # 声明修改全局变量
    global_var = 20
    print(f"函数内修改后：{global_var}")

test_modify_global()
print(f"函数外查看：{global_var}")  # 20（已被修改）
```

### 注意：避免局部变量与全局变量同名
```python
global_var = 10

def test_confuse():
    global_var = 20  # 定义局部变量，与全局变量同名
    print(f"函数内局部变量：{global_var}")  # 20

test_confuse()
print(f"函数外全局变量：{global_var}")  # 10（全局变量未被修改）
```


# 五、常见问题与解决方案
## 1. 参数数量/类型不匹配
- 表现：抛出`TypeError`，如“missing 1 required positional argument”“unsupported operand type(s)”；
- 原因：调用时参数数量与定义不一致，或参数类型不符合函数逻辑；
- 解决方案：
  1. 检查参数数量，确保必填参数全部传递；
  2. 验证参数类型，确保与函数定义要求一致（如需字符串则传递字符串）。

#### 错误示例与修正
```python
def add(a, b):
    return a + b

# 错误1：少传参数
# add(1)  # 报错：missing 1 required positional argument: 'b'

# 错误2：类型不匹配
# add("1", 2)  # 报错：unsupported operand type(s) for +: 'str' and 'int'

# 正确示例
print(add(1, 2))  # 3
```

## 2. 遗漏返回值
- 表现：函数调用后返回`None`，与预期结果不符；
- 原因：函数逻辑需返回结果，但未添加`return`语句；
- 解决方案：明确函数需返回结果时，在逻辑结束处添加`return`语句。

#### 错误示例与修正
```python
# 错误示例：遗漏return
def get_double(num):
    result = num * 2  # 无return语句

print(get_double(5))  # None（预期为10）

# 正确示例：添加return
def get_double(num):
    result = num * 2
    return result

print(get_double(5))  # 10
```

## 3. 修改全局变量未声明`global`
- 表现：函数内修改全局变量后，外部查看无变化；
- 原因：未通过`global`关键字声明，函数内修改的是局部变量；
- 解决方案：修改全局变量前，添加`global 变量名`声明。

## 4. 默认参数使用可变类型
- 表现：默认参数多次调用后保留上次的值；
- 原因：Python中可变类型（列表、字典等）的默认参数仅初始化一次；
- 解决方案：默认参数使用`None`，函数内初始化可变类型。

#### 错误示例与修正
```python
# 错误示例：默认参数为列表
def add_item(item, lst=[]):
    lst.append(item)
    return lst

print(add_item(1))  # [1]
print(add_item(2))  # [1,2]（预期为[2]）

# 正确示例：默认参数为None
def add_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst

print(add_item(1))  # [1]
print(add_item(2))  # [2]
```

## 5. 函数名重复
- 表现：后定义的函数覆盖先定义的函数，调用时执行错误逻辑；
- 原因：函数命名重复；
- 解决方案：函数名按功能命名，避免重复（如`add`用于两数求和，`add_all`用于任意数求和）。


# 六、实战案例：用户信息管理工具（函数版）
基于函数封装实现用户信息管理功能，包含添加、查询、删除、批量验证等模块，代码模块化且易维护。

```python
# 全局变量：存储用户信息（姓名→字典）
users = {}

def add_user(name, age, phone):
    """
    添加用户（含手机号格式验证）
    参数：
        name: 姓名（字符串）
        age: 年龄（整数）
        phone: 手机号（字符串）
    返回值：
        添加成功返回True，失败返回False
    """
    if not check_phone(phone):
        print("错误：手机号格式不合法（需为11位数字）。")
        return False
    if name in users:
        print("错误：该用户已存在。")
        return False
    users[name] = {"age": age, "phone": phone}
    print("用户添加成功。")
    return True

def query_user(name):
    """
    查询用户信息
    参数：
        name: 姓名（字符串）
    返回值：
        存在返回用户信息字典，不存在返回None
    """
    user_info = users.get(name)
    if user_info:
        print(f"\n===== {name}的用户信息 =====")
        print(f"姓名：{name}")
        print(f"年龄：{user_info['age']}岁")
        print(f"手机号：{user_info['phone']}")
    else:
        print("未找到该用户。")
    return user_info

def delete_user(name):
    """
    删除用户
    参数：
        name: 姓名（字符串）
    返回值：
        删除成功返回True，失败返回False
    """
    if name in users:
        del users[name]
        print("用户删除成功。")
        return True
    else:
        print("未找到该用户，删除失败。")
        return False

def check_phone(phone):
    """验证手机号格式（11位数字）"""
    if not isinstance(phone, str):
        return False
    return phone.isdigit() and len(phone) == 11

def batch_check_phones():
    """批量验证所有用户的手机号格式"""
    if not users:
        print("暂无用户信息，无法进行批量验证。")
        return
    print("\n===== 手机号批量验证结果 =====")
    invalid_users = []
    for name, info in users.items():
        if not check_phone(info["phone"]):
            invalid_users.append((name, info["phone"]))
    if invalid_users:
        for name, phone in invalid_users:
            print(f"{name}：{phone}（格式错误）")
    else:
        print("所有用户的手机号格式均合法。")

def main():
    """主函数：交互式菜单"""
    while True:
        print("\n===== 用户信息管理工具 =====")
        print("1：添加用户")
        print("2：查询用户")
        print("3：删除用户")
        print("4：批量验证手机号")
        print("5：退出程序")
        cmd = input("请输入操作指令（1-5）：")
        
        if cmd == "1":
            name = input("请输入姓名：")
            age = input("请输入年龄：")
            phone = input("请输入手机号：")
            if not age.isdigit():
                print("错误：年龄必须为数字。")
                continue
            add_user(name, int(age), phone)
        elif cmd == "2":
            name = input("请输入要查询的姓名：")
            query_user(name)
        elif cmd == "3":
            name = input("请输入要删除的姓名：")
            delete_user(name)
        elif cmd == "4":
            batch_check_phones()
        elif cmd == "5":
            print("退出程序，感谢使用。")
            break
        else:
            print("未知指令，请输入1-5之间的数字。")

# 程序入口
if __name__ == "__main__":
    main()
```

**运行示例**：
```
===== 用户信息管理工具 =====")
1：添加用户
2：查询用户
3：删除用户
4：批量验证手机号
5：退出程序
请输入操作指令（1-5）：1
请输入姓名：张三
请输入年龄：20
请输入手机号：13812345678
用户添加成功。

===== 用户信息管理工具 =====")
1：添加用户
2：查询用户
3：删除用户
4：批量验证手机号
5：退出程序
请输入操作指令（1-5）：2
请输入要查询的姓名：张三

===== 张三的用户信息 =====
姓名：张三
年龄：20岁
手机号：13812345678
```


# 七、总结
本节系统讲解了Python函数的核心知识，涵盖内置函数与自定义函数的全场景应用：
1. 函数的核心价值：代码复用、模块化开发、易维护性，是编程从“零散执行”到“结构化开发”的关键；
2. 内置函数：Python自带的基础工具集，覆盖日常开发80%的基础需求，优先使用可避免重复造轮子；
3. 自定义函数：
   - 定义语法：`def 函数名(参数): 函数体 return 返回值`；
   - 参数类型：位置参数、关键字参数、默认参数、可变参数（`*args`/`**kwargs`）；
   - 返回值：支持无返回值、单个返回值、多个返回值，`return`执行后终止函数；
4. 作用域：局部变量（函数内生效）、全局变量（函数外定义，修改需`global`声明）；
5. 常见问题：参数匹配错误、返回值遗漏、全局变量修改未声明、默认参数使用可变类型等。

函数是Python编程的核心基础，后续的模块、类、框架开发均依赖函数实现。建议将之前编写的逻辑封装为函数，体会模块化编程的优势，为后续复杂项目开发打下基础。

下一节，咱们会学习**字符串核心进阶：格式化方法（%+format+f-string）**


# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后，专栏内所有文章可永久免费查看，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/157651759>
