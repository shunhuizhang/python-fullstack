

# Python全栈入门到实战【AI实战篇 01】DeepSeek官方API全流程实战，第一个AI项目前置必备
欢迎来到Python全栈专栏的AI实战模块！从本篇开始，我们将开启**第一个完整的AI实战项目**——基于DeepSeek API的智能助手开发。这个项目将带你从零开始，掌握AI大模型API的调用方法、接口封装、功能扩展，最终实现一个可集成到Python全栈项目中的智能助手。本篇作为该项目的第一篇，将手把手带你完成DeepSeek官方API的注册、密钥获取、环境配置与基础调用全流程，这是所有AI功能开发的前置基础。

本文为Python全栈开发者与零基础AI入门者量身打造，全程采用步骤式教学，每一个操作都附带详细的截图、参数说明、新手避坑提示，即使是完全没有AI开发经验的同学，也能跟着一步步完成所有操作，成功调用DeepSeek API实现AI对话。

本节核心学习内容：
- DeepSeek API前期准备：官网访问、账号登录与实名认证全流程
- API Key获取：充值操作、密钥创建与安全保存规范
- 官方接口文档查阅：核心信息定位、Python调用示例查找方法
- Python开发环境配置：依赖包安装、环境变量配置与安全规范
- 基础API调用代码编写：官方模板解析、自定义提示词修改
- 代码运行调试：常见环境问题解决、调用结果验证
- 新手常见问题与解决方案：覆盖90%新手会遇到的API调用问题
- 核心总结：DeepSeek API基础操作速查表，方便开发时快速查阅

# 一、前期准备
在开始调用DeepSeek API之前，我们需要先完成官网访问、账号注册与实名认证，这是使用国内所有AI大模型API的通用前置步骤。

## 1.1 访问DeepSeek官网
DeepSeek的所有API服务都通过其官方开放平台提供，首先我们需要访问DeepSeek官方网站。

### 操作步骤
1. 打开电脑上的任意浏览器（推荐Chrome、Edge、Firefox，兼容性更好）；
2. 在浏览器地址栏输入DeepSeek官方地址：`https://www.deepseek.com/`，按下回车键访问网站。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b6d6403481964a6886f970d5fad19ee7.png#pic_center)

## 1.2 账号登录与实名认证
API的调用必须绑定个人账号，且需要完成实名认证，这是国内所有AI开放平台的统一要求，未完成实名认证将无法进行充值和API调用。

### 操作步骤
1. 在DeepSeek官网首页，找到右上角的「API开放平台」按钮，点击进入API服务专属页面；
2. 跳转至登录界面后，输入你的手机号或邮箱，配合密码或验证码完成账号登录；若没有账号，点击「注册」按钮，按照提示完成账号注册；
3. 首次登录API开放平台时，系统会自动引导你完成实名认证，按照页面提示填写真实姓名、身份证号，完成人脸验证即可；
4. 实名认证审核通过后，即可解锁API的所有基础功能。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/27fa3dd21c2349a5a6a812940cdf0c24.png#pic_center)

### 新手避坑提示
- 实名认证必须使用本人真实信息，否则会审核失败；
- 一个身份证只能绑定一个DeepSeek账号，无法重复绑定；
- 实名认证完成后，部分平台会赠送少量免费额度，DeepSeek目前需要先充值才能使用API。

# 二、获取API Key
API Key是调用DeepSeek API的唯一身份凭证，相当于你访问API服务的"钥匙"，所有的API请求都需要携带有效的API Key才能成功执行。

## 2.1 充值操作
DeepSeek API采用按Token计费的模式，Token是大模型处理文本的基本单位，输入和输出的文本都会消耗Token，因此需要先完成充值才能正常使用API。

### 操作步骤
1. 登录并完成实名认证后，在API开放平台首页，找到顶部的「去充值」按钮并点击；
2. 进入充值界面后，可根据学习和测试需求选择充值金额，**建议首次充值1~5元即可**：DeepSeek的Token单价极低，1元可以调用数千次基础对话，足够完成整个智能助手项目的学习和测试；
3. 选择微信或支付宝支付方式，完成支付后，系统会自动将金额计入你的账号余额，可在页面顶部查看实时余额。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ddee2cbe1e89488b867b9e6bbbd986de.png#pic_center)

## 2.2 创建并安全保存API Key
充值完成后，我们需要创建专属的API Key，这是调用API的核心凭证，必须妥善保存，绝对不能泄露给他人。

### 操作步骤
1. 充值完成后，点击左侧导航栏的「API keys」选项，进入密钥管理页面；
2. 点击页面右上角的「创建API key」按钮，在弹出的窗口中输入密钥名称（比如"python-ai-demo"，用于区分不同用途的密钥），点击「创建」；
3. 密钥创建成功后，**立即复制该API Key**（示例密钥：`sk-084ea9d9cc8042eda12aea6400010300`），建议保存至本地加密笔记或文本文档中；
4. 重要提示：**一旦关闭创建成功的弹窗，将无法再次查看该API Key**，如果忘记只能删除旧密钥并重新创建。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0d111114d81a4044bb407db482541c16.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b2214e06ee98448497d8ae96ab0860d7.png#pic_center)

### 安全规范（必须遵守）
1. 绝对不要将API Key硬编码到Python代码中，更不要将包含API Key的代码上传到GitHub、Gitee等公开代码平台，否则会导致密钥泄露，产生不必要的费用损失；
2. 不同的项目、不同的环境，建议创建不同的API Key，方便权限管理和问题排查；
3. 如果发现API Key泄露，立即在API keys页面删除该密钥，重新创建新的密钥。

# 三、查阅官方接口文档
官方接口文档是编写API调用代码的唯一权威参考，包含了API的调用规则、参数说明、返回值格式、错误码等所有核心信息，学会查阅接口文档是AI开发的必备技能。

## 3.1 找到接口文档入口
### 操作步骤
1. 回到API开放平台主页面，下滑页面找到「开发者文档」板块；
2. 点击「接口文档」选项，进入DeepSeek API的官方文档页面。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/88a1e274fef44f5c89f6da21316e2fdc.png#pic_center)

## 3.2 定位Python调用示例
DeepSeek官方提供了多种编程语言的调用示例，其中Python是最常用、最简洁的方式，我们需要找到对应的Python代码模板。

### 操作步骤
1. 在接口文档左侧导航栏，找到「快速开始」→「Python」选项，点击进入；
2. 该页面包含了完整的Python调用代码示例、依赖安装说明、参数解释，这是我们后续编写代码的基础模板。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a555da1b85734345b52ccc75a1b7e2fd.png#pic_center)

# 四、配置Python开发环境
在编写代码之前，我们需要先配置Python开发环境，安装API调用所需的依赖包，并配置环境变量存储API Key，保证代码的安全性。

## 4.1 安装依赖包
DeepSeek API兼容OpenAI的接口规范，因此我们可以直接使用`openai`第三方包来调用DeepSeek API，无需额外安装其他依赖。

### 操作步骤
1. 打开电脑的命令提示符（CMD）、终端，或者你常用的Python编辑器（PyCharm、VS Code）的终端窗口；
2. 输入以下安装命令，按下回车键执行：
   ```bash
   pip install openai
   ```
3. 等待安装完成，若终端出现「Successfully installed openai-xxx」的提示，说明依赖包安装成功。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/00eb0a55522341d6b8e1a678191f8381.png#pic_center)

### 新手避坑提示
- 如果安装速度慢，可以使用国内镜像源加速，命令如下：
  ```bash
  pip install openai -i https://pypi.tuna.tsinghua.edu.cn/simple
  ```
- 确保你的Python版本在3.8及以上，否则可能会出现兼容性问题。

## 4.2 配置环境变量
为了避免API Key硬编码泄露，我们将API Key存储在系统环境变量中，代码运行时自动从环境变量中读取，这是行业通用的安全规范。

### Windows系统环境变量配置步骤
1. 右键点击桌面的「此电脑」，选择「属性」；
2. 在弹出的窗口中，点击左侧的「高级系统设置」；
3. 在「系统属性」窗口中，点击「环境变量」按钮；
4. 在「用户变量」区域（仅当前用户可用），点击「新建」；
5. 变量名填写：`DEEPSEEK_API_KEY`，变量值填写你之前复制的API Key；
6. 依次点击「确定」保存所有设置，完成环境变量配置。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a0507c0fb52c478dbc5caae7b1e9e0f5.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e6b60c190f474603854c108e05cddd67.png#pic_center)

### 新手避坑提示
- 配置完环境变量后，**必须重启所有已打开的终端和Python编辑器**，否则新的环境变量不会被加载；
- 变量名必须严格按照`DEEPSEEK_API_KEY`书写，大小写敏感，否则代码会无法读取到API Key。

# 五、编写基础API调用代码
完成环境配置后，我们就可以编写第一个API调用代码，实现与DeepSeek大模型的基础对话。

## 5.1 复制官方基础代码模板
### 操作步骤
1. 打开你的Python编辑器（PyCharm、VS Code等）；
2. 新建一个Python文件，命名为`deepseek_basic_demo.py`；
3. 将官方接口文档中的Python基础调用代码复制到该文件中：
   ```python
   from openai import OpenAI

   client = OpenAI(
       api_key="你的API Key",
       base_url="https://api.deepseek.com"
   )

   response = client.chat.completions.create(
       model="deepseek-chat",
       messages=[
           {"role": "system", "content": "You are a helpful assistant"},
           {"role": "user", "content": "Hello"},
       ],
       stream=False
   )

   print(response.choices[0].message.content)
   ```

## 5.2 修改代码为安全规范版本
将硬编码的API Key替换为从环境变量读取，符合安全规范：
```python
# 导入所需模块
from openai import OpenAI
import os

# 从环境变量中读取API Key，避免硬编码泄露
api_key = os.getenv("DEEPSEEK_API_KEY")

# 初始化DeepSeek客户端
client = OpenAI(
    api_key=api_key,
    base_url="https://api.deepseek.com"
)

# 调用聊天接口
response = client.chat.completions.create(
    # 指定使用的模型，deepseek-chat是DeepSeek的通用对话模型
    model="deepseek-chat",
    # 对话消息列表，包含系统提示词和用户提示词
    messages=[
        # 系统提示词：定义AI的身份和行为规则
        {"role": "system", "content": "你是一个专业的Python编程助手，擅长用通俗易懂的语言解释编程知识"},
        # 用户提示词：用户的问题或指令
        {"role": "user", "content": "请解释一下Python列表推导式的用法，并给出一个简单的示例"},
    ],
    # 是否开启流式输出，False表示一次性返回所有结果
    stream=False
)

# 打印AI返回的结果
print("DeepSeek的回答：")
print(response.choices[0].message.content)
```

### 代码核心解析
1. `OpenAI`客户端：DeepSeek兼容OpenAI接口，因此直接使用OpenAI客户端，只需修改`base_url`为DeepSeek的API地址；
2. `model`参数：指定要使用的大模型，`deepseek-chat`是DeepSeek最新的通用对话模型，适合绝大多数场景；
3. `messages`参数：对话消息列表，是API调用的核心参数，包含：
   - `system`角色：定义AI的身份、性格、回答规则，会影响AI的所有回答；
   - `user`角色：用户的问题或指令，是AI需要处理的内容；
4. `stream`参数：是否开启流式输出，开启后AI会逐字返回结果，体验更好，我们会在后续章节讲解流式输出的实现。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b056d87bed8c40499522fbc60853332a.png#pic_center)

# 六、运行调试与结果验证
完成代码编写后，我们运行代码，验证API调用是否成功。

## 6.1 解决环境变量加载问题
如果运行代码时出现`api_key`为`None`的错误，说明环境变量没有被正确加载，解决方案如下：
1. 关闭当前所有打开的终端和Python编辑器；
2. 重新打开Python编辑器，加载更新后的环境变量；
3. 再次运行代码，问题即可解决。

### 临时测试方案（仅用于本地测试，禁止生产环境使用）
如果重启编辑器后仍然无法加载环境变量，可以临时在代码中直接写入API Key进行测试：
```python
# 仅用于本地临时测试，绝对不要提交到公开代码平台
client = OpenAI(
    api_key="sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    base_url="https://api.deepseek.com"
)
```
测试完成后，立即改回从环境变量读取的方式，避免密钥泄露。

## 6.2 运行代码并验证结果
### 操作步骤
1. 在编辑器中点击「运行」按钮，或在终端执行以下命令：
   ```bash
   python deepseek_basic_demo.py
   ```
2. 等待代码执行完成（通常1~3秒），终端会输出DeepSeek API返回的回答；
3. 如果终端成功显示AI对"Python列表推导式用法"的解释，说明API调用成功。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/06571085c5de4118b4119314bb016097.png#pic_center)

### 验证成功的标志
- 终端没有报错，正常输出文本内容；
- 输出的内容符合你的提问要求，逻辑清晰、回答准确；
- 可以在DeepSeek API开放平台的「账单明细」页面，看到本次调用的消耗记录。

# 七、新手常见问题与解决方案
新手在首次调用DeepSeek API时，90%的问题都集中在以下场景，这里整理了对应的根因分析和解决方案，遇到问题可以直接对照排查。

---

### 问题1：运行报错`API Key not found`或`api_key is None`
#### 根因分析
- 环境变量配置错误，变量名拼写错误；
- 配置完环境变量后，没有重启Python编辑器和终端，环境变量未加载。
#### 解决方案
1. 核对环境变量名是否为`DEEPSEEK_API_KEY`，大小写完全一致；
2. 核对环境变量值是否为正确的API Key，没有多余的空格或字符；
3. 完全关闭所有终端和Python编辑器，重新打开后再次运行代码。

---

### 问题2：调用扣费异常，余额消耗过快
#### 根因分析
- 提示词过长，输入的文本内容太多，消耗了大量Token；
- 开启了循环调用，导致API被反复调用；
- API Key泄露，被他人盗用。
#### 解决方案
1. 精简提示词，只保留必要的内容，避免大段文本输入；
2. 检查代码是否存在循环调用API的逻辑，及时修复；
3. 立即在API keys页面删除旧密钥，重新创建新的密钥，更换环境变量中的值。

---

### 问题3：实名认证失败
#### 根因分析
- 填写的姓名、身份证号与实际不符；
- 人脸验证时光线不好、角度不对，导致识别失败；
- 该身份证已经绑定过其他DeepSeek账号。
#### 解决方案
1. 核对姓名和身份证号，确保与身份证上的信息完全一致；
2. 在光线充足的环境下重新进行人脸验证，保持面部正对摄像头；
3. 一个身份证只能绑定一个账号，使用未绑定过的身份证进行实名认证。

---

### 问题4：报错`Invalid API Key`
#### 根因分析
- API Key复制错误，多复制了空格或字符；
- API Key已经被删除，失效；
- 环境变量中的API Key值错误。
#### 解决方案
1. 重新复制正确的API Key，更新环境变量；
2. 在API keys页面确认该密钥是否存在，若已删除则重新创建。

---

### 问题5：报错`Insufficient balance`
#### 根因分析
账号余额不足，无法支付本次API调用的费用。
#### 解决方案
在API开放平台进行充值，充值完成后即可正常使用。

# 八、核心总结：DeepSeek API基础操作速查表
为了方便后续开发时快速查阅，整理了DeepSeek API基础操作的核心速查表：
| 操作步骤     | 核心命令/操作                                                | 关键说明                     |
| ------------ | ------------------------------------------------------------ | ---------------------------- |
| 安装依赖     | `pip install openai`                                         | 使用国内镜像源可加速安装     |
| 环境变量配置 | 变量名：`DEEPSEEK_API_KEY`                                   | 配置后必须重启编辑器         |
| 初始化客户端 | `OpenAI(api_key=os.getenv("DEEPSEEK_API_KEY"), base_url="https://api.deepseek.com")` | 绝对不要硬编码API Key        |
| 基础调用     | `client.chat.completions.create(model="deepseek-chat", messages=[...])` | messages包含system和user角色 |
| 获取结果     | `response.choices[0].message.content`                        | 一次性返回所有结果           |
| 流式输出     | 设置`stream=True`                                            | 逐字返回结果，体验更好       |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163644578>
