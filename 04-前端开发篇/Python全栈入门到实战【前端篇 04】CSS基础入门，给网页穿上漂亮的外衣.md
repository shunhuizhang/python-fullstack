


# Python全栈入门到实战【前端篇 04】CSS基础入门，给网页穿上漂亮的"外衣"
上一篇《前端篇 03》中，我们已经掌握了列表、表格、表单等HTML结构化标签，能够构建出完整的网页骨架。但纯HTML写出来的页面非常简陋，没有任何样式和美观度。本篇作为前端篇的第四篇，我们将正式学习**CSS（层叠样式表）**，它的作用就是给HTML页面添加样式，控制页面的颜色、字体、布局、动画等所有视觉效果，让网页变得美观、易用。

本文为Python全栈开发者量身打造，从CSS的核心概念讲起，详细讲解CSS的三种引入方式、基础选择器和常用样式属性。每一个知识点都配有可直接运行的实战示例，最后通过美化个人介绍页面的综合实战，让你直观感受CSS的强大，快速掌握CSS基础用法。

本节核心学习内容：
1.  CSS概述：什么是CSS、为什么需要CSS
2.  CSS的三种引入方式：行内样式、内部样式、外部样式
3.  CSS基础语法与常用选择器
4.  常用文本样式属性详解
5.  常用背景样式属性详解
6.  综合实战：用CSS美化个人介绍页面
7.  常见误区：CSS入门最容易踩的坑
8.  核心总结：CSS基础速查表

# 一、CSS概述
## 1.1 什么是CSS
CSS（Cascading Style Sheets）即**层叠样式表**，是一种用于描述HTML文档样式的标记语言。

简单来说：
- HTML负责定义网页的**结构和内容**
- CSS负责定义网页的**样式和布局**

## 1.2 为什么需要CSS
在CSS出现之前，网页的样式都是直接写在HTML标签中的，这会导致以下问题：
- 代码冗余：相同的样式需要在每个标签中重复编写
- 维护困难：修改一个样式需要修改所有相关的HTML标签
- 结构与样式混杂：HTML代码变得臃肿，可读性差

CSS的出现实现了**结构与样式的分离**：
- HTML只负责结构，代码更加清晰简洁
- 所有样式统一写在CSS中，修改一次即可全局生效
- 便于团队协作和后期维护

# 二、CSS的三种引入方式
CSS有三种引入方式，分别适用于不同的场景。

## 2.1 行内样式
行内样式是将CSS样式直接写在HTML标签的`style`属性中。

**语法格式**：
```html
<标签名 style="属性1: 值1; 属性2: 值2;">内容</标签名>
```

**实战示例**：
```html
<h1 style="color: red; font-size: 30px;">这是一个红色的大标题</h1>
<p style="color: blue; font-size: 16px;">这是一个蓝色的段落</p>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b88740cc32b24a02affc283c1899611f.png#pic_center)



**优缺点**：

- 优点：简单直接，优先级最高
- 缺点：结构与样式混杂，代码冗余，只能作用于单个标签
- 适用场景：临时修改单个元素的样式，不推荐大量使用

## 2.2 内部样式
内部样式是将CSS代码写在HTML文档的`<head>`标签中的`<style>`标签内。

**语法格式**：
```html
<head>
    <style>
        选择器 {
            属性1: 值1;
            属性2: 值2;
        }
    </style>
</head>
```

**实战示例**：
```html
<head>
    <style>
        h1 {
            color: red;
            font-size: 30px;
        }
        p {
            color: blue;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <h1>这是一个红色的大标题</h1>
    <p>这是一个蓝色的段落</p>
</body>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b88740cc32b24a02affc283c1899611f.png#pic_center)

**优缺点**：

- 优点：结构与样式分离，作用于当前整个页面
- 缺点：只能作用于当前页面，不能在多个页面之间共享
- 适用场景：单个页面的样式编写，学习阶段常用

## 2.3 外部样式（推荐使用）
外部样式是将CSS代码写在单独的`.css`文件中，然后在HTML文档中通过`<link>`标签引入。这是生产环境中推荐使用的方式。

**语法格式**：
1. 创建一个后缀为`.css`的文件，如`style.css`，在其中编写CSS代码
2. 在HTML的`<head>`标签中引入该CSS文件
```html
<link rel="stylesheet" href="css文件路径">
```

**实战示例**：
1. 创建`style.css`文件：
```css
h1 {
    color: red;
    font-size: 30px;
}
p {
    color: blue;
    font-size: 16px;
}
```

2. 在HTML中引入：
```html
<head>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>这是一个红色的大标题</h1>
    <p>这是一个蓝色的段落</p>
</body>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b88740cc32b24a02affc283c1899611f.png#pic_center)

**优缺点**：

- 优点：完全实现结构与样式分离，样式可以在多个页面之间共享，便于维护和管理
- 缺点：需要额外创建CSS文件
- 适用场景：所有正式项目，推荐使用

# 三、CSS基础语法与常用选择器
## 3.1 CSS基础语法
CSS的基本语法由**选择器**和**声明块**两部分组成：
```css
选择器 {
    属性1: 值1;
    属性2: 值2;
    /* 这是注释 */
}
```

- **选择器**：用于选中需要添加样式的HTML元素
- **声明块**：由一对大括号包裹，包含多个样式声明
- **样式声明**：由`属性: 值`的形式组成，多个声明之间用分号分隔
- **注释**：CSS的注释格式为`/* 注释内容 */`

## 3.2 常用基础选择器
选择器是CSS的核心，用于精准选中需要添加样式的元素。下面介绍四个最基础、最常用的选择器。

### 1. 标签选择器
标签选择器根据HTML标签名来选中元素，会选中页面中所有该类型的标签。

**语法格式**：
```css
标签名 {
    样式声明;
}
```

**示例**：
```css
/* 选中所有的h1标签 */
h1 {
    color: red;
}

/* 选中所有的p标签 */
p {
    font-size: 16px;
}
```

### 2. 类选择器
类选择器根据元素的`class`属性来选中元素，可以选中多个元素。

**语法格式**：
```css
.类名 {
    样式声明;
}
```

**示例**：
```html
<p class="red-text">这段文字是红色的</p>
<p class="red-text">这段文字也是红色的</p>
<p>这段文字是默认颜色</p>
```
```css
.red-text {
    color: red;
}
```

> 💡 一个元素可以有多个类名，用空格分隔：`<p class="red-text big-text">内容</p>`

### 3. id选择器
id选择器根据元素的`id`属性来选中元素，一个页面中id必须是唯一的。

**语法格式**：
```css
#id名 {
    样式声明;
}
```

**示例**：
```html
<h1 id="main-title">这是主标题</h1>
```
```css
#main-title {
    font-size: 36px;
    text-align: center;
}
```

### 4. 通配符选择器
通配符选择器用`*`表示，选中页面中所有的元素。

**语法格式**：
```css
* {
    样式声明;
}
```

**示例**：
```css
/* 清除所有元素的默认内外边距，这是CSS的常用初始化操作 */
* {
    margin: 0;
    padding: 0;
}
```

## 3.3 选择器优先级
当多个选择器同时作用于同一个元素时，会按照优先级来决定应用哪个样式：
**id选择器 > 类选择器 > 标签选择器 > 通配符选择器**

行内样式的优先级高于所有上述选择器。

# 四、常用文本样式属性
文本样式是最常用的CSS样式，用于控制文字的颜色、字体、大小、对齐方式等。

| 属性              | 作用                 | 常用值                                                       |
| ----------------- | -------------------- | ------------------------------------------------------------ |
| `color`           | 设置文字颜色         | 颜色名（如red）、十六进制（如#ff0000）、RGB值（如rgb(255,0,0)） |
| `font-size`       | 设置文字大小         | 像素值（如16px）                                             |
| `font-family`     | 设置文字字体         | 微软雅黑、宋体、Arial等                                      |
| `font-weight`     | 设置文字粗细         | normal（正常）、bold（加粗）                                 |
| `text-align`      | 设置文本水平对齐方式 | left（左对齐）、center（居中）、right（右对齐）              |
| `line-height`     | 设置行高             | 像素值（如24px）、数字（如1.5，表示字体大小的1.5倍）         |
| `text-decoration` | 设置文本装饰         | none（无装饰）、underline（下划线）、line-through（删除线）  |

**实战示例**：
```css
h1 {
    color: #333333;
    font-size: 32px;
    font-family: "微软雅黑";
    font-weight: bold;
    text-align: center;
}

p {
    color: #666666;
    font-size: 16px;
    line-height: 1.5;
    text-indent: 2em; /* 首行缩进2个字符 */
}

a {
    color: #0066cc;
    text-decoration: none; /* 去掉超链接的下划线 */
}
```

# 五、常用背景样式属性
背景样式用于控制元素的背景颜色、背景图片等。

| 属性                  | 作用                 | 常用值                                                       |
| --------------------- | -------------------- | ------------------------------------------------------------ |
| `background-color`    | 设置背景颜色         | 同color属性                                                  |
| `background-image`    | 设置背景图片         | `url('图片路径')`                                            |
| `background-repeat`   | 设置背景图片是否重复 | no-repeat（不重复）、repeat-x（水平重复）、repeat-y（垂直重复） |
| `background-position` | 设置背景图片位置     | 方位词（top、center、bottom、left、right）、像素值           |
| `background-size`     | 设置背景图片大小     | cover（覆盖整个元素）、contain（完全包含在元素内）、像素值   |

**实战示例**：
```css
/* 设置页面背景颜色 */
body {
    background-color: #f5f5f5;
}

/* 设置元素背景图片 */
.banner {
    width: 100%;
    height: 300px;
    background-image: url('images/banner.jpg');
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
}
```

# 六、综合实战：用CSS美化个人介绍页面
下面我们用前面学到的CSS知识，美化上一篇写的个人介绍页面。

## 6.1 创建外部CSS文件
创建`style.css`文件，编写以下样式：
```css
/* 全局初始化 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    background-color: #f5f5f5;
    font-family: "微软雅黑", Arial, sans-serif;
    color: #333;
    line-height: 1.6;
}

/* 页面容器 */
.container {
    width: 800px;
    margin: 50px auto;
    background-color: #fff;
    padding: 40px;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0,0,0,0.1);
}

/* 标题样式 */
h1 {
    text-align: center;
    color: #2c3e50;
    margin-bottom: 30px;
    padding-bottom: 10px;
    border-bottom: 2px solid #3498db;
}

h2 {
    color: #3498db;
    margin: 25px 0 15px;
}

/* 水平线样式 */
hr {
    border: none;
    border-top: 1px solid #eee;
    margin: 20px 0;
}

/* 头像样式 */
.avatar {
    display: block;
    width: 150px;
    height: 150px;
    border-radius: 50%;
    margin: 20px auto;
    object-fit: cover;
}

/* 链接样式 */
a {
    color: #3498db;
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}
```

## 6.2 修改HTML文件
修改`about.html`文件，引入CSS文件并添加相应的类名：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>个人介绍</title>
    <!-- 引入外部CSS文件 -->
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>小辉辉</h1>
        <h2>Python全栈开发工程师</h2>
        <hr>

        <h3>个人简介</h3>
        <p>大家好，我是<strong>小辉辉</strong>，一名专注于Python全栈开发的工程师。</p>
        <p>拥有多年开发经验，擅长后端接口开发、数据库设计和前端页面开发，能够独立完成完整的Web项目开发。</p>
        <hr>

        <h3>个人头像</h3>
        <img class="avatar" src="https://www.python.org/static/img/python-logo.png" alt="个人头像">
        <hr>

        <h3>技能列表</h3>
        <p>后端：Python、Django、Flask、FastAPI</p>
        <p>数据库：MySQL、Redis</p>
        <p>前端：HTML、CSS、JavaScript、Vue3</p>
        <hr>

        <h3>联系方式</h3>
        <p>邮箱：example@example.com</p>
        <p>博客：<a href="https://blog.csdn.net/zsh_1314520" target="_blank">CSDN博客</a></p>
    </div>
</body>
</html>
```

保存文件后用Live Server运行，你会发现原本简陋的个人介绍页面变得非常美观、专业。

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8261db53b1af4269810271d2c9bbc623.png#pic_center)


# 七、常见误区与避坑指南

1.  **滥用行内样式**：行内样式优先级最高，会覆盖所有其他样式，导致后期维护困难，尽量不要使用。
2.  **id重复使用**：一个页面中id必须是唯一的，多个元素需要相同样式时应该使用类选择器。
3.  **忘记加单位**：CSS中除了0以外，所有数值都必须加单位（如px），否则样式不生效。
4.  **选择器命名不规范**：选择器命名应该语义化，不要用拼音或数字，多个单词用连字符分隔（如`main-title`）。
5.  **颜色值写法错误**：十六进制颜色值必须以`#`开头，RGB值的格式是`rgb(红,绿,蓝)`，每个值范围0-255。
6.  **CSS注释写法错误**：CSS的注释是`/* 注释 */`，不要用HTML的`<!-- 注释 -->`或JavaScript的`// 注释`。

# 八、核心总结：CSS基础速查表
## CSS引入方式
| 引入方式 | 语法                                       | 适用场景           |
| -------- | ------------------------------------------ | ------------------ |
| 行内样式 | `<标签 style="属性:值;">`                  | 临时修改单个元素   |
| 内部样式 | `<style>...</style>`                       | 单个页面           |
| 外部样式 | `<link rel="stylesheet" href="style.css">` | 正式项目，推荐使用 |

## 基础选择器
| 选择器       | 语法     | 优先级 |
| ------------ | -------- | ------ |
| id选择器     | `#id`    | 最高   |
| 类选择器     | `.class` | 中     |
| 标签选择器   | `标签名` | 低     |
| 通配符选择器 | `*`      | 最低   |

## 常用样式属性
| 类别     | 常用属性                                                     |
| -------- | ------------------------------------------------------------ |
| 文本样式 | color、font-size、font-family、text-align、line-height、text-decoration |
| 背景样式 | background-color、background-image、background-repeat、background-position |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
