
# Python全栈入门到实战【Django篇 18】教育平台实战（一）：需求分析与数据模型，从博客到在线教育平台的跨越
上一篇《Django篇 17》中，我们完成了博客系统的所有功能并进行了优化总结。博客是Django在"内容管理"场景的典型应用，接下来我们将挑战一个更复杂的商业场景——**在线教育平台**。

在线教育平台和博客有本质区别：博客是"内容→读者"的单向发布，教育平台是"课程→学员"的商业交易——涉及课程发布、视频学习、选课收藏、订单支付、学习进度追踪、数据统计看板。模型关系更复杂、业务逻辑更多样、用户角色也分"学员"和"教师"。

本篇作为教育平台的起点，将完成需求分析和模型设计——把复杂的业务需求转化为清晰的数据库表结构和关联关系。模型设计做对了，后续的开发就是水到渠成的事情。

本节核心学习内容：
1.  教育平台需求分析：功能清单、用户角色、业务流程
2.  核心模型设计：课程/章节/视频/分类/教师/订单/收藏/学习记录
3.  复杂关联关系：教师↔课程（多对多含中间表字段）、订单状态流转
4.  Admin后台配置：批量配置、内联管理
5.  测试数据填充与数据库初始化

# 一、需求分析
## 1.1 功能清单
| 模块 | 功能 | 谁用 |
|------|------|------|
| 课程浏览 | 课程列表、分类筛选、搜索、课程详情（含章节大纲） | 所有人 |
| 视频学习 | 章节视频播放、进度记录、断点续学 | 学员 |
| 用户系统 | 注册/登录、头像、积分、学习记录 | 所有人 |
| 选课与收藏 | 加入购物车、课程收藏、已购课程 | 学员 |
| 订单支付 | 生成订单、支付接口对接、订单状态追踪 | 学员 |
| 教师后台 | 课程管理、收入统计、学员数据 | 教师 |
| 数据看板 | 今日收入、学员数、课程排名、分析图表 | 教师+管理员 |

## 1.2 用户角色
| 角色 | 权限 |
|------|------|
| 学员 | 浏览课程、购买课程、学习视频、发表评价、收藏 |
| 教师 | 发布/管理自己的课程、查看学员数据、收入统计 |
| 管理员 | 全平台管理（通过Admin后台） |

# 二、核心数据模型设计
## 2.1 课程分类模型
```python
# courses/models.py
from django.db import models
from django.urls import reverse

class Category(models.Model):
    """课程分类"""
    name = models.CharField(max_length=50, verbose_name='分类名称')
    slug = models.SlugField(max_length=50, unique=True, verbose_name='URL别名')
    description = models.TextField(blank=True, verbose_name='分类描述')
    order = models.IntegerField(default=0, verbose_name='排序（数字越大越靠前）')

    class Meta:
        verbose_name = '课程分类'
        verbose_name_plural = '课程分类'
        ordering = ['-order', 'name']

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        return reverse('courses:category-detail', kwargs={'slug': self.slug})
```

## 2.2 课程模型（核心）
```python
from django.contrib.auth.models import User

class Course(models.Model):
    """课程"""
    # 基本信息
    title = models.CharField(max_length=200, verbose_name='课程标题')
    slug = models.SlugField(max_length=200, unique=True, verbose_name='URL别名')
    description = models.TextField(verbose_name='课程简介')
    detail = models.TextField(verbose_name='课程详情（支持HTML）')

    # 分类
    category = models.ForeignKey(
        Category, on_delete=models.PROTECT,
        related_name='courses', verbose_name='所属分类'
    )

    # 教师（多对多，一个课程可以有多个教师）
    teachers = models.ManyToManyField(
        User, through='CourseTeacher', related_name='teaching_courses',
        verbose_name='授课教师'
    )

    # 价格
    price = models.DecimalField(max_digits=8, decimal_places=2, verbose_name='价格')
    original_price = models.DecimalField(
        max_digits=8, decimal_places=2, blank=True, null=True,
        verbose_name='原价（划线价）'
    )

    # 难度等级
    LEVEL_CHOICES = [
        ('beginner', '入门'),
        ('intermediate', '中级'),
        ('advanced', '高级'),
    ]
    level = models.CharField(max_length=20, choices=LEVEL_CHOICES,
                             default='beginner', verbose_name='难度')

    # 封面图
    cover = models.ImageField(upload_to='courses/%Y/%m/',
                              verbose_name='课程封面')

    # 状态
    STATUS_CHOICES = [
        ('draft', '草稿'),
        ('published', '已发布'),
        ('offline', '已下架'),
    ]
    status = models.CharField(max_length=20, choices=STATUS_CHOICES,
                              default='draft', verbose_name='状态')

    # 统计字段
    total_students = models.PositiveIntegerField(default=0, verbose_name='学员总数')
    total_chapters = models.PositiveIntegerField(default=0, verbose_name='章节数')
    total_duration = models.PositiveIntegerField(default=0, verbose_name='总时长（分钟）')
    average_rating = models.DecimalField(
        max_digits=3, decimal_places=1, default=0.0, verbose_name='平均评分'
    )
    review_count = models.PositiveIntegerField(default=0, verbose_name='评价数量')

    # 时间
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    updated_at = models.DateTimeField(auto_now=True, verbose_name='更新时间')

    class Meta:
        verbose_name = '课程'
        verbose_name_plural = '课程列表'
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['status', '-created_at']),
            models.Index(fields=['category', 'status']),
        ]

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('courses:course-detail', kwargs={'slug': self.slug})

    @property
    def discount_percent(self):
        """折扣百分比"""
        if self.original_price and self.original_price > 0:
            return int((1 - self.price / self.original_price) * 100)
        return 0
```

## 2.3 课程-教师中间表
```python
class CourseTeacher(models.Model):
    """课程与教师的关系（支持角色描述）"""
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    role = models.CharField(max_length=50, default='讲师',
                           verbose_name='角色（如：主讲、助教）')

    class Meta:
        unique_together = ['course', 'user']
        verbose_name = '课程教师'
        verbose_name_plural = '课程教师'

    def __str__(self):
        return f'{self.user.username} - {self.course.title} ({self.role})'
```

## 2.4 章节与视频模型
```python
class Chapter(models.Model):
    """课程章节"""
    course = models.ForeignKey(Course, on_delete=models.CASCADE,
                               related_name='chapters', verbose_name='所属课程')
    title = models.CharField(max_length=200, verbose_name='章节标题')
    order = models.PositiveIntegerField(default=0, verbose_name='排序')

    class Meta:
        verbose_name = '章节'
        verbose_name_plural = '章节列表'
        ordering = ['order']

    def __str__(self):
        return f'{self.course.title} - {self.title}'

class Video(models.Model):
    """章节下的视频"""
    chapter = models.ForeignKey(Chapter, on_delete=models.CASCADE,
                                related_name='videos', verbose_name='所属章节')
    title = models.CharField(max_length=200, verbose_name='视频标题')
    url = models.URLField(verbose_name='视频地址（或嵌入代码）')
    duration = models.PositiveIntegerField(default=0, verbose_name='时长（分钟）')
    order = models.PositiveIntegerField(default=0, verbose_name='排序')
    is_free = models.BooleanField(default=False, verbose_name='是否免费试看')

    class Meta:
        verbose_name = '视频'
        verbose_name_plural = '视频列表'
        ordering = ['order']

    def __str__(self):
        return f'{self.chapter.title} - {self.title}'
```

## 2.5 订单模型
```python
import uuid
from datetime import datetime

class Order(models.Model):
    """课程订单"""
    order_no = models.CharField(max_length=32, unique=True, verbose_name='订单号')
    user = models.ForeignKey(User, on_delete=models.CASCADE,
                             related_name='orders', verbose_name='下单用户')
    course = models.ForeignKey(Course, on_delete=models.PROTECT,
                               verbose_name='购买的课程')
    amount = models.DecimalField(max_digits=8, decimal_places=2, verbose_name='实付金额')

    STATUS_CHOICES = [
        ('pending', '待支付'),
        ('paid', '已支付'),
        ('refunded', '已退款'),
        ('cancelled', '已取消'),
    ]
    status = models.CharField(max_length=20, choices=STATUS_CHOICES,
                              default='pending', verbose_name='订单状态')

    paid_at = models.DateTimeField(null=True, blank=True, verbose_name='支付时间')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')

    class Meta:
        verbose_name = '订单'
        verbose_name_plural = '订单列表'
        ordering = ['-created_at']

    def save(self, *args, **kwargs):
        """保存时自动生成订单号"""
        if not self.order_no:
            self.order_no = datetime.now().strftime('%Y%m%d') + uuid.uuid4().hex[:8].upper()
        super().save(*args, **kwargs)

    def __str__(self):
        return f'订单 {self.order_no}'
```

## 2.6 收藏与学习记录
```python
class CourseFavorite(models.Model):
    """课程收藏"""
    user = models.ForeignKey(User, on_delete=models.CASCADE,
                             related_name='favorites')
    course = models.ForeignKey(Course, on_delete=models.CASCADE,
                               related_name='favorites')
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ['user', 'course']  # 同一用户不能重复收藏同一课程

class LearningProgress(models.Model):
    """学习进度记录"""
    user = models.ForeignKey(User, on_delete=models.CASCADE,
                             related_name='learning_progress')
    video = models.ForeignKey(Video, on_delete=models.CASCADE)
    progress = models.FloatField(default=0, verbose_name='观看进度（百分比）')
    last_position = models.FloatField(default=0, verbose_name='上次观看位置（秒）')
    is_completed = models.BooleanField(default=False, verbose_name='是否完成')
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        unique_together = ['user', 'video']  # 每个用户每个视频一条进度记录
```

## 2.7 课程评价
```python
class CourseReview(models.Model):
    """课程评价"""
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='reviews')
    course = models.ForeignKey(Course, on_delete=models.CASCADE, related_name='reviews')
    rating = models.PositiveSmallIntegerField(verbose_name='评分（1-5星）')
    content = models.TextField(blank=True, verbose_name='评价内容')
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ['user', 'course']

    def __str__(self):
        return f'{self.user.username} - {self.rating}星'
```

# 三、Admin后台配置
```python
# courses/admin.py
from django.contrib import admin
from .models import Category, Course, Chapter, Video, Order, CourseTeacher

class ChapterInline(admin.TabularInline):
    model = Chapter
    extra = 1  # 默认显示1个空行
    fields = ['title', 'order']

class VideoInline(admin.TabularInline):
    model = Video
    extra = 1

class CourseTeacherInline(admin.TabularInline):
    model = CourseTeacher
    extra = 1

@admin.register(Course)
class CourseAdmin(admin.ModelAdmin):
    list_display = ['title', 'category', 'price', 'status', 'total_students', 'created_at']
    list_filter = ['status', 'category', 'level']
    search_fields = ['title', 'description']
    prepopulated_fields = {'slug': ('title',)}
    inlines = [CourseTeacherInline, ChapterInline]  # 在课程编辑页直接管理教师和章节

@admin.register(Chapter)
class ChapterAdmin(admin.ModelAdmin):
    inlines = [VideoInline]

@admin.register(Video)
class VideoAdmin(admin.ModelAdmin):
    list_display = ['title', 'chapter', 'duration', 'is_free']

@admin.register(Order)
class OrderAdmin(admin.ModelAdmin):
    list_display = ['order_no', 'user', 'course', 'amount', 'status', 'created_at']
    list_filter = ['status', 'created_at']
    readonly_fields = ['order_no']

admin.site.register(Category)
admin.site.register(CourseReview)
```

# 四、创建项目与应用
```bash
cd blog_project  # 沿用博客项目的虚拟环境和settings.py
python manage.py startapp courses

# 在 settings.py 中注册
INSTALLED_APPS = [
    # ... 已有应用
    'courses',
]
```

# 五、模型关系全景图
```
Category ─── FK ───→ Course
                      ├── M2M(through CourseTeacher) ───→ User (教师)
                      ├── FK ───→ Chapter
                      │              └── FK ───→ Video
                      ├── FK ───→ Order (用户购买)
                      ├── FK ───→ CourseFavorite (收藏)
                      ├── FK ───→ CourseReview (评价)
                      └── M2M ───→ LearningProgress (→Video)
                                    (via 间接关联，实际通过User+Video)
```

# 六、核心总结
教育平台相比博客的核心升级：
| 维度 | 博客 | 教育平台 |
|------|------|---------|
| 内容形态 | 文章（文本） | 课程→章节→视频（结构化内容） |
| 交易 | 无 | 订单+支付 |
| 关联复杂度 | 简单外键 | 中间表+through定义 |
| 用户角色 | 作者/读者 | 学员/教师/管理员 |
| 数据统计 | 阅读量 | 学员数/收入/进度/评分 |

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
