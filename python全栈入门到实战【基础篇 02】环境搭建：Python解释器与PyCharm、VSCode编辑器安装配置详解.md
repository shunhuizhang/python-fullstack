

> 很多同学可能缺乏编程基础或对电脑操作不太熟悉，在安装解释器或编辑器时容易遇到问题。但请别因这些小挫折灰心——学习路上本就没有一帆风顺，跨过这些门槛，就能迎来更顺畅的学习之旅。接下来，就让我们一步步完成环境搭建，每个步骤都标注关键注意点，确保零失误！

# 一、Python解释器的下载与安装

**核心作用**：Python解释器是运行Python代码的“发动机”，没有它，写好的代码无法执行，必须优先安装并配置。

**官网地址**：<https://www.python.org/downloads/>（唯一安全下载渠道，避免第三方平台）

![img](https://i-blog.csdnimg.cn/direct/e298dd3722114d858093d98927805f69.png)

## 什么是Python解释器？
Python解释器是执行Python代码的核心程序。与C、Java等编译型语言不同，Python是解释型语言，代码无需提前编译成机器语言，运行时由解释器逐行翻译、逐行执行，新手无需理解底层原理，只需确保安装成功即可。

### 下载Python解释器（Windows系统，以Win10/11为例）
> Mac用户可跳过此部分，直接查看Mac版安装步骤

### 第一步：访问官方下载页
1. 打开浏览器，输入链接：[Python解释器Windows下载网址](https://www.python.org/downloads/windows/)，进入官网下载专区。
2. 页面会自动识别你的操作系统（Windows），无需手动选择系统类型。

### 第二步：选择合适的版本（关键！）
1. 页面顶部会显示“Latest Python 3 Release - Python x.x.x”（x.x.x为最新版本号，如3.12.4），这是**推荐新手下载的最新稳定版**，兼容性和安全性最优。
2. 若需特定历史版本（如项目要求Python 3.9），可下拉页面找到“Looking for a specific release?”，点击“View the full list of downloads”，在历史版本列表中选择对应版本。
3. **版本选择原则**：新手必选Python 3.8及以上版本（3.8~3.12均可），Python 2.x已停止维护，绝对不要下载。

![img](https://i-blog.csdnimg.cn/direct/a7d1372d02164568a277a6fc933fcdd7.png)

### 第三步：下载安装包
1. 找到对应版本的“Files”下载区域，根据电脑位数选择：
   - 64位系统（主流电脑均为64位）：选择“Windows Installer (64-bit)”
   - 32位系统（老旧电脑）：选择“Windows Installer (32-bit)”
2. 点击下载链接，等待安装包保存到电脑（默认保存在“下载”文件夹，建议记住保存路径）。

![img](https://i-blog.csdnimg.cn/direct/75db679467cf4324886bcea44fd3fba8.png)

### 第四步：安装操作（每一步都标清注意点）
1. 找到下载好的安装包（文件名如python-3.12.4-amd64.exe），**双击运行**。
2. 弹出安装界面，关键操作：勾选底部的“Add Python.exe to PATH”（添加到环境变量），这一步直接影响后续能否正常使用Python，**必须勾选**！
   - 选择安装类型：
     - 新手推荐“Install Now”（默认安装）：自动安装到系统默认路径，无需手动设置。
     - 有需求的用户可选择“Customize installation”（自定义安装）：可修改安装路径（建议安装到非C盘，如D:\Python312）。

![img](https://i-blog.csdnimg.cn/direct/d6af71ee1d4f4d7faab41b267d26ee28.png)

3. 若选择“自定义安装”，进入下一步后，将“Optional Features”（可选功能）全部勾选：
   - Documentation（文档）、pip（包管理工具，后续装插件必备）、tcl/tk and IDLE（简易编辑器）、Python test suite（测试工具）、py launcher（版本切换工具）、for all users（所有用户可用），全部勾选后点击“Next”。

![img](https://i-blog.csdnimg.cn/direct/fb1c3ddf32e0414db1b60f003d52b9b2.png)

4. 若需修改安装路径，点击“Browse”选择目标文件夹（如D:\Python312），确认后点击“Install”。

![img](https://i-blog.csdnimg.cn/direct/c0e1182f7633463ba9b417e96c396829.png)

5. 等待安装完成：安装过程中会显示进度条，耗时1~5分钟（取决于电脑配置），期间不要关闭窗口，耐心等待。

![img](https://i-blog.csdnimg.cn/direct/9ba9c8baa22f49ed8ffb1aef622e9e5a.png)

6. 安装成功：弹出“Setup was successful”提示，说明安装完成，点击“Close”关闭窗口。

![img](https://i-blog.csdnimg.cn/direct/7c7687bde72d44739e238daa534d61ba.png)

### 第五步：验证安装是否成功（必做！）
1. 按下Win+R键，输入“cmd”，打开命令提示符（黑窗口）。
2. 在命令行中输入`python --version`（注意空格），按下回车。
3. 若显示“Python x.x.x”（如Python 3.12.4），说明安装和环境变量配置成功；若提示“python不是内部或外部命令”，则是未勾选“Add to PATH”，需重新安装并勾选该选项。

## 下载Python解释器（Mac系统，以macOS 10.15及以上为例）
> Windows用户可跳过此部分

### 第一步：访问官方下载页
1. 打开浏览器，输入官网地址：<https://www.python.org/downloads/>。
2. 点击页面顶部“Downloads”，在下拉菜单中选择“macOS”，进入Mac版下载专区。

![img](https://i-blog.csdnimg.cn/direct/f80692b745fb411da2e41321580246a4.png)

![img](https://i-blog.csdnimg.cn/direct/c1be9b6b10684ec1a3d6723edadcf17d.png)

### 第二步：选择合适的版本
1. 页面顶部会推荐最新稳定版（如Python 3.12.4），新手直接选择此版本即可。
2. 若需历史版本，下拉页面点击“View the full list of downloads”，在历史版本中选择对应版本。
3. **版本选择原则**：新手选3.8~3.12的最新稳定版，确保与macOS版本兼容（官网会自动匹配，无需手动判断）。

### 第三步：下载安装包
1. 在对应版本的“Files”区域，选择“macOS 64-bit universal2 installer”（兼容Intel和M1/M2芯片的Mac），点击下载。
2. 安装包默认保存到“下载”文件夹，可在浏览器下载记录中找到。

![img](https://i-blog.csdnimg.cn/direct/98d3dca5903a433586e8eb143ecc251c.png)

![img](https://i-blog.csdnimg.cn/direct/f42170ffc8d04506b71543918401ba42.png)

### 第四步：安装操作（详细步骤）
1. 找到下载好的安装包（文件名如python-3.12.4-macos11.pkg），双击运行。
2. 弹出安装向导，点击“继续”。

![img](https://i-blog.csdnimg.cn/direct/ab5f325517134f4b8bcfd664432bf234.png)

3. 再次点击“继续”，查看软件许可协议，点击“同意”。

![img](https://i-blog.csdnimg.cn/direct/b6cc82d8a96e4ecea21c6fb9d9a8cf44.png)
![img](https://i-blog.csdnimg.cn/direct/4d972a8a2faf4d7b8d40351c3f765d45.png)

4. 点击“安装”，系统会提示输入Mac登录密码或验证指纹（授权安装），输入后继续。

![img](https://i-blog.csdnimg.cn/direct/acd55a67fc0e4a5a9c5f1eae444d5cfa.png)

![img](https://i-blog.csdnimg.cn/direct/4fa9841fbc3a45efb83d931081e0355b.png)

![img](https://i-blog.csdnimg.cn/direct/41d17602a6f34f0b8059e700893cb5ce.png)

![img](https://i-blog.csdnimg.cn/direct/b03ffbe958d44d6ab5df2ee654436112.png)

5. 安装完成后，点击“关闭”退出向导。

### 第五步：验证安装是否成功（必做！）
1. 打开“终端”（通过Launchpad搜索“终端”或在“应用程序-实用工具”中找到）。
2. 在终端中输入`python3 --version`（注意是python3，不是python），按下回车。
3. 若显示“Python x.x.x”（如Python 3.12.4），说明安装成功；若提示“command not found”，需重新检查安装步骤。

![img](https://i-blog.csdnimg.cn/direct/5887248bf38b4f7ab467c552b24b2e4c.png)

（注：Mac也可通过Homebrew安装，但Homebrew本身需要配置，新手优先选择上述官网安装方式，更简单不易出错）

# 二、代码编辑器的下载与安装

Python解释器仅能运行代码，需搭配编辑器编写代码。推荐两款常用工具，**二选一即可**：
- 优先选PyCharm：专为Python设计，功能全面，新手易上手，适合后端开发。
- 选VSCode：轻量级，支持多语言，适合前端开发或简单脚本编写。

## （一）PyCharm的下载与安装（推荐新手）
### 第一步：了解PyCharm版本区别
- 社区版（Community）：免费、功能够用于日常学习（支持Python开发核心功能），**新手首选**。
- 专业版（Professional）：付费（可试用30天），支持Web开发、数据库连接等高级功能，新手暂时无需，如后续需要Pycharm专业版的同学，订阅专栏并关注博主后，可以私信博主，博主给你安排。

### 第二步：下载PyCharm
1. 访问官网：<https://www.jetbrains.com/pycharm/>。

![img](https://i-blog.csdnimg.cn/direct/c834cd13d8c94defbbfcad9edb9aea76.png)

2. 官网默认显示英文，可点击页面右上角语言图标，切换为“简体中文”，方便查看。

![img](https://i-blog.csdnimg.cn/direct/65698fa2f2fa4cee9c704581a73b86dc.png)

3. 点击“下载”，进入下载页面后，默认推荐专业版，下拉页面找到“社区版”，点击对应系统的下载按钮（Windows/Mac自动识别）。

![img](https://i-blog.csdnimg.cn/direct/d629494ab8a346668b6a04551ac280a7.png)

4. 若需特定版本（如旧版兼容老系统），点击页面底部“其他版本”，选择对应版本下载。

![img](https://i-blog.csdnimg.cn/direct/b9b35c8de72e493e83dec987c7552fb6.png)

![img](https://i-blog.csdnimg.cn/direct/dd579ff5e37f4ccf96baa4d1ee275cfd.png)

### 第三步：安装PyCharm（详细步骤）
1. 找到下载好的安装包（Windows为.exe文件，Mac为.dmg文件），双击运行。
   - Windows系统：双击安装包后，点击“下一步”。

![img](https://i-blog.csdnimg.cn/direct/4b0bac1d3fc24a6d941e2ba32d97f1be.png)

![img](https://i-blog.csdnimg.cn/direct/a45bdb9b938846389d9c4edb4f447087.png)

2. 自定义安装路径（建议安装到非C盘，如D:\Program Files\JetBrains\PyCharm Community Edition 2024.1），选择后点击“下一步”。

![img](https://i-blog.csdnimg.cn/direct/eacca7e14d7542dcb4176f2e4e611d21.png)

3. 勾选安装选项（全部勾选，无需修改）：
   - 64-bit launcher（创建64位快捷方式）
   - Add launchers dir to PATH（添加到环境变量）
   - Create associations（关联.py文件，双击.py文件用PyCharm打开）
   - Update context menu（添加右键菜单）
     勾选后点击“下一步”。

![img](https://i-blog.csdnimg.cn/direct/afa43814fd674ca6a9a3b5e940913f01.png)

4. 保持默认配置（开始菜单文件夹无需修改），点击“安装”，等待安装完成（耗时3~10分钟，取决于电脑配置）。

![img](https://i-blog.csdnimg.cn/direct/25d2d6de4f1b482db953824a76fbd73a.png)

5. 安装完成后，勾选“否，我会在之后重新启动”，点击“完成”，启动PyCharm。

![img](https://i-blog.csdnimg.cn/direct/75c3d685a8c141da860254944190f3af.png)

### 第四步：首次打开PyCharm并配置（关键！）
1. 首次启动会弹出欢迎界面，选择“Do not import settings”（不导入之前的配置），点击“OK”。

2. 选择界面语言：默认已为简体中文（若未切换，可在后续设置中修改），点击“下一步”。

![img](https://i-blog.csdnimg.cn/direct/2a2e833602f9448cb95a1cf26e0c5d57.png)

3. 勾选用户许可协议，点击“确定”。

![img](https://i-blog.csdnimg.cn/direct/23d5cacfa79b4450b31cd838e24569a5.png)

4. 弹出数据共享提示，选择“不发送”（可选，根据个人意愿）。

![img](https://i-blog.csdnimg.cn/direct/bb88e0004757496c9ef4d5baf04cb70f.png)

5. 进入新建项目界面，点击“新建项目”。

![img](https://i-blog.csdnimg.cn/direct/b4683dd5319b4975ab5b9ea29b3e3acc.png)

6. 弹出项目信任提示，点击“信任项目”（否则无法正常编辑代码）。

![img](https://i-blog.csdnimg.cn/direct/62bdc9db1962476897a87df1e7b4a1ae.png)

7. 配置Python解释器（核心步骤，否则无法运行代码）：
   - 点击界面右下角“添加解释器”（或进入设置：中文点击“文件-设置”，英文点击“File-Settings”）。

![img](https://i-blog.csdnimg.cn/direct/c01272dce69b480daa6da696e7b2bb52.png)

   - 在设置中找到“项目：xxx- Python解释器”，点击右上角“添加”。

   中文界面：
   ![img](https://i-blog.csdnimg.cn/direct/622886a6d54d4343ab42f816cb2fc214.png)
   英文界面：
   ![img](https://i-blog.csdnimg.cn/direct/5571e67d9d004c07b14d72259cbeb21a.png)

   - 选择“现有环境”，点击“浏览”，找到之前安装的Python解释器可执行文件：
     - Windows：默认路径如C:\Users\用户名\AppData\Local\Programs\Python\Python312\python.exe（若自定义安装则为对应路径）
     - Mac：默认路径如/Library/Frameworks/Python.framework/Versions/3.12/bin/python3
   - 选中python.exe（或python3）后，点击“确定”。

![img](https://i-blog.csdnimg.cn/direct/c19b602067a64c2093c2a037047a6dba.png)

   - 回到设置界面，点击“应用”→“确定”，完成解释器配置。

![img](https://i-blog.csdnimg.cn/direct/e48a6b4601744879ba4e4fef8440a7db.png)

### 第五步：验证PyCharm是否可用
1. 在项目左侧的“项目”面板中，右键点击项目名称→“新建”→“Python文件”。

![img](https://i-blog.csdnimg.cn/direct/e3fc30b334f84543a89119b6781adc76.png)

2. 输入文件名（如hello），无需加后缀（PyCharm自动添加.py），按回车创建文件。

![img](https://i-blog.csdnimg.cn/direct/1073b3a9a2ee4f80b7737ab2c630b731.png)

3. 在编辑区输入代码：
   ```python
   print('hello world')
   ```
4. 右键点击编辑区空白处，选择“运行'hello'”，或点击代码编辑区右上角的绿色三角形按钮运行。
5. 若底部控制台输出“hello world”，说明PyCharm安装和配置全部成功！

![img](https://i-blog.csdnimg.cn/direct/d6e4e75de0aa4b17a7fe1845a9cf9668.png)

## （二）VSCode的下载与安装（适合偏好轻量编辑器的用户）
### 第一步：下载VSCode
1. 访问官网：<https://code.visualstudio.com/>。
2. 官网会自动识别你的操作系统（Windows/Mac），点击“Download”下载对应安装包。

![img](https://i-blog.csdnimg.cn/direct/6c43d4c2fbd14bfbb00fdc29a2b43728.png)

### 第二步：安装VSCode（详细步骤）
1. Windows系统安装：
   - 找到下载的安装包（如VSCodeUserSetup-x64-1.89.1.exe），双击运行。
   - 点击“我同意此协议”，点击“下一步”。
   - 修改安装路径（建议非C盘，如D:\Program Files\Microsoft VS Code），点击“下一步”。
   - 勾选“创建桌面快捷方式”“将Code注册为受支持的文件类型的编辑器”，点击“下一步”。

![img](https://i-blog.csdnimg.cn/direct/55b2b051fddc4f028eb91b1940bff675.png)

![img](https://i-blog.csdnimg.cn/direct/5dddb5489c6e43d28c775d0264439f22.png)

   - 点击“安装”，等待完成后点击“完成”，启动VSCode。

![img](https://i-blog.csdnimg.cn/direct/49b5e231f490439eb434170ba08ff1dc.png)

2. Mac系统安装：
   - 双击下载的.dmg文件，将VSCode拖拽到“应用程序”文件夹中，完成安装。

### 第三步：配置VSCode支持Python开发（关键！）
1. 切换中文界面（可选）：
   - 打开VSCode，点击左侧“扩展”图标（或按Ctrl+Shift+X），在搜索框输入“Chinese (Simplified)”，找到对应插件点击“安装”。
   - 安装完成后，点击右下角“重启”，VSCode将切换为中文。

![img](https://i-blog.csdnimg.cn/direct/3c4b986d31414732948497cc1b095a74.png#pic_center)

![img](https://i-blog.csdnimg.cn/direct/84f41e4243d84fba864ab235a7da0c42.png)

2. 安装Python插件（必装）：
   - 在扩展面板搜索框输入“Python”，找到微软官方发布的“Python”插件（图标为蓝色Python标志），点击“安装”。

![img](https://i-blog.csdnimg.cn/direct/ca35004477e147278c1c1abfd3d32f42.png)

3. 选择Python解释器（核心步骤）：
   - 方法一：点击VSCode左下角的“选择解释器”（显示“未选择解释器”）。
   - 方法二：按Ctrl+Shift+P，在弹出的命令框中输入“Python: Select Interpreter”，按回车。
   - 在下拉列表中找到之前安装的Python解释器（Windows为python.exe，Mac为python3），点击选择。

![img](https://i-blog.csdnimg.cn/direct/19ff1ea6fabd49358610517d578b39d0.png)

### 第四步：验证VSCode是否可用
1. 新建Python文件：
   - 点击VSCode左侧“资源管理器”→“打开文件夹”，选择一个文件夹作为项目目录（如D:\PythonProject）。
   - 右键点击文件夹→“新建文件”，输入文件名（如hello.py），按回车创建。

![img](https://i-blog.csdnimg.cn/direct/4c649fb7f9354266829c6c76b1f0356e.png)

![img](https://i-blog.csdnimg.cn/direct/9a932d6ce7454338928d378576759480.png)

2. 输入代码：
   ```python
   print('hello world')
   ```
3. 运行代码：
   - 右键点击编辑区空白处，选择“在终端中运行Python文件”，或点击右上角的绿色三角形按钮。
   - 若底部终端输出“hello world”，说明VSCode安装和配置成功！

![img](https://i-blog.csdnimg.cn/direct/7e6d891af26448b89d1badc392485579.png)

# 三、总结

恭喜你！已经完成了Python环境搭建的全部核心步骤——Python解释器是“发动机”，编辑器是“工作台”，现在你已经拥有了完整的Python开发环境，可以正式开始编写代码了！

如果在安装过程中遇到问题（如找不到解释器路径、运行代码无输出等），欢迎在评论区留言，说明你的操作系统和具体问题，我会第一时间帮你解决。

关注并订阅专栏，后续将持续更新Python基础语法、实战案例等内容，让我们一起从入门到精通！

# 四、专栏订阅

> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后，专栏内所有文章可永久免费查看，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/156536391>
