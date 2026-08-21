
# Python全栈入门到实战【Django篇 10】DRF入门：序列化器与API视图，构建前后端分离接口
前面的Django学习中，我们一直在用模板渲染HTML页面——视图查数据，传给模板，Django帮你拼接成HTML返回给浏览器。这种模式叫"**服务端渲染**"，用户看到的每一个页面都是Django在服务器端生成好的完整HTML。

但在现代Web开发中，还有一种更主流的模式叫"**前后端分离**"——后端只负责提供JSON格式的数据（API接口），前端（可能是Vue/React，也可能是我们JS篇写的原生JS项目）通过Fetch/AJAX获取数据，自己渲染页面。这种模式下，后端和前端是两个独立的项目，通过API进行通信。

Django本身是服务端渲染框架，但通过**Django REST Framework（DRF）**这个扩展库，它也能成为功能强大的API框架。DRF是目前Python生态中最流行的REST API框架，让Django具备了"服务端渲染"和"API服务"双重能力。作为全栈开发者，掌握DRF意味着你能为任何前端（Vue/React/小程序/JS）提供后端接口支持。

本节核心学习内容：
1.  为什么需要DRF：服务端渲染 vs 前后端分离的对比
2.  RESTful API设计规范：资源、HTTP方法、状态码
3.  安装与配置DRF
4.  Serializer序列化器：模型对象↔JSON的双向转换
5.  APIView与@api_view：编写API接口
6.  请求解析与响应格式化
7.  实战：博客文章API的CRUD完整实现

# 一、为什么需要DRF
## 1.1 服务端渲染 vs 前后端分离
| 特性 | 服务端渲染（Django原生） | 前后端分离（DRF） |
|------|------------------------|------------------|
| 后端返回 | 完整的HTML页面 | JSON数据 |
| 前端技术 | Django模板 | 任意前端框架（Vue/React/原生JS） |
| 用户体验 | 每次点击整页刷新 | 局部刷新（SPA单页应用） |
| 移动端兼容 | 需要单独开发 | 一套API适配Web+App+小程序 |
| 开发分工 | 全栈一人完成 | 后端API + 前端工程师可并行开发 |

## 1.2 Django原生也能返回JSON，为什么还要DRF
你当然可以用Django原生方式返回JSON：
```python
from django.http import JsonResponse
import json

def article_list(request):
    articles = list(Article.objects.values())  # 手动序列化
    return JsonResponse({'data': articles})
```

但这种方式缺少：自动序列化/反序列化、输入验证、认证权限、分页、自动API文档。这些在DRF中都是内置的。

# 二、安装与配置DRF
```bash
pip install djangorestframework
```

```python
# settings.py
INSTALLED_APPS = [
    # ... Django内置应用
    'rest_framework',  # 注册DRF
    'blog',
]

# DRF全局配置（可选）
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
}
```

# 三、Serializer序列化器
## 3.1 Serializer是什么
Serializer（序列化器）的核心作用是**模型对象 ↔ JSON的双向转换**：

```
Python 模型对象  ←→  Serializer  ←→  JSON 字符串
  Article(id=1,                   {"id":1,
   title="标题",                     "title":"标题",
   author=User(...))                "author":"张三"}

序列化（Serialization）：模型对象 → JSON（给前端）
反序列化（Deserialization）：JSON → 模型对象（前端提交的数据存入数据库）
```

## 3.2 定义Serializer
在应用下创建`serializers.py`：

```python
# blog/serializers.py
from rest_framework import serializers
from .models import Article

class ArticleSerializer(serializers.Serializer):
    """和Form类似，手动定义每个字段"""
    id = serializers.IntegerField(read_only=True)  # read_only：只序列化，不反序列化
    title = serializers.CharField(max_length=200)
    content = serializers.CharField()
    is_published = serializers.BooleanField(default=True)
    pub_date = serializers.DateTimeField(read_only=True)
    views = serializers.IntegerField(read_only=True)

    def create(self, validated_data):
        """创建新文章"""
        return Article.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """更新文章"""
        instance.title = validated_data.get('title', instance.title)
        instance.content = validated_data.get('content', instance.content)
        instance.is_published = validated_data.get('is_published', instance.is_published)
        instance.save()
        return instance
```

## 3.3 ModelSerializer：自动生成序列化器
上面手动写的Serializer和ModelForm一样有大量重复代码。DRF提供了`ModelSerializer`自动生成：

```python
# blog/serializers.py
from rest_framework import serializers
from .models import Article

class ArticleSerializer(serializers.ModelSerializer):
    # 自动从模型生成字段
    # 只需重写需要特殊处理的字段

    # 自定义字段：显示作者名而不是author_id
    author_name = serializers.CharField(source='author.username', read_only=True)
    # 计算字段
    word_count = serializers.SerializerMethodField()

    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'author', 'author_name',
                  'is_published', 'pub_date', 'views', 'word_count']
        read_only_fields = ['id', 'pub_date', 'views', 'word_count']  # 只读字段

    def get_word_count(self, obj):
        """计算文章字数"""
        return len(obj.content)

    def validate_title(self, value):
        """自定义字段验证（类似Form的clean_字段）"""
        if len(value) < 5:
            raise serializers.ValidationError("标题不能少于5个字符")
        return value

    def validate(self, data):
        """全局验证（类似Form的clean）"""
        if 'title' in data and 'content' in data:
            if data['title'] in data['content']:
                raise serializers.ValidationError("标题不能直接出现在正文中")
        return data
```

## 3.4 在Shell中测试Serializer
```python
python manage.py shell
```

```python
from blog.models import Article
from blog.serializers import ArticleSerializer

# 1. 序列化：模型对象 → Python字典
article = Article.objects.first()
serializer = ArticleSerializer(article)
print(serializer.data)
# {'id': 1, 'title': '...', 'author': 1, 'author_name': 'admin', ...}

# 2. 序列化多个对象
articles = Article.objects.all()
serializer = ArticleSerializer(articles, many=True)
print(serializer.data)  # 列表形式的多个对象

# 3. 反序列化：JSON数据 → 验证 → 保存
data = {'title': '新文章', 'content': '内容...', 'is_published': True}
serializer = ArticleSerializer(data=data)
if serializer.is_valid():
    article = serializer.save(author=request.user)  # save()可传入额外参数
    print(article.id)
else:
    print(serializer.errors)
```

# 四、API视图：@api_view 与 APIView
## 4.1 @api_view装饰器（FBV风格）
```python
# blog/api_views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Article
from .serializers import ArticleSerializer

@api_view(['GET', 'POST'])
def article_list(request):
    """文章列表API：GET获取列表，POST创建文章"""
    if request.method == 'GET':
        # 获取所有已发布的文章
        articles = Article.objects.filter(is_published=True)
        serializer = ArticleSerializer(articles, many=True)  # many=True 处理多条
        return Response(serializer.data)

    elif request.method == 'POST':
        # 创建文章
        serializer = ArticleSerializer(data=request.data)  # request.data 是DRF解析后的数据
        if serializer.is_valid():
            serializer.save(author=request.user)  # 设置作者为当前用户
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
def article_detail(request, pk):
    """文章详情API：GET获取，PUT更新，DELETE删除"""
    try:
        article = Article.objects.get(pk=pk)
    except Article.DoesNotExist:
        return Response({'error': '文章不存在'}, status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = ArticleSerializer(article)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = ArticleSerializer(article, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        article.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

## 4.2 APIView（CBV风格）
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.shortcuts import get_object_or_404

class ArticleListAPIView(APIView):
    """文章列表API"""
    def get(self, request):
        articles = Article.objects.filter(is_published=True)
        serializer = ArticleSerializer(articles, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = ArticleSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(author=request.user)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class ArticleDetailAPIView(APIView):
    """文章详情API"""
    def get_object(self, pk):
        return get_object_or_404(Article, pk=pk)

    def get(self, request, pk):
        article = self.get_object(pk)
        serializer = ArticleSerializer(article)
        return Response(serializer.data)

    def put(self, request, pk):
        article = self.get_object(pk)
        serializer = ArticleSerializer(article, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        article = self.get_object(pk)
        article.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

## 4.3 对比Django原生视图和DRF视图
| 特性 | Django FBV | DRF @api_view | DRF APIView |
|------|-----------|---------------|-------------|
| 请求对象 | `request` | `request`（增强版） | `request`（增强版） |
| 数据获取 | `request.POST` / `request.GET` | `request.data`（统一处理JSON/表单） | 同左 |
| 响应返回 | `HttpResponse` / `JsonResponse` | `Response`（自动根据请求头选择JSON/HTML） | 同左 |
| 内容协商 | 无 | 自动 | 自动 |
| 异常处理 | 需手动try-except | 自动转换为JSON错误响应 | 同左 |
| 认证权限 | 无 | 可在装饰器中配置 | 可在类属性中配置 |

# 五、API的URL配置
```python
# blog/urls.py
from django.urls import path
from . import views          # 模板视图
from . import api_views       # API视图

urlpatterns = [
    # 模板视图（给浏览器用户看）
    path('', views.index, name='index'),

    # API接口（给前端JS/Vue/React/App调用）
    path('api/articles/', api_views.article_list, name='api-article-list'),
    path('api/articles/<int:pk>/', api_views.article_detail, name='api-article-detail'),

    # 如果用APIView
    # path('api/articles/', api_views.ArticleListAPIView.as_view()),
    # path('api/articles/<int:pk>/', api_views.ArticleDetailAPIView.as_view()),
]
```

# 六、在浏览器中测试API
DRF自带一个可视化的API浏览器界面。在浏览器中直接访问API URL（如`http://127.0.0.1:8000/blog/api/articles/`），你会看到一个漂亮的API交互页面——不需要Postman或APIFOX就能测试接口。

这个界面自动包含：
- GET请求的结果展示
- POST/PUT的输入表单
- DELETE的确认按钮
- OPTIONS请求的方法列表

> 这个界面只在开发环境出现。生产环境可以关闭或替换为Swagger文档（下篇讲）。

# 七、用我们JS篇的Fetch调用DRF API
还记得JS篇15和16写的Fetch获取数据吗？现在你可以用自己写的DRF API替代jsonplaceholder了：

```javascript
// 获取文章列表
async function fetchArticles() {
    const response = await fetch('/blog/api/articles/');
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    const articles = await response.json();
    console.log(articles);
    // 渲染到数据看板中...
}

// 创建文章
async function createArticle(title, content) {
    const response = await fetch('/blog/api/articles/', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-CSRFToken': getCSRFToken(),  // Django要求POST带CSRF token
        },
        body: JSON.stringify({ title, content, is_published: true })
    });
    return response.json();
}
```

这就是前后端分离的完整闭环：**Python Django + DRF提供数据 → JavaScript Fetch获取数据 → DOM渲染到页面**。

# 八、常见误区与避坑指南
1.  **Serializer和Form混淆**：两者长得像（都做验证和数据处理），但用途不同。Form用于传统Django模板表单（返回HTML），Serializer用于API数据（返回JSON）。

2.  **忘记在视图中设置many=True**：序列化多条记录时必须`ArticleSerializer(articles, many=True)`。如果忘记，DRF会试图把QuerySet当作单条记录处理，抛出错误。

3.  **request.data vs request.POST**：在DRF视图中应该使用`request.data`而不是`request.POST`。因为`request.data`能处理JSON请求体，而`request.POST`只能处理表单格式的数据。

4.  **创建时忘记在save()中传入额外字段**：`serializer.save(author=request.user)`可以在保存时传入ModelSerializer不包含的字段。

5.  **ModelSerializer的fields使用__all__**：和ModelForm一样，`fields = '__all__'`会暴露所有字段。显式列出需要暴露的字段列表是更好的实践。

6.  **API路由和模板路由混淆**：建议把API视图和模板视图分别放在不同的文件（如`views.py`和`api_views.py`），URL也加`api/`前缀区分。

# 九、核心总结：DRF入门速查表
## Serializer
```python
class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ['id', 'title', 'content']
        read_only_fields = ['id']

# 序列化：对象 → 数据
serializer = ArticleSerializer(article)        # 单条
serializer = ArticleSerializer(articles, many=True)  # 多条

# 反序列化：数据 → 验证 → 保存
serializer = ArticleSerializer(data=request.data)
if serializer.is_valid():
    article = serializer.save(author=request.user)
```

## API视图
```python
# FBV风格
@api_view(['GET', 'POST'])
def article_list(request):
    # request.data 获取请求数据
    # Response(data) 返回响应

# CBV风格
class ArticleListAPIView(APIView):
    def get(self, request): ...
    def post(self, request): ...
```

## HTTP方法对应的CRUD
| HTTP方法 | 含义 | Django URL |
|----------|------|-----------|
| `GET` | 读取 | `/api/articles/`（列表）`/api/articles/1/`（详情） |
| `POST` | 创建 | `/api/articles/` |
| `PUT` | 更新 | `/api/articles/1/` |
| `DELETE` | 删除 | `/api/articles/1/` |

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
