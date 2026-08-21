
# Python全栈入门到实战【Django篇 22】教育平台实战（五）：购物车与订单支付，构建课程交易系统
上一篇《Django篇 21》中，我们实现了学员系统——用户可以查看已购课程、收藏课程、追踪学习进度。但教育平台要真正"跑起来"，还需要一个关键环节：**支付**。用户看了课程详情觉得不错，点击购买按钮后，需要经历"创建订单→选择支付→完成支付→更新订单状态→开通课程访问权"的完整流程。

订单系统是电商类应用的核心——它不是简单的"加一条记录"，而是一个有状态流转的业务流程。本篇将以支付宝沙箱环境为例，实现完整的课程购买流程：购物车管理、订单生成、支付接口对接、支付回调验签、订单状态更新。

本节核心学习内容：
1.  购物车功能：加入/删除/修改数量（使用Session存储）
2.  订单管理：订单详情节、状态流转、自动生成订单号
3.  支付集成：支付宝沙箱环境配置与对接
4.  支付回调：验签 + 更新订单状态 + 开通课程权限
5.  我的订单：查看历史订单记录

# 一、购物车（Session实现）
```python
# courses/cart.py — 购物车工具类
from .models import Course

class Cart:
    """用Django Session存储购物车数据"""
    def __init__(self, request):
        self.session = request.session
        cart = self.session.get('cart', {})
        if not isinstance(cart, dict):
            cart = {}
        self.cart = cart

    def add(self, course_id):
        """添加课程到购物车"""
        course_id = str(course_id)
        if course_id not in self.cart:
            self.cart[course_id] = {
                'added_at': str(timezone.now()),
            }
        self.save()

    def remove(self, course_id):
        """从购物车移除"""
        course_id = str(course_id)
        if course_id in self.cart:
            del self.cart[course_id]
            self.save()

    def get_courses(self):
        """获取购物车中的所有课程对象"""
        course_ids = [int(cid) for cid in self.cart.keys()]
        return Course.objects.filter(id__in=course_ids, status='published')

    def get_total_price(self):
        """计算总价"""
        courses = self.get_courses()
        return sum(c.price for c in courses)

    def clear(self):
        """清空购物车"""
        self.session['cart'] = {}
        self.save()

    def count(self):
        return len(self.cart)

    def save(self):
        self.session['cart'] = self.cart
        self.session.modified = True
```

```python
# courses/views.py — 购物车视图
from .cart import Cart

@login_required
def cart_view(request):
    """购物车页面"""
    cart = Cart(request)
    return render(request, 'courses/cart.html', {
        'cart_courses': cart.get_courses(),
        'total_price': cart.get_total_price(),
    })

@login_required
def cart_add(request, course_slug):
    """添加课程到购物车"""
    course = get_object_or_404(Course, slug=course_slug, status='published')
    cart = Cart(request)
    cart.add(course.id)
    messages.success(request, f'《{course.title}》已添加到购物车')
    return redirect('courses:cart')

@login_required
def cart_remove(request, course_id):
    """从购物车移除"""
    cart = Cart(request)
    cart.remove(course_id)
    return redirect('courses:cart')
```

# 二、订单创建
```python
@login_required
def create_order(request):
    """从购物车创建订单"""
    cart = Cart(request)
    courses = cart.get_courses()
    if not courses:
        messages.warning(request, '购物车为空')
        return redirect('courses:cart')

    total_amount = cart.get_total_price()

    # 检查是否重复购买
    purchased_course_ids = Order.objects.filter(
        user=request.user, course__in=courses, status='paid'
    ).values_list('course_id', flat=True)

    new_courses = [c for c in courses if c.id not in purchased_course_ids]
    if not new_courses:
        messages.info(request, '购物车中的课程都已购买')
        return redirect('courses:my-courses')

    # 因为是课程（每个课程一个订单）
    orders = []
    for course in new_courses:
        order = Order.objects.create(
            user=request.user, course=course, amount=course.price
        )
        orders.append(order)

    # 清空购物车
    cart.clear()

    # 如果只买了一个课程，直接跳到订单详情
    if len(orders) == 1:
        return redirect('courses:order-detail', order_no=orders[0].order_no)

    return redirect('courses:my-orders')
```

# 三、订单详情与支付入口
```python
from django.views.generic import DetailView, ListView

class OrderDetailView(LoginRequiredMixin, DetailView):
    """订单详情页（支付入口）"""
    model = Order
    template_name = 'courses/order_detail.html'
    context_object_name = 'order'
    slug_field = 'order_no'
    slug_url_kwarg = 'order_no'

    def get_queryset(self):
        return Order.objects.filter(user=self.request.user).select_related('course')
```

```html
<!-- templates/courses/order_detail.html -->
{% extends 'base.html' %}
{% block title %}订单详情{% endblock %}
{% block content %}
<div class="order-detail">
    <h2>订单详情</h2>
    <div class="order-info-card">
        <p><strong>订单号：</strong>{{ order.order_no }}</p>
        <p><strong>课程：</strong>{{ order.course.title }}</p>
        <p><strong>金额：</strong>¥{{ order.amount }}</p>
        <p><strong>状态：</strong>
            {% if order.status == 'pending' %}
                <span class="status-pending">待支付</span>
            {% elif order.status == 'paid' %}
                <span class="status-paid">已支付</span>
            {% endif %}
        </p>
        <p><strong>创建时间：</strong>{{ order.created_at|date:"Y-m-d H:i" }}</p>
    </div>

    {% if order.status == 'pending' %}
    <div class="pay-methods">
        <h3>选择支付方式</h3>
        <a href="{% url 'courses:alipay-pay' order.order_no %}" class="btn-alipay">
            支付宝支付
        </a>
        <a href="{% url 'courses:mock-pay' order.order_no %}" class="btn-mock">
            模拟支付（开发测试用）
        </a>
    </div>
    {% endif %}
</div>
{% endblock %}
```

# 四、支付宝支付集成
## 4.1 支付宝沙箱环境配置
```python
# settings.py
ALIPAY_APP_ID = '你的沙箱APPID'
ALIPAY_APP_PRIVATE_KEY = '''-----BEGIN RSA PRIVATE KEY-----
你的应用私钥
-----END RSA PRIVATE KEY-----'''

ALIPAY_ALIPAY_PUBLIC_KEY = '''-----BEGIN PUBLIC KEY-----
支付宝公钥
-----END PUBLIC KEY-----'''

ALIPAY_NOTIFY_URL = 'https://你的域名/courses/alipay/notify/'  # 生产环境
ALIPAY_RETURN_URL = 'https://你的域名/courses/alipay/return/'
# 开发测试用
ALIPAY_GATEWAY = 'https://openapi-sandbox.dl.alipaydev.com/gateway.do'  # 沙箱
```

## 4.2 支付宝工具类
```python
# courses/alipay.py
import json
from Crypto.PublicKey import RSA
from Crypto.Signature import PKCS1_v1_5
from Crypto.Hash import SHA256
import base64
from urllib.parse import quote_plus
from django.conf import settings

class AliPay:
    def __init__(self):
        self.app_id = settings.ALIPAY_APP_ID
        self.app_private_key = settings.ALIPAY_APP_PRIVATE_KEY
        self.alipay_public_key = settings.ALIPAY_ALIPAY_PUBLIC_KEY
        self.gateway = settings.ALIPAY_GATEWAY
        self.notify_url = settings.ALIPAY_NOTIFY_URL
        self.return_url = settings.ALIPAY_RETURN_URL

    def get_pay_url(self, order_no, amount, subject):
        """生成支付页面URL"""
        biz_content = {
            'out_trade_no': order_no,
            'product_code': 'FAST_INSTANT_TRADE_PAY',
            'total_amount': str(amount),
            'subject': subject,
        }
        params = {
            'app_id': self.app_id,
            'method': 'alipay.trade.page.pay',
            'charset': 'utf-8',
            'sign_type': 'RSA2',
            'timestamp': timezone.now().strftime('%Y-%m-%d %H:%M:%S'),
            'version': '1.0',
            'notify_url': self.notify_url,
            'return_url': self.return_url,
            'biz_content': json.dumps(biz_content, ensure_ascii=False),
        }
        # 生成签名
        sign = self._sign(params)
        params['sign'] = sign
        # 拼接URL
        query_string = '&'.join(f'{k}={quote_plus(str(v))}' for k, v in params.items())
        return f'{self.gateway}?{query_string}'

    def verify(self, data):
        """验证支付宝异步通知签名"""
        sign = data.pop('sign', '')
        unsigned_str = '&'.join(
            f'{k}={data[k]}' for k in sorted(data.keys()) if data[k] not in ('', None)
        )
        return self._verify(unsigned_str, sign)

    def _sign(self, data):
        """签名"""
        unsigned_str = '&'.join(
            f'{k}={data[k]}' for k in sorted(data.keys()) if data[k] not in ('', None)
        )
        key = RSA.importKey(self.app_private_key)
        signer = PKCS1_v1_5.new(key)
        signature = signer.sign(SHA256.new(unsigned_str.encode('utf-8')))
        return base64.b64encode(signature).decode('utf-8')

    def _verify(self, unsigned_str, signature):
        """验签"""
        key = RSA.importKey(self.alipay_public_key)
        verifier = PKCS1_v1_5.new(key)
        return verifier.verify(
            SHA256.new(unsigned_str.encode('utf-8')),
            base64.b64decode(signature)
        )
```

## 4.3 支付视图
```python
from django.views.decorators.csrf import csrf_exempt
from .alipay import AliPay

@login_required
def alipay_pay(request, order_no):
    """发起支付宝支付"""
    order = get_object_or_404(Order, order_no=order_no, user=request.user, status='pending')
    alipay = AliPay()
    pay_url = alipay.get_pay_url(
        order_no=order.order_no,
        amount=order.amount,
        subject=f'购买课程：{order.course.title}'
    )
    return redirect(pay_url)

@csrf_exempt
def alipay_notify(request):
    """支付宝异步通知（需要配置csrf_exempt，因为支付宝POST到你的服务器）"""
    if request.method != 'POST':
        return HttpResponse('error')

    alipay = AliPay()
    data = {k: v[0] for k, v in dict(request.POST).items()}

    # 验签
    if not alipay.verify(data):
        return HttpResponse('fail')

    # 验证订单号和金额
    order_no = data.get('out_trade_no')
    total_amount = data.get('total_amount')
    trade_status = data.get('trade_status')

    try:
        order = Order.objects.get(order_no=order_no)
    except Order.DoesNotExist:
        return HttpResponse('fail')

    if trade_status == 'TRADE_SUCCESS':
        order.status = 'paid'
        order.paid_at = timezone.now()
        order.save()
        # 更新课程销量
        from django.db.models import F
        Course.objects.filter(pk=order.course_id).update(
            total_students=F('total_students') + 1
        )
        return HttpResponse('success')

    return HttpResponse('fail')

def alipay_return(request):
    """用户支付后同步跳回"""
    return redirect('courses:my-courses')

@login_required
def mock_pay(request, order_no):
    """模拟支付（开发测试用）"""
    order = get_object_or_404(Order, order_no=order_no, user=request.user)
    if order.status == 'pending':
        order.status = 'paid'
        order.paid_at = timezone.now()
        order.save()
        Course.objects.filter(pk=order.course_id).update(
            total_students=F('total_students') + 1
        )
        messages.success(request, '支付成功！可以开始学习了')
    return redirect('courses:my-courses')
```

```python
# courses/urls.py 追加
urlpatterns = [
    path('cart/', views.cart_view, name='cart'),
    path('cart/add/<slug:course_slug>/', views.cart_add, name='cart-add'),
    path('cart/remove/<int:course_id>/', views.cart_remove, name='cart-remove'),
    path('order/create/', views.create_order, name='create-order'),
    path('order/<str:order_no>/', views.OrderDetailView.as_view(), name='order-detail'),
    path('my-orders/', views.MyOrdersView.as_view(), name='my-orders'),
    path('alipay/pay/<str:order_no>/', views.alipay_pay, name='alipay-pay'),
    path('alipay/notify/', views.alipay_notify, name='alipay-notify'),
    path('alipay/return/', views.alipay_return, name='alipay-return'),
    path('mock-pay/<str:order_no>/', views.mock_pay, name='mock-pay'),
]
```

# 五、我的订单
```python
class MyOrdersView(LoginRequiredMixin, ListView):
    """我的订单列表"""
    model = Order
    template_name = 'courses/my_orders.html'
    context_object_name = 'orders'
    paginate_by = 20

    def get_queryset(self):
        return Order.objects.filter(
            user=self.request.user
        ).select_related('course').order_by('-created_at')
```

# 六、订单状态流转图
```
购物车 → 创建订单 → status='pending'
                        ↓
                发起支付（alipay_pay）
                        ↓
                    跳转到支付宝
                        ↓
                用户完成支付（扫码/登录）
                        ↓
            支付宝POST异步通知（alipay_notify）
                        ↓
              验签通过 → status='paid'
                        ↓
              更新课程 total_students+1
                        ↓
            用户获得课程访问权限
```

# 七、核心总结
## 关键安全措施
| 措施 | 说明 |
|------|------|
| 异步通知验签 | 确认通知确实来自支付宝（防伪造） |
| 金额校验 | 确认通知中的金额和订单金额一致（防篡改） |
| 幂等处理 | 同一订单只处理一次（防重复通知） |
| csrf_exempt | 异步通知接口必须取消CSRF保护（支付宝POST不带CSRF token） |

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
