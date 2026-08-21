

# Python全栈入门到实战【AI实战篇 08】AI智能助手流式输出全流程实战，第一个AI项目体验优化核心
上一篇《AI实战篇 07》中，我们已经实现了AI应用的会话记忆功能，解决了AI无法理解连续追问的核心问题，让智能助手具备了真正的连续对话能力。本篇作为第一个AI项目的第八篇，我们将解决另一个严重影响用户体验的痛点——**非流式输出的等待卡顿问题**，实现“AI边思考边输出”的打字机效果，让我们的智能助手体验媲美ChatGPT等主流AI产品。

本文为Python全栈开发者与零基础AI入门者量身打造，全程还原从踩坑到优化的完整开发过程，详细讲解每一个错误的根源和解决方案，最终给出可直接复制使用的完整代码，同时整理了所有新手必踩的坑和避坑指南，即使是完全没有经验的同学，也能跟着一步步实现流畅的流式输出效果。

本节核心学习内容：
1. 问题引入：非流式输出的核心体验痛点
2. 踩坑实录：直接开启流式输出的报错问题与根源分析
3. 原理解析：DeepSeek API流式响应的数据结构与特点
4. 第一版实现：基础流式输出代码与重复渲染问题
5. 最终优化：使用`st.empty()`组件解决重复渲染，实现流畅输出
6. 完整整合：历史消息+会话记忆+流式输出三合一完整代码
7. 避坑指南：空值处理、会话状态保存、异常处理、输出速度控制
8. 效果对比：各阶段实现效果的直观对比与核心总结
9. 核心速查表：流式输出实现关键步骤，方便开发时快速查阅

# 一、问题引入：非流式输出的体验痛点
在之前的AI聊天助手开发中，我们使用的是**非流式输出模式**（`stream=False`），这种模式虽然实现简单，但存在一个致命的体验问题：
- 用户发送问题后，需要等待AI生成完整回答后，才能一次性看到所有内容
- 回答越长，等待时间越久，用户会产生“程序卡住了”的错觉
- 缺乏实时反馈，交互体验生硬，与ChatGPT等主流AI产品的体验差距明显

为了解决这个问题，我们需要将AI的响应方式修改为**流式输出**，实现“AI边思考边输出”的效果，大幅提升用户体验。

# 二、初步尝试：开启流式输出（踩坑第一步）
首先，我们尝试最简单的修改方式：将API调用中的`stream`参数从`False`改为`True`，但直接修改会触发报错，下面详细讲解整个过程。

## 2.1 初始修改代码
```python
# 调用OpenAI API
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},
        *st.session_state.messages,# 获取session_state.messages中的所有消息
    ],
    stream=True# 直接将这里修改为True，开启流式输出
)
```

## 2.2 运行结果与报错分析
运行代码后，浏览器会出现报错，效果如下：

![在这里插入图片](https://i-blog.csdnimg.cn/direct/971148945d9f46e6aab73095a6c31565.png#pic_center)

### 报错核心原因
这个错误是所有初学者在实现流式输出时都会遇到的经典问题，本质是**混淆了流式输出和非流式输出的返回数据格式**：
- **非流式输出（`stream=False`）**：API返回一个完整的响应对象，包含`choices`属性，可以直接通过`response.choices[0].message.content`获取完整的回答内容
- **流式输出（`stream=True`）**：API不会一次性返回完整结果，而是返回一个**可迭代的`Stream`对象**，需要通过循环遍历这个对象，逐个获取数据块（chunk），再拼接成完整的回答

# 三、分析流式响应结构：明确数据格式
为了正确提取流式数据，我们需要先了解流式响应的原始结构。这里使用APIFOX工具调用DeepSeek API，开启流式输出，查看返回的数据格式：

![在这里插入图片](https://i-blog.csdnimg.cn/direct/3ec6875d92ab42f4888b3f6eebb6dd64.png#pic_center)

## 3.1 流式响应核心特点
1. **分块返回**：AI生成的回答会被拆分成多个小数据包，逐个返回给客户端
2. **统一格式**：每个数据包的结构完全一致，核心内容在`choices[0].delta.content`字段中
3. **拼接成完整内容**：将所有数据包的`delta.content`字段依次拼接，就能得到完整的AI回答
4. **空值处理**：部分数据包的`delta.content`可能为`None`（如开始和结束的控制包），需要特殊处理，避免拼接出`None`字符串

# 四、第一版流式实现：解决报错但出现新问题
根据流式响应的结构，我们修改代码，通过循环遍历`Stream`对象，逐个提取数据块并拼接，解决报错问题。

## 4.1 第一版代码实现
```python
# 调用OpenAI API
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},
        *st.session_state.messages,# 获取session_state.messages中的所有消息
    ],
    stream=True# 开启流式输出
)
# 初始化流式输出变量，用于拼接所有数据块
output = ""
# 遍历流式响应的每个数据块
for chunk in response:
    # 从当前数据块中提取文本内容
    chunk_str = chunk.choices[0].delta.content# 获取当前chunk的content
    output += chunk_str# 将当前chunk的content添加到output中
    # 将拼接后的内容显示在页面上
    st.chat_message("assistant").write(output)
```

## 4.2 运行结果与问题分析
运行代码后，报错问题解决了，也能看到文本逐步输出，但出现了一个更严重的体验问题：**AI的文本会重复渲染，页面上会出现多个重叠的消息框**。
效果如下：

![在这里插入图片](https://i-blog.csdnimg.cn/direct/181f5479aa3b470b84ae26f58922a92d.gif#pic_center)

### 问题根源
这个问题是由Streamlit的渲染机制导致的：
- 每次调用`st.chat_message("assistant").write(output)`时，Streamlit都会在页面上**新建一个助手消息框**
- 循环执行多少次，就会新建多少个消息框
- 最终页面上会出现多个重叠的消息框，显示不同阶段的拼接内容，严重影响用户体验

# 五、最终优化：用`st.empty()`组件实现流畅流式渲染
Streamlit提供了`st.empty()`组件，专门用于解决动态更新内容的问题。它可以在页面上定义一个**可重复使用的空白区域**，所有流式文本都往这个空白区域中更新，而不是重复创建新的消息框。

## 5.1 `st.empty()`组件核心作用
- 在页面上创建一个空的占位符
- 后续可以通过这个占位符，在同一个位置反复更新内容
- 每次更新都会覆盖之前的内容，不会新建元素
- 完美解决流式输出的重复渲染问题

## 5.2 最终优化版代码
```python
# 调用OpenAI API
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},
        *st.session_state.messages,# 获取session_state.messages中的所有消息
    ],
    stream=True# 开启流式输出
)
# 核心优化：创建一个空的占位符组件，用于动态更新助手消息
response_message = st.empty()
# 初始化流式输出变量，用于拼接所有数据块
output = ""
# 遍历流式响应的每个数据块
for chunk in response:
    # 从当前数据块中提取文本内容，添加空值兜底，避免拼接None
    chunk_str = chunk.choices[0].delta.content or ""
    output += chunk_str# 将当前chunk的content添加到output中
    # 往空占位符中更新内容（覆盖而非新增，避免重复渲染）
    response_message.chat_message("assistant").write(output)
# 流式输出结束后，将完整的回答添加到会话状态中
st.session_state.messages.append({"role": "assistant", "content": output})
```

## 5.3 优化后效果
运行代码后，流式输出变得非常流畅，文本会在**同一个助手消息框**内逐步追加，没有任何重复渲染，效果如下：

![在这里插入图片](https://i-blog.csdnimg.cn/direct/c0116ef0dfbe4498a45751117df94cfb.gif#pic_center)

# 六、完整最终代码（整合所有功能）
以下是整合了历史消息展示、会话记忆和流式输出的完整代码，可直接复制使用：
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

# 七、关键注意事项与避坑指南
## 7.1 必须处理空值
部分模型返回的数据包中，`delta.content`可能为`None`（如开始和结束的控制包），如果不处理，会导致拼接出`None`字符串。因此必须添加空值兜底：
```python
# 错误写法
chunk_str = chunk.choices[0].delta.content
# 正确写法（空值兜底）
chunk_str = chunk.choices[0].delta.content or ""
```

## 7.2 必须保存完整回答到会话状态
流式输出结束后，一定要将拼接好的完整回答`output`添加到`st.session_state.messages`中，否则：
- 刷新页面后，这条AI回答会丢失
- 后续的连续提问无法基于这条回答进行上下文关联

## 7.3 添加异常处理
流式输出过程中可能会出现网络中断、API限额耗尽、服务器错误等问题，建议添加异常处理，避免程序崩溃：
```python
try:
    response = client.chat.completions.create(
        model="deepseek-chat",
        messages=[
            {"role": "system", "content": system_prompt},
            *st.session_state.messages,
        ],
        stream=True
    )
    response_message = st.empty()
    output = ""
    for chunk in response:
        chunk_str = chunk.choices[0].delta.content or ""
        output += chunk_str
        response_message.chat_message("assistant").write(output)
    st.session_state.messages.append({"role": "assistant", "content": output})
except Exception as e:
    response_message.chat_message("assistant").write(f"😢 响应失败，请稍后重试\n错误信息：{str(e)}")
```

## 7.4 控制输出速度
如果模型返回数据的速度过快，前端渲染可能会出现卡顿，可以添加微小的延时，让输出更流畅：
```python
import time
for chunk in response:
    chunk_str = chunk.choices[0].delta.content or ""
    output += chunk_str
    response_message.chat_message("assistant").write(output)
    time.sleep(0.01)# 添加10毫秒延时，优化渲染体验
```

# 八、效果对比与总结
## 8.1 各阶段效果对比
| 阶段                | 存在的问题                       | 优化后效果                                           |
| ------------------- | -------------------------------- | ---------------------------------------------------- |
| 非流式输出          | 等待时间长、无实时反馈、体验生硬 | -                                                    |
| 直接开启stream=True | 程序报错、无法正常运行           | -                                                    |
| 第一版流式实现      | 文本重复渲染、页面混乱           | -                                                    |
| 最终优化版          | -                                | 文本实时追加、单消息框渲染、体验流畅、支持上下文关联 |

## 8.2 核心总结
本文从问题引入、踩坑尝试、结构分析到最终优化，完整讲解了AI智能助手流式输出的实现过程，核心要点总结：
1. 流式输出和非流式输出的返回数据格式完全不同，不能混用取值方式
2. 流式输出需要遍历`Stream`对象，逐个提取数据块并拼接成完整内容
3. 使用`st.empty()`组件可以完美解决Streamlit中流式输出的重复渲染问题
4. 必须处理空值、保存完整回答到会话状态、添加异常处理，保证程序的稳定性

# 九、核心总结：流式输出实现核心速查表
为了方便后续开发时快速查阅，整理了流式输出实现的核心速查表：

| 核心环节     | 关键操作                                                  | 核心要点                       |
| ------------ | --------------------------------------------------------- | ------------------------------ |
| 开启流式输出 | 设置`stream=True`                                         | 与非流式输出返回格式完全不同   |
| 数据提取     | 遍历`Stream`对象，提取`chunk.choices[0].delta.content`    | 必须添加空值兜底`or ""`        |
| 解决重复渲染 | 使用`st.empty()`创建占位符                                | 所有更新都在同一个占位符中进行 |
| 会话状态保存 | 流式结束后将完整`output`添加到`st.session_state.messages` | 否则会丢失历史和上下文         |
| 稳定性优化   | 添加try-except异常处理                                    | 捕获网络错误、API错误等        |
| 体验优化     | 添加微小延时`time.sleep(0.01)`                            | 避免输出过快导致渲染卡顿       |

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163831596>
