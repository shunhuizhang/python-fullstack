

# Python全栈入门到实战【AI实战篇 03】利用APIFOX调用DeepSeek API全流程实战，第一个AI项目接口调试核心
上一篇《AI实战篇 02》中，我们已经完成了APIFOX的安装、账号登录、项目创建与基础接口调试，掌握了全栈开发必备的接口调试工具使用方法。本篇作为第一个AI项目的第三篇，我们将聚焦核心实战环节，利用APIFOX完整调试DeepSeek官方聊天接口，从接口核心信息解析、基础参数配置、非流式输出调试、中文提示词适配到流式输出功能实现，一步步带你吃透DeepSeek API的完整调用逻辑，为后续Python代码封装、智能助手功能开发打下坚实基础。

本文为Python全栈开发者与AI入门者量身打造，全程采用实战驱动教学，每一个参数都有详细说明，每一个步骤都有配套截图，同时覆盖所有常见报错的排查与解决方案，即使是完全没有接口调试经验的同学，也能跟着一步步完成所有调试，成功实现AI接口的可视化调用。

本节核心学习内容：
- DeepSeek API核心信息解析：官方文档查阅、核心接口地址、必选请求头与请求体参数详解
- APIFOX接口基础配置：请求方式设置、接口地址填写、请求头参数配置
- 非流式输出基础调试：请求体配置、发送请求、响应结果验证
- 中文提示词适配调试：自定义中文系统提示词、用户提示词，验证中文响应效果
- 流式输出功能调试：流式参数配置、数据包解析、自动合并结果验证
- 常见报错排查：401权限错误、400参数错误、请求超时等问题的根因分析与解决方案
- 核心总结：DeepSeek API调用参数速查表，方便开发时快速查阅

# 一、DeepSeek API接口核心信息解析
在使用APIFOX调试接口之前，我们首先需要从DeepSeek官方文档中获取接口的所有核心配置信息，这是接口调用成功的前提，任何参数的错误都会导致调用失败。

## 1.1 官方文档与核心接口地址
DeepSeek的所有API接口规范都在官方文档中有详细说明，我们需要先找到聊天完成接口的核心信息。

### 操作步骤
1. 打开浏览器，访问DeepSeek API官方中文文档地址：`https://api-docs.deepseek.com/zh-cn`；
2. 在左侧导航栏找到「API 参考」→「聊天完成」，进入聊天接口的详细文档页面；
3. 下滑页面找到curl调用示例，从中提取核心接口地址：`https://api.deepseek.com/chat/completions`。

> ⚠️ 重要提示：该地址是DeepSeek聊天接口的唯一官方地址，不可修改、不可拼写错误，否则会导致请求失败。

![在这里插入图片](https://i-blog.csdnimg.cn/direct/d629cac1906242ab9a2e9ee88962c2de.png#pic_center)

## 1.2 必选请求头参数说明
所有调用DeepSeek API的请求，都必须携带以下两个请求头参数，缺一不可，且格式必须严格符合规范：

| 请求头参数名  | 参数值                      | 详细说明                                                     |
| ------------- | --------------------------- | ------------------------------------------------------------ |
| Content-Type  | application/json            | 固定取值，用于声明请求体的格式为JSON，不可修改、不可遗漏     |
| Authorization | Bearer 你的DeepSeek API Key | 身份鉴权核心参数，`Bearer`和API Key之间**必须保留1个英文空格**，API Key替换为你自己在DeepSeek平台创建的密钥 |

> 新手避坑提示：很多新手会遗漏`Bearer`和API Key之间的空格，或者写成中文空格，这是导致401权限错误的最常见原因。

## 1.3 核心请求体参数详解
请求体是接口调用的核心，采用标准JSON格式，包含模型选择、对话消息、输出模式等核心配置，基础示例如下：
```json
{
  "model": "deepseek-chat",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ],
  "stream": false
}
```

下面我们对每一个核心参数进行详细讲解：

### 1. model：模型类型参数
用于指定调用的DeepSeek大模型，目前支持两个核心模型：
- `deepseek-chat`：普通对话模型，非思考模式，响应速度快、token成本低，适用于日常对话、内容生成、文本总结等常规场景；
- `deepseek-reasoner`：推理模型，思考模式，擅长逻辑推理、数学计算、代码编写、复杂问题分析，响应耗时更长、token成本更高。

### 2. messages：对话消息列表
用于传递对话上下文和提示词，是一个数组，数组中的每个元素是一个消息对象，包含`role`和`content`两个字段：
- `role: "system"`：系统提示词，用于设定AI的身份、性格、回答规则、输出格式等，会全程影响AI的所有回答，一个请求中最多只能有一个system消息；
- `role: "user"`：用户提示词，用于传递用户的提问、需求、指令等核心内容，是AI需要处理的具体任务；
- `content`：对应角色的具体文本内容，支持中英文，无强制字数限制。

### 3. stream：输出模式参数
用于控制接口的响应方式，是布尔类型，只能取`true`或`false`（必须全小写）：
- `stream: false`：非流式输出，接口会一次性返回完整的响应结果，适合接口调试、后端服务调用；
- `stream: true`：流式输出，接口会分段返回响应文本片段，实现打字机效果，适合前端实时对话页面。

非流式输出效果：
![在这里插入图片](https://i-blog.csdnimg.cn/direct/cc551201829d4a898396ae699ad86ce9.gif#pic_center)

流式输出效果：
![在这里插入图片](https://i-blog.csdnimg.cn/direct/2339d440f30344afa9720d1df02387ae.gif#pic_center)

# 二、APIFOX基础接口配置
获取完接口核心信息后，我们打开APIFOX，开始配置DeepSeek聊天接口的基础信息，这是调试的第一步。

## 2.1 接口基础信息填写
### 操作步骤
1. 启动APIFOX软件，登录账号后，进入我们之前创建的「DeepSeek智能助手项目」；
2. 在左侧「接口管理」板块点击「+」号，选择「新建接口」；
3. 在接口创建页面填写基础信息：
   - 接口名称：填写「DeepSeek聊天接口」；
   - 接口描述：填写「调用DeepSeek大模型实现AI对话功能」；
   - 请求方式：**必须选择POST**，DeepSeek聊天接口仅支持POST请求，不支持GET等其他方式；
4. 点击「保存」按钮，进入接口调试页面。

![在这里插入图片](https://i-blog.csdnimg.cn/direct/bfd5d2ff471e460e8a12b074da69fee5.png#pic_center)

## 2.2 请求头参数配置
基础信息填写完成后，我们配置必选的两个请求头参数，这是接口鉴权和格式声明的关键。

### 操作步骤
1. 在接口调试页面，切换到「Headers」（请求头）标签页；
2. 点击「添加参数」按钮，新增两条请求头参数，严格按照以下内容填写：
   | 参数名        | 参数值                                     |
   | ------------- | ------------------------------------------ |
   | Content-Type  | application/json                           |
   | Authorization | Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |
3. 填写完成后，再次核对参数名和参数值，确保无拼写错误、无多余空格、无中文符号。

> 重要提示：将`sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`替换为你自己的DeepSeek API Key，注意`Bearer`和Key之间必须有一个英文空格。

# 三、非流式输出基础调试
完成基础配置后，我们先进行最基础的非流式输出调试，验证接口是否能正常调用，这是所有后续调试的基础。

## 3.1 请求体参数配置
### 操作步骤
1. 在接口调试页面，切换到「Body」标签页；
2. 在Body格式选项中，选择「JSON」，确保请求体格式与接口要求完全匹配；
3. 在JSON编辑框中，粘贴官方文档的基础请求参数示例：
   ```json
   {
     "model": "deepseek-chat",
     "messages": [
       {"role": "system", "content": "You are a helpful assistant."},
       {"role": "user", "content": "Hello!"}
     ],
     "stream": false
   }
   ```
4. 粘贴完成后，检查JSON格式是否正确：确保所有括号、引号都是英文符号，最后一个字段末尾没有逗号，布尔值`false`为全小写。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9120ca115551467189ec8d0d05051ebd.png#pic_center)

## 3.2 发送请求与响应结果验证
### 操作步骤
1. 确认所有配置信息（地址、请求头、请求体）无误后，点击页面中的「发送」按钮，发起接口请求；
2. 等待请求完成（通常1~3秒），在页面下方的「响应」区域查看返回结果；
3. 验证标准：
   - HTTP状态码为`200 OK`，说明请求成功；
   - 响应数据中包含`choices[0].message.content`字段，且内容为AI生成的英文回答，说明接口调用完全成功。

![在这里插入图片](https://i-blog.csdnimg.cn/direct/70c7fcc4ee05468d833c781bfd92b409.png#pic_center)

# 四、中文提示词适配调试
基础英文调试成功后，我们将提示词修改为中文，适配中文使用场景，这是我们实际项目中最常用的方式。

## 4.1 中文提示词参数修改
### 操作步骤
1. 回到Body的JSON编辑框，修改`messages`列表中的`content`内容，将英文提示词替换为自定义的中文内容，示例如下：
   ```json
   {
     "model": "deepseek-chat",
     "messages": [
       {"role": "system", "content": "我是一名专业的Python编程老师，名字叫小辉辉，有10年的教学经验，擅长用通俗易懂的语言讲解编程知识"},
       {"role": "user", "content": "你是谁？能帮助我学习Python吗？"}
     ],
     "stream": false
   }
   ```
2. 保持`model`和`stream`参数不变，再次检查JSON格式是否正确，确保没有中文标点、引号缺失等问题。

![在这里插入图片](https://i-blog.csdnimg.cn/direct/bf84edc74a384ca6a8e4cac180ace374.png#pic_center)

## 4.2 中文响应结果验证
### 操作步骤
1. 点击「发送」按钮，重新发起接口请求；
2. 查看响应结果，确认接口返回了符合中文提示词设定的内容：AI会介绍自己是Python编程老师小辉辉，并说明能提供的Python学习帮助；
3. 验证中文显示正常，无乱码、无语义偏差，说明中文提示词适配成功。

> 全栈开发技巧：你可以根据自己的需求，自定义系统提示词，让AI扮演不同的角色（比如代码助手、文案助手、翻译助手），实现不同的功能。

# 五、流式输出功能调试
完成非流式输出调试后，我们测试流式输出功能，这是实现前端实时对话打字机效果的核心，也是智能助手项目必备的功能。

## 5.1 流式输出参数配置
### 操作步骤
1. 回到Body的JSON编辑框，保持其他所有参数不变，仅将`stream`参数的取值从`false`修改为`true`：
   ```json
   {
     "model": "deepseek-chat",
     "messages": [
       {"role": "system", "content": "我是一名专业的Python编程老师，名字叫小辉辉，有10年的教学经验，擅长用通俗易懂的语言讲解编程知识"},
       {"role": "user", "content": "你是谁？能帮助我学习Python吗？"}
     ],
     "stream": true
   }
   ```
2. 确认参数修改无误，JSON格式规范，布尔值`true`为全小写（不能写成True）。

![在这里插入图片](https://i-blog.csdnimg.cn/direct/3ec6875d92ab42f4888b3f6eebb6dd64.png#pic_center)

## 5.2 流式数据包解析
### 操作步骤
1. 点击「发送」按钮，发起流式请求；
2. 请求发送后，APIFOX会实时展示接口返回的多个数据包，每个数据包仅包含一段响应文本片段，实现逐字输出的效果；
3. 点击任意一个数据包，可查看该片段的详细JSON结构，流式输出的完整响应结果，就是将所有数据包的`choices[0].delta.content`字段按顺序拼接而成。

![在这里插入图片](https://i-blog.csdnimg.cn/direct/492d6354e3c34e688215170b99cdb215.png#pic_center)

## 5.3 自动合并结果验证
APIFOX提供了便捷的自动合并功能，无需手动拼接数据包，即可查看完整的响应结果：
1. 在流式响应结果区域，点击右上角的「自动合并」按钮；
2. 查看合并后的完整响应内容，确认合并后的文本与非流式输出的完整内容完全一致，说明流式输出功能调试成功。

![在这里插入图片](https://i-blog.csdnimg.cn/direct/1d0b8d30be5843f1a2be5854568ec124.png#pic_center)

# 六、常见问题与注意事项
新手在调试DeepSeek API的过程中，90%的问题都集中在以下场景，这里整理了对应的根因分析和详细解决方案，遇到问题可以直接对照排查。

## 6.1 常见报错与解决方案
---
### 报错1：401 Unauthorized（权限错误）
#### 根因分析
1. API Key填写错误，多复制了空格或字符；
2. `Authorization`参数格式错误，遗漏了`Bearer`前缀，或`Bearer`和API Key之间没有空格；
3. API Key已被删除、过期或禁用；
4. DeepSeek账号余额不足，无法调用API。

#### 解决方案
1. 重新复制正确的API Key，更新到请求头中；
2. 检查`Authorization`参数格式，确保是`Bearer 你的API Key`（中间有一个英文空格）；
3. 登录DeepSeek API开放平台，确认API Key状态正常；
4. 检查账号余额，余额不足时进行充值。

---
### 报错2：400 Bad Request（参数错误）
#### 根因分析
1. 请求体JSON格式语法错误，比如缺少逗号、括号不匹配、使用了中文标点；
2. 必填参数缺失，比如没有`model`或`messages`参数；
3. 参数取值错误，比如布尔值写成了大写`True/False`、模型名拼写错误；
4. 请求头`Content-Type`设置错误，不是`application/json`。

#### 解决方案
1. 使用在线JSON校验工具检查请求体的格式，修正语法错误；
2. 核对所有必填参数是否完整，确保没有遗漏；
3. 确认`model`、`stream`等参数的取值符合官方规范，布尔值必须全小写；
4. 检查请求头，确保`Content-Type`为`application/json`。

---
### 报错3：请求无响应/超时
#### 根因分析
1. 电脑网络环境异常，无法连接外网；
2. 开启了VPN、代理软件，导致请求被拦截；
3. 接口地址填写错误，拼写错误或遗漏了`https://`前缀；
4. DNS解析异常，无法解析DeepSeek的域名。

#### 解决方案
1. 检查电脑网络是否正常，打开浏览器访问其他网站测试；
2. 关闭所有VPN和代理软件，重新发送请求；
3. 仔细核对接口地址，确保是`https://api.deepseek.com/chat/completions`；
4. 切换手机热点网络，再次尝试调用。

## 6.2 核心使用注意事项
1. API Key属于高敏感信息，绝对不能泄露给他人，也不能上传到GitHub、Gitee等公开代码平台，否则会导致账号被盗用，产生不必要的费用损失；
2. JSON格式中，布尔值`true`和`false`必须为全小写，不能使用Python中的大写`True/False`，否则会导致参数解析失败；
3. 思考模式`deepseek-reasoner`的响应耗时更长，token计费也更高，调试时优先使用`deepseek-chat`模型；
4. 正式项目中，根据场景选择合适的输出模式：后端服务调用使用非流式输出，前端对话页面使用流式输出；
5. 所有错误码的详细说明，都可以在DeepSeek官方文档的「错误码」章节查询，遇到陌生错误时优先查阅官方文档。

# 七、核心总结：DeepSeek API调用参数速查表
为了方便后续开发时快速查阅，整理了DeepSeek聊天接口的核心参数速查表：
| 参数类型 | 参数名        | 可选值                                    | 核心说明                                 |
| -------- | ------------- | ----------------------------------------- | ---------------------------------------- |
| 接口地址 | -             | https://api.deepseek.com/chat/completions | 固定值，仅支持POST请求                   |
| 请求头   | Content-Type  | application/json                          | 固定值，声明请求体格式                   |
| 请求头   | Authorization | Bearer 你的API Key                        | 身份鉴权，Bearer和Key之间必须有空格      |
| 请求体   | model         | deepseek-chat / deepseek-reasoner         | 模型选择，普通对话用chat，推理用reasoner |
| 请求体   | messages      | 数组格式，包含system和user消息            | 对话上下文和提示词                       |
| 请求体   | stream        | true / false                              | 输出模式，false一次性返回，true流式输出  |

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163674724>
