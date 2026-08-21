

# Python全栈入门到实战【AI实战篇 11】小辉AI聊天助手项目完整总结，第一个AI项目收官篇
上一篇《AI实战篇 10》中，我们已经完成了AI应用本地会话管理功能的开发，实现了会话的自动保存、新建、加载和删除，解决了数据丢失和多主题管理的核心痛点。至此，我们的第一个AI项目——小辉AI聊天助手已经全部开发完成。本篇作为第一个AI项目的收官篇，我们将对整个项目进行全面的总结，回顾从0到1的完整开发历程，梳理核心技术要点，总结项目亮点与不足，并展望未来的扩展方向。

本文为Python全栈开发者与零基础AI入门者量身打造，通过系统性的复盘，帮助你巩固整个项目的核心知识，理清AI应用开发的完整流程，为后续更复杂的AI项目开发打下坚实的基础。

本节核心学习内容：
1. 项目概述：整体技术栈与最终实现的全功能清单
2. 分阶段开发总结：回顾从基础聊天到会话管理的6个开发阶段
3. 核心技术要点：提炼整个项目中最关键的4个技术点
4. 完整效果展示：最终项目的页面布局与功能演示
5. 项目亮点与不足：总结项目优势，明确未来的改进方向
6. 总结与展望：第一个AI项目的收获与后续扩展规划
7. 核心速查表：整个项目的核心知识点汇总，方便快速查阅

# 一、项目概述
本项目基于**Streamlit + DeepSeek API**纯Python开发，全程无需任何HTML/CSS/JS前端基础，从零开始构建了一个功能完整、体验流畅的本地AI聊天助手。项目从最基础的单次聊天功能起步，逐步迭代实现了**历史消息展示、会话上下文记忆、流式输出、自定义AI角色与技能、本地会话持久化管理**五大核心模块，最终形成了一个可直接使用、易于扩展的AI应用原型。

## 1.1 项目技术栈
| 技术/库    | 版本要求   | 核心作用                                     |
| ---------- | ---------- | -------------------------------------------- |
| Python     | ≥3.8       | 项目开发语言                                 |
| Streamlit  | ≥1.30.0    | 快速构建Web聊天界面，提供所有UI组件          |
| OpenAI SDK | 最新稳定版 | 对接DeepSeek API（完全兼容OpenAI接口规范）   |
| json       | Python内置 | 会话数据的序列化与反序列化                   |
| os         | Python内置 | 文件系统操作（创建目录、读写文件、删除文件） |
| datetime   | Python内置 | 生成基于时间戳的唯一会话ID                   |

## 1.2 最终实现功能清单
1. ✅ 基础聊天功能：用户输入→AI回复的核心交互
2. ✅ 历史消息展示：页面刷新后保留所有聊天记录
3. ✅ 会话上下文记忆：AI能理解连续提问的关联关系
4. ✅ 流式输出：AI边思考边输出，提升实时交互体验
5. ✅ 自定义AI角色：实时修改AI的身份（如编程老师、英语老师）
6. ✅ 自定义AI技能：指定AI的能力范围（如Python编程、简历优化）
7. ✅ 本地会话持久化：所有会话自动保存到本地JSON文件
8. ✅ 新建会话：一键创建新的空白聊天主题
9. ✅ 历史会话列表：按时间倒序展示所有保存的会话
10. ✅ 加载历史会话：一键恢复任意会话的聊天记录和配置
11. ✅ 删除会话：一键清理无用会话，释放本地空间

# 二、分阶段开发总结
整个项目分为6个清晰的开发阶段，每个阶段都解决一个核心问题，逐步完善应用功能，形成了完整的开发闭环。

## 2.1 第一阶段：基础单次聊天功能
**开发目标**：实现最核心的"用户输入→API调用→AI回复"流程
**核心代码**：
```python
# 调用DeepSeek API
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt},
    ],
    stream=False
)
# 显示AI回复
st.chat_message("assistant").write(response.choices[0].message.content)
```
**解决的问题**：打通了前端界面与大模型API的连接，实现了AI聊天的基本功能
**存在的问题**：每次发送新消息后，历史消息会被覆盖，无法保留聊天记录

## 2.2 第二阶段：历史消息展示功能
**开发目标**：解决消息被覆盖的问题，实现历史消息的持久化展示
**核心技术**：`st.session_state`会话状态管理
**核心代码**：
```python
# 初始化聊天消息列表
if "messages" not in st.session_state:
    st.session_state.messages = []
# 循环展示所有历史消息
for message in st.session_state.messages:
    if message["role"] == "user":
        st.chat_message("user").write(message["content"])
    else:
        st.chat_message("assistant").write(message["content"])
# 新消息追加到列表
st.session_state.messages.append({"role": "user", "content": prompt})
st.session_state.messages.append({"role": "assistant", "content": ai_reply})
```
**解决的问题**：利用`st.session_state`存储聊天记录，页面刷新后数据不丢失
**存在的问题**：仅能展示历史消息，AI无法基于历史上下文进行回复

## 2.3 第三阶段：会话上下文记忆功能
**开发目标**：让AI能够理解连续提问的关联关系
**核心原理**：将全量历史消息传递给大模型API
**核心代码**：
```python
# 调用API时传入所有历史消息
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},
        *st.session_state.messages,  # 解包所有历史消息
    ],
    stream=False
)
```
**解决的问题**：AI现在可以基于之前的对话内容进行回复，支持连续追问
**存在的问题**：非流式输出导致等待时间长，用户体验差

## 2.4 第四阶段：流式输出功能
**开发目标**：实现AI"边思考边输出"的效果，提升用户体验
**核心技术**：`st.empty()`动态更新组件
**核心代码**：
```python
# 开启流式输出
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[...],
    stream=True
)
# 流式渲染
response_message = st.empty()
output = ""
for chunk in response:
    chunk_str = chunk.choices[0].delta.content or ""
    output += chunk_str
    response_message.chat_message("assistant").write(output)
```
**解决的问题**：解决了长回复等待时间长的问题，实现了与ChatGPT一致的输出体验
**存在的问题**：AI角色和技能固定，无法根据用户需求灵活切换

## 2.5 第五阶段：侧边栏自定义角色与技能功能
**开发目标**：让用户可以实时修改AI的身份和能力范围
**核心技术**：动态生成系统提示词
**核心代码**：
```python
# 侧边栏配置
with st.sidebar:
    st.subheader("助手信息")
    set_up_roles = st.text_input("角色", value=st.session_state.role)
    set_up_skills = st.text_area("技能", value=st.session_state.skills)
    if set_up_roles:
        st.session_state.role = set_up_roles
    if set_up_skills:
        st.session_state.skills = set_up_skills
# 动态生成系统提示词
system_prompt = f"我是一名资深的{st.session_state.role}，我的名字叫做小辉辉，我的技能是{st.session_state.skills}"
```
**解决的问题**：无需修改代码，即可将AI切换为不同的角色，适配不同的使用场景
**存在的问题**：所有会话数据仅存在于内存中，关闭浏览器后永久丢失

## 2.6 第六阶段：本地会话持久化管理功能
**开发目标**：实现会话数据的本地保存，支持多会话管理
**核心技术**：JSON文件持久化
**核心功能**：
- 自动保存当前会话（用户发送消息后自动执行）
- 新建会话（保存旧会话→重置状态→创建新会话）
- 历史会话列表展示（按时间倒序排列）
- 加载历史会话（恢复聊天记录和配置）
- 删除会话（物理删除JSON文件）
  **解决的问题**：解决了数据丢失问题，支持同时管理多个独立的聊天主题

# 三、核心技术要点总结
整个项目的核心技术可以提炼为以下4点，这些也是所有Streamlit AI应用开发的通用基础：

## 3.1 `st.session_state`的使用
`st.session_state`是Streamlit中最重要的概念之一，也是本项目的核心基础：
- 它是一个会话级别的字典，生命周期与用户浏览器会话一致
- 不同用户的`st.session_state`相互隔离，不会互相干扰
- 所有需要跨交互保留的数据都应该存储在`st.session_state`中
- 必须添加`if "xxx" not in st.session_state:`判断进行初始化，避免变量未定义错误

## 3.2 流式输出的实现原理
- 大模型API的流式输出会将回复拆分成多个小数据包，逐个返回
- 不能再使用`response.choices[0].message.content`获取完整回复
- 需要遍历`Stream`对象，逐个提取数据包中的内容并拼接
- 使用`st.empty()`创建一个可动态更新的占位符，每次更新都覆盖之前的内容，避免重复渲染

## 3.3 本地数据持久化方案
- 选择JSON格式存储数据，简单易读，易于解析
- 使用基于时间戳的唯一ID作为文件名，保证唯一性
- 自动创建`sessions`目录，统一管理所有会话文件
- 实现了"保存-加载-删除"的完整文件操作流程

## 3.4 系统提示词的作用
- 系统提示词定义了AI的身份、能力和行为准则
- 动态生成系统提示词是实现自定义角色和技能的核心
- 系统提示词必须放在`messages`数组的第一个位置，否则可能无法生效
- 系统提示词越具体，AI的回答越符合预期

# 四、完整项目效果展示
运行`streamlit run ai_chat.py`后，浏览器会自动打开AI聊天助手页面，完整效果如下：

![在这里插入图片](https://i-blog.csdnimg.cn/direct/621a620e7c9047c4a9f9b5f3e31e8f2f.gif#pic_center)

## 页面区域详细说明
1. **顶部区域**：
   - 页面标题："小辉AI"
   - 应用Logo：左上角显示自定义的AI助手Logo
   - 当前会话ID：显示当前正在使用的会话的唯一标识（时间戳格式）
2. **左侧侧边栏区域（AI控制面板）**：
   - **AI控制面板**：
     - "新建会话"按钮：点击后保存当前会话并创建新的空白会话
     - 历史会话列表：按时间倒序展示所有保存的会话，当前会话按钮为蓝色高亮
     - 每个会话项包含两个按钮：左侧是加载按钮（💬），右侧是删除按钮（❌️）
   - **助手信息配置**：
     - "角色"输入框：自定义AI的身份，默认值为"编程老师"
     - "技能"输入框：自定义AI的能力范围，默认值为"编程，教学"
3. **右侧主聊天区域**：
   - 历史聊天记录：按时间顺序展示所有用户消息和AI回复
   - 用户消息：蓝色气泡，头像在左侧
   - AI消息：灰色气泡，头像在左侧
   - 聊天输入框：页面底部的输入框，支持输入文本并发送

# 五、完整项目代码展示

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
    # 非流式输出代码
    # st.chat_message("assistant").write(response.choices[0].message.content)

    # 将OpenAI API返回的答案添加到session_state.messages中
    # st.session_state.messages.append({"role": "assistant", "content": response.choices[0].message.content})
```

# 六、项目亮点与不足
## 6.1 项目亮点
1. **纯Python开发**：无需任何前端知识，Python开发者即可快速上手
2. **功能完整**：涵盖了AI聊天应用的所有核心功能，可直接使用
3. **代码结构清晰**：采用模块化设计，每个功能拆分为独立的函数，易于维护和扩展
4. **用户体验良好**：实现了流式输出、自动保存、状态高亮等细节优化
5. **部署简单**：无需复杂的后端服务和数据库，本地运行即可使用
6. **扩展性强**：基于Streamlit的组件生态，可以轻松添加更多功能

## 6.2 项目不足与改进方向
1. **单用户限制**：本方案是单用户本地部署方案，不支持多用户同时使用。多用户场景需要结合用户认证和数据库实现
2. **本地存储限制**：所有数据都保存在本地，无法跨设备访问。可以考虑对接云端数据库（如SQLite、MySQL）实现云端存储
3. **会话命名不直观**：默认使用时间戳作为会话名称，不够直观。可以添加会话重命名功能
4. **缺乏搜索功能**：当历史会话较多时，难以快速找到目标会话。可以添加会话搜索功能
5. **没有异常重试机制**：API调用失败时没有自动重试，需要用户手动重新发送

# 七、总结与展望
本项目从零开始，一步步构建了一个功能完整的AI聊天助手，涵盖了从基础交互到高级会话管理的所有环节。通过这个项目，我们深入理解了Streamlit的运行机制、大模型API的调用方式、数据持久化的实现方法等核心技术，掌握了纯Python开发AI应用的完整流程。

虽然目前的版本已经能够满足个人日常使用的需求，但还有很大的扩展空间。未来可以继续添加以下功能：
- 多用户支持与用户认证
- 云端数据存储与同步
- 会话重命名与标签分类
- 聊天内容搜索
- 语音输入与输出
- 文件上传与解析
- 插件系统（支持调用外部工具）

相信通过不断的迭代和优化，这个AI聊天助手可以变得更加完善和实用，成为日常工作和学习的得力助手。

# 八、核心总结：小辉AI聊天助手核心速查表
为了方便后续开发时快速查阅，整理了整个项目的核心速查表：

| 开发阶段 | 核心功能         | 核心技术             | 核心代码                                        |
| -------- | ---------------- | -------------------- | ----------------------------------------------- |
| 第一阶段 | 基础单次聊天     | DeepSeek API调用     | `client.chat.completions.create()`              |
| 第二阶段 | 历史消息展示     | `st.session_state`   | `st.session_state.messages.append()`            |
| 第三阶段 | 会话上下文记忆   | 全量历史消息传递     | `*st.session_state.messages`                    |
| 第四阶段 | 流式输出         | `st.empty()`动态更新 | `response_message.chat_message().write(output)` |
| 第五阶段 | 自定义角色与技能 | 动态系统提示词       | `f"我是一名资深的{st.session_state.role}"`      |
| 第六阶段 | 本地会话管理     | JSON文件持久化       | `json.dump()`/`json.load()`                     |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163862506>
