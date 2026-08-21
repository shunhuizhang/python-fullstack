
# Python全栈入门到实战【Django篇 06】表单、ModelForm与文件上传，从用户输入到数据入库
上一篇《Django篇 05》中，我们掌握了ORM的增删改查——能在Python代码中自如地操作数据库了。但最终，数据的来源应该是**用户**——用户通过网页上的表单输入内容并提交，后端接收数据后存入数据库。这个"用户输入→表单验证→数据入库"的完整流程，就是Django表单系统要解决的问题。

表单是Web应用中连接用户和数据库的"桥梁"。它看似简单（几个input框+一个提交按钮），但实际涉及：HTML渲染、输入验证（邮箱格式对不对？密码够不够安全？）、错误提示、XSS防护、CSRF保护、数据清洗后入库。如果全部用原生HTML+JS手写，每一个环节都需要你亲自处理——极易遗漏产生安全漏洞。

Django的表单系统把这些事情都"包圆"了。你只需定义一个Python类（Form或ModelForm），Django自动帮你生成HTML表单、验证用户输入、返回友好的错误信息、将清洗后的数据安全地存入数据库。作为全栈开发者，掌握Django表单系统，你就拥有了高效且安全地处理用户数据的能力。

本节核心学习内容：
1.  Form vs ModelForm：何时用哪个
2.  手动Form：定义字段、验证、cleaned_data读取
3.  ModelForm：模型自动生成表单、save()入库
4.  自定义验证：clean_field()单字段验证与clean()全局验证
5.  表单在视图中的完整处理流程（GET展示 + POST处理）
6.  文件上传：FileField/ImageField、MEDIA配置、enctype="multipart/form-data"
7.  多文件上传与前端预览
8.  CSRF保护原理与{% csrf_token %}

# 一、Form vs ModelForm
## 1.1 Django表单的两种类型
| 类型 | 定义方式 | 适合场景 | 与数据库的关系 |
|------|---------|---------|--------------|
| **Form** | 手写字段，独立于模型 | 搜索、联系、登录等不直接对应一张表的表单 | 无关 |
| **ModelForm** | 自动从Model生成字段 | 文章创建/编辑、用户注册等直接对应表中一行数据的表单 | 直接映射 |

**选择原则**：如果表单的内容正好对应某个模型的字段 → 用`ModelForm`（代码量少一半）。如果表单内容不对应任何模型（如搜索框、登录表单、联系表单） → 用`Form`。

## 1.2 Python类比
Django Form ≈ Python数据类的定义方式。Form定义字段类型和验证规则，就像你用dataclass定义数据结构一样。区别是Django Form还会自动生成HTML、验证输入、返回错误信息——一站式解决。

# 二、Form：手写表单
## 2.1 定义Form类
在应用下创建`forms.py`：

```python
# blog/forms.py
from django import forms

class ContactForm(forms.Form):
    """联系表单 —— 不对应任何模型"""
    name = forms.CharField(
        max_length=50,
        label='姓名',
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': '请输入您的姓名'
        })
    )
    email = forms.EmailField(
        label='邮箱',
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'example@email.com'
        })
    )
    subject = forms.CharField(
        max_length=100,
        label='主题',
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    message = forms.CharField(
        label='留言内容',
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'rows': 5,
            'placeholder': '请输入您的留言...'
        })
    )

class SearchForm(forms.Form):
    """搜索表单"""
    keyword = forms.CharField(
        max_length=100,
        label='',
        widget=forms.TextInput(attrs={
            'class': 'search-input',
            'placeholder': '搜索文章...'
        })
    )
```

## 2.2 常用Form字段
| Form字段 | 对应HTML | 验证规则 |
|---------|---------|---------|
| `CharField` | `<input type="text">` | 字符串，需要指定max_length |
| `EmailField` | `<input type="email">` | 邮箱格式验证 |
| `IntegerField` | `<input type="number">` | 整数验证 |
| `FloatField` | `<input type="number">` | 浮点数验证 |
| `DecimalField` | `<input type="number">` | 精确小数 |
| `BooleanField` | `<input type="checkbox">` | True/False |
| `ChoiceField` | `<select>` | 从预设选项中选择 |
| `MultipleChoiceField` | `<select multiple>` | 多选 |
| `DateField` | `<input type="date">` | 日期格式验证 |
| `DateTimeField` | `<input type="datetime-local">` | 日期时间验证 |
| `URLField` | `<input type="url">` | URL格式验证 |
| `FileField` | `<input type="file">` | 文件上传 |
| `ImageField` | `<input type="file" accept="image/*">` | 图片上传 |

## 2.3 在视图中使用Form（完整GET+POST流程）
```python
# blog/views.py
from django.shortcuts import render, redirect
from .forms import ContactForm

def contact(request):
    if request.method == 'POST':
        # POST请求：用户提交了表单
        form = ContactForm(request.POST)  # 将提交的数据传入Form
        if form.is_valid():
            # 验证通过：数据在 form.cleaned_data 中
            # 清理过的数据是一个字典，所有值都通过了验证
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            message = form.cleaned_data['message']

            # 这里可以发送邮件、保存到数据库等
            print(f"收到留言：{name}({email}) - {message}")

            # 重定向到成功页面（防止用户刷新重新提交）
            return redirect('contact-success')
        else:
            # 验证失败：form.errors 包含错误信息
            pass
    else:
        # GET请求：展示空表单
        form = ContactForm()

    return render(request, 'blog/contact.html', {'form': form})
```

## 2.4 在模板中渲染Form
```html
<!-- blog/templates/blog/contact.html -->
{% extends 'blog/base.html' %}

{% block content %}
<h1>联系我们</h1>

<form method="POST">
    {% csrf_token %}

    <!-- 方式1：快速渲染所有字段（自动生成） -->
    <!-- {{ form.as_p }} -->
    <!-- {{ form.as_table }} -->
    <!-- {{ form.as_div }} -->

    <!-- 方式2：逐个字段手动控制布局（推荐） -->
    <div class="form-group">
        <label>{{ form.name.label }}</label>
        {{ form.name }}
        {% if form.name.errors %}
            <span class="error">{{ form.name.errors.0 }}</span>
        {% endif %}
    </div>

    <div class="form-group">
        <label>{{ form.email.label }}</label>
        {{ form.email }}
        {% if form.email.errors %}
            <span class="error">{{ form.email.errors.0 }}</span>
        {% endif %}
    </div>

    <div class="form-group">
        <label>{{ form.subject.label }}</label>
        {{ form.subject }}
    </div>

    <div class="form-group">
        <label>{{ form.message.label }}</label>
        {{ form.message }}
        {% if form.message.errors %}
            <span class="error">{{ form.message.errors.0 }}</span>
        {% endif %}
    </div>

    <button type="submit">提交留言</button>
</form>
{% endblock %}
```

# 三、ModelForm：模型驱动的表单
## 3.1 定义ModelForm
当表单字段与模型字段一一对应时，`ModelForm`可以自动生成表单，代码量显著减少：

```python
# blog/forms.py
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article                          # 关联的模型
        fields = ['title', 'content', 'author', 'is_published']  # 需要显示的字段
        # fields = '__all__'                     # 所有字段（不推荐，容易暴露不该暴露的字段）
        # exclude = ['views', 'updated_at']      # 排除不需要的字段

        # 自定义字段的widget
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': '请输入文章标题'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 15,
                'placeholder': '请输入文章内容（支持Markdown）'
            }),
            'is_published': forms.CheckboxInput(attrs={
                'class': 'form-check-input'
            }),
        }

        # 自定义字段的标签
        labels = {
            'title': '文章标题',
            'content': '文章内容',
            'is_published': '立即发布',
            'author': '作者',
        }

        # 自定义帮助文本
        help_texts = {
            'title': '标题将在文章列表和详情页中显示',
        }

    # 你可以覆盖模型的某个字段（不想用模型默认的类型）
    # author = forms.ModelChoiceField(
    #     queryset=User.objects.filter(is_staff=True),
    #     label='作者'
    # )
```

## 3.2 在视图中使用ModelForm
```python
# blog/views.py
from django.shortcuts import render, redirect, get_object_or_404
from .forms import ArticleForm
from .models import Article

def article_create(request):
    """创建文章"""
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save()  # ModelForm.save() 返回创建的对象
            return redirect('article-detail', id=article.id)
    else:
        form = ArticleForm()

    return render(request, 'blog/article_form.html', {'form': form, 'action': '创建'})

def article_edit(request, id):
    """编辑文章"""
    article = get_object_or_404(Article, id=id)

    if request.method == 'POST':
        form = ArticleForm(request.POST, instance=article)  # ← 传入已有实例（编辑模式）
        if form.is_valid():
            form.save()
            return redirect('article-detail', id=article.id)
    else:
        form = ArticleForm(instance=article)  # ← GET也传入实例（展示已有数据）

    return render(request, 'blog/article_form.html', {'form': form, 'action': '编辑'})
```

**创建 vs 编辑的关键区别**：
```python
# 创建：不带instance，save()执行INSERT
form = ArticleForm(request.POST)
form.save()

# 编辑：带instance，save()执行UPDATE
form = ArticleForm(request.POST, instance=existing_article)
form.save()
```

# 四、自定义验证
## 4.1 clean_字段名()：验证单个字段
命名规则：`clean_` + 小写字段名。返回值是清洗后的值：

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']

    def clean_title(self):
        """验证标题"""
        title = self.cleaned_data['title']  # 获取字段的原始值

        # 规则1：不能太短
        if len(title) < 5:
            raise forms.ValidationError("文章标题不能少于5个字符")

        # 规则2：不能重复
        if Article.objects.filter(title=title).exists():
            raise forms.ValidationError("该标题已存在，请换一个")

        # 规则3：不能包含敏感词
        sensitive_words = ['广告', '推广', '赚钱']
        for word in sensitive_words:
            if word in title:
                raise forms.ValidationError(f"标题中不能包含敏感词：{word}")

        return title  # 必须返回清洗后的值
```

## 4.2 clean()：全局验证（跨字段验证）
当验证需要同时检查多个字段时（如"密码确认"），在`clean()`中处理：

```python
class RegisterForm(forms.Form):
    password = forms.CharField(widget=forms.PasswordInput)
    confirm_password = forms.CharField(widget=forms.PasswordInput)
    email = forms.EmailField()

    def clean(self):
        """跨字段验证"""
        cleaned_data = super().clean()  # 先调用父类的clean获取清洗后的数据

        password = cleaned_data.get('password')
        confirm_password = cleaned_data.get('confirm_password')

        if password and confirm_password:
            if password != confirm_password:
                raise forms.ValidationError("两次输入的密码不一致")

        return cleaned_data  # 必须返回cleaned_data
```

## 4.3 内置验证器
Django提供了现成的验证器，直接在字段中使用：

```python
from django.core.validators import MinLengthValidator, MaxLengthValidator, EmailValidator, RegexValidator

class UserForm(forms.Form):
    username = forms.CharField(
        validators=[
            MinLengthValidator(3, message='用户名至少3个字符'),
            MaxLengthValidator(20, message='用户名最多20个字符'),
            RegexValidator(r'^[a-zA-Z0-9_]+$', message='用户名只能包含字母、数字和下划线'),
        ]
    )
```

# 五、文件上传
## 5.1 配置文件上传
```python
# settings.py
import os

# 上传文件的根路径
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# 上传文件对应的URL前缀
MEDIA_URL = '/media/'
```

```python
# 项目 urls.py —— 开发环境下配置MEDIA URL
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... 你的路由
]

# 仅在DEBUG=True时生效（生产环境由Nginx处理）
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## 5.2 模型中使用ImageField/FileField
```python
# blog/models.py
class Article(models.Model):
    title = models.CharField(max_length=200)
    # ImageField：自动验证上传的是图片
    cover = models.ImageField(
        upload_to='covers/%Y/%m/',   # 文件按年/月分目录存储
        default='covers/default.jpg', # 默认封面
        verbose_name='封面图片'
    )
    # FileField：可以上传任意文件
    attachment = models.FileField(
        upload_to='attachments/%Y/%m/',
        blank=True,
        null=True,
        verbose_name='附件'
    )
```

## 5.3 视图处理文件上传
```python
# blog/views.py
def article_create(request):
    if request.method == 'POST':
        # 文件上传必须同时传 request.POST 和 request.FILES
        form = ArticleForm(request.POST, request.FILES)  # ← 关键！
        if form.is_valid():
            form.save()
            return redirect('index')
    else:
        form = ArticleForm()

    return render(request, 'blog/article_form.html', {'form': form})
```

## 5.4 HTML表单设置
```html
<!-- 文件上传必须在form标签中设置 enctype="multipart/form-data" -->
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">提交</button>
</form>
```

## 5.5 多文件上传
```python
# forms.py
from django import forms

class MultipleFileInput(forms.ClearableFileInput):
    allow_multiple_selected = True  # HTML5多选属性

class MultipleFileField(forms.FileField):
    def __init__(self, *args, **kwargs):
        kwargs.setdefault("widget", MultipleFileInput())
        super().__init__(*args, **kwargs)

    def clean(self, data, initial=None):
        single_file_clean = super().clean
        if isinstance(data, (list, tuple)):
            return [single_file_clean(d, initial) for d in data]
        return [single_file_clean(data, initial)]

class GalleryForm(forms.Form):
    images = MultipleFileField(label='上传图片（可多选）')
```

```python
# views.py 中处理多文件
def upload_gallery(request):
    if request.method == 'POST':
        form = GalleryForm(request.POST, request.FILES)
        if form.is_valid():
            files = request.FILES.getlist('images')  # 获取所有上传的文件
            for file in files:
                # 处理每个文件...
                handle_uploaded_file(file)
            return redirect('gallery')
    else:
        form = GalleryForm()
    return render(request, 'blog/gallery_upload.html', {'form': form})
```

# 六、CSRF保护
## 6.1 CSRF是什么
CSRF（跨站请求伪造）是一种攻击：攻击者伪造一个表单，诱导你的用户在不知情的情况下提交数据。Django对此提供了内置保护——每个POST表单必须携带CSRF token。

## 6.2 在模板中使用
```html
<form method="POST">
    {% csrf_token %}  <!-- Django自动生成隐藏的token字段 -->
    <!-- 如果不加这一行，提交POST时Django返回403 Forbidden -->
</form>
```

## 6.3 在AJAX请求中使用
当用JS的Fetch发送POST请求时，也需要CSRF token：

```javascript
// 从Cookie中获取CSRF token
function getCSRFToken() {
    const name = 'csrftoken=';
    const cookies = document.cookie.split(';');
    for (let cookie of cookies) {
        cookie = cookie.trim();
        if (cookie.startsWith(name)) {
            return cookie.substring(name.length);
        }
    }
    return '';
}

// Fetch POST时携带CSRF token
fetch('/blog/api/submit/', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': getCSRFToken(),
    },
    body: JSON.stringify({ key: 'value' })
});
```

# 七、常见误区与避坑指南
1.  **文件上传忘记enctype**：如果表单中包含`<input type="file">`，必须设置`enctype="multipart/form-data"`，否则`request.FILES`是空的。

2.  **忘记同时传request.POST和request.FILES**：`ArticleForm(request.POST)`不会包含文件数据。正确写法是`ArticleForm(request.POST, request.FILES)`。

3.  **ModelForm中fields和exclude的选择**：不要用`fields='__all__'`，这会暴露所有字段（包括你可能不想让用户直接编辑的）。显式声明`fields = ['title', 'content']`更安全。

4.  **自定义验证方法忘记返回值**：`clean_字段名()`必须返回清洗后的值，`clean()`必须返回cleaned_data。如果忘记返回，字段的值会变成None。

5.  **is_valid()失败后直接返回200**：表单验证失败时应该重新渲染表单页面（让用户看到错误提示），不要重定向到成功页。

6.  **生产环境忘记配置MEDIA文件服务**：开发环境中Django自动代理MEDIA文件，但生产环境需要Nginx配置`/media/`路径映射到`MEDIA_ROOT`。

7.  **上传文件大小限制**：Django默认不对上传文件大小做限制，但Web服务器（Nginx）和浏览器都可能有限制。可以在`settings.py`中设置`FILE_UPLOAD_MAX_MEMORY_SIZE`和`DATA_UPLOAD_MAX_MEMORY_SIZE`。

# 八、核心总结：表单速查表
## Form vs ModelForm
```python
# Form：独立于模型
class ContactForm(forms.Form):
    name = forms.CharField(max_length=50)

# ModelForm：绑定到模型
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']
```

## 视图中使用表单（标准流程）
```python
def view(request):
    if request.method == 'POST':
        form = XXXForm(request.POST, request.FILES)  # 文件上传必须加FILES
        if form.is_valid():
            form.save()  # ModelForm专用
            return redirect('success')  # PRG模式
    else:
        form = XXXForm()
    return render(request, 'template.html', {'form': form})
```

## 自定义验证
```python
def clean_字段名(self):          # 单字段验证
    data = self.cleaned_data['字段名']
    if 不满足条件:
        raise ValidationError("错误信息")
    return data

def clean(self):                 # 全局验证（跨字段）
    data = super().clean()
    if 跨字段不满足条件:
        raise ValidationError("错误信息")
    return data
```

## 文件上传配置
```python
# settings.py
MEDIA_ROOT = BASE_DIR / 'media'
MEDIA_URL = '/media/'

# urls.py (开发环境)
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
