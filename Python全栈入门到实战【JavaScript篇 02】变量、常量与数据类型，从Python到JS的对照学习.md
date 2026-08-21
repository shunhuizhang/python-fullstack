
# Python全栈入门到实战【JavaScript篇 02】变量、常量与数据类型，从Python到JS的对照学习
上一篇《JavaScript篇 01》中，我们认识了JavaScript的定位，学会了打开浏览器Console，写出了第一个JS程序。你可能已经注意到了JS和Python的第一个明显区别——在JS中，你不能像Python那样`name = "张三"`直接赋值，而必须先声明`let name = "张三"`。

本篇作为JavaScript篇的第二篇，我们将系统学习JS的**变量声明方式和数据类型系统**。你将发现JS在这两个方面和Python的差异是最多的——Python"赋值即声明"，而JS有三种声明关键字（let/const/var）各有不同规则；Python有`int/float/str/bool/list/dict`等丰富类型，而JS只有7种基本类型——这些差异看似繁琐，但理解了它们的设计逻辑后，它们反而会成为你写出更安全代码的工具。本文全程采用Python对比教学法，每学一个JS特性都会和Python对照，帮助你借助已有的Python知识快速建立JS的类型体系认知。

本节核心学习内容：
1.  JS变量声明的三种方式：let、const、var的区别与选型
2.  变量命名规范：驼峰命名 vs Python蛇形命名
3.  JS的7种基本数据类型详解
4.  typeof运算符：检查变量类型
5.  字符串模板：JS模板字符串 vs Python f-string
6.  undefined vs null：JS独有的"双空值"设计
7.  与Python数据类型的完整对照表
8.  常见误区与避坑指南

# 一、变量声明的三种方式
## 1.1 Python vs JavaScript：变量定义的哲学差异
在Python中，变量不需要声明，赋值即创建：

```python
# Python：赋值即声明
name = "张三"
age = 25
PI = 3.14159
```

这是Python的简洁哲学。但在JavaScript中，你必须使用声明关键字来告诉JS引擎：**"我要创建一个新变量"**。

```javascript
// JS：必须使用声明关键字
let name = "张三";
const age = 25;
var PI = 3.14159;  // 老旧写法
```

JS提供了三种声明变量的关键字：**`let`、`const`、`var`**。它们各自有不同的行为和适用场景，选错关键字可能导致难以排查的bug。

## 1.2 const：声明常量（首选）
`const`用于声明**常量**——一旦赋值，就不能再修改。这是三种方式中**使用频率最高**的。

**语法格式**：
```javascript
const PI = 3.14159;
const BASE_URL = "https://api.example.com";
const MAX_RETRY_COUNT = 3;

// 尝试修改const常量会报错
PI = 3.14;  // ❌ TypeError: Assignment to constant variable
```

**适用场景**：任何不应该被重新赋值的变量。在实际开发中，你应该**优先使用const**声明变量——只有当你确定变量值需要被修改时，才使用let。这能让代码更安全、更可预测。

**Python类比**：Python中没有真正的常量机制（全大写命名只是约定），JS的const提供了语言层面的保护。如果你在Python中用`MAX_SIZE = 100`只是"约定俗成"表示常量，在JS中用`const MAX_SIZE = 100`则是"真正不可修改"的常量。

## 1.3 let：声明可变变量
`let`用于声明**可以被重新赋值**的变量。它的行为最接近Python中的普通变量。

**语法格式**：
```javascript
let count = 0;
count = 1;    // ✓ 可以修改
count = 2;    // ✓ 可以修改

let name = "张三";
name = "李四";  // ✓ 可以修改
```

**适用场景**：计数器、循环变量、需要重新赋值的状态值。

**Python类比**：JS的`let` ≈ Python的普通变量，值可以被任意重新赋值。

## 1.4 var：老旧的关键字（不推荐）
`var`是ES6之前（2015年之前）唯一声明变量的方式。它有很多设计缺陷，已经被`let`和`const`取代。

```javascript
var name = "张三";  // 老旧写法，不推荐

// var的主要问题：
// 1. 可以重复声明同一个变量（let不允许）
var x = 10;
var x = 20;  // 不会报错，非常混乱

// 2. 没有块级作用域（let有）
if (true) {
    var y = 100;
}
console.log(y);  // 100，y泄露到了外部！如果用let则不会
```

> **原则**：永远不要在代码中使用`var`。所有现代JS项目都使用`let`和`const`。var出现在这里只是为了让你的"识别老代码"——如果你在看其他教程或老项目中看到var，你知道它是过时的写法。

## 1.5 let vs const 选择原则
一个简单的选择口诀：

```
能确定不会重新赋值？ → const
可能会重新赋值？    → let
var？              → 忘了它
```

**为什么优先const**：
- 代码意图更清晰："这个值从生到死都不会变"
- 防止意外修改：如果你试图改一个const变量，JS会立刻报错，帮你发现bug
- 符合函数式编程理念：不可变性让代码更容易推理和测试
- 主流框架（Vue/React）的最佳实践

```javascript
// 典型的实际代码片段
const userName = "张三";        // 用户名通常不会变
const apiUrl = "/api/users";   // API地址不会变
let page = 1;                  // 页码会随翻页变化
let isLoading = false;         // 加载状态会变化
```

# 二、变量命名规范
## 2.1 JS命名规则
JS变量命名的基本规则和Python相同：
- 只能包含字母、数字、下划线 `_` 和美元符号 `$`
- 不能以数字开头
- 区分大小写（`userName` 和 `username` 是两个不同的变量）
- 不能用关键字（`let`、`const`、`if`、`for`等）

## 2.2 命名风格：驼峰 vs 蛇形
这是Python和JS在代码风格上最直观的差异：

| 命名类型 | Python风格 | JS风格 | 示例（JS） |
|----------|-----------|--------|------------|
| 变量名 | `user_name` | `userName` | `let userName = "张三";` |
| 函数名 | `get_user_by_id` | `getUserById` | `function getUserById() {}` |
| 类名 | `UserProfile` | `UserProfile` | `class UserProfile {}` |
| 常量 | `MAX_SIZE` | `MAX_SIZE` | `const MAX_SIZE = 100;` |

> **建议**：在JS代码中统一使用驼峰命名法。虽然用蛇形命名不会被报错，但会让你的代码在JS社区中显得"不专业"，而且大多数JS工具链（ESLint等）默认检测驼峰命名。

## 2.3 JS独有的`$`符号
你可能会在代码中看到以`$`开头的变量：

```javascript
const $container = document.querySelector(".container");
const $button = document.getElementById("submit-btn");
```

这是一种**约定俗成的命名习惯**：以`$`开头的变量名通常表示它是一个**DOM元素**或**jQuery对象**。这不是语言规则，只是社区的编码约定。

# 三、JavaScript的7种基本数据类型
JS的数据类型分为两大类：**基本类型（Primitive）** 和 **引用类型（Reference/Object）**。本文先讲基本类型，引用类型（对象、数组等）后续文章专题讲解。

## 3.1 number：数字类型
JS中没有Python那样的`int`和`float`区分——所有数字（整数和小数）都是`number`类型。

```javascript
const integer = 42;        // 整数 → number
const float = 3.14;        // 浮点数 → number
const negative = -10;      // 负数 → number
const scientific = 1.5e6;  // 科学计数法（1500000）→ number

// JS只有一个数字类型
typeof 42;     // "number"
typeof 3.14;   // "number"
```

**Python对比**：
```python
# Python 区分 int 和 float
type(42)     # <class 'int'>
type(3.14)   # <class 'float'>
```

**特殊数值**：
```javascript
const nan = NaN;           // Not a Number（非数值，如 0/0 的结果）
const infinity = Infinity; // 无穷大（如 1/0 的结果）
```

> `NaN`是JS中的一个特殊值，表示"不是一个数字"。有趣的是`typeof NaN`返回`"number"`——一个"不是数字"的值的类型是"数字"。这是JS设计上的一个经典笑话。

## 3.2 string：字符串类型
JS的字符串有三种写法，Python也有三种写法，可以对应理解：

| 写法 | JS | Python |
|------|-----|--------|
| 单引号 | `'hello'` | `'hello'` |
| 双引号 | `"hello"` | `"hello"` |
| 模板/格式化 | `` `Hello ${name}` `` | `f"Hello {name}"` |

```javascript
const single = '单引号字符串';
const double = "双引号字符串";
const template = `模板字符串，可以嵌入变量：${single}`;
```

**JS和Python的区别**：
- JS中单引号和双引号**完全等价**，没有区别
- Python中单双引号也等价，但社区习惯单引号为主
- JS社区习惯单引号为主（但双引号也随处可见）
- JS的模板字符串（反引号）↔ Python的f-string

**模板字符串 vs Python f-string**：
```javascript
// JS：用反引号 + ${}
const name = "张三";
const age = 25;
const greeting = `我叫${name}，今年${age}岁`;

// Python：用 f + {}
name = "张三"
age = 25
greeting = f"我叫{name}，今年{age}岁"
```

两者用法几乎一样，只是符号不同。JS用反引号<code>`</code>包裹，`${}`嵌入变量；Python用`f`前缀和`{}`嵌入变量。

**多行字符串**：
```javascript
// JS 模板字符串可以直接换行
const poem = `床前明月光，
疑是地上霜。
举头望明月，
低头思故乡。`;

// Python 三引号也是直接换行
poem = """床前明月光，
疑是地上霜。
举头望明月，
低头思故乡。"""
```

## 3.3 boolean：布尔类型
只有两个值：`true`和`false`。和Python一样，区别是JS的布尔值全小写。

```javascript
// JS（全小写）
const isActive = true;
const isDeleted = false;

// Python（首字母大写）
is_active = True
is_deleted = False
```

## 3.4 undefined：未定义
`undefined`是JS独有的类型，表示**变量已声明但未赋值**。

```javascript
let x;
console.log(x);       // undefined
console.log(typeof x); // "undefined"
```

**什么时候会出现undefined**：
- 变量声明了但没有赋值
- 函数没有return语句（默认返回undefined）
- 访问对象不存在的属性

**Python对比**：Python没有`undefined`。在Python中如果使用未赋值的变量会直接抛出`NameError`。

## 3.5 null：空值
`null`表示"空"或者"没有值"。它和Python的`None`是对等的。

```javascript
const empty = null;
console.log(typeof null); // "object" ⚠️ 这是一个著名的JS bug！
```

**Python对比**：
```python
# Python
empty = None
print(type(None))  # <class 'NoneType'>
```

> **`typeof null === "object"`**：这是JS从1995年诞生就存在的bug。由于历史原因，这个bug一直保留到现在没有被修复（因为大量老网站依赖这个行为）。作为开发者，你只需要知道这是JS的一个"设计遗产"，看到`typeof null`返回`"object"`时不要惊讶。

## 3.6 undefined vs null：使用建议
| 场景 | 使用 | 含义 |
|------|------|------|
| 变量声明了但还没赋值 | `undefined` | JS引擎自动设置的初始值 |
| 主动表示"空"或者"不存在" | `null` | 开发者手动设置的空值 |

```javascript
// 让JS自动设置为undefined
let userName;  // userName === undefined

// 手动设置null表示"没有用户"
let currentUser = null;
```

**Python类比**：
- `undefined` ≈ Python中函数没有return时的`None`（自动的）
- `null` ≈ Python中你主动写的`None`（手动的）

## 3.7 symbol：符号类型（ES6新增）
`symbol`用于创建全局唯一的标识符，主要用于框架底层和复杂场景。入门阶段了解即可。

```javascript
const id1 = Symbol("id");
const id2 = Symbol("id");
console.log(id1 === id2); // false，每个Symbol都是唯一的
```

## 3.8 bigint：大整数类型（ES2020新增）
用于表示超出`Number`安全范围的极大整数。很少使用，知道即可。

```javascript
const bigNumber = 9007199254740991n; // 末尾加n表示bigint
```

# 四、typeof运算符
`typeof`用于检测变量或值的类型，返回一个表示类型的字符串。

```javascript
typeof 42;          // "number"
typeof "hello";     // "string"
typeof true;        // "boolean"
typeof undefined;   // "undefined"
typeof null;        // "object"（历史bug）
typeof Symbol();    // "symbol"
typeof {};          // "object"
typeof [];          // "object"（数组也是对象的一种）
typeof console.log; // "function"
```

**Python类比**：
```python
# Python的type()
type(42)      # <class 'int'>
type("hello") # <class 'str'>
type(True)    # <class 'bool'>
```

# 五、JS vs Python 数据类型完整对照表
| 含义 | Python | JavaScript |
|------|--------|------------|
| 整数 | `int` | `number` |
| 浮点数 | `float` | `number` |
| 字符串 | `str` | `string` |
| 布尔值 | `bool` (`True`/`False`) | `boolean` (`true`/`false`) |
| 空值 | `None` | `null` |
| 未定义 | 不存在（访问即报错） | `undefined` |
| 列表/数组 | `list` | `Array`（引用类型） |
| 字典/对象 | `dict` | `Object`（引用类型） |
| 集合 | `set` | `Set`（引用类型） |
| 元组 | `tuple` | 无原生元组 |
| 类型检查 | `type(x)` | `typeof x` 或 `x instanceof Type` |

# 六、动态类型与类型转换
JS和Python一样是**动态类型语言**——变量的类型在运行时确定，同一个变量可以先后赋值为不同类型。

```javascript
let value = "hello";   // value现在是string
value = 42;            // value现在是number
value = true;          // value现在是boolean
// 这在JS中是完全合法的，和Python一样
```

但带来了一个问题：两个不同类型的值做运算时，JS会进行**隐式类型转换**——这是JS中最容易出bug的特性之一。

```javascript
console.log("5" + 3);    // "53" —— 数字3被转为字符串"3"然后拼接
console.log("5" - 3);    // 2 —— 字符串"5"被转为数字5然后相减
console.log("5" * "3");  // 15 —— 两个字符串都被转为数字
```

> 类型转换的详细规则将在下一篇文章《运算符与类型转换》中深入讲解。这里先记住一句：当你不确定的时候，**不要依赖JS的隐式类型转换，手动转换**。

# 七、常见误区与避坑指南
1.  **用`var`而不是`let/const`**：看到其他教程或老项目中使用`var`，不要模仿。`let`和`const`是ES6后的标准写法，var有三个致命缺陷：允许重复声明、没有块级作用域、存在变量提升导致的奇怪行为。

2.  **该用const却用了let**：如果你声明了一个变量但在代码中从未重新赋值，应该使用`const`而不是`let`。这不仅仅是一个代码风格问题——`const`能防止意外修改，让代码更安全。

3.  **使用`==`比较`null`和`undefined`**：`null == undefined`使用`==`时返回`true`（因为JS会做类型转换），但`null === undefined`使用`===`时返回`false`。如果你没有意识到这一点，可能会导致难以排查的逻辑错误。

4.  **混淆undefined和null的使用场景**：基本原则是——让JS自己产生`undefined`（如声明变量不赋值），你手动设置的空值用`null`。不要写成`let x = undefined;`，这没有必要。

5.  **以为typeof null返回"null"**：它返回`"object"`。这是JS从1995年继承至今的经典bug。检查一个值是否为null的正确方式是：`value === null`。

6.  **用蛇形命名法写JS变量**：`user_name`在JS中虽然不会报错，但不合社区规范。统一使用驼峰命名法`userName`。

7.  **忘记用const声明引用类型**：`const arr = [1, 2, 3]; arr.push(4);` 这段代码不会报错。因为const固定的是引用地址，数组/对象的内容本身可以修改。这是JS中最需要理解的一个细节。

# 八、核心总结：变量与类型速查表
## 变量声明
| 关键字 | 可否修改 | 块级作用域 | 重复声明 | 推荐度 |
|--------|---------|-----------|---------|--------|
| `const` | ❌ | ✅ | ❌ | ⭐⭐⭐（首选） |
| `let` | ✅ | ✅ | ❌ | ⭐⭐（需要时使用） |
| `var` | ✅ | ❌ | ✅ | ❌（永不使用） |

## 基本数据类型
| 类型 | 示例 | typeof结果 | Python对比 |
|------|------|-----------|-----------|
| `number` | `42`, `3.14`, `NaN`, `Infinity` | `"number"` | `int`/`float` |
| `string` | `"hello"`, `'hello'`, `` `hello` `` | `"string"` | `str` |
| `boolean` | `true`, `false` | `"boolean"` | `bool` (True/False) |
| `undefined` | `let x;` | `"undefined"` | 无（访问未定义变量即报错） |
| `null` | `null` | `"object"`（⚠️bug） | `None` |
| `symbol` | `Symbol("id")` | `"symbol"` | 无 |
| `bigint` | `9007199254740991n` | `"bigint"` | `int`（Python无范围限制） |

## 常用速查
| 操作 | JS | Python |
|------|-----|--------|
| 声明变量 | `let x = 10;` | `x = 10` |
| 声明常量 | `const X = 10;` | `X = 10`（约定，不强制） |
| 类型检查 | `typeof x` | `type(x)` |
| 字符串模板 | `` `我叫${name}` `` | `f"我叫{name}"` |
| 空值 | `null` | `None` |
| 多行字符串 | `` `第一行\\n第二行` `` | `"""第一行\\n第二行"""` |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
