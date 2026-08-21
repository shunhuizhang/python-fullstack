
# Python全栈入门到实战【Django篇 15】博客实战（四）：评论系统，Ajax异步评论与回复嵌套
上一篇《Django篇 14》中，我们完成了文章详情页——Markdown渲染、代码高亮、文章目录、上一篇/下一篇导航。现在文章详情页底部还留着一句"评论区将在下一篇实现"。本篇就来填这个坑，实现一个功能完整的评论系统。

评论系统是博客交互的核心——让读者不仅能看到文章，还能与作者和其他读者交流。一个完整的评论系统包括评论提交（支持Ajax异步提交，不刷新页面）、回复嵌套（A评论了B的评论，C又回复了A）、评论列表展示、XSS安全防护等。这些功能组合起来，能让你的博客从"静态内容展示"升级为"动态互动社区"。

本节核心学习内容：
1.  评论提交：Form表单 + Ajax异步提交（无需刷新页面）
2.  回复嵌套：自引用外键实现评论树形结构
3.  评论列表：递归展示回复层级
4.  XSS防护：django模板自动转义 + html.escape
5.  评论管理：用户删除自己的评论、管理员隐藏评论
6.  综合实战：文章详情页集成完整评论系统

# 一、评论数据模型
评论模型在上一篇（12篇）已经定义好了，这里回顾关键结构：

```python
# blog/models.py (已定义，这里回顾)
class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE, related_name='comments')
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='comments')
    parent = models.ForeignKey('self', null=True, blank=True, on_delete=models.CASCADE, related_name='replies')
    content = models.TextField(verbose_name='评论内容')
    created_at = models.DateTimeField(auto_now_add=True)
    is_active = models.BooleanField(default=True)

    class Meta:
        ordering = ['created_at']
```

自引用外键`parent`是实现回复嵌套的关键：`parent=None`是顶级评论，`parent=某评论`是对该评论的回复。

# 二、评论表单
```python
# blog/forms.py
from django import forms
from .models import Comment

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content']

        widgets = {
            'content': forms.Textarea(attrs={
                'class': 'comment-textarea',
                'rows': 3,
                'placeholder': '写下你的评论...',
            }),
        }

        labels = {
            'content': '',
        }

    def clean_content(self):
        """验证评论内容"""
        content = self.cleaned_data['content'].strip()
        if len(content) < 2:
            raise forms.ValidationError("评论内容不能少于2个字符")
        if len(content) > 500:
            raise forms.ValidationError("评论内容不能超过500个字符")
        # 清洗：HTML转义（防止XSS）
        import html
        return html.escape(content)
```

# 三、评论视图：支持Ajax异步提交
```python
# blog/views.py
from django.shortcuts import get_object_or_404
from django.http import JsonResponse
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_POST
from .models import Article, Comment
from .forms import CommentForm

@login_required
@require_POST
def comment_create(request, article_slug):
    """创建评论（Ajax异步接口）"""
    article = get_object_or_404(Article, slug=article_slug, status='published')
    form = CommentForm(request.POST)

    if form.is_valid():
        comment = form.save(commit=False)
        comment.article = article
        comment.user = request.user
        comment.save()

        # 返回JSON响应（给Ajax回调）
        return JsonResponse({
            'status': 'success',
            'data': {
                'id': comment.id,
                'content': comment.content,
                'username': comment.user.username,
                'created_at': comment.created_at.strftime('%Y-%m-%d %H:%M'),
                'parent_id': comment.parent_id,  # 如果是回复，返回被回复的评论ID
            }
        })
    else:
        return JsonResponse({
            'status': 'error',
            'errors': form.errors,
        }, status=400)

@login_required
def comment_delete(request, pk):
    """删除评论（只允许评论者本人或管理员）"""
    comment = get_object_or_404(Comment, pk=pk)

    if request.user != comment.user and not request.user.is_staff:
        return JsonResponse({'status': 'error', 'message': '无权限'}, status=403)

    comment.delete()
    return JsonResponse({'status': 'success', 'message': '评论已删除'})
```

# 四、URL路由
```python
# blog/urls.py 追加
urlpatterns = [
    # ... 已有路由
    path('articles/<slug:article_slug>/comment/', views.comment_create, name='comment-create'),
    path('comments/<int:pk>/delete/', views.comment_delete, name='comment-delete'),
]
```

# 五、评论模板：替换文章详情页中的占位符
将上一篇14篇中详情模板末尾的评论区占位符替换为完整实现：

```html
<!-- 替换文章详情模板中的评论区 -->
{% if user.is_authenticated %}
<section class="comments-section">
    <h3>{{ article.comments.filter.is_active.count }} 条评论</h3>

    <!-- 评论输入框 -->
    <div class="comment-form-box">
        <div class="comment-user-avatar">{{ user.username.0 }}</div>
        <form id="comment-form" class="comment-form">
            {% csrf_token %}
            <input type="hidden" id="parent-id" name="parent_id" value="">
            <textarea name="content" class="comment-textarea" rows="3" placeholder="写下你的评论..." required></textarea>
            <div class="comment-form-footer">
                <span id="reply-to" class="reply-hint" style="display:none">
                    回复 <span id="reply-username"></span>
                    <button type="button" id="cancel-reply" class="cancel-reply-btn">取消</button>
                </span>
                <button type="submit" class="btn-primary btn-sm">发表评论</button>
            </div>
        </form>
    </div>

    <!-- 评论列表 -->
    <div id="comments-list" class="comments-list">
        {% include 'blog/_comments_tree.html' with comments=article.comments.filter.is_active.parent__isnull.True %}
    </div>
</section>
{% else %}
<section class="comments-section">
    <p class="login-hint">
        <a href="{% url 'users:login' %}?next={{ request.path }}">登录</a> 后可以发表评论
    </p>
    <div class="comments-list">
        {% include 'blog/_comments_tree.html' with comments=article.comments.filter.is_active.parent__isnull.True %}
    </div>
</section>
{% endif %}
```

# 六、评论树递归展示组件
```html
<!-- templates/blog/_comments_tree.html -->
{% for comment in comments %}
<div class="comment-item" id="comment-{{ comment.id }}">
    <div class="comment-avatar">{{ comment.user.username.0 }}</div>
    <div class="comment-body">
        <div class="comment-header">
            <span class="comment-author">{{ comment.user.username }}</span>
            <span class="comment-time">{{ comment.created_at|date:"Y-m-d H:i" }}</span>
            {% if comment.parent %}
                <span class="reply-indicator">回复 @{{ comment.parent.user.username }}</span>
            {% endif %}
        </div>
        <div class="comment-content">{{ comment.content|safe }}</div>
        <div class="comment-actions">
            {% if user.is_authenticated %}
                <button class="reply-btn" data-comment-id="{{ comment.id }}" data-username="{{ comment.user.username }}">
                    回复
                </button>
            {% endif %}
            {% if user == comment.user or user.is_staff %}
                <button class="delete-btn" data-comment-id="{{ comment.id }}" data-url="{% url 'blog:comment-delete' comment.id %}">
                    删除
                </button>
            {% endif %}
        </div>

        <!-- 递归展示回复 -->
        {% if comment.replies.exists %}
        <div class="replies">
            {% include 'blog/_comments_tree.html' with comments=comment.replies.all %}
        </div>
        {% endif %}
    </div>
</div>
{% endfor %}
```

# 七、Ajax前端代码
```javascript
// static/js/blog.js
document.addEventListener('DOMContentLoaded', function() {
    const commentForm = document.getElementById('comment-form');
    const commentsList = document.getElementById('comments-list');
    const parentIdInput = document.getElementById('parent-id');
    const replyTo = document.getElementById('reply-to');
    const replyUsername = document.getElementById('reply-username');
    const cancelReplyBtn = document.getElementById('cancel-reply');
    const csrfToken = document.querySelector('[name=csrfmiddlewaretoken]').value;

    // CSRF Token 获取函数
    function getCSRFToken() {
        return csrfToken;
    }

    // ========== 回复按钮 ==========
    // 使用事件委托：在comments-list上监听所有回复按钮的点击
    commentsList.addEventListener('click', function(e) {
        // 回复按钮
        const replyBtn = e.target.closest('.reply-btn');
        if (replyBtn) {
            const commentId = replyBtn.dataset.commentId;
            const username = replyBtn.dataset.username;
            parentIdInput.value = commentId;
            replyUsername.textContent = username;
            replyTo.style.display = 'block';
            document.querySelector('.comment-textarea').focus();
            return;
        }

        // 删除按钮
        const deleteBtn = e.target.closest('.delete-btn');
        if (deleteBtn) {
            if (!confirm('确认删除这条评论？')) return;
            const url = deleteBtn.dataset.url;
            fetch(url, {
                method: 'POST',
                headers: { 'X-CSRFToken': getCSRFToken() }
            })
            .then(res => res.json())
            .then(data => {
                if (data.status === 'success') {
                    deleteBtn.closest('.comment-item').remove();
                }
            });
            return;
        }
    });

    // ========== 取消回复 ==========
    cancelReplyBtn.addEventListener('click', function() {
        parentIdInput.value = '';
        replyTo.style.display = 'none';
    });

    // ========== 提交评论 ==========
    commentForm.addEventListener('submit', function(e) {
        e.preventDefault();

        const textarea = commentForm.querySelector('textarea');
        const content = textarea.value.trim();
        if (!content) return;

        const formData = new FormData();
        formData.append('content', content);
        const parentId = parentIdInput.value;
        if (parentId) {
            formData.append('parent_id', parentId);
        }

        // 获取当前文章的slug（从URL中提取）
        const pathParts = window.location.pathname.split('/');
        const articleSlug = pathParts[pathParts.length - 2]; // /articles/slug/ → slug

        fetch(`/articles/${articleSlug}/comment/`, {
            method: 'POST',
            headers: { 'X-CSRFToken': getCSRFToken() },
            body: formData
        })
        .then(res => res.json())
        .then(data => {
            if (data.status === 'success') {
                // 清空输入框
                textarea.value = '';
                parentIdInput.value = '';
                replyTo.style.display = 'none';

                // 简单刷新：重新加载评论区
                location.reload();
                // 实际项目中可以动态插入新评论到DOM，避免完全刷新
            } else {
                alert('评论失败：' + JSON.stringify(data.errors));
            }
        })
        .catch(err => {
            console.error('评论提交失败：', err);
            alert('评论提交失败，请稍后重试');
        });
    });
});
```

# 八、CSS评论区样式
```css
/* 追加到 style.css */
.comments-section {
    margin: 40px 0;
    padding-top: 30px;
    border-top: 1px solid #eee;
}
.comments-section h3 { margin-bottom: 20px; font-size: 18px; }

/* 评论输入框 */
.comment-form-box { display: flex; gap: 12px; margin-bottom: 25px; }
.comment-user-avatar {
    width: 38px; height: 38px; border-radius: 50%;
    background: var(--primary); color: white;
    display: flex; justify-content: center; align-items: center;
    font-size: 14px; flex-shrink: 0;
}
.comment-form { flex: 1; }
.comment-textarea {
    width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 5px;
    resize: vertical; font-size: 14px; outline: none;
}
.comment-textarea:focus { border-color: var(--primary); }
.comment-form-footer {
    display: flex; justify-content: space-between; align-items: center;
    margin-top: 8px;
}
.reply-hint { font-size: 13px; color: var(--text-light); }
.cancel-reply-btn {
    background: none; border: none; color: #e74c3c; cursor: pointer;
    margin-left: 6px; font-size: 13px;
}

/* 评论列表 */
.comments-list { margin-top: 20px; }
.comment-item { display: flex; gap: 12px; padding: 15px 0; border-bottom: 1px solid #f0f0f0; }
.comment-item:last-child { border-bottom: none; }
.comment-avatar {
    width: 36px; height: 36px; border-radius: 50%;
    background: #e0e0e0; color: #666;
    display: flex; justify-content: center; align-items: center;
    font-size: 14px; flex-shrink: 0;
}
.comment-body { flex: 1; }
.comment-header { display: flex; gap: 12px; align-items: center; margin-bottom: 5px; }
.comment-author { font-size: 13px; font-weight: 600; color: #333; }
.comment-time { font-size: 12px; color: #999; }
.reply-indicator { font-size: 12px; color: var(--primary); }
.comment-content { font-size: 14px; color: #444; line-height: 1.6; }
.comment-actions { margin-top: 6px; }
.reply-btn, .delete-btn {
    background: none; border: none; font-size: 12px; cursor: pointer;
    margin-right: 12px; padding: 0;
}
.reply-btn { color: var(--primary); }
.delete-btn { color: #e74c3c; }

/* 回复嵌套 */
.replies { margin-left: 30px; border-left: 1px solid #eee; padding-left: 15px; }

/* 登录提示 */
.login-hint { text-align: center; padding: 20px; background: #f8f8f8; border-radius: 5px; color: #999; font-size: 14px; }
.login-hint a { color: var(--primary); }
```

# 九、核心总结：评论系统架构
```
用户输入评论 → Ajax POST → Django View → CommentForm验证
    → 保存到 Comment 表 → 返回 JSON {id, content, username, time}
    → 前端接收JSON → 可选：动态插入DOM（体验更好）或刷新页面
    → 评论以树形结构展示（parent=None为顶级，递归展示replies）
```

## 关键安全措施
| 措施 | 实现 |
|------|------|
| XSS防护 | `html.escape(content)` + 模板自动转义 + safe只在信任时使用 |
| 登录保护 | `@login_required` 装饰器 |
| CSRF保护 | `{% csrf_token %}` + Ajax请求头携带X-CSRFToken |
| 权限校验 | 删除时检查 `request.user == comment.user` |

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
