
# Python全栈入门到实战【JavaScript篇 10】DOM操作入门，用JavaScript操控网页元素

上一篇《JavaScript篇 09》中，我们攻克了JS的作用域、闭包和this三大难点。至此，你已经完整地学完了JavaScript作为一门编程语言的语法和核心概念。但相信你一直在等待这一刻——之前所有的`console.log`输出都只在控制台里，我们什么时候才能用JavaScript真正**操作网页**，实现点击按钮换颜色、点击切换菜单、动态改变页面内容？

现在，这一刻到了。从本篇开始，JavaScript回归它的终极使命：**操作网页**。在浏览器中，网页被抽象为一棵"节点树"——**DOM（Document Object Model，文档对象模型）**。JavaScript通过DOM API可以访问和修改这棵树上的任意节点——增删改查元素、修改样式、响应事件——所有的网页交互都是基于DOM操作实现的。

本篇作为DOM操作的第一篇，将从DOM的基本概念讲起，然后学习最核心的操作：获取元素、修改内容、修改样式、修改属性。每学一个API，都有可直接运行的实战示例。学完这一篇，你就能写出"点击按钮改变页面样式"这样真正有交互的代码了。

本节核心学习内容：
1.  DOM是什么：把HTML变成JavaScript可以操控的"对象树"
2.  获取元素：querySelector/querySelectorAll与getElementById系列
3.  修改内容：innerHTML vs textContent vs innerText
4.  修改样式：style属性与classList操作
5.  修改属性：setAttribute/getAttribute
6.  实战：点击按钮切换主题色、动态切换导航激活状态

# 一、DOM是什么
## 1.1 从HTML到DOM树
当浏览器加载一个HTML页面时，它会做两件事：
1. 解析HTML，构建一个**节点树**（DOM Tree）
2. 把JS中操作这个树的API暴露出来（DOM API）

每个HTML标签、文本、注释都变成了DOM树上的一个节点：

```html
<html>
  <head>
    <title>页面标题</title>
  </head>
  <body>
    <h1>Hello</h1>
    <p>这是一段文字</p>
  </body>
</html>
```
对应的DOM树结构：
```
document
  └── html
       ├── head
       │    └── title ("页面标题")
       └── body
            ├── h1 ("Hello")
            └── p ("这是一段文字")
```

JS可以通过DOM API访问和修改这棵树上的**任意节点**。这就是为什么JS能"操控网页"的根本原因——网页在浏览器内部就是一棵可以编程操作的对象树。

## 1.2 关键的DOM概念
| 概念 | 含义 | 类比 |
|------|------|------|
| **document** | 整个页面的入口对象 | Python中的根命名空间 |
| **节点** | DOM树中的每一个元素/文本/属性 | 树结构中的节点 |
| **元素节点** | HTML标签对应的节点（最常用） | 对象实例 |
| **属性** | 元素的属性（id、class、src等） | 对象的属性 |
| **文本节点** | 元素内部的文字内容 | 对象的字符串值 |

## 1.3 document对象
`document`是浏览器提供的全局对象，它代表整个HTML页面，是DOM操作的**入口**。你在Console中直接输入`document`，浏览器会返回整个页面的HTML结构。

```javascript
console.log(document);           // 整个HTML文档
console.log(document.title);     // 页面标题
console.log(document.URL);       // 当前URL
console.log(document.body);      // <body>元素
console.log(document.head);      // <head>元素
```

# 二、获取元素
要操作页面上的元素，首先需要"找到"它。JS提供了多种获取元素的方法，分为**现代方法**和**传统方法**两类。

## 2.1 querySelector（推荐）
`querySelector`使用CSS选择器语法来查找元素，返回**第一个匹配的元素**：

```html
<div class="container">
    <h1 id="title">标题</h1>
    <p class="content">第一段文字</p>
    <p class="content">第二段文字</p>
</div>
```

```javascript
// 通过标签名获取
const h1 = document.querySelector("h1");

// 通过类名获取（和CSS选择器一样加.）
const firstP = document.querySelector(".content");

// 通过ID获取（和CSS选择器一样加#）
const title = document.querySelector("#title");

// 复合选择器
const pInDiv = document.querySelector("div p");

// 如果没有匹配的元素，返回null
const notFound = document.querySelector(".not-exist");
console.log(notFound); // null
```

**为什么推荐querySelector**：
- 语法和CSS选择器完全一致，不需要额外学习新的选择器规则
- 支持任何CSS选择器（复合选择器、伪类选择器等）
- 一个方法统一所有查找场景

## 2.2 querySelectorAll（推荐）
`querySelectorAll`返回**所有匹配的元素**，结果是一个`NodeList`（类数组对象）：

```javascript
// 获取所有 .content 元素
const allContents = document.querySelectorAll(".content");
console.log(allContents.length); // 2

// 遍历NodeList
allContents.forEach((element, index) => {
    console.log(`第${index + 1}个：`, element.textContent);
});

// 通过索引访问
console.log(allContents[0]); // 第一个 .content 元素
```

> **注意**：`querySelectorAll`返回的是`NodeList`，**不是真正的数组**。它支持`forEach`遍历，但不支持`map`、`filter`等数组方法。如果要用这些方法，需要先转为数组：`[...nodeList]`或`Array.from(nodeList)`。

## 2.3 传统方法（了解即可）
这些是querySelector出现之前的获取元素方式，现在仍能见到但一般不推荐写新代码使用：

```javascript
// 通过ID获取（只返回一个元素）
const title = document.getElementById("title");

// 通过类名获取（返回HTMLCollection，实时更新）
const contents = document.getElementsByClassName("content");

// 通过标签名获取（返回HTMLCollection）
const paragraphs = document.getElementsByTagName("p");

// 通过name属性获取（用于表单元素）
const inputs = document.getElementsByName("username");
```

| 方法 | 参数 | 返回值 | 是否实时 |
|------|------|--------|---------|
| `querySelector(css)` | CSS选择器 | 第一个元素或null | 静态 |
| `querySelectorAll(css)` | CSS选择器 | NodeList（静态） | 静态 |
| `getElementById(id)` | ID字符串 | 一个元素或null | — |
| `getElementsByClassName(class)` | 类名字符串 | HTMLCollection | 实时 |
| `getElementsByTagName(tag)` | 标签名字符串 | HTMLCollection | 实时 |

# 三、修改元素内容
获取元素后，最常用的操作就是修改元素的文字或HTML内容。

## 3.1 textContent：纯文本（推荐）
`textContent`获取或设置元素的**纯文本内容**（不解析HTML标签）：

```javascript
const element = document.querySelector("h1");

// 获取文本
console.log(element.textContent);

// 设置文本
element.textContent = "新标题";

// 如果设置的内容包含HTML标签，会被当作普通文本显示
element.textContent = "<strong>加粗文字</strong>";
// 页面上会显示：<strong>加粗文字</strong>（标签被转义）
```

## 3.2 innerHTML：HTML内容（谨慎使用）
`innerHTML`获取或设置元素的**HTML内容**（会解析HTML标签）：

```javascript
const div = document.querySelector(".container");

// 获取HTML
console.log(div.innerHTML);

// 设置HTML（标签会被解析）
div.innerHTML = "<h2>这是标题</h2><p>这是段落</p>";

// ⚠️ 安全警告：不要用innerHTML插入用户输入的内容！
// 这会导致XSS（跨站脚本攻击）漏洞
// const userInput = "<img src=x onerror=alert('XSS')>";
// div.innerHTML = userInput; // 危险！
```

## 3.3 innerText：渲染后的文本（不推荐）
`innerText`和`textContent`类似，但会考虑CSS样式（如`display: none`的元素内容不会被获取），性能更差：

```javascript
element.innerText = "文字内容";
```

## 3.4 三者的区别对比
| 属性 | 获取内容 | 解析HTML | 考虑CSS样式 | 性能 | 推荐度 |
|------|---------|---------|------------|------|--------|
| `textContent` | 所有文本内容（包括隐藏元素） | ✗ | ✗ | 快 | ⭐⭐⭐ |
| `innerHTML` | HTML源码 | ✓ | ✗ | 慢 | ⭐（仅必要时） |
| `innerText` | 可见文本（不包括隐藏元素） | ✗ | ✓ | 最慢 | ✗ |

> **一句话建议**：修改纯文本用`textContent`，需要动态生成HTML结构时才用`innerHTML`，且永远不要用`innerHTML`插入用户输入的内容。

# 四、修改元素样式
## 4.1 style属性：修改内联样式
`element.style`可以读取和设置元素的内联样式（相当于给元素写了`style="..."`属性）：

```javascript
const box = document.querySelector(".box");

// 修改单个属性
box.style.color = "red";
box.style.backgroundColor = "blue";  // 注意：CSS的background-color在JS中变驼峰
box.style.fontSize = "20px";
box.style.display = "none";

// 一次修改多个属性
Object.assign(box.style, {
    color: "white",
    backgroundColor: "#333",
    padding: "10px",
    borderRadius: "5px"
});
```

**CSS属性名到JS属性名的转换规则**：
- 去掉连字符，连字符后的字母大写（驼峰命名）
- `background-color` → `backgroundColor`
- `font-size` → `fontSize`
- `border-radius` → `borderRadius`
- `margin-top` → `marginTop`

## 4.2 classList：操作类名（推荐）
操作样式的最佳方式不是直接修改`style`属性，而是**通过切换CSS类名来控制样式**。`classList`就是专门用于操作类名的API：

```javascript
const btn = document.querySelector("button");

// 添加类名
btn.classList.add("active");
btn.classList.add("primary", "large"); // 一次添加多个

// 删除类名
btn.classList.remove("active");

// 切换类名：有则删除，无则添加
btn.classList.toggle("dark-theme");

// 检查是否包含某个类名
if (btn.classList.contains("active")) {
    console.log("按钮处于激活状态");
}

// 替换类名
btn.classList.replace("old-class", "new-class");
```

**为什么要用classList而不是直接修改style**：
- 样式逻辑保留在CSS中，JS只负责"状态切换"，职责分离
- 修改类名比逐条修改style属性性能更好
- 代码更清晰：`btn.classList.add("active")`比`btn.style.color = "red"`更能表达意图

# 五、修改元素属性
## 5.1 标准属性
标准HTML属性可以直接通过点号访问和修改：

```javascript
// 链接
const link = document.querySelector("a");
link.href = "https://www.baidu.com";
link.target = "_blank";

// 图片
const img = document.querySelector("img");
img.src = "new-image.jpg";
img.alt = "新图片";

// 输入框
const input = document.querySelector("input");
input.value = "默认文字";
input.placeholder = "请输入内容";
input.disabled = true;

// 复选框/单选框
const checkbox = document.querySelector("input[type=checkbox]");
checkbox.checked = true;
```

## 5.2 getAttribute / setAttribute
对于非标准属性（自定义属性），需要用`getAttribute`和`setAttribute`：

```javascript
// 获取属性值
const value = element.getAttribute("data-id");

// 设置属性
element.setAttribute("data-status", "active");

// 删除属性
element.removeAttribute("disabled");

// 检查属性是否存在
element.hasAttribute("href"); // true或false
```

## 5.3 data-* 自定义属性
HTML5引入了`data-*`属性规范，用于在HTML元素上存储自定义数据。JS提供了专用的`dataset`API：

```html
<div id="user-card" data-user-id="1001" data-user-name="张三" data-role="admin">
    用户信息
</div>
```

```javascript
const card = document.querySelector("#user-card");

// dataset访问（自动转为驼峰命名）
console.log(card.dataset.userId);   // "1001"（data-user-id）
console.log(card.dataset.userName); // "张三"（data-user-name）
console.log(card.dataset.role);     // "admin"（data-role）

// 设置
card.dataset.status = "online"; // 等价于 data-status="online"
```

# 六、综合实战
## 6.1 点击按钮切换主题色
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>主题切换</title>
    <style>
        body {
            transition: background-color 0.3s, color 0.3s;
            font-family: sans-serif;
            padding: 40px;
            text-align: center;
        }
        body.light {
            background-color: #ffffff;
            color: #333333;
        }
        body.dark {
            background-color: #1a1a2e;
            color: #e0e0e0;
        }
        .btn {
            padding: 12px 30px;
            border: none;
            border-radius: 5px;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s;
        }
        .btn-light {
            background-color: #1a1a2e;
            color: white;
        }
        .btn-dark {
            background-color: #f0f0f0;
            color: #333;
        }
    </style>
</head>
<body class="light">
    <h1>主题切换演示</h1>
    <p>点击按钮切换明暗主题</p>
    <button id="theme-btn" class="btn btn-dark">切换暗色主题</button>

    <script>
        const body = document.body;
        const btn = document.querySelector("#theme-btn");

        btn.addEventListener("click", () => {
            // 判断当前主题
            if (body.classList.contains("light")) {
                // 切换到暗色
                body.classList.replace("light", "dark");
                btn.classList.replace("btn-dark", "btn-light");
                btn.textContent = "切换亮色主题";
            } else {
                // 切换到亮色
                body.classList.replace("dark", "light");
                btn.classList.replace("btn-light", "btn-dark");
                btn.textContent = "切换暗色主题";
            }
        });
    </script>
</body>
</html>
```

## 6.2 动态切换导航激活状态
```html
<style>
    .tab-nav {
        display: flex;
        list-style: none;
        gap: 5px;
        padding: 0;
    }
    .tab-nav a {
        display: block;
        padding: 10px 20px;
        background: #f0f0f0;
        border-radius: 5px;
        text-decoration: none;
        color: #666;
        transition: all 0.3s;
    }
    .tab-nav a.active {
        background: #3498db;
        color: white;
    }
    .tab-content {
        display: none;
        padding: 20px;
        background: #fff;
        border-radius: 5px;
        margin-top: 10px;
    }
    .tab-content.active {
        display: block;
    }
</style>

<nav>
    <ul class="tab-nav">
        <li><a href="#python" class="active">Python</a></li>
        <li><a href="#javascript">JavaScript</a></li>
        <li><a href="#mysql">MySQL</a></li>
    </ul>
</nav>

<div id="python" class="tab-content active">Python是最适合初学者的编程语言</div>
<div id="javascript" class="tab-content">JavaScript是前端交互的核心语言</div>
<div id="mysql" class="tab-content">MySQL是最流行的关系型数据库</div>

<script>
    const links = document.querySelectorAll(".tab-nav a");
    const contents = document.querySelectorAll(".tab-content");

    links.forEach(link => {
        link.addEventListener("click", (e) => {
            e.preventDefault(); // 阻止默认的锚点跳转

            // 1. 移除所有链接和内容的active状态
            links.forEach(l => l.classList.remove("active"));
            contents.forEach(c => c.classList.remove("active"));

            // 2. 激活当前点击的链接
            link.classList.add("active");

            // 3. 激活对应的内容
            const targetId = link.getAttribute("href").slice(1); // 去掉#号取id
            document.getElementById(targetId).classList.add("active");
        });
    });
</script>
```

# 七、常见误区与避坑指南
1.  **在DOM加载完成前操作元素**：如果`<script>`放在`<head>`中且不加`defer`，JS会在页面元素加载之前执行，此时`querySelector`会找不到任何元素返回null。解决方案：**把script放在body底部**，或使用`DOMContentLoaded`事件。

2.  **混淆textContent和innerHTML**：用`textContent`设置带标签的内容，标签不会被解析；用`innerHTML`插入用户输入可能导致XSS攻击。规则：纯文本用`textContent`，需要HTML结构用`innerHTML`且确保内容是安全的。

3.  **querySelectorAll返回的不是数组**：`NodeList`有`forEach`但没`map`/`filter`。需要数组方法时先转换：`[...document.querySelectorAll("p")]`。

4.  **修改style时连字符写成CSS原样**：`element.style.background-color`是错的，应该是`element.style.backgroundColor`（驼峰）。但用`element.style["background-color"]`也可以。

5.  **忘记innerHTML会覆盖原有内容**：`element.innerHTML = "<p>新内容</p>"`会**完全替换**元素内部的所有HTML和文本。如果需要在现有内容后追加，用`element.innerHTML +=`（注意性能）。

6.  **classList操作的不是数组**：`classList.add`、`classList.remove`去掉对应参数。不要写成`classList = ["active"]`。

# 八、核心总结：DOM操作速查表
## 获取元素
| 方法 | 参数 | 返回值 |
|------|------|--------|
| `querySelector(css)` | CSS选择器 | 第一个匹配元素 / null |
| `querySelectorAll(css)` | CSS选择器 | NodeList（静态） |
| `getElementById(id)` | ID字符串 | 元素 / null |

## 修改内容
| 属性 | 用途 | 安全 |
|------|------|------|
| `element.textContent` | 纯文本 | ✓ |
| `element.innerHTML` | HTML内容 | ⚠️（注意XSS） |
| `element.innerText` | 可见文本 | ✓ |

## 修改样式
| 方法 | 用途 |
|------|------|
| `element.style.property = "value"` | 直接设置内联样式 |
| `element.classList.add("class")` | 添加类 |
| `element.classList.remove("class")` | 删除类 |
| `element.classList.toggle("class")` | 切换类 |
| `element.classList.contains("class")` | 检查是否包含类 |

## 修改属性
| 方法 | 用途 |
|------|------|
| `element.attr = value` | 标准属性（href, src, value等）|
| `element.getAttribute("attr")` | 获取属性 |
| `element.setAttribute("attr", "val")` | 设置属性 |
| `element.removeAttribute("attr")` | 删除属性 |
| `element.dataset.name` | 访问data-name属性 |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
