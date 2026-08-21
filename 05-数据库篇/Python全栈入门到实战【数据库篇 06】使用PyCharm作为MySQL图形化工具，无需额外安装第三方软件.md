

# Python全栈入门到实战【数据库篇 06】使用PyCharm作为MySQL图形化工具，无需额外安装第三方软件
上一篇《数据库篇 05》中，我们已经完成了MySQL数据库的安装与基础配置，掌握了SQL的基本语法和常用操作。本篇作为数据库篇的第六篇，我们将学习**PyCharm自带的Database数据库工具**的使用方法。无需额外安装Navicat、DBeaver等第三方图形化工具，在PyCharm中即可一站式完成Python代码开发和数据库操作，大幅提升开发效率。

本文为Python全栈开发者与数据库入门者量身打造，全程采用步骤式教学，每一个操作都有清晰的截图和详细说明，即使是完全没有数据库工具使用经验的同学，也能跟着一步步完成MySQL数据库的连接和基本操作。

本节核心学习内容：
1. 工具介绍：PyCharm Database工具的优势与基本功能
2. 驱动安装：首次使用时MySQL驱动的自动下载与配置
3. 连接配置：数据库连接参数的填写与连接测试
4. 界面介绍：连接成功后的界面布局与核心功能区
5. 基本操作：SQL代码编写执行、表数据可视化编辑
6. 常见问题：连接失败、驱动异常等问题的排查与解决
7. 核心总结：PyCharm数据库工具使用速查表，方便开发时快速查阅

# 一、PyCharm Database工具介绍
PyCharm Professional版（社区版不支持）内置了功能强大的Database数据库工具，支持连接MySQL、PostgreSQL、SQLite、Oracle等几乎所有主流数据库，具备以下核心优势：
- **一站式开发**：在同一个IDE中完成Python代码编写和数据库操作，无需切换软件
- **语法高亮与提示**：支持SQL语法高亮、自动补全和错误提示
- **可视化操作**：可以直接在界面中查看表结构、编辑表数据、执行SQL查询
- **版本控制集成**：数据库脚本可以和Python代码一起纳入版本控制
- **无需额外安装**：PyCharm自带，开箱即用，无需下载安装第三方软件

# 二、MySQL数据库连接详细步骤
## 2.1 打开Database工具面板
1. 打开PyCharm，找到屏幕最右侧的**database**
2. 点击该按钮，展开Database工具面板

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5a5e2867148b46049e668ce96c4f3031.png#pic_center)


## 2.2 添加MySQL数据源
1. 在Database工具面板的顶部，点击这个 + 号
2. 在弹出的菜单中，先点击**Data Source**，然后选择MySQL

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e8c9b9ae6aba4332a358221585d59335.png#pic_center)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/29dfbe8c339e4b7bb077a97d79ef2363.png#pic_center)


## 2.3 安装MySQL驱动
首次打开这个界面，需要安装mysql驱动，你的这个地方会出现一个安装驱动的提示(因为我已经安装过，所以这里没有提示了，不过如果你们没有安装过，肯定会有的。)，点击左下角的Download下载，等待下载完成即可

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/dda8bcd387264140923cbd9ab1bc082b.png#pic_center)


> ⚠️ 注意：如果驱动下载失败，可以检查网络连接，或者手动下载驱动包后导入到PyCharm中。

## 2.4 填写数据库连接参数
在数据源配置界面中，填写以下基本连接信息：
| 参数名称 | 说明               | 默认值                     |
| -------- | ------------------ | -------------------------- |
| Host     | MySQL服务器地址    | localhost（本地数据库）    |
| Port     | MySQL端口号        | 3306                       |
| User     | 数据库用户名       | root（默认管理员用户）     |
| Password | 数据库密码         | 你安装MySQL时设置的密码    |
| Database | 要连接的数据库名称 | 可选，不填则显示所有数据库 |

## 2.5 测试并保存连接
1. 填写基本信息如下：host、User、Password、port，然后点击Test Connection测试连接。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/958a1824a44e4de88cbfaff9177b279f.png#pic_center)


2. 连接成功会出现如下提示：

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1fe5a05db46644d89a9c6fb0b2b67846.png#pic_center)


# 三、连接成功后的界面与基本操作
## 3.1 连接成功后的界面布局
成功连接MySQL数据库后，界面会发生以下变化：
1. 左侧Database工具面板中会显示你连接的MySQL数据库，展开后可以看到所有的数据库和表
2. 底部会自动打开一个**console**控制台，用于编写和执行SQL代码
3. 双击任意表名，可以在右侧打开表数据的可视化编辑界面

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/56411e3cefd04c72b6a57812a4e86fb1.png#pic_center)


## 3.2 执行SQL代码
1. 在底部的Console控制台中，编写你要执行的SQL语句
2. 选中要执行的SQL语句（不选中则执行全部）
3. 点击控制台上方的绿色运行按钮，或按`Ctrl+Enter`快捷键执行
4. 执行结果会显示在控制台下方的结果面板中

## 3.3 可视化编辑表数据
1. 在左侧Database工具面板中，双击要编辑的表名
2. 右侧会打开表数据的可视化界面
3. 直接在表格中修改数据，修改完成后点击上方的**Submit**按钮提交更改
4. 也可以点击上方的**+**号添加新行，或点击**-**号删除选中的行

# 四、常见问题排查与解决
1. **驱动下载失败**：检查网络连接，或手动下载MySQL Connector/J驱动包，然后在数据源配置界面的**Driver**选项卡中导入
2. **连接超时**：检查MySQL服务器是否已启动，确认Host和Port参数是否正确
3. **访问被拒绝**：检查用户名和密码是否正确，确认MySQL用户是否有远程访问权限
4. **数据库不显示**：在数据源配置界面的**Schemas**选项卡中，勾选要显示的数据库

# 五、核心总结：PyCharm数据库工具使用速查表
为了方便后续开发时快速查阅，整理了PyCharm数据库工具的核心使用速查表：

| 操作步骤   | 关键操作                         | 核心要点                                  |
| ---------- | -------------------------------- | ----------------------------------------- |
| 打开工具   | 点击右侧Database按钮             | 仅PyCharm Professional版支持              |
| 添加数据源 | 点击+号→Data Source→MySQL        | 支持几乎所有主流数据库                    |
| 安装驱动   | 点击左下角Download按钮           | 首次使用必须安装对应数据库的驱动          |
| 配置连接   | 填写Host、Port、User、Password   | 本地数据库默认Host为localhost，Port为3306 |
| 测试连接   | 点击Test Connection按钮          | 绿色Succeeded提示表示连接成功             |
| 执行SQL    | 在Console中编写代码→按Ctrl+Enter | 可以选中部分SQL语句单独执行               |
| 编辑数据   | 双击表名→直接修改表格→点击Submit | 修改后必须提交才能保存到数据库            |
| 查看表结构 | 右键点击表名→选择Table Editor    | 可以查看和修改表的字段、索引等结构        |

# 六、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/162073132>
