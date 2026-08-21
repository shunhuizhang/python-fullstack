
# Python全栈入门到实战【Django篇 07】用户认证系统，注册登录与权限控制完整实现
上一篇《Django篇 06》中，我们掌握了Django表单系统——能优雅地处理用户输入并存入数据库了。但还有一个核心问题没有解决：我们如何知道"谁"在操作？谁发的文章？谁提交的评论？每个用户只能修改自己的文章吗？

这就是**用户认证系统**（Authentication）要解决的问题。用户认证是Web应用中最重要的基础设施之一——它负责识别"你是谁"（身份认证）和决定"你能做什么"（权限控制）。如果你的博客没有用户系统，所有人都能随意发表、修改、删除任何文章——这显然是不行的。

Django内置了非常完善的用户认证系统，从用户模型、密码哈希、登录/退出、权限组到装饰器保护视图。你不需要从零实现——Django已经写好了一切。作为全栈开发者，你需要做的是：理解这个系统怎么工作，然后正确地使用它。

本节核心学习内容：
1.  Django内置User模型详解：字段、方法与密码加密原理
2.  用户注册：UserCreationForm与自定义注册视图
3.  用户登录：authenticate() / login() / logout()底层原理
4.  登录保护：login_required装饰器、LoginRequiredMixin
5.  自定义用户模型：AbstractUser扩展与AbstractBaseUser重写
6.  权限控制：用户→权限组→权限的完整体系
7.  综合实战：用户注册/登录/修改密码/个人中心完整实现

# 一、Django内置User模型
## 1.1 不用从零实现用户系统
Django已经为你准备好了一个完整的User模型（位于`django.contrib.auth.models.User`）。打开shell看一眼它有哪些字段：

```python
from django.contrib.auth.models import User

# 查看User模型的字段
user = User.objects.first()
print(user.username)       # 用户名（必填，唯一）
print(user.email)          # 邮箱
print(user.first_name)     # 名
print(user.last_name)      # 姓
print(user.password)       # 加密后的密码（不是明文！）
print(user.is_staff)       # 能否登录Admin后台
print(user.is_superuser)   # 超级管理员（所有权限）
print(user.is_active)      # 是否激活（可用于注销用户而不删除）
print(user.date_joined)    # 注册时间
print(user.last_login)     # 最后登录时间
```

## 1.2 内置User的字段一览
| 字段 | 类型 | 说明 |
|------|------|------|
| `username` | CharField(150) | 用户名，必填且唯一 |
| `password` | CharField(128) | 密码（自动哈希加密存储，不存明文） |
| `email` | EmailField | 邮箱（可选） |
| `first_name` | CharField(150) | 名（可选） |
| `last_name` | CharField(150) | 姓（可选） |
| `is_staff` | BooleanField | 是否可以登录Admin（默认False） |
| `is_superuser` | BooleanField | 是否拥有所有权限（默认False） |
| `is_active` | BooleanField | 账户是否激活（默认True，设为False可禁用账户） |
| `date_joined` | DateTimeField | 账户创建时间 |
| `last_login` | DateTimeField | 最后登录时间 |

## 1.3 密码安全：永远不存明文
这是最重要的安全原则之一：**数据库里永远不存用户的明文密码**。Django的`user.password`存储形式如下：

```
pbkdf2_sha256$600000$xxxxxx$yyyyyyy...
```

这是通过PBKDF2算法（带盐值的密码哈希算法）加密后的结果。即使数据库泄露，攻击者也无法从哈希值反推出原始密码。Django在用户登录时，将用户输入的密码重新哈希，与数据库中的哈希值比对，不会把明文密码写入日志或数据库。

# 二、用户注册
## 2.1 使用Django内置的UserCreationForm
Django提供了现成的注册表单：

```python
# users/forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    """自定义注册表单 —— 在UserCreationForm基础上添加邮箱字段"""
    email = forms.EmailField(
        required=True,
        label='邮箱',
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )

    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']
        # password1: 密码输入
        # password2: 确认密码（自动验证 password1 == password2）

    def clean_email(self):
        """验证邮箱唯一性"""
        email = self.cleaned_data['email']
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError("该邮箱已被注册")
        return email

    def save(self, commit=True):
        """保存新用户"""
        user = super().save(commit=False)
        user.email = self.cleaned_data['email']
        if commit:
            user.save()
        return user
```

## 2.2 注册视图
```python
# users/views.py
from django.shortcuts import render, redirect
from django.contrib.auth import login
from .forms import RegisterForm

def register(request):
    if request.method == 'POST':
        form = RegisterForm(request.POST)
        if form.is_valid():
            user = form.save()  # 创建用户（密码自动加密存储）
            login(request, user)  # 注册后自动登录
            return redirect('home')
    else:
        form = RegisterForm()

    return render(request, 'users/register.html', {'form': form})
```

**关键代码解读**：
- `form.save()`：调用UserCreationForm的save，执行`User.objects.create_user()`，密码自动通过`set_password()`加密
- `login(request, user)`：为新注册的用户创建Session，让他"处于登录状态"

## 2.3 注册模板
```html
<!-- users/templates/users/register.html -->
{% extends 'base.html' %}

{% block content %}
<h2>用户注册</h2>

<form method="POST">
    {% csrf_token %}

    <div>
        <label>{{ form.username.label }}</label>
        {{ form.username }}
        {% if form.username.errors %}
            <span class="error">{{ form.username.errors.0 }}</span>
        {% endif %}
    </div>

    <div>
        <label>{{ form.email.label }}</label>
        {{ form.email }}
        {% if form.email.errors %}
            <span class="error">{{ form.email.errors.0 }}</span>
        {% endif %}
    </div>

    <div>
        <label>{{ form.password1.label }}</label>
        {{ form.password1 }}
        {% if form.password1.errors %}
            <span class="error">{{ form.password1.errors.0 }}</span>
        {% endif %}
        <small>{{ form.password1.help_text }}</small>
    </div>

    <div>
        <label>{{ form.password2.label }}</label>
        {{ form.password2 }}
        {% if form.password2.errors %}
            <span class="error">{{ form.password2.errors.0 }}</span>
        {% endif %}
    </div>

    <button type="submit">注册</button>
</form>

<p>已有账号？<a href="{% url 'login' %}">立即登录</a></p>
{% endblock %}
```

# 三、用户登录与退出
## 3.1 Django认证体系的核心函数
```python
from django.contrib.auth import authenticate, login, logout

# authenticate(username, password)：验证用户名密码是否正确
# 正确 → 返回 User 对象
# 错误 → 返回 None

# login(request, user)：将用户和当前session关联
# 之后 request.user 就是已登录用户

# logout(request)：清除session，用户变为匿名状态
```

## 3.2 登录视图
```python
# users/views.py
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from django.contrib import messages

def user_login(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')

        # 第一步：验证用户名密码
        user = authenticate(request, username=username, password=password)

        if user is not None:
            # 第二步：创建登录session
            login(request, user)

            # 第三步：重定向（next参数支持登录后跳回原页面）
            next_url = request.GET.get('next', 'home')
            return redirect(next_url)
        else:
            messages.error(request, '用户名或密码错误')

    return render(request, 'users/login.html')
```

**authenticate vs login 的区别**：
- `authenticate()`：仅验证用户名密码是否正确（返回User或None）
- `login()`：创建Session，让浏览器"记住"这个用户（写入session表并设置cookie）
- 两者必须配合使用：先authenticate验证，再login建立会话

## 3.3 退出视图
```python
from django.contrib.auth import logout

def user_logout(request):
    logout(request)  # 清除session，用户变为匿名
    return redirect('home')
```

## 3.4 登录模板
```html
<!-- users/templates/users/login.html -->
{% extends 'base.html' %}

{% block content %}
<h2>用户登录</h2>

{% if messages %}
    {% for message in messages %}
        <div class="alert alert-{{ message.tags }}">{{ message }}</div>
    {% endfor %}
{% endif %}

<form method="POST">
    {% csrf_token %}
    <div>
        <label>用户名</label>
        <input type="text" name="username" required>
    </div>
    <div>
        <label>密码</label>
        <input type="password" name="password" required>
    </div>
    <button type="submit">登录</button>
</form>

<p>还没有账号？<a href="{% url 'register' %}">立即注册</a></p>
{% endblock %}
```

## 3.5 模板中使用用户信息
```html
<!-- 在任何模板中都可以直接使用 {{ user }} -->
<!-- request.user 通过 context_processors 自动注入到模板中 -->

{% if user.is_authenticated %}
    <span>欢迎，{{ user.username }}！</span>
    <a href="{% url 'logout' %}">退出</a>

    {% if user.is_superuser %}
        <a href="/admin/">后台管理</a>
    {% endif %}
{% else %}
    <a href="{% url 'login' %}">登录</a>
    <a href="{% url 'register' %}">注册</a>
{% endif %}
```

`{{ user }}`是Django的`auth` context processor自动注入到每个模板的，你不需要在视图中手动传递。当用户未登录时，`user`是一个`AnonymousUser`对象。

# 四、登录保护
## 4.1 login_required装饰器：函数视图
```python
from django.contrib.auth.decorators import login_required

@login_required
def article_create(request):
    # 只有登录用户才能访问
    return render(request, 'blog/article_form.html')

@login_required(login_url='/users/login/')  # 自定义未登录时的跳转地址
def article_edit(request, id):
    # 未登录用户访问会自动重定向到 /users/login/?next=/blog/article/1/edit/
    # 登录后会通过next参数自动跳回
    pass
```

`@login_required`的执行逻辑：检查`request.user.is_authenticated`，如果为False则重定向到登录页面，并在URL中附加`?next=原URL`参数，方便登录后自动跳回。

## 4.2 LoginRequiredMixin：类视图
```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import CreateView

class ArticleCreateView(LoginRequiredMixin, CreateView):
    model = Article
    fields = ['title', 'content']
    login_url = '/users/login/'  # 未登录时的跳转地址
```

## 4.3 在视图中手动检查登录状态
```python
def my_view(request):
    if not request.user.is_authenticated:
        return redirect('login')

    # 或检查特定权限
    if not request.user.has_perm('blog.add_article'):
        return HttpResponseForbidden("你没有发布文章的权限")

    # 处理业务...
```

# 五、自定义用户模型
## 5.1 什么时候需要自定义
Django内置的User模型只提供了基础字段（用户名、密码、邮箱等）。如果你的项目需要额外字段（如手机号、头像、个人简介），有两种扩展方式：

| 方式 | 适用场景 | 复杂度 |
|------|---------|--------|
| **OneToOneFile关联UserProfile** | 只是添加额外字段、项目已在开发中 | 简单 |
| **继承AbstractUser** | 新项目，需要对User字段做扩展 | 中等 |
| **继承AbstractBaseUser** | 完全自定义认证方式（如邮箱登录） | 复杂 |

## 5.2 方式一：OneToOneFile关联（已开发项目推荐）
```python
# users/models.py
from django.db import models
from django.contrib.auth.models import User

class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')
    avatar = models.ImageField(upload_to='avatars/', default='avatars/default.png')
    bio = models.TextField(max_length=500, blank=True, verbose_name='个人简介')
    phone = models.CharField(max_length=11, blank=True, verbose_name='手机号')
    website = models.URLField(blank=True, verbose_name='个人网站')

    def __str__(self):
        return f"{self.user.username}的Profile"

# 使用：user.profile.avatar, user.profile.bio
```

## 5.3 方式二：继承AbstractUser（新项目推荐）
```python
# users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    """扩展内置User"""
    phone = models.CharField(max_length=11, blank=True, verbose_name='手机号')
    avatar = models.ImageField(upload_to='avatars/', default='avatars/default.png')
    nickname = models.CharField(max_length=50, blank=True, verbose_name='昵称')
    bio = models.TextField(max_length=500, blank=True)

    class Meta:
        db_table = 'users'
        verbose_name = '用户'
        verbose_name_plural = '用户'

    def __str__(self):
        return self.username
```

**注意**：继承AbstractUser必须在**项目最开始**就配置好。如果在已有数据之后再改用户模型，迁移会非常复杂。

在`settings.py`中配置：
```python
AUTH_USER_MODEL = 'users.CustomUser'  # app名.模型类名
```

之后所有用到User的地方（ForeignKey、request.user等）都会自动使用你的CustomUser。

# 六、用户中心：修改密码
```python
# users/views.py
from django.contrib.auth import update_session_auth_hash
from django.contrib.auth.forms import PasswordChangeForm
from django.contrib import messages

@login_required
def change_password(request):
    if request.method == 'POST':
        form = PasswordChangeForm(user=request.user, data=request.POST)
        if form.is_valid():
            user = form.save()  # 修改密码
            update_session_auth_hash(request, user)  # 关键：更新session，防止被踢出
            messages.success(request, '密码修改成功！')
            return redirect('profile')
    else:
        form = PasswordChangeForm(user=request.user)

    return render(request, 'users/change_password.html', {'form': form})
```

**`update_session_auth_hash`的重要性**：修改密码后，如果不调用这个函数，Django会认为当前session失效，用户会被踢出登录状态（因为老session关联的是旧密码哈希）。

# 七、综合实战：用户注册→登录→中心完整视图
```python
# users/urls.py
from django.urls import path
from . import views

app_name = 'users'

urlpatterns = [
    path('register/', views.register, name='register'),
    path('login/', views.user_login, name='login'),
    path('logout/', views.user_logout, name='logout'),
    path('profile/', views.profile, name='profile'),
    path('change-password/', views.change_password, name='change-password'),
]
```

```python
# users/views.py 完整版
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login, logout, update_session_auth_hash
from django.contrib.auth.decorators import login_required
from django.contrib.auth.forms import PasswordChangeForm
from django.contrib import messages
from .forms import RegisterForm

def register(request):
    if request.method == 'POST':
        form = RegisterForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            messages.success(request, '注册成功！欢迎加入！')
            return redirect('home')
    else:
        form = RegisterForm()
    return render(request, 'users/register.html', {'form': form})

def user_login(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            messages.success(request, f'欢迎回来，{username}！')
            return redirect(request.GET.get('next', 'home'))
        else:
            messages.error(request, '用户名或密码错误')
    return render(request, 'users/login.html')

def user_logout(request):
    logout(request)
    messages.info(request, '已退出登录')
    return redirect('home')

@login_required
def profile(request):
    return render(request, 'users/profile.html')

@login_required
def change_password(request):
    if request.method == 'POST':
        form = PasswordChangeForm(user=request.user, data=request.POST)
        if form.is_valid():
            user = form.save()
            update_session_auth_hash(request, user)
            messages.success(request, '密码修改成功')
            return redirect('profile')
    else:
        form = PasswordChangeForm(user=request.user)
    return render(request, 'users/change_password.html', {'form': form})
```

# 八、常见误区与避坑指南
1.  **忘记update_session_auth_hash**：修改密码后如果不调用此函数，用户会被立即登出。这个错误非常常见且让用户困惑——"我明明改了密码，怎么被踢出来了？"

2.  **混淆authenticate和login**：`authenticate`只是查询数据库验证密码，不会让用户"登录"。必须调用`login(request, user)`创建session。

3.  **用User.objects.create()而不是create_user()创建用户**：`User.objects.create(password='123456')`会把密码明文存储！始终使用`User.objects.create_user(username, password='xxx')`，它会自动调用`set_password()`进行哈希加密。

4.  **在模板中检查user而不是user.is_authenticated**：`{% if user %}`会返回True（即使user是AnonymousUser在Python中也不是None）。正确写法：`{% if user.is_authenticated %}`。

5.  **login_required忘记设置login_url**：如果不设置，未登录用户会被重定向到`/accounts/login/`。如果你的登录URL不是这个，需要显式指定。

6.  **在settings.py中没修改AUTH_USER_MODEL就迁移**：如果在运行了第一次migrate之后才配置AUTH_USER_MODEL，会导致迁移失败。自定义用户模型必须在新项目首次migrate前配置。

# 九、核心总结：用户认证速查表
## 内置User模型
| 字段 | 用途 |
|------|------|
| `username` | 用户名（必填唯一） |
| `password` | 加密密码 |
| `email` | 邮箱 |
| `is_staff` | Admin权限 |
| `is_superuser` | 超级管理员 |
| `is_active` | 是否激活 |

## 认证函数
| 函数 | 作用 |
|------|------|
| `authenticate(username, password)` | 验证用户名密码 |
| `login(request, user)` | 创建登录session |
| `logout(request)` | 清除session |
| `update_session_auth_hash(request, user)` | 修改密码后保持登录 |

## 登录保护
| 方式 | 适用 |
|------|------|
| `@login_required` | 函数视图 |
| `LoginRequiredMixin` | 类视图 |
| `user.is_authenticated` | 手动判断 |

## 创建用户正确方式
```python
# ✅ 正确：密码自动加密
User.objects.create_user(username='xxx', password='xxx', email='xxx')

# ✅ 正确（已有实例）
user = User(username='xxx')
user.set_password('xxx')
user.save()
```

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
