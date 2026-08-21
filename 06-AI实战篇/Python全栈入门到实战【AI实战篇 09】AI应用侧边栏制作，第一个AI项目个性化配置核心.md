

# Python全栈入门到实战【AI实战篇 09】AI应用侧边栏制作，第一个AI项目个性化配置核心
上一篇《AI实战篇 08》中，我们已经实现了AI智能助手的流式输出功能，解决了非流式输出的等待卡顿问题，让交互体验媲美主流AI产品。本篇作为第一个AI项目的第九篇，我们将开发**侧边栏个性化配置功能**，实现AI角色和技能的自定义切换，让用户无需修改代码，就能根据不同场景（编程教学、英语翻译、文案创作等）快速调整AI的身份和能力，大幅提升应用的实用性和灵活性。

本文为Python全栈开发者与零基础AI入门者量身打造，全程采用步骤式教学，从Streamlit侧边栏组件基础、会话状态持久化、UI界面开发到动态系统提示词关联，每一个环节都有完整代码、逐行讲解和效果验证，即使是完全没有Web开发经验的同学，也能跟着一步步完成侧边栏功能，打造属于自己的个性化AI助手。

本节核心学习内容：
1. 前置知识：Streamlit Sidebar组件的两种使用方式与最佳实践
2. 会话状态初始化：侧边栏配置的持久化存储，刷新页面不丢失
3. 侧边栏UI开发：角色输入框、技能输入框的实现与实时同步
4. 动态系统提示词：将侧边栏配置与AI响应关联，实现实时生效
5. 完整代码整合：侧边栏+历史消息+会话记忆+流式输出三合一
6. 效果验证：自定义角色与技能的实际使用测试
7. 关键注意事项：输入同步时机、空值处理、布局优化等避坑点
8. 核心总结：侧边栏开发核心速查表，方便开发时快速查阅

# 一、开发背景
在前序开发中，我们已经完整实现了AI聊天应用的**消息展示、会话记忆、流式输出**三大核心功能。但目前的AI角色和能力是硬编码在代码中的，用户如果想切换AI的身份，必须修改代码并重启服务，非常不方便。

为了解决这个问题，我们将开发侧边栏配置功能，让用户可以在浏览器中实时修改AI的角色和技能，所有配置会自动保存到会话状态中，刷新页面也不会丢失，真正实现AI助手的个性化定制。

# 二、前置知识：Streamlit Sidebar组件
Streamlit的`sidebar`组件用于创建页面左侧的侧边栏区域，支持嵌入所有Streamlit原生交互元素（输入框、按钮、下拉框、滑块等），有两种标准使用方式，下面详细对比并说明推荐方案：

## 方式1：直接调用法
```python
# 直接通过sidebar前缀调用元素
st.sidebar.subheader("助手信息")
st.sidebar.text_input("AI角色")
st.sidebar.text_area("角色技能")
```
- 优点：写法极简，适合侧边栏只有2-3个简单元素的场景
- 缺点：元素较多时代码层级混乱，可读性差，难以维护

## 方式2：上下文管理器法（强烈推荐）
```python
# 通过with语句包裹所有侧边栏元素
with st.sidebar:
    st.subheader("助手信息")
    st.text_input("AI角色")
    st.text_area("角色技能")
```
- 优点：代码结构清晰，所有侧边栏元素统一在一个代码块中，便于阅读和维护
- 特点：支持复杂的元素嵌套和布局，符合Python上下文管理的编码规范
- 适用场景：所有侧边栏开发场景，尤其是元素较多、逻辑较复杂的情况

# 三、侧边栏开发详细步骤
## 3.1 初始化侧边栏配置的会话状态
侧边栏的配置（角色、技能）需要在页面刷新、用户交互后保留，因此必须使用`st.session_state`进行存储。我们需要在页面最开始的位置，初始化两个会话状态变量：
```python
# 初始化AI角色：默认值为"编程老师"
# 作用：用户第一次打开页面时，显示默认角色；后续修改后，值会被更新
if "role" not in st.session_state:
    st.session_state.role = "编程老师"
# 初始化AI技能：默认值为"编程，教学"
# 作用：用户第一次打开页面时，显示默认技能；后续修改后，值会被更新
if "skills" not in st.session_state:
    st.session_state.skills = "编程，教学"
```

### 关键说明
- 必须添加`if "xxx" not in st.session_state:`判断：如果不加这个判断，每次页面刷新时，变量都会被重新初始化为默认值，用户的自定义配置会丢失
- 变量名建议语义化：使用`role`和`skills`作为键名，便于后续代码中引用和维护
- 默认值设置合理：设置用户最常用的角色和技能作为默认值，提升首次使用体验

## 3.2 实现侧边栏UI界面
使用`with st.sidebar:`上下文管理器，创建侧边栏的标题、角色输入框和技能输入框，并实现**配置回显**和**实时同步**功能：
```python
# 侧边栏部分
with st.sidebar:
    st.subheader("助手信息")
    # 创建一个输入框，用于输入角色
    set_up_roles = st.text_input("角色",placeholder="例如：编程老师等等",value="编程老师")
    # 将用户输入的角色添加到session_state.role中
    if set_up_roles.strip() != "":
        st.session_state.role = set_up_roles
    # 创建一个输入框，用于输入该角色的技能
    set_up_skills = st.text_area("技能",placeholder="例如：编程，数学等等",value="编程,教学")
    # 将用户输入的技能添加到session_state.skills中
    if set_up_skills.strip() != "":
        st.session_state.skills = set_up_skills
```

### 逐行参数详解
- `label`：输入框的标签，必须清晰明确，让用户知道这个输入框是用来做什么的
- `placeholder`：输入框为空时的灰色提示文字，用于引导用户正确输入格式
- `value`：输入框的初始值，这里绑定`st.session_state`中的变量，实现“修改后刷新页面不丢失”的效果
- `height`：仅`text_area`组件有，用于设置多行输入框的高度，提升长文本输入体验

### 核心逻辑说明
- **配置回显**：通过`value=st.session_state.xxx`，将session_state中存储的配置值显示在输入框中，用户刷新页面后，之前输入的内容不会消失
- **实时同步**：通过`if set_up_xxx: st.session_state.xxx = set_up_xxx`，当用户在输入框中修改内容并按下回车时，session_state中的值会立即更新，后续的AI响应会使用最新的配置

## 3.3 构建动态系统提示词（关联侧边栏与AI响应）
侧边栏的配置最终要作用于AI的响应，这是通过**动态生成系统提示词**实现的。我们将session_state中存储的角色和技能，通过f-string拼接成系统提示词，传递给DeepSeek API：
```python
# 动态生成系统提示词
# 作用：根据侧边栏配置的角色和技能，实时定义AI的身份和能力
system_prompt = f"我是一名资深的{st.session_state.role}，我的名字叫做小辉辉，我的技能是{st.session_state.skills}。"
```

### 关键说明
- 系统提示词必须写在**侧边栏代码之后、API调用之前**：这样才能保证每次调用API时，使用的是最新的侧边栏配置
- 添加约束条件：在系统提示词末尾加上“严格按照身份回答”“不要回答超出技能范围的内容”，可以有效避免AI回答跑偏
- 格式灵活：可以根据需求调整系统提示词的格式，例如增加“回答风格”“回答长度”等要求

# 四、完整代码汇总
以下是整合了侧边栏功能、消息展示、会话记忆和流式输出的完整代码，可直接复制使用：
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
# 初始化角色
if "role" not in st.session_state:
    st.session_state.role = "编程老师"# 创建一个变量，用于存储角色, 默认值为编程老师
# 初始化技能
if "skills" not in st.session_state:
    st.session_state.skills = "编程，教学"# 创建一个变量，用于存储技能, 默认值为编程
# 将聊天信息显示在页面上
for message in st.session_state.messages:
    if message["role"] == "user":
        st.chat_message("user").write(message["content"])
    else:
        st.chat_message("assistant").write(message["content"])
# 侧边栏部分
with st.sidebar:
    st.subheader("助手信息")
    # 创建一个输入框，用于输入角色
    set_up_roles = st.text_input("角色",placeholder="例如：编程老师等等",value="编程老师")
    # 将用户输入的角色添加到session_state.role中
    if set_up_roles.strip() != "":
        st.session_state.role = set_up_roles
    # 创建一个输入框，用于输入该角色的技能
    set_up_skills = st.text_area("技能",placeholder="例如：编程，数学等等",value="编程,教学")
    # 将用户输入的技能添加到session_state.skills中
    if set_up_skills.strip() != "":
        st.session_state.skills = set_up_skills
# 定义系统提示词
system_prompt = f"我是一名资深的{st.session_state.role}，我的名字叫做小辉辉，我的技能是{st.session_state.skills}"
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
            *st.session_state.messages,# 获取session_state.messages中的所有消息
        ],
        stream=True
    )
    # 创建一个空变量，用于存储流式输出的答案
    response_message = st.empty()
    # 初始化流式输出变量
    output = ""
    # 流式输出代码
    for chunk in response:
        chunk_str = chunk.choices[0].delta.content# 获取当前chunk的content
        output += chunk_str# 将当前chunk的content添加到output中
        response_message.chat_message("assistant").write(output)# 将output往空组件中进行显示
    # 将OpenAI API返回的答案添加到session_state.messages中
    st.session_state.messages.append({"role": "assistant", "content": output})
    # 非流式输出代码
    # st.chat_message("assistant").write(response.choices[0].message.content)
    # 将OpenAI API返回的答案添加到session_state.messages中
    # st.session_state.messages.append({"role": "assistant", "content": response.choices[0].message.content})
```

# 五、侧边栏与核心聊天逻辑的关联
侧边栏配置完成后，无需修改任何聊天交互逻辑，只需在调用DeepSeek API时，传入上面动态生成的`system_prompt`即可：
```python
# 调用DeepSeek API（流式输出）
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},  # 传入动态生成的系统提示词
        *st.session_state.messages,  # 传入历史会话消息
    ],
    stream=True
)
```

# 六、效果展示与验证
运行代码后，浏览器页面效果如下：

![在这里插入图片](https://i-blog.csdnimg.cn/direct/badeffdcda404bb783ec1bbb7f33615c.png#pic_center)

## 效果验证步骤
1. 在侧边栏的「AI角色」输入框中输入“英语老师”，按下回车
2. 在「角色技能」输入框中输入“初中英语语法、英语口语、英语作文批改”，按下回车
3. 在聊天输入框中输入“帮我批改一下这篇英语作文”
4. 观察AI的回复：会以英语老师的身份，按照配置的技能进行回答
5. 刷新浏览器页面：侧边栏的配置和聊天记录都会保留，不会丢失

# 七、关键注意事项
1. **输入框同步时机**：`st.text_input`和`st.text_area`组件只有在用户按下回车或点击输入框外的区域时，才会更新值并同步到session_state
2. **空值处理**：如果用户清空输入框并按下回车，`set_up_roles`或`set_up_skills`会变为空字符串，此时session_state中的值也会被清空。可以添加判断避免这种情况：
   ```python
   if set_up_roles.strip() != "":
       st.session_state.role = set_up_roles
   ```
3. **系统提示词优先级**：系统提示词会覆盖AI的默认行为，因此配置越具体，AI的回答越符合预期
4. **侧边栏布局优化**：可以使用`st.divider()`添加分割线，区分不同的配置区域，提升视觉体验：
   ```python
   with st.sidebar:
       st.subheader("助手信息配置")
       st.divider()  # 添加分割线
       st.text_input("AI角色")
       st.text_area("角色技能")
   ```

# 八、核心总结：侧边栏开发核心速查表
为了方便后续开发时快速查阅，整理了侧边栏开发的核心速查表：

| 核心环节   | 关键操作                           | 核心要点                            |
| ---------- | ---------------------------------- | ----------------------------------- |
| 组件选择   | 使用`with st.sidebar:`上下文管理器 | 代码结构清晰，便于维护              |
| 配置持久化 | 用`st.session_state`存储角色和技能 | 必须添加初始化判断，避免刷新丢失    |
| UI开发     | `st.text_input`+`st.text_area`     | 配置placeholder引导用户输入         |
| 实时同步   | 输入框值变化时更新session_state    | 按下回车或点击空白处生效            |
| 关联AI响应 | 动态生成系统提示词                 | 必须写在侧边栏代码之后、API调用之前 |
| 空值处理   | 添加`strip() != ""`判断            | 避免空字符串覆盖有效配置            |
| 布局优化   | 使用`st.divider()`添加分割线       | 区分不同配置区域，提升视觉体验      |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163831675>
