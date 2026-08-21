
# Python全栈入门到实战【Django篇 20】教育平台实战（三）：课程详情与视频学习，构建在线学习核心体验
上一篇《Django篇 19》中，我们实现了课程列表与多条件筛选。用户现在可以浏览课程、按分类和难度筛选、按价格和评分排序。但点击课程卡片后，还需要一个课程详情页——展示课程的完整信息（简介、大纲、教师）、提供选课/收藏入口，以及最核心的功能：**视频播放和学习进度记录**。

课程详情页是教育平台的"心脏"——它的质量直接决定了用户是否下单购买。一个好的课程详情页需要：课程封面和大标题吸引眼球、课程大纲清晰展示章节和视频结构、教师介绍增加信任感、价格和优惠信息驱动购买决策、同时支持免费视频试看以降低用户决策门槛。

本篇将实现课程详情页的完整功能，包括章节视频展示、视频播放页面、学习进度记录与追踪。

本节核心学习内容：
1.  课程详情页：教师信息、章节大纲、选课/收藏状态
2.  视频播放页：video标签播放 + 学习进度JS上报
3.  学习进度记录：Fetch API异步保存进度到数据库
4.  课程评价：星级评分 + 评价列表
5.  相关课程推荐

# 一、课程详情视图
```python
# courses/views.py
from django.views.generic import DetailView
from django.shortcuts import get_object_or_404
from django.core.exceptions import PermissionDenied
from .models import Course, CourseReview, CourseFavorite, Order

class CourseDetailView(DetailView):
    """课程详情页"""
    model = Course
    template_name = 'courses/course_detail.html'
    context_object_name = 'course'

    def get_queryset(self):
        return Course.objects.filter(
            status='published'
        ).select_related('category').prefetch_related(
            'teachers', 'chapters__videos'
        )

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        course = self.object
        user = self.request.user

        # 用户是否已购买该课程
        context['is_purchased'] = False
        context['is_favorited'] = False
        if user.is_authenticated:
            context['is_purchased'] = Order.objects.filter(
                user=user, course=course, status='paid'
            ).exists()
            context['is_favorited'] = CourseFavorite.objects.filter(
                user=user, course=course
            ).exists()

        # 课程评价
        context['reviews'] = CourseReview.objects.filter(
            course=course
        ).select_related('user').order_by('-created_at')[:10]

        # 该课程教师的其他课程
        teacher_ids = course.teachers.values_list('id', flat=True)
        context['teacher_other_courses'] = Course.objects.filter(
            teachers__in=teacher_ids, status='published'
        ).exclude(pk=course.pk).distinct()[:4]

        # 相关推荐（同分类的其他课程）
        context['related_courses'] = Course.objects.filter(
            category=course.category, status='published'
        ).exclude(pk=course.pk).order_by('-total_students')[:4]

        # 用户学习进度（已购课程）
        if context['is_purchased']:
            from .models import LearningProgress
            context['learning_progress'] = LearningProgress.objects.filter(
                user=user, video__chapter__course=course
            ).select_related('video__chapter')

        return context
```

# 二、课程详情模板
```html
<!-- templates/courses/course_detail.html -->
{% extends 'base.html' %}
{% block title %}{{ course.title }} - 在线教育{% endblock %}

{% block content %}
<div class="course-detail-page">
    <!-- 课程头部信息 -->
    <div class="course-hero">
        <img src="{{ course.cover.url }}" alt="{{ course.title }}" class="hero-cover">
        <div class="hero-info">
            <span class="category-tag">{{ course.category.name }}</span>
            <h1>{{ course.title }}</h1>
            <p class="course-brief">{{ course.description }}</p>
            <div class="hero-meta">
                <span>难度：{{ course.get_level_display }}</span>
                <span>·</span>
                <span>{{ course.total_students }} 名学员</span>
                <span>·</span>
                <span>{{ course.total_duration }} 分钟</span>
                <span>·</span>
                <span class="rating-stars">
                    {% if course.average_rating > 0 %}
                        {% for i in "12345" %}
                            {% if forloop.counter <= course.average_rating|floatformat:0 %}★{% else %}☆{% endif %}
                        {% endfor %}
                        {{ course.average_rating }}
                    {% else %}
                        暂无评价
                    {% endif %}
                </span>
            </div>
            <div class="hero-price">
                <span class="price-current">
                    {% if course.price == 0 %}免费{% else %}¥{{ course.price }}{% endif %}
                </span>
                {% if course.original_price and course.original_price > course.price %}
                    <span class="price-original">¥{{ course.original_price }}</span>
                    <span class="price-discount">-{{ course.discount_percent }}%</span>
                {% endif %}
            </div>
            <div class="hero-actions">
                {% if user.is_authenticated %}
                    {% if is_purchased %}
                        <a href="#chapter-list" class="btn-primary">继续学习</a>
                    {% else %}
                        <a href="{% url 'courses:add-to-cart' course.slug %}" class="btn-primary">立即购买</a>
                    {% endif %}
                    <a href="{% url 'courses:toggle-favorite' course.slug %}" class="btn-favorite">
                        {% if is_favorited %}★ 已收藏{% else %}☆ 收藏{% endif %}
                    </a>
                {% else %}
                    <a href="{% url 'users:login' %}?next={{ request.path }}" class="btn-primary">登录后购买</a>
                {% endif %}
            </div>
        </div>
    </div>

    <!-- 课程详情与大纲 -->
    <div class="course-body">
        <div class="course-main">
            <!-- 课程详情 -->
            <section>
                <h2>课程详情</h2>
                <div class="course-description">{{ course.detail|safe }}</div>
            </section>

            <!-- 课程大纲 -->
            <section id="chapter-list">
                <h2>课程大纲（{{ course.total_chapters }}章 · {{ course.total_duration }}分钟）</h2>
                <div class="chapter-list">
                    {% for chapter in course.chapters.all %}
                    <div class="chapter-item">
                        <div class="chapter-header">
                            <span class="chapter-title">第{{ forloop.counter }}章：{{ chapter.title }}</span>
                            <span class="chapter-video-count">{{ chapter.videos.all|length }}个视频</span>
                        </div>
                        <div class="video-list">
                            {% for video in chapter.videos.all %}
                            <div class="video-item">
                                <span class="video-icon">▶</span>
                                {% if is_purchased or video.is_free %}
                                    <a href="{% url 'courses:video-play' video.id %}">
                                        {{ video.title }}
                                        {% if video.is_free %}<span class="free-tag">免费</span>{% endif %}
                                    </a>
                                {% else %}
                                    <span class="locked">{{ video.title }} <span class="lock-icon">🔒</span></span>
                                {% endif %}
                                <span class="video-duration">{{ video.duration }}分钟</span>
                            </div>
                            {% endfor %}
                        </div>
                    </div>
                    {% endfor %}
                </div>
            </section>

            <!-- 教师介绍 -->
            <section>
                <h2>授课教师</h2>
                <div class="teacher-list">
                    {% for teacher_rel in course.courseteacher_set.all %}
                    <div class="teacher-card">
                        <span class="teacher-name">{{ teacher_rel.user.username }}</span>
                        <span class="teacher-role">{{ teacher_rel.role }}</span>
                    </div>
                    {% endfor %}
                </div>
            </section>
        </div>
    </div>
</div>
{% endblock %}
```

# 三、视频播放页面
```python
# courses/views.py
class VideoPlayView(DetailView):
    """视频播放页面"""
    model = Video
    template_name = 'courses/video_play.html'
    context_object_name = 'video'
    pk_url_kwarg = 'video_id'

    def get_queryset(self):
        return Video.objects.select_related('chapter__course').all()

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        video = self.object
        course = video.chapter.course
        user = self.request.user

        # 权限检查：已购或免费视频
        if not video.is_free:
            if not user.is_authenticated:
                raise PermissionDenied("请登录后观看")
            has_access = Order.objects.filter(user=user, course=course, status='paid').exists()
            if not has_access:
                raise PermissionDenied("请购买课程后再观看")

        # 所有章节（侧边栏导航）
        context['chapters'] = course.chapters.prefetch_related('videos').all()

        # 学习进度（如果已登录）
        if user.is_authenticated:
            from .models import LearningProgress
            progress, _ = LearningProgress.objects.get_or_create(
                user=user, video=video,
                defaults={'progress': 0, 'last_position': 0}
            )
            context['progress'] = progress
            context['last_position'] = progress.last_position

        # 上一节和下一节
        all_videos = list(Video.objects.filter(
            chapter__course=course
        ).order_by('chapter__order', 'order').values_list('id', flat=True))
        current_index = all_videos.index(video.id)
        context['prev_video'] = Video.objects.get(id=all_videos[current_index - 1]) if current_index > 0 else None
        context['next_video'] = Video.objects.get(id=all_videos[current_index + 1]) if current_index < len(all_videos) - 1 else None

        return context
```

```python
# courses/urls.py 追加
urlpatterns = [
    path('video/<int:video_id>/', views.VideoPlayView.as_view(), name='video-play'),
    path('api/video/<int:video_id>/progress/', views.save_progress, name='save-progress'),
]
```

# 四、学习进度保存API
```python
# courses/views.py
from django.http import JsonResponse
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_POST

@login_required
@require_POST
def save_progress(request, video_id):
    """保存学习进度（Ajax接口）"""
    import json
    data = json.loads(request.body)
    progress_percent = data.get('progress', 0)
    current_time = data.get('current_time', 0)

    video = get_object_or_404(Video, id=video_id)
    progress, _ = LearningProgress.objects.update_or_create(
        user=request.user,
        video=video,
        defaults={
            'progress': progress_percent,
            'last_position': current_time,
            'is_completed': progress_percent >= 95,  # >=95% 算完成
        }
    )
    return JsonResponse({'status': 'ok'})
```

# 五、视频播放模板
```html
<!-- templates/courses/video_play.html -->
{% extends 'base.html' %}
{% block title %}{{ video.title }} - 在线学习{% endblock %}

{% block content %}
<div class="video-page">
    <div class="video-main">
        <h2>{{ video.title }}</h2>
        <div class="video-player-wrapper">
            <video id="course-video" controls width="100%" data-video-id="{{ video.id }}">
                <source src="{{ video.url }}" type="video/mp4">
            </video>
        </div>
        {% if next_video %}
            <a href="{% url 'courses:video-play' next_video.id %}" class="btn-next">下一节：{{ next_video.title }} →</a>
        {% endif %}
    </div>

    <!-- 侧边栏课程大纲 -->
    <aside class="video-sidebar">
        <h3>{{ video.chapter.course.title }}</h3>
        {% for chapter in chapters %}
        <div class="sidebar-chapter">
            <h4>{{ chapter.title }}</h4>
            {% for v in chapter.videos.all %}
            <a href="{% url 'courses:video-play' v.id %}"
               class="sidebar-video {% if v.id == video.id %}active{% endif %}">
                <span>{{ v.title }}</span>
                {% if v.is_free %}<span class="free-tag">免费</span>{% endif %}
            </a>
            {% endfor %}
        </div>
        {% endfor %}
    </aside>
</div>
{% endblock %}

{% block extra_js %}
<script>
    const video = document.getElementById('course-video');
    const videoId = video.dataset.videoId;

    // 恢复上次播放位置
    {% if last_position %}
    video.currentTime = {{ last_position }};
    {% endif %}

    // 每5秒上报一次进度
    let lastReportTime = 0;
    video.addEventListener('timeupdate', function() {
        const now = Date.now();
        if (now - lastReportTime < 5000) return; // 防止频繁请求
        lastReportTime = now;

        const progress = (video.currentTime / video.duration * 100).toFixed(1);
        saveProgress(progress, video.currentTime);
    });

    // 视频结束时标记完成
    video.addEventListener('ended', function() {
        saveProgress(100, video.duration);
    });

    function saveProgress(progress, currentTime) {
        fetch(`/courses/api/video/${videoId}/progress/`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-CSRFToken': document.querySelector('[name=csrfmiddlewaretoken]').value,
            },
            body: JSON.stringify({ progress, current_time: currentTime })
        });
    }
</script>
{% endblock %}
```

# 六、CSS视频播放样式
```css
.video-page { display: grid; grid-template-columns: 1fr 280px; gap: 25px; margin: 25px 0; }
.video-main h2 { margin-bottom: 15px; }
.video-player-wrapper { background: #000; border-radius: 8px; overflow: hidden; }
.btn-next { display: inline-block; margin-top: 15px; padding: 10px 20px; background: var(--primary); color: white; border-radius: 5px; }
.video-sidebar { max-height: calc(100vh - 100px); overflow-y: auto; }
.video-sidebar h3 { font-size: 15px; margin-bottom: 15px; }
.sidebar-chapter h4 { font-size: 13px; color: #666; margin: 10px 0 5px; }
.sidebar-video { display: block; padding: 6px 8px; font-size: 13px; color: #555; border-radius: 3px; }
.sidebar-video.active { background: #e8f4fe; color: var(--primary); font-weight: 600; }
.sidebar-video:hover { background: #f0f0f0; }
.free-tag { font-size: 10px; color: #2ecc71; border: 1px solid #2ecc71; padding: 0 4px; border-radius: 2px; margin-left: 5px; }
.locked { color: #ccc; }
.lock-icon { font-size: 12px; }
```

# 七、核心总结
## 课程详情页的数据流
```
URL: /courses/<slug>/ → DetailView.get_object() → Course + select_related/prefetch_related
    → get_context_data → is_purchased / is_favorited / reviews / progress
    → 模板渲染 → 根据已购/未购显示不同按钮 → 根据is_free显示/锁定视频
```

## 视频播放与进度
```
视频播放 → JS监听timeupdate事件 → 每5秒 Fetch上报进度
    → Django POST /api/video/<id>/progress/
    → update_or_create LearningProgress记录
    → 下次观看时恢复 last_position
```

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
