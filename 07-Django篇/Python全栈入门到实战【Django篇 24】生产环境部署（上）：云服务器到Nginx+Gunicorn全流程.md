
# Python全栈入门到实战【Django篇 24】生产环境部署（上）：云服务器到Nginx+Gunicorn全流程
前面23篇文章中，我们在本地开发环境（`python manage.py runserver`）中完成了博客系统和在线教育平台两个完整的Django项目。但`runserver`只适用于本地开发——它性能低、不安全、没有并发处理能力。要让我们的项目真正上线，被全世界的人访问，必须部署到**生产环境**。

部署是很多学习者的"最后一公里"难题——模型写好了、视图写好了、模板写好了，本地`127.0.0.1:8000`一切正常，但不知道如何把它放到互联网上。本篇从零开始，手把手带你将Django项目部署到一台云服务器上，走完从买服务器到`https://你的域名`可以访问的完整流程。

本节核心学习内容：
1.  云服务器选购与初始化：安全组、SSH、基础环境
2.  Python环境配置：pyenv/virtualenv、依赖安装
3.  MySQL数据库配置：创建数据库、字符集、用户权限
4.  Gunicorn：WSGI应用服务器配置与systemd守护
5.  Nginx：反向代理 + 静态文件 + 域名配置
6.  HTTPS：Certbot免费SSL证书
7.  部署检查清单

# 一、服务器选购与初始化
## 1.1 选择云服务器
选购一台最低配置的云服务器即可运行Django项目：

| 项目 | 推荐配置 |
|------|---------|
| CPU | 1-2核 |
| 内存 | 2GB+ （MySQL需要较多内存） |
| 系统盘 | 40GB+ |
| 操作系统 | Ubuntu 22.04 LTS（推荐）或 CentOS 7 |
| 带宽 | 1Mbps+ |

国内推荐阿里云/腾讯云/华为云，国外推荐AWS/DigitalOcean。

## 1.2 安全组配置
在云服务商控制台配置**安全组（防火墙规则）**：

| 端口 | 协议 | 用途 | 来源 |
|------|------|------|------|
| 22 | TCP | SSH远程登录 | 你的IP（推荐）/ 0.0.0.0 |
| 80 | TCP | HTTP | 0.0.0.0 |
| 443 | TCP | HTTPS | 0.0.0.0 |

**安全建议**：SSH端口（22）最好限制为只有你的IP才能访问，防止暴力破解。

## 1.3 SSH登录服务器
```bash
# 本地终端
ssh root@你的服务器公网IP

# 首次登录提示确认指纹，输入yes
# 输入密码（云服务商会提供初始密码）
```

```bash
# 登录后第一步：更新系统
sudo apt update && sudo apt upgrade -y
```

# 二、Python环境搭建
```bash
# 1. 安装Python3和必要工具
sudo apt install -y python3 python3-pip python3-venv git

# 2. 创建项目目录
mkdir -p /var/www/blog_project
cd /var/www/blog_project

# 3. 创建虚拟环境
python3 -m venv venv
source venv/bin/activate

# 4. 安装项目依赖（先从本地上传 requirements.txt）
pip install -r requirements.txt

# 5. 安装Gunicorn（WSGI服务器）
pip install gunicorn
```

# 三、代码与静态文件部署
```bash
# 方式1：Git拉取（推荐）
cd /var/www/blog_project
git clone https://github.com/你的用户名/blog_project.git src
# 或使用私有仓库

# 方式2：scp上传（临时）
# scp -r blog_project root@IP:/var/www/blog_project/

# 收集静态文件到统一目录
python src/manage.py collectstatic --noinput
# 静态文件会被收集到 STATIC_ROOT 目录（settings.py中配置）
```

```python
# settings.py 生产环境配置
import os

DEBUG = False
ALLOWED_HOSTS = ['你的域名', '服务器IP']

# 静态文件收集目录
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATIC_URL = 'static/'

# 媒体文件
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = 'media/'

# SECRET_KEY 从环境变量读取（不要硬编码在代码中！）
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
```

# 四、MySQL数据库配置
```bash
# 1. 安装MySQL
sudo apt install -y mysql-server

# 2. 安全初始化
sudo mysql_secure_installation
# 按提示设置root密码、移除匿名用户、禁止远程root登录等

# 3. 创建数据库和用户
sudo mysql -u root -p
```

```sql
-- 创建数据库（utf8mb4支持emoji和中文）
CREATE DATABASE blog_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 创建专用数据库用户
CREATE USER 'django_user'@'localhost' IDENTIFIED BY '强密码';
GRANT ALL PRIVILEGES ON blog_db.* TO 'django_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

```python
# settings.py 生产环境数据库配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'blog_db',
        'USER': 'django_user',
        'PASSWORD': os.environ.get('DB_PASSWORD'),  # 从环境变量读取
        'HOST': 'localhost',
        'PORT': '3306',
        'OPTIONS': {
            'charset': 'utf8mb4',
        },
    }
}
```

```bash
# 执行数据库迁移
cd /var/www/blog_project/src
python manage.py migrate

# 创建超级管理员
python manage.py createsuperuser
```

# 五、Gunicorn配置与systemd守护
## 5.1 测试Gunicorn
```bash
gunicorn --bind 0.0.0.0:8000 blog_project.wsgi:application
# 如果正常输出"Listening at: http://0.0.0.0:8000"，表示配置正确
```

## 5.2 systemd服务配置
创建服务文件让Gunicorn开机自启并后台运行：

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

```ini
[Unit]
Description=Gunicorn daemon for Django blog project
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/blog_project/src
ExecStart=/var/www/blog_project/venv/bin/gunicorn \
    --workers 3 \
    --bind unix:/var/www/blog_project/gunicorn.sock \
    --access-logfile /var/log/gunicorn/access.log \
    --error-logfile /var/log/gunicorn/error.log \
    blog_project.wsgi:application

Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
# 创建日志目录
sudo mkdir -p /var/log/gunicorn
sudo chown www-data:www-data /var/log/gunicorn

# 设置目录权限
sudo chown -R www-data:www-data /var/www/blog_project

# 启动Gunicorn服务
sudo systemctl start gunicorn
sudo systemctl enable gunicorn  # 开机自启
sudo systemctl status gunicorn   # 查看运行状态
```

# 六、Nginx配置
## 6.1 安装Nginx
```bash
sudo apt install -y nginx
```

## 6.2 配置反向代理
```bash
sudo nano /etc/nginx/sites-available/blog
```

```nginx
server {
    listen 80;
    server_name 你的域名.com www.你的域名.com;

    # 静态文件
    location /static/ {
        alias /var/www/blog_project/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 媒体文件
    location /media/ {
        alias /var/www/blog_project/media/;
        expires 30d;
    }

    # 反向代理到Gunicorn
    location / {
        proxy_pass http://unix:/var/www/blog_project/gunicorn.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 上传文件大小限制
        client_max_body_size 50M;
    }
}
```

```bash
# 激活配置
sudo ln -s /etc/nginx/sites-available/blog /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default  # 删除默认配置

# 测试配置
sudo nginx -t

# 重启Nginx
sudo systemctl restart nginx
```

访问 `http://你的域名`，应该能看到你的Django网站了。

# 七、HTTPS配置
```bash
# 安装Certbot
sudo apt install -y certbot python3-certbot-nginx

# 一键获取SSL证书
sudo certbot --nginx -d 你的域名.com -d www.你的域名.com

# 按提示输入邮箱、同意协议
# Certbot会自动修改Nginx配置添加HTTPS
```
Certbot会自动续期证书：
```bash
# 测试自动续期
sudo certbot renew --dry-run
```

# 八、部署检查清单
```
☐ DEBUG = False
☐ SECRET_KEY 从环境变量读取
☐ ALLOWED_HOSTS 已配置域名和IP
☐ MySQL密码已修改为强密码
☐ 静态文件已收集（collectstatic）
☐ 数据库迁移已执行（migrate）
☐ 超级管理员账号已创建（createsuperuser）
☐ Gunicorn服务已启动且设为自启
☐ Nginx配置已生效（nginx -t通过）
☐ HTTPS证书已生效
☐ 防火墙/安全组：80和443端口已开放
```

# 九、核心总结
## 请求完整路径（生产环境）
```
用户浏览器（https://你的域名.com）
    → Nginx（监听80/443端口）
        → 静态文件：直接返回（/static/, /media/）
        → 动态请求：反向代理到Gunicorn
            → Gunicorn（通过Unix Socket接收）
                → Django应用（wsgi.py）
                    → 处理请求 → 返回响应
```

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
