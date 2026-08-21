哈喽各位小伙伴！今天咱们进入Python实操的核心环节——这篇文章会手把手带大家搞定「PyCharm编辑器使用」「编写第一个Python程序」，再深入讲解「输入输出语法」，全程实操无废话，新手跟着做就能上手，赶紧码住开始学习～

# 一、前置准备：PyCharm安装与基础配置
在写代码前，咱们得先搞定“装备”——PyCharm是Python最常用的编辑器之一，功能强大还容易上手，新手直接冲「社区版」就行（免费且足够日常学习/不足后续开发），如后续需要Pycharm专业版的同学，订阅专栏并关注博主后，可以私信博主，博主给你安排。

## 1. 下载与安装（超简单步骤）
1.  下载地址：[PyCharm官方下载](https://www.jetbrains.com/pycharm/download/#section=windows)（根据自己的系统选Windows/macOS/Linux，认准「Community Edition」社区版）；
2.  安装步骤：双击安装包，选择安装路径（建议自定义路径，避免装在C盘，比如D:\PyCharm\），勾选「Create Desktop Shortcut」创建桌面快捷方式，其余默认下一步，最后点击「Finish」完成安装。

✨ 补充说明：PyCharm的详细下载安装步骤（含避坑细节）可查看上篇文章，链接：[此处预留上篇文章链接位置]，确保大家能顺利搞定安装~

## 2. 首次启动与核心配置（避坑重点）
打开PyCharm后，首次启动会有一些基础设置，跟着走就行：
- 选择界面主题：新手推荐「Light」浅色主题，眼睛更舒服，后续可在「File → Settings → Appearance & Behavior → Appearance」中修改；

- 配置Python解释器（关键！）： 
  点击「New Project」创建新项目，在「Location」中选择项目保存路径（比如D:\PythonProjects\FirstProject），由于大家还是初学者，所以这里就先不给大家上难度了，所以这里先不使用虚拟环境，虚拟环境的内容在后面的文章会进行讲解，直接在「Custom environment」中选择咱们之前安装好的Python.exe路径（比如D:\Python39\python.exe），最后点击「Create」完成项目创建，具体步骤如下图所示：

  ![76740996730](https://i-blog.csdnimg.cn/direct/e9fd669dca5e456eb9d2245e068075b0.png#pic_center)

  ✨ 避坑提醒：如果找不到Python解释器，大概率是安装Python时没勾选「Add Python to PATH」，详细的Python下载安装步骤可查看上文，链接：[此处预留上篇文章链接位置]，也可重新安装Python并勾选该选项，或手动浏览找到Python安装目录下的python.exe。

# 二、实操第一步：编写你的第一个Python程序（Hello World）
配置好PyCharm后，咱们直接上手写第一个程序——经典的「Hello World」，感受Python的简洁！

## 1. 创建Python文件

通常在Pycharm中，左侧面板通常是存放项目文件区，右侧是编辑代码区，如下图所示：

![76741047257](https://i-blog.csdnimg.cn/direct/d773dcc4f1114684a44e78c1ab1909c5.png#pic_center)

在PyCharm左侧的「Project」面板中，右键点击咱们创建的项目文件夹（FirstProject），选择「New → Python File」，文件名输入「hello_world」（Python文件名建议用小写字母+下划线，避免中文和特殊符号），「回车」创建文件，如下图所示：

![](https://i-blog.csdnimg.cn/direct/95c5c52c7f9f4c0991aca4510bb5cb1d.gif#pic_center)

## 2. 编写并运行代码
在新建的hello_world.py文件中，输入以下代码：
```python
# 第一个Python程序：输出Hello World
print("Hello World! 我开始学Python全栈啦～")
```
代码解释：
- `#` 后面的内容是注释，注释不会被执行，用来标注代码含义，方便自己和他人阅读；
- `print()` 是Python的内置打印函数，作用是将括号内的内容输出到控制台，括号内可以放字符串（用双引号或单引号包裹）、数字等内容。

运行程序的3种方式（任选其一）：
1.  点击文件编辑区右上角的绿色三角按钮（「Run 'hello_world'」）；

![76770553220](https://i-blog.csdnimg.cn/direct/db44dc38f67b4316bc8318d1bdfced89.png#pic_center)

2. 右键点击代码编辑区，选择「Run 'hello_world'」；

![](https://i-blog.csdnimg.cn/direct/b9d21b77c28946dbbc16c21597abc595.gif#pic_center)

3. 快捷键：Ctrl+Shift+F10（Windows）/ Command+Shift+F10（Mac）。

运行成功后，会在PyCharm底部的「Run」控制台中看到输出结果：
```
Hello World! 我开始学Python全栈啦～
```
恭喜你！成功写出并运行了第一个Python程序，迈出了Python实操的第一步～

## 3. PyCharm基础使用技巧（提高效率）
新手刚用PyCharm可能会觉得功能太多，这里挑几个最常用的技巧，帮大家快速上手：
- 格式化代码：快捷键Ctrl+Alt+L（Windows）/ Command+Option+L（Mac），一键让代码格式更规整，避免手动调整缩进；
- 注释/取消注释：选中代码后，快捷键Ctrl+/（Windows/Mac），可快速添加单行注释（#），再按一次取消注释；
- 控制台清屏：在底部「Run」面板中，右键点击空白处，选择「Clear All」即可清屏，方便查看新的运行结果。

# 三、核心语法：Python输入与输出（交互必备）
学会了打印输出，接下来咱们学习Python的「输入输出」语法——这是实现程序与用户交互的核心，比如让用户输入信息，程序再根据输入内容给出反馈。

## 1. 输出函数：print() 详解
print() 是Python中最常用的输出函数，除了输出简单字符串，还有很多实用用法，咱们逐一讲解：

### （1）输出单个内容
```python
# 输出字符串
print("Python全栈从入门到实战")
# 输出数字（无需加引号）
print(100)
# 输出变量（后续会讲变量，先了解用法）
name = "张三"
print(name)
```
运行结果：
```
Python全栈从入门到实战
100
张三
```

### （2）输出多个内容
多个内容用逗号「,」分隔，print() 会自动用空格连接它们：
```python
print("姓名：", name, "年龄：", 20)
```
运行结果：
```
姓名： 张三 年龄： 20
```

### （3）自定义分隔符
用 `sep` 参数指定多个内容的分隔符，默认是空格：
```python
# 用逗号作为分隔符
print("张三", "20", "男", sep=",")
# 用横线作为分隔符
print("Python", "Java", "C++", sep="---")
```
运行结果：
```
张三,20,男
Python---Java---C++
```

### （4）取消自动换行
print() 默认输出后会自动换行，用 `end` 参数可以取消换行，或指定换行符：
```python
# 取消自动换行，用空格结尾
print("Hello", end=" ")
print("World")
# 用换行符结尾（默认）
print("第一行", end="\n")
print("第二行")
```
运行结果：
```
Hello World
第一行
第二行
```

## 2. 输入函数：input() 详解
input() 函数用来获取用户从控制台输入的内容，语法格式：`变量名 = input("提示信息")`

### （1）基础用法
```python
# 获取用户输入的姓名，提示信息为“请输入你的姓名：”
name = input("请输入你的姓名：")
# 输出用户输入的内容
print("你好，", name)
```
运行程序后，控制台会显示「请输入你的姓名：」，此时输入内容（比如“李四”），按回车确认，程序会输出：
```
请输入你的姓名：李四
你好， 李四
```

### （2）input() 的核心注意点（新手必踩坑）
⚠️ 重点提醒：input() 函数获取到的内容，**默认是字符串类型**，哪怕用户输入的是数字，也会被当作字符串处理！

比如：
```python
# 输入数字18，此时age是字符串类型
age = input("请输入你的年龄：")
print("你的年龄是：", age)
print("age的类型是：", type(age))  # type() 函数用来查看变量类型
```
运行结果：
```
请输入你的年龄：18
你的年龄是： 18
age的类型是： <class 'str'>  # str表示字符串类型
```

如果想让输入的数字作为整数/浮点数使用，需要用 `int()` 或 `float()` 进行类型转换：
```python
# 将字符串转换为整数类型
age = int(input("请输入你的年龄："))
print("明年你的年龄是：", age + 1)  # 可以进行数学运算
# 转换为浮点数类型（适合小数）
height = float(input("请输入你的身高（m）："))
print("你的身高是：", height, "m")
```
运行结果：
```
请输入你的年龄：18
明年你的年龄是： 19
请输入你的身高（m）：1.75
你的身高是： 1.75 m
```

### （3）进阶：输入多个内容
用 `split()` 方法可以让用户一次性输入多个内容，用空格分隔：
```python
# 输入姓名和年龄，用空格分隔
name, age = input("请输入你的姓名和年龄（用空格分隔）：").split()
age = int(age)  # 转换年龄为整数
print(f"姓名：{name}，年龄：{age}岁")  # f-string格式化输出（后续会详解）
```
运行结果：
```
请输入你的姓名和年龄（用空格分隔）：王五 22
姓名：王五，年龄：22岁
```

# 四、实战案例：综合运用输入输出（动手练一练）
咱们结合今天学的内容，写一个简单的「个人信息录入与展示」程序，巩固输入输出和PyCharm的使用：

```python
# 个人信息录入与展示程序
print("=" * 30)  # 输出30个等号，作为分隔线
print("      个人信息录入系统")
print("=" * 30)

# 获取用户输入
name = input("请输入你的姓名：")
age = int(input("请输入你的年龄："))
height = float(input("请输入你的身高（m）："))
hobby = input("请输入你的爱好（用逗号分隔）：")

# 格式化输出信息
print("\n" + "=" * 30)  # 换行后输出分隔线
print("      你的个人信息如下")
print("=" * 30)
print(f"姓名：{name}")
print(f"年龄：{age}岁")
print(f"身高：{height}m")
print(f"爱好：{hobby}")
print("=" * 30)
print("信息录入完成！")
```

运行程序后，按照提示输入信息，最终会展示整理后的个人信息，比如：
```
==============================
      个人信息录入系统
==============================
请输入你的姓名：赵六
请输入你的年龄：25
请输入你的身高（m）：1.8
请输入你的爱好（用逗号分隔）：篮球,编程,旅游

==============================
      你的个人信息如下
==============================
姓名：赵六
年龄：25岁
身高：1.8m
爱好：篮球,编程,旅游
==============================
信息录入完成！
```

大家可以自己修改代码，比如增加「职业」「联系方式」等输入项，练手巩固今天的知识点～

# 五、常见问题排查（新手避坑指南）
1.  运行程序时提示「No Python interpreter configured」： 
    解决：PyCharm未配置解释器，重新进入项目设置（File → Settings → Project: 项目名 → Python Interpreter），点击右上角齿轮，选择「Add」，找到本地Python安装路径下的python.exe，添加即可。
2.  输入数字后，程序报错「ValueError: invalid literal for int() with base 10」： 
    解决：input() 获取的是字符串，转换为整数时，输入内容必须是纯数字，不能包含字母、符号等，重新输入纯数字即可。
3.  print() 输出中文时乱码： 
    解决：打开PyCharm设置（File → Settings → Editor → File Encodings），将「Global Encoding」「Project Encoding」都设置为「UTF-8」，保存后重新运行程序。
4.  代码缩进报错「IndentationError: expected an indented block」： 
    解决：Python对缩进敏感，不同代码块需要统一缩进（建议用4个空格），检查代码是否有缩进不一致的地方，调整后即可。

# 六、总结
今天的内容全是Python实操的核心基础：咱们学会了PyCharm的安装配置和基础使用，编写并运行了第一个Python程序，还掌握了print()输出和input()输入的核心用法，最后通过实战案例巩固了知识点——这些内容是后续学习的基石，一定要多动手练一练，熟练掌握！

下一节，咱们会进入Python核心语法的学习，讲解「变量与简单数据类型，以及python的运行模式」，带大家认识Python中的各种数据，比如字符串、数字、列表等，为后续编写更复杂的程序打基础～

# 七、专栏订阅

> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后，专栏内所有文章可永久免费查看，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/156656860>
