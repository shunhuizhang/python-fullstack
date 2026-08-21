
# Python全栈入门到实战【JavaScript篇 04】流程控制：条件判断与循环，从Python到JS的全面对照
上一篇《JavaScript篇 03》中，我们深入学习了JS的运算符体系，尤其是`===`和`==`的区别、隐式类型转换的坑、以及真值假值判断。这些知识是通往下一篇流程控制的关键基础——因为条件判断和循环的本质，就是在用运算符计算出一个布尔值，然后根据这个值决定代码走哪条分支。

本篇作为JavaScript篇的第四篇，我们将系统学习**JS的流程控制语句：条件判断和循环**。好消息是：JS的流程控制语法和Python高度相似——同样的`if-else`逻辑、同样的`for`/`while`循环，只是语法符号不同（花括号代替冒号+缩进）。作为Python开发者，你不需要重新理解"条件判断是什么"或"循环是什么"，只需要把Python的语法形式转换成JS的语法形式。本文重点列出两者语法上的精确对照，同时标注JS独有的特征（如`for...of`循环、`do...while`循环等）。

本节核心学习内容：
1.  if-else if-else：JS的条件判断与Python完整对照
2.  switch-case：多分支判断（Python 3.10 match-case的前身）
3.  for循环：三种写法详解（经典for、for...in、for...of）
4.  for...of vs Python的for...in：最容易混淆的两个语法
5.  while / do...while循环
6.  break / continue：循环控制
7.  条件判断和循环：JS vs Python代码对照速查

# 一、条件判断：if-else if-else
## 1.1 基本语法
JS的条件判断语法和Python完全一致，只是用括号和花括号代替了冒号和缩进：

```javascript
// JS 条件判断
if (条件) {
    // 条件为true时执行
} else if (条件2) {
    // 条件2为true时执行
} else {
    // 所有条件都不满足时执行
}
```

**Python对比**：
```python
# Python 条件判断
if 条件:
    # 条件为True时执行
elif 条件2:
    # 条件2为True时执行
else:
    # 所有条件都不满足时执行
```

**关键差异**：
| 特征 | JS | Python |
|------|-----|--------|
| 条件包裹 | 必须用小括号 `()` 包住 | 不需要括号（可选） |
| 代码块 | 花括号 `{}` | 冒号 `:` + 缩进 |
| else if | `else if`（两个词，空格分开） | `elif`（一个词） |
| 代码块结束 | `}` | 缩进回退 |

## 1.2 实战示例：成绩等级判断

```javascript
// JS 成绩等级判断
const score = 85;
let grade;

if (score >= 90) {
    grade = "A";
    console.log("优秀！");
} else if (score >= 80) {
    grade = "B";
    console.log("良好！");
} else if (score >= 70) {
    grade = "C";
    console.log("中等！");
} else if (score >= 60) {
    grade = "D";
    console.log("及格！");
} else {
    grade = "E";
    console.log("不及格！");
}

console.log(`成绩等级：${grade}`);
```

**Python对照**：
```python
# Python 成绩等级判断
score = 85

if score >= 90:
    grade = "A"
    print("优秀！")
elif score >= 80:
    grade = "B"
    print("良好！")
elif score >= 70:
    grade = "C"
    print("中等！")
elif score >= 60:
    grade = "D"
    print("及格！")
else:
    grade = "E"
    print("不及格！")

print(f"成绩等级：{grade}")
```

## 1.3 条件中的真值假值
条件小括号中的表达式会被自动转为布尔值。根据上一篇学的知识，JS有6个假值，其余都是真值：

```javascript
// 以下条件都是false
if (0)        { }  // false
if ("")       { }  // false
if (null)     { }  // false
if (undefined){ }  // false
if (NaN)      { }  // false

// 以下条件都是true
if (42)       { }  // true
if ("hello")  { }  // true
if ([])       { }  // true（⚠️ 空数组是真值，和Python不同！）
if ({})       { }  // true（⚠️ 空对象是真值）
```

## 1.4 单行if（无花括号的简洁写法）
当if或else只有一行代码时，可以省略花括号（但不推荐）：

```javascript
// 可行，但不推荐
if (score >= 60) console.log("及格");

// 推荐：始终使用花括号
if (score >= 60) {
    console.log("及格");
}
```

# 二、多分支判断：switch-case
## 2.1 基本语法
当需要根据一个变量的多个值做不同处理时，`switch-case`比多层`if-else`更清晰：

```javascript
const day = 3;
let dayName;

switch (day) {
    case 1:
        dayName = "星期一";
        break;
    case 2:
        dayName = "星期二";
        break;
    case 3:
        dayName = "星期三";
        break;
    case 4:
        dayName = "星期四";
        break;
    case 5:
        dayName = "星期五";
        break;
    case 6:
    case 7:
        dayName = "周末";
        break;
    default:
        dayName = "无效日期";
}

console.log(dayName); // "星期三"
```

**Python 3.10+ 对照**（match-case）：
```python
# Python 3.10+ match-case
day = 3

match day:
    case 1:
        day_name = "星期一"
    case 2:
        day_name = "星期二"
    case 3:
        day_name = "星期三"
    case 4:
        day_name = "星期四"
    case 5:
        day_name = "星期五"
    case 6 | 7:   # Python的|表示多个值
        day_name = "周末"
    case _:
        day_name = "无效日期"
```

## 2.2 break的重要性
JS的switch中，**每个case后面必须加`break`**，否则代码会"穿透"到下一个case继续执行：

```javascript
// 忘记break导致的穿透问题
const color = "red";

switch (color) {
    case "red":
        console.log("停止");
        // ⚠️ 忘记写break了！会继续执行下面的case
    case "yellow":
        console.log("减速");
        break;
    case "green":
        console.log("通行");
        break;
}
// 输出：
// "停止"
// "减速"  ← 这就是穿透，yellow也被执行了
```

**什么时候故意不用break**：当你希望多个case执行相同的代码时（如上面day的例子中`case 6:`和`case 7:`共享代码）。

## 2.3 switch-case vs if-else 使用建议
| 场景 | 推荐 | 原因 |
|------|------|------|
| 判断范围（>、<） | `if-else` | switch只能判断相等 |
| 判断一个变量的多个具体值 | `switch-case` | 更清晰，性能略好 |
| 只有2-3个分支 | `if-else` | switch反而显得冗余 |

# 三、for循环：JS的三种写法
JS有**三种for循环写法**，Python只有一种。这是两者差异最大的语法点之一。

## 3.1 经典for循环（C风格）
```javascript
// 经典for：初始化; 条件; 步进
for (let i = 0; i < 5; i++) {
    console.log(`第${i}次循环`);
}
// 输出：0, 1, 2, 3, 4
```

**Python使用range对照**：
```python
# Python for循环
for i in range(5):
    print(f"第{i}次循环")
# 输出：0, 1, 2, 3, 4
```

**经典for的三部分详解**：
```javascript
for (①初始化; ②条件; ③每次循环后执行) {
    // ④循环体
}

// 执行顺序：①(一次) → ②(判断) → ④(执行) → ③(步进) → ②(判断) → ...
```

常见写法：
```javascript
// 正向遍历
for (let i = 0; i < 10; i++) { }

// 反向遍历
for (let i = 9; i >= 0; i--) { }

// 步长为2
for (let i = 0; i < 10; i += 2) { }
```

## 3.2 for...in：遍历对象的键（不推荐用于数组）
`for...in`循环用于遍历对象的**可枚举属性**（键名）。注意，它是遍历"键/索引"，不是遍历"值"：

```javascript
// 遍历对象（正确用法）
const user = { name: "张三", age: 25, city: "北京" };
for (const key in user) {
    console.log(`${key}: ${user[key]}`);
}
// 输出：
// "name: 张三"
// "age: 25"
// "city: 北京"

// 遍历数组（不推荐，得到的是索引）
const colors = ["红", "绿", "蓝"];
for (const index in colors) {
    console.log(index, colors[index]);
}
// 输出：
// "0" "红"  ← index是字符串"0"，不是数字0
// "1" "绿"
// "2" "蓝"
```

> **⚠️ 重点**：遍历数组时不要用`for...in`，用`for...of`或`forEach`。

**Python对照**：
```python
# Python的for...in就是遍历值，不是遍历键
colors = ["红", "绿", "蓝"]
for color in colors:
    print(color)
# "红" "绿" "蓝"

# Python遍历字典键
user = {"name": "张三", "age": 25}
for key in user:
    print(key, user[key])
```

## 3.3 for...of：遍历可迭代对象的值（推荐用于数组）
`for...of`是ES6引入的，用于遍历**可迭代对象**（数组、字符串、Map、Set等）的**值**。这才是和Python的`for...in`对应的语法：

```javascript
// 遍历数组的值（推荐！和Python的for...in一样）
const colors = ["红", "绿", "蓝"];
for (const color of colors) {
    console.log(color);
}
// 输出："红" "绿" "蓝"

// 遍历字符串
const word = "Hello";
for (const char of word) {
    console.log(char);
}
// 输出：H e l l o
```

## 3.4 for...in vs for...of 对比总结
这是Python开发者最容易混淆的两个语法。下面这张表请牢记：

| 特性 | for...in | for...of |
|------|----------|----------|
| 遍历的是什么 | **键/索引** | **值** |
| 适用于 | 对象（Object） | 数组、字符串、Map、Set等可迭代对象 |
| JavaScfipt推荐度 | 仅用于遍历对象属性 | 遍历数组的首选方式 |
| 和Python的对应 | Python遍历dict键 `for key in dict` | Python遍历list值 `for item in list` ⭐ |

```javascript
// 对比
const arr = ["a", "b", "c"];

// for...in 返回索引（0, 1, 2）
for (const i in arr) { console.log(i); }

// for...of 返回值（"a", "b", "c"）
for (const v of arr) { console.log(v); }

// Python中 for x in arr 返回值
# for x in arr: print(x)  # "a", "b", "c"
# JS的for...of = Python的for...in
```

## 3.5 遍历数组的其他方式：forEach
JS的数组有一个内置的`forEach`方法，也是遍历数组的常用方式：

```javascript
const colors = ["红", "绿", "蓝"];

colors.forEach(function(item, index) {
    console.log(`${index}: ${item}`);
});
// 输出：
// "0: 红"
// "1: 绿"
// "2: 蓝"

// 使用箭头函数更简洁
colors.forEach((item, index) => {
    console.log(`${index}: ${item}`);
});
```

> `forEach`的用法将在后续"数组方法"文章中详细讲解。

# 四、while 和 do...while 循环
## 4.1 while循环
JS的`while`循环和Python完全一致：

```javascript
let count = 0;
while (count < 5) {
    console.log(count);
    count++;
}
// 0 1 2 3 4
```

**Python对比**：
```python
count = 0
while count < 5:
    print(count)
    count += 1
```

## 4.2 do...while循环（JS独有）
`do...while`保证循环体**至少执行一次**（先执行，后判断条件）：

```javascript
let count = 10;

// 即使条件一开始就不满足，循环体也会执行一次
do {
    console.log(count);  // 输出 10
    count++;
} while (count < 5);
// 只输出一次 10
```

**Python没有do...while**。Python中如果需要先执行再判断的模式，可以用while+break模拟：
```python
count = 10
while True:
    print(count)
    count += 1
    if count >= 5:
        break
```

# 五、break 和 continue
`break`和`continue`在JS和Python中行为完全一致：

```javascript
// break：终止整个循环
for (let i = 0; i < 10; i++) {
    if (i === 5) {
        break;   // 循环在这儿终止
    }
    console.log(i);
}
// 输出：0 1 2 3 4

// continue：跳过本次循环，进入下一次
for (let i = 0; i < 10; i++) {
    if (i % 2 === 0) {
        continue; // 跳过偶数
    }
    console.log(i);
}
// 输出：1 3 5 7 9
```

# 六、综合实战
## 6.1 打印九九乘法表
```javascript
// JS 九九乘法表
for (let i = 1; i <= 9; i++) {
    let row = "";
    for (let j = 1; j <= i; j++) {
        row += `${j}×${i}=${i * j}\t`;
    }
    console.log(row);
}
```

**Python对照**：
```python
# Python 九九乘法表
for i in range(1, 10):
    row = ""
    for j in range(1, i + 1):
        row += f"{j}×{i}={i * j}\t"
    print(row)
```

## 6.2 FizzBuzz经典面试题
输出1到100，遇到3的倍数输出"Fizz"，5的倍数输出"Buzz"，既是3又是5的倍数输出"FizzBuzz"。

```javascript
for (let i = 1; i <= 100; i++) {
    if (i % 3 === 0 && i % 5 === 0) {
        console.log("FizzBuzz");
    } else if (i % 3 === 0) {
        console.log("Fizz");
    } else if (i % 5 === 0) {
        console.log("Buzz");
    } else {
        console.log(i);
    }
}
```

# 七、常见误区与避坑指南
1.  **for...in遍历数组得到的是索引而非值**：这是Python开发者最常见的错误。Python中`for x in list`遍历的是值，但JS中`for (x in arr)`遍历的是索引（而且索引的类型是字符串）。**遍历数组请用`for...of`或`forEach`**。

2.  **忘记在switch的case后面加break**：每个case执行完后必须用`break;`跳出，否则会穿透到下一个case，导致意外执行。只有故意让多个case共享代码时才省略break。

3.  **条件中小括号缺失**：JS的if条件必须用小括号包裹`if (x > 0)`，这和Python不同（Python的`if x > 0:`不需要括号）。

4.  **循环中声明循环变量用var**：在经典for循环中`for (var i = 0; ...)`在函数作用域内i会泄露到循环外部。请使用`for (let i = 0; ...)`确保i只在循环块内有效。

5.  **用`==`在循环条件中做比较**：循环条件中的比较也遵守`===`原则。`while (x == 5)`应该写作`while (x === 5)`。

6.  **混淆else if和elif**：JS是`else if`（两个单词），Python是`elif`（一个单词）。在JS中写`elif`会报错。

# 八、核心总结：流程控制速查表
## 条件判断对照表
| 语句 | JS写法 | Python写法 |
|------|--------|-----------|
| if | `if (x > 0) { }` | `if x > 0:` |
| else if | `else if (x > 0) { }` | `elif x > 0:` |
| else | `else { }` | `else:` |
| 多分支 | `switch (x) { case 1: ...}` | `match x: case 1: ...`（3.10+） |
| 三元 | `x ? a : b` | `a if x else b` |

## 循环对照表
| 循环类型 | JS写法 | Python写法 |
|----------|--------|-----------|
| for遍历值 | `for (const x of arr) { }` | `for x in arr:` |
| for遍历索引 | `for (let i = 0; i < n; i++) { }` | `for i in range(n):` |
| for遍历对象键 | `for (const key in obj) { }` | `for key in dict:` |
| while | `while (x < 10) { }` | `while x < 10:` |
| do-while | `do { } while (x < 10);` | 无（用while True+break模拟） |
| 终止循环 | `break;` | `break` |
| 跳过本次 | `continue;` | `continue` |

## for...in vs for...of 快速判断
```
需要遍历数组的值？ → for...of
需要遍历对象的键？ → for...in
需要精确控制循环次数字？ → 经典for
简单遍历数组？ → .forEach()
```

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
