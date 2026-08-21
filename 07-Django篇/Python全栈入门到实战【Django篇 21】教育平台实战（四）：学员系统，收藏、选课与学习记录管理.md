
# Python全栈入门到实战【Django篇 21】教育平台实战（四）：学员系统，收藏、选课与学习记录管理
上一篇《Django篇 20》中，我们实现了课程详情页和视频播放——用户可以浏览课程信息、查看章节大纲、观看视频并自动记录学习进度。但"学习"是一个持续的过程：用户需要一个地方能看到"我收藏了哪些课程"、"我购买了哪些课程"、"我学到哪里了"。

这就是学员系统的核心价值——给每个用户一个专属的**学习空间**，集中管理选课、收藏和学习记录。这和博客的个人中心不同：博客的用户中心侧重"我发布了什么"，教育平台侧重"我学到了什么"。

本篇将实现学员系统的完整功能：课程收藏切换、我的课程（已购+收藏）、学习进度汇总、学习时长统计。

本节核心学习内容：
1.  课程收藏：Ajax异步切换收藏状态
2.  我的课程：已购课程列表 + 收藏课程列表
3.  学习进度：每个课程的学习完成百分比
4.  学习记录：最近观看的视频、断点续学
5.  学习时长统计：总学习时长 + 今日学习时长
6.  学员中心模板设计

# 一、收藏与选课视图
```python
# courses/views.py
from django.shortcuts import get_object_or_404, redirect
from django.http import JsonResponse
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from .models import Course, CourseFavorite, Order

@login_required
def toggle_favorite(request, course_slug):
    """切换课程收藏状态（Ajax + 普通跳转兼容）"""
    course = get_object_or_404(Course, slug=course_slug, status='published')
    favorite, created = CourseFavorite.objects.get_or_create(
        user=request.user, course=course
    )
    if not created:
        favorite.delete()
        favorited = False
    else:
        favorited = True

    # 如果请求是Ajax，返回JSON
    if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
        return JsonResponse({'status': 'ok', 'favorited': favorited})

    # 否则重定向回课程详情页
    messages.success(request, '已添加到收藏' if favorited else '已取消收藏')
    return redirect('courses:course-detail', slug=course_slug)

@login_required
def add_to_cart(request, course_slug):
    """添加课程到购物车（简化版：直接创建待支付订单）"""
    course = get_object_or_404(Course, slug=course_slug, status='published')

    # 检查是否已购买
    if Order.objects.filter(user=request.user, course=course, status='paid').exists():
        messages.info(request, '你已经购买过该课程了')
        return redirect('courses:course-detail', slug=course_slug)

    # 检查是否已有待支付订单
    existing = Order.objects.filter(user=request.user, course=course, status='pending').first()
    if existing:
        return redirect('courses:order-detail', order_no=existing.order_no)

    # 创建新订单
    order = Order.objects.create(
        user=request.user,
        course=course,
        amount=course.price
    )
    return redirect('courses:order-detail', order_no=order.order_no)
```

```python
# courses/urls.py 追加
urlpatterns = [
    path('<slug:slug>/favorite/', views.toggle_favorite, name='toggle-favorite'),
    path('<slug:slug>/add-to-cart/', views.add_to_cart, name='add-to-cart'),
    path('my-courses/', views.MyCoursesView.as_view(), name='my-courses'),
]
```

# 二、我的课程视图
```python
from django.views.generic import TemplateView
from django.db.models import Count, Sum, Q

class MyCoursesView(LoginRequiredMixin, TemplateView):
    """学员中心：我的课程、收藏、学习记录"""
    template_name = 'courses/my_courses.html'

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        user = self.request.user

        # 1. 已购课程
        purchased_course_ids = Order.objects.filter(
            user=user, status='paid'
        ).values_list('course_id', flat=True)
        context['purchased_courses'] = Course.objects.filter(
            id__in=purchased_course_ids, status='published'
        ).select_related('category')

        # 2. 每个课程的学习进度（附加到课程对象上，避免模板中的get_item问题）
        from .models import LearningProgress, Video
        for course in context['purchased_courses']:
            total_videos = Video.objects.filter(chapter__course=course).count()
            completed_videos = LearningProgress.objects.filter(
                user=user, video__chapter__course=course, is_completed=True
            ).count()
            if total_videos > 0:
                course.progress_info = {
                    'percent': int(completed_videos / total_videos * 100),
                    'completed': completed_videos,
                    'total': total_videos,
                }
            else:
                course.progress_info = None

        # 3. 收藏的课程
        context['favorite_courses'] = Course.objects.filter(
            favorites__user=user, status='published'
        ).select_related('category')

        # 4. 最近学习记录
        context['recent_progress'] = LearningProgress.objects.filter(
            user=user
        ).select_related('video__chapter__course').order_by('-updated_at')[:10]

        # 5. 学习统计
        total_seconds = sum(
            p.last_position for p in LearningProgress.objects.filter(user=user)
        )
        context['total_hours'] = round(total_seconds / 3600, 1)
        today_progress = LearningProgress.objects.filter(
            user=user, updated_at__date=timezone.now().date()
        )
        today_seconds = sum(p.last_position for p in today_progress)
        context['today_minutes'] = round(today_seconds / 60)

        return context
```

# 三、学员中心模板
```html
<!-- templates/courses/my_courses.html -->
{% extends 'base.html' %}
{% block title %}我的学习 - 在线教育{% endblock %}

{% block content %}
<div class="my-courses-page">
    <h1>我的学习</h1>

    <!-- 学习统计卡片 -->
    <div class="stats-row">
        <div class="stat-card">
            <span class="stat-value">{{ purchased_courses|length }}</span>
            <span class="stat-label">已购课程</span>
        </div>
        <div class="stat-card">
            <span class="stat-value">{{ total_hours }}</span>
            <span class="stat-label">累计学习/小时</span>
        </div>
        <div class="stat-card">
            <span class="stat-value">{{ today_minutes }}</span>
            <span class="stat-label">今日学习/分钟</span>
        </div>
    </div>

    <!-- 已购课程 -->
    <section class="section">
        <h2>已购课程</h2>
        {% if purchased_courses %}
        <div class="purchased-list">
            {% for course in purchased_courses %}
            <div class="purchased-item">
                <img src="{{ course.cover.url }}" alt="{{ course.title }}" class="purchased-cover">
                <div class="purchased-info">
                    <h3><a href="{% url 'courses:course-detail' course.slug %}">{{ course.title }}</a></h3>
                    {% if course.progress_info %}
                    <div class="progress-bar-wrapper">
                        <div class="progress-bar" style="width:{{ course.progress_info.percent }}%"></div>
                    </div>
                    <span class="progress-text">已学 {{ course.progress_info.percent }}%（{{ course.progress_info.completed }}/{{ course.progress_info.total }}）</span>
                    {% endif %}
                    <a href="{% url 'courses:course-detail' course.slug %}#chapter-list" class="btn-continue">继续学习</a>
                </div>
            </div>
            {% endfor %}
        </div>
        {% else %}
        <p class="empty-hint">还没有购买课程，<a href="{% url 'courses:course-list' %}">去选课</a></p>
        {% endif %}
    </section>

    <!-- 收藏课程 -->
    <section class="section">
        <h2>收藏的课程</h2>
        {% if favorite_courses %}
        <div class="course-grid">
            {% for course in favorite_courses %}
                {% include 'courses/_course_card.html' with course=course %}
            {% endfor %}
        </div>
        {% else %}
        <p class="empty-hint">还没有收藏课程</p>
        {% endif %}
    </section>

    <!-- 最近学习记录 -->
    {% if recent_progress %}
    <section class="section">
        <h2>最近学习</h2>
        <div class="recent-list">
            {% for record in recent_progress %}
            <a href="{% url 'courses:video-play' record.video.id %}" class="recent-item">
                <span>▶ {{ record.video.title }}</span>
                <span class="recent-meta">
                    {{ record.progress|floatformat:0 }}% · {{ record.updated_at|date:"Y-m-d H:i" }}
                </span>
            </a>
            {% endfor %}
        </div>
    </section>
    {% endif %}
</div>
{% endblock %}
```

# 四、CSS学员中心样式
```css
.stats-row { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 15px; margin-bottom: 30px; }
.stat-card { background: white; padding: 20px; border-radius: 8px; text-align: center; box-shadow: 0 1px 4px rgba(0,0,0,0.06); }
.stat-value { display: block; font-size: 28px; font-weight: bold; color: var(--primary); }
.stat-label { font-size: 13px; color: #999; }
.section { margin-bottom: 35px; }
.section h2 { font-size: 18px; margin-bottom: 15px; padding-bottom: 8px; border-bottom: 2px solid var(--primary); }
.purchased-item { display: flex; gap: 15px; background: white; padding: 15px; border-radius: 8px; margin-bottom: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
.purchased-cover { width: 120px; height: 75px; object-fit: cover; border-radius: 5px; }
.purchased-info h3 { font-size: 15px; margin-bottom: 8px; }
.progress-bar-wrapper { height: 6px; background: #eee; border-radius: 3px; margin: 8px 0; }
.progress-bar { height: 100%; background: var(--primary); border-radius: 3px; transition: width 0.3s; }
.progress-text { font-size: 12px; color: #999; }
.btn-continue { display: inline-block; margin-top: 8px; padding: 5px 15px; background: var(--primary); color: white; border-radius: 4px; font-size: 13px; }
.recent-item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid #f0f0f0; font-size: 14px; }
.recent-meta { font-size: 12px; color: #999; }
```

# 五、课程收藏Ajax前端
```javascript
// 收藏按钮（在课程详情页中，用class="btn-favorite"标记）
document.addEventListener('click', function(e) {
    const btn = e.target.closest('.btn-favorite');
    if (!btn) return;
    e.preventDefault();

    fetch(btn.href, {
        headers: { 'X-Requested-With': 'XMLHttpRequest' }
    })
    .then(res => res.json())
    .then(data => {
        btn.innerHTML = data.favorited ? '★ 已收藏' : '☆ 收藏';
        btn.classList.toggle('favorited', data.favorited);
    });
});
```

# 六、核心总结
## 学员系统数据流向
```
用户登录 → 学员中心 (/my-courses/)
    ├── 已购课程 ← Order {user, course, status='paid'}
    │   └── 每课进度 ← LearningProgress {user, video__chapter__course}
    ├── 收藏课程 ← CourseFavorite {user, course}
    ├── 最近记录 ← LearningProgress {user}.order_by('-updated_at')
    └── 统计汇总 ← SUM(LearningProgress.last_position)
```

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
