
# Python全栈入门到实战【Django篇 19】教育平台实战（二）：课程列表与多条件筛选，构建课程发现系统
上一篇《Django篇 18》中，我们完成了教育平台的需求分析和数据模型设计——定义了Course、Chapter、Video、Order等核心模型。现在数据库中已经有了课程数据（通过Admin添加或Shell填充），但用户还看不到任何课程页面。

和博客的文章列表不同，教育平台的课程列表更像一个"电商货架"——用户需要按分类筛选、按难度筛选、按价格排序、按评分排序、按学员数排序、通过关键词搜索。多个筛选条件需要**同时生效**（URL参数叠加），不能相互覆盖。这比博客的单一分类筛选要复杂。

本篇将实现一个功能齐全的课程列表页，支持多条件组合筛选、排序切换、分页、搜索，以及美观的课程卡片展示。

本节核心学习内容：
1.  课程列表ListView：多条件filter叠加查询
2.  多条件筛选URL设计：参数保留与组合
3.  排序切换：最新/最热/价格/评分
4.  全文搜索：Q对象组合
5.  课程卡片：封面/标题/教师/价格/评分/标签展示
6.  侧边栏：分类导航+热门课程

# 一、URL与基础配置
```python
# courses/urls.py
from django.urls import path
from . import views

app_name = 'courses'

urlpatterns = [
    path('', views.CourseListView.as_view(), name='course-list'),
    path('<slug:slug>/', views.CourseDetailView.as_view(), name='course-detail'),
    path('category/<slug:slug>/', views.CourseListView.as_view(), name='category-detail'),
    path('search/', views.CourseSearchView.as_view(), name='search'),
]
```

```python
# 项目 urls.py
urlpatterns = [
    path('', include('blog.urls')),
    path('users/', include('users.urls')),
    path('courses/', include('courses.urls')),
]
```

# 二、课程列表视图
```python
# courses/views.py
from django.views.generic import ListView, DetailView
from django.db.models import Q, Count, Avg
from .models import Course, Category

class CourseListView(ListView):
    """课程列表页（支持多条件筛选）"""
    model = Course
    template_name = 'courses/course_list.html'
    context_object_name = 'courses'
    paginate_by = 12

    def get_queryset(self):
        queryset = Course.objects.filter(
            status='published'
        ).select_related('category').prefetch_related('teachers')

        # 1. 分类筛选（从URL参数获取）
        category = self.request.GET.get('category', '')
        if category:
            queryset = queryset.filter(category__slug=category)

        # 2. 难度筛选
        level = self.request.GET.get('level', '')
        if level:
            queryset = queryset.filter(level=level)

        # 3. 价格区间筛选
        price_min = self.request.GET.get('price_min', '')
        price_max = self.request.GET.get('price_max', '')
        if price_min:
            queryset = queryset.filter(price__gte=float(price_min))
        if price_max:
            queryset = queryset.filter(price__lte=float(price_max))

        # 4. 关键词搜索
        keyword = self.request.GET.get('q', '')
        if keyword:
            queryset = queryset.filter(
                Q(title__icontains=keyword) |
                Q(description__icontains=keyword)
            )

        # 5. 排序
        sort = self.request.GET.get('sort', '-created_at')
        valid_sorts = ['-created_at', 'created_at', '-total_students',
                       'price', '-price', '-average_rating']
        if sort in valid_sorts:
            queryset = queryset.order_by(sort)

        return queryset

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        # 所有分类（给筛选侧边栏用）
        context['categories'] = Category.objects.annotate(
            course_count=Count('courses', filter=Q(courses__status='published'))
        )
        # 保留当前筛选参数（分页时拼接到URL中）
        context['current_params'] = self.request.GET.urlencode()
        # 当前激活的筛选条件
        context['current_category'] = self.request.GET.get('category', '')
        context['current_level'] = self.request.GET.get('level', '')
        context['current_sort'] = self.request.GET.get('sort', '-created_at')
        context['current_keyword'] = self.request.GET.get('q', '')
        return context
```

# 三、课程列表模板
```html
<!-- templates/courses/course_list.html -->
{% extends 'base.html' %}

{% block title %}
    {% if current_category %} Python在线教育 - 分类课程
    {% else %} Python在线教育 - 全部课程
    {% endif %}
{% endblock %}

{% block content %}
<div class="course-page">
    <!-- 顶部筛选栏 -->
    <div class="filter-bar">
        <div class="filter-left">
            <span class="filter-label">难度：</span>
            <a href="?{% if current_category %}category={{ current_category }}&{% endif %}{% if current_keyword %}q={{ current_keyword }}&{% endif %}sort={{ current_sort }}"
               class="filter-option {% if not current_level %}active{% endif %}">全部</a>
            <a href="?category={{ current_category }}&level=beginner&sort={{ current_sort }}"
               class="filter-option {% if current_level == 'beginner' %}active{% endif %}">入门</a>
            <a href="?category={{ current_category }}&level=intermediate&sort={{ current_sort }}"
               class="filter-option {% if current_level == 'intermediate' %}active{% endif %}">中级</a>
            <a href="?category={{ current_category }}&level=advanced&sort={{ current_sort }}"
               class="filter-option {% if current_level == 'advanced' %}active{% endif %}">高级</a>
        </div>
        <div class="filter-right">
            <span class="filter-label">排序：</span>
            <a href="?category={{ current_category }}&level={{ current_level }}&sort=-created_at"
               class="filter-option {% if current_sort == '-created_at' %}active{% endif %}">最新</a>
            <a href="?category={{ current_category }}&level={{ current_level }}&sort=-total_students"
               class="filter-option {% if current_sort == '-total_students' %}active{% endif %}">最热</a>
            <a href="?category={{ current_category }}&level={{ current_level }}&sort=price"
               class="filter-option {% if current_sort == 'price' %}active{% endif %}">价格↑</a>
            <a href="?category={{ current_category }}&level={{ current_level }}&sort=-average_rating"
               class="filter-option {% if current_sort == '-average_rating' %}active{% endif %}">评分</a>
        </div>
    </div>

    <!-- 课程网格 -->
    <div class="course-grid">
        {% for course in courses %}
        <article class="course-card">
            <a href="{% url 'courses:course-detail' course.slug %}">
                <div class="course-cover">
                    <img src="{{ course.cover.url }}" alt="{{ course.title }}">
                    {% if course.discount_percent %}
                        <span class="discount-badge">-{{ course.discount_percent }}%</span>
                    {% endif %}
                </div>
                <div class="course-info">
                    <h3 class="course-title">{{ course.title }}</h3>
                    <p class="course-meta">
                        <span>{{ course.total_students }} 学员</span>
                        <span class="dot">·</span>
                        <span>{{ course.total_chapters }} 章节</span>
                    </p>
                    <div class="course-bottom">
                        <span class="course-price">
                            {% if course.price == 0 %}
                                <span class="free">免费</span>
                            {% else %}
                                ¥{{ course.price }}
                                {% if course.original_price %}
                                    <span class="original-price">¥{{ course.original_price }}</span>
                                {% endif %}
                            {% endif %}
                        </span>
                        <span class="course-rating">
                            {% if course.average_rating > 0 %}
                                ★ {{ course.average_rating }}
                            {% endif %}
                        </span>
                    </div>
                </div>
            </a>
        </article>
        {% empty %}
        <div class="empty-state">
            <p>暂无课程，请稍后再来</p>
        </div>
        {% endfor %}
    </div>

    <!-- 分页 -->
    {% include 'blog/_pagination.html' %}
</div>
{% endblock %}
```

# 四、CSS课程卡片样式
```css
/* 课程页面 */
.course-page { margin: 30px 0; }

/* 筛选栏 */
.filter-bar {
    display: flex; justify-content: space-between; flex-wrap: wrap;
    background: white; padding: 15px 20px; border-radius: 8px;
    margin-bottom: 25px; gap: 10px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.06);
}
.filter-label { font-size: 13px; color: #999; margin-right: 5px; }
.filter-option {
    padding: 3px 12px; border-radius: 15px; font-size: 13px;
    color: #666; transition: all 0.2s;
}
.filter-option.active { background: var(--primary); color: white; }
.filter-option:hover:not(.active) { background: #f0f0f0; }

/* 课程网格 */
.course-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
    gap: 20px;
}
.course-card {
    background: white; border-radius: 8px; overflow: hidden;
    box-shadow: 0 1px 4px rgba(0,0,0,0.06);
    transition: transform 0.3s, box-shadow 0.3s;
}
.course-card:hover { transform: translateY(-4px); box-shadow: 0 8px 25px rgba(0,0,0,0.1); }
.course-cover { position: relative; overflow: hidden; }
.course-cover img { width: 100%; height: 160px; object-fit: cover; }
.discount-badge {
    position: absolute; top: 10px; right: 10px;
    background: #e74c3c; color: white; padding: 3px 8px;
    border-radius: 4px; font-size: 12px;
}
.course-info { padding: 15px; }
.course-title { font-size: 15px; color: #333; margin-bottom: 8px; line-height: 1.5; }
.course-meta { font-size: 12px; color: #999; margin-bottom: 12px; }
.course-bottom { display: flex; justify-content: space-between; align-items: center; }
.course-price { font-size: 18px; color: #e74c3c; font-weight: bold; }
.course-price .free { color: #2ecc71; }
.original-price { font-size: 13px; color: #ccc; text-decoration: line-through; margin-left: 6px; font-weight: normal; }
.course-rating { font-size: 13px; color: #f39c12; }
.dot { margin: 0 5px; }
```

# 五、搜索视图
```python
class CourseSearchView(ListView):
    """课程搜索结果"""
    template_name = 'courses/search_results.html'
    context_object_name = 'courses'
    paginate_by = 12

    def get_queryset(self):
        query = self.request.GET.get('q', '').strip()
        if not query:
            return Course.objects.none()
        return Course.objects.filter(
            (Q(title__icontains=query) | Q(description__icontains=query)) &
            Q(status='published')
        ).select_related('category').order_by('-total_students')

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['query'] = self.request.GET.get('q', '')
        context['result_count'] = self.get_queryset().count()
        return context
```

# 六、核心总结：多条件筛选的设计思路
```
URL参数组合: ?category=python&level=beginner&sort=-total_students&q=django
    ├── category → queryset.filter(category__slug='python')
    ├── level → queryset.filter(level='beginner')
    ├── sort → queryset.order_by('-total_students')
    ├── q → queryset.filter(Q(title__icontains='django') | ...)
    └── 所有参数独立叠加，互不干扰

分页时保留筛选参数：
    ?page=2&category=python&level=beginner&sort=-total_students
    → 第2页仍然是"python分类+入门级别+按热度排序"的结果
```

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
