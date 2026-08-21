
# Python全栈入门到实战【Django篇 11】DRF进阶：ViewSet、Router、认证与自动API文档
上一篇《Django篇 10》中，我们学习了DRF的基础——用Serializer序列化模型、用APIView编写接口。但你会注意到：文章列表和文章详情是两个视图类，URL也要写两条规则。如果每个模型都这样写，代码量并不比FBV少太多。而且，接口的身份认证（谁可以调这个API？）、权限控制（能做什么操作？）、API文档（前端怎么知道有哪些接口？）这些在真实项目中的刚需问题还没有解决。

DRF的真正威力在于它的**ViewSet + Router架构**和**内置认证权限系统**。ViewSet把同一个资源的所有操作（列表/创建/详情/更新/删除）合并到一个类中，Router自动生成URL——你写一个类，DRF帮你生成一套完整的RESTful接口。再加上Token/JWT认证和Swagger文档自动生成，你就拥有了一个企业级的API服务。

本节核心学习内容：
1.  ViewSet：一个类搞定CRUD所有操作
2.  Router：自动生成RESTful URL路由
3.  ModelViewSet：最精简的代码生成完整API
4.  认证系统：Session/Token/JWT三种方式对比
5.  权限控制：IsAuthenticated、IsAdminUser、自定义权限
6.  drf-spectacular：一行命令生成Swagger/OpenAPI文档
7.  实战：博客API完整版 + 用JS看板对接

# 一、ViewSet + Router：自动化API的神器
## 1.1 为什么需要ViewSet
回顾上一篇文章，我们为文章写了两个类：
```python
class ArticleListAPIView(APIView):    # GET(列表) + POST(创建)
class ArticleDetailAPIView(APIView):  # GET(详情) + PUT(更新) + DELETE(删除)
```

如果有10个模型（文章、评论、分类、标签、用户...），就需要20个类、20条URL。ViewSet解决的就是这个问题——把一个资源的所有操作合并到一个类中。

## 1.2 ViewSet实现
```python
# blog/viewsets.py
from rest_framework import viewsets
from rest_framework.response import Response
from .models import Article
from .serializers import ArticleSerializer

class ArticleViewSet(viewsets.ViewSet):
    """自定义ViewSet：手动实现每个方法"""

    def list(self, request):
        """获取列表（对应 GET /articles/）"""
        queryset = Article.objects.filter(is_published=True)
        serializer = ArticleSerializer(queryset, many=True)
        return Response(serializer.data)

    def retrieve(self, request, pk=None):
        """获取详情（对应 GET /articles/1/）"""
        article = Article.objects.get(pk=pk)
        serializer = ArticleSerializer(article)
        return Response(serializer.data)

    def create(self, request):
        """创建文章（对应 POST /articles/）"""
        serializer = ArticleSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(author=request.user)
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)

    def update(self, request, pk=None):
        """全量更新（对应 PUT /articles/1/）"""
        article = Article.objects.get(pk=pk)
        serializer = ArticleSerializer(article, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=400)

    def destroy(self, request, pk=None):
        """删除（对应 DELETE /articles/1/）"""
        article = Article.objects.get(pk=pk)
        article.delete()
        return Response(status=204)
```

## 1.3 Router：自动生成URL
```python
# blog/urls.py 或项目 urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import viewsets

router = DefaultRouter()
router.register(r'articles', viewsets.ArticleViewSet, basename='article')

urlpatterns = [
    path('api/', include(router.urls)),
]
```

Router自动生成以下URL：
```
GET    /api/articles/         → list()     → 文章列表
POST   /api/articles/         → create()   → 创建文章
GET    /api/articles/{pk}/    → retrieve() → 文章详情
PUT    /api/articles/{pk}/    → update()   → 更新文章
PATCH  /api/articles/{pk}/    → partial_update() → 部分更新
DELETE /api/articles/{pk}/    → destroy()  → 删除文章
```

一个`router.register()`，六条URL自动生成。不用手写URL映射，不用为HTTP方法分发写if-else。

# 二、ModelViewSet：最精简的API实现
## 2.1 ModelViewSet的出现
对于标准的增删改查API，上面的`viewsets.ViewSet`写了很多重复代码。DRF提供了`ModelViewSet`——只需指定queryset和serializer_class，CRUD全部自动生成：

```python
# blog/viewsets.py
from rest_framework import viewsets
from .models import Article
from .serializers import ArticleSerializer

class ArticleModelViewSet(viewsets.ModelViewSet):
    """最精简的API实现：三行代码 = 完整的CRUD接口"""
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer

    # 根据需求重写需要定制的方法（可选）
    def get_queryset(self):
        """只返回已发布的文章"""
        return Article.objects.filter(is_published=True)

    def perform_create(self, serializer):
        """创建时自动设置作者"""
        serializer.save(author=self.request.user)
```

**三行代码 = 六条API接口**。这就是DRF的生产力。

## 2.2 ViewSet的继承层次
```
APIView            → 最基础的手动控制
GenericViewSet      → 手动操作 + 自动序列化
    ├── ViewSet    → 需手动实现每个方法
    └── GenericAPIView
    └── ModelViewSet → 全自动CRUD（最常用）
```

## 2.3 在ModelViewSet中重写行为
```python
class ArticleModelViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer

    def get_queryset(self):
        """自定义查询逻辑：按请求参数筛选"""
        queryset = Article.objects.all()
        category = self.request.query_params.get('category', None)
        if category:
            queryset = queryset.filter(category__slug=category)
        return queryset

    def get_serializer_class(self):
        """根据操作使用不同的Serializer"""
        if self.action == 'list':
            return ArticleListSerializer  # 列表用精简版
        return ArticleDetailSerializer     # 详情用完整版

    def perform_create(self, serializer):
        """创建时的钩子函数"""
        serializer.save(author=self.request.user)

    def perform_update(self, serializer):
        """更新时的钩子函数"""
        serializer.save(updated_at=timezone.now())
```

ModelViewSet提供了丰富的钩子函数，让你可以在不破坏默认行为的前提下插入自定义逻辑。

# 三、认证系统：谁在调用API
## 3.1 DRF支持的认证方式
| 认证方式 | 适用场景 | 前端如何传凭证 |
|---------|---------|--------------|
| `SessionAuthentication` | 前后端同域名（传统的模板+API混合项目） | Cookie自动传递 |
| `TokenAuthentication` | 前后端分离项目（Token永不过期） | Header: `Authorization: Token xxx` |
| `JWTAuthentication` | 前后端分离项目（Token有过期时间的Token） | Header: `Authorization: Bearer xxx` |

## 3.2 全局配置与局部配置
```python
# settings.py 全局配置（默认适用于所有APIView）
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',  # 浏览器cookie
    ],
}

# 视图级别配置（覆盖全局）
from rest_framework.authentication import SessionAuthentication, TokenAuthentication
from rest_framework.permissions import IsAuthenticated

class ArticleModelViewSet(viewsets.ModelViewSet):
    # 局部配置：只对这个ViewSet生效
    authentication_classes = [TokenAuthentication]
    permission_classes = [IsAuthenticated]  # 只有认证用户才能访问
```

## 3.3 Token认证配置
```bash
# 安装DRF自带的Token认证
# settings.py
INSTALLED_APPS = [
    'rest_framework.authtoken',  # 注册
]
```
```bash
python manage.py migrate  # 创建Token表
```

```python
# 为用户创建Token
from rest_framework.authtoken.models import Token
from django.contrib.auth.models import User

user = User.objects.get(username='admin')
token, created = Token.objects.get_or_create(user=user)
print(token.key)  # "ee8c3d4e..." — 这就是API Token
```

前端调用时携带Token：
```javascript
fetch('/api/articles/', {
    headers: {
        'Authorization': 'Token ee8c3d4e...',
    }
})
```

# 四、权限控制：能做什么操作
## 4.1 内置权限类
| 权限类 | 含义 | 使用场景 |
|--------|------|---------|
| `AllowAny` | 任何人（包括未登录者） | 公开API |
| `IsAuthenticated` | 只有登录用户 | 需要登录才能调用的接口 |
| `IsAdminUser` | 只有管理员（is_staff=True） | 管理后台API |
| `IsAuthenticatedOrReadOnly` | 未登录者只读，登录者可写 | 公开阅读，登录操作 |
| `DjangoModelPermissions` | 基于Django Model的权限 | 细粒度权限控制 |

```python
from rest_framework.permissions import IsAuthenticated, IsAdminUser, AllowAny

class ArticleModelViewSet(viewsets.ModelViewSet):
    # 列表和详情：任何人可看
    # 创建：需要登录
    # 编辑/删除：只有作者本人可以
    def get_permissions(self):
        """根据操作动态设置权限"""
        if self.action in ['list', 'retrieve']:
            return [AllowAny()]  # 任何人都可以浏览
        if self.action in ['create']:
            return [IsAuthenticated()]  # 登录用户可以创建
        return [permissions.IsAdminUser()]  # 只有管理员可以编辑/删除
```

## 4.2 自定义权限类
```python
# blog/permissions.py
from rest_framework.permissions import BasePermission

class IsAuthorOrReadOnly(BasePermission):
    """只有文章作者可以修改，其他人只能读取"""

    def has_object_permission(self, request, view, obj):
        # 读操作（GET/HEAD/OPTIONS）始终允许
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        # 写操作（POST/PUT/DELETE）只允许作者
        return obj.author == request.user
```

```python
class ArticleModelViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthorOrReadOnly]
```

# 五、自动API文档：Swagger
## 5.1 安装drf-spectacular
```bash
pip install drf-spectacular
```

```python
# settings.py
INSTALLED_APPS = [
    'drf_spectacular',
]

REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

SPECTACULAR_SETTINGS = {
    'TITLE': '博客系统API',
    'DESCRIPTION': 'Python全栈博客系统的RESTful API接口文档',
    'VERSION': '1.0.0',
}
```

```python
# urls.py
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView

urlpatterns = [
    # 生成OpenAPI Schema文件
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    # Swagger UI可视化界面
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    # API路由
    path('api/', include(router.urls)),
]
```

访问`http://127.0.0.1:8000/api/docs/`，你会看到一个完整的Swagger API文档页面，所有接口的路径、参数、请求体结构、响应示例自动展示，并且可以直接在页面中测试每个接口。

这就是**自动文档驱动开发**——你写完ViewSet，前端工程师打开Swagger页面，所有接口一目了然，不需要你额外写任何文档。

# 六、前后端分离的完整闭环
结合我们前面学过的所有知识，看一个完整的前后端协作过程：

**后端（Django + DRF）**：
```python
# blog/viewsets.py
class ArticleModelViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.filter(is_published=True)
    serializer_class = ArticleSerializer
    authentication_classes = [TokenAuthentication]
    permission_classes = [IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

**前端（JavaScript原生）**：
```javascript
// 和Django篇的后端API对接
async function loadArticles() {
    const res = await fetch('/api/articles/');
    const articles = await res.json();  // 后端DRF返回的JSON被JS接收
    renderArticles(articles);           // 渲染到页面
}

async function createArticle() {
    const res = await fetch('/api/articles/', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Token ' + localStorage.getItem('token'),
        },
        body: JSON.stringify({
            title: '新文章',
            content: '内容...',
        })
    });
    if (res.status === 201) {
        const article = await res.json();
        console.log('创建成功：', article);
    }
}
```

这就是全栈开发的完整闭环——JS Fetch发出请求 → DRF APIView接收 → Serializer验证 → Model存入数据库 → Response返回JSON → JS接收并渲染。你掌握了这个循环，就是真正的全栈开发者。

# 七、常见误区与避坑指南
1.  **在不需要完整CRUD的场景用了ModelViewSet**：如果你只需要只读接口（不含创建/编辑/删除），用`ReadOnlyModelViewSet`而不是`ModelViewSet`，更安全且语义更明确。

2.  **Token认证把Token放在URL参数中**：`Authorization: Token xxx`放在请求头中是最佳实践。放在URL中（如`/api/?token=xxx`）会被代理服务器、CDN记录到日志中，造成Token泄露。

3.  **忘记在Router中设置basename**：如果ViewSet没有指定`queryset`属性（如手动写的序列化逻辑），Router无法自动推断模型名，此时必须手动指定`basename`，否则会报错。

4.  **自定义权限函数中忘记添加只读判断**：大多数自定义权限都应该允许GET/HEAD/OPTIONS方法（只读操作）。如果`has_object_permission`直接返回`obj.author == request.user`，未登录用户连文章都看不到。

5.  **权限类中的返回False是403不是401**：权限不足返回`403 Forbidden`，认证失败返回`401 Unauthorized`。这是两个不同的HTTP状态码，测试时注意区分。

# 八、核心总结
## ViewSet + Router
```python
# 1. 定义ViewSet
class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer

# 2. 注册Router
router = DefaultRouter()
router.register('articles', ArticleViewSet)

# 3. 一行URL规则自动生成6条接口
urlpatterns = [path('api/', include(router.urls))]
```

## 认证与权限
```python
# 全局配置
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': ['rest_framework.authentication.TokenAuthentication'],
    'DEFAULT_PERMISSION_CLASSES': ['rest_framework.permissions.IsAuthenticated'],
}

# 局部覆盖
class MyViewSet(viewsets.ModelViewSet):
    authentication_classes = [SessionAuthentication]
    permission_classes = [IsAuthenticatedOrReadOnly]
```

## Router自动生成的URL
```
GET    /{prefix}/         → list
POST   /{prefix}/         → create
GET    /{prefix}/{pk}/    → retrieve
PUT    /{prefix}/{pk}/    → update
PATCH  /{prefix}/{pk}/    → partial_update
DELETE /{prefix}/{pk}/    → destroy
```

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
