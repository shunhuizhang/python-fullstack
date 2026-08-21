
# Python全栈入门到实战【Django篇 13】博客实战（二）：首页与文章列表，打通视图模板完整流程
上一篇《Django篇 12》中，我们完成了博客系统的需求分析和数据模型设计——定义了Article、Category、Tag、Comment四张核心表，配置好了Admin后台。现在模型中已经有了数据（通过Shell创建的测试文章），但用户还看不到任何页面。

从本篇开始，我们要让数据"活起来"——通过**URL → View → Template**这条完整的链路，把数据库中的文章展示给用户。这包括：首页显示最新文章列表、按分类筛选文章、按标签筛选文章、搜索文章、分页导航、侧边栏展示热门文章和分类。

本篇的目标是：当你在浏览器中输入`http://127.0.0.1:8000/`时，看到一个美观的博客首页，上面有文章卡片、分类列表、标签云。

本节核心学习内容：
1.  项目模板架构：公共母版base.html的完整设计
2.  首页视图：ListView实现最新文章列表+分页
3.  分类与标签视图：按分类/标签筛选文章
4.  搜索视图：Q对象全文搜索
5.  上下文处理器：全局注入分类、标签、热门文章数据
6.  模板实现：文章卡片、侧边栏、分页导航组件

# 一、项目模板架构设计
## 1.1 母版模板（base.html）
在项目根目录创建`templates/base.html`作为所有页面的公共骨架：

```html
<!-- templates/base.html -->
{% load static %}
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Python全栈之路{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    {% block extra_css %}{% endblock %}
</head>
<body>
    <!-- 顶部导航栏 -->
    <nav class="navbar">
        <div class="container navbar-inner">
            <a href="{% url 'blog:index' %}" class="navbar-brand">Python全栈之路</a>
            <ul class="nav-links">
                <li><a href="{% url 'blog:index' %}">首页</a></li>
                <li><a href="{% url 'blog:article-list' %}">文章</a></li>
                <li><a href="{% url 'blog:search' %}">搜索</a></li>
                {% if user.is_authenticated %}
                    <li><a href="{% url 'blog:article-create' %}">写文章</a></li>
                    <li>
                        <a href="{% url 'users:profile' %}">{{ user.username }}</a>
                        <form method="POST" action="{% url 'users:logout' %}" style="display:inline">
                            {% csrf_token %}
                            <button type="submit" class="link-btn">退出</button>
                        </form>
                    </li>
                {% else %}
                    <li><a href="{% url 'users:login' %}">登录</a></li>
                {% endif %}
            </ul>
        </div>
    </nav>

    <!-- 主内容区 -->
    <main class="container">
        <!-- 消息提示 -->
        {% if messages %}
        <div class="messages">
            {% for message in messages %}
                <div class="alert alert-{{ message.tags }}">{{ message }}</div>
            {% endfor %}
        </div>
        {% endif %}

        <div class="main-layout">
            <!-- 左侧内容区 -->
            <div class="content-area">
                {% block content %}{% endblock %}
            </div>

            <!-- 右侧侧边栏 -->
            <aside class="sidebar">
                {% include 'blog/_sidebar.html' %}
            </aside>
        </div>
    </main>

    <!-- 底部 -->
    <footer class="footer">
        <div class="container">
            <p>&copy; {% now "Y" %} Python全栈之路 - Django博客实战</p>
        </div>
    </footer>

    <script src="{% static 'js/main.js' %}"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>
```

## 1.2 侧边栏组件（_sidebar.html）
```html
<!-- templates/blog/_sidebar.html -->
<div class="sidebar-card">
    <h3>全站搜索</h3>
    <form action="{% url 'blog:search' %}" method="GET" class="search-sidebar">
        <input type="text" name="q" placeholder="搜索文章..." value="{{ request.GET.q }}">
        <button type="submit">搜索</button>
    </form>
</div>

<div class="sidebar-card">
    <h3>文章分类</h3>
    <ul class="category-list">
        {% for category in global_categories %}
        <li>
            <a href="{% url 'blog:category-detail' category.slug %}">
                {{ category.name }}
                <span class="count">({{ category.article_count }})</span>
            </a>
        </li>
        {% empty %}
        <li>暂无分类</li>
        {% endfor %}
    </ul>
</div>

<div class="sidebar-card">
    <h3>标签云</h3>
    <div class="tag-cloud">
        {% for tag in global_tags %}
        <a href="{% url 'blog:tag-detail' tag.slug %}" class="tag-item">{{ tag.name }}</a>
        {% endfor %}
    </div>
</div>

<div class="sidebar-card">
    <h3>热门文章</h3>
    <ul class="hot-list">
        {% for article in global_hot_articles %}
        <li>
            <a href="{% url 'blog:article-detail' article.slug %}">{{ article.title }}</a>
            <span class="views">{{ article.views }} 阅读</span>
        </li>
        {% endfor %}
    </ul>
</div>
```

# 二、上下文处理器：全局数据注入
侧边栏的分类、标签、热门文章在每个页面都需要，通过上下文处理器统一注入：

```python
# blog/context_processors.py
from .models import Category, Tag, Article
from django.db import models
from django.db.models import Count

def global_sidebar_data(request):
    """给所有模板提供侧边栏需要的全局数据"""
    categories = Category.objects.annotate(
        article_count=Count('articles', filter=models.Q(articles__status='published'))
    ).order_by('name')

    tags = Tag.objects.annotate(
        article_count=Count('articles', filter=models.Q(articles__status='published'))
    ).order_by('name')

    hot_articles = Article.objects.filter(
        status='published'
    ).order_by('-views')[:8]

    return {
        'global_categories': categories,
        'global_tags': tags,
        'global_hot_articles': hot_articles,
    }
```

注册到settings.py：
```python
TEMPLATES = [
    {
        'OPTIONS': {
            'context_processors': [
                # ... Django内置的
                'blog.context_processors.global_sidebar_data',  # 新增
            ],
        },
    },
]
```

# 三、URL路由配置
```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # 首页
    path('', views.IndexView.as_view(), name='index'),

    # 文章列表页
    path('articles/', views.ArticleListView.as_view(), name='article-list'),
    path('articles/<slug:slug>/', views.ArticleDetailView.as_view(), name='article-detail'),
    path('articles/create/', views.ArticleCreateView.as_view(), name='article-create'),
    path('articles/<slug:slug>/edit/', views.ArticleUpdateView.as_view(), name='article-edit'),
    path('articles/<slug:slug>/delete/', views.ArticleDeleteView.as_view(), name='article-delete'),

    # 分类与标签
    path('category/<slug:slug>/', views.CategoryDetailView.as_view(), name='category-detail'),
    path('tag/<slug:slug>/', views.TagDetailView.as_view(), name='tag-detail'),

    # 搜索
    path('search/', views.SearchView.as_view(), name='search'),
]
```

# 四、视图实现
## 4.1 首页视图
```python
# blog/views.py
from django.views.generic import ListView, DetailView
from django.db.models import Count, Q
from .models import Article, Category, Tag

class IndexView(ListView):
    """首页：最新发布的文章列表"""
    model = Article
    template_name = 'blog/index.html'
    context_object_name = 'articles'
    paginate_by = 6

    def get_queryset(self):
        return Article.objects.filter(
            status='published'
        ).select_related('category', 'author').prefetch_related('tags').order_by(
            '-created_at'
        )

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        # 精选文章（阅读量最高的5篇）
        context['featured_articles'] = Article.objects.filter(
            status='published'
        ).order_by('-views')[:5]
        return context
```

## 4.2 文章列表视图
```python
class ArticleListView(ListView):
    """文章列表页"""
    model = Article
    template_name = 'blog/article_list.html'
    context_object_name = 'articles'
    paginate_by = 10

    def get_queryset(self):
        return Article.objects.filter(
            status='published'
        ).select_related('category', 'author').order_by('-created_at')
```

## 4.3 分类与标签视图
```python
class CategoryDetailView(ListView):
    """按分类浏览文章"""
    template_name = 'blog/category_detail.html'
    context_object_name = 'articles'
    paginate_by = 10

    def get_queryset(self):
        slug = self.kwargs.get('slug')
        return Article.objects.filter(
            category__slug=slug,
            status='published'
        ).select_related('author')

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['category'] = Category.objects.get(slug=self.kwargs['slug'])
        return context

class TagDetailView(ListView):
    """按标签浏览文章"""
    template_name = 'blog/tag_detail.html'
    context_object_name = 'articles'
    paginate_by = 10

    def get_queryset(self):
        slug = self.kwargs.get('slug')
        return Article.objects.filter(
            tags__slug=slug,
            status='published'
        ).distinct().select_related('author')

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['tag'] = Tag.objects.get(slug=self.kwargs['slug'])
        return context
```

## 4.4 搜索视图
```python
class SearchView(ListView):
    """搜索文章"""
    template_name = 'blog/search_results.html'
    context_object_name = 'articles'
    paginate_by = 10

    def get_queryset(self):
        query = self.request.GET.get('q', '').strip()
        if not query:
            return Article.objects.none()  # 空查询返回空结果
        return Article.objects.filter(
            Q(title__icontains=query) | Q(content__icontains=query),
            status='published'
        ).select_related('author').order_by('-created_at')

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['query'] = self.request.GET.get('q', '')
        return context
```

# 五、模板实现
## 5.1 首页模板（index.html）
```html
<!-- templates/blog/index.html -->
{% extends 'base.html' %}

{% block title %}Python全栈之路 - 博客首页{% endblock %}

{% block content %}
<div class="section-header">
    <h1>最新文章</h1>
</div>

{% if featured_articles %}
<div class="featured-grid">
    {% for article in featured_articles %}
    <article class="featured-card">
        {% if article.cover %}
            <img src="{{ article.cover.url }}" alt="{{ article.title }}" class="featured-img">
        {% endif %}
        <div class="featured-body">
            <span class="category-badge">{{ article.category.name }}</span>
            <h2><a href="{% url 'blog:article-detail' article.slug %}">{{ article.title }}</a></h2>
            <div class="meta">
                <span>{{ article.author.username }}</span>
                <span>{{ article.created_at|date:"Y-m-d" }}</span>
                <span>{{ article.views }} 阅读</span>
            </div>
        </div>
    </article>
    {% endfor %}
</div>
{% endif %}

<div class="article-grid">
    {% for article in articles %}
    <article class="article-card">
        {% if article.cover %}
            <img src="{{ article.cover.url }}" alt="{{ article.title }}" class="card-img">
        {% else %}
            <div class="card-img-placeholder">暂无封面</div>
        {% endif %}
        <div class="card-body">
            <div class="card-meta-top">
                <a href="{% url 'blog:category-detail' article.category.slug %}" class="category-link">
                    {{ article.category.name }}
                </a>
                <span class="date">{{ article.created_at|date:"Y-m-d" }}</span>
            </div>
            <h3><a href="{% url 'blog:article-detail' article.slug %}">{{ article.title }}</a></h3>
            {% if article.tags.all %}
            <div class="card-tags">
                {% for tag in article.tags.all %}
                <a href="{% url 'blog:tag-detail' tag.slug %}" class="tag-link">{{ tag.name }}</a>
                {% endfor %}
            </div>
            {% endif %}
        </div>
    </article>
    {% empty %}
    <div class="empty-state">
        <p>暂无文章</p>
    </div>
    {% endfor %}
</div>

<!-- 分页导航 -->
{% include 'blog/_pagination.html' %}
{% endblock %}
```

## 5.2 分页导航组件（_pagination.html）
```html
<!-- templates/blog/_pagination.html -->
{% if is_paginated %}
<nav class="pagination">
    {% if page_obj.has_previous %}
        <a href="?page={{ page_obj.previous_page_number }}{% if request.GET.q %}&q={{ request.GET.q }}{% endif %}" class="page-link">上一页</a>
    {% endif %}

    {% for num in paginator.page_range %}
        {% if num == page_obj.number %}
            <span class="page-link active">{{ num }}</span>
        {% elif num > page_obj.number|add:'-3' and num < page_obj.number|add:'3' %}
            <a href="?page={{ num }}{% if request.GET.q %}&q={{ request.GET.q }}{% endif %}" class="page-link">{{ num }}</a>
        {% endif %}
    {% endfor %}

    {% if page_obj.has_next %}
        <a href="?page={{ page_obj.next_page_number }}{% if request.GET.q %}&q={{ request.GET.q }}{% endif %}" class="page-link">下一页</a>
    {% endif %}

    <span class="page-info">共 {{ paginator.count }} 条</span>
</nav>
{% endif %}
```

## 5.3 CSS样式节选
```css
/* static/css/style.css */
:root {
    --primary: #3498db;
    --text: #333;
    --text-light: #666;
    --bg: #f5f7fa;
    --card-bg: #fff;
    --border: #e8e8e8;
    --radius: 8px;
}

* { margin: 0; padding: 0; box-sizing: border-box; }
body { font-family: "微软雅黑", sans-serif; background: var(--bg); color: var(--text); line-height: 1.6; }
a { text-decoration: none; color: inherit; }

.container { max-width: 1200px; margin: 0 auto; padding: 0 20px; }

/* 导航栏 */
.navbar { background: #1a1a2e; color: white; padding: 15px 0; position: sticky; top: 0; z-index: 100; }
.navbar-inner { display: flex; justify-content: space-between; align-items: center; }
.navbar-brand { font-size: 20px; font-weight: bold; color: var(--primary); }
.nav-links { display: flex; list-style: none; gap: 20px; align-items: center; }
.nav-links a { color: #ccc; font-size: 14px; transition: color 0.3s; }
.nav-links a:hover { color: white; }
.link-btn { background: none; border: none; color: #ccc; cursor: pointer; font-size: 14px; }
.link-btn:hover { color: white; }

/* 主布局 */
.main-layout { display: grid; grid-template-columns: 1fr 300px; gap: 30px; margin: 30px 0; }
@media (max-width: 768px) { .main-layout { grid-template-columns: 1fr; } }

/* 文章卡片 */
.article-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; margin-top: 20px; }
.article-card { background: var(--card-bg); border-radius: var(--radius); overflow: hidden; box-shadow: 0 1px 4px rgba(0,0,0,0.06); transition: transform 0.3s; }
.article-card:hover { transform: translateY(-3px); box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
.card-img { width: 100%; height: 180px; object-fit: cover; }
.card-body { padding: 15px; }
.card-meta-top { display: flex; justify-content: space-between; font-size: 12px; color: var(--text-light); margin-bottom: 8px; }
.card-body h3 { font-size: 16px; margin-bottom: 8px; line-height: 1.4; }
.card-body h3 a:hover { color: var(--primary); }
.card-tags { display: flex; flex-wrap: wrap; gap: 5px; }
.tag-link { font-size: 11px; padding: 2px 8px; background: #eaf4fe; color: var(--primary); border-radius: 3px; }
.tag-link:hover { background: var(--primary); color: white; }

/* 侧边栏 */
.sidebar-card { background: var(--card-bg); border-radius: var(--radius); padding: 18px; margin-bottom: 20px; box-shadow: 0 1px 4px rgba(0,0,0,0.06); }
.sidebar-card h3 { font-size: 15px; margin-bottom: 12px; padding-bottom: 8px; border-bottom: 2px solid var(--primary); }
.category-list { list-style: none; }
.category-list li { padding: 6px 0; font-size: 14px; border-bottom: 1px solid #f0f0f0; }
.category-list .count { color: #999; font-size: 12px; }
.tag-cloud { display: flex; flex-wrap: wrap; gap: 6px; }
.tag-cloud .tag-item { font-size: 12px; padding: 3px 10px; background: #f0f0f0; border-radius: 15px; }
.tag-cloud .tag-item:hover { background: var(--primary); color: white; }
.hot-list { list-style: none; }
.hot-list li { padding: 8px 0; border-bottom: 1px solid #f0f0f0; font-size: 14px; }
.hot-list .views { display: block; font-size: 11px; color: #999; }

/* 分页 */
.pagination { display: flex; justify-content: center; align-items: center; gap: 5px; margin: 40px 0 20px; }
.page-link { padding: 6px 12px; border: 1px solid var(--border); border-radius: 4px; font-size: 14px; }
.page-link.active { background: var(--primary); color: white; border-color: var(--primary); }
.page-info { font-size: 13px; color: #999; margin-left: 15px; }

/* 消息系统 */
.messages { margin: 15px 0; }
.alert { padding: 12px 15px; border-radius: 5px; font-size: 14px; margin-bottom: 10px; }
.alert-success { background: #d4edda; color: #155724; }
.alert-error { background: #f8d7da; color: #721c24; }
```

# 六、当前项目进度
```
阶段一：数据模型设计 ✓
    ├── Category、Tag、Article、Comment ✓
    ├── Admin后台配置 ✓
    └── 测试数据填充 ✓

阶段二：前端展示 ✓
    ├── base.html母版 ✓
    ├── 首页文章列表 ✓
    ├── 分类/标签筛选 ✓
    ├── 搜索功能 ✓
    ├── 分页导航 ✓
    └── 侧边栏组件 ✓

阶段三：文章详情（下一篇）
    待续...
```

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
