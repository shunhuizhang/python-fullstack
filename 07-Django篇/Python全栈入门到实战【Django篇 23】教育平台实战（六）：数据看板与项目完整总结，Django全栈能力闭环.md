
# Python全栈入门到实战【Django篇 23】教育平台实战（六）：数据看板与项目完整总结，Django全栈能力闭环
上一篇《Django篇 22》中，我们实现了课程交易系统——从购物车到订单创建到支付宝支付回调。至此，教育平台的业务功能全部完成。但对于教师和管理员来说，还缺了一个重要的"后台视角"：**数据看板**——我要知道我今天收入多少、有多少学员在学习、哪门课程最受欢迎、最近的销售趋势如何。

数据看板不是给普通用户看的，它是给**运营者和决策者**的工具。一个好的看板能让人一眼看清"发生了什么"——无需翻数据库，无需写SQL，打开页面就一目了然。同时，看板也是对技术栈的综合运用：ORM聚合查询、Django模板渲染、甚至可以利用JS篇学过的技巧做前端图表可视化。

本篇作为教育平台的收官之作，将实现教师端数据看板和教育平台完整项目总结。

本节核心学习内容：
1.  教师端数据看板：今日收入、学员数、课程销量排名
2.  图表可视化：Django数据 + ECharts前端柱状图/饼图
3.  管理员全局Dashboard
4.  教育平台完整项目总结
5.  两个Django实战项目：博客 vs 教育平台对比

# 一、教师端数据看板视图
```python
# courses/views.py
from django.contrib.auth.mixins import LoginRequiredMixin
from django.utils import timezone
from datetime import timedelta
from django.db import models
from django.db.models import Sum, Count

class TeacherDashboardView(LoginRequiredMixin, TemplateView):
    """教师端数据看板"""
    template_name = 'courses/teacher_dashboard.html'

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        user = self.request.user
        today = timezone.now().date()

        # 只显示该教师自己的课程
        my_courses = Course.objects.filter(teachers=user, status='published')

        # 1. 概览统计
        context['total_courses'] = my_courses.count()
        context['total_students'] = my_courses.aggregate(total=Sum('total_students'))['total'] or 0

        # 2. 今日收入
        today_start = timezone.make_aware(timezone.datetime.combine(today, timezone.datetime.min.time()))
        today_end = today_start + timedelta(days=1)
        context['today_income'] = Order.objects.filter(
            course__in=my_courses, status='paid',
            paid_at__gte=today_start, paid_at__lt=today_end
        ).aggregate(total=Sum('amount'))['total'] or 0

        # 3. 本月收入
        month_start = today.replace(day=1)
        month_start_dt = timezone.make_aware(timezone.datetime.combine(month_start, timezone.datetime.min.time()))
        context['month_income'] = Order.objects.filter(
            course__in=my_courses, status='paid',
            paid_at__gte=month_start_dt
        ).aggregate(total=Sum('amount'))['total'] or 0

        # 4. 课程销量排名
        context['course_sales_rank'] = my_courses.annotate(
            sales_count=Count('order', filter=models.Q(order__status='paid'))
        ).order_by('-sales_count')[:10]

        # 5. 近7天每日收入（给ECharts）
        context['last_7_days'] = self.get_last_7_days_data(my_courses)

        # 6. 课程难度分布（给ECharts饼图）
        context['level_distribution'] = list(my_courses.values('level').annotate(
            count=Count('id')
        ))

        return context

    def get_last_7_days_data(self, courses):
        """近7天每日收入"""
        data = []
        today = timezone.now().date()
        for i in range(6, -1, -1):
            day = today - timedelta(days=i)
            day_start = timezone.make_aware(timezone.datetime.combine(day, timezone.datetime.min.time()))
            day_end = day_start + timedelta(days=1)
            income = Order.objects.filter(
                course__in=courses, status='paid',
                paid_at__gte=day_start, paid_at__lt=day_end
            ).aggregate(total=Sum('amount'))['total'] or 0
            data.append({
                'date': day.strftime('%m-%d'),
                'income': float(income)
            })
        return data
```

```python
# courses/urls.py 追加
urlpatterns = [
    path('teacher/dashboard/', views.TeacherDashboardView.as_view(), name='teacher-dashboard'),
]
```

# 二、看板模板（集成ECharts）
```html
<!-- templates/courses/teacher_dashboard.html -->
{% extends 'base.html' %}
{% block title %}教师数据看板{% endblock %}

{% block content %}
<div class="dashboard-page">
    <h1>教师数据看板</h1>

    <!-- 概览卡片 -->
    <div class="stats-grid">
        <div class="stat-card">
            <span class="stat-value">{{ total_courses }}</span>
            <span class="stat-label">发布课程</span>
        </div>
        <div class="stat-card">
            <span class="stat-value">{{ total_students }}</span>
            <span class="stat-label">总学员数</span>
        </div>
        <div class="stat-card">
            <span class="stat-value">¥{{ today_income }}</span>
            <span class="stat-label">今日收入</span>
        </div>
        <div class="stat-card">
            <span class="stat-value">¥{{ month_income }}</span>
            <span class="stat-label">本月收入</span>
        </div>
    </div>

    <!-- 近7天收入趋势图 -->
    <div class="chart-card">
        <h3>近7天收入趋势</h3>
        <div id="income-chart" style="height:300px;"></div>
    </div>

    <!-- 课程难度分布饼图 -->
    <div class="charts-row">
        <div class="chart-card half">
            <h3>课程难度分布</h3>
            <div id="level-chart" style="height:280px;"></div>
        </div>
        <!-- 课程销量排名 -->
        <div class="chart-card half">
            <h3>课程销量排名</h3>
            <table class="sales-table">
                <thead><tr><th>#</th><th>课程</th><th>销量</th></tr></thead>
                <tbody>
                    {% for course in course_sales_rank %}
                    <tr>
                        <td>{{ forloop.counter }}</td>
                        <td>{{ course.title|truncatechars:15 }}</td>
                        <td>{{ course.sales_count }}</td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
    </div>
</div>
{% endblock %}

{% block extra_js %}
<script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
<script>
    // 近7天收入柱状图
    const incomeChart = echarts.init(document.getElementById('income-chart'));
    incomeChart.setOption({
        tooltip: { trigger: 'axis' },
        xAxis: {
            type: 'category',
            data: [{% for d in last_7_days %}'{{ d.date }}',{% endfor %}]
        },
        yAxis: { type: 'value', name: '收入（元）' },
        series: [{
            name: '收入',
            type: 'bar',
            data: [{% for d in last_7_days %}{{ d.income }},{% endfor %}],
            itemStyle: { color: '#3498db' }
        }]
    });

    // 课程难度分布饼图
    const levelChart = echarts.init(document.getElementById('level-chart'));
    levelChart.setOption({
        tooltip: { trigger: 'item' },
        series: [{
            type: 'pie',
            radius: ['40%', '70%'],
            data: [
                {% for item in level_distribution %}
                { value: {{ item.count }}, name: '{{ item.get_level_display }}' },
                {% endfor %}
            ]
        }]
    });

    // 响应式
    window.addEventListener('resize', () => {
        incomeChart.resize();
        levelChart.resize();
    });
</script>
{% endblock %}
```

# 三、教育平台完整项目总结
## 3.1 目录结构
```
blog_project/                  # 同一项目目录（复用基础配置）
├── courses/                   # 教育平台应用
│   ├── models.py              # 12个模型类
│   ├── views.py               # 全部视图
│   ├── urls.py                # 路由
│   ├── admin.py               # Admin配置
│   ├── cart.py                # 购物车工具类
│   ├── alipay.py              # 支付宝工具类
│   └── context_processors.py
├── blog/                      # 博客应用（共存）
├── users/                     # 用户应用（共用）
└── templates/
    └── courses/
        ├── course_list.html
        ├── course_detail.html
        ├── video_play.html
        ├── my_courses.html
        ├── cart.html
        ├── order_detail.html
        └── teacher_dashboard.html
```

## 3.2 已实现功能清单
| 模块 | 功能 | 核心技术 |
|------|------|---------|
| 课程浏览 | 列表+多条件筛选+分页 | ListView + Q对象 + filter叠加 |
| 课程详情 | 章节大纲、教师介绍 | DetailView + prefetch_related |
| 视频播放 | 播放+进度上报+断点续学 | JS timeupdate事件 + Fetch POST |
| 学员系统 | 已购/收藏/学习记录/统计 | 多表关联查询+aggregate |
| 购物车 | Session存储 | 自定义Cart工具类 |
| 订单系统 | 创建→支付→回调→开通 | 支付宝沙箱+异步通知验签 |
| 数据看板 | 今日收入/学员/销量/ECharts图 | aggregate + annotate + ECharts |

## 3.3 博客 vs 教育平台对比
| 维度 | 博客系统 | 教育平台 |
|------|---------|---------|
| 核心模型 | 4个（Article/Category/Tag/Comment） | 12个（Course/Chapter/Video/Order...） |
| 关联复杂度 | 简单外键+多对多 | 多对多含中间表+自引用外键+状态流转 |
| 用户角色 | 作者/读者 | 学员/教师/管理员 |
| 交易系统 | 无 | 购物车+订单+支付回调 |
| 内容形态 | Markdown文本 | 结构化视频列表 |
| 数据统计 | 阅读量 | 收入/销量/进度/时长 |

# 四、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
