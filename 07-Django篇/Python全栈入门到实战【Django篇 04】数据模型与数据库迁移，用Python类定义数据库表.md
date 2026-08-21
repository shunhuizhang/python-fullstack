
# Python全栈入门到实战【Django篇 04】数据模型与数据库迁移，用Python类定义数据库表
上一篇《Django篇 03》中，我们掌握了Django模板系统——能用HTML模板动态渲染页面了。但还有一个关键问题没有解决：数据从哪来？目前所有传给模板的数据都是我们在视图中"硬编码"写死的（如`context = {'title': '首页'}`），真实项目的数据来自**数据库**。

传统开发中，操作数据库需要写SQL语句——建表用`CREATE TABLE`，查数据用`SELECT`，插入用`INSERT`。这种方式有两个问题：SQL语句散落在Python代码中难以维护；而且每次换数据库（从MySQL换成PostgreSQL），SQL语法可能有差异。

Django的**ORM（Object-Relational Mapping，对象关系映射）**彻底改变了这种开发方式。你不需要写一行SQL——用Python类来描述数据表结构，Django自动把类转化为数据库表，把Python方法调用转化为SQL语句。你操作Python对象，就等于操作数据库。作为Python全栈开发者，这是你从"手工写SQL"到"用对象抽象操作数据库"的重要升级。

本节核心学习内容：
1.  ORM是什么：Python类到数据库表的映射原理
2.  定义模型：常用字段类型、字段选项、Meta类
3.  字段关系：ForeignKey（一对多）、ManyToManyField（多对多）、OneToOneField（一对一）
4.  数据库迁移系统：makemigrations / migrate / sqlmigrate完整流程
5.  迁移文件的原理：正向执行与反向回滚
6.  在视图中使用模型：从数据库查询真实数据传给模板

# 一、ORM是什么
## 1.1 传统开发方式 vs ORM方式
**传统方式（写SQL）**：
```python
# 用Python的MySQL驱动执行SQL
import pymysql
conn = pymysql.connect(...)
cursor = conn.cursor()

# 建表
cursor.execute("""
    CREATE TABLE blog_article (
        id INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(200) NOT NULL,
        content TEXT,
        pub_date DATETIME DEFAULT CURRENT_TIMESTAMP
    )
""")

# 查询
cursor.execute("SELECT * FROM blog_article WHERE id = 1")
article = cursor.fetchone()

# 插入
cursor.execute(
    "INSERT INTO blog_article (title, content) VALUES (%s, %s)",
    ("文章标题", "文章内容")
)
```

**ORM方式（操作Python对象）**：
```python
# 定义模型 = 建表
class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    pub_date = models.DateTimeField(auto_now_add=True)

# 查询 = 操作Python对象
article = Article.objects.get(id=1)
print(article.title)

# 插入 = 创建Python对象
article = Article.objects.create(title="文章标题", content="文章内容")
```

> 同样是查询和插入，ORM方式用Python类和方法调用代替了SQL字符串。你不关心底层用的是MySQL还是PostgreSQL，ORM自动适配。

## 1.2 ORM的核心优势
| 优势 | 说明 |
|------|------|
| **不用写SQL** | 用Python语法操作数据库，减少SQL注入风险 |
| **数据库无关** | 换数据库只需改settings配置，代码不动 |
| **类型安全** | 字段类型在Python层面就做了校验 |
| **自动迁移** | 修改模型后自动生成数据库变更脚本 |
| **关联查询** | 跨表关联（外键/多对多）操作像操作对象属性一样简单 |

## 1.3 Django模型与数据库表的对应关系
```
一个Django Model类     →   数据库中的一张表
Model类的一个属性      →   表中的一个字段（列）
Model类的一个实例      →   表中的一行记录
```

# 二、定义第一个模型
## 2.1 models.py：数据表的蓝图
打开`blog/models.py`，定义文章模型：

```python
# blog/models.py
from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User

class Article(models.Model):
    """文章模型"""
    # 字段 = 表的列
    title = models.CharField(max_length=200, verbose_name="标题")
    content = models.TextField(verbose_name="内容")
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name="作者")
    pub_date = models.DateTimeField(default=timezone.now, verbose_name="发布时间")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="更新时间")
    is_published = models.BooleanField(default=True, verbose_name="是否发布")
    views = models.PositiveIntegerField(default=0, verbose_name="阅读量")

    class Meta:
        # 表级别的配置
        db_table = 'articles'                    # 自定义表名（默认: blog_article）
        ordering = ['-pub_date']                 # 默认排序：按发布时间倒序
        verbose_name = '文章'                    # 单数别名（Admin中显示）
        verbose_name_plural = '文章列表'         # 复数别名

    def __str__(self):
        """对象的字符串表示（Admin和调试中使用）"""
        return self.title
```

这个Python类被Django ORM自动翻译成以下SQL（大概是这样，实际细节略有不同）：
```sql
CREATE TABLE articles (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    author_id INTEGER NOT NULL REFERENCES auth_user(id),
    pub_date DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    is_published BOOL NOT NULL DEFAULT 1,
    views INTEGER UNSIGNED NOT NULL DEFAULT 0
);
```

## 2.2 常用模型字段类型
| 字段类型 | 数据库类型 | 适用场景 | Django特有参数 |
|---------|-----------|---------|---------------|
| `CharField` | `VARCHAR` | 短文本（标题、用户名） | `max_length`（必填） |
| `TextField` | `TEXT` | 长文本（文章内容、简介） | `blank=True`允许空 |
| `IntegerField` | `INTEGER` | 整数 | — |
| `PositiveIntegerField` | `INTEGER UNSIGNED` | 正整数（计数、ID） | — |
| `DecimalField` | `DECIMAL` | 精确小数（金钱） | `max_digits`、`decimal_places` |
| `FloatField` | `FLOAT` | 浮点数（评分、比率） | — |
| `BooleanField` | `BOOL` | 是/否（是否发布） | `default`建议设置 |
| `DateTimeField` | `DATETIME` | 日期时间 | `auto_now_add`、`auto_now` |
| `DateField` | `DATE` | 日期（生日） | — |
| `EmailField` | `VARCHAR` | 邮箱（自带格式校验） | `max_length` |
| `URLField` | `VARCHAR` | URL链接 | — |
| `ImageField` | `VARCHAR` | 图片路径 | `upload_to` |
| `FileField` | `VARCHAR` | 文件路径 | `upload_to` |
| `SlugField` | `VARCHAR` | URL友好的短标签 | `unique=True`常配合 |
| `ForeignKey` | 外键 | 一对多关联 | `on_delete`（必填） |
| `ManyToManyField` | 中间表 | 多对多关联 | — |
| `OneToOneField` | 外键+唯一约束 | 一对一关联 | `on_delete` |

**DateTimeField的两个关键参数**：
```python
# auto_now_add=True：只在创建时自动设置时间（首次保存时填充）
pub_date = models.DateTimeField(auto_now_add=True)

# auto_now=True：每次保存都自动更新为当前时间
updated_at = models.DateTimeField(auto_now=True)
```

## 2.3 通用字段选项
以下选项几乎适用于所有字段类型：

```python
class Article(models.Model):
    # null=True：数据库层面，允许该列为NULL
    middle_name = models.CharField(max_length=50, null=True)

    # blank=True：表单验证层面，允许提交时该字段为空
    bio = models.TextField(blank=True)

    # null=True + blank=True：数据库允许NULL，表单也允许空（搜索的文本适用）
    subtitle = models.CharField(max_length=200, null=True, blank=True)

    # default：默认值
    views = models.IntegerField(default=0)

    # unique=True：该字段的值在整个表中必须唯一
    slug = models.SlugField(unique=True)

    # choices：限定字段只能从预设选项中选取
    STATUS_CHOICES = [
        ('draft', '草稿'),
        ('published', '已发布'),
        ('archived', '已归档'),
    ]
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='draft')

    # db_index=True：为该字段创建数据库索引（常用于搜索/排序字段）
    title = models.CharField(max_length=200, db_index=True)

    # verbose_name：Admin和表单中的显示名称
    cover = models.ImageField(upload_to='covers/%Y/%m/', verbose_name="封面图片")

    # help_text：辅助说明（Admin和表单中显示在字段下方）
    tags = models.CharField(max_length=500, help_text="多个标签用逗号分隔")
```

## 2.4 Meta类：表级别配置
```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    # ... 其他字段

    class Meta:
        # 自定义数据库表名
        db_table = 'articles'

        # 默认排序（- 表示降序）
        ordering = ['-pub_date', 'title']

        # 联合唯一约束（一篇文章在同一分类下标题唯一）
        unique_together = [['category', 'title']]

        # 联合索引（常用于按分类+时间查询）
        indexes = [
            models.Index(fields=['category', '-pub_date']),
        ]

        # Admin中的显示名称
        verbose_name = '文章'
        verbose_name_plural = '文章列表'

        # 指定默认的排序和管理方式
        # abstract = True  # 如果设为True，这个类不会创建表（作为其他模型的抽象基类）
```

# 三、字段关系：外键、多对多、一对一
## 3.1 ForeignKey：一对多关系
**场景**：一篇文章属于一个作者，一个作者可以有多篇文章。这是最常用的关系。

```python
from django.contrib.auth.models import User

class Article(models.Model):
    title = models.CharField(max_length=200)
    # ForeignKey(关联模型, on_delete行为, related_name反向查询名)
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,     # 作者删除时，他的文章也级联删除
        related_name='articles',       # 通过 user.articles.all() 反向查询文章
        verbose_name='作者'
    )

# 使用示例
# 正向：article.author → 获取作者
# 反向：user.articles.all() → 获取该作者的所有文章
```

**on_delete参数详解（必填参数）**：
| 选项 | 行为 | 使用场景 |
|------|------|---------|
| `CASCADE` | 级联删除：删除作者时，关联的文章也删除 | 文章-作者（作者没了文章也没意义） |
| `PROTECT` | 保护：如果有关联数据，禁止删除 | 分类-文章（有文章时不允许删分类） |
| `SET_NULL` | 设为NULL：删除作者时，文章的作者字段变为NULL | 评论-用户（用户删了评论还在） |
| `SET_DEFAULT` | 设为默认值 | 分配到默认分类 |
| `DO_NOTHING` | 什么都不做（可能导致数据库完整性错误） | 特殊场景 |

## 3.2 ManyToManyField：多对多关系
**场景**：一篇文章可以有多个标签，一个标签可以属于多篇文章。

```python
class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)

class Article(models.Model):
    title = models.CharField(max_length=200)
    tags = models.ManyToManyField(
        Tag,
        related_name='articles',     # 通过 tag.articles.all() 反向查询
        verbose_name='标签'
    )

# Django会自动创建一张中间表：blog_article_tags
# 包含 article_id 和 tag_id 两列

# 使用示例
# article.tags.all()     → 获取文章的所有标签
# article.tags.add(tag)  → 给文章添加标签
# tag.articles.all()     → 获取该标签下的所有文章
```

## 3.3 OneToOneField：一对一关系
**场景**：扩展Django内置User模型（给用户增加头像、个人简介等字段）。

```python
class UserProfile(models.Model):
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile'
    )
    avatar = models.ImageField(upload_to='avatars/')
    bio = models.TextField(blank=True)
    website = models.URLField(blank=True)

# 使用示例
# user.profile.avatar → 获取用户头像
# profile.user.email → 获取用户邮箱
```

# 四、数据库迁移系统
## 4.1 迁移是什么
数据库迁移（Migration）是Django最强大的功能之一。它的核心作用是：**追踪模型的变化并同步到数据库**。

当你修改了模型（添加字段、修改类型、添加新表等），Django会自动生成迁移文件（Python脚本），然后执行这些脚本把数据库结构更新。

## 4.2 迁移的三步流程
```bash
# 步骤1：生成迁移文件（Python脚本）
python manage.py makemigrations
# 输出：blog/migrations/0001_initial.py（自动生成）

# 步骤2：查看迁移文件对应的SQL（可选，用于验证）
python manage.py sqlmigrate blog 0001
# 输出：
# BEGIN;
# CREATE TABLE "blog_tag" (...);
# CREATE TABLE "blog_article" (...);
# COMMIT;

# 步骤3：执行迁移（把变更应用到数据库）
python manage.py migrate
# 输出：
# Applying blog.0001_initial... OK
```

## 4.3 迁移文件的内部结构
打开`blog/migrations/0001_initial.py`：

```python
from django.db import migrations, models
import django.db.models.deletion
import django.utils.timezone

class Migration(migrations.Migration):
    initial = True           # 首次迁移标记
    dependencies = []        # 依赖的其他迁移文件

    operations = [
        migrations.CreateModel(
            name='Article',
            fields=[
                ('id', models.BigAutoField(...)),
                ('title', models.CharField(max_length=200)),
                ('content', models.TextField()),
                ('views', models.PositiveIntegerField(default=0)),
                # ... 其他字段
            ],
        ),
        # 可能有多个 CreateModel、AddField、AlterField 等操作
    ]
```

每个迁移文件记录了模型的一次变更。Django按创建顺序依次执行所有迁移文件，从而将数据库从初始状态一步步演变到当前状态。

## 4.4 迁移管理常用命令
```bash
# 查看哪些迁移尚未执行
python manage.py showmigrations
# [ ] 表示未执行，[X] 表示已执行

# 回滚到指定迁移（撤销某个迁移）
python manage.py migrate blog 0001   # 回滚到 0001 的状态

# 回滚所有迁移（回到初始状态，数据会丢失！）
python manage.py migrate blog zero

# 查看某个迁移的SQL
python manage.py sqlmigrate blog 0002

# 合并多个迁移文件（当迁移文件过多时）
python manage.py makemigrations --merge

# 模拟执行迁移（不真正操作数据库，只输出SQL）
python manage.py migrate --fake
```

**常用工作流**：
```bash
# 1. 修改 models.py
# 2. 生成迁移
python manage.py makemigrations

# 3. 检查生成的SQL是否正确
python manage.py sqlmigrate blog 0002

# 4. 应用到数据库
python manage.py migrate
```

## 4.5 迁移常见场景
**场景1：添加新字段**
```python
# models.py中新增字段
class Article(models.Model):
    # ... 原有字段
    summary = models.CharField(max_length=500, blank=True)  # 新增
```
```bash
python manage.py makemigrations
# Django会提示："You are trying to add a non-nullable field..."
# 如果字段有 default 或 blank=True，迁移正常生成
# 如果字段必填且没有默认值，Django会问你现在怎么填充已有数据
python manage.py migrate
```

**场景2：修改字段选项**
```python
# 将 title 的 max_length 从200改为300
title = models.CharField(max_length=300)  # 原来是200
```
```bash
python manage.py makemigrations
python manage.py migrate
```

**场景3：删除字段**
```python
# 删除 summary 字段
```
```bash
python manage.py makemigrations
python manage.py migrate
# 注意：迁移会删除该列的所有数据！确认无误再执行
```

# 五、在视图中使用模型
## 5.1 从数据库查询数据
修改视图，用真实数据替代硬编码数据：

```python
# blog/views.py
from django.shortcuts import render
from .models import Article

def index(request):
    # 查询所有已发布的文章，按发布时间倒序
    articles = Article.objects.filter(is_published=True).order_by('-pub_date')[:10]

    context = {
        'articles': articles,
    }
    return render(request, 'blog/index.html', context)
```

## 5.2 在模板中展示
```html
<!-- blog/templates/blog/index.html -->
{% extends 'blog/base.html' %}

{% block content %}
<h1>最新文章</h1>

{% for article in articles %}
    <article>
        <h2>{{ article.title }}</h2>
        <p class="meta">
            作者：{{ article.author.username }} |
            发布时间：{{ article.pub_date|date:"Y-m-d H:i" }} |
            阅读量：{{ article.views }}
        </p>
        <p>{{ article.content|truncatechars:200 }}</p>
        <a href="#">阅读全文</a>
    </article>
{% empty %}
    <p>暂无文章。</p>
{% endfor %}
{% endblock %}
```

现在访问首页，你看到的数据来自数据库（如果数据库中有文章数据的话）。你可以通过Django Admin或manage.py shell来添加测试数据：

```bash
# 进入Django交互式Shell
python manage.py shell
```

```python
# 在Shell中创建测试数据
from blog.models import Article
from django.contrib.auth.models import User

# 获取或创建测试用户
user = User.objects.first()

# 创建文章
Article.objects.create(
    title="第一篇Django文章",
    content="这是通过ORM创建的第一篇文章内容。ORM让我们不需要写SQL，直接用Python操作数据库。",
    author=user,
)

Article.objects.create(
    title="Django模型与迁移详解",
    content="Django的ORM是业界最强大的ORM之一...",
    author=user,
)

# 查询验证
print(Article.objects.all().count())  # 2
```

# 六、常见误区与避坑指南
1.  **修改模型后忘记makemigrations和migrate**：改完models.py后，数据库不会自动更新。必须执行`makemigrations`（生成迁移文件）和`migrate`（应用到数据库）两步。这是新手最常忘记的操作。

2.  **null vs blank 混淆**：`null=True`针对**数据库**，允许该列存储NULL；`blank=True`针对**表单验证**，允许用户提交时该字段为空。两者的控制层面不同。对于字符串字段，数据库层面不推荐用NULL（空值应该用空字符串），所以CharField/TextField通常只设`blank=True`而不设`null=True`。

3.  **ForeignKey忘记on_delete参数**：Django 2.0+要求ForeignKey必须指定`on_delete`。如果不指定，makemigrations会报错。最常用的是`on_delete=models.CASCADE`。

4.  **直接在模板中调用 `.all()` 做查询**：模板是用于展示数据的，不应该在模板中执行数据库查询（如`{% for article in Article.objects.all %}`）。把查询逻辑放在视图中，模板只负责渲染。

5.  **迁移文件冲突（多人协作）**：两个开发者在不同分支都修改了模型，都生成了迁移文件，合并时可能产生冲突。解决方案是用`python manage.py makemigrations --merge`合并，或在合并前协调谁先生成迁移。

6.  **生产环境不要直接删除模型类**：删除模型类之前，先删除所有对该模型的外键引用，生成迁移，确认无误后再删除模型类。直接删除模型类而不处理引用会导致迁移失败。

# 七、核心总结
## 常用字段类型速查
| Django字段 | 数据库类型 | 当...时使用 |
|-----------|-----------|-----------|
| `CharField` | VARCHAR | 标题、用户名等短文本 |
| `TextField` | TEXT | 文章内容、简介等长文本 |
| `IntegerField` | INTEGER | 普通整数 |
| `BooleanField` | BOOL | 是/否开关 |
| `DateTimeField` | DATETIME | 发布时间、更新时间 |
| `DecimalField` | DECIMAL | 精确金额 |
| `ForeignKey` | 外键列 | 一对多关系 |
| `ManyToManyField` | 中间表 | 多对多关系 |
| `ImageField` | VARCHAR | 图片路径 |

## 迁移命令速查
| 命令 | 用途 |
|------|------|
| `makemigrations` | 根据模型变更生成迁移文件 |
| `migrate` | 将迁移文件应用到数据库 |
| `sqlmigrate app 0001` | 查看迁移对应的SQL |
| `showmigrations` | 查看迁移状态 |
| `migrate app 0001` | 回滚到指定迁移 |
| `migrate app zero` | 回滚所有迁移 |

## 开发流程
```
修改models.py → makemigrations → migrate → 在视图中使用模型 → 模板渲染
```

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
