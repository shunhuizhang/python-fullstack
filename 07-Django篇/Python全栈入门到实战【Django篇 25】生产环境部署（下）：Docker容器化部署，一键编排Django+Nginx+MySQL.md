
# Python全栈入门到实战【Django篇 25】生产环境部署（下）：Docker容器化部署，一键编排Django+Nginx+MySQL
上一篇《Django篇 24》中，我们完成了Django项目的传统部署——从买服务器到配Nginx到HTTPS证书。这种方式被称为"**裸机部署**"，它的每一步都需要手动操作，换一台服务器就要重复一遍所有步骤。如果项目有多个服务（Django+Gunicorn、Nginx、MySQL、Redis），每个服务需要分别安装和配置，维护起来非常复杂。

Docker的出现改变了这一切。它把应用及其所有依赖打包成一个**标准化的容器**——无论在哪个服务器上，只要安装了Docker引擎，一个`docker-compose up`命令就能启动整个应用的所有服务。环境一致、配置即代码、部署可复制——这就是Docker的核心理念。

本篇作为Django篇的收官之作，将学习如何将Django项目容器化，用Docker Compose编排Django+Nginx+MySQL多服务架构。

本节核心学习内容：
1.  Docker核心概念：镜像、容器、docker-compose编排
2.  Django应用的Dockerfile编写：多阶段构建优化镜像大小
3.  docker-compose.yml：Django/Nginx/MySQL服务编排
4.  环境变量管理：.env文件、敏感信息保护
5.  数据持久化：volumes + MySQL数据备份
6.  容器健康检查与日志管理
7.  传统部署 vs Docker部署对比

# 一、Docker核心概念速览
| 概念 | 说明 | 类比 |
|------|------|------|
| **镜像（Image）** | 应用的"安装包"，包含代码+运行时+依赖 | Python的.whl包 |
| **容器（Container）** | 镜像的运行实例，相互隔离 | 虚拟环境（但更彻底） |
| **Dockerfile** | 定义如何构建镜像的脚本 | requirements.txt + setup.py |
| **docker-compose** | 编排多个容器的工具 | Django的settings.py（定义所有服务） |
| **Volume** | 数据持久化存储（容器删除后数据不丢） | 外挂硬盘 |
| **Network** | 容器间的通信网络 | 局域网 |

# 二、Django应用Dockerfile
```dockerfile
# Dockerfile（放在项目根目录）
# ========== 构建阶段 ==========
FROM python:3.12-slim AS builder

WORKDIR /app

# 安装系统依赖（MySQL客户端和编译工具）
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    default-libmysqlclient-dev \
    && rm -rf /var/lib/apt/lists/*

# 先复制依赖文件（利用Docker缓存层）
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# ========== 运行阶段 ==========
FROM python:3.12-slim

WORKDIR /app

# 安装运行时依赖（只需要MySQL客户端库）
RUN apt-get update && apt-get install -y --no-install-recommends \
    default-libmysqlclient-dev \
    && rm -rf /var/lib/apt/lists/*

# 从构建阶段复制pip安装的包
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# 复制项目代码
COPY . .

# 收集静态文件
RUN python manage.py collectstatic --noinput

# 创建非root用户运行应用
RUN useradd -m django && chown -R django:django /app
USER django

# Gunicorn启动
CMD ["gunicorn", "blog_project.wsgi:application", \
     "--bind", "0.0.0.0:8000", \
     "--workers", "3", \
     "--access-logfile", "-"]
```

**多阶段构建的关键优势**：构建阶段（builder）包含编译工具（gcc等），生成的`.whl`包复制到运行阶段。最终镜像不包含编译工具，体积减小50%以上。

# 三、Nginx Dockerfile
```dockerfile
# nginx/Dockerfile
FROM nginx:1.25-alpine

# 删除默认配置
RUN rm /etc/nginx/conf.d/default.conf

# 复制自定义Nginx配置
COPY nginx.conf /etc/nginx/conf.d/
```

```nginx
# nginx/nginx.conf
upstream django_app {
    server web:8000;  # web是docker-compose中Django服务的名称
}

server {
    listen 80;
    server_name _;

    # 静态文件
    location /static/ {
        alias /app/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 媒体文件
    location /media/ {
        alias /app/media/;
        expires 30d;
    }

    # 反向代理到Gunicorn
    location / {
        proxy_pass http://django_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        client_max_body_size 50M;
    }
}
```

# 四、docker-compose.yml：多服务编排
```yaml
# docker-compose.yml（项目根目录）
version: '3.8'

services:
  # ===== 1. Django + Gunicorn =====
  web:
    build: .
    container_name: django_web
    restart: always
    ports:
      - "8000:8000"   # 开发时可暴露，生产环境由Nginx代理
    depends_on:
      mysql:
        condition: service_healthy  # 等MySQL健康检查通过后才启动
      redis:
        condition: service_started
    environment:
      - DJANGO_SECRET_KEY=${DJANGO_SECRET_KEY}
      - DB_NAME=${DB_NAME}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_HOST=mysql       # docker-compose的服务名作为主机名
      - DB_PORT=3306
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media
    command: >
      sh -c "python manage.py migrate --noinput &&
             python manage.py collectstatic --noinput &&
             gunicorn blog_project.wsgi:application --bind 0.0.0.0:8000 --workers 3"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ===== 2. Nginx =====
  nginx:
    build: ./nginx
    container_name: django_nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - web
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media
      - ./nginx/ssl:/etc/nginx/ssl  # SSL证书目录

  # ===== 3. MySQL =====
  mysql:
    image: mysql:8.0
    container_name: django_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql           # 数据持久化
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql  # 初始化SQL脚本
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

  # ===== 4. Redis（可选，用于缓存和Session） =====
  redis:
    image: redis:7-alpine
    container_name: django_redis
    restart: always
    volumes:
      - redis_data:/data

volumes:
  static_volume:
  media_volume:
  mysql_data:
  redis_data:
```

# 五、环境变量文件（.env）
```bash
# .env（不要提交到Git！）
DJANGO_SECRET_KEY=django-insecure-你的生产环境密钥
MYSQL_ROOT_PASSWORD=强root密码
DB_NAME=blog_db
DB_USER=django_user
DB_PASSWORD=强数据库密码
```

```bash
# .env.example（提交到Git，供团队成员参考）
DJANGO_SECRET_KEY=请替换为你的密钥
MYSQL_ROOT_PASSWORD=请替换
DB_NAME=blog_db
DB_USER=django_user
DB_PASSWORD=请替换
```

# 六、构建与启动
```bash
# 1. 在服务器上安装Docker和Docker Compose
curl -fsSL https://get.docker.com | bash
sudo apt install -y docker-compose-plugin

# 2. 上传项目代码（Git clone）
git clone https://github.com/你的用户名/blog_project.git
cd blog_project

# 3. 创建.env文件（填入真实值）
nano .env

# 4. 构建并启动所有服务
docker compose up -d --build

# 5. 查看运行状态
docker compose ps

# 6. 创建超级管理员（在web容器内执行）
docker compose exec web python manage.py createsuperuser

# 7. 查看日志
docker compose logs -f web
```

# 七、数据库备份
```bash
# 手动备份
docker compose exec mysql mysqldump -u root -p blog_db > backup_$(date +%Y%m%d).sql

# 定时备份脚本（crontab）
# 0 2 * * * cd /var/www/blog_project && docker compose exec -T mysql mysqldump -u root -p${MYSQL_ROOT_PASSWORD} blog_db > backups/$(date +\%Y\%m\%d).sql
```

# 八、传统部署 vs Docker部署对比
| 维度 | 传统部署 | Docker部署 |
|------|---------|-----------|
| 环境一致性 | 每台服务器手动配置 | 镜像确保环境完全一致 |
| 服务启动 | 每个服务单独安装 | 一条命令启动所有服务 |
| 服务器迁移 | 重复所有配置步骤 | git clone + docker compose up |
| 依赖隔离 | 虚拟环境（仅Python） | 容器完全隔离（OS级） |
| 资源占用 | 低 | 略高（每个容器有开销） |
| 学习曲线 | 需了解Linux运维 | 需了解Docker体系 |
| 适合场景 | 单服务器、长期稳定项目 | 多服务器、微服务、快速扩缩容 |

# 九、Django篇总结：从入门到部署的完整技术栈
```
Python基础（语法、函数、面向对象、并发）
    ↓
MySQL（数据库设计、SQL、事务）
    ↓
HTML/CSS（Flex/Grid布局、响应式）
    ↓
JavaScript（DOM操作、事件、异步Fetch）
    ↓
Django（25篇）
    ├── 框架基础（9篇）：路由/视图/模板/模型/ORM/表单/用户/类视图/中间件
    ├── DRF（2篇）：Serializer/APIView/ViewSet/Router/认证/Swagger
    ├── 博客实战（6篇）：文章系统/评论/用户/SEO/完整项目
    ├── 教育平台（6篇）：课程/视频/学员/订单/支付/数据看板
    └── 部署上线（2篇）：传统部署/Docker容器化
    ↓
全栈开发者 ← 能独立完成从数据库到前端到部署的完整Web应用
```

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
