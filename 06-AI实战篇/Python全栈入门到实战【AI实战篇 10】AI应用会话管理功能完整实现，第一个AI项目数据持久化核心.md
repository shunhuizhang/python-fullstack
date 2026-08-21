

# Python全栈入门到实战【AI实战篇 10】AI应用会话管理功能完整实现，第一个AI项目数据持久化核心
上一篇《AI实战篇 09》中，我们已经完成了AI应用侧边栏的开发，实现了AI角色和技能的自定义切换，让智能助手可以适配不同的使用场景。本篇作为第一个AI项目的第十篇，我们将解决**数据持久化**这个核心痛点——目前所有会话数据仅存在于浏览器内存中，关闭标签页后永久丢失，且无法同时管理多个独立的聊天主题。我们将从零开始实现一套完整的本地会话管理系统，支持自动保存、新建、加载、删除会话，让我们的AI应用真正具备实用价值。

本文为Python全栈开发者与零基础AI入门者量身打造，全程采用分层设计思路，从需求分析、架构设计到分步开发、完整测试，每一个环节都有详细讲解和可直接复制的代码，即使是完全没有后端开发经验的同学，也能跟着一步步实现完整的会话管理功能，无需任何额外的数据库或后端服务。

本节核心学习内容：
1. 需求分析：明确会话管理的5个核心功能需求
2. 整体架构：数据层、状态层、逻辑层、界面层、交互层五层设计
3. 分步开发：从依赖导入、状态初始化到保存、新建、加载、删除功能的完整实现
4. 自动保存：用户发送消息后自动持久化数据，无需手动操作
5. 完整代码整合：会话管理+历史消息+会话记忆+流式输出+自定义角色全功能整合
6. 全面测试：覆盖所有功能场景的测试步骤与验证标准
7. 关键注意事项：文件权限、多用户场景、数据备份等核心避坑点
8. 核心总结：会话管理实现核心速查表，方便开发时快速查阅

# 一、背景与需求分析
在前期的AI应用开发中，我们已经实现了**消息展示、会话记忆、流式输出、自定义角色与技能**四大核心功能，基本满足了单会话的聊天需求。但随着使用场景的深入，我们发现了一个严重影响用户体验的核心痛点：**所有会话数据仅存在于浏览器内存中，关闭标签页后数据永久丢失，且无法同时管理多个独立的聊天主题**。

为了解决这个问题，我们需要实现一套完整的本地会话管理系统，具体需满足以下5个核心需求：
1. **自动保存当前会话**：用户发送消息后，自动将聊天记录、角色配置、技能配置保存到本地文件，无需手动操作
2. **手动新建会话**：点击按钮即可创建一个全新的空白会话，同时自动保存当前正在进行的会话
3. **历史会话列表展示**：在侧边栏展示所有保存过的会话，按时间倒序排列，最新的会话在最上方
4. **加载历史会话**：点击任意历史会话，即可恢复该会话的所有聊天记录和配置，继续之前的对话
5. **删除无用会话**：一键删除不需要的会话，清理本地存储空间

## 技术栈前置说明
本次实现完全基于Python生态，无需任何额外的后端服务或数据库，仅依赖以下5个库：
- `streamlit`：快速构建Web界面，提供按钮、输入框、分栏等所有UI组件
- `json`：Python内置库，用于将Python对象序列化为JSON字符串，保存到本地文件
- `os`：Python内置库，用于创建目录、检查文件是否存在、删除文件等文件系统操作
- `datetime`：Python内置库，用于生成基于时间戳的唯一会话ID
- `openai`：用于调用DeepSeek大模型API，实现聊天功能（非会话管理核心依赖）

# 二、整体开发流程总览
在开始写代码之前，我们先梳理清楚整个会话管理功能的开发步骤，让读者清晰地知道每一步该做什么：
1. **前置准备**：确保已经完成了前面的所有功能，代码可以正常运行
2. **导入新增依赖**：导入本次功能需要的`datetime`和`json`库
3. **初始化会话ID状态**：在全局状态中添加`session_id`变量，用于标识每个会话
4. **实现保存会话功能**：编写`save_session()`函数，将当前会话状态保存到本地JSON文件
5. **实现新建会话功能**：在侧边栏添加"新建会话"按钮，绑定保存旧会话和创建新会话的逻辑
6. **实现历史会话列表展示**：编写`load_sessions()`函数，读取所有会话文件并在侧边栏展示
7. **实现加载会话功能**：编写`load_session()`函数，读取指定会话文件并恢复状态
8. **实现删除会话功能**：编写`delete_session()`函数，删除指定会话文件并处理当前会话
9. **整合自动保存逻辑**：在用户发送消息后自动调用`save_session()`函数
10. **全面测试**：测试所有功能是否正常运行，排查可能的问题

# 三、核心实现思路总览
在开始写代码之前，我们需要先梳理清楚实现会话管理功能的整体思路。这套思路分为**数据层、状态层、逻辑层、界面层、交互层**五个层次，层层递进，形成完整的闭环：

## 3.1 数据层思路：选择合适的持久化方案
会话管理的核心是**数据持久化**，即把内存中的数据保存到本地硬盘，关闭浏览器后数据不丢失。我们需要选择一个简单、可靠、易维护的方案：
- **数据格式选择**：选择JSON格式，因为它是轻量级、易读、易解析的文本格式，几乎所有编程语言都支持，且人类可以直接打开查看
- **存储位置选择**：在项目根目录下创建一个专门的`sessions`文件夹，所有会话文件都存放在这里，便于管理和备份
- **文件命名选择**：使用基于时间戳的唯一ID作为文件名（如`2026-04-11_14-30-25.json`），保证文件名的唯一性，同时可以通过文件名直观地看到会话的创建时间

## 3.2 状态层思路：利用Streamlit的会话状态管理
Streamlit是无状态的Web框架，每次用户交互都会重新运行整个脚本。因此，我们需要利用`st.session_state`来管理跨交互的全局状态：
- **状态变量设计**：设计4个核心状态变量：
  - `messages`：存储当前会话的所有聊天记录
  - `role`：存储当前会话的AI角色配置
  - `skills`：存储当前会话的AI技能配置
  - `session_id`：存储当前会话的唯一标识
- **状态初始化**：在代码最开始的位置，检查这4个变量是否存在，如果不存在则初始化为默认值
- **状态切换**：当用户新建、加载或删除会话时，更新对应的状态变量，然后刷新页面使状态生效

## 3.3 逻辑层思路：功能模块化，职责单一化
为了让代码结构清晰、易于维护和调试，我们将不同的功能拆分成独立的函数，每个函数只负责一件事：
- `save_session()`：负责将当前会话的状态保存到本地JSON文件
- `load_sessions()`：负责读取`sessions`目录，返回所有历史会话的ID列表
- `load_session(session_id)`：负责读取指定ID的JSON文件，恢复会话状态
- `delete_session(session_id)`：负责删除指定ID的JSON文件，并处理当前会话被删除的情况

## 3.4 界面层思路：侧边栏集中管理，主区域专注聊天
为了提升用户体验，我们将界面分为两个区域：
- **侧边栏区域**：作为"AI控制面板"，集中放置所有会话管理功能和AI配置功能，包括：
  - "新建会话"按钮
  - 历史会话列表（包含加载和删除按钮）
  - AI角色和技能配置输入框
- **主区域**：专注于聊天交互，包括：
  - 当前会话ID显示
  - 历史聊天记录展示
  - 聊天输入框

## 3.5 交互层思路：事件驱动，实时反馈
用户的所有操作都是事件驱动的，我们需要为每个按钮绑定对应的逻辑：
- 点击"新建会话"按钮：先保存当前会话，再重置状态，最后刷新页面
- 点击历史会话的加载按钮：调用加载函数，恢复状态，刷新页面
- 点击历史会话的删除按钮：调用删除函数，删除文件，刷新页面
- 用户发送新消息：展示消息，调用API，展示回复，自动保存会话

# 四、分步开发详细教程
## 4.1 前置准备
确保你已经完成了前面所有功能的开发，并且代码可以正常运行。你的代码应该已经包含以下部分：
- 页面配置和UI元素
- OpenAI客户端初始化
- 聊天记录展示
- 自定义角色和技能配置
- 流式输出功能

如果你的代码还没有这些功能，请先完成前面的文章内容，再继续本次开发。

## 4.2 导入新增的依赖库
在代码的最顶部，导入本次功能需要的两个库：
```python
# 在原有导入语句后面添加这两个导入
from datetime import datetime
import json
```

## 4.3 初始化会话ID状态
在全局状态初始化部分，添加`session_id`变量的初始化：
```python
# 原有状态初始化
if "messages" not in st.session_state:
    st.session_state.messages = []
if "role" not in st.session_state:
    st.session_state.role = "编程老师"
if "skills" not in st.session_state:
    st.session_state.skills = "编程，教学"

# 新增：初始化会话ID
if "session_id" not in st.session_state:
    # 创建一个变量，用于存储会话标识
    # 格式：年-月-日_时-分-秒，例如：2026-04-11_14-30-25
    st.session_state.session_id = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
```

## 4.4 在主页面显示当前会话ID
在页面标题下方，添加当前会话ID的显示，让用户知道自己正在哪个会话中：
```python
# 原有代码
st.title("小辉AI")
st.logo("imgs/logo.png", size="large")

# 新增：显示当前会话ID
st.text(f"当前会话：{st.session_state.session_id}")
```

## 4.5 实现保存会话功能
在所有导入语句的后面，编写`save_session()`函数：
```python
# 保存当前会话
def save_session():
    # 仅当会话ID存在时执行保存操作
    # 防止程序初始化时，会话ID还未生成就执行保存，导致报错
    if st.session_state.session_id:
        # 封装会话核心数据为字典
        # 所有需要持久化的数据都放在这个字典中
        session_data = {
            "session_id": st.session_state.session_id,  # 唯一会话标识，作为文件名
            "messages": st.session_state.messages,      # 所有聊天记录
            "role": st.session_state.role,              # 当前AI的角色配置
            "skills": st.session_state.skills           # 当前AI的技能配置
        }
        # 检查sessions目录是否存在
        # 如果不存在，自动创建该目录，用于存放所有会话文件
        if not os.path.exists("sessions"):
            os.mkdir("sessions")
        # 将数据写入JSON文件
        # 文件名使用会话ID，确保唯一性
        with open(f"sessions/{st.session_state.session_id}.json", "w", encoding="utf-8") as f:
            # json.dump：将Python字典序列化为JSON字符串并写入文件
            # ensure_ascii=False：强制保留中文，否则中文会被转义为\uXXXX格式
            # indent=2：格式化JSON文件，每个层级缩进2个空格，便于人类阅读
            json.dump(session_data, f, ensure_ascii=False, indent=2)
```

## 4.6 实现新建会话功能
在侧边栏部分，添加"新建会话"按钮并绑定逻辑：
```python
# 侧边栏部分
with st.sidebar:
    st.subheader("AI控制面板")
    
    # 新增：新建会话按钮
    if st.button("新建会话", width="stretch", icon="📝"):
        # 如果有聊天信息，则保存
        if st.session_state.messages:
            # 第一步：将当前的会话进行保存
            save_session()
            # 第二步：创建新会话
            st.session_state.messages = []  # 清空session_state.messages
            st.session_state.session_id = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")  # 创建新的会话标识
            st.session_state.role = "编程老师"  # 重置为默认角色
            st.session_state.skills = "编程，教学"  # 重置为默认技能
            save_session()  # 保存新建会话
            st.rerun()  # 重新运行页面

    # 原有代码：助手信息配置
    st.divider()
    st.subheader("助手信息")
    # ... 原有角色和技能输入框代码 ...
```

## 4.7 测试保存和新建会话功能
![在这里插入图片](https://i-blog.csdnimg.cn/direct/446ed8c38aa2415a9780890757716d51.png#pic_center)

![在这里插入图片](https://i-blog.csdnimg.cn/direct/8e43dd44cec94082b1303dc45bc02ba7.png#pic_center)

现在我们可以测试前两个功能是否正常运行：
1. 运行程序：在终端执行`streamlit run ai_chat.py`
2. 在聊天输入框中输入"Python中列表和元组的区别是什么？"，点击发送
3. 等待AI回复完成后，查看项目根目录，会发现自动创建了一个`sessions`文件夹
4. 打开`sessions`文件夹，会看到一个以时间戳命名的JSON文件，这就是刚才保存的会话
5. 点击侧边栏的"新建会话"按钮
6. 页面会刷新，聊天区清空，顶部的"当前会话"标识更新为新的时间戳
7. 再次查看`sessions`文件夹，会看到多了一个新的JSON文件

如果以上步骤都正常，说明保存和新建会话功能已经实现成功。

## 4.8 实现历史会话列表展示
首先编写`load_sessions()`函数，用于读取所有会话文件：
```python
# 加载所有会话标识
def load_sessions():
    # 初始化会话标识列表
    sessions_list = []
    if os.path.exists("sessions"):
        for file in os.listdir("sessions"):
            if file.endswith(".json"):
                file_name = file[:-5]  # 去掉.json后缀
                sessions_list.append(file_name)
    return sessions_list
```

然后在侧边栏的"新建会话"按钮下方，添加历史会话列表的展示：
```python
with st.sidebar:
    st.subheader("AI控制面板")
    if st.button("新建会话", width="stretch", icon="📝"):
        # ... 原有新建会话逻辑 ...
    
    # 新增：历史会话列表
    st.text("历史会话")
    session_list = load_sessions()
    # 对列表进行排序，按时间倒序排列，最新的会话在最上面
    session_list.sort(reverse=True)
    for session in session_list:
        # 分栏：4份宽度用于加载按钮，1份宽度用于删除按钮
        clu1, clu2 = st.columns([4, 1])
        # 创建一个按钮，用于选择会话
        with clu1:
            if st.button(
                session, 
                width="stretch", 
                icon="💬", 
                key=f"load_{session}",
                type="primary" if session == st.session_state.session_id else "secondary"
            ):
                pass  # 后续绑定加载逻辑
        # 创建一个按钮，用于删除会话
        with clu2:
            if st.button("", width="stretch", icon="❌️", key=f"del_{session}"):
                pass  # 后续绑定删除逻辑
    
    # 原有代码：助手信息配置
    st.divider()
    st.subheader("助手信息")
    # ... 原有角色和技能输入框代码 ...
```

## 4.9 测试历史会话列表展示
再次运行程序，你会看到侧边栏的"历史会话"区域已经显示出了我们之前创建的两个会话，当前会话的按钮是蓝色的，其他会话是灰色的。

![在这里插入图片](https://i-blog.csdnimg.cn/direct/7cfc31871e024ac7b46968c4e536d8c7.png#pic_center)

## 4.10 实现加载会话功能
编写`load_session()`函数，用于读取指定会话文件并恢复状态：
```python
# 加载当前会话
def load_session(session_id):
    try:
        if os.path.exists(f"sessions/{session_id}.json"):
            with open(f"sessions/{session_id}.json", "r", encoding="utf-8") as f:
                session_data = json.load(f)
                st.session_state.messages = session_data["messages"]  # 聊天信息
                st.session_state.role = session_data["role"]  # 角色
                st.session_state.skills = session_data["skills"]  # 技能
                st.session_state.session_id = session_id  # 会话标识
    except Exception:
        st.error("会话加载失败")
```

然后绑定加载按钮的点击逻辑：
```python
with clu1:
    if st.button(
        session, 
        width="stretch", 
        icon="💬", 
        key=f"load_{session}",
        type="primary" if session == st.session_state.session_id else "secondary"
    ):
        load_session(session)
        st.rerun()
```

## 4.11 测试加载会话功能
![在这里插入图片](https://i-blog.csdnimg.cn/direct/621a620e7c9047c4a9f9b5f3e31e8f2f.gif#pic_center)

1. 点击历史会话列表中的第一个会话（最新的那个）
2. 页面会刷新，聊天区会显示该会话的所有聊天记录
3. 侧边栏的"角色"和"技能"输入框会自动填充该会话的配置
4. 继续输入新的问题，AI会基于该会话的上下文进行回复

如果以上步骤都正常，说明加载会话功能已经实现成功。

## 4.12 实现删除会话功能
编写`delete_session()`函数，用于删除指定会话文件：
```python
# 删除会话
def delete_session(session_id):
    try:
        if os.path.exists(f"sessions/{session_id}.json"):
            os.remove(f"sessions/{session_id}.json")
            if session_id == st.session_state.session_id:
                st.session_state.messages = []
                st.session_state.session_id = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
                st.session_state.role = "编程老师"
                st.session_state.skills = "编程，教学"
            st.rerun()
    except Exception:
        st.error("会话删除失败")
```

然后绑定删除按钮的点击逻辑：
```python
with clu2:
    if st.button("", width="stretch", icon="❌️", key=f"del_{session}"):
        delete_session(session)
        st.rerun()
```

## 4.13 测试删除会话功能
1. 点击某个会话右侧的叉号按钮
2. 该会话会从历史列表中消失
3. 查看`sessions`文件夹，对应的JSON文件已经被删除
4. 如果删除的是当前正在使用的会话，页面会自动跳转到一个新的空会话

如果以上步骤都正常，说明删除会话功能已经实现成功。

## 4.14 整合自动保存逻辑
最后，在用户发送消息后自动调用`save_session()`函数，确保数据不会丢失：
```python
# 原有代码
st.session_state.messages.append({"role": "assistant", "content": output})
# 新增：自动保存会话
save_session()
```

# 五、完整代码整合
以下是整合了所有功能的完整代码，**完全保留原始代码逻辑，未做任何修改**：
```python
import streamlit as st
import os
from openai import OpenAI
from datetime import datetime
import json

# 保存当前会话
def save_session():
    if st.session_state.session_id:
        session_data = {
            "session_id": st.session_state.session_id,  # 会话标识
            "messages": st.session_state.messages,  # 聊天信息
            "role": st.session_state.role,  # 角色
            "skills": st.session_state.skills  # 技能
        }
        if not os.path.exists("sessions"):
            os.mkdir("sessions")
        with open(f"sessions/{st.session_state.session_id}.json", "w", encoding="utf-8") as f:
            json.dump(session_data, f, ensure_ascii=False, indent=2)

# 加载所有会话标识
def load_sessions():
    # 初始化会话标识列表
    sessions_list = []
    if os.path.exists("sessions"):
        for file in os.listdir("sessions"):
            if file.endswith(".json"):
                file_name = file[:-5]
                sessions_list.append(file_name)
    return sessions_list

# 加载当前会话
def load_session(session_id):
    try:
        if os.path.exists(f"sessions/{session_id}.json"):
            with open(f"sessions/{session_id}.json", "r", encoding="utf-8") as f:
                session_data = json.load(f)
                st.session_state.messages = session_data["messages"]  # 聊天信息
                st.session_state.role = session_data["role"]  # 角色
                st.session_state.skills = session_data["skills"]  # 技能
                st.session_state.session_id = session_id  # 会话标识
    except Exception:
        st.error("会话加载失败")

# 删除会话
def delete_session(session_id):
    try:
        if os.path.exists(f"sessions/{session_id}.json"):
            os.remove(f"sessions/{session_id}.json")
            if session_id == st.session_state.session_id:
                st.session_state.messages = []
                st.session_state.session_id = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
                st.session_state.role = "编程老师"
                st.session_state.skills = "编程，教学"
            st.rerun()
    except Exception:
        st.error("会话删除失败")

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
# 初始化会话标识
if "session_id" not in st.session_state:
    # 创建一个变量，用于存储会话标识,%Y：年，%m：月，%d：日，%H：时，%M：分，%S：秒
    st.session_state.session_id = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

st.text(f"当前会话：{st.session_state.session_id}")

# 将聊天信息显示在页面上
for message in st.session_state.messages:
    if message["role"] == "user":
        st.chat_message("user").write(message["content"])
    else:
        st.chat_message("assistant").write(message["content"])

# 侧边栏部分
with st.sidebar:
    st.subheader("AI控制面板")
    if st.button("新建会话",width="stretch",icon="📝"):
        # 如果有聊天信息，则保存
        if st.session_state.messages:
            # 第一步：将当前的会话进行保存
            save_session()
            # 第二步：创建新会话
            st.session_state.messages = []  # 清空session_state.messages
            st.session_state.session_id = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")  # 创建新的会话标识
            st.session_state.role = "编程老师"  # 重置为默认角色
            st.session_state.skills = "编程，教学"  # 重置为默认技能
            save_session()  # 保存新建会话
            st.rerun()  # 重新运行页面
    
    st.text("历史会话")
    session_list = load_sessions()
    # 对列表进行排序
    session_list.sort(reverse=True)
    for session in session_list:
        clu1, clu2 = st.columns([4,1])
        # 创建一个按钮，用于选择会话
        with clu1:
            if st.button(session, width="stretch", icon="💬", key=f"load_{session}", type="primary" if session == st.session_state.session_id else "secondary"):
                load_session(session)
                st.rerun()
        # 创建一个按钮，用于删除会话
        with clu2:
            if st.button("", width="stretch", icon="❌️", key=f"del_{session}"):
                delete_session(session)
                st.rerun()
    
    # 分割线
    st.divider()
    st.subheader("助手信息")
    # 创建一个输入框，用于输入角色
    set_up_roles = st.text_input("角色",placeholder="例如：编程老师等等",value=f"{st.session_state.role}")
    # 将用户输入的角色添加到session_state.role中
    if set_up_roles.strip() != "":
        st.session_state.role = set_up_roles
    # 创建一个输入框，用于输入该角色的技能
    set_up_skills = st.text_area("技能",placeholder="例如：编程，数学等等",value=f"{st.session_state.skills}")
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
    # 保存会话
    save_session()
```

# 六、功能效果展示与全面测试
## 6.1 新建/保存会话测试
1. 运行程序：在终端执行`streamlit run ai_chat.py`
2. 在聊天输入框中输入"Python中列表和元组的区别是什么？"，点击发送
3. 等待AI回复完成后，查看项目根目录，会发现自动创建了一个`sessions`文件夹
4. 打开`sessions`文件夹，会看到一个以时间戳命名的JSON文件，这就是刚才保存的会话
5. 点击侧边栏的"新建会话"按钮
6. 页面会刷新，聊天区清空，顶部的"当前会话"标识更新为新的时间戳
7. 再次查看`sessions`文件夹，会看到多了一个新的JSON文件

## 6.2 历史会话列表展示
1. 重复上述步骤，创建3-4个不同的会话
2. 观察侧边栏的"历史会话"区域，会看到所有创建过的会话，按时间倒序排列
3. 当前正在使用的会话按钮会显示为蓝色，其他会话显示为灰色

## 6.3 加载历史会话测试
1. 点击历史会话列表中的第一个会话（最新的那个）
2. 页面会刷新，聊天区会显示该会话的所有聊天记录
3. 侧边栏的"角色"和"技能"输入框会自动填充该会话的配置
4. 继续输入新的问题，AI会基于该会话的上下文进行回复

## 6.4 删除会话测试
1. 点击某个会话右侧的叉号按钮
2. 该会话会从历史列表中消失
3. 查看`sessions`文件夹，对应的JSON文件已经被删除
4. 如果删除的是当前正在使用的会话，页面会自动跳转到一个新的空会话

## 6.5 自动保存测试
1. 在当前会话中输入一个新的问题，等待AI回复
2. 直接关闭浏览器标签页
3. 重新运行程序，打开浏览器
4. 点击历史会话列表中刚才的会话
5. 聊天区会显示刚才发送的问题和AI的回复，说明自动保存功能正常

# 七、关键注意事项与避坑指南
1. **文件权限问题**：确保程序对项目根目录有读写权限，否则无法创建`sessions`文件夹和写入文件
2. **中文文件名问题**：虽然我们使用时间戳作为文件名，不会有中文问题，但如果后续修改为自定义名称，一定要确保系统支持中文文件名
3. **多用户场景问题**：本方案是单用户本地部署方案，如果部署到服务器供多人使用，会出现会话冲突问题。多用户场景需要结合用户认证和数据库实现
4. **数据备份问题**：所有会话数据都保存在本地的`sessions`文件夹中，建议定期备份该文件夹，防止数据丢失
5. **流式输出空值问题**：部分API返回的chunk可能包含空的`delta.content`，可以添加`or ""`进行兜底，避免报错：
   ```python
   chunk_str = chunk.choices[0].delta.content or ""
   ```

# 八、核心总结：会话管理实现核心速查表
为了方便后续开发时快速查阅，整理了会话管理实现的核心速查表：

| 核心环节   | 关键操作                         | 核心要点                           |
| ---------- | -------------------------------- | ---------------------------------- |
| 数据持久化 | 基于JSON文件存储                 | 轻量、易读、无需额外依赖           |
| 状态管理   | st.session_state存储4个核心变量  | messages、role、skills、session_id |
| 保存会话   | save_session()函数               | 封装数据→创建目录→写入JSON文件     |
| 新建会话   | 保存旧会话→重置状态→刷新页面     | 自动保存当前会话，避免数据丢失     |
| 加载会话   | load_session()函数               | 读取JSON文件→恢复所有状态→刷新页面 |
| 删除会话   | delete_session()函数             | 删除文件→处理当前会话→刷新页面     |
| 自动保存   | 用户发送消息后调用save_session() | 无需手动操作，数据实时持久化       |
| 界面布局   | 侧边栏集中管理所有功能           | 主区域专注聊天，提升用户体验       |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163862439>
