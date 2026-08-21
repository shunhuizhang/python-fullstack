
# Python全栈入门到实战【Django篇 01】Django概述与第一个项目，从零搭建Web开发环境
上一篇《JavaScript篇 16》中，我们用JavaScript完成了交互式数据看板——从API获取数据、动态渲染页面、响应用户交互。但你是否注意到了一个问题：那个项目用的是`jsonplaceholder`的在线API，数据是别人提供的。作为一名全栈开发者，你最终需要**自己写后端接口**——自己定义数据模型、自己写业务逻辑、自己设计API、自己管理数据库。

而Django，就是Python世界中最强大、最成熟的Web框架。从这一刻起，我们将正式进入**Django篇**——在这里，之前学过的所有知识开始融合：Python基础语法是写Django的"编程语言"，MySQL数据库是Django的"数据仓库"，HTML/CSS/JavaScript是Django渲染的"前端输出"。Django像一个统帅，把这些技术整合成一个完整的Web应用。

本文为Python全栈开发者量身打造，采用"概念讲解+逐行代码实操"的教学模式。从Django的定位讲起，清晰解释MVT架构原理，手把手带你搭建虚拟环境、安装Django、创建第一个项目和应用，逐行解析每个生成的文件的作用。学完这一篇，你就拥有了一个可以运行起来的Django项目骨架。

本节核心学习内容：
1.  Django是什么：为什么Python开发Web首选Django
2.  MVT架构：Model-View-Template的分工与请求处理流程
3.  开发环境搭建：虚拟环境（venv）创建与激活
4.  安装Django：pip安装与版本验证
5.  第一个Django项目：startproject启动与项目目录逐行解析
6.  第一个Django应用：startapp创建应用与应用目录解析
7.  运行开发服务器：runserver启动与浏览器访问
8.  settings.py核心配置项快速了解
9.  Django vs Flask vs FastAPI选型对比

# 一、Django是什么
## 1.1 Django的定位
Django是Python生态中**功能最全、使用最广、生态最成熟**的Web框架。它的核心设计理念是"**功能齐全、开箱即用**"——你需要用户认证功能？内置了。你需要后台管理系统？内置了。你需要ORM操作数据库？内置了。你需要表单验证？内置了。你需要防御CSRF攻击？内置了。

这种"全包含"的设计哲学，让Django成为企业级Web应用开发的首选。你不需要到处拼装第三方库，Django自己就是一个完整的"全栈框架"。

**Python类比**：如果FastAPI是一把轻便的瑞士军刀（轻量、灵活、专注API），Django就是一座功能齐全的工厂（什么都有，拿来就用）。Flask介于两者之间——轻量但需要你自己装插件。

## 1.2 Django能做哪些项目
Django广泛应用于多种Web场景：

| 场景 | 实例 | 知名企业使用 |
|------|------|------------|
| 内容管理系统 | CSDN博客的后台管理 | — |
| 电商平台 | 后台商品管理+订单处理+支付 | Instagram、Pinterest |
| 社交平台 | 用户系统+动态流+评论 | Disqus |
| 数据分析后台 | 带权限的内部管理面板 | Mozilla、NASA |
| REST API | Django REST Framework | Instagram API |
| 新闻/门户网站 | 频道分类+搜索+SEO | The Washington Times |

> **一个事实**：Instagram的后端就是Django。一个支撑10亿+用户的应用稳定运行在Django上，足以证明它的可靠性和扩展性。

## 1.3 本专栏为什么选Django
作为全栈开发者，你有三个主流的Python Web框架可选：

| 框架 | 定位 | 优点 | 缺点 |
|------|------|------|------|
| **Django** | 重量级全栈框架 | 功能齐全、ORM强大、Admin开箱即用、文档完善 | 学习曲线陡、灵活性相对低 |
| **Flask** | 轻量级微框架 | 灵活自由、学习简单 | 什么都要自己装插件 |
| **FastAPI** | 现代高性能API框架 | 异步+类型提示+自动文档、性能最高 | 专注API、模板/Admin需自己搭建 |

Django最适合全栈初学者，因为它**什么都有**——你不需要在学框架的同时还要去评估和选择各种第三方插件。先学通Django，把Web开发的全套流程（数据模型→业务逻辑→模板渲染→API→部署）吃透，之后再学Flask/FastAPI会发现"原来就是精简版的Django"。

# 二、Django的MVT架构
## 2.1 MVT vs MVC
大多数Web框架使用**MVC**架构（Model-View-Controller），Django使用的是**MVT**（Model-View-Template）。两者本质类似，只是名称不同：

| MVT概念 | MVC对应 | 职责 | 生活类比 |
|---------|---------|------|----------|
| **Model（模型）** | Model | 定义数据结构，操作数据库 | 仓库管理员（管理货物进出） |
| **View（视图）** | Controller | 处理业务逻辑，决定返回什么数据 | 餐厅后厨（根据订单做菜） |
| **Template（模板）** | View | 负责数据展示，生成HTML | 摆盘装饰（把菜呈现给顾客） |

## 2.2 Django的请求处理流程
理解这个流程是理解Django的前提：

```
用户浏览器
    │
    ├── 1. 发送HTTP请求（如 GET /articles/）
    │
    ▼
Django URL路由器（urls.py）
    │
    ├── 2. 匹配URL规则，找到对应的View
    │
    ▼
View（视图函数 views.py）
    │
    ├── 3. 从Model获取数据（ORM查询数据库）
    ├── 4. 处理业务逻辑（筛选、排序、分页）
    ├── 5. 将数据传递给Template
    │
    ▼
Template（模板文件 .html）
    │
    ├── 6. 用模板语法渲染数据，生成HTML
    │
    ▼
返回HTTP Response到浏览器
    │
    ├── 7. 用户看到页面
```

**四行总结**：
- **URL**：路径到视图的映射表
- **View**：接收请求→处理数据→返回响应
- **Model**：Python类对应数据库表，操作数据库
- **Template**：HTML模板，用变量和标签动态生成页面

# 三、开发环境搭建：虚拟环境
## 3.1 为什么需要虚拟环境
在真实的Python开发中，不同项目可能需要不同版本的Django和其他依赖包。虚拟环境（virtual environment）为每个项目创建**独立的Python包环境**，项目A装Django 5.0，项目B装Django 4.2，互不干扰。

这和Python学习的"全局环境"不同——在基础篇中学习Python语法时，你直接在系统Python环境中运行代码。但在正式项目开发中，**每个项目都应该有自己的虚拟环境**。

## 3.2 创建虚拟环境
在你的项目目录中打开命令行（终端），执行以下命令：

```bash
# 创建一个名为 venv 的虚拟环境目录
python -m venv venv
```

命令解释：
- `python -m venv`：调用Python的venv模块
- 最后一个`venv`：虚拟环境的目录名（可以改成任何名字，但venv是社区约定）

执行后会在当前目录生成一个`venv/`文件夹，包含独立的Python解释器副本和pip包管理器。

## 3.3 激活虚拟环境
虚拟环境创建后需要先**激活**才能使用：

**Windows（PowerShell/CMD）**：
```bash
venv\Scripts\activate
```

**Windows（PowerShell执行策略问题）**：
如果遇到"无法加载文件xxx.ps1，因为在此系统上禁止运行脚本"，先执行：
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
然后再`venv\Scripts\activate`。

**Mac/Linux**：
```bash
source venv/bin/activate
```

激活成功后，命令行前面会出现`(venv)`标识：
```
(venv) D:\django-projects\myblog>
```

这表示你已经在虚拟环境中，之后所有`pip install`的包都会被安装到这个虚拟环境中，不影响系统Python和全局项目。

## 3.4 退出虚拟环境
```bash
deactivate
```

# 四、安装Django
## 4.1 pip安装
激活虚拟环境后，用pip安装Django：

```bash
pip install django
```

这会安装Django的最新稳定版。如果你需要指定版本（生产环境常用）：

```bash
pip install django==5.0
```

## 4.2 验证安装
```bash
# 查看Django版本
python -m django --version

# 或者进入Python交互式环境
python -c "import django; print(django.get_version())"
```

## 4.3 查看Django提供的命令
```bash
django-admin --help
```

你会看到一个命令列表，包括`startproject`（创建项目）、`startapp`（创建应用）、`runserver`（启动开发服务器）等，我们接下来会一个个用到。

# 五、创建第一个Django项目
## 5.1 什么是Django项目（Project）
一个**项目（Project）** 是一个完整的Django网站实例。它包含所有配置（settings.py）、URL总路由（urls.py）、以及一个或多个**应用（App）**。一个域名通常对应一个项目。

简单类比：**项目 = 整个网站，应用 = 网站中的一个功能模块**。比如一个完整的博客网站是一个Django项目，里面可能有"文章管理"应用、"用户系统"应用、"评论"应用等。

## 5.2 startproject创建项目
在你想存放项目的目录下执行：

```bash
django-admin startproject myblog
```

这会在当前目录创建一个`myblog/`文件夹，结构如下：

```
myblog/
├── manage.py                      # 项目管理入口（核心工具）
└── myblog/                        # 项目配置目录（和项目同名）
    ├── __init__.py                 # 空文件，告诉Python这是一个包
    ├── settings.py                 # 项目全局配置文件
    ├── urls.py                     # URL路由配置（总入口）
    ├── asgi.py                     # ASGI配置（异步部署用）
    └── wsgi.py                     # WSGI配置（传统部署用）
```

## 5.3 逐行解析项目文件

### manage.py：项目管理入口
`manage.py`是Django项目的**命令行工具**，通过它执行所有项目管理操作。你不需要修改它。

```bash
# 常用manage.py命令
python manage.py runserver     # 启动开发服务器
python manage.py startapp      # 创建应用
python manage.py makemigrations # 生成数据库迁移文件
python manage.py migrate       # 执行数据库迁移
python manage.py createsuperuser # 创建管理员账号
python manage.py shell         # 进入Django的交互式Shell
python manage.py test          # 运行测试
```

### settings.py：全局配置
这是整个项目最重要的文件，所有配置都在这里。我们只看最核心的几个：

```python
# settings.py 关键配置项

# 1. SECRET_KEY：项目密钥（生产环境必须保密，用于加密签名）
SECRET_KEY = 'django-insecure-xxxxxxxxxxxxxxxxxxxxxxxxxxxxx'

# 2. DEBUG：调试模式（开发时True，上线必须改为False）
DEBUG = True

# 3. ALLOWED_HOSTS：允许访问的域名/IP（DEBUG=True时可留空，上线必须配置）
ALLOWED_HOSTS = []

# 4. INSTALLED_APPS：已安装的应用列表
INSTALLED_APPS = [
    'django.contrib.admin',      # 后台管理
    'django.contrib.auth',       # 用户认证
    'django.contrib.contenttypes', # 内容类型
    'django.contrib.sessions',   # 会话管理
    'django.contrib.messages',   # 消息框架
    'django.contrib.staticfiles', # 静态文件管理
]

# 5. DATABASES：数据库配置（默认SQLite）
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# 6. TEMPLATES：模板引擎配置
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],  # 模板文件目录
        'APP_DIRS': True,  # 在每个应用的templates目录中查找模板
    },
]

# 7. LANGUAGE_CODE和TIME_ZONE：国际化和时区
LANGUAGE_CODE = 'zh-hans'  # 中文
TIME_ZONE = 'Asia/Shanghai'  # 东八区（北京时间）
```

> 你暂时不需要记住每个配置项的含义，后面的文章会在使用到时逐一讲解。现阶段只需要知道settings.py是项目的"大脑"，所有配置都在这里。

### urls.py：路由总入口
```python
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),  # 映射 /admin/ 到Django后台管理
]
```

这是项目的**URL路由表**。`path('admin/', ...)`的意思是：当用户访问`/admin/`时，由Django内置的Admin模块处理。随着项目的开发，你会在这里添加越来越多的路由规则。

### __init__.py
空文件，它的存在只是为了告诉Python：`myblog/`这个目录是一个Python包（package），可以被其他模块导入。

### wsgi.py 和 asgi.py
这两个文件是部署相关的配置文件：
- `wsgi.py`：用于传统的同步Web服务器（如Gunicorn）
- `asgi.py`：用于支持异步的Web服务器（如Daphne、Uvicorn）

现在暂时不需要管它们，部署篇会详细讲解。

# 六、创建第一个Django应用
## 6.1 什么是应用（App）
一个Django项目由多个**应用（App）** 组成。应用是一个功能模块，通常对应项目中的一个业务板块。比如一个博客项目：

```
myblog/（项目）
├── articles/（应用：文章管理）
├── users/（应用：用户系统）
├── comments/（应用：评论功能）
└── search/（应用：搜索功能）
```

每个应用有独立的模型、视图、模板、URL配置，可以独立开发，通过项目组合在一起。

## 6.2 startapp创建应用
在项目目录下（`manage.py`所在目录），执行：

```bash
# 切换到项目目录
cd myblog

# 创建名为 blog 的应用
python manage.py startapp blog
```

创建后的目录结构：
```
myblog/
├── manage.py
├── myblog/           # 项目配置目录
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── ...
└── blog/             # 新创建的blog应用
    ├── __init__.py
    ├── admin.py      # Admin后台管理配置
    ├── apps.py       # 应用配置
    ├── models.py     # 数据模型（建表用）
    ├── views.py      # 视图函数
    ├── urls.py       # 应用级URL路由（需要自己创建）
    ├── tests.py      # 单元测试
    └── migrations/   # 数据库迁移文件目录
        └── __init__.py
```

## 6.3 逐行解析应用文件

| 文件 | 作用 | 何时修改 |
|------|------|---------|
| `models.py` | 定义数据模型（Python类对应数据库表） | 每当你需要存储新类型的数据 |
| `views.py` | 编写视图函数，处理请求并返回响应 | 每当你需要处理一个URL请求 |
| `urls.py` | 应用内部的路由映射（需手动创建） | 新增URL路径时 |
| `admin.py` | 将模型注册到Django后台管理 | 需要后台管理某个数据模型时 |
| `apps.py` | 应用的配置信息 | 一般不需要改 |
| `tests.py` | 编写单元测试 | 编写测试代码时 |
| `migrations/` | 存放数据库迁移文件 | Django自动生成，一般不需要手动改 |

## 6.4 注册应用
创建应用后，需要将应用注册到项目中——在`settings.py`的`INSTALLED_APPS`列表中添加应用名称：

```python
# myblog/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # ↓ 添加你自己的应用 ↓
    'blog',                     # 或 'blog.apps.BlogConfig'（更规范的写法）
]
```

> **注意**：Django是靠`INSTALLED_APPS`来发现应用的。如果忘记注册，Django不会扫描到这个应用中的models、templates等资源。

## 6.5 编写第一个视图
打开`blog/views.py`，编写第一个视图函数：

```python
# blog/views.py
from django.http import HttpResponse

def index(request):
    """首页视图：返回一句欢迎语"""
    return HttpResponse("你好！欢迎来到Django博客！")
```

**代码解析**：
- `request`：Django自动传入的请求对象，包含用户请求的所有信息（URL、方法、参数等）。这和JS中addEventListener的回调函数接收`event`对象是一样的道理。
- `HttpResponse`：返回一个HTTP响应给浏览器。`"你好！..."`这个字符串就是浏览器最终显示的内容。
- `index`：视图函数名（可以任意命名，但要见名知意）

## 6.6 配置应用级URL路由
在`blog/`目录下**手动创建**`urls.py`文件：

```python
# blog/urls.py（需要手动创建）
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),  # 访问 /blog/ 时调用 index 视图
]
```

然后，在项目的总路由中**引入**应用的URL：

```python
# myblog/urls.py
from django.contrib import admin
from django.urls import path, include  # 添加 include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),  # 将 /blog/ 开头的URL转发给 blog 应用处理
]
```

**路由分发逻辑**：
```
用户访问 /blog/ 
    → myblog/urls.py 匹配到 path('blog/', ...)
    → 将 /blog/ 之后的路径交给 blog/urls.py 处理
    → blog/urls.py 中 path('', ...) 匹配空路径
    → 调用 views.index() 视图
    → 返回 "你好！欢迎来到Django博客！"
```

这种"项目级路由→应用级路由"的分层设计，让每个应用管理自己的URL，项目只需用`include`导入，非常清晰。

# 七、运行开发服务器
## 7.1 启动服务器
在项目目录（`manage.py`所在位置）执行：

```bash
python manage.py runserver
```

启动成功后会看到：
```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

Django version 5.0, using settings 'myblog.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

## 7.2 浏览器访问
打开浏览器，访问以下地址：

| URL | 对应内容 |
|-----|---------|
| http://127.0.0.1:8000/ | 默认Django欢迎页（没有配置/路由） |
| http://127.0.0.1:8000/blog/ | 我们写的第一个视图："你好！欢迎来到Django博客！" |
| http://127.0.0.1:8000/admin/ | Django后台管理登录页（后面详讲） |

看到"你好！欢迎来到Django博客！"的那一刻，你的第一个Django应用就跑起来了！

## 7.3 runserver参数
```bash
# 指定端口
python manage.py runserver 8080        # 使用8080端口

# 指定IP和端口（让局域网内其他设备也能访问）
python manage.py runserver 0.0.0.0:8000

# 不自动重载
python manage.py runserver --noreload
```

> **注意**：`runserver`是**开发服务器**，仅用于本地开发和调试。它不是生产级别的Web服务器（性能、安全都不达标）。生产环境部署使用Gunicorn/Nginx，将在部署篇讲解。

# 八、创建应用的标准流程图
```
1. python manage.py startapp 应用名       # 创建应用
2. settings.py中INSTALLED_APPS注册该应用   # 注册应用
3. models.py中定义数据模型                  # 建表（下一篇讲）
4. views.py中编写视图函数                   # 处理逻辑
5. 创建应用级urls.py并配置路由              # URL映射
6. 项目级urls.py中include应用的URL         # 接入总路由
```

# 九、常见误区与避坑指南
1.  **忘记激活虚拟环境**：在PowerShell中直接`pip install django`会安装到全局Python中，而不是虚拟环境。记得先激活`(venv)`再操作。

2.  **混淆项目（project）和应用（app）**：项目是"网站整体"，应用是"功能模块"。一个项目可以有多个应用，一个应用可以被多个项目复用。

3.  **创建应用后忘记注册**：`startapp`只是创建了目录结构，Django不会自动发现它。必须在`INSTALLED_APPS`中注册，否则这个应用的models、templates等都不会被加载。

4.  **修改了settings.py不生效**：`runserver`默认会自动监测代码变化并重启。但如果改了settings.py有时候需要手动重启（Ctrl+C停止，再重新启动）。

5.  **把业务逻辑写在urls.py中**：urls.py只负责路由映射，视图函数写在views.py中。保持每个文件的职责单一。

6.  **在应用目录下忘记创建urls.py**：Django不会自动为应用创建urls.py，你需要手动创建，并在项目urls.py中用`include()`导入。

# 十、核心总结
1.  Django是Python最强大的Web框架，遵循MVT架构（Model-View-Template），功能齐全、开箱即用。
2.  每个Django项目都应该创建独立的虚拟环境（venv），隔离项目依赖。
3.  项目（Project）是网站的顶层配置，应用（App）是独立的功能模块。一个项目包含多个应用。
4.  开发流程：创建项目 → 创建应用 → 注册应用 → 编写视图 → 配置路由 → 运行开发服务器。
5.  `runserver`是开发服务器，仅用于本地开发调试，不能用于生产环境。

# 十一、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
