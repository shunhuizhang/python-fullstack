
# Python全栈入门到实战【Django篇 08】类视图（CBV）与Admin后台管理系统详解
前面7篇文章中，我们所有的视图都是**函数视图（FBV）**——一个Python函数接收request并返回response。这种方式简单直观，但当你发现：列表页、详情页、创建页、编辑页、删除页的逻辑模式几乎一致（查数据→渲染/保存→返回），每写一个视图都要重复"获取对象→处理POST→处理GET"的代码，就会开始思考：能不能有一个更优雅的方式？

这就是**类视图（CBV, Class-Based View）**的价值。Django提供了一系列继承了通用视图逻辑的类，你只需要继承它们，指定模型和模板，最常见的增删改查操作可以在一两行代码内完成。同时，本篇还将学习**Django Admin后台管理系统**——这是Django最强大的"免费午餐"：你只需注册模型，就自动获得一个功能齐全的数据管理后台。

本节核心学习内容：
1.  FBV vs CBV：何时用函数，何时用类
2.  TemplateView：渲染静态页面
3.  ListView：列表页（分页、搜索、排序）
4.  DetailView：详情页（根据主键获取对象）
5.  CreateView / UpdateView / DeleteView：增删改一条龙
6.  CBV中重写方法：get_queryset、get_context_data
7.  Django Admin：注册模型、自定义显示、搜索、过滤器
8.  两个体系如何配合：用户后台用Admin，前台用CBV

# 一、FBV vs CBV
## 1.1 代码对比
同一个"文章列表"功能，两种写法：

**FBV写法**（函数视图）：
```python
def article_list(request):
    articles = Article.objects.filter(is_published=True).order_by('-pub_date')
    page = request.GET.get('page', 1)
    paginator = Paginator(articles, 10)
    page_obj = paginator.get_page(page)
    return render(request, 'blog/article_list.html', {
        'articles': page_obj,
        'page_obj': page_obj
    })
```

**CBV写法**（类视图）：
```python
from django.views.generic import ListView

class ArticleListView(ListView):
    model = Article
    template_name = 'blog/article_list.html'
    context_object_name = 'articles'
    paginate_by = 10

    def get_queryset(self):
        return Article.objects.filter(is_published=True).order_by('-pub_date')
```

同样的功能，CBV把重复的模式封装在了父类中，你只需指定模型、模板、每页数量。代码量明显减少，可读性提高。

## 1.2 什么时候用哪种
| 场景 | 推荐 | 原因 |
|------|------|------|
| 简单的列表/详情/编辑页 | CBV | 通用逻辑已被封装，写配置即可 |
| 复杂的业务逻辑（搜索、多表查询） | FBV | 逻辑复杂时函数视图更灵活 |
| 需要GET和POST共用变量 | FBV | 同一个函数中可以共享局部变量 |
| 快速原型开发 | CBV | 一行代码完成常见操作 |

> 实际项目中，**两者混合使用**是常态。简单的CRUD用CBV省代码，复杂的业务逻辑用FBV更清晰。

# 二、TemplateView：最简单的CBV
用于渲染**纯模板页面**（不需要从数据库查数据，或数据通过其他方式获取）：

```python
from django.views.generic import TemplateView

class HomeView(TemplateView):
    template_name = 'home.html'

    def get_context_data(self, **kwargs):
        """提供模板所需的数据"""
        context = super().get_context_data(**kwargs)
        context['title'] = '欢迎来到我的博客'
        context['article_count'] = Article.objects.count()
        return context
```

```python
# urls.py
urlpatterns = [
    path('', HomeView.as_view(), name='home'),  # 注意：CBV需要 .as_view()
]
```

**`.as_view()`的作用**：将类视图转换为Django可调用的视图函数。每个请求到达时，Django会实例化这个类，并根据HTTP方法（GET/POST）调用对应的方法。

# 三、CBV的核心视图体系
## 3.1 Django CBV的继承层次
```
View（所有CBV的基类）
├── TemplateView（渲染模板）
└── GenericDisplayView
    ├── DetailView（详情页：显示单条记录）
    └── ListView（列表页：显示多条记录）
            └── GenericEditView
                ├── FormView（处理表单）
                │   └── CreateView（创建记录）
                └── UpdateView（更新记录）
                       └── DeleteView（删除记录）
```

## 3.2 内置的通用视图一览
| CBV类 | 用途 | 常用属性 |
|-------|------|---------|
| `View` | 所有CBV的基类 | — |
| `TemplateView` | 渲染静态模板 | `template_name` |
| `RedirectView` | 重定向 | `url` / `pattern_name` |
| `ListView` | 对象列表 | `model`、`queryset`、`paginate_by` |
| `DetailView` | 对象详情 | `model`、`pk_url_kwarg`、`slug_url_kwarg` |
| `CreateView` | 创建对象 | `model`、`form_class`、`fields`、`success_url` |
| `UpdateView` | 更新对象 | 同CreateView |
| `DeleteView` | 删除对象 | `model`、`success_url` |
| `ArchiveIndexView` | 归档列表（按日期） | `date_field` |

# 四、ListView：列表视图
```python
from django.views.generic import ListView
from .models import Article

class ArticleListView(ListView):
    # 基础配置
    model = Article                                    # 关联的模型
    template_name = 'blog/article_list.html'           # 指定模板（默认: blog/article_list.html）
    context_object_name = 'articles'                   # 模板中的变量名（默认: object_list）
    paginate_by = 10                                   # 每页显示数量（不写则不分页）

    def get_queryset(self):
        """自定义查询集（重写数据处理逻辑的入口）"""
        return Article.objects.filter(
            is_published=True
        ).select_related('author').order_by('-pub_date')

    def get_context_data(self, **kwargs):
        """给模板传递额外数据"""
        context = super().get_context_data(**kwargs)
        context['popular_articles'] = Article.objects.order_by('-views')[:5]
        context['categories'] = Category.objects.all()
        return context
```

**ListView的执行流程**：
```
as_view() → dispatch() → get() → get_queryset() → get_context_data()
                                  ↓                    ↓
                           拿到QuerySet          拼装context
                                  ↓                    ↓
                                 渲染 template_name 指定的模板
```

**模板中的默认变量**：
```html
<!-- 访问列表 -->
{% for article in articles %}  <!-- context_object_name定义的变量名 -->
    {{ article.title }}
{% endfor %}

<!-- 分页相关变量（设置了paginate_by后自动可用） -->
{% if is_paginated %}
    {% if page_obj.has_previous %}
        <a href="?page={{ page_obj.previous_page_number }}">上一页</a>
    {% endif %}

    {% for num in paginator.page_range %}
        <a href="?page={{ num }}">{{ num }}</a>
    {% endfor %}

    {% if page_obj.has_next %}
        <a href="?page={{ page_obj.next_page_number }}">下一页</a>
    {% endif %}

    <span>共 {{ paginator.count }} 条</span>
{% endif %}
```

# 五、DetailView：详情视图
```python
from django.views.generic import DetailView

class ArticleDetailView(DetailView):
    model = Article
    template_name = 'blog/article_detail.html'
    context_object_name = 'article'
    # pk_url_kwarg = 'id'  # URL中主键的参数名（默认: pk）

    def get_queryset(self):
        """限定查询范围：只查已发布的"""
        return Article.objects.filter(is_published=True)

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        # 获取当前对象的上一篇和下一篇
        article = self.get_object()
        context['prev_article'] = Article.objects.filter(
            id__lt=article.id, is_published=True
        ).order_by('-id').first()
        context['next_article'] = Article.objects.filter(
            id__gt=article.id, is_published=True
        ).order_by('id').first()
        return context

    def get_object(self, queryset=None):
        """获取对象时自动增加阅读量"""
        obj = super().get_object(queryset)
        Article.objects.filter(pk=obj.pk).update(views=F('views') + 1)
        obj.refresh_from_db()
        return obj
```

```python
# urls.py
urlpatterns = [
    path('article/<int:pk>/', ArticleDetailView.as_view(), name='article-detail'),
]
```

# 六、CreateView + UpdateView + DeleteView：增删改
```python
from django.views.generic import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy  # 解决CBV中反向解析的加载时机问题

class ArticleCreateView(LoginRequiredMixin, CreateView):
    model = Article
    form_class = ArticleForm
    # fields = ['title', 'content', 'is_published']  # 如果不用ModelForm可以用fields
    template_name = 'blog/article_form.html'
    success_url = reverse_lazy('article-list')  # 创建成功后跳转

    def form_valid(self, form):
        """表单验证通过后，保存前设置作者"""
        form.instance.author = self.request.user  # 自动设置当前用户为作者
        messages.success(self.request, '文章创建成功！')
        return super().form_valid(form)

class ArticleUpdateView(LoginRequiredMixin, UpdateView):
    model = Article
    form_class = ArticleForm
    template_name = 'blog/article_form.html'
    # UpdateView会自动根据URL中的pk找到要编辑的对象

    def get_success_url(self):
        """动态决定编辑成功后的跳转地址"""
        return reverse('article-detail', kwargs={'pk': self.object.pk})

    def get_queryset(self):
        """确保用户只能编辑自己的文章"""
        return Article.objects.filter(author=self.request.user)

class ArticleDeleteView(LoginRequiredMixin, DeleteView):
    model = Article
    template_name = 'blog/article_confirm_delete.html'
    success_url = reverse_lazy('article-list')

    def get_queryset(self):
        return Article.objects.filter(author=self.request.user)
```

```python
# urls.py
urlpatterns = [
    path('article/create/', ArticleCreateView.as_view(), name='article-create'),
    path('article/<int:pk>/edit/', ArticleUpdateView.as_view(), name='article-edit'),
    path('article/<int:pk>/delete/', ArticleDeleteView.as_view(), name='article-delete'),
]
```

**reateView/UpdateView共用的模板**（article_form.html）：
```html
<h1>{% if form.instance.pk %}编辑{% else %}创建{% endif %}文章</h1>

<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">保存</button>
</form>
```

**DeleteView的确认模板**：
```html
<h1>确认删除</h1>
<p>确认删除文章 "{{ object.title }}"？此操作不可撤销。</p>

<form method="POST">
    {% csrf_token %}
    <button type="submit" class="btn-danger">确认删除</button>
    <a href="{% url 'article-list' %}">取消</a>
</form>
```

## 6.1 reverse vs reverse_lazy
| 函数 | 何时评估 | 适用 |
|------|---------|------|
| `reverse()` | 立即执行 | 函数体内 |
| `reverse_lazy()` | 延迟到第一次使用时 | 类属性（定义时URL配置可能还没加载） |

在CBV的类属性中使用`success_url`时必须用`reverse_lazy`，因为类定义时URL配置还没加载完成。

# 七、Django Admin后台管理系统
## 7.1 Admin是什么
Django Admin是Django最令人惊叹的内置功能——你只需注册模型，系统自动生成一个功能齐全的数据后台管理系统：

- 列表页（带分页、排序、筛选、搜索）
- 详情/编辑/创建页（根据模型字段自动生成Form）
- 批量操作（批量删除、批量发布等）
- 历史记录追踪

这是Django作为"全栈框架"的典型体现——不需要安装任何第三方库，不需要写一行后台代码。

## 7.2 注册模型到Admin
```python
# blog/admin.py
from django.contrib import admin
from .models import Article, Category, Tag

# 方式1：简单注册（默认行为）
admin.site.register(Category)
admin.site.register(Tag)

# 方式2：自定义显示（ModelAdmin类）
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    # 列表页显示的列
    list_display = ['id', 'title', 'author', 'category', 'is_published', 'pub_date', 'views']

    # 可点击进入编辑页的列（默认是第一个字段）
    list_display_links = ['id', 'title']

    # 右侧过滤侧边栏
    list_filter = ['is_published', 'category', 'pub_date']

    # 搜索框（支持模糊搜索）
    search_fields = ['title', 'content']

    # 每页显示的记录数
    list_per_page = 20

    # 可直接在列表页编辑的字段
    list_editable = ['is_published']

    # 默认排序
    ordering = ['-pub_date']

    # 自定义批量操作
    actions = ['make_published', 'make_draft']

    def make_published(self, request, queryset):
        """批量发布"""
        updated = queryset.update(is_published=True)
        self.message_user(request, f'{updated} 篇文章已设为发布')

    def make_draft(self, request, queryset):
        """批量设为草稿"""
        queryset.update(is_published=False)
        self.message_user(request, f'{queryset.count()} 篇文章已设为草稿')

    make_published.short_description = '批量发布'
    make_draft.short_description = '批量设为草稿'
```

## 7.3 Admin界面自定义
```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    # 自定义详情页字段布局（分组分块）
    fieldsets = [
        ('基本信息', {
            'fields': ['title', 'slug', 'content']
        }),
        ('分类与标签', {
            'fields': ['category', 'tags'],
            'classes': ['collapse']  # 默认折叠收起
        }),
        ('发布信息', {
            'fields': ['author', 'is_published', 'pub_date'],
        }),
    ]

    # 自动填充字段
    prepopulated_fields = {'slug': ('title',)}  # 输入title时自动生成slug

    # 外键字段的自动补全（大数据量时替代下拉框）
    autocomplete_fields = ['author']

    # 只读字段
    readonly_fields = ['pub_date', 'views']

    # 自定义详情页的样式和JS
    class Media:
        css = {'all': ('css/admin_custom.css',)}
        js = ('js/admin_custom.js',)
```

## 7.4 创建超级管理员账号
```bash
python manage.py createsuperuser
# 按提示输入：用户名、邮箱、密码
# 密码不会显示在屏幕上（安全设计）
```

然后访问 `http://127.0.0.1:8000/admin/`，用刚创建的管理员账号登录。

# 八、CBV与Admin的配合策略
在实际项目中，两者的分工很明确：

| 功能 | 使用 | 面向用户 |
|------|------|---------|
| 文章浏览、搜索 | CBV（ListView） | 普通访客 |
| 文章发布、编辑 | CBV（CreateView/UpdateView） | 登录用户（作者） |
| 内容审核、数据管理 | Admin | 管理员（站长） |
| 批量操作、用户管理 | Admin | 管理员（站长） |

**Admin适合内部运维，CBV适合外部用户交互。**

# 九、常见误区与避坑指南
1.  **CBV中忘记继承LoginRequiredMixin**：CreateView/UpdateView/DeleteView默认不检查登录状态。你需要把`LoginRequiredMixin`放在继承列表的最前面（它必须在View之前）。

2.  **success_url用reverse而不是reverse_lazy**：类属性中不能使用`reverse()`（URL配置还没加载）。应该用`reverse_lazy('article-list')`。

3.  **get_context_data中忘记调用super()**：如果不调用`super().get_context_data(**kwargs)`，ListView/DetailView自带的context数据（如object_list、paginator）会丢失。

4.  **Admin中list_display不能放ManyToManyField**：多对多字段不能在list_display中显示（会导致大量SQL查询）。如果需要显示，在ModelAdmin中定义方法，用`select_related`或`prefetch_related`优化。

5.  **UpdateView/DeleteView没有指定queryset限制**：如果不重写get_queryset，任何用户都可以通过猜测ID来编辑/删除别人的文章。始终在get_queryset中做权限过滤。

6.  **混淆get_object和get_queryset**：`get_queryset`返回可用于详情页的候选查询集；`get_object`从queryset中取出一条记录。如果你在`get_object`中做权限检查（如raise PermissionDenied），比在`get_queryset`中过滤更严格。

# 十、核心总结：CBV速查表
## 常用CBV
| CBV | 作用 | 核心属性 |
|-----|------|---------|
| `ListView` | 对象列表 | model、template_name、paginate_by |
| `DetailView` | 对象详情 | model、pk_url_kwarg |
| `CreateView` | 创建对象 | model、form_class / fields、success_url |
| `UpdateView` | 更新对象 | model、form_class / fields |
| `DeleteView` | 删除对象 | model、success_url |
| `TemplateView` | 渲染模板 | template_name |

## CBV的URL配置
```python
# CBV必须用 .as_view()
path('articles/', ArticleListView.as_view(), name='article-list')
```

## Admin核心配置
```python
@admin.register(MyModel)
class MyModelAdmin(admin.ModelAdmin):
    list_display = ['field1', 'field2']     # 列表显示列
    list_filter = ['field1']                # 侧边筛选
    search_fields = ['field1', 'field2']    # 搜索框
    ordering = ['-field1']                  # 排序
    fieldsets = [...]                       # 详情页布局
```

## 常用方法重写顺序
```
dispatch() → 根据method分发
  ├── get() → get_queryset() → get_context_data() → render
  ├── post() → get_form() → form_valid() / form_invalid()
  └── ...
```

# 十一、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
