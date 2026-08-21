
# Python全栈入门到实战【Django篇 03】模板系统详解，动态渲染HTML页面
上一篇《Django篇 02》中，我们掌握了URL路由和视图函数。但目前所有的视图都是直接`return HttpResponse("文字")`——返回的内容全是在Python代码中拼接的字符串，没有HTML结构、没有CSS样式。这样的页面和"电子版打印纸"差不多，完全不符合现代Web应用的标准。

真正的前端页面应该是精美的HTML结构+CSS样式，其中数据部分是动态填充的（比如文章标题、用户名称是从数据库查出来的）。Django的**模板系统（DTL）** 就是用来实现这种"HTML骨架+动态数据填充"的。你把HTML写好，留出数据占位符，Django在运行时把真实数据填进去，生成完整的HTML页面返回给浏览器。

本篇为Python全栈开发者量身打造，从模板的基本语法讲起，覆盖变量、过滤器、标签、模板继承、静态文件引入和自定义模板标签。学完这一篇，你就能把后端的Python数据优雅地渲染到前端HTML页面上——这是Django全栈开发中最核心的技能之一。

本节核心学习内容：
1.  模板语法：{{ }}变量与{% %}标签的核心区别
2.  变量渲染：从视图传递数据到模板
3.  内置过滤器：字符串、日期、数字格式化
4.  流程控制标签：if/for、循环变量、empty/empty
5.  模板继承：block/extends/include 复用HTML骨架
6.  静态文件：CSS/JS/图片的正确引入方式
7.  自定义模板标签与过滤器
8.  常见误区与避坑指南

# 一、模板系统基础
## 1.1 什么是Django模板系统
Django模板系统（DTL, Django Template Language）是一套在HTML中嵌入Python逻辑的语法。它让你在HTML中写`{{ 变量 }}`和`{% 标签 %}`来动态生成内容。

**Python类比**：Python的`f-string`是`f"Hello {name}"`，Django模板是`<h1>Hello {{ name }}</h1>`。思维模式相同——在字符串中嵌入变量。

## 1.2 模板存放位置
Django默认在**每个应用的`templates/`目录**下查找模板文件。标准的目录结构是：

```
blog/                        # 应用目录
├── templates/               # 模板目录（需手动创建）
│   └── blog/                # 加一层应用名作为命名空间（防止跨应用模板重名）
│       ├── index.html        # 首页模板
│       └── detail.html       # 详情页模板
├── views.py
├── models.py
└── ...
```

> 为什么模板目录下还要加一层`blog/`？假如项目有多个应用，每个应用都有一个`index.html`，Django无法区分该用哪个。加一层应用名就能区分：render时写`'blog/index.html'`和`'shop/index.html'`，不会冲突。

## 1.3 第一个模板
**创建模板目录**：
```bash
# 在 blog/ 应用目录下
mkdir templates
mkdir templates/blog
```

**编写模板文件** `blog/templates/blog/index.html`：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
    <h1>{{ heading }}</h1>
    <p>欢迎来到 {{ site_name }}！</p>
    <p>当前时间：{{ current_time }}</p>
</body>
</html>
```

**在视图中渲染模板**：
```python
# blog/views.py
from django.shortcuts import render
from datetime import datetime

def index(request):
    # 准备数据：一个字典，key是模板中的变量名，value是实际数据
    context = {
        'title': 'Python全栈博客',
        'heading': '欢迎来到我的博客',
        'site_name': 'Django学习站',
        'current_time': datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
    }
    # render() 把数据和模板合成为HTML并返回
    return render(request, 'blog/index.html', context)
```

**render()函数的三要素**：
```
render(request, 模板路径, 数据字典)
   ↑         ↑         ↑
 请求对象   HTML模板   context上下文
```

访问`/blog/`，你会看到一个完整的HTML页面，其中的`{{ title }}`、`{{ heading }}`等占位符已经被Python数据替换了。

# 二、模板语法：{{ }}变量
## 2.1 基本变量渲染
`{{ variable }}` 将变量的值输出到模板。变量来自视图的`context`字典：

```python
# views.py 中的 context
context = {
    'name': '张三',
    'age': 25,
    'skills': ['Python', 'Django', 'MySQL'],
    'user': {'username': 'zhangsan', 'email': 'zhangsan@example.com'},
    'is_admin': True,
}
```

```html
<!-- blog/index.html -->
<p>姓名：{{ name }}</p>                      <!-- 张三 -->
<p>年龄：{{ age }}</p>                       <!-- 25 -->
<p>技能：{{ skills }}</p>                    <!-- ['Python', 'Django', 'MySQL'] -->
<p>第一个技能：{{ skills.0 }}</p>             <!-- Python -->
<p>用户名：{{ user.username }}</p>            <!-- zhangsan -->
<p>是否管理员：{{ is_admin }}</p>             <!-- True -->
```

## 2.2 点号（.）访问的优先级
当模板遇到`{{ obj.prop }}`时，Django按以下顺序查找`prop`：
1. 字典查找：`obj["prop"]`
2. 属性查找：`obj.prop`
3. 列表索引：`obj[0]`（`obj.0`被当作索引0）
4. 方法调用：`obj.prop()`（无参数的方法）

```html
<!-- 字典：user.username → user["username"] -->
{{ user.username }}

<!-- 列表：items.0 → items[0] -->
{{ items.0 }}

<!-- 对象属性：article.title → article.title -->
{{ article.title }}

<!-- 方法：article.get_absolute_url → article.get_absolute_url() -->
{{ article.get_absolute_url }}
```

> 和JS的`.`比较：JS中`obj.prop`和`obj["prop"]`都可以访问，Django模板中统一用`.`。

# 三、内置过滤器：|pipe处理变量
## 3.1 什么是过滤器
过滤器用于**修改变量的显示格式**。语法是在变量后面加`|`管道符和过滤器名：

```
{{ 变量 | 过滤器:参数 }}
```

这和Linux命令行的管道（`|`）概念一样：数据从左到右流动，每个过滤器对数据做一次处理。

## 3.2 常用内置过滤器

| 过滤器 | 作用 | 示例 | 输出 |
|--------|------|------|------|
| `length` | 返回长度 | `{{ list\|length }}` | `5` |
| `default` | 变量为False时显示默认值 | `{{ name\|default:"匿名" }}` | `"匿名"` |
| `date` | 格式化日期 | `{{ time\|date:"Y-m-d" }}` | `"2026-07-30"` |
| `date:"Y-m-d H:i"` | 日期+时间 | `{{ time\|date:"Y-m-d H:i" }}` | `"2026-07-30 14:30"` |
| `truncatechars` | 截断字符串（含...） | `{{ text\|truncatechars:10 }}` | `"Python全..."` |
| `truncatechars_html` | 截断HTML字符串（不截断标签） | `{{ html\|truncatechars_html:10 }}` | 安全截断 |
| `floatformat` | 浮点数保留小数位 | `{{ 3.14159\|floatformat:2 }}` | `"3.14"` |
| `lower` | 转为小写 | `{{ "HELLO"\|lower }}` | `"hello"` |
| `upper` | 转为大写 | `{{ "hello"\|upper }}` | `"HELLO"` |
| `safe` | 标记为安全（不转义HTML） | `{{ html_content\|safe }}` | 渲染原始HTML |
| `linebreaks` | 将换行符转为`<br>`和`<p>` | `{{ text\|linebreaks }}` | 带段落的HTML |
| `join` | 用分隔符连接列表 | `{{ list\|join:", " }}` | `"a, b, c"` |
| `add` | 加法（或字符串拼接） | `{{ value\|add:5 }}` | `15` |
| `yesno` | 布尔值显示为自定义文字 | `{{ is_active\|yesno:"是,否" }}` | `"是"` |
| `slice` | 切片 | `{{ list\|slice:":3" }}` | 前三个元素 |

**实战示例**：
```html
<h2>{{ article.title }}</h2>
<p>作者：{{ article.author|default:"匿名作者" }}</p>
<p>发布时间：{{ article.pub_date|date:"Y年m月d日" }}</p>
<p>摘要：{{ article.content|truncatechars:100 }}</p>
<p>标签：{{ tags|join:" / " }}</p>
<p>状态：{{ article.is_published|yesno:"已发布,草稿" }}</p>
<p>价格：¥{{ price|floatformat:2 }}</p>
```

## 3.3 过滤器链式使用
过滤器可以串联使用，从左到右依次处理：

```html
<!-- 先截断到20字符，再转为大写 -->
{{ text|truncatechars:20|upper }}

<!-- 先获取第一个元素，再显示其名称 -->
{{ categories|first|default:"未分类" }}
```

# 四、模板标签：{% %}流程控制
## 4.1 {% if %} 条件判断
```html
<!-- 基本用法 -->
{% if user.is_authenticated %}
    <p>欢迎回来，{{ user.username }}！</p>
{% elif user.is_staff %}
    <p>管理员您好</p>
{% else %}
    <p>请先登录</p>
{% endif %}

<!-- 逻辑运算 -->
{% if score >= 90 %}
    <span class="grade-a">优秀</span>
{% elif score >= 60 %}
    <span class="grade-c">及格</span>
{% else %}
    <span class="grade-f">不及格</span>
{% endif %}

<!-- 多条件组合 -->
{% if age >= 18 and is_student %}
    <p>成年学生</p>
{% endif %}

{% if category == "python" or category == "django" %}
    <p>Python相关</p>
{% endif %}
```

## 4.2 {% for %} 循环遍历
```html
<!-- 遍历列表 -->
<ul>
{% for article in articles %}
    <li>
        <a href="/article/{{ article.id }}/">{{ article.title }}</a>
        <span>{{ article.pub_date|date:"Y-m-d" }}</span>
    </li>
{% empty %}
    <li>暂无文章</li>
{% endfor %}
</ul>

<!-- 遍历字典 -->
<dl>
{% for key, value in user_dict.items %}
    <dt>{{ key }}</dt>
    <dd>{{ value }}</dd>
{% endfor %}
</dl>
```

**`{% empty %}`**：当被遍历的对象为空时显示的内容。相当于Python的`if not items: print("暂无数据")`。

## 4.3 forloop内置变量
在`{% for %}`循环内，Django提供特殊的`forloop`变量来获取循环信息：

```html
{% for article in articles %}
    <div>
        {{ forloop.counter }}. {{ article.title }}
        <!-- forloop.counter：当前迭代次数（从1开始） -->
        <!-- forloop.counter0：当前迭代次数（从0开始） -->
        <!-- forloop.revcounter：剩余迭代次数（从1开始倒数） -->
        <!-- forloop.first：是否第一次迭代 -->
        <!-- forloop.last：是否最后一次迭代 -->
    </div>
{% endfor %}
```

**实战：隔行变色表格**：
```html
<table>
{% for row in data %}
    <tr class="{% if forloop.counter0|divisibleby:2 %}even{% else %}odd{% endif %}">
        <td>{{ forloop.counter }}</td>
        <td>{{ row.name }}</td>
        <td>{{ row.value }}</td>
    </tr>
{% endfor %}
</table>
```

## 4.4 其他常用标签
```html
{# 1. 单行注释 #}

{% comment %}
    2. 多行注释
    这里的所有内容都不会被渲染到HTML中
{% endcomment %}

{# 3. 防止CSRF攻击（表单中使用） #}
<form method="POST">
    {% csrf_token %}
    <input type="text" name="username">
    <button type="submit">提交</button>
</form>

{# 4. URL反向解析 #}
<a href="{% url 'blog:article-detail' id=article.id %}">查看文章</a>
{{# blog: 命名空间, article-detail: 路由名称 #}}

{# 5. with：给变量起别名 #}
{% with total=items|length %}
    <p>共 {{ total }} 条记录</p>
{% endwith %}

{# 6. now：显示当前时间 #}
{% now "Y-m-d H:i" %}

{# 7. lorem：生成随机文本（开发测试用） #}
{% lorem 3 p %}
```

# 五、模板继承：DRY原则的体现
## 5.1 为什么不重复写HTML骨架
一个网站的所有页面通常有相同的头部（导航栏）、底部（页脚）、侧边栏。如果在每个HTML文件中都复制粘贴这些公共部分，一旦导航栏要加一个新链接，就需要改几十个文件——这完全违背了**DRY（Don't Repeat Yourself）**原则。

**模板继承**解决了这个问题：定义一个**母版模板**包含公共部分，每个页面只需要定义自己独有的内容。

## 5.2 block / extends / include
**母版模板** `blog/templates/blog/base.html`：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}我的博客{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>
    <!-- 公共导航栏 -->
    <nav>
        <a href="/">首页</a>
        <a href="/blog/">博客</a>
        <a href="/about/">关于</a>
    </nav>

    <!-- 动态内容区域（子模板填充这里） -->
    <main>
        {% block content %}
        <!-- 子模板的内容会插入到这里 -->
        {% endblock %}
    </main>

    <!-- 公共页脚 -->
    <footer>
        <p>&copy; 2026 我的博客</p>
    </footer>

    <!-- 公共JS -->
    <script src="{% static 'js/main.js' %}"></script>

    {% block extra_js %}
    <!-- 子模板可以在这里添加额外的JS -->
    {% endblock %}
</body>
</html>
```

**子模板** `blog/templates/blog/index.html`：
```html
{% extends 'blog/base.html' %}  <!-- ← 声明继承哪个母版 -->

{% block title %}首页 - 我的博客{% endblock %}  <!-- 覆盖母版的title块 -->

{% block content %}
    <!-- 这里写首页独有的内容 -->
    <h1>最新文章</h1>
    {% for article in articles %}
        <article>
            <h2>{{ article.title }}</h2>
            <p>{{ article.summary }}</p>
        </article>
    {% endfor %}
{% endblock %}

{% block extra_js %}
    <script>
        console.log("首页专属JS");
    </script>
{% endblock %}
```

**渲染结果**：最终生成的HTML会自动拼接——母版提供骨架，子模板的`{% block %}`内容替换母版中对应位置。用户看到的是一个完整的HTML页面。

**Python类比**：模板继承就像Python的类继承——`base.html`是基类，`index.html`是子类，`{% block %}`是可被覆盖的方法。

## 5.3 include：插入公共组件
除了继承，`{% include %}`可以插入一个单独的公共HTML片段：

```html
<!-- 侧边栏公共组件：blog/templates/blog/_sidebar.html -->
<aside>
    <h3>热门文章</h3>
    <ul>
        {% for article in hot_articles %}
        <li><a href="{{ article.get_absolute_url }}">{{ article.title }}</a></li>
        {% endfor %}
    </ul>
</aside>

<!-- 在任何模板中插入这个侧边栏 -->
{% include 'blog/_sidebar.html' %}
<!-- 或者带参数传递 -->
{% include 'blog/_sidebar.html' with show_count=5 %}
```

**extends vs include**：
| 特性 | extends | include |
|------|---------|---------|
| 用途 | 继承骨架 | 插入组件 |
| 关系 | 父子关系 | 引用关系 |
| 多块内容 | 可以 | 不可以 |
| 使用场景 | 页面的主体框架 | 侧边栏、导航、页脚等公共组件 |

# 六、静态文件：CSS/JS/图片
## 6.1 静态文件配置
静态文件（static files）是指不会动态变化的文件——CSS样式表、JavaScript脚本、图片、字体等。

**目录结构**：
```
blog/
├── static/                   # 静态文件目录（需手动创建）
│   └── blog/                 # 加应用名命名空间
│       ├── css/
│       │   └── style.css
│       ├── js/
│       │   └── main.js
│       └── images/
│           └── logo.png
├── templates/
└── ...
```

## 6.2 模板中引入静态文件
```html
{% load static %}  <!-- 先加载static标签库 -->

<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="{% static 'blog/css/style.css' %}">
</head>
<body>
    <img src="{% static 'blog/images/logo.png' %}" alt="Logo">

    <script src="{% static 'blog/js/main.js' %}"></script>
</body>
</html>
```

**`{% static %}`做了什么**：它在开发环境中指向你的`static/`目录，在生产环境中指向Nginx配置的静态文件路径。你不需要关心文件的具体物理路径，把路径管理交给Django和部署工具。

## 6.3 全局静态文件
除了应用级别的静态文件，也可以有项目级别的全局静态文件（如全局CSS框架）：

```
myblog/
├── static/                   # 项目级静态文件目录
│   ├── css/
│   │   └── bootstrap.min.css
│   └── js/
│       └── jquery.min.js
├── blog/
└── ...
```

在`settings.py`中配置全局静态文件目录：
```python
STATICFILES_DIRS = [
    BASE_DIR / 'static',  # 项目根目录下的 static/
]
```

# 七、自定义模板标签与过滤器
## 7.1 为什么需要自定义
Django内置了常用的过滤器，但你的项目可能需要特定业务逻辑的过滤器。比如：把数字转为"万"为单位、计算文章阅读时长的估计值、根据用户角色显示不同的标签。

## 7.2 创建自定义过滤器
在应用目录下创建`templatetags/`包（注意：必须叫这个名字，且必须有`__init__.py`）：

```
blog/
├── templatetags/             # 模板标签目录
│   ├── __init__.py           # 空文件
│   └── blog_tags.py          # 自定义标签文件
└── ...
```

```python
# blog/templatetags/blog_tags.py
from django import template

register = template.Library()

@register.filter(name='wan')
def to_wan(value):
    """将数字转为万为单位"""
    try:
        value = int(value)
        if value >= 10000:
            return f"{value / 10000:.1f}万"
        return str(value)
    except (ValueError, TypeError):
        return value

@register.filter(name='estimate_read_time')
def estimate_read_time(content, words_per_minute=300):
    """估算阅读时间（分钟）"""
    if not content:
        return "不到1分钟"
    # 假设中文字符数 ≈ 词数
    char_count = len(content)
    minutes = char_count / words_per_minute
    if minutes < 1:
        return "不到1分钟"
    return f"{minutes:.0f}分钟"

@register.simple_tag(name='current_year')
def current_year():
    """返回当前年份"""
    from datetime import datetime
    return datetime.now().year

@register.inclusion_tag('blog/_hot_articles.html')
def show_hot_articles(count=5):
    """渲染热门文章组件"""
    from blog.models import Article
    hot_articles = Article.objects.order_by('-views')[:count]
    return {'hot_articles': hot_articles, 'count': count}
```

## 7.3 在模板中使用自定义标签
```html
{% load blog_tags %}  <!-- 加载自定义标签库 -->

<!-- 使用自定义过滤器 -->
<p>阅读量：{{ article.views|wan }}</p>
<p>预计阅读时间：{{ article.content|estimate_read_time:200 }}</p>

<!-- 使用自定义标签 -->
<footer>
    <p>&copy; {% current_year %} 我的博客</p>
</footer>

<!-- 使用inclusion_tag：直接渲染一个完整组件 -->
{% show_hot_articles 10 %}
```

> 过滤器 vs 简单标签 vs inclusion_tag：
> - **过滤器**：处理单个值并返回（`{{ value|filter_name }}`）
> - **简单标签**：返回任意数据（`{% tag_name %}`）
> - **inclusion_tag**：渲染整个HTML模板片段

# 八、常见误区与避坑指南
1.  **模板目录结构不对**：Django找模板的默认路径是`app/templates/`。如果你的模板在`app/templates/app/`，视图中要写`render(request, 'app/index.html', ...)`——要加应用名前缀。

2.  **忘记load static**：使用`{% static %}`前必须先`{% load static %}`。这是新手最常见的错误，报错会提示无效的模板标签。

3.  **safe过滤器的滥用**：`{{ content|safe }}`会让Django不转义HTML，这意味着如果content中包含用户输入的`<script>恶意代码</script>`，它会被直接执行（XSS攻击）。只有在你100%确定内容是安全的（如你自己数据库中的Markdown渲染结果），才使用safe。

4.  **extends必须放在模板第一行**：`{% extends 'base.html' %}`必须是模板文件的第一行非注释内容，否则模板渲染会出错。

5.  **在for循环中修改被遍历的数据**：Django模板不应该修改数据，只负责展示。如果你需要在遍历时修改数据，在视图函数中提前处理好再传给模板。

6.  **模板变量和Python变量语法混淆**：Django模板中访问列表索引用`list.0`而不是`list[0]`，访问字典键用`dict.key`而不是`dict['key']`。

# 九、核心总结：模板语法速查表
## 变量与过滤器
| 语法 | 含义 | 示例 |
|------|------|------|
| `{{ var }}` | 输出变量 | `{{ name }}` |
| `{{ var\|filter }}` | 应用过滤器 | `{{ name\|upper }}` |
| `{{ var\|filter:arg }}` | 带参数的过滤器 | `{{ time\|date:"Y-m-d" }}` |
| `{{ list.0 }}` | 列表索引 | `{{ items.0 }}` |
| `{{ dict.key }}` | 字典查询 | `{{ user.name }}` |

## 流程控制标签
| 标签 | 语法 |
|------|------|
| if | `{% if condition %}...{% elif %}...{% else %}...{% endif %}` |
| for | `{% for item in list %}...{% empty %}...{% endfor %}` |
| forloop.counter | 循环计数（从1开始） |

## 模板继承
| 标签 | 位置 | 含义 |
|------|------|------|
| `{% extends 'base.html' %}` | 子模板第一行 | 继承母版 |
| `{% block name %}...{% endblock %}` | 母版+子模板 | 定义/覆盖内容块 |
| `{% include 'component.html' %}` | 任意位置 | 插入公共组件 |

## 静态文件
| 标签 | 示例 |
|------|------|
| `{% load static %}` | 加载静态文件标签库 |
| `{% static 'app/css/style.css' %}` | 引用静态文件 |

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
