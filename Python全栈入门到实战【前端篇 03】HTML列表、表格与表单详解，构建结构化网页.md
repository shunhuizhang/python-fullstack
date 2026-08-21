

# Python全栈入门到实战【前端篇 03】HTML列表、表格与表单详解，构建结构化网页
上一篇《前端篇 02》中，我们已经掌握了文本、图片、超链接等基础HTML标签，能够编写简单的静态页面。本篇作为前端篇的第三篇，我们将学习HTML中最重要的三个**结构化标签**：列表、表格和表单。列表用于展示有序或无序的内容，表格用于展示规整的结构化数据，表单用于收集用户输入的信息并提交给后端，这三个标签是构建复杂网页的基础，也是前后端数据交互的核心入口。

本文为Python全栈开发者量身打造，详细讲解每一种标签的语法、属性和使用场景，每一个知识点都配有可直接运行的实战示例。最后通过一个完整的个人简历页面实战，将所有知识点融会贯通，让你快速掌握HTML结构化标签的核心用法。

本节核心学习内容：
1.  列表标签：无序列表、有序列表与自定义列表
2.  表格标签：基本结构、常用属性与单元格合并
3.  表单标签：核心概念与常用表单元素详解
4.  综合实战：编写一个完整的个人简历页面
5.  常见误区：HTML结构化标签使用的常见错误
6.  核心总结：列表、表格与表单速查表

# 一、列表标签
列表用于将内容按照一定的顺序或分类进行展示，HTML提供了三种列表：无序列表、有序列表和自定义列表。

## 1.1 无序列表
无序列表用于展示没有先后顺序的内容，列表项前默认显示实心圆点。

**语法格式**：
```html
<ul>
    <li>列表项1</li>
    <li>列表项2</li>
    <li>列表项3</li>
</ul>
```

**实战示例**：
```html
<h3>我的技能</h3>
<ul>
    <li>Python后端开发</li>
    <li>MySQL数据库</li>
    <li>HTML/CSS前端开发</li>
    <li>Linux系统运维</li>
</ul>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/880635b6c3a54714823a72734f05cb26.png#pic_center)


## 1.2 有序列表

有序列表用于展示有先后顺序的内容，列表项前默认显示数字序号。

**语法格式**：
```html
<ol>
    <li>列表项1</li>
    <li>列表项2</li>
    <li>列表项3</li>
</ol>
```

**实战示例**：
```html
<h3>学习步骤</h3>
<ol>
    <li>学习Python基础语法</li>
    <li>学习MySQL数据库</li>
    <li>学习Web框架</li>
    <li>学习前端开发</li>
    <li>全栈项目实战</li>
</ol>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/19dc35d83da24b0ba4c4828ba4b5e434.png#pic_center)


## 1.3 自定义列表

自定义列表用于展示"术语-描述"形式的内容，由一个术语和多个描述项组成。

**语法格式**：
```html
<dl>
    <dt>术语1</dt>
    <dd>描述1</dd>
    <dd>描述2</dd>
    <dt>术语2</dt>
    <dd>描述1</dd>
</dl>
```

**实战示例**：
```html
<h3>技术栈说明</h3>
<dl>
    <dt>后端</dt>
    <dd>Python、Django、Flask、FastAPI</dd>
    <dt>数据库</dt>
    <dd>MySQL、Redis、MongoDB</dd>
    <dt>前端</dt>
    <dd>HTML、CSS、JavaScript、Vue3</dd>
</dl>
```
**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8be50b5ba75841fe8800975b056194fc.png#pic_center)


> ⚠️ 注意：列表标签可以嵌套使用，一个列表项中可以包含另一个完整的列表。

# 二、表格标签
表格用于展示规整的结构化数据，比如课程表、成绩单、员工信息表等。

## 2.1 表格基本结构
一个完整的表格由`<table>`、`<tr>`、`<th>`和`<td>`标签组成：
- `<table>`：定义整个表格
- `<tr>`：定义表格的一行
- `<th>`：定义表头单元格，文字默认加粗居中
- `<td>`：定义普通单元格

**语法格式**：
```html
<table>
    <tr>
        <th>表头1</th>
        <th>表头2</th>
        <th>表头3</th>
    </tr>
    <tr>
        <td>单元格1</td>
        <td>单元格2</td>
        <td>单元格3</td>
    </tr>
    <tr>
        <td>单元格4</td>
        <td>单元格5</td>
        <td>单元格6</td>
    </tr>
</table>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c330892e66b2446f870d4a9abc82d496.png#pic_center)


## 2.2 表格常用属性

| 属性          | 作用                       | 说明                        |
| ------------- | -------------------------- | --------------------------- |
| `border`      | 设置表格边框宽度           | 单位是像素                  |
| `cellpadding` | 设置单元格内容与边框的距离 | 单位是像素                  |
| `cellspacing` | 设置单元格之间的距离       | 单位是像素                  |
| `width`       | 设置表格宽度               | 单位是像素或百分比          |
| `align`       | 设置表格在页面中的对齐方式 | 可选值：left、center、right |

## 2.3 合并单元格
表格支持合并单元格，分为两种：
- **跨行合并**：`rowspan="n"`，合并n行
- **跨列合并**：`colspan="n"`，合并n列

合并单元格的步骤：
1. 确定合并的方向（跨行或跨列）
2. 在第一个单元格上添加对应的属性
3. 删除被合并的单元格

## 2.4 实战示例：课程表
```html
<h3>本周课程表</h3>
<table border="1" cellpadding="10" cellspacing="0" width="600" align="center">
    <tr>
        <th>时间</th>
        <th>周一</th>
        <th>周二</th>
        <th>周三</th>
        <th>周四</th>
        <th>周五</th>
    </tr>
    <tr>
        <td>上午1-2节</td>
        <td>Python</td>
        <td>MySQL</td>
        <td>HTML</td>
        <td>CSS</td>
        <td>JavaScript</td>
    </tr>
    <tr>
        <td>上午3-4节</td>
        <td>Linux</td>
        <td>Python</td>
        <td>MySQL</td>
        <td>HTML</td>
        <td>CSS</td>
    </tr>
    <tr>
        <td>下午1-2节</td>
        <td colspan="5">项目实战</td>
    </tr>
    <tr>
        <td>下午3-4节</td>
        <td>自习</td>
        <td>自习</td>
        <td>自习</td>
        <td>自习</td>
        <td>自习</td>
    </tr>
</table>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/28d26e1914c84c1d8734114d574b9f7a.png#pic_center)


# 三、表单标签

表单是用户与网站交互的核心，用于收集用户输入的信息（如用户名、密码、邮箱等），并将这些信息提交给后端服务器处理。这是前后端数据交互最重要的入口。

## 3.1 表单基本结构
表单由`<form>`标签和多个表单元素组成。

**语法格式**：
```html
<form action="提交地址" method="提交方式">
    <!-- 各种表单元素 -->
</form>
```

**核心属性**：
- `action`：指定表单数据提交到的后端接口地址
- `method`：指定提交方式，常用的有`GET`和`POST`
  - `GET`：数据会显示在地址栏中，不安全，适合提交少量数据
  - `POST`：数据不会显示在地址栏中，安全，适合提交大量数据和敏感数据

## 3.2 常用表单元素
### 1. input标签
`<input>`是最常用的表单元素，通过`type`属性可以指定不同的输入类型。

| type属性值 | 作用                             |
| ---------- | -------------------------------- |
| `text`     | 单行文本输入框                   |
| `password` | 密码输入框，输入内容会被隐藏     |
| `radio`    | 单选按钮                         |
| `checkbox` | 复选框                           |
| `submit`   | 提交按钮，点击后提交表单         |
| `reset`    | 重置按钮，点击后清空表单内容     |
| `button`   | 普通按钮，需要配合JavaScript使用 |

**实战示例**：
```html
<form action="/register" method="post">
    <!-- 用户名 -->
    <p>用户名：<input type="text" name="username" placeholder="请输入用户名"></p>
    
    <!-- 密码 -->
    <p>密码：<input type="password" name="password" placeholder="请输入密码"></p>
    
    <!-- 性别（单选按钮，name属性必须相同才能实现单选） -->
    <p>性别：
        <input type="radio" name="gender" value="1" checked> 男
        <input type="radio" name="gender" value="2"> 女
    </p>
    
    <!-- 爱好（复选框） -->
    <p>爱好：
        <input type="checkbox" name="hobby" value="coding"> 编程
        <input type="checkbox" name="hobby" value="reading"> 读书
        <input type="checkbox" name="hobby" value="sports"> 运动
    </p>
    
    <!-- 按钮 -->
    <p>
        <input type="submit" value="注册">
        <input type="reset" value="重置">
    </p>
</form>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/22a55a4628a2482988c8dd0a65f7cd4c.png#pic_center)


### 2. textarea标签

`<textarea>`用于定义多行文本输入框，适合输入较长的内容。

**语法格式**：
```html
<textarea name="content" rows="5" cols="30" placeholder="请输入内容"></textarea>
```

**属性说明**：
- `rows`：指定文本框的行数
- `cols`：指定文本框的列数

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0d4dd703f38e4d6e8f70a9ebf83208d7.png#pic_center)


### 3. select标签
`<select>`用于定义下拉列表，`<option>`用于定义列表项。

**语法格式**：
```html
<select name="city">
    <option value="beijing">北京</option>
    <option value="shanghai">上海</option>
    <option value="guangzhou" selected>广州</option>
    <option value="shenzhen">深圳</option>
</select>
```

> ⚠️ 注意：所有表单元素都必须添加`name`属性，否则后端无法获取到该元素的值。

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/274d8db2ed33466089d0d6d17848f02a.png#pic_center)


# 四、综合实战：个人简历页面

下面我们将前面学到的列表、表格和表单标签结合起来，编写一个完整的个人简历页面。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>个人简历</title>
</head>
<body>
    <h1>个人简历</h1>
    <hr>

    <h2>基本信息</h2>
    <table border="1" cellpadding="10" cellspacing="0" width="600">
        <tr>
            <th>姓名</th>
            <td>张顺辉</td>
            <th>性别</th>
            <td>男</td>
        </tr>
        <tr>
            <th>年龄</th>
            <td>25</td>
            <th>电话</th>
            <td>13800138000</td>
        </tr>
        <tr>
            <th>邮箱</th>
            <td>example@example.com</td>
            <th>求职意向</th>
            <td>Python全栈开发工程师</td>
        </tr>
    </table>
    <hr>

    <h2>教育经历</h2>
    <table border="1" cellpadding="10" cellspacing="0" width="600">
        <tr>
            <th>时间</th>
            <th>学校</th>
            <th>专业</th>
            <th>学历</th>
        </tr>
        <tr>
            <td>2018-2022</td>
            <td>XX大学</td>
            <td>计算机科学与技术</td>
            <td>本科</td>
        </tr>
    </table>
    <hr>

    <h2>技能特长</h2>
    <ul>
        <li>熟练掌握Python语言，熟悉Django、Flask、FastAPI等Web框架</li>
        <li>熟练掌握MySQL数据库，能够进行数据库设计和优化</li>
        <li>掌握HTML、CSS、JavaScript基础，了解Vue3框架</li>
        <li>熟悉Linux系统，能够进行服务器部署和运维</li>
    </ul>
    <hr>

    <h2>项目经历</h2>
    <ol>
        <li>
            <strong>电商后台管理系统</strong>
            <p>使用Django+MySQL开发，实现了商品管理、订单管理、用户管理等功能</p>
        </li>
        <li>
            <strong>博客系统</strong>
            <p>使用FastAPI+Vue3开发，实现了文章发布、评论、分类等功能</p>
        </li>
    </ol>
    <hr>

    <h2>留言板</h2>
    <form action="/message" method="post">
        <p>姓名：<input type="text" name="name" placeholder="请输入您的姓名"></p>
        <p>邮箱：<input type="text" name="email" placeholder="请输入您的邮箱"></p>
        <p>留言内容：<br>
            <textarea name="content" rows="5" cols="50" placeholder="请输入您的留言"></textarea>
        </p>
        <p>
            <input type="submit" value="提交留言">
            <input type="reset" value="重置">
        </p>
    </form>
</body>
</html>
```

**效果展示**:

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b92d107c57df44fba5c999be24e55bfd.png#pic_center)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7753a3fb9dd24ac2b33fa2ac26b49daa.png#pic_center)


# 五、常见误区与避坑指南

1.  **表单元素忘记加name属性**：这是最常见的错误，没有`name`属性的表单元素，后端无法获取到它的值。
2.  **单选按钮name属性不同**：单选按钮的`name`属性必须相同，才能实现互斥选择。
3.  **合并单元格后忘记删除多余单元格**：合并单元格后，必须删除被合并的单元格，否则表格会变形。
4.  **使用GET方式提交敏感数据**：密码、身份证号等敏感数据必须使用`POST`方式提交，不能使用`GET`方式。
5.  **滥用表格进行布局**：表格的作用是展示数据，不是用来做页面布局的，页面布局应该使用CSS来实现。
6.  **列表标签嵌套错误**：`<ul>`和`<ol>`的直接子元素只能是`<li>`，不能直接写其他内容。

# 六、核心总结：HTML结构化标签速查表
为了方便后续开发时快速查阅，整理了本文讲解的核心标签速查表：

## 列表标签
| 标签   | 作用           |
| ------ | -------------- |
| `<ul>` | 无序列表       |
| `<ol>` | 有序列表       |
| `<li>` | 列表项         |
| `<dl>` | 自定义列表     |
| `<dt>` | 自定义列表术语 |
| `<dd>` | 自定义列表描述 |

## 表格标签
| 标签      | 作用           |
| --------- | -------------- |
| `<table>` | 定义表格       |
| `<tr>`    | 定义表格行     |
| `<th>`    | 定义表头单元格 |
| `<td>`    | 定义普通单元格 |
| `rowspan` | 跨行合并单元格 |
| `colspan` | 跨列合并单元格 |

## 表单标签
| 标签/属性                 | 作用           |
| ------------------------- | -------------- |
| `<form>`                  | 定义表单       |
| `<input type="text">`     | 单行文本输入框 |
| `<input type="password">` | 密码输入框     |
| `<input type="radio">`    | 单选按钮       |
| `<input type="checkbox">` | 复选框         |
| `<input type="submit">`   | 提交按钮       |
| `<input type="reset">`    | 重置按钮       |
| `<textarea>`              | 多行文本输入框 |
| `<select>`                | 下拉列表       |
| `<option>`                | 下拉列表项     |

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
