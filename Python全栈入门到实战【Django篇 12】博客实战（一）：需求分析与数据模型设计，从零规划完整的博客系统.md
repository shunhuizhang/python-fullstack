
# Python全栈入门到实战【Django篇 12】博客实战（一）：需求分析与数据模型设计，从零规划完整的博客系统
前面11篇文章，我们系统学习了Django的框架基础和DRF的API开发。从本篇开始，我们将进入**实战阶段**——用Django从零构建一个完整的博客系统。这不是"演示几个功能"的教学Demo，而是从**需求分析 → 模型设计 → URL规划 → 视图实现 → 模板编写 → 评论/搜索/用户系统**一系列完整的项目开发流程。

本篇作为博客项目的开篇，将带你完成项目的**需求分析**和**数据模型设计**。需求分析决定了"要做什么"，模型设计决定了"数据怎么存"——这两个环节是项目的根基，根基打牢了，后面的开发就是一件按部就班的事情。

本文采用"**步骤式项目驱动**"教学法，你可以边看边跟着操作。每做一步，理解一步的原因。完成这6篇博客实战后，你将拥有一个可以直接发布上线的Django博客系统。

本节核心学习内容：
1.  博客系统需求分析：功能清单、页面结构、URL设计
2.  项目初始化：创建Django项目、配置MySQL、管理应用划分
3.  博客核心模型设计：文章、分类、标签、评论的数据结构
4.  模型关系设计：外键关联、多对多关联、related_name命名
5.  Admin后台配置：让模型在管理后台中直观可操作
6.  初始数据填充：fixtures创建测试数据

# 一、需求分析
## 1.1 功能清单
在动手写代码之前，先明确博客系统需要哪些功能：

| 模块 | 功能 | 谁可以用 |
|------|------|---------|
| 文章浏览 | 首页文章列表、文章详情页、按分类/标签筛选、归档 | 所有人（包括未登录） |
| 文章发布 | 创建文章（Markdown编辑器）、编辑文章、删除文章 | 登录用户 |
| 评论系统 | 发表评论、回复评论、删除自己的评论 | 登录用户 |
| 用户系统 | 注册、登录、退出、个人中心 | 所有人 |
| 搜索 | 搜索标题和内容 | 所有人 |
| 后台管理 | 管理文章/评论/用户/分类/标签 | 管理员 |

## 1.2 数据实体分析
从功能清单中提取出需要存储的数据实体（对应数据库的表）：

```
文章（Article）  ───── FK ───→  分类（Category）
    │
    ├── M2M ───→  标签（Tag）
    │
    └── FK ───→  作者（User，Django内置）
                           │
评论（Comment） ─── FK ───→ 文章
                           │
                           └── FK ───→ 用户（评论者）
```

## 1.3 URL设计
在写代码之前先画好URL地图，你就能清晰地看到整个项目的"面"：

```
首页：
  /                                          首页（最新文章列表）

文章：
  /articles/                                 文章列表（分页）
  /articles/<slug>/                           文章详情
  /articles/create/                           发布文章
  /articles/<slug>/edit/                      编辑文章
  /articles/<slug>/delete/                    删除文章

分类与标签：
  /category/<slug>/                           按分类浏览文章
  /tag/<slug>/                                按标签浏览文章

搜索：
  /search/?q=keyword                          搜索结果

用户：
  /users/register/                           注册
  /users/login/                               登录
  /users/logout/                              退出
  /users/profile/                             个人中心
  /users/change-password/                     修改密码
```

# 二、项目初始化
## 2.1 创建项目和虚拟环境
```bash
# 1. 创建项目目录
mkdir blog_project
cd blog_project

# 2. 创建虚拟环境
python -m venv venv
venv\Scripts\activate  # Windows
# source venv/bin/activate  # Mac/Linux

# 3. 安装依赖
pip install django==5.0
pip install pillow  # 图片处理
pip install markdown  # Markdown渲染
pip install Pygments  # 代码语法高亮（配合markdown的codehilite扩展）
```

```bash
# 4. 创建Django项目
django-admin startproject blog_project .
# 注意最后的 . 表示在当前目录创建项目（不会多嵌套一层目录）

# 5. 创建应用
python manage.py startapp blog
python manage.py startapp users
```

## 2.2 配置settings.py
```python
# blog_project/settings.py
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = 'django-insecure-你的密钥'  # 生产环境从环境变量读取
DEBUG = True
ALLOWED_HOSTS = []

# 注册应用
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 第三方
    'django_markdown_deux',
    # 自定义
    'blog',
    'users',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'blog_project.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],  # 项目级模板目录
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

# 数据库：使用MySQL（数据库篇已学过MySQL的同学可以切换）
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
# 如果要切换MySQL（取消注释并安装 mysqlclient）：
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.mysql',
#         'NAME': 'blog_db',
#         'USER': 'root',
#         'PASSWORD': 'your_password',
#         'HOST': 'localhost',
#         'PORT': '3306',
#         'OPTIONS': {
#             'charset': 'utf8mb4',
#         },
#     }
# }

LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_TZ = True

STATIC_URL = 'static/'
STATICFILES_DIRS = [BASE_DIR / 'static']  # 项目级静态文件

MEDIA_URL = 'media/'
MEDIA_ROOT = BASE_DIR / 'media'

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

## 2.3 项目URL总路由
```python
# blog_project/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
    path('users/', include('users.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

# 三、核心数据模型设计
## 3.1 分类模型
```python
# blog/models.py
from django.db import models
from django.urls import reverse

class Category(models.Model):
    """文章分类"""
    name = models.CharField(max_length=50, verbose_name='分类名称')
    slug = models.SlugField(max_length=50, unique=True, verbose_name='URL别名')
    description = models.TextField(blank=True, verbose_name='分类描述')

    class Meta:
        verbose_name = '分类'
        verbose_name_plural = '分类列表'
        ordering = ['name']

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        """返回分类页的URL（模板和视图中使用）"""
        return reverse('blog:category-detail', kwargs={'slug': self.slug})
```

**设计说明**：`slug`字段用于URL中的友好短标签（如`/category/python-tutorial/`而不是`/category/3/`）。`get_absolute_url()`是Django的约定——定义了"这个对象的URL是什么"，在Admin和CBV中自动使用。

## 3.2 标签模型
```python
class Tag(models.Model):
    """文章标签（通过ManyToMany与文章关联）"""
    name = models.CharField(max_length=30, verbose_name='标签名')
    slug = models.SlugField(max_length=30, unique=True, verbose_name='URL别名')

    class Meta:
        verbose_name = '标签'
        verbose_name_plural = '标签列表'
        ordering = ['name']

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        return reverse('blog:tag-detail', kwargs={'slug': self.slug})
```

## 3.3 文章模型（核心）
```python
from django.contrib.auth.models import User
from django.utils import timezone

class Article(models.Model):
    """文章（博客系统的核心模型）"""
    # 基本信息
    title = models.CharField(max_length=200, verbose_name='标题')
    slug = models.SlugField(max_length=200, unique=True, verbose_name='URL别名',
                           help_text='自动从标题生成，也可手动编辑')
    content = models.TextField(verbose_name='文章内容')

    # 分类与标签
    category = models.ForeignKey(
        Category,
        on_delete=models.PROTECT,  # 有文章的分类不能删除（保护）
        related_name='articles',
        verbose_name='所属分类'
    )
    tags = models.ManyToManyField(
        Tag,
        blank=True,
        related_name='articles',
        verbose_name='标签'
    )

    # 作者
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,  # 作者删除时，他的文章也删除
        related_name='articles',
        verbose_name='作者'
    )

    # 状态
    STATUS_CHOICES = [
        ('draft', '草稿'),
        ('published', '已发布'),
    ]
    status = models.CharField(
        max_length=20,
        choices=STATUS_CHOICES,
        default='draft',
        verbose_name='状态'
    )

    # 封面图
    cover = models.ImageField(
        upload_to='covers/%Y/%m/',
        blank=True,
        null=True,
        verbose_name='封面图片'
    )

    # 统计
    views = models.PositiveIntegerField(default=0, verbose_name='阅读量')

    # 时间
    created_at = models.DateTimeField(default=timezone.now, verbose_name='创建时间')
    updated_at = models.DateTimeField(auto_now=True, verbose_name='更新时间')

    class Meta:
        verbose_name = '文章'
        verbose_name_plural = '文章列表'
        ordering = ['-created_at']  # 按创建时间倒序
        indexes = [
            models.Index(fields=['-created_at']),  # 时间索引
            models.Index(fields=['status', '-created_at']),  # 状态+时间联合索引
        ]

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('blog:article-detail', kwargs={'slug': self.slug})

    def is_published(self):
        """便捷方法：判断是否已发布"""
        return self.status == 'published'

    def increase_views(self):
        """增加阅读量（使用F对象原子操作）"""
        from django.db.models import F
        Article.objects.filter(pk=self.pk).update(views=F('views') + 1)
        self.refresh_from_db()
```

**模型设计关键决策**：

| 设计决策 | 选择 | 原因 |
|---------|------|------|
| `on_delete=PROTECT`（分类） | 保护 | 有文章时不能删除分类，防止文章变成"孤儿" |
| `on_delete=CASCADE`（作者） | 级联删除 | 作者注销时，他的文章一并删除（符合大多数博客的预期） |
| `slug`为主键进行URL访问 | 唯一Slug | URL友好（`/articles/django-tutorial/`比`/articles/42/`更有SEO价值） |
| `status`字段 | 草稿/已发布 | 支持"保存为草稿"功能，不直接暴露 |
| `views`使用F对象 | 原子自增 | 防止高并发时阅读量丢失 |

## 3.4 评论模型
```python
class Comment(models.Model):
    """评论"""
    article = models.ForeignKey(
        Article,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name='所属文章'
    )
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name='评论者'
    )
    # 支持回复：parent指向被回复的评论（自引用外键）
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='replies',
        verbose_name='回复的评论'
    )
    content = models.TextField(verbose_name='评论内容')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='评论时间')
    is_active = models.BooleanField(default=True, verbose_name='是否可见')

    class Meta:
        verbose_name = '评论'
        verbose_name_plural = '评论列表'
        ordering = ['created_at']

    def __str__(self):
        return f'{self.user.username} 评论了 {self.article.title}'
```

**自引用外键的设计**：`parent = ForeignKey('self', ...)`让评论可以回复其他评论。`parent=None`表示这是一条顶级评论（不是回复），`parent=某评论`表示这是对该评论的回复。通过`related_name='replies'`可以获取一条评论的所有回复。

## 3.5 模型关系图（可视化总结）
```
Article ─── FK(on_delete=PROTECT) ───→ Category
    │                                    ├── name, slug, description
    │                                    └── related_name='articles'
    │
    ├── M2M(blank=True) ───→ Tag
    │                         ├── name, slug
    │                         └── related_name='articles'
    │
    ├── FK(on_delete=CASCADE) ───→ User (Django内置)
    │                               └── related_name='articles'
    │
    └── 自有字段：title, slug, content, status, cover, views, created_at, updated_at

Comment ─── FK(on_delete=CASCADE) ───→ Article
    │                                    └── related_name='comments'
    ├── FK(on_delete=CASCADE) ───→ User (评论者)
    │                               └── related_name='comments'
    └── FK(on_delete=CASCADE) ───→ self (parent评论)
                                    └── related_name='replies'
```

# 四、Admin后台配置
```python
# blog/admin.py
from django.contrib import admin
from .models import Category, Tag, Article, Comment

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'article_count']
    prepopulated_fields = {'slug': ('name',)}  # 输入name时自动生成slug
    search_fields = ['name']

    def article_count(self, obj):
        """显示该分类下的文章数量"""
        return obj.articles.count()

@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'article_count']
    prepopulated_fields = {'slug': ('name',)}

    def article_count(self, obj):
        return obj.articles.count()

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'category', 'author', 'status', 'views', 'created_at']
    list_filter = ['status', 'category', 'created_at']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'created_at'
    filter_horizontal = ['tags']  # 多对多使用横向选择器
    readonly_fields = ['views', 'created_at', 'updated_at']

    fieldsets = [
        ('基本信息', {'fields': ['title', 'slug', 'content']}),
        ('分类与标签', {'fields': ['category', 'tags']}),
        ('作者与状态', {'fields': ['author', 'status', 'cover']}),
        ('统计信息', {'fields': ['views'], 'classes': ['collapse']}),
        ('时间信息', {'fields': ['created_at', 'updated_at'], 'classes': ['collapse']}),
    ]

    def save_model(self, request, obj, form, change):
        """保存文章时自动设置作者"""
        if not change:  # 创建新文章
            obj.author = request.user
        super().save_model(request, obj, form, change)

@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['id', 'user', 'article', 'content_preview', 'created_at', 'is_active']
    list_filter = ['is_active', 'created_at']
    search_fields = ['content', 'user__username']
    actions = ['approve_comments', 'hide_comments']

    def content_preview(self, obj):
        return obj.content[:50] + ('...' if len(obj.content) > 50 else '')

    def approve_comments(self, request, queryset):
        queryset.update(is_active=True)
    approve_comments.short_description = '批量通过选中评论'

    def hide_comments(self, request, queryset):
        queryset.update(is_active=False)
    hide_comments.short_description = '批量隐藏选中评论'
```

# 五、创建并填充测试数据
## 5.1 执行数据库迁移
```bash
python manage.py makemigrations blog
python manage.py makemigrations users
python manage.py migrate
```

## 5.2 在Django Shell中创建测试数据
```bash
python manage.py shell
```

```python
from blog.models import Category, Tag, Article
from django.contrib.auth.models import User

# 1. 创建管理员用户（如果还没有）
User.objects.create_superuser('admin', 'admin@example.com', 'admin123')

# 2. 创建分类
python_cat = Category.objects.create(name='Python教程', slug='python', description='Python编程相关教程')
django_cat = Category.objects.create(name='Django框架', slug='django', description='Django Web框架教程')
frontend_cat = Category.objects.create(name='前端开发', slug='frontend', description='前端技术分享')

# 3. 创建标签
tags = []
for name in ['Python', 'Django', 'ORM', 'CBV', 'API', '全栈', '入门', '进阶', 'MySQL', 'JavaScript']:
    tag, _ = Tag.objects.get_or_create(name=name, defaults={'slug': name.lower()})
    tags.append(tag)

# 4. 创建示例文章
admin_user = User.objects.get(username='admin')

articles_data = [
    {'title': 'Python全栈开发从入门到精通指南', 'content': 'Python全栈开发涉及...（长内容）', 'category': python_cat, 'status': 'published'},
    {'title': 'Django模型层深度解析：ORM的20个高级技巧', 'content': 'Django ORM是...（长内容）', 'category': django_cat, 'status': 'published'},
    {'title': '类视图（CBV）最佳实践：从ListView到ModelViewSet', 'content': '类视图的设计哲学...（长内容）', 'category': django_cat, 'status': 'published'},
    {'title': '前端JavaScript异步编程：从回调到async/await', 'content': '回调→Promise→async/await...（长内容）', 'category': frontend_cat, 'status': 'published'},
    {'title': 'MySQL性能优化22条实战军规', 'content': '索引优化、查询优化...（长内容）', 'category': python_cat, 'status': 'draft'},
]

for data in articles_data:
    from django.utils.text import slugify
    article = Article.objects.create(
        title=data['title'],
        slug=slugify(data['title']),
        content=data['content'],
        category=data['category'],
        author=admin_user,
        status=data['status'],
        views=50,
    )
    article.tags.add(*tags[:3])  # 给每篇文章添加前3个标签
    print(f'创建文章：{article.title}')
```

## 5.3 创建超级管理员
```bash
python manage.py createsuperuser
# 输入：admin / admin@example.com / admin123
```

访问`http://127.0.0.1:8000/admin/`，用管理员账号登录，你能看到配置好的Admin界面——分类、标签、文章、评论四个模块，每个都有搜索、筛选、排序功能。

# 六、常见误区与避坑指南
1.  **slug字段忘记设置unique=True**：如果两个文章的slug相同（如两篇"Python教程"），URL会产生冲突。slug必须全局唯一。可以用`slug = models.SlugField(unique=True)`确保这一点。

2.  **ForeignKey的related_name命名冲突**：如果两个模型都使用`ForeignKey(User, related_name='comments')`（复数），Django会报错因为冲突。在每个外键中使用不同的related_name。

3.  **on_delete选择不当**：`on_delete=models.CASCADE`最常用（主记录删除时从记录也删），但分类这种"关键引用"适合用`PROTECT`（有子记录时不能删除）。

4.  **在Admin中把多对多字段当作普通下拉框**：如果有几十上百个标签，用`filter_horizontal`提供左右选择器体验更好。

5.  **shell中创建的数据在数据库中持久化，但重新创建数据库时需要重新生成**：建议把初始数据保存为fixture文件：`python manage.py dumpdata blog --output=blog/fixtures/initial_data.json`。

# 七、核心总结
## 项目初始化的文件结构
```
blog_project/
├── manage.py
├── blog_project/        # 项目配置
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── blog/                # 博客应用
│   ├── models.py        # ★ 本篇主要编写的文件
│   ├── admin.py         # ★ 本篇主要编写的文件
│   ├── views.py
│   └── urls.py
├── users/               # 用户应用
│   └── models.py
├── static/              # 静态文件
├── media/               # 上传文件
└── templates/           # 模板文件
```

## 下一篇预告
博客项目第二篇将实现**首页和文章列表**——从URL路由到视图函数到HTML模板的完整打通。做好模型设计后，视图和模板编写就顺畅多了。

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
