
# Python全栈入门到实战【Django篇 16】博客实战（五）：用户中心，注册登录、个人主页与账号管理
上一篇《Django篇 15》中，我们实现了评论系统——用户可以评论文章、回复他人的评论。但你可能会注意到一个问题：评论功能要求用户先登录，而我们独立的users应用还只有一个空壳子。

本篇将实现博客系统中与用户相关的所有功能：注册、登录、退出、个人中心（查看自己的文章和评论）、头像上传、修改密码。这些功能完成后，博客的用户体系就完整了——一个完整的"用户入口"：从注册账户开始，到登录、发布文章、发表评论、管理个人资料。

本节核心学习内容：
1.  用户注册：UserCreationForm + 邮箱字段扩展
2.  用户登录/退出：authenticate + login + logout + next参数跳回
3.  个人中心：展示用户文章列表、评论历史
4.  头像上传：Pillow处理图片 + 缩略图生成
5.  修改密码：PasswordChangeForm + update_session_auth_hash
6.  全局用户态集成：导航栏、文章作者显示、评论用户显示

# 一、users应用URL配置
```python
# users/urls.py
from django.urls import path
from . import views

app_name = 'users'

urlpatterns = [
    path('register/', views.RegisterView.as_view(), name='register'),
    path('login/', views.UserLoginView.as_view(), name='login'),
    path('logout/', views.user_logout, name='logout'),
    path('profile/', views.ProfileView.as_view(), name='profile'),
    path('profile/edit/', views.ProfileEditView.as_view(), name='profile-edit'),
    path('change-password/', views.ChangePasswordView.as_view(), name='change-password'),
]
```

# 二、用户注册视图
```python
# users/views.py
from django.shortcuts import render, redirect
from django.views.generic import CreateView, TemplateView, UpdateView
from django.contrib.auth import authenticate, login, logout, update_session_auth_hash
from django.contrib.auth.forms import UserCreationForm, PasswordChangeForm
from django.contrib.auth.mixins import LoginRequiredMixin
from django.contrib import messages
from django.urls import reverse_lazy
from django.contrib.auth.models import User
from .forms import RegisterForm

class RegisterView(CreateView):
    """用户注册"""
    model = User
    form_class = RegisterForm
    template_name = 'users/register.html'
    success_url = reverse_lazy('blog:index')

    def form_valid(self, form):
        """注册成功后自动登录"""
        response = super().form_valid(form)
        user = self.object
        login(self.request, user)
        messages.success(self.request, f'注册成功！欢迎 {user.username}！')
        return response
```

```python
# users/forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    email = forms.EmailField(
        required=True,
        label='邮箱',
        widget=forms.EmailInput(attrs={'class': 'form-control', 'placeholder': '请输入邮箱'})
    )

    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']

    def clean_email(self):
        email = self.cleaned_data['email']
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError('该邮箱已被注册')
        return email

    def save(self, commit=True):
        user = super().save(commit=False)
        user.email = self.cleaned_data['email']
        if commit:
            user.save()
        return user
```

# 三、登录与退出
```python
class UserLoginView(TemplateView):
    """登录视图"""
    template_name = 'users/login.html'

    def post(self, request, *args, **kwargs):
        username = request.POST.get('username', '')
        password = request.POST.get('password', '')
        user = authenticate(request, username=username, password=password)

        if user is not None:
            login(request, user)
            messages.success(request, f'欢迎回来，{username}！')
            # 如果有next参数，登录后跳回原页面
            next_url = request.POST.get('next', None)
            if next_url:
                return redirect(next_url)
            return redirect('blog:index')
        else:
            messages.error(request, '用户名或密码错误，请重试')
            return render(request, self.template_name)

def user_logout(request):
    logout(request)
    messages.info(request, '已退出登录')
    return redirect('blog:index')
```

```html
<!-- templates/users/login.html -->
{% extends 'base.html' %}
{% block title %}登录{% endblock %}
{% block content %}
<div class="auth-container">
    <h2>用户登录</h2>
    <form method="POST">
        {% csrf_token %}
        <input type="hidden" name="next" value="{{ request.GET.next }}">
        <div class="form-group">
            <input type="text" name="username" placeholder="用户名" required class="form-control">
        </div>
        <div class="form-group">
            <input type="password" name="password" placeholder="密码" required class="form-control">
        </div>
        <button type="submit" class="btn-primary btn-block">登录</button>
    </form>
    <p class="auth-link">还没有账号？<a href="{% url 'users:register' %}">立即注册</a></p>
</div>
{% endblock %}
```

# 四、个人中心
```python
from django.views.generic import DetailView, UpdateView
from blog.models import Article, Comment

class ProfileView(LoginRequiredMixin, DetailView):
    """个人中心"""
    template_name = 'users/profile.html'
    context_object_name = 'profile_user'

    def get_object(self):
        return self.request.user

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        user = self.request.user
        context['my_articles'] = Article.objects.filter(author=user).order_by('-created_at')
        context['my_comments'] = Comment.objects.filter(user=user).select_related('article').order_by('-created_at')[:20]
        return context
```

```html
<!-- templates/users/profile.html -->
{% extends 'base.html' %}
{% block title %}个人中心 - {{ user.username }}{% endblock %}
{% block content %}
<div class="profile-page">
    <div class="profile-header">
        <div class="profile-avatar">{{ user.username.0 }}</div>
        <div class="profile-info">
            <h2>{{ user.username }}</h2>
            <p>注册时间：{{ user.date_joined|date:"Y-m-d" }}</p>
            <div class="profile-actions">
                <a href="{% url 'users:profile-edit' %}" class="btn-secondary">编辑资料</a>
                <a href="{% url 'users:change-password' %}" class="btn-secondary">修改密码</a>
            </div>
        </div>
    </div>

    <!-- 我的文章 -->
    <section class="profile-section">
        <h3>我的文章（{{ my_articles|length }}）</h3>
        {% if my_articles %}
        <ul class="article-list-simple">
            {% for article in my_articles %}
            <li>
                <span class="status-badge status-{{ article.status }}">{{ article.get_status_display }}</span>
                <a href="{% url 'blog:article-detail' article.slug %}">{{ article.title }}</a>
                <span class="date">{{ article.created_at|date:"Y-m-d" }}</span>
            </li>
            {% endfor %}
        </ul>
        {% else %}
        <p class="empty-hint">还没有发布文章，<a href="{% url 'blog:article-create' %}">写一篇</a>吧</p>
        {% endif %}
    </section>

    <!-- 我的评论 -->
    <section class="profile-section">
        <h3>我的评论（{{ my_comments|length }}）</h3>
        {% if my_comments %}
        <ul class="comment-list-simple">
            {% for comment in my_comments %}
            <li>
                <p>"{{ comment.content }}"</p>
                <span class="meta">
                    评论了 <a href="{% url 'blog:article-detail' comment.article.slug %}">{{ comment.article.title }}</a>
                    {{ comment.created_at|date:"Y-m-d H:i" }}
                </span>
            </li>
            {% endfor %}
        </ul>
        {% endif %}
    </section>
</div>
{% endblock %}
```

# 五、修改密码
```python
class ChangePasswordView(LoginRequiredMixin, TemplateView):
    template_name = 'users/change_password.html'

    def post(self, request):
        form = PasswordChangeForm(user=request.user, data=request.POST)
        if form.is_valid():
            user = form.save()
            update_session_auth_hash(request, user)  # 保持登录
            messages.success(request, '密码修改成功')
            return redirect('users:profile')
        else:
            for field, errors in form.errors.items():
                for error in errors:
                    messages.error(request, error)
            return render(request, self.template_name, {'form': form})

    def get_context_data(self, **kwargs):
        kwargs['form'] = PasswordChangeForm(user=self.request.user)
        return super().get_context_data(**kwargs)
```

# 六、注册模板
```html
<!-- templates/users/register.html -->
{% extends 'base.html' %}
{% block title %}注册{% endblock %}
{% block content %}
<div class="auth-container">
    <h2>用户注册</h2>
    <form method="POST">
        {% csrf_token %}
        <div class="form-group">
            {{ form.username }}
            {% if form.username.errors %}<span class="error">{{ form.username.errors.0 }}</span>{% endif %}
        </div>
        <div class="form-group">
            <input type="email" name="email" placeholder="邮箱" required class="form-control">
        </div>
        <div class="form-group">
            {{ form.password1 }}
            <small class="help-text">{{ form.password1.help_text }}</small>
        </div>
        <div class="form-group">
            {{ form.password2 }}
            {% if form.password2.errors %}<span class="error">{{ form.password2.errors.0 }}</span>{% endif %}
        </div>
        <button type="submit" class="btn-primary btn-block">注册</button>
    </form>
    <p class="auth-link">已有账号？<a href="{% url 'users:login' %}">立即登录</a></p>
</div>
{% endblock %}
```

# 七、核心总结：用户认证流程
```
注册: UserCreationForm → create_user() → set_password() → login()
登录: authenticate(username, password) → login(request, user)
退出: logout(request)
密码: PasswordChangeForm → update_session_auth_hash()
```

## Bootstrap样式补充
```css
/* 表单控件样式 */
.form-control {
    width: 100%; padding: 10px 12px; border: 1px solid #ddd;
    border-radius: 5px; font-size: 14px; outline: none;
}
.form-control:focus { border-color: var(--primary); }
.btn-block { width: 100%; }
.auth-container { max-width: 400px; margin: 40px auto; }

/* 个人中心 */
.profile-page { background: white; border-radius: 8px; padding: 30px; }
.profile-header { display: flex; gap: 20px; align-items: center; margin-bottom: 30px; padding-bottom: 20px; border-bottom: 1px solid #eee; }
.profile-avatar { width: 80px; height: 80px; border-radius: 50%; background: var(--primary); color: white; display: flex; justify-content: center; align-items: center; font-size: 32px; }
.profile-actions { display: flex; gap: 10px; margin-top: 10px; }
.btn-secondary { padding: 6px 14px; border: 1px solid #ddd; border-radius: 5px; font-size: 13px; color: #666; }
.profile-section { margin: 25px 0; }
.profile-section h3 { font-size: 16px; margin-bottom: 15px; padding-bottom: 8px; border-bottom: 1px solid #eee; }
.article-list-simple, .comment-list-simple { list-style: none; }
.article-list-simple li, .comment-list-simple li { padding: 8px 0; font-size: 14px; }
.status-badge { font-size: 11px; padding: 1px 6px; border-radius: 3px; margin-right: 8px; }
.status-published { background: #d4edda; color: #155724; }
.status-draft { background: #fff3cd; color: #856404; }
```

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
