

# Python全栈入门到实战【AI实战篇 07】AI应用会话记忆功能全实现，第一个AI项目连续对话核心
上一篇《AI实战篇 06》中，我们已经完成了基于Streamlit的AI聊天消息展示模块开发，解决了Streamlit无状态特性导致的历史消息被覆盖问题，实现了聊天记录的持久化展示。但此时的应用仅能被动展示历史消息，**不具备真正的会话记忆能力**——AI无法关联连续提问的上下文，导致追问类场景的回复完全偏离预期。本篇作为第一个AI项目的第七篇，我们将从问题发现、本质分析、思路梳理到代码落地，一步步带你彻底实现AI应用的会话记忆功能，这是所有对话类AI应用的核心体验基础。

本文为Python全栈开发者与零基础AI入门者量身打造，全程遵循“发现问题→分析问题→解决问题→验证效果”的完整开发逻辑，仅需修改一行核心代码即可实现功能，同时深入讲解底层原理和关键注意事项，即使是完全没有AI开发经验的同学，也能理解会话记忆的本质，并成功实现连续对话功能。

本节核心学习内容：
1. 问题复现：直观展示AI无法理解连续追问的异常现象
2. 本质解析：深入理解大模型API无状态设计的核心特性
3. 思路梳理：从问题定位到技术选型的完整解决思路推导
4. 代码落地：核心原理讲解、一行代码修改实现会话记忆、完整最终代码
5. 功能测试：多场景验证会话记忆效果、用户隔离验证
6. 关键注意事项：上下文长度限制、消息格式、系统提示词位置等核心避坑点
7. 核心总结：会话记忆实现核心速查表，方便开发时快速查阅

# 一、问题发现与本质分析
在完成历史消息展示功能后，我们进行连续对话测试，发现了一个严重影响用户体验的核心问题：应用仅能被动展示历史聊天消息，但AI无法关联连续提问的上下文，导致追问类场景的回复完全偏离预期。

## 1.1 问题场景复现
我们通过最典型的“分苹果”连续追问场景，稳定复现这个问题：
1. 第一轮提问：在聊天输入框中输入“现在有12个苹果，4个人怎么均分苹果？”，点击发送
2. AI正常回复：“12个苹果分给4个人，每人可以分到3个苹果”
3. 第二轮追问：紧接着输入“那两个人呢？”，点击发送
4. AI异常回复：无法理解“那两个人呢？”的指代含义，会回答“两个人可以一起学习、一起工作”等完全无关的内容

![在这里插入图片](https://i-blog.csdnimg.cn/direct/19dcb3dcae1e432f9a24d661996c76f1.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fef185259ed44f3f80c5f0f2613d6468.png#pic_center)

## 1.2 问题本质深度解析
很多人会误以为这是AI模型的能力缺陷，但实际上这是**所有大模型API的通用设计特性**，并非bug。我们通过排查代码和API调用日志，最终定位到问题的根源：

### 大模型API的无状态特性
- 所有大模型API都采用“单次请求-单次响应”的无状态设计
- 服务器不会为任何用户存储历史会话信息
- 模型的所有输出，都**100%依赖本次请求中传入的`messages`参数**

### 此前代码的缺陷
此前的代码中，调用DeepSeek API时的`messages`参数是这样的：
```python
messages=[
    {"role": "system", "content": system_prompt},
    {"role": "user", "content": prompt},
]
```
可以清晰地看到，这里只传入了**当前用户的单次提问**，没有携带任何历史会话信息。因此，模型根本不知道用户之前问过“分苹果”的问题，自然无法理解“那两个人呢？”的指代含义。

# 二、核心解决思路梳理
基于上述问题分析，我们梳理出了一套完整的解决思路，从问题定位到最终验证，环环相扣：

## 2.1 问题定位思路
1. 先通过**最小可复现场景**（分苹果连续追问）稳定复现问题
2. 排除基础错误：检查代码语法、API密钥、网络连接、模型名称是否正确
3. 抓包查看API请求参数，对比正常请求和异常请求的差异
4. 发现核心差异：异常请求的`messages`参数中缺少历史会话信息
5. 得出结论：问题根源是API调用时未传递上下文，而非模型能力不足

## 2.2 原理推导思路
1. 既然大模型API本身不存储任何会话，那么要实现会话记忆，只能由**客户端主动存储并传递上下文**
2. 模型只能识别`[{"role": "xxx", "content": "xxx"}]`格式的消息列表，因此历史消息也必须遵循这个格式
3. 每次调用API时，需要将“系统提示词 + 所有历史用户提问 + 所有历史AI回复”一起传递给模型
4. 这样模型就能看到完整的对话过程，从而理解连续提问的上下文关联

## 2.3 技术选型思路
我们需要一个**轻量、会话级、支持用户隔离**的存储容器来保存历史消息，逐一排除不合适的方案：
- ❌ 普通变量：Streamlit每次交互都会重新运行整个脚本，普通变量会被重新初始化，无法保留数据
- ❌ 本地文件：多用户场景下会出现文件读写冲突，且无法实现用户隔离
- ❌ 数据库：对于轻量应用来说过于复杂，增加开发和部署成本
- ✅ **Streamlit原生`st.session_state`**：会话级别的字典存储，生命周期与用户浏览器会话一致，天然支持用户隔离，无需额外配置，是最优方案

## 2.4 代码实现思路
1. 初始化阶段：在`st.session_state`中创建一个空列表`messages`，用于存储所有历史聊天消息
2. 页面渲染阶段：遍历`messages`列表，按角色（用户/助手）渲染所有历史消息
3. 用户输入阶段：捕获用户的新提问，先渲染到页面，再追加到`messages`列表
4. API调用阶段：将系统提示词与`messages`列表中的所有历史消息拼接，作为`messages`参数传入API
5. 结果处理阶段：提取AI的回复，渲染到页面，再追加到`messages`列表，为下一次提问准备上下文

## 2.5 验证思路
1. 再次执行“分苹果”的连续追问场景，验证AI能否正确关联上下文
2. 多轮连续提问，验证会话记忆是否能持续生效
3. 刷新浏览器页面，验证历史消息是否仍然保留
4. 打开多个浏览器标签页，验证不同用户的会话是否相互隔离

# 三、解决方案与代码落地
## 3.1 核心原理：存储-传递-更新三步法
实现会话记忆的核心逻辑可以总结为“存储-传递-更新”三步法：
1. **存储**：用`st.session_state.messages`存储所有历史聊天消息，格式为`[{"role": "xxx", "content": "xxx"}]`
2. **传递**：调用API时，将系统提示词与所有历史消息一起传递给模型
3. **更新**：每次用户提问、AI回复后，都将新内容追加到`st.session_state.messages`中，形成上下文闭环

## 3.2 核心代码修改（仅需修改一行）
对比历史消息展示版本的代码，我们**只需要修改API调用时的`messages`参数**这一行，就能实现完整的会话记忆功能。

### 修改前的代码
```python
# 调用OpenAI API
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt},
    ],
    stream=False
)
```

### 修改后的代码
```python
# 调用OpenAI API（核心修改：传入全量历史消息）
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},
        *st.session_state.messages,  # 解包session_state中的所有历史消息
    ],
    stream=False
)
```

## 3.3 核心代码逐行解析
- `*st.session_state.messages`：这是Python中的**解包语法**，作用是将`st.session_state.messages`列表中的每一个元素，都作为独立的元素传入`messages`数组中。

举个具体的例子，如果`st.session_state.messages`中有两条历史消息：
```python
st.session_state.messages = [
    {"role": "user", "content": "现在有12个苹果，4个人怎么均分苹果？"},
    {"role": "assistant", "content": "12个苹果分给4个人，每人可以分到3个苹果"}
]
```
那么`*st.session_state.messages`解包后，最终传递给API的`messages`参数就变成了：
```python
messages=[
    {"role": "system", "content": "我是一名Python编程老师，我的名字叫做小辉辉，有10年的教学经验"},
    {"role": "user", "content": "现在有12个苹果，4个人怎么均分苹果？"},
    {"role": "assistant", "content": "12个苹果分给4个人，每人可以分到3个苹果"}
]
```
这样，模型就能完整看到之前的对话过程，当用户再问“那两个人呢？”时，模型就知道是在问“12个苹果分给2个人怎么分”。

## 3.4 完整最终代码
将`ai_chat.py`中的代码全部替换为以下内容：
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
    # 调用OpenAI API（核心修改：传入全量历史消息）
    response = client.chat.completions.create(
        model="deepseek-chat",
        messages=[
            {"role": "system", "content": system_prompt},
            *st.session_state.messages,# 获取session_state.messages中的所有消息
        ],
        stream=False
    )
    st.chat_message("assistant").write(response.choices[0].message.content)
    print(response.choices[0].message.content)
    # 将OpenAI API返回的答案添加到session_state.messages中
    st.session_state.messages.append({"role": "assistant", "content": response.choices[0].message.content})
```

# 四、功能测试与效果验证
## 4.1 基础测试步骤
1. 按`Ctrl+C`终止之前的Streamlit服务，重新执行启动命令：
   ```bash
   streamlit run .\ai_chat.py
   ```
2. 第一轮提问：输入“现在有12个苹果，4个人怎么均分苹果？”，点击发送
3. 等待AI回复完成后，第二轮追问：输入“那两个人呢？”，点击发送

## 4.2 测试结果
AI成功关联上一轮的上下文，返回了完全符合预期的结果：
- 第一轮回复：“12个苹果分给4个人，每人可以分到3个苹果”
- 第二轮回复：“12个苹果分给2个人的话，每人可以分到6个苹果”

![在这里插入图片](https://i-blog.csdnimg.cn/direct/099684a1f70c4228899b74edb3838850.png#pic_center)

![在这里插入图片](https://i-blog.csdnimg.cn/direct/2bfce268bed940c4848a351afe87148a.png#pic_center)

## 4.3 进一步验证
1. 继续第三轮追问：“那三个人呢？”，AI会正确回答“每人分4个苹果”
2. 刷新浏览器页面，所有历史消息仍然保留在页面上
3. 打开一个新的浏览器标签页，访问相同地址，会得到一个全新的空白会话，验证了不同用户的会话相互隔离

# 五、关键注意事项
1. **上下文长度限制**：DeepSeek-chat模型的上下文窗口为128k token，当会话消息过多时，会超出token限制导致API调用失败。此时可以通过截断历史消息解决，例如只保留最近的20条消息：
   ```python
   # 在调用API前添加这行代码
   st.session_state.messages = st.session_state.messages[-20:]
   ```
2. **消息格式正确性**：`st.session_state.messages`中的每一条消息都必须严格遵循`{"role": "xxx", "content": "xxx"}`的格式，不能有其他格式的元素，否则会导致API调用报错。
3. **系统提示词位置**：系统提示词必须放在`messages`数组的第一个位置，不能放在中间或末尾，否则模型可能无法正确识别系统提示词。
4. **会话生命周期**：`st.session_state`的生命周期与浏览器标签页一致，关闭标签页后会话会自动清除。如果需要跨会话记忆，需要结合数据库实现消息持久化。

# 六、核心总结：会话记忆实现核心速查表
为了方便后续开发时快速查阅，整理了会话记忆实现的核心速查表：

| 核心环节     | 关键操作                                     | 核心要点                             |
| ------------ | -------------------------------------------- | ------------------------------------ |
| 问题本质     | 大模型API无状态，不存储历史会话              | 所有输出仅依赖本次传入的messages参数 |
| 核心原理     | 存储-传递-更新三步法                         | 客户端主动存储并传递完整历史上下文   |
| 存储方案     | st.session_state.messages                    | 会话级存储，天然支持用户隔离         |
| 核心代码修改 | 解包全量历史消息传入API                      | `*st.session_state.messages`         |
| 消息格式     | {"role": "user/assistant", "content": "xxx"} | 必须严格遵循，否则API调用失败        |
| 系统提示词   | 必须放在messages数组的第一个位置             | 不能放在中间或末尾                   |
| 长度控制     | 截断早期非关键对话                           | 避免超出模型的token上限              |
| 用户隔离     | 不同用户的st.session_state相互独立           | 无需额外配置，自动实现               |

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163779577>
