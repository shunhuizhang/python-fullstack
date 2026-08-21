
# Python全栈入门到实战【Django篇 14】博客实战（三）：文章详情页，Markdown渲染与阅读体验优化
上一篇《Django篇 13》中，我们完成了博客首页和文章列表——用户可以浏览文章、按分类和标签筛选、搜索。但点击文章标题后，我们还缺一个最重要的页面：**文章详情页**。这相当于一个新闻网站点进标题后看到的正文内容——它应该是博客系统中信息最丰富、排版最讲究的页面。

一个优秀的文章详情页不仅仅是"把文章内容显示出来"，它还需要：支持Markdown渲染文章格式（标题、列表、代码块、图片）、显示文章目录方便快速导航、记录阅读量、提供上一篇/下一篇的导航、展示文章元信息（作者、发布时间、标签、阅读量）。这些都是提升阅读体验的细节。

本篇将一步步实现一个功能齐全的文章详情页。学完这篇后，你的博客系统就具备了"内容展示"的完整能力。

本节核心学习内容：
1.  文章详情页视图：DetailView + slug查询
2.  Markdown渲染：集成markdown库 + 代码语法高亮
3.  文章目录生成：从Markdown提取h2/h3标题自动生成锚点导航
4.  阅读量计数：F对象原子自增与防刷
5.  上一篇/下一篇：相邻文章导航
6.  文章元信息展示与SEO优化
7.  文章发布与编辑表单

# 一、文章详情视图
## 1.1 DetailView实现
```python
# blog/views.py
from django.views.generic import DetailView
from django.shortcuts import get_object_or_404
from django.db.models import F
from .models import Article
import markdown
import re

class ArticleDetailView(DetailView):
    """文章详情页"""
    model = Article
    template_name = 'blog/article_detail.html'
    context_object_name = 'article'

    def get_object(self, queryset=None):
        """通过slug获取文章并增加阅读量"""
        slug = self.kwargs.get('slug')
        article = get_object_or_404(
            Article.objects.select_related('author', 'category').prefetch_related('tags'),
            slug=slug,
            status='published'
        )
        # 阅读量+1（F对象原子操作）
        Article.objects.filter(pk=article.pk).update(views=F('views') + 1)
        article.refresh_from_db()
        return article

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        article = self.object

        # 1. Markdown渲染文章内容
        context['content_html'] = self.render_markdown(article.content)

        # 2. 生成文章目录（TOC）
        context['toc'] = self.generate_toc(article.content)

        # 3. 上一篇/下一篇
        context['prev_article'] = Article.objects.filter(
            created_at__lt=article.created_at,
            status='published'
        ).order_by('-created_at').first()

        context['next_article'] = Article.objects.filter(
            created_at__gt=article.created_at,
            status='published'
        ).order_by('created_at').first()

        # 4. 相关推荐（同分类的文章，排除当前）
        context['related_articles'] = Article.objects.filter(
            category=article.category,
            status='published'
        ).exclude(pk=article.pk).order_by('-views')[:4]

        return context

    def render_markdown(self, content):
        """将Markdown文本转为HTML"""
        md = markdown.Markdown(extensions=[
            'extra',                    # 表格、定义列表等扩展语法
            'codehilite',               # 代码语法高亮
            'toc',                      # 自动生成目录
            'fenced_code',              # 围栏代码块（```...```）
            'nl2br',                    # 换行转<br>
        ])
        html = md.convert(content)
        return html

    def generate_toc(self, content):
        """从Markdown内容中提取h2/h3标题生成目录"""
        toc_items = []
        lines = content.split('\n')
        for line in lines:
            # 匹配 Markdown 的标题行（## 开头 或 ### 开头）
            match = re.match(r'^(#{2,3})\s+(.+)$', line)
            if match:
                level = len(match.group(1))  # 2 = h2, 3 = h3
                text = match.group(2).strip()
                # 生成锚点ID（和markdown库生成的锚点格式一致）
                anchor = re.sub(r'[^\w\s-]', '', text).strip().lower()
                anchor = re.sub(r'[\s]+', '-', anchor)
                toc_items.append({
                    'level': level,
                    'text': text,
                    'anchor': anchor,
                })
        return toc_items
```

## 1.2 安装markdown支持库
```bash
pip install markdown
pip install Pygments  # 代码语法高亮依赖
pip install django-markdown-deux  # Django模板中使用的markdown过滤器
```

# 二、文章详情模板
```html
<!-- templates/blog/article_detail.html -->
{% extends 'base.html' %}
{% load static %}

{% block title %}{{ article.title }} - Python全栈之路{% endblock %}

{% block extra_css %}
<link rel="stylesheet" href="{% static 'css/pygments.css' %}">
{% endblock %}

{% block content %}
<!-- 文章头部 -->
<article class="article-detail">
    <header class="article-header">
        <!-- 面包屑导航 -->
        <nav class="breadcrumb">
            <a href="{% url 'blog:index' %}">首页</a>
            <span>/</span>
            <a href="{% url 'blog:category-detail' article.category.slug %}">{{ article.category.name }}</a>
            <span>/</span>
            <span>{{ article.title }}</span>
        </nav>

        <h1 class="article-title">{{ article.title }}</h1>

        <div class="article-meta">
            <div class="author-info">
                <span class="author-name">{{ article.author.username }}</span>
            </div>
            <span class="meta-sep">·</span>
            <time datetime="{{ article.created_at|date:'Y-m-d' }}">
                发布于 {{ article.created_at|date:"Y年m月d日" }}
            </time>
            <span class="meta-sep">·</span>
            <span>{{ article.views }} 次阅读</span>
            {% if article.updated_at > article.created_at %}
            <span class="meta-sep">·</span>
            <span>更新于 {{ article.updated_at|date:"Y-m-d" }}</span>
            {% endif %}
        </div>

        {% if article.cover %}
        <div class="article-cover">
            <img src="{{ article.cover.url }}" alt="{{ article.title }}">
        </div>
        {% endif %}
    </header>

    <!-- 文章正文 + 侧边目录 -->
    <div class="article-layout">
        <!-- 正文 -->
        <div class="article-body markdown-body">
            {{ content_html|safe }}
        </div>

        <!-- 侧边目录导航 -->
        {% if toc %}
        <aside class="article-toc-sidebar">
            <div class="toc-container">
                <h4>文章目录</h4>
                <nav class="toc-nav">
                    {% for item in toc %}
                    <a href="#{{ item.anchor }}" class="toc-item toc-level-{{ item.level }}">
                        {{ item.text }}
                    </a>
                    {% endfor %}
                </nav>
            </div>
        </aside>
        {% endif %}
    </div>

    <!-- 文章标签 -->
    {% if article.tags.all %}
    <div class="article-tags">
        <span class="tags-label">标签：</span>
        {% for tag in article.tags.all %}
        <a href="{% url 'blog:tag-detail' tag.slug %}" class="tag-link">{{ tag.name }}</a>
        {% endfor %}
    </div>
    {% endif %}

    <!-- 文章导航（上一篇/下一篇） -->
    <nav class="article-nav">
        <div class="nav-prev">
            {% if prev_article %}
            <span class="nav-label">上一篇</span>
            <a href="{% url 'blog:article-detail' prev_article.slug %}">{{ prev_article.title }}</a>
            {% endif %}
        </div>
        <div class="nav-next">
            {% if next_article %}
            <span class="nav-label">下一篇</span>
            <a href="{% url 'blog:article-detail' next_article.slug %}">{{ next_article.title }}</a>
            {% endif %}
        </div>
    </nav>
</article>

<!-- 相关推荐 -->
{% if related_articles %}
<section class="related-articles">
    <h3>相关推荐</h3>
    <div class="article-grid">
        {% for article in related_articles %}
        <article class="article-card">
            {% if article.cover %}
                <img src="{{ article.cover.url }}" alt="{{ article.title }}" class="card-img">
            {% endif %}
            <div class="card-body">
                <h3><a href="{% url 'blog:article-detail' article.slug %}">{{ article.title }}</a></h3>
                <div class="meta">
                    <span>{{ article.created_at|date:"Y-m-d" }}</span>
                    <span>{{ article.views }} 阅读</span>
                </div>
            </div>
        </article>
        {% endfor %}
    </div>
</section>
{% endif %}

<!-- 评论区（下一篇实现） -->
<section class="comments-section">
    <h3>评论</h3>
    <p class="placeholder">评论区将在下一篇"评论系统"中实现</p>
</section>
{% endblock %}
```

# 三、文章发布与编辑
## 3.1 创建/编辑表单
```python
# blog/forms.py
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'category', 'tags', 'content', 'cover', 'status']

        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': '请输入文章标题'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 20,
                'placeholder': '支持Markdown语法...\n\n# 一级标题\n## 二级标题\n\n段落文字...\n\n```python\nprint("代码块")\n```'
            }),
            'category': forms.Select(attrs={'class': 'form-control'}),
            'tags': forms.SelectMultiple(attrs={'class': 'form-control'}),
            'status': forms.Select(attrs={'class': 'form-control'}),
            'cover': forms.FileInput(attrs={'class': 'form-control-file'}),
        }
```

## 3.2 视图
```python
# blog/views.py
from django.views.generic import CreateView, UpdateView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.urls import reverse_lazy
from django.utils.text import slugify
from django.contrib import messages
from .forms import ArticleForm
from .models import Article

class ArticleCreateView(LoginRequiredMixin, CreateView):
    model = Article
    form_class = ArticleForm
    template_name = 'blog/article_form.html'
    login_url = '/users/login/'

    def form_valid(self, form):
        """表单验证通过后的处理"""
        form.instance.author = self.request.user
        # 如果标题有变动，重新生成slug（确保唯一） 如果slug冲突怎么办）
        if not form.instance.slug:
            form.instance.slug = slugify(form.instance.title)
        # 确保slug唯一
        base_slug = form.instance.slug
        counter = 1
        while Article.objects.filter(slug=form.instance.slug).exists():
            form.instance.slug = f"{base_slug}-{counter}"
            counter += 1
        messages.success(self.request, '文章发布成功！')
        return super().form_valid(form)

class ArticleUpdateView(LoginRequiredMixin, UpdateView):
    model = Article
    form_class = ArticleForm
    template_name = 'blog/article_form.html'
    login_url = '/users/login/'

    def get_queryset(self):
        """只能编辑自己的文章"""
        return Article.objects.filter(author=self.request.user)

    def form_valid(self, form):
        messages.success(self.request, '文章更新成功！')
        return super().form_valid(form)
```

## 3.3 模板（创建/编辑共用）
```html
<!-- templates/blog/article_form.html -->
{% extends 'base.html' %}

{% block title %}{% if form.instance.pk %}编辑文章{% else %}发布文章{% endif %} - Python全栈之路{% endblock %}

{% block content %}
<div class="form-container">
    <h1>{% if form.instance.pk %}编辑文章{% else %}发布文章{% endif %}</h1>

    <form method="POST" enctype="multipart/form-data">
        {% csrf_token %}

        <div class="form-group">
            <label>{{ form.title.label }}</label>
            {{ form.title }}
            {% if form.title.errors %}
                <span class="error-text">{{ form.title.errors.0 }}</span>
            {% endif %}
        </div>

        <div class="form-row">
            <div class="form-group half">
                <label>{{ form.category.label }}</label>
                {{ form.category }}
            </div>
            <div class="form-group half">
                <label>{{ form.status.label }}</label>
                {{ form.status }}
            </div>
        </div>

        <div class="form-group">
            <label>{{ form.tags.label }}</label>
            {{ form.tags }}
        </div>

        <div class="form-group">
            <label>{{ form.cover.label }}</label>
            {{ form.cover }}
            {% if form.instance.cover %}
                <p class="help-text">当前封面：{{ form.instance.cover.name }}</p>
            {% endif %}
        </div>

        <div class="form-group">
            <label>{{ form.content.label }}</label>
            {{ form.content }}
            {% if form.content.errors %}
                <span class="error-text">{{ form.content.errors.0 }}</span>
            {% endif %}
            <p class="help-text">支持Markdown语法编写文章</p>
        </div>

        <div class="form-actions">
            <button type="submit" class="btn-primary">
                {% if form.instance.pk %}更新文章{% else %}发布文章{% endif %}
            </button>
            {% if form.instance.pk %}
            <a href="{% url 'blog:article-detail' form.instance.slug %}" class="btn-secondary">取消</a>
            {% endif %}
        </div>
    </form>
</div>
{% endblock %}
```

# 四、markdown-body样式
```css
/* 文章正文样式（附加到上篇的style.css中） */
.markdown-body {
    font-size: 15px;
    line-height: 1.8;
    color: #333;
    word-wrap: break-word;
}
.markdown-body h1 { font-size: 28px; margin: 30px 0 15px; border-bottom: 2px solid #eee; padding-bottom: 8px; }
.markdown-body h2 { font-size: 22px; margin: 25px 0 12px; border-bottom: 1px solid #eee; padding-bottom: 6px; }
.markdown-body h3 { font-size: 18px; margin: 20px 0 10px; }
.markdown-body p { margin: 12px 0; }
.markdown-body ul, .markdown-body ol { padding-left: 25px; margin: 12px 0; }
.markdown-body li { margin: 5px 0; }
.markdown-body img { max-width: 100%; border-radius: 5px; margin: 10px 0; }
.markdown-body blockquote {
    border-left: 4px solid var(--primary);
    background: #f8faff;
    padding: 12px 20px;
    margin: 15px 0;
    color: #555;
    border-radius: 0 5px 5px 0;
}
.markdown-body code {
    background: #f0f0f0;
    padding: 2px 6px;
    border-radius: 3px;
    font-family: Consolas, monospace;
    font-size: 13px;
    color: #e74c3c;
}
.markdown-body pre {
    background: #1e1e1e;
    color: #d4d4d4;
    padding: 20px;
    border-radius: 8px;
    overflow-x: auto;
    margin: 15px 0;
    font-size: 14px;
    line-height: 1.5;
}
.markdown-body pre code { background: none; padding: 0; color: inherit; font-size: inherit; }
.markdown-body table { border-collapse: collapse; width: 100%; margin: 15px 0; }
.markdown-body th, .markdown-body td { border: 1px solid #ddd; padding: 8px 12px; text-align: left; }
.markdown-body th { background: #f5f5f5; font-weight: bold; }

/* Pygments代码高亮样式（VS Code Dark主题配色） */
.codehilite { background: #1e1e1e; color: #d4d4d4; padding: 20px; border-radius: 8px; overflow-x: auto; margin: 15px 0; font-size: 13px; line-height: 1.6; }
.codehilite pre { background: none; padding: 0; margin: 0; color: inherit; overflow: visible; }
.codehilite .hll { background-color: #ffffcc }
.codehilite .c   { color: #6a9955; font-style: italic } /* Comment */
.codehilite .k   { color: #569cd6 } /* Keyword */
.codehilite .o   { color: #d4d4d4 } /* Operator */
.codehilite .s   { color: #ce9178 } /* String */
.codehilite .p   { color: #d4d4d4 } /* Punctuation */
.codehilite .n   { color: #9cdcfe } /* Name */
.codehilite .mi  { color: #b5cea8 } /* Number */
.codehilite .nf  { color: #dcdcaa } /* Function */
.codehilite .kd  { color: #569cd6 } /* Keyword.Declaration */
.codehilite .nb  { color: #9cdcfe } /* Builtin */
.codehilite .kn  { color: #569cd6 } /* Keyword.Namespace */
.codehilite .bp  { color: #9cdcfe } /* Builtin.Pseudo */
.codehilite .nc  { color: #4ec9b0 } /* Name.Class */
.codehilite .nt  { color: #569cd6 } /* Name.Tag */
.codehilite .na  { color: #9cdcfe } /* Name.Attribute */
.codehilite .fm  { color: #dcdcaa } /* Function.Magic */
.codehilite .si  { color: #ce9178 } /* String.Interpol */
.codehilite .err { color: #f44747 } /* Error */

/* 文章目录侧边栏 */
.article-layout {
    display: grid;
    grid-template-columns: 1fr 220px;
    gap: 30px;
    margin: 30px 0;
}
@media (max-width: 1024px) {
    .article-layout { grid-template-columns: 1fr; }
    .article-toc-sidebar { display: none; }
}
.article-toc-sidebar { position: sticky; top: 80px; align-self: start; }
.toc-container {
    background: #fafafa;
    border-radius: 5px;
    padding: 15px;
    border-left: 3px solid var(--primary);
}
.toc-container h4 { font-size: 14px; margin-bottom: 10px; color: #333; }
.toc-nav a {
    display: block;
    padding: 4px 0;
    font-size: 13px;
    color: #666;
    transition: color 0.2s;
}
.toc-nav a:hover { color: var(--primary); }
.toc-level-2 { padding-left: 0; font-weight: 500; }
.toc-level-3 { padding-left: 15px; font-size: 12px; }

/* 文章导航 */
.article-nav { display: flex; justify-content: space-between; margin: 30px 0; padding: 20px 0; border-top: 1px solid #eee; }
.nav-label { display: block; font-size: 12px; color: #999; margin-bottom: 5px; }
.article-nav a { color: var(--primary); font-size: 14px; }

/* 相关推荐 */
.related-articles { margin: 40px 0; }
.related-articles h3 { margin-bottom: 20px; padding-bottom: 8px; border-bottom: 2px solid var(--primary); }
```

# 五、常见误区与避坑指南
1.  **Markdown渲染时忘记使用safe过滤器**：`{{ content_html|safe }}`中的`safe`是必要的——它告诉Django这个HTML是可信的，不需要转义。但你**必须确保**只有你自己的文章内容（不是用户输入的评论）才用safe。

2.  **阅读量用save()而不是update()**：高并发时`article.views += 1; article.save()`会丢失更新。始终用`Article.objects.filter(pk=id).update(views=F('views') + 1)`做原子操作。

3.  **slug生成后不检查唯一性**：两篇文章的标题相似会生成相同的slug。在保存时检查并追加后缀（如`django-tutorial-1`, `django-tutorial-2`）。

4.  **DetailView中通过pk而不是slug查询**：文章详情页用slug比用pk更友好（SEO和可读性）。将`pk_url_kwarg`替换为`slug_url_kwarg = 'slug'`。

5.  **编辑文章时修改了slug会改变URL**：如果已发布的文章改了slug，原来的URL会失效（除非你设置301重定向）。建议slug生成后不再自动修改，或者只随标题首次生成。

# 六、核心总结
## 文章详情页功能清单
| 功能 | 实现方式 |
|------|---------|
| Markdown渲染 | `markdown`库 + `extra`/`codehilite`/`fenced_code`扩展 |
| 代码语法高亮 | Pygments + codehilite |
| 文章目录 | 正则提取h2/h3标题 → 生成锚点链接 |
| 阅读量 | `update(views=F('views') + 1)` 原子自增 |
| 上一篇/下一篇 | `created_at__lt` / `created_at__gt` 邻接查询 |
| 相关推荐 | 同分类按阅读量排序取前4 |
| 发布/编辑 | CreateView/UpdateView + ModelForm |

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
