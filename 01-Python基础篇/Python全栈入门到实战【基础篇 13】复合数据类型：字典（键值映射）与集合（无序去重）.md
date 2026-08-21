# Python全栈入门到实战【基础篇 13】复合数据类型：字典（键值映射）与集合（无序去重）

# 前言

哈喽各位小伙伴！前面咱们学了字符串、数字这些基础类型，也掌握了列表这种有序序列——但实际开发中，仅靠这些还不够：
- 想存储“姓名-年龄-手机号”这种“键值对应”的用户信息，用列表只能按位置存（["张三", 20, "13812345678"]），查手机号要记索引，极不方便；
- 想快速给一堆重复数据去重（比如[1,2,2,3,3,3]），用列表要写循环判断，效率低；
- 想对比两个列表的“共同元素”“差异元素”，用列表遍历要写大量代码。

这些场景的最优解，就是Python的两大复合数据类型：**字典（dict）** 和 **集合（set）**——字典是“键值映射”的宝库，能通过“键”快速定位“值”，无需记忆位置；集合是“无序去重”的利器，能一键去重、高效做集合运算。

这节咱们用“原理+实操+场景+避坑”的方式，吃透字典和集合的核心用法：
- 字典：定义、增删改查、嵌套字典、常用方法（`get()`/`keys()`/`items()`）；
- 集合：定义、增删改查、去重与集合运算（交集/并集/差集）；
- 字典vs集合vs列表：适用场景对比，避免用错类型。

吃透这两个类型，你就能高效处理“键值关联数据”和“去重/集合对比”场景，代码简洁度和效率直接翻倍～

@[TOC](文章目录)


# 一、前置引入：为什么需要字典和集合？
前面学的列表（list）是“有序序列”，但有两个明显短板：
1. 查找元素依赖索引，无法通过“语义化标识”（比如“姓名”“手机号”）快速定位；
2. 允许重复元素，去重需要额外写代码；
3. 集合运算（比如找共同元素）效率低，时间复杂度是O(n²)。

而字典和集合正好弥补了这些短板：
- 字典：用“键（key）-值（value）”对应关系存储，查找元素时间复杂度O(1)（和索引查找一样快），比如通过`user["phone"]`直接获取手机号，无需记位置；
- 集合：自动去重，支持交集（&）、并集（|）、差集（-）等运算，时间复杂度O(1)，处理重复数据和集合对比时效率极高。

简单说：
- 存储“键值对应”数据（用户信息、配置参数、JSON数据）→ 用字典；
- 去重、集合对比（共同好友、商品分类交集）→ 用集合；
- 有序存储、按位置访问 → 用列表。


# 二、字典（dict）：键值映射的“万能容器”

![](D:\python全栈专栏\学习图片\dict思维导图.png)

字典是Python中最常用的复合类型之一，核心是“键值对（key-value pair）”，每个键唯一对应一个值，就像一本“字典”：“词语”（键）对应“解释”（值）。

## 1. 字典的定义：键值对的集合
### 语法
- 用`{}`包裹，键值对之间用`,`分隔；
- 键（key）：必须是**不可变类型**（字符串、数字、元组），且唯一（重复键会被覆盖）；
- 值（value）：可以是任意类型（字符串、数字、列表、字典等）。

### 示例：定义用户信息字典
```python
# 基础字典（字符串键）
user = {
    "name": "张三",
    "age": 20,
    "phone": "13812345678",
    "is_student": True
}

# 数字键（不常用，但合法）
score = {1: 90, 2: 85, 3: 95}

# 元组键（不可变类型，合法）
point = {(1, 2): "原点附近", (3, 4): "第一象限"}

# 错误：列表作为键（可变类型，不合法）
# invalid_dict = {[1,2]: "错误"}  # TypeError: unhashable type: 'list'

print(f"用户字典：{user}")
print(f"字典类型：{type(user)}")  # <class 'dict'>
```

### 空字典定义
```python
# 方法1：{}
empty_dict1 = {}
# 方法2：dict()
empty_dict2 = dict()
```

## 2. 字典的核心操作：增删改查（高频）
字典的操作围绕“键值对”展开，核心是“通过键操作值”，比列表的操作更高效。

### （1）查：获取键对应的值
#### 方法1：`dict[key]`（直接访问，键不存在抛异常）
```python
user = {"name": "张三", "age": 20, "phone": "13812345678"}
print(f"姓名：{user['name']}")  # 姓名：张三
print(f"年龄：{user['age']}")   # 年龄：20

# 键不存在时抛异常
try:
    print(user["address"])
except KeyError as e:
    print(f"错误：键{e}不存在")  # 错误：键'address'不存在
```

#### 方法2：`dict.get(key, default=None)`（推荐，键不存在返回默认值）
```python
user = {"name": "张三", "age": 20}
# 键存在，返回对应值
print(f"姓名：{user.get('name')}")  # 姓名：张三
# 键不存在，返回默认值"未知"
print(f"地址：{user.get('address', '未知')}")  # 地址：未知
# 键不存在，默认返回None
print(f"邮箱：{user.get('email')}")  # 邮箱：None
```

#### 方法3：`dict.keys()`/`dict.values()`/`dict.items()`（批量获取）
```python
user = {"name": "张三", "age": 20, "phone": "13812345678"}
# 获取所有键（返回可迭代对象）
keys = user.keys()
print(f"所有键：{list(keys)}")  # 所有键：['name', 'age', 'phone']

# 获取所有值
values = user.values()
print(f"所有值：{list(values)}")  # 所有值：['张三', 20, '13812345678']

# 获取所有键值对（返回元组迭代对象）
items = user.items()
print(f"所有键值对：{list(items)}")  # 所有键值对：[('name', '张三'), ('age', 20), ('phone', '13812345678')]

# 遍历所有键值对（最常用）
for key, value in user.items():
    print(f"{key}：{value}")
```

### （2）增：添加新键值对
#### 方法1：`dict[key] = value`（键不存在则新增）
```python
user = {"name": "张三", "age": 20}
# 新增键值对
user["address"] = "北京市"
user["email"] = "zhangsan@xxx.com"
print(f"添加后：{user}")  # {'name': '张三', 'age': 20, 'address': '北京市', 'email': 'zhangsan@xxx.com'}
```

#### 方法2：`dict.update(other_dict)`（批量添加/更新）
```python
user = {"name": "张三", "age": 20}
# 批量添加新键值对
user.update({"phone": "13812345678", "is_student": True})
print(f"update后：{user}")  # {'name': '张三', 'age': 20, 'phone': '13812345678', 'is_student': True}

# 同时更新已有键和添加新键
user.update({"age": 21, "address": "上海市"})
print(f"更新后：{user}")  # {'name': '张三', 'age': 21, 'phone': '13812345678', 'is_student': True, 'address': '上海市'}
```

### （3）改：修改已有键的值
#### 方法：`dict[key] = value`（键存在则修改）
```python
user = {"name": "张三", "age": 20}
# 修改age的值
user["age"] = 22
print(f"修改后：{user}")  # {'name': '张三', 'age': 22}
```

### （4）删：删除键值对
#### 方法1：`del dict[key]`（键不存在抛异常）
```python
user = {"name": "张三", "age": 20, "phone": "13812345678"}
# 删除phone键
del user["phone"]
print(f"删除后：{user}")  # {'name': '张三', 'age': 20}

# 删不存在的键抛异常
try:
    del user["address"]
except KeyError as e:
    print(f"错误：键{e}不存在")
```

#### 方法2：`dict.pop(key, default)`（推荐，键不存在返回默认值）
```python
user = {"name": "张三", "age": 20, "phone": "13812345678"}
# 删除phone键，返回对应值
phone = user.pop("phone")
print(f"删除的手机号：{phone}")  # 删除的手机号：13812345678
print(f"pop后：{user}")  # {'name': '张三', 'age': 20}

# 删除不存在的键，返回默认值
address = user.pop("address", "无地址")
print(f"地址：{address}")  # 地址：无地址
```

#### 方法3：`dict.clear()`（清空所有键值对）
```python
user = {"name": "张三", "age": 20}
user.clear()
print(f"清空后：{user}")  # {}
```

## 3. 字典的进阶用法：嵌套字典
字典的值可以是字典，形成“嵌套结构”，适合存储复杂数据（比如多级配置、用户的详细信息）。

### 示例：嵌套字典（用户的详细信息）
```python
# 嵌套字典：用户信息包含基础信息和成绩
user = {
    "basic": {
        "name": "张三",
        "age": 20,
        "phone": "13812345678"
    },
    "scores": {
        "math": 90,
        "english": 85,
        "python": 95
    }
}

# 访问嵌套字典的值（逐层通过键访问）
print(f"姓名：{user['basic']['name']}")  # 姓名：张三
print(f"Python成绩：{user['scores']['python']}")  # Python成绩：95

# 修改嵌套字典的值
user["scores"]["math"] = 92
print(f"修改后数学成绩：{user['scores']['math']}")  # 修改后数学成绩：92

# 遍历嵌套字典
for info_type, info_dict in user.items():
    print(f"\n{info_type}：")
    for key, value in info_dict.items():
        print(f"  {key}：{value}")
```

**运行结果**：
```
姓名：张三
Python成绩：95
修改后数学成绩：92

basic：
  name：张三
  age：20
  phone：13812345678

scores：
  math：92
  english：85
  python：95
```

## 4. 字典的核心避坑要点
### 坑1：键必须是不可变类型
- 错误：用列表、字典作为键（可变类型）；
- 正确：用字符串、数字、元组（不可变类型）作为键。

### 坑2：重复键会被覆盖
```python
# 重复键，后面的覆盖前面的
user = {"name": "张三", "name": "李四"}
print(user)  # {'name': '李四'}
```

### 坑3：字典是无序的？（Python 3.7+已有序）
- Python 3.7之前：字典的键值对顺序不固定；
- Python 3.7及以后：字典会保留插入顺序；
- 如需严格有序（如配置文件），可用`collections.OrderedDict`（但3.7+后基本无需使用）。

### 坑4：遍历字典时修改键会报错
```python
user = {"name": "张三", "age": 20, "phone": "13812345678"}
# 错误：遍历过程中添加新键，会触发RuntimeError
# for key in user.keys():
#     user[f"{key}_new"] = "新值"

# 正确：遍历字典的副本（用list()转成列表）
for key in list(user.keys()):
    user[f"{key}_new"] = "新值"
print(user)
```

# 三、集合（set）：无序去重的“高效工具”

![](D:\python全栈专栏\学习图片\set思维导图.png)

集合是“无序、不重复”的元素集合，核心功能是**去重**和**集合运算**，底层是哈希表，操作效率极高。

## 1. 集合的定义：无序不重复元素
### 语法
- 用`{}`包裹，元素之间用`,`分隔（注意：空集合不能用`{}`，要用`set()`）；
- 元素必须是**不可变类型**（字符串、数字、元组），且不重复（重复元素会自动去重）。

### 示例：定义集合
```python
# 基础集合（自动去重）
nums = {1, 2, 2, 3, 3, 3}
print(nums)  # {1, 2, 3}（重复元素被自动去除）

# 字符串集合
chars = {"a", "b", "c", "a"}
print(chars)  # {'a', 'b', 'c'}

# 元组元素集合（不可变类型，合法）
tuples_set = {(1,2), (3,4), (1,2)}
print(tuples_set)  # {(1, 2), (3, 4)}

# 错误：元素是列表（可变类型）
# invalid_set = {[1,2], [3,4]}  # TypeError: unhashable type: 'list'

# 空集合（必须用set()，不能用{}）
empty_set = set()
print(f"空集合类型：{type(empty_set)}")  # <class 'set'>
print(f"错误的空集合类型：{type({})}")  # <class 'dict'>（{}是空字典）
```

### 用`set()`转换其他类型为集合
```python
# 列表转集合（去重）
list_nums = [1, 2, 2, 3, 3, 3]
set_nums = set(list_nums)
print(f"列表转集合（去重）：{set_nums}")  # {1, 2, 3}

# 字符串转集合（拆分字符，去重）
str_chars = "abac"
set_chars = set(str_chars)
print(f"字符串转集合：{set_chars}")  # {'a', 'b', 'c'}

# 元组转集合
tuple_nums = (1, 2, 3, 3)
set_tuple = set(tuple_nums)
print(f"元组转集合：{set_tuple}")  # {1, 2, 3}
```

## 2. 集合的核心操作：增删改查
集合的操作围绕“元素”展开，核心是“添加、删除、判断元素是否存在”。

### （1）查：判断元素是否存在（`in`/`not in`）
```python
nums = {1, 2, 3, 4, 5}
# 判断元素是否在集合中
print(f"3是否在集合中：{3 in nums}")  # True
print(f"6是否在集合中：{6 in nums}")  # False
print(f"7是否不在集合中：{7 not in nums}")  # True
```

### （2）增：添加元素
#### 方法1：`set.add(element)`（添加单个元素）
```python
nums = {1, 2, 3}
# 添加单个元素
nums.add(4)
print(f"添加4后：{nums}")  # {1, 2, 3, 4}

# 添加重复元素（无效果，不报错）
nums.add(3)
print(f"添加重复元素3后：{nums}")  # {1, 2, 3, 4}
```

#### 方法2：`set.update(iterable)`（批量添加元素）
```python
nums = {1, 2, 3}
# 批量添加列表元素
nums.update([4, 5, 6])
print(f"update列表后：{nums}")  # {1, 2, 3, 4, 5, 6}

# 批量添加集合元素
nums.update({7, 8})
print(f"update集合后：{nums}")  # {1, 2, 3, 4, 5, 6, 7, 8}
```

### （3）删：删除元素
#### 方法1：`set.remove(element)`（元素不存在抛异常）
```python
nums = {1, 2, 3, 4}
# 删除元素3
nums.remove(3)
print(f"remove后：{nums}")  # {1, 2, 4}

# 删除不存在的元素抛异常
try:
    nums.remove(5)
except KeyError as e:
    print(f"错误：元素{e}不存在")
```

#### 方法2：`set.discard(element)`（推荐，元素不存在不报错）
```python
nums = {1, 2, 3, 4}
# 删除元素3
nums.discard(3)
print(f"discard后：{nums}")  # {1, 2, 4}

# 删除不存在的元素，无效果
nums.discard(5)
print(f"discard不存在元素后：{nums}")  # {1, 2, 4}
```

#### 方法3：`set.pop()`（随机删除一个元素，返回该元素）
```python
nums = {1, 2, 3, 4}
# 随机删除一个元素（无序，所以随机）
deleted = nums.pop()
print(f"删除的元素：{deleted}")
print(f"pop后：{nums}")
```

#### 方法4：`set.clear()`（清空集合）
```python
nums = {1, 2, 3}
nums.clear()
print(f"清空后：{nums}")  # set()
```

## 3. 集合的核心功能：集合运算（交集/并集/差集）
集合最强大的功能是“集合运算”，能快速找到两个集合的共同元素、合并元素、差异元素，无需写循环。

### 核心集合运算表（以`a={1,2,3,4}`，`b={3,4,5,6}`为例）
| 运算类型                       | 符号 | 方法                        | 示例              | 结果            |
| ------------------------------ | ---- | --------------------------- | ----------------- | --------------- |
| 交集（共同元素）               | `&`  | `a.intersection(b)`         | `a & b`           | `{3,4}`         |
| 并集（所有元素，去重）         | `|`  | `a.union(b)`                | `a | b`           | `{1,2,3,4,5,6}` |
| 差集（a有b没有的元素）         | `-`  | `a.difference(b)`           | `a - b`           | `{1,2}`         |
| 对称差集（a和b互不相同的元素） | `^`  | `a.symmetric_difference(b)` | `a ^ b`           | `{1,2,5,6}`     |
| 子集判断（a是否完全在b中）     | `<`  | `a.issubset(b)`             | `{1,2} < b`       | `False`         |
| 超集判断（b是否完全包含a）     | `>`  | `a.issuperset(b)`           | `{1,2,3,4,5} > a` | `True`          |

### 示例：集合运算实战
```python
# 定义两个集合（比如两个用户的好友列表）
user1_friends = {"张三", "李四", "王五", "赵六"}
user2_friends = {"王五", "赵六", "孙七", "周八"}

# 1. 交集：共同好友（两个集合都有的元素）
common_friends = user1_friends & user2_friends
print(f"共同好友：{common_friends}")  # {'王五', '赵六'}

# 2. 并集：所有好友（去重）
all_friends = user1_friends | user2_friends
print(f"所有好友：{all_friends}")  # {'张三', '李四', '王五', '赵六', '孙七', '周八'}

# 3. 差集：user1有但user2没有的好友
user1_only = user1_friends - user2_friends
print(f"user1独有的好友：{user1_only}")  # {'张三', '李四'}

# 4. 对称差集：互不相同的好友（仅一方有）
different_friends = user1_friends ^ user2_friends
print(f"仅一方有的好友：{different_friends}")  # {'张三', '李四', '孙七', '周八'}

# 5. 子集判断
is_subset = {"王五", "赵六"} < user1_friends
print(f"{'王五','赵六'}是user1好友的子集：{is_subset}")  # True
```

## 4. 集合的核心避坑要点
### 坑1：空集合不能用`{}`
- 错误：`empty_set = {}`（这是空字典）；
- 正确：`empty_set = set()`。

### 坑2：集合是无序的，不能用索引访问
```python
nums = {1, 2, 3}
# 错误：集合无索引
# print(nums[0])  # TypeError: 'set' object is not subscriptable

# 正确：如需按顺序访问，转成列表
print(list(nums)[0])  # 1（但顺序不固定）
```

### 坑3：元素必须是不可变类型
- 错误：`set([1,2])`（元素是列表，可变）；
- 正确：`set((1,2))`（元素是元组，不可变）。

### 坑4：集合运算返回新集合，不修改原集合
```python
a = {1,2,3}
b = {3,4,5}
c = a & b  # c是新集合{3}
print(a)  # {1,2,3}（原集合不变）
```


# 四、字典vs集合vs列表：怎么选？
| 特性         | 列表（list）                   | 字典（dict）                   | 集合（set）                  |
| ------------ | ------------------------------ | ------------------------------ | ---------------------------- |
| 存储方式     | 有序序列（索引）               | 无序键值对                     | 无序不重复元素               |
| 元素类型限制 | 无（可重复）                   | 键：不可变类型；值：无         | 不可变类型（不重复）         |
| 查找效率     | O(n)（慢）                     | O(1)（快）                     | O(1)（快）                   |
| 核心功能     | 有序存储、按位置访问           | 键值映射、快速查找             | 去重、集合运算               |
| 适用场景     | 批量存储有序数据（如任务列表） | 存储键值对应数据（如用户信息） | 去重、集合对比（如共同好友） |

### 选型口诀
- 有序、按位置 → 列表；
- 键值、快速查 → 字典；
- 去重、集合算 → 集合。


# 五、实战案例：用户信息管理系统（整合字典与集合）
需求：制作一个简单的用户信息管理系统，支持以下功能：
1. 添加用户（姓名、年龄、手机号，手机号唯一）；
2. 查询用户（通过姓名查询信息）；
3. 删除用户（通过姓名删除）；
4. 统计所有用户的手机号前缀（去重）。

```python
def user_management():
    # 用字典存储用户信息（姓名为键，值为字典）
    users = {}
    # 用集合存储手机号，确保唯一
    phone_set = set()

    while True:
        print("\n===== 用户信息管理系统 =====")
        print("1：添加用户")
        print("2：查询用户")
        print("3：删除用户")
        print("4：统计手机号前缀")
        print("5：退出程序")
        cmd = input("请输入操作指令（1-5）：")

        if cmd == "1":
            # 添加用户
            name = input("请输入姓名：")
            age = input("请输入年龄：")
            phone = input("请输入手机号：")

            # 验证手机号唯一
            if phone in phone_set:
                print("错误：该手机号已被使用！")
                continue

            # 验证年龄是数字
            if not age.isdigit():
                print("错误：年龄必须是数字！")
                continue

            # 添加到字典和集合
            users[name] = {"age": int(age), "phone": phone}
            phone_set.add(phone)
            print(f"用户{name}添加成功！")

        elif cmd == "2":
            # 查询用户
            name = input("请输入要查询的姓名：")
            user_info = users.get(name)
            if user_info:
                print(f"姓名：{name}")
                print(f"年龄：{user_info['age']}")
                print(f"手机号：{user_info['phone']}")
            else:
                print(f"未找到用户{name}！")

        elif cmd == "3":
            # 删除用户
            name = input("请输入要删除的姓名：")
            user_info = users.get(name)
            if user_info:
                # 从集合中删除手机号
                phone_set.remove(user_info["phone"])
                # 从字典中删除用户
                del users[name]
                print(f"用户{name}删除成功！")
            else:
                print(f"未找到用户{name}！")

        elif cmd == "4":
            # 统计手机号前缀（前3位）
            if not phone_set:
                print("暂无用户手机号！")
                continue
            # 提取前缀，用集合去重
            prefix_set = {phone[:3] for phone in phone_set}
            print(f"所有手机号前缀（去重）：{prefix_set}")

        elif cmd == "5":
            print("退出程序，感谢使用！")
            break

        else:
            print("未知指令，请输入1-5！")

# 运行系统
user_management()
```

**运行示例**：
```
===== 用户信息管理系统 =====
1：添加用户
2：查询用户
3：删除用户
4：统计手机号前缀
5：退出程序
请输入操作指令（1-5）：1
请输入姓名：张三
请输入年龄：20
请输入手机号：13812345678
用户张三添加成功！

===== 用户信息管理系统 =====
1：添加用户
2：查询用户
3：删除用户
4：统计手机号前缀
5：退出程序
请输入操作指令（1-5）：1
请输入姓名：李四
请输入年龄：22
请输入手机号：13987654321
用户李四添加成功！

===== 用户信息管理系统 =====
1：添加用户
2：查询用户
3：删除用户
4：统计手机号前缀
5：退出程序
请输入操作指令（1-5）：4
所有手机号前缀（去重）：{'138', '139'}
```


# 六、总结
今天咱们吃透了字典和集合这两大复合数据类型，核心要点梳理如下：
1. **字典（dict）**：
   - 核心：键值映射，键唯一且为不可变类型，值任意；
   - 操作：增（`[]`/`update`）、删（`del`/`pop`）、改（`[]`）、查（`[]`/`get`）；
   - 进阶：嵌套字典存储复杂数据；
   - 避坑：键不可变、重复键覆盖、遍历修改需用副本。

2. **集合（set）**：
   - 核心：无序不重复，元素为不可变类型；
   - 操作：增（`add`/`update`）、删（`remove`/`discard`）、查（`in`）；
   - 核心功能：去重、集合运算（交集/并集/差集）；
   - 避坑：空集合用`set()`、无索引、元素不可变。

3. **选型原则**：
   - 有序存储→列表；
   - 键值查找→字典；
   - 去重/集合对比→集合。

字典和集合是Python开发中“效率利器”，尤其是处理大量数据时，能显著简化代码、提升性能。建议你把实战案例扩展为“商品分类管理”“好友关系统计”等场景，加深对两个类型的理解～

下一节，咱们会学习**Python循环结构进阶**：除了基础的for/while循环，还会讲循环控制（break/continue）、嵌套循环、推导式，让程序能自动化完成重复任务～


# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后，专栏内所有文章可永久免费查看，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/157434261>
