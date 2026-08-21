

# Python全栈入门到实战【AI实战篇 06】基于Streamlit构建AI聊天消息展示模块，第一个AI项目前端交互核心
上一篇《AI实战篇 05》中，我们已经彻底掌握了提示词工程的核心方法，能写出高质量的提示词大幅提升AI输出效果。本篇作为第一个AI项目的第六篇，我们将用**纯Python、无需任何HTML/CSS/JS前端基础**的Streamlit框架，快速构建一个带历史消息展示的基础AI聊天应用，这是第一个AI项目的前端交互核心，也是后续功能扩展的基础。

本文为Python全栈开发者与零基础AI入门者量身打造，全程采用步骤式教学，从环境准备、基础单次聊天、问题解析、历史消息实现，每一个环节都有完整代码、逐行超详细讲解、新手避坑提示，即使是完全没有Web开发经验的同学，也能跟着一步步完成所有操作。

本节核心学习内容：
1. 项目环境准备：工具选型、标准目录结构搭建、核心依赖库安装、API密钥安全配置
2. 第一次代码：实现基础单次聊天功能（页面全局配置、UI元素渲染、DeepSeek API调用）
3. 问题复现与深度解析：Streamlit无状态特性导致的历史消息被覆盖问题
4. 第二次代码：新增3个核心部分（会话状态初始化、历史消息循环展示、新消息持久化），解决历史消息问题
5. 完整代码与全功能测试
6. 核心总结：Streamlit AI聊天应用核心速查表，方便开发时快速查阅

# 一、项目环境准备
本项目全程使用纯Python开发，所有工具和依赖均为Python生态通用组件，零基础也能按照步骤完成。

## 1.1 工具与依赖选型
| 工具/组件    | 推荐版本                       | 核心作用                 | 兼容性说明                                                   |
| ------------ | ------------------------------ | ------------------------ | ------------------------------------------------------------ |
| 代码编辑器   | PyCharm Community/Professional | 代码编写、调试、项目管理 | 任意版本均可，推荐2023及以上版本，提供完善的Python代码提示和断点调试功能 |
| Python解释器 | 3.12.2                         | 运行Python代码           | 最低支持3.8版本，3.10-3.12版本稳定性最佳，与Streamlit和OpenAI SDK兼容性最好 |
| Streamlit    | 最新稳定版                     | 快速构建Web聊天界面      | 1.30.0及以上版本支持完整的聊天组件（`st.chat_input`/`st.chat_message`） |
| OpenAI SDK   | 最新稳定版                     | 对接DeepSeek API         | DeepSeek完全兼容OpenAI接口规范，可直接使用OpenAI SDK调用     |

## 1.2 标准项目目录结构搭建
在开始写代码前，必须先创建标准的项目目录结构，确保后续文件路径完全正确，避免出现图片不显示、文件找不到等低级错误：
```
your_ai_project/          # 项目根文件夹（可自定义名称，如"xiaohui_ai"）
├── ai_chat.py            # 主程序文件（后续所有代码都写在这里）
└── imgs/                 # 静态资源文件夹（必须手动创建，名称不能写错）
    └── logo.png          # 自定义AI助手Logo（建议尺寸：200x200像素，PNG格式）
```

## 1.3 核心依赖库安装步骤
### （1）升级pip（必做，避免安装失败）
打开终端：
- Windows系统：按`Win+R`组合键，输入`cmd`，回车打开命令提示符
- Mac/Linux系统：直接打开终端应用

执行以下命令，将pip升级到最新版本：
```bash
pip install --upgrade pip
```

### （2）安装核心依赖库
执行以下命令，安装Streamlit库（OpenAI SDK会在后续API调用时自动依赖，若安装失败可单独执行`pip install openai`补充安装）：
```bash
pip install streamlit
```

### （3）验证安装是否成功
分别执行以下两个命令，若终端显示对应的版本号，则说明安装成功：
```bash
# 验证Streamlit安装
streamlit --version
# 验证OpenAI SDK安装
pip show openai
```

## 1.4 DeepSeek API密钥安全配置
### （1）申请DeepSeek API密钥
1. 打开浏览器，访问DeepSeek开发者平台：https://platform.deepseek.com/
2. 使用手机号或邮箱登录你的DeepSeek账号
3. 进入左侧导航栏的「API密钥」页面
4. 点击右上角的「创建新密钥」按钮
5. 复制生成的API密钥（格式：`sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`）

> ⚠️ 重要提示：密钥创建后**必须立即复制保存**，关闭页面后将无法再次查看！

### （2）配置系统环境变量（推荐，安全无泄露风险）
#### Windows系统永久配置
1. 右键点击桌面「此电脑」图标 → 选择「属性」
2. 点击左侧「高级系统设置」选项
3. 在弹出的「系统属性」窗口中，点击右下角「环境变量」按钮
4. 在「用户变量」区域（仅对当前用户生效），点击「新建」按钮
5. 变量名填写：`DEEPSEEK_API_KEY`（必须完全一致，大小写敏感）
6. 变量值填写：你刚才复制的DeepSeek API密钥
7. 依次点击「确定」保存所有设置，**必须重启终端**才能使环境变量生效

#### Mac/Linux系统永久配置
1. 打开终端，执行以下命令编辑配置文件（根据你的shell类型选择）：
   ```bash
   # 如果你用的是bash（Linux系统默认）
   nano ~/.bashrc
   # 如果你用的是zsh（Mac系统默认）
   nano ~/.zshrc
   ```
2. 在文件末尾添加以下内容：
   ```bash
   export DEEPSEEK_API_KEY="你的DeepSeek API密钥"
   ```
3. 按`Ctrl+O`组合键保存文件，按`Ctrl+X`组合键退出编辑器
4. 执行以下命令，使配置立即生效：
   ```bash
   source ~/.bashrc  # 或 source ~/.zshrc
   ```

# 二、第一次代码：实现基础单次聊天功能
我们先从最简单的基础单次聊天功能开始，快速搭建一个能“用户输入→API调用→AI回答”的最小可用产品。

## 2.1 完整代码
在PyCharm中新建一个Python文件，命名为`ai_chat.py`（可自定义名称），将以下代码完整复制到文件中：
```python
import streamlit as st
import os
from openai import OpenAI
# 设置页面的配置项
st.set_page_config(
    page_title="小辉AI",# 标题
    page_icon="imgs/logo.png",# 图标
    layout="wide",# 布局（占页面全部）
    initial_sidebar_state="expanded",# 侧边栏状态
    menu_items={}
)
# 页面的大标题
st.title("小辉AI")
# 设置logo图片
st.logo("imgs/logo.png",size="large")
# 创建OpenAI客户端
client = OpenAI(
    api_key=os.environ.get('DEEPSEEK_API_KEY'),
    base_url="https://api.deepseek.com"
)
# 定义系统提示词
system_prompt = "我是一名Python编程老师，我的名字叫做小辉辉，有10年的教学经验"
# 消息输入框
prompt = st.chat_input("请输入您的问题")
# prompt获取到用户输入的提示词
if prompt:
    st.chat_message("user").write(prompt)
    # 调用OpenAI API
    response = client.chat.completions.create(
        model="deepseek-chat",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": prompt},
        ],
        stream=False
    )
    st.chat_message("assistant").write(response.choices[0].message.content)
    print(response.choices[0].message.content)
```

## 2.2 代码模块详细说明
| 代码模块       | 核心作用                                        | 关键注意事项                                                 |
| -------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| 导入库模块     | 导入项目所需的Streamlit、os和OpenAI库           | 必须确保所有库都已正确安装，否则会报错                       |
| 页面配置模块   | 通过`st.set_page_config()`设置页面的全局样式    | 这行代码必须放在所有Streamlit代码的最前面，否则会抛出异常    |
| UI元素模块     | 渲染页面的主标题和左上角的Logo                  | Logo路径必须与项目目录结构一致，否则会显示为空白             |
| API客户端模块  | 创建OpenAI客户端实例，用于后续调用DeepSeek API  | 必须正确填写`base_url`为DeepSeek的地址，否则会调用OpenAI的官方接口 |
| 系统提示词模块 | 定义AI的身份、能力和回答风格                    | 修改这里的内容，就能让AI变成不同的角色（如英语老师、文案助手等） |
| 聊天交互模块   | 创建聊天输入框，接收用户输入，调用API并显示回答 | `st.chat_input()`会在页面底部创建输入框，用户提交后返回输入的字符串 |

## 2.3 第一次代码运行与测试
### （1）启动Streamlit服务
1. 在PyCharm中打开底部的「Terminal」终端窗口
2. 确认终端当前路径为项目根目录（即`ai_chat.py`所在的文件夹）
3. 执行以下命令启动Streamlit服务，将`文件名称`替换为你的Python文件名（如`ai_chat.py`）：
   ```bash
   streamlit run .\文件名称
   ```
4. 命令执行后，终端会输出类似以下内容，同时**自动打开默认浏览器**，进入AI聊天页面：
   ```
   You can now view your Streamlit app in your browser.
   Local URL: http://localhost:8501
   Network URL: http://192.168.1.100:8501
   ```

### （2）基础功能测试
1. 在浏览器打开的聊天页面中，输入一个Python相关的问题，例如：
   > Python中列表和元组的区别是什么？
2. 点击输入框右侧的蓝色发送按钮，或直接按回车键
3. 等待2-3秒，页面会显示AI的回答，效果如下：
  ![在这里插入图片](https://i-blog.csdnimg.cn/direct/4508b648a40e447083d094f3f0f84372.png#pic_center)
4. 同时，终端会打印出AI的回答内容，方便调试和排查问题。

# 三、问题发现：历史会话消息被覆盖
第一次代码运行成功后，我们会发现一个严重的问题：**历史会话消息会被覆盖**，每次只能看到最新的一条对话。

## 3.1 问题复现步骤
1. 在第一次代码运行的页面中，输入第一条问题并发送，等待AI回答
2. 输入第二条问题并发送，例如：
   > Python中如何定义函数？
3. 观察页面变化：**第一条用户消息和对应的AI回答会完全消失，页面上只能看到最新的第二条对话**

## 3.2 问题根源深度解析
这个问题是由Streamlit的**核心运行机制**决定的：
- Streamlit是**无状态**的Web框架，每次用户与页面进行交互（发送消息、点击按钮、滑动滑块等）时，都会**从上到下重新运行整个Python脚本**
- 所有普通变量（如`prompt`、`response`等）都会被重新初始化，之前的值会被覆盖
- 之前渲染的页面元素也会被全部清除，然后重新渲染当前脚本的输出
- 因此，历史聊天消息无法被保留，每次只能看到最新的一条对话

## 3.3 解决方案确定
Streamlit专门提供了`st.session_state`会话状态工具，用于解决无状态问题：
- `st.session_state`是一个字典类型的对象，它的生命周期与用户的浏览器会话一致
- 页面刷新、用户交互时，`st.session_state`中的数据不会丢失
- 不同用户的`st.session_state`是相互隔离的，不会互相干扰
- 关闭浏览器标签页后，会话状态会自动清除

我们可以将所有历史聊天消息存储在`st.session_state`中，每次页面重新运行时，先从`st.session_state`中读取历史消息并重新渲染，从而解决消息被覆盖的问题。

# 四、第二次代码：新增部分单独展示与讲解
对比第一次代码，第二次代码主要新增了**3个核心部分**，用于解决历史消息被覆盖的问题。以下是新增部分的单独代码展示和超详细作用讲解。

## 4.1 新增部分1：会话状态初始化
### 新增代码展示
```python
# 初始化聊天信息
if "messages" not in st.session_state:
    st.session_state.messages = []# 创建一个列表，用于存储聊天信息, 默认值为空
```

### 逐行超详细讲解
- `# 初始化聊天信息`：这是单行注释，用于说明这段代码的整体功能，方便后续自己或他人阅读、维护代码。
- `if "messages" not in st.session_state:`：这是整段代码最关键的一行，作用是进行条件判断，检查`st.session_state`这个Streamlit提供的会话状态字典中，是否已经存在名为`"messages"`的键。`st.session_state`是专门用来存储整个浏览器会话期间数据的容器，不同用户的会话状态相互隔离。这个判断的核心意义是**仅在用户第一次打开页面时执行初始化操作**，如果不加这个判断，每次页面刷新、用户发送消息等交互时，都会重新创建一个空列表，之前存储的所有历史消息都会被清空。
- `st.session_state.messages = []`：当条件判断为真（即用户第一次打开页面）时，在`st.session_state`中创建一个名为`messages`的键，并将其值设置为一个空列表。选择用列表来存储消息是最佳方案，因为列表是有序的数据结构，可以严格按照对话的时间顺序依次存储消息。空列表表示用户刚打开页面，还没有产生任何聊天记录，这个列表会在整个浏览器会话期间一直存在，用于存储所有的历史聊天消息。
- `# 创建一个列表，用于存储聊天信息, 默认值为空`：这是行内注释，专门解释上一行代码的具体作用，说明为什么要创建这个空列表。

### 执行时机
这段代码**仅在以下两种情况下执行**：
1. 用户**第一次**在浏览器中打开这个页面
2. 用户关闭浏览器标签页后，**重新打开**页面（旧的会话结束，新的会话开始）

在用户发送消息、刷新页面等后续交互中，由于`"messages"`键已经存在，这段代码**不会再执行**，从而避免了历史数据被清空。

## 4.2 新增部分2：历史消息循环展示
### 新增代码展示
```python
# 将聊天信息显示在页面上
for message in st.session_state.messages:
    if message["role"] == "user":
        st.chat_message("user").write(message["content"])
    else:
        st.chat_message("assistant").write(message["content"])
```

### 逐行超详细讲解
- `# 将聊天信息显示在页面上`：单行注释，说明这段代码的功能是把存储的历史消息渲染到页面上。
- `for message in st.session_state.messages:`：这是一个for循环，作用是依次遍历`st.session_state.messages`列表中的每一个元素，也就是每一条历史消息，并将当前遍历到的消息赋值给`message`变量。由于列表是有序的，遍历的顺序就是消息产生的时间顺序，最早的消息会最先被遍历到，最新的消息会最后被遍历到。每次页面重新运行时，这个循环都会执行一次，把所有历史消息重新渲染一遍。
- `if message["role"] == "user":`：这是条件判断语句，用于检查当前这条消息的`"role"`字段的值是不是`"user"`。我们存储的每一条消息都是一个字典，格式为`{"role": "角色", "content": "消息内容"}`，其中`"role"`字段用于区分消息是用户发送的还是AI发送的。我们需要根据不同的角色，用不同的样式来显示消息，用户消息的头像在右侧，AI消息的头像在左侧。
- `st.chat_message("user").write(message["content"])`：如果判断当前消息是用户发送的，就执行这行代码。其中`st.chat_message("user")`会创建一个用户侧的消息气泡，自带右侧头像的样式；`.write(message["content"])`会把消息字典中`"content"`字段对应的文本内容，写入到这个消息气泡中显示出来。
- `else:`：这是条件判断的分支，如果`"role"`字段的值不是`"user"`，那么就一定是`"assistant"`（也就是AI发送的消息），因为我们的消息列表中只存储这两种角色的消息。
- `st.chat_message("assistant").write(message["content"])`：如果判断当前消息是AI发送的，就执行这行代码。逻辑和渲染用户消息完全一致，只是把角色换成了`"assistant"`，会创建一个左侧带AI头像的消息气泡，并写入AI的回答内容。

### 执行时机
这段代码**在每次页面重新运行时都会执行**，包括：
1. 用户第一次打开页面（此时列表为空，循环不执行任何操作）
2. 用户发送一条新消息后
3. 用户刷新浏览器页面后
4. 用户点击页面上的任何按钮后

### 代码位置的重要性
这段代码**必须放在`st.chat_input()`（消息输入框）之前**，原因如下：
1. Streamlit页面是从上到下依次执行代码的
2. 先执行循环，把所有历史消息显示出来
3. 再显示输入框，处理新的用户输入
4. 如果位置放反了，历史消息会显示在输入框的下方，严重影响用户体验

## 4.3 新增部分3：新消息持久化
### 新增代码展示1（用户消息持久化）
```python
# 将用户输入的提示词添加到session_state.messages中
st.session_state.messages.append({"role": "user", "content": prompt})
```

### 新增代码展示2（AI消息持久化）
```python
# 将OpenAI API返回的答案添加到session_state.messages中
st.session_state.messages.append({"role": "assistant", "content": response.choices[0].message.content})
```

### 代码作用讲解
这两部分代码是实现历史消息存储的**关键操作**，分别在用户发送消息和AI返回回答后执行，核心作用如下：
1. **用户消息持久化**：当用户输入新消息并提交后，立即将消息封装成标准的字典格式（`{"role": "user", "content": prompt}`），然后通过`append()`方法追加到`st.session_state.messages`列表的末尾。
2. **AI消息持久化**：当DeepSeek API返回回答后，同样将回答封装成标准的字典格式（`{"role": "assistant", "content": ai_answer}`），追加到`st.session_state.messages`列表的末尾。
3. **保持消息顺序**：严格按照对话的时间顺序依次追加消息，确保列表中的消息顺序与实际对话的时间顺序完全一致。
4. **为下一次渲染做准备**：新消息追加到列表后，下次页面重新运行时，会被上面的「历史消息循环展示」部分读取并显示，从而实现完整的会话记录。

## 4.4 三部分代码的配合执行流程
初始化、循环展示、新消息持久化这三部分代码是**前后呼应、紧密配合**的，它们的完整执行流程如下：
### 第一次打开页面
1. 执行「会话状态初始化」：`st.session_state.messages`被创建为空列表
2. 执行「历史消息循环展示」：列表为空，循环不执行，页面上没有任何消息
3. 显示输入框，等待用户输入

### 用户发送第一条消息后
1. 页面重新运行
2. 执行「会话状态初始化」：`"messages"`键已存在，**跳过**初始化
3. 执行「历史消息循环展示」：此时列表为空，循环不执行
4. 显示输入框，接收用户输入
5. 渲染当前用户消息
6. 执行「用户消息持久化」：将第一条用户消息追加到列表
7. 调用API获取AI回答
8. 渲染AI回答
9. 执行「AI消息持久化」：将第一条AI回答追加到列表

### 用户发送第二条消息后
1. 页面重新运行
2. 执行「会话状态初始化」：`"messages"`键已存在，**跳过**初始化
3. 执行「历史消息循环展示」：此时列表中已有第一条用户消息和第一条AI回答，循环执行，依次显示这两条消息
4. 显示输入框，接收用户输入
5. 渲染当前用户消息
6. 执行「用户消息持久化」：将第二条用户消息追加到列表
7. 调用API获取AI回答
8. 渲染AI回答
9. 执行「AI消息持久化」：将第二条AI回答追加到列表

# 五、第二次代码：完整代码与测试
## 5.1 完整代码
将`ai_chat.py`中的代码全部替换为以下内容（包含第一次代码的所有内容和第二次新增的3个部分）：
```python
import streamlit as st
import os
from openai import OpenAI
# 设置页面的配置项
st.set_page_config(
    page_title="小辉AI",# 标题
    page_icon="imgs/logo.png",# 图标
    layout="wide",# 布局（占页面全部）
    initial_sidebar_state="expanded",# 侧边栏状态
    menu_items={}
)
# 页面的大标题
st.title("小辉AI")
# 设置logo图片
st.logo("imgs/logo.png",size="large")
# 创建OpenAI客户端
client = OpenAI(
    api_key=os.environ.get('DEEPSEEK_API_KEY'),
    base_url="https://api.deepseek.com"
)
# 初始化聊天信息
if "messages" not in st.session_state:
    st.session_state.messages = []# 创建一个列表，用于存储聊天信息, 默认值为空
# 定义系统提示词
system_prompt = "我是一名Python编程老师，我的名字叫做小辉辉，有10年的教学经验"
# 将聊天信息显示在页面上
for message in st.session_state.messages:
    if message["role"] == "user":
        st.chat_message("user").write(message["content"])
    else:
        st.chat_message("assistant").write(message["content"])
# 消息输入框
prompt = st.chat_input("请输入您的问题")
# prompt获取到用户输入的提示词
if prompt:
    st.chat_message("user").write(prompt)
    # 将用户输入的提示词添加到session_state.messages中
    st.session_state.messages.append({"role": "user", "content": prompt})
    # 调用OpenAI API
    response = client.chat.completions.create(
        model="deepseek-chat",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": prompt},
        ],
        stream=False
    )
    st.chat_message("assistant").write(response.choices[0].message.content)
    print(response.choices[0].message.content)
    # 将OpenAI API返回的答案添加到session_state.messages中
    st.session_state.messages.append({"role": "assistant", "content": response.choices[0].message.content})
```

## 5.2 第二次代码运行与测试
### （1）重启Streamlit服务
1. 在终端中按`Ctrl+C`组合键，终止之前的Streamlit服务
2. 重新执行启动命令：
   ```bash
   streamlit run .\ai_chat.py
   ```

### （2）历史消息展示功能测试
1. 输入第一条问题：“我有12个苹果，4个人怎么均分呢？”，点击发送
2. 等待AI回答，此时第一条对话会显示在页面上
3. 输入第二条追问：“那两个人呢？”，点击发送
4. 观察页面变化：**第一条和第二条对话都会保留在页面上，不会被覆盖**，效果如下：
  ![在这里插入图片](https://i-blog.csdnimg.cn/direct/d6c46c3b3c9241d093e2adf3b4ffe14d.png#pic_center)

  ![在这里插入图片](https://i-blog.csdnimg.cn/direct/f82ddffa3e8b40a79a55c59c0d58d356.png#pic_center)
5. 继续输入第三条、第四条问题，所有历史对话都会依次显示在页面上，但是目前我们的代码还无法让ai可以做到会话记忆，我们下篇文章继续~

# 六、项目总结
本文完整实现了基于Streamlit构建DeepSeek AI聊天消息展示模块的全过程，核心完成了以下内容：
1. 完成了项目环境的全面配置，包括工具选型、依赖库安装、API密钥申请与环境变量配置
2. 编写了第一次代码，实现了最基础的“用户输入→API调用→AI回答”的单次聊天功能
3. 复现并深度解析了Streamlit无状态特性导致的历史消息被覆盖问题
4. 确定了使用`st.session_state`解决问题的方案，并将新增的3个核心部分单独展示和逐行超详细讲解
5. 梳理了三部分代码的配合执行流程，清晰展示了历史消息从存储到展示的完整逻辑
6. 编写了第二次代码，成功实现了历史聊天消息的持久化存储和展示

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163720894>
