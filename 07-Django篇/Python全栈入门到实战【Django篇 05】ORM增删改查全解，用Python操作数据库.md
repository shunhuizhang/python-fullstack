
# Python全栈入门到实战【Django篇 05】ORM增删改查全解，用Python操作数据库
上一篇《Django篇 04》中，我们学会了如何定义数据模型（Model）并同步到数据库。但定义模型只是搭建了数据的"骨架"，真正的能力体现在对数据的**增删改查**操作上——用户发布文章（增）、修改文章内容（改）、管理员删除违规文章（删）、读者浏览文章列表（查）。这四种操作涵盖了Web应用中99%的数据库操作场景。

Django ORM提供了非常丰富的查询API，从最简单的`all()`到复杂的跨表查询、聚合统计、子查询。作为全栈开发者，掌握ORM的增删改查是你操作数据库的核心技能。作为Python开发者，你会发现Django ORM的QuerySet和Python的列表推导式在思维模式上高度一致——`filter`就像列表过滤，`order_by`就像`sorted`，`annotate`就像`groupby`。

本节核心学习内容：
1.  创建记录：create()、save()、bulk_create()、get_or_create()
2.  查询记录：all()、filter()、exclude()、get()完整语法
3.  QuerySet延迟求值：什么时候真正执行查询
4.  复杂查询：Q对象（复合条件）、F对象（字段间比较）
5.  聚合与分组：aggregate()、annotate()
6.  跨表查询：正向查询与外键反向查询（related_name）
7.  更新与删除：update()、delete()
8.  select_related与prefetch_related：性能优化的核心
9.  原生SQL执行（特殊场景）

# 一、创建记录（CREATE）
## 1.1 create()：一步创建
```python
from blog.models import Article
from django.contrib.auth.models import User

user = User.objects.first()

# 方式1：objects.create() —— 一步创建并返回对象
article = Article.objects.create(
    title="Django ORM入门",
    content="创建记录是最基本的数据库操作...",
    author=user,
    is_published=True
)
# create() 会自动调用 save()，不需要再单独保存

# 方式2：实例化 + save() —— 分步创建（可以在保存前修改）
article = Article(
    title="Django ORM入门",
    content="创建记录...",
    author=user,
)
article.views = 100  # 保存前修改
article.save()       # 手动保存到数据库
```

**Python类比**：Django的`create()`等同于Python中的`list.append()`并自动持久化。区别是Django的数据会存入数据库，下次重启依然存在。

## 1.2 bulk_create()：批量创建
当需要一次性创建大量记录时，逐条create()会执行大量SQL，效率极低。`bulk_create()`将多条记录合并为一条SQL语句，大幅提升性能：

```python
# ❌ 低效：1000条记录 = 1000条SQL
for i in range(1000):
    Article.objects.create(title=f"文章{i}", content="...", author=user)

# ✓ 高效：1000条记录 ≈ 1条SQL
articles = [
    Article(title=f"文章{i}", content="...", author=user)
    for i in range(1000)
]
Article.objects.bulk_create(articles)

# 批量大小可控制（大数据量时防止内存溢出）
Article.objects.bulk_create(articles, batch_size=500)
```

## 1.3 get_or_create()：不存在则创建
常用于"如果存在就获取，不存在就创建"的场景，避免重复数据。返回一个`(object, created)`元组：

```python
# 查找标题为"ORM教程"的文章，如果不存在则创建
article, created = Article.objects.get_or_create(
    title="ORM教程",
    defaults={            # 仅在创建时使用的字段
        'content': '默认内容',
        'author': user,
        'is_published': True
    }
)

if created:
    print("新创建了一篇文章")
else:
    print("文章已存在：", article.title)
```

> `defaults`参数只在创建新记录时使用。如果记录已存在，`get_or_create`只返回已存在的记录，不会更新它。

## 1.4 update_or_create()：不存在则创建，存在则更新
```python
article, created = Article.objects.update_or_create(
    title="ORM教程",
    defaults={
        'content': '更新后的内容',  # 不管创建还是更新，都会应用
        'author': user,
    }
)
```

# 二、查询记录（READ）
## 2.1 基础查询方法
```python
# all()：获取所有记录（返回QuerySet，类似Python列表的查询对象）
all_articles = Article.objects.all()

# filter()：条件过滤（返回QuerySet，可能为空）
published = Article.objects.filter(is_published=True)
python_articles = Article.objects.filter(title__contains="Python")
recent = Article.objects.filter(pub_date__year=2026)

# exclude()：排除条件（取反）
not_published = Article.objects.exclude(is_published=True)

# get()：获取单条记录（返回对象，找不到或找到多条都会报错）
article = Article.objects.get(id=1)  # 精确获取
article = Article.objects.get(pk=1)  # pk = primary key，自动指向主键

# 切割 QuerySet（类似Python切片）
first_five = Article.objects.all()[:5]       # 前5条
second_page = Article.objects.all()[5:10]    # 第5-10条
latest = Article.objects.order_by('-id')[0]  # 最新的一条
```

## 2.2 get() vs filter() vs all()
| 方法 | 返回类型 | 找不到时 | 找到多个时 | 何时使用 |
|------|---------|---------|-----------|---------|
| `get()` | 模型实例 | 抛出 `DoesNotExist` | 抛出 `MultipleObjectsReturned` | 已知唯一记录（ID） |
| `filter()` | QuerySet（可为空） | 返回空QuerySet | 返回所有匹配的 | 多条件筛选 |
| `all()` | QuerySet | 返回空QuerySet | 返回所有 | 获取全部 |

```python
# ✅ 正确：用get获取单条记录
article = Article.objects.get(id=42)

# ✅ 正确：用filter获取多条记录
articles = Article.objects.filter(author=user)

# ❌ 错误：get找到一个空的filter结果
article = Article.objects.get(is_published=False)  # 可能有多条草稿！
```

## 2.3 字段查询条件（Field Lookups）
Django提供了丰富的查询条件，用`字段名__条件`的形式使用：

| 条件 | 含义 | SQL等效 | 示例 |
|------|------|---------|------|
| `exact` | 精确匹配 | `=` | `title__exact="Python"` |
| `iexact` | 不区分大小写精确匹配 | `ILIKE` | `title__iexact="python"` |
| `contains` | 包含 | `LIKE %...%` | `title__contains="Django"` |
| `icontains` | 不区分大小写包含 | `ILIKE %...%` | `title__icontains="django"` |
| `startswith` | 以...开头 | `LIKE ...%` | `title__startswith="Python"` |
| `endswith` | 以...结尾 | `LIKE %...` | `title__endswith="教程"` |
| `in` | 在列表中 | `IN (...)` | `id__in=[1,2,3]` |
| `gt` | 大于 | `>` | `views__gt=100` |
| `gte` | 大于等于 | `>=` | `views__gte=100` |
| `lt` | 小于 | `<` | `price__lt=100` |
| `lte` | 小于等于 | `<=` | `price__lte=100` |
| `range` | 范围（包含边界） | `BETWEEN` | `pub_date__range=(start, end)` |
| `isnull` | 是否为NULL | `IS NULL` | `summary__isnull=True` |
| `year/month/day` | 日期提取 | `EXTRACT` | `pub_date__year=2026` |
| `regex` | 正则匹配 | `REGEXP` | `title__regex=r'^Py'` |

```python
# 实战示例
# 标题包含Python的文章（不区分大小写）
Article.objects.filter(title__icontains='python')

# 阅读量大于100的已发布文章
Article.objects.filter(views__gt=100, is_published=True)

# ID在[1, 5, 10]之中的文章
Article.objects.filter(id__in=[1, 5, 10])

# 2026年发布的文章
import datetime
Article.objects.filter(pub_date__year=2026)
Article.objects.filter(pub_date__gte=datetime.date(2026, 1, 1))

# 标题不为空的文章
Article.objects.filter(title__isnull=False).exclude(title='')
```

## 2.4 QuerySet是惰性求值的
**QuerySet对象在被创建时，不会立即执行数据库查询**。只有在真正需要数据时（遍历、切片、调用len等），Django才会执行SQL。这种设计叫**惰性求值**，意味着可以链式调用而不会产生多次查询：

```python
# 下面的代码全部没有执行数据库查询：
qs = Article.objects.all()                  # 不查询
qs = qs.filter(is_published=True)            # 不查询
qs = qs.filter(views__gt=100)               # 不查询
qs = qs.order_by('-pub_date')               # 不查询
qs = qs[:10]                                # 不查询

# 下面任一操作会触发查询：
# 1. 遍历
for article in qs:  # ← 这里才执行SQL
    print(article.title)

# 2. list()
articles = list(qs)   # 立即查询并转为列表

# 3. len()
count = len(qs)       # 立即查询

# 4. 判断
exists = qs.exists()  # 检查是否有结果（比len效率高）
```

> **验证惰性求值**：在Django Shell中执行`qs = Article.objects.all()`不会看到任何SQL输出（需要配置`LOGGING`才能看到SQL日志），但执行`print(qs)`时会立刻看到SQL。

# 三、复杂查询：Q对象与F对象
## 3.1 Q对象：逻辑或（OR）查询
普通的filter()中多个参数是**AND（与）**关系。要实现**OR（或）**查询，需要使用Q对象：

```python
from django.db.models import Q

# AND查询（默认）：标题包含Python AND 已发布
Article.objects.filter(title__icontains='python', is_published=True)

# OR查询（用Q）：标题包含Python OR 标题包含Django
Article.objects.filter(
    Q(title__icontains='python') | Q(title__icontains='django')
)

# NOT查询（用~）：不是草稿状态
Article.objects.filter(~Q(status='draft'))

# 混合使用：查询（标题包含Python OR Django）AND 已发布 AND 阅读量>100
Article.objects.filter(
    Q(title__icontains='python') | Q(title__icontains='django'),
    is_published=True,
    views__gt=100
)
# 注意：Q对象和关键字参数混用时，Q对象必须放在前面
```

**Python类比**：Q对象 ≈ Python的`or`和`and`操作，但在SQL层面。
```
Django: Q(a) | Q(b)      Python:  条件A or 条件B
Django: Q(a) & Q(b)      Python:  条件A and 条件B
Django: ~Q(a)            Python:  not 条件A
```

## 3.2 F对象：字段间比较与自增
普通的查询只能比较字段和固定值。F对象让你可以在查询中**引用模型自身的字段**：

```python
from django.db.models import F

# 场景1：查询阅读量大于收藏量的文章
Article.objects.filter(views__gt=F('favorites'))

# 场景2：查询内容长度大于标题长度的文章
from django.db.models.functions import Length
Article.objects.filter(
    Length('content') > F('title_length')
)

# 场景3：自增操作（不需要先查出来再+1再保存！）
# 传统做法（效率低，有并发问题）
article = Article.objects.get(id=1)
article.views += 1
article.save()

# F对象做法（一条SQL搞定，不会产生竞争条件）
Article.objects.filter(id=1).update(views=F('views') + 1)
# SQL: UPDATE articles SET views = views + 1 WHERE id = 1;
```

# 四、聚合与分组
## 4.1 aggregate()：全表聚合
对整个表执行统计计算，返回一个字典：

```python
from django.db.models import Count, Sum, Avg, Max, Min

# 各种聚合函数
result = Article.objects.aggregate(
    total=Count('id'),             # 总记录数
    total_views=Sum('views'),      # 总阅读量
    avg_views=Avg('views'),        # 平均阅读量
    max_views=Max('views'),        # 最高阅读量
    min_views=Min('views'),        # 最低阅读量
)

print(result)
# {'total': 150, 'total_views': 45000, 'avg_views': 300.0, 'max_views': 5000, 'min_views': 10}
```

## 4.2 annotate()：分组统计
相当于SQL的`GROUP BY`。对QuerySet的每个分组做聚合：

```python
# 每个作者的文章数量
from django.db.models import Count
authors = User.objects.annotate(
    article_count=Count('articles')   # articles 是 related_name
)
for user in authors:
    print(f"{user.username}: {user.article_count}篇")

# 每个作者的文章总阅读量
authors = User.objects.annotate(
    total_views=Sum('articles__views')
)

# 每个分类下已发表文章的数量
categories = Category.objects.annotate(
    published_count=Count('articles', filter=Q(articles__is_published=True))
)

# 多级排序：按文章数量倒序
authors = User.objects.annotate(
    article_count=Count('articles')
).order_by('-article_count')
```

**Python类比**：
```python
# Django annotate + Count
User.objects.annotate(count=Count('articles'))

# Python 等价于
from collections import Counter
Counter(article.author for article in articles)
```

# 五、排序与去重
```python
# 排序
Article.objects.order_by('pub_date')            # 按发布时间升序
Article.objects.order_by('-pub_date')            # 按发布时间降序（- 号）
Article.objects.order_by('-pub_date', 'title')   # 多级排序

# 去重（去除相同的记录）
Article.objects.values('author').distinct()       # 不重复的作者列表
Article.objects.filter(tags__name='python').distinct()  # 去重（多对多容易重复）
```

# 六、更新与删除记录
## 6.1 更新单条记录
```python
# 方式1：获取对象 → 修改属性 → save()
article = Article.objects.get(id=1)
article.title = "修改后的标题"
article.save()

# 方式2：直接update()（不调用save，不触发信号）
Article.objects.filter(id=1).update(title="修改后的标题")
```

## 6.2 批量更新
```python
# 所有草稿改为已发布
Article.objects.filter(status='draft').update(
    status='published',
    pub_date=timezone.now()
)

# 所有文章的阅读量翻倍（使用F对象）
Article.objects.all().update(views=F('views') * 2)
```

## 6.3 删除记录
```python
# 删除单条
article = Article.objects.get(id=1)
article.delete()

# 批量删除
Article.objects.filter(status='draft').delete()

# 删除所有记录（危险操作！）
# Article.objects.all().delete()
```

> **级联删除**：如果模型中有`ForeignKey(on_delete=CASCADE)`，删除主表记录时会自动删除从表关联的记录。比如删除作者会同时删除他的所有文章。

# 七、select_related与prefetch_related：查询性能优化
## 7.1 N+1查询问题
这是数据库查询中最常见的性能陷阱：

```python
# ❌ N+1问题：查询作者 + N次查询文章
articles = Article.objects.all()  # 查询1：获取所有文章
for article in articles:
    print(article.author.username)  # 每篇文章查询一次作者（N次查询！）
# 总计：1 + N 次数据库查询
```

## 7.2 select_related：JOIN查询（外键/一对一）
`select_related`通过SQL的`JOIN`一次性获取关联对象，适合**外键**和**一对一**关系：

```python
# ✅ 优化：一次SQL JOIN查询搞定
articles = Article.objects.select_related('author', 'category').all()
# SQL: SELECT ... FROM article LEFT JOIN user ON ... LEFT JOIN category ON ...

for article in articles:
    print(article.author.username)  # 不再产生额外查询！
# 总计：1 次数据库查询
```

## 7.3 prefetch_related：分别查询后合并（多对多/反向外键）
`prefetch_related`对于**多对多**和**反向外键**关系更高效。它执行两次查询（主表 + 关联表），然后在Python层面合并结果：

```python
# ✅ 优化：减少查询次数
articles = Article.objects.prefetch_related('tags', 'comments').all()
# SQL 1: SELECT * FROM article
# SQL 2: SELECT * FROM tag WHERE id IN (文章中出现的tag_id)
# SQL 3: SELECT * FROM comment WHERE article_id IN (文章的ID)
# 总计：3次查询（不管有多少文章）

for article in articles:
    for tag in article.tags.all():     # 不再产生额外查询！
        print(tag.name)
```

## 7.4 使用指南
| 关系类型 | 使用 | 原理 |
|---------|------|------|
| 外键（ForeignKey） | `select_related('author')` | SQL JOIN |
| 一对一（OneToOne） | `select_related('profile')` | SQL JOIN |
| 反向外键（article.author的逆向） | `prefetch_related('articles')` | 两次查询+Python合并 |
| 多对多（ManyToMany） | `prefetch_related('tags')` | 两次查询+Python合并 |

```python
# 可以同时使用两者
articles = Article.objects.select_related('author') \
                          .prefetch_related('tags', 'comments') \
                          .all()
```

# 八、原生SQL执行
虽然ORM覆盖了大多数场景，但有时你确实需要执行原生SQL。Django提供了两种方式：

```python
# 方式1：raw() —— 返回模型实例
articles = Article.objects.raw('SELECT * FROM blog_article WHERE views > %s', [100])
for article in articles:
    print(article.title)

# 方式2：直接执行SQL（绕过ORM）
from django.db import connection
with connection.cursor() as cursor:
    cursor.execute("UPDATE blog_article SET views = 0 WHERE id = %s", [1])
    affected_rows = cursor.rowcount
```

> **SQL注入防护**：永远不要用字符串拼接构造SQL！始终使用参数化查询（`%s`占位符 + 参数列表）。

# 九、常见误区与避坑指南
1.  **filter链式调用时每次调用都产生新的QuerySet**：QuerySet是不可变的，每次调用filter/exclude/order_by都返回一个新的QuerySet。这是惰性求值的基础。

2.  **get()找不到/找到多个会抛异常**：使用`get()`时应该用try-except捕获异常，或使用`get_object_or_404()`快捷方式（后面详讲）。

3.  **忘记F对象做自增操作**：并发场景下，`article.views += 1; article.save()`会出现**丢失更新**问题。使用`update(views=F('views')+1)`在数据库层面原子操作。

4.  **在循环中逐个查询**：`for article in articles: print(article.author.name)` 这是经典的N+1问题。用`select_related`一次JOIN解决。

5.  **filter(foreign_key_id=1) vs filter(foreign_key__id=1)**：两种写法结果相同，但`author_id=1`不需要JOIN查询author表，效率更高。

6.  **混淆values()和values_list()**：`values()`返回字典列表，`values_list()`返回元组列表。`values_list('id', flat=True)`返回单字段的值列表。

# 十、核心总结：ORM速查表
## 创建
| 方法 | 说明 |
|------|------|
| `Model.objects.create(**kwargs)` | 创建并保存 |
| `obj = Model(...); obj.save()` | 分步创建 |
| `Model.objects.bulk_create(list)` | 批量创建 |
| `Model.objects.get_or_create(defaults, **kwargs)` | 不存在则创建 |

## 查询
| 方法 | 说明 |
|------|------|
| `all()` | 所有记录 |
| `filter(**kwargs)` | 条件过滤 |
| `exclude(**kwargs)` | 排除条件 |
| `get(**kwargs)` | 获取单条（找不到报错） |
| `order_by('field')` | 排序（`-` 降序） |
| `values('field')` | 返回字典QuerySet |
| `distinct()` | 去重 |

## 高级查询
| 工具 | 用途 |
|------|------|
| `Q(条件) \| Q(条件)` | OR查询 |
| `F('字段名')` | 字段间比较 / 自增 |
| `aggregate(Count('id'))` | 全表聚合 |
| `annotate(Count('related'))` | 分组统计 |
| `select_related('fk')` | JOIN优化外键 |
| `prefetch_related('m2m')` | 分别查询+合并优化多对多 |

## 更新与删除
| 方法 | 说明 |
|------|------|
| `obj.save()` | 保存单条 |
| `queryset.update(**kwargs)` | 批量更新 |
| `obj.delete()` / `queryset.delete()` | 删除 |

# 十一、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
