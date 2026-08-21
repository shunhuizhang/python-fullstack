

# Python全栈入门到实战【模块篇 01】核心模块random：吃透随机数生成逻辑，掌握全栈开发中的随机场景应用
前面我们已经完整掌握了Python**基础语法、并发编程、网络编程**三大核心体系，正式进入Python**核心内置模块实战篇**——这是提升全栈开发效率的关键，无需安装第三方库，Python内置的核心模块就能覆盖90%的日常开发需求。

而`random`模块，是全栈开发中**使用频率最高的内置模块之一**，从游戏开发的随机抽卡、抽奖，到数据处理的随机采样、打乱数据集，再到Web开发的验证码生成、测试数据生成，都离不开它。本节课作为模块篇的开篇，专门讲透`random`模块的核心知识，**从伪随机数的底层逻辑到全栈开发的高频实战**，摒弃繁琐的数学原理，只讲开发中**必须掌握的核心函数**，同时搭配**带超详细注释的可复用代码+文本讲解**，帮你彻底吃透`random`模块，将其灵活应用到全栈开发的各个场景中。
本节核心学习内容：
- 伪随机数：开发视角的底层逻辑，理解为什么Python的随机数是“伪随机”
- random模块的核心配置：随机种子的设置与作用，掌握可复现的随机数生成
- 高频基础函数：生成整数、浮点数、布尔值的核心方法，覆盖基础随机需求
- 高频序列函数：随机选择、随机采样、随机打乱序列的核心方法，覆盖数据处理、游戏开发场景
- 全栈实战1：Web开发的验证码生成（4位数字+字母混合验证码）
- 全栈实战2：数据处理的随机采样与数据集打乱（机器学习/数据分析必备）
- 全栈实战3：游戏开发的随机抽卡与抽奖（带概率控制的进阶用法）
- 新手必避的5个random模块坑点：随机种子误用、序列修改、概率计算错误等避坑方案
- 核心总结：random模块的高频函数速查表，方便开发时快速查阅

# 一、伪随机数：开发视角的底层逻辑
### 1.1 核心定义
Python的`random`模块生成的是**伪随机数**，而非真正的物理随机数——它是基于**随机种子（Seed）**和**确定性算法**生成的，只要随机种子相同，生成的随机数序列就完全一致。

简单来说：**伪随机数是“看起来随机，但实际上可复现”的数字序列**，这既是它的“缺点”（不够“真随机”），也是它的“优点”（可复现测试结果、调试代码）。

### 1.2 开发视角的意义
1. **可复现性**：在调试代码、机器学习训练模型时，设置固定的随机种子，能保证每次运行的结果完全一致，方便排查问题和对比实验；
2. **安全性**：`random`模块的伪随机数**不适合用于加密场景**（如密码生成、加密密钥），加密场景需使用`secrets`模块（后续模块篇会讲）；
3. **日常开发足够**：对于游戏开发、数据处理、Web验证码等日常开发场景，`random`模块的伪随机数已经完全够用，无需追求物理随机数。

# 二、random模块的核心配置：随机种子的设置与作用
在使用`random`模块的核心函数之前，先掌握**随机种子的设置**，这是实现可复现随机数的核心，也是新手最容易忽略的配置。

### 2.1 随机种子的设置函数
```python
import random  # 导入Python内置的random模块，无需安装第三方库

# 设置随机种子：seed()函数的参数可以是任意整数、浮点数、字符串（会被哈希为整数）
# 常用参数：整数（如0、1、12345），方便记忆和复现
random.seed(12345)
```

### 2.2 随机种子的作用演示（带超详细注释）
```python
import random

# 演示1：不设置随机种子，每次运行的结果不同
print("【演示1：不设置随机种子】")
for i in range(3):
    print(random.randint(1, 10))  # 生成1-10之间的随机整数
print("-" * 50)

# 演示2：设置相同的随机种子，每次运行的结果完全一致
print("【演示2：设置相同的随机种子12345】")
random.seed(12345)  # 第一次设置种子
for i in range(3):
    print(random.randint(1, 10))
print("-" * 50)

print("【演示3：再次设置相同的随机种子12345】")
random.seed(12345)  # 第二次设置相同的种子
for i in range(3):
    print(random.randint(1, 10))
print("-" * 50)

# 演示3的运行结果和演示2完全一致，验证了可复现性
```

**运行结果：**

```
【演示1：不设置随机种子】
1
7
6
--------------------------------------------------
【演示2：设置相同的随机种子12345】
7
1
5
--------------------------------------------------
【演示3：再次设置相同的随机种子12345】
7
1
5
--------------------------------------------------
```

### 2.3 开发中的使用场景

1. **调试代码**：设置固定的随机种子，保证每次运行的随机数序列一致，方便排查问题；
2. **机器学习训练**：设置固定的随机种子，保证数据集的划分、模型的初始化一致，方便对比不同模型的性能；
3. **游戏开发测试**：设置固定的随机种子，保证每次测试的抽卡、抽奖结果一致，方便测试游戏的平衡性。

# 三、random模块的高频基础函数：生成整数、浮点数、布尔值
掌握随机种子的设置后，学习`random`模块的**高频基础函数**，覆盖基础的随机数生成需求，每一个函数都带超详细注释和使用演示。

### 3.1 生成指定范围内的整数：randint(a, b)
```python
import random

# 生成a到b之间的**闭区间**随机整数（包括a和b）
# 常用场景：生成验证码的数字部分、游戏的随机等级、测试数据的ID
print("【randint(a, b)：生成闭区间随机整数】")
print(random.randint(1, 10))   # 生成1-10之间的随机整数
print(random.randint(100, 200)) # 生成100-200之间的随机整数
print(random.randint(-5, 5))    # 生成-5到5之间的随机整数
print("-" * 50)
```

### 3.2 生成指定范围内的浮点数：uniform(a, b)
```python
import random

# 生成a到b之间的**闭区间**随机浮点数（包括a和b）
# 常用场景：生成测试数据的浮点数、游戏的随机伤害值、数据处理的随机权重
print("【uniform(a, b)：生成闭区间随机浮点数】")
print(random.uniform(0, 1))    # 生成0-1之间的随机浮点数（最常用）
print(random.uniform(1.5, 3.5)) # 生成1.5-3.5之间的随机浮点数
print(random.uniform(-2.0, 2.0)) # 生成-2.0到2.0之间的随机浮点数
print("-" * 50)
```

### 3.3 生成0到1之间的随机浮点数：random()
```python
import random

# 生成0到1之间的**左闭右开区间**随机浮点数（包括0，不包括1）
# 等价于random.uniform(0, 1)，但更简洁，是uniform的底层实现
# 常用场景：生成概率值、作为其他随机函数的基础
print("【random()：生成0-1左闭右开随机浮点数】")
print(random.random())
print(random.random())
print("-" * 50)
```

### 3.4 生成指定步长的整数：randrange(start, stop[, step])
```python
import random

# 生成start到stop之间的**左闭右开区间**、指定步长的随机整数
# 等价于random.choice(range(start, stop, step))，但更高效
# 常用场景：生成偶数、奇数、指定间隔的测试数据
print("【randrange(start, stop[, step])：生成指定步长的随机整数】")
print(random.randrange(0, 10))       # 等价于randint(0, 9)
print(random.randrange(0, 10, 2))    # 生成0-9之间的随机偶数
print(random.randrange(1, 10, 2))    # 生成1-9之间的随机奇数
print(random.randrange(10, 100, 10)) # 生成10-90之间的随机整十数
print("-" * 50)
```

### 3.5 生成随机布尔值：choice([True, False])
```python
import random

# random模块没有直接的布尔值生成函数，用choice([True, False])实现
# 常用场景：生成测试数据的布尔值、游戏的随机事件触发
print("【choice([True, False])：生成随机布尔值】")
print(random.choice([True, False]))
print(random.choice([True, False]))
print("-" * 50)
```

# 四、random模块的高频序列函数：随机选择、随机采样、随机打乱
掌握基础随机数生成后，学习`random`模块的**高频序列函数**，这是全栈开发中使用频率最高的部分，覆盖数据处理、游戏开发、Web开发等场景，每一个函数都带超详细注释和使用演示。

### 4.1 从序列中随机选择一个元素：choice(seq)
```python
import random

# 从非空序列（列表、元组、字符串等）中随机选择一个元素
# 常用场景：生成验证码的字母部分、游戏的随机道具、随机选择测试数据
print("【choice(seq)：从序列中随机选择一个元素】")
# 从列表中选择
fruits = ["苹果", "香蕉", "橙子", "葡萄", "西瓜"]
print(random.choice(fruits))
# 从元组中选择
colors = ("红色", "绿色", "蓝色", "黄色")
print(random.choice(colors))
# 从字符串中选择（选择单个字符）
letters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
print(random.choice(letters))
print("-" * 50)
```

### 4.2 从序列中随机选择多个不重复的元素：sample(seq, k)
```python
import random

# 从非空序列中随机选择k个**不重复**的元素，返回一个新的列表
# 常用场景：数据处理的随机采样、机器学习的验证集划分、抽奖的中奖名单
print("【sample(seq, k)：从序列中随机选择k个不重复的元素】")
# 从列表中采样3个不重复的水果
fruits = ["苹果", "香蕉", "橙子", "葡萄", "西瓜", "草莓", "蓝莓", "芒果"]
print(random.sample(fruits, 3))
# 从字符串中采样4个不重复的字符（生成验证码的字母部分）
letters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
print(random.sample(letters, 4))
# 注意：k不能超过序列的长度，否则会抛出ValueError
# print(random.sample(fruits, 10))  # 会报错，因为fruits只有8个元素
print("-" * 50)
```

### 4.3 从序列中随机选择多个可重复的元素：choices(seq, k=1, weights=None, cum_weights=None)
```python
import random

# 从非空序列中随机选择k个**可重复**的元素，返回一个新的列表
# 支持设置权重（weights）或累积权重（cum_weights），实现带概率的随机选择
# 常用场景：游戏的随机抽卡、抽奖（带概率控制）、生成带权重的测试数据
print("【choices(seq, k=1, weights=None)：从序列中随机选择k个可重复的元素（支持权重）】")
# 不带权重的可重复选择
fruits = ["苹果", "香蕉", "橙子"]
print(random.choices(fruits, k=5))  # 选择5个可重复的水果
print("-" * 30)

# 带权重的可重复选择（核心！游戏抽卡/抽奖必备）
# weights参数：一个和seq长度相同的列表，表示每个元素的权重（权重越大，被选中的概率越高）
# 概率计算：元素的概率 = 元素的权重 / 所有元素的权重之和
print("【带权重的可重复选择：游戏抽卡】")
gacha_items = ["SSR", "SR", "R", "N"]
gacha_weights = [1, 10, 30, 59]  # SSR概率1%，SR10%，R30%，N59%
# 抽10次卡
results = random.choices(gacha_items, k=10, weights=gacha_weights)
print("抽卡结果：", results)
# 统计抽卡结果
print("SSR数量：", results.count("SSR"))
print("SR数量：", results.count("SR"))
print("R数量：", results.count("R"))
print("N数量：", results.count("N"))
print("-" * 50)
```

### 4.4 随机打乱序列的顺序：shuffle(seq)
```python
import random

# 随机打乱**可变序列**（列表）的顺序，直接修改原序列，不返回新序列
# 常用场景：数据处理的数据集打乱、机器学习的训练集打乱、游戏的卡牌洗牌
print("【shuffle(seq)：随机打乱可变序列的顺序】")
# 打乱列表的顺序
fruits = ["苹果", "香蕉", "橙子", "葡萄", "西瓜"]
print("打乱前：", fruits)
random.shuffle(fruits)
print("打乱后：", fruits)
# 注意：shuffle只能打乱可变序列（列表），不能打乱不可变序列（元组、字符串）
# 若要打乱不可变序列，需先转为列表，打乱后再转回原类型
print("-" * 30)

# 打乱不可变序列（元组）的示例
colors_tuple = ("红色", "绿色", "蓝色", "黄色")
print("打乱前的元组：", colors_tuple)
# 先转为列表
colors_list = list(colors_tuple)
# 打乱列表
random.shuffle(colors_list)
# 再转回元组
colors_tuple_shuffled = tuple(colors_list)
print("打乱后的元组：", colors_tuple_shuffled)
print("-" * 50)
```

# 五、全栈实战：random模块的高频应用场景
掌握`random`模块的核心函数后，学习**3个全栈开发的高频实战场景**，每一个场景都带超详细注释的可复用代码，可直接应用到实际项目中。

## 5.1 全栈实战1：Web开发的验证码生成（4位数字+字母混合验证码）
### 文本讲解：实战思路
1. 定义验证码的字符集：数字（0-9）+ 大小写字母（a-z, A-Z）；
2. 使用`random.sample()`从字符集中随机选择4个不重复的字符；
3. 将选择的字符拼接成字符串，生成验证码；
4. （可选）设置随机种子，方便测试；
5. （可选）使用`PIL`库生成验证码图片（后续Web开发篇会讲），本次实战先生成纯文本验证码。

### 实战代码（带超详细注释）
```python
import random

def generate_verification_code(length=4, use_seed=False, seed=12345):
    """
    生成指定长度的数字+字母混合验证码
    :param length: 验证码的长度，默认4位
    :param use_seed: 是否使用随机种子，默认False（生产环境设为False，测试环境设为True）
    :param seed: 随机种子，默认12345
    :return: 生成的验证码字符串
    """
    # 1. 定义验证码的字符集：数字+大小写字母
    # 数字：0-9
    digits = "0123456789"
    # 小写字母：a-z
    lowercase_letters = "abcdefghijklmnopqrstuvwxyz"
    # 大写字母：A-Z
    uppercase_letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    # 拼接成完整的字符集
    char_set = digits + lowercase_letters + uppercase_letters

    # 2. 设置随机种子（可选）
    if use_seed:
        random.seed(seed)

    # 3. 从字符集中随机选择length个不重复的字符
    # random.sample()返回的是列表，需要用join()拼接成字符串
    code_list = random.sample(char_set, length)
    verification_code = "".join(code_list)

    # 4. 返回生成的验证码
    return verification_code

# 测试生成验证码
if __name__ == "__main__":
    # 测试1：不使用随机种子，每次生成的验证码不同
    print("【测试1：不使用随机种子】")
    for i in range(3):
        print(f"第{i+1}次生成的验证码：{generate_verification_code()}")
    print("-" * 50)

    # 测试2：使用随机种子，每次生成的验证码完全一致
    print("【测试2：使用随机种子12345】")
    for i in range(3):
        print(f"第{i+1}次生成的验证码：{generate_verification_code(use_seed=True)}")
    print("-" * 50)

    # 测试3：生成6位验证码
    print("【测试3：生成6位验证码】")
    print(f"6位验证码：{generate_verification_code(length=6)}")
```

## 5.2 全栈实战2：数据处理的随机采样与数据集打乱（机器学习/数据分析必备）
### 文本讲解：实战思路
1. 生成一个模拟的数据集（列表，包含100个样本）；
2. 使用`random.shuffle()`打乱数据集的顺序；
3. 使用`random.sample()`从打乱后的数据集中随机采样20个样本作为验证集；
4. 剩下的80个样本作为训练集；
5. 设置固定的随机种子，保证数据集的划分可复现。

### 实战代码（带超详细注释）
```python
import random

def split_dataset(dataset, train_ratio=0.8, seed=12345):
    """
    随机打乱数据集，并按指定比例划分为训练集和验证集
    :param dataset: 原始数据集（列表）
    :param train_ratio: 训练集的比例，默认0.8（80%）
    :param seed: 随机种子，默认12345
    :return: 训练集列表、验证集列表
    """
    # 1. 设置固定的随机种子，保证数据集的划分可复现
    random.seed(seed)

    # 2. 随机打乱数据集的顺序（直接修改原数据集，若不想修改原数据集，可先复制一份）
    # 复制原数据集：dataset_shuffled = dataset.copy()
    dataset_shuffled = dataset.copy()
    random.shuffle(dataset_shuffled)

    # 3. 计算训练集的大小
    train_size = int(len(dataset_shuffled) * train_ratio)

    # 4. 划分训练集和验证集
    # 训练集：前train_size个样本
    train_dataset = dataset_shuffled[:train_size]
    # 验证集：后len(dataset_shuffled)-train_size个样本
    val_dataset = dataset_shuffled[train_size:]

    # 5. 返回训练集和验证集
    return train_dataset, val_dataset

# 测试数据集划分
if __name__ == "__main__":
    # 1. 生成一个模拟的数据集（100个样本，每个样本是一个ID：sample_0到sample_99）
    original_dataset = [f"sample_{i}" for i in range(100)]
    print("【原始数据集】")
    print(f"原始数据集大小：{len(original_dataset)}")
    print(f"前5个样本：{original_dataset[:5]}")
    print("-" * 50)

    # 2. 划分训练集和验证集（80%训练，20%验证）
    train_dataset, val_dataset = split_dataset(original_dataset, train_ratio=0.8)
    print("【划分后的数据集】")
    print(f"训练集大小：{len(train_dataset)}")
    print(f"训练集前5个样本：{train_dataset[:5]}")
    print(f"验证集大小：{len(val_dataset)}")
    print(f"验证集前5个样本：{val_dataset[:5]}")
    print("-" * 50)

    # 3. 再次划分，验证可复现性
    print("【再次划分，验证可复现性】")
    train_dataset_2, val_dataset_2 = split_dataset(original_dataset, train_ratio=0.8)
    print(f"训练集前5个样本是否一致：{train_dataset[:5] == train_dataset_2[:5]}")
    print(f"验证集前5个样本是否一致：{val_dataset[:5] == val_dataset_2[:5]}")
```

## 5.3 全栈实战3：游戏开发的随机抽卡与抽奖（带概率控制的进阶用法）
### 文本讲解：实战思路
1. 定义抽卡/抽奖的物品列表和对应的权重列表；
2. 使用`random.choices()`带权重的可重复选择，实现抽卡/抽奖；
3. 封装成一个通用的抽卡/抽奖函数，支持自定义物品、权重、抽卡次数；
4. 统计抽卡/抽奖的结果，验证概率是否符合预期；
5. 设置固定的随机种子，方便测试游戏的平衡性。

### 实战代码（带超详细注释）
```python
import random
from collections import Counter  # 导入Counter，用于统计抽卡结果

def gacha_or_lottery(items, weights, times=1, seed=None):
    """
    通用的抽卡/抽奖函数，支持带概率控制
    :param items: 抽卡/抽奖的物品列表（如["SSR", "SR", "R", "N"]）
    :param weights: 物品对应的权重列表（长度必须和items一致，权重越大，概率越高）
    :param times: 抽卡/抽奖的次数，默认1次
    :param seed: 随机种子，默认None（生产环境设为None，测试环境设为固定值）
    :return: 抽卡/抽奖的结果列表
    """
    # 1. 检查参数合法性：items和weights的长度必须一致
    if len(items) != len(weights):
        raise ValueError("物品列表和权重列表的长度必须一致！")
    # 检查参数合法性：权重必须都是非负数
    for w in weights:
        if w < 0:
            raise ValueError("权重必须是非负数！")
    # 检查参数合法性：抽卡次数必须是正整数
    if not isinstance(times, int) or times <= 0:
        raise ValueError("抽卡次数必须是正整数！")

    # 2. 设置随机种子（可选）
    if seed is not None:
        random.seed(seed)

    # 3. 带权重的可重复选择，实现抽卡/抽奖
    results = random.choices(items, k=times, weights=weights)

    # 4. 返回抽卡/抽奖的结果列表
    return results

# 测试抽卡/抽奖函数
if __name__ == "__main__":
    # 1. 定义抽卡的物品和权重
    gacha_items = ["SSR角色", "SR角色", "R角色", "N角色"]
    gacha_weights = [2, 15, 40, 43]  # SSR概率2%，SR15%，R40%，N43%
    # 计算每个物品的理论概率
    total_weight = sum(gacha_weights)
    print("【抽卡物品与理论概率】")
    for item, weight in zip(gacha_items, gacha_weights):
        probability = (weight / total_weight) * 100
        print(f"{item}：权重{weight}，理论概率{probability:.2f}%")
    print("-" * 50)

    # 2. 测试单抽
    print("【测试单抽】")
    single_result = gacha_or_lottery(gacha_items, gacha_weights, times=1)
    print(f"单抽结果：{single_result[0]}")
    print("-" * 50)

    # 3. 测试十连抽（10次）
    print("【测试十连抽】")
    ten_results = gacha_or_lottery(gacha_items, gacha_weights, times=10)
    print(f"十连抽结果：{ten_results}")
    # 统计十连抽结果
    ten_counter = Counter(ten_results)
    print("十连抽统计：")
    for item in gacha_items:
        print(f"{item}：{ten_counter.get(item, 0)}次")
    print("-" * 50)

    # 4. 测试万连抽（10000次），验证实际概率是否接近理论概率
    print("【测试万连抽，验证实际概率】")
    ten_thousand_results = gacha_or_lottery(gacha_items, gacha_weights, times=10000)
    # 统计万连抽结果
    ten_thousand_counter = Counter(ten_thousand_results)
    print("万连抽统计与实际概率：")
    for item in gacha_items:
        count = ten_thousand_counter.get(item, 0)
        actual_probability = (count / 10000) * 100
        print(f"{item}：{count}次，实际概率{actual_probability:.2f}%")
    print("-" * 50)

    # 5. 测试固定随机种子，验证可复现性
    print("【测试固定随机种子12345，验证可复现性】")
    results_1 = gacha_or_lottery(gacha_items, gacha_weights, times=10, seed=12345)
    results_2 = gacha_or_lottery(gacha_items, gacha_weights, times=10, seed=12345)
    print(f"第一次十连抽结果：{results_1}")
    print(f"第二次十连抽结果：{results_2}")
    print(f"两次结果是否一致：{results_1 == results_2}")
```

# 六、必避的5个random模块坑点（重点，避免踩雷）
新手在使用`random`模块时，80%的错误都集中在以下5个坑点，附详细避坑方案，看完直接绕开99%的问题：

### 坑1：random模块的伪随机数用于加密场景
**问题**：用`random`模块生成密码、加密密钥等加密场景的随机数，存在安全隐患（伪随机数可被预测）；
**避坑**：
1. 加密场景**必须使用`secrets`模块**（后续模块篇会讲），它生成的是**加密安全的伪随机数**；
2. 日常开发场景（游戏、数据处理、验证码）可以使用`random`模块。

### 坑2：随机种子的位置设置错误，导致无法复现
**问题**：在循环内部设置随机种子，导致每次循环的随机数都一样，无法复现整个序列；
**避坑**：
1. 随机种子**必须在循环外部、使用random模块的核心函数之前设置**；
2. 一个程序中**只需要设置一次随机种子**即可。

### 坑3：shuffle()函数修改原序列，导致原序列丢失
**问题**：直接对原序列使用`shuffle()`函数，原序列的顺序被打乱，无法恢复；
**避坑**：
1. 若不想修改原序列，**先复制一份原序列**，再对复制后的序列使用`shuffle()`函数；
2. 复制列表的方法：`new_list = old_list.copy()`或`new_list = list(old_list)`。

### 坑4：sample()函数的k值超过序列的长度，导致ValueError
**问题**：使用`sample()`函数时，k值（采样数量）超过了序列的长度，抛出`ValueError: Sample larger than population or is negative`；
**避坑**：
1. 使用`sample()`函数前，**先检查k值是否小于等于序列的长度**；
2. 若需要采样的数量超过序列的长度，**使用`choices()`函数**（可重复采样）。

### 坑5：choices()函数的权重设置错误，导致概率计算错误
**问题**：使用`choices()`函数时，权重列表的长度和物品列表的长度不一致，或权重为负数，导致概率计算错误或抛出异常；
**避坑**：
1. 使用`choices()`函数前，**先检查权重列表的长度是否和物品列表的长度一致**；
2. **检查所有权重是否都是非负数**；
3. 可以在函数内部添加参数合法性检查（如实战3中的`gacha_or_lottery()`函数）。

# 七、核心总结：random模块的高频函数速查表
为了方便开发时快速查阅，整理了`random`模块的**高频函数速查表**，涵盖所有日常开发场景：
| 函数名                            | 功能描述                                              | 常用场景                                 | 注意事项                                                    |
| --------------------------------- | ----------------------------------------------------- | ---------------------------------------- | ----------------------------------------------------------- |
| `seed(a=None)`                    | 设置随机种子                                          | 调试代码、机器学习训练、游戏测试         | 一个程序只需要设置一次，在循环外部设置                      |
| `randint(a, b)`                   | 生成a到b之间的闭区间随机整数                          | 验证码数字、游戏等级、测试数据ID         | 包括a和b                                                    |
| `randrange(start, stop[, step])`  | 生成start到stop之间的左闭右开区间、指定步长的随机整数 | 偶数、奇数、指定间隔的测试数据           | 等价于`choice(range(start, stop, step))`，但更高效          |
| `random()`                        | 生成0到1之间的左闭右开随机浮点数                      | 概率值、其他随机函数的基础               | 等价于`uniform(0, 1)`，但更简洁                             |
| `uniform(a, b)`                   | 生成a到b之间的闭区间随机浮点数                        | 测试数据浮点数、游戏伤害值、数据处理权重 | 包括a和b                                                    |
| `choice(seq)`                     | 从非空序列中随机选择一个元素                          | 验证码字母、游戏道具、随机测试数据       | seq必须是非空的                                             |
| `sample(seq, k)`                  | 从非空序列中随机选择k个不重复的元素                   | 数据处理采样、验证集划分、中奖名单       | k必须小于等于seq的长度                                      |
| `choices(seq, k=1, weights=None)` | 从非空序列中随机选择k个可重复的元素，支持权重         | 游戏抽卡、抽奖、带权重的测试数据         | weights的长度必须和seq一致，权重必须是非负数                |
| `shuffle(seq)`                    | 随机打乱可变序列的顺序，直接修改原序列                | 数据集打乱、训练集打乱、卡牌洗牌         | seq必须是可变序列（列表），若要打乱不可变序列，需先转为列表 |

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置模块、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布

