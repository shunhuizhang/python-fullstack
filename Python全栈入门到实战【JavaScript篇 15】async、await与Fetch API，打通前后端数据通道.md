
# Python全栈入门到实战【JavaScript篇 15】async/await与Fetch API，打通前后端数据通道
上一篇《JavaScript篇 14》中，我们攻克了Promise——用链式调用替代回调地狱，让异步代码变得扁平化、可组合。但Promise链仍然是"回调"的思维方式：你必须在then中写下一步的操作。如果有一种方式，让异步代码看起来像同步代码一样"从上到下顺序执行"，会不会更好？

这就是**async/await**的使命。它是Promise的语法糖，让异步代码写起来像同步代码一样直观——你用`await`等待一个异步操作完成，代码会"暂停"在这里等结果，但不会阻塞整个线程。同时，本篇还将学习**Fetch API**——现代浏览器内置的HTTP请求接口，用于前后端数据交互。作为Python开发者，你对`await`的概念应该不陌生（Python也有async/await），而Fetch相当于Python的`requests`库（但更底层）。

学完本篇后，你就彻底掌握了JS与后端通信的能力，为最终的实战项目做好全部技术准备。

本节核心学习内容：
1.  async/await：让异步代码像同步代码一样写
2.  try-catch处理async/await错误（对照Python的try-except-await）
3.  Fetch API：前后端通信的现代方式
4.  GET请求：从后端获取数据并解析JSON
5.  POST请求：向服务器提交数据
6.  HTTP状态码和网络错误的区别处理
7.  async/await vs Promise.then：何时用哪种

# 一、async/await：让异步代码回归直觉
## 1.1 async函数
在函数声明前加上`async`关键字，这个函数就会自动返回一个Promise：

```javascript
// 普通函数声明前加async
async function fetchData() {
    return "数据"; // 自动包装为 Promise.resolve("数据")
}

// 等价于
function fetchData() {
    return Promise.resolve("数据");
}

// async函数的返回值被自动包装为Promise
fetchData().then(data => console.log(data)); // "数据"

// async也可以用于箭头函数
const fetchDataAsync = async () => {
    return "数据";
};
```

## 1.2 await：等待Promise完成
`await`关键字只能在`async`函数中使用，它的作用是等待一个Promise完成并**直接返回resolve的值**：

```javascript
// 模拟异步操作
function fetchUser(userId) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ id: userId, name: "张三" });
        }, 1500);
    });
}

// Promise.then写法
function getUserWithThen() {
    return fetchUser(1001)
        .then(user => {
            console.log(user);
            return user.name;
        });
}

// async/await写法（功能完全相同，但更像同步代码）
async function getUserWithAwait() {
    const user = await fetchUser(1001); // 等待Promise完成，直接取出结果
    console.log(user);
    return user.name; // 自动包装为Promise
}
```

**await的执行逻辑**：
```javascript
async function demo() {
    console.log("1. 开始");

    // await会暂停这个async函数的执行，等待Promise完成
    // 但不会阻塞主线程！其他代码可以继续运行
    const result = await fetchUser(1001);

    console.log("2. 拿到结果：", result); // 等Promise完成后才执行
    console.log("3. 结束");
}
```

**Python对照**：
```python
import asyncio

async def fetch_user(user_id):
    await asyncio.sleep(1.5)
    return {"id": user_id, "name": "张三"}

async def demo():
    print("1. 开始")
    result = await fetch_user(1001)
    print(f"2. 拿到结果：{result}")
    print("3. 结束")
```

## 1.3 错误处理：try-catch
在async函数中，用`try-catch`捕获错误（而不是`.catch()`链）：

```javascript
async function getUser() {
    try {
        const user = await fetchUser(1001); // 如果这里抛出错误
        console.log("用户信息：", user);
        return user;
    } catch (error) {
        console.error("获取用户失败：", error.message);
        // 可以返回默认值或重新抛出错误
        return null;
    } finally {
        console.log("请求结束（无论成败）");
    }
}
```

**Python对照**：
```python
async def get_user():
    try:
        user = await fetch_user(1001)
        print(f"用户信息：{user}")
        return user
    except Exception as e:
        print(f"获取用户失败：{e}")
        return None
    finally:
        print("请求结束（无论成败）")
```

## 1.4 多个异步操作：串行 vs 并行
```javascript
// 串行执行：一个一个来（两请求总耗时 = 1.5s + 1.5s = 3s）
async function serial() {
    const user = await fetchUser(1001);   // 等1.5s
    const orders = await fetchOrders(1001); // 再等1.5s
    return { user, orders };
}

// 并行执行：同时发起（两请求总耗时 = max(1.5s, 1.5s) = 1.5s）
async function parallel() {
    // 同时发起两个请求（都不await，先拿到Promise对象）
    const userPromise = fetchUser(1001);
    const ordersPromise = fetchOrders(1001);

    // 然后一起等待
    const [user, orders] = await Promise.all([userPromise, ordersPromise]);
    return { user, orders };
}
```

> 关键理解：`const userPromise = fetchUser(1001)`时请求就已经发出了（不写await请求也会发出），await只是等待结果。并行就是把多个Promise先都创建出来，然后一起等待。

# 二、Fetch API：前后端通信的现代方式
## 2.1 什么是Fetch
Fetch API是浏览器内置的现代HTTP请求接口，用于向服务器发送请求并获取响应。它是`XMLHttpRequest`（XHR）的替代品。学习Fetch的目标非常明确——用JS从后端API获取数据，并渲染到前端页面。这标志着前后端数据通道的打通，是实现全栈开发的核心技能。

## 2.2 基本语法
```javascript
// 基本的GET请求
fetch(url)
    .then(response => response.json())  // 解析JSON响应体
    .then(data => console.log(data))    // 使用数据
    .catch(error => console.error(error)); // 处理错误
```

**Python对照**：
```python
import requests
response = requests.get(url)
data = response.json()
print(data)
```

## 2.3 GET请求：获取数据
```javascript
// 方式1：Promise.then链
function getUsers() {
    fetch("https://jsonplaceholder.typicode.com/users")
        .then(response => {
            // response是Response对象，包含状态码、头部等
            if (!response.ok) {
                throw new Error(`HTTP错误：${response.status}`);
            }
            return response.json(); // response.json()本身也是异步的（返回Promise）
        })
        .then(data => {
            console.log("获取到用户列表：", data);
            // 在这里渲染到页面
        })
        .catch(error => {
            console.error("请求失败：", error.message);
        });
}

// 方式2：async/await（推荐）
async function getUsersAsync() {
    try {
        const response = await fetch("https://jsonplaceholder.typicode.com/users");
        if (!response.ok) {
            throw new Error(`HTTP错误：${response.status}`);
        }
        const data = await response.json();
        console.log("获取到用户列表：", data);
        return data;
    } catch (error) {
        console.error("请求失败：", error.message);
        return [];
    }
}
```

**Fetch GET请求带参数（URL查询参数）**：
```javascript
// 构建带参数的URL
const baseUrl = "https://jsonplaceholder.typicode.com/posts";

// 方式1：手动拼接（简单场景）
const url1 = `${baseUrl}?userId=1&_limit=10`;

// 方式2：使用URLSearchParams（推荐）
const params = new URLSearchParams({
    userId: 1,
    _limit: 10,
    _sort: "id",
    _order: "desc"
});
const url2 = `${baseUrl}?${params.toString()}`;

const response = await fetch(url2);
const posts = await response.json();
```

## 2.4 POST请求：提交数据
POST请求用于向服务器发送数据（创建新资源）：

```javascript
async function createPost(title, body, userId) {
    try {
        const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
            method: "POST",                     // 请求方法
            headers: {
                "Content-Type": "application/json" // 告诉服务器数据类型是JSON
            },
            body: JSON.stringify({              // 请求体（JS对象 → JSON字符串）
                title,
                body,
                userId
            })
        });

        if (!response.ok) {
            throw new Error(`HTTP错误：${response.status}`);
        }

        const data = await response.json();
        console.log("创建成功：", data);
        return data;
    } catch (error) {
        console.error("创建失败：", error.message);
    }
}

// 调用
createPost("JavaScript异步编程", "async/await详解", 1);
```

**Python对照**：
```python
import requests
import json

response = requests.post(
    url,
    headers={"Content-Type": "application/json"},
    data=json.dumps({"title": "...", "body": "...", "userId": 1})
)
data = response.json()
```

## 2.5 HTTP状态码 vs 网络错误
这是Fetch API中的一个重要概念：

```javascript
async function fetchWithErrorHandling(url) {
    try {
        const response = await fetch(url);

        // fetch只在网络错误时reject（如DNS解析失败、连接超时）
        // HTTP错误（如404、500）不会导致fetch reject！
        // 需要手动检查 response.ok
        if (!response.ok) {
            throw new Error(`服务器返回错误：${response.status} ${response.statusText}`);
        }
        return await response.json();

    } catch (error) {
        // 这里捕获两种错误：
        // 1. 网络错误（fetch reject）：断网、域名不存在等
        // 2. 手动抛出的HTTP错误（response.ok为false）
        console.error("请求失败：", error.message);
        return null;
    }
}
```

| 情况 | fetch是否reject | 示例 |
|------|----------------|------|
| 网络断开 | ✓ | `TypeError: Failed to fetch` |
| DNS解析失败 | ✓ | `TypeError: Failed to fetch` |
| 404 Not Found | ✗（需手动检查） | `response.status === 404` |
| 500 Server Error | ✗（需手动检查） | `response.status === 500` |
| 200 OK | ✗ | `response.ok === true` |

## 2.6 Response对象的常用属性和方法
```javascript
const response = await fetch("https://jsonplaceholder.typicode.com/users/1");

// 属性
console.log(response.ok);         // true（200-299范围）
console.log(response.status);     // 200（HTTP状态码）
console.log(response.statusText); // "OK"（状态文本）
console.log(response.headers);    // Headers对象

// 方法（用于解析响应体）
const jsonData = await response.json();  // 解析为JSON对象（最常用）
const textData = await response.text();  // 解析为纯文本
const blobData = await response.blob();  // 解析为二进制（图片、文件）
```

# 三、实战：从API获取数据并渲染到页面
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>用户列表 - Fetch API实战</title>
    <style>
        body { font-family: sans-serif; max-width: 800px; margin: 30px auto; padding: 0 20px; }
        h2 { color: #333; }
        .loading {
            text-align: center;
            padding: 40px;
            color: #999;
        }
        .loading::after {
            content: "";
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 2px solid #ddd;
            border-top-color: #3498db;
            border-radius: 50%;
            animation: spin 0.8s linear infinite;
            margin-left: 8px;
            vertical-align: middle;
        }
        @keyframes spin { to { transform: rotate(360deg); } }
        .error {
            text-align: center;
            padding: 40px;
            color: #e74c3c;
            background: #fde8e8;
            border-radius: 8px;
        }
        .user-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
            gap: 15px;
        }
        .user-card {
            background: white;
            border: 1px solid #eee;
            border-radius: 8px;
            padding: 18px;
            transition: box-shadow 0.3s;
        }
        .user-card:hover {
            box-shadow: 0 3px 15px rgba(0,0,0,0.1);
        }
        .user-card h3 { margin: 0 0 8px; color: #333; font-size: 17px; }
        .user-card p { margin: 4px 0; font-size: 14px; color: #666; }
        .user-card .label { color: #999; font-size: 12px; }
        .retry-btn {
            display: block;
            margin: 20px auto;
            padding: 10px 25px;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h2>用户列表</h2>
    <div id="app"></div>

    <script>
        const app = document.querySelector("#app");

        async function fetchUsers() {
            // 显示加载状态
            app.innerHTML = '<div class="loading">正在加载用户列表</div>';

            try {
                const response = await fetch("https://jsonplaceholder.typicode.com/users");

                if (!response.ok) {
                    throw new Error(`服务器错误：${response.status}`);
                }

                const users = await response.json();
                renderUsers(users);
            } catch (error) {
                app.innerHTML = `
                    <div class="error">
                        <p>加载失败：${error.message}</p>
                        <button class="retry-btn" onclick="fetchUsers()">重新加载</button>
                    </div>
                `;
            }
        }

        function renderUsers(users) {
            if (users.length === 0) {
                app.innerHTML = '<p style="text-align:center;color:#999;">暂无用户数据</p>';
                return;
            }

            const html = users.map(user => `
                <div class="user-card">
                    <h3>${user.name}</h3>
                    <p><span class="label">用户名：</span>${user.username}</p>
                    <p><span class="label">邮箱：</span>${user.email}</p>
                    <p><span class="label">电话：</span>${user.phone}</p>
                    <p><span class="label">网站：</span>${user.website}</p>
                    <p><span class="label">公司：</span>${user.company.name}</p>
                </div>
            `).join("");

            app.innerHTML = `<div class="user-grid">${html}</div>`;
        }

        // 页面加载时获取数据
        fetchUsers();
    </script>
</body>
</html>
```

# 四、async/await vs Promise.then
| 维度 | async/await | Promise.then |
|------|------------|-------------|
| 代码风格 | 同步风格，线性阅读 | 链式调用 |
| 错误处理 | try-catch（传统方式） | .catch() |
| 条件判断 | 自然写在if/else中 | 需要在then中嵌套 |
| 循环中的异步 | for循环正常使用await | 需用递归或Promise.all |
| 调试 | 可以逐行调试 | 链式调试不便 |
| 学习曲线 | 平缓（对Python开发者尤其友好） | 中等 |

```javascript
// 条件逻辑中的对比
// Promise.then：需要嵌套
function getDataWithThen(needDetails) {
    return fetchUser().then(user => {
        if (needDetails) {
            return fetchUserDetails(user.id)
                .then(details => ({ ...user, details }));
        }
        return user;
    });
}

// async/await：像同步代码一样写
async function getDataWithAwait(needDetails) {
    const user = await fetchUser();
    if (needDetails) {
        const details = await fetchUserDetails(user.id);
        return { ...user, details };
    }
    return user;
}
```

# 五、常见误区与避坑指南
1.  **fetch在非HTTP错误时不reject**：这是Fetch最大的"坑"。`fetch()`只有在网络故障时才reject（如断网），HTTP 404/500这类错误不会reject。你必须在then中或await后手动检查`response.ok`。

2.  **忘记await response.json()**：`response.json()`本身也返回一个Promise，必须用await或.then获取结果。直接console.log(response.json())会输出一个pending的Promise。

3.  **并行请求误写成串行**：如果两个请求之间没有依赖关系，不要顺序写await——它们会串行执行。应该先创建Promise对象，再用Promise.all等待：
    ```javascript
    // ❌ 串行（3秒）
    const user = await fetchUser(1);
    const posts = await fetchPosts(1);
    
    // ✓ 并行（1.5秒）
    const [user, posts] = await Promise.all([
        fetchUser(1),
        fetchPosts(1)
    ]);
    ```

4.  **忘记设置Content-Type header**：在POST请求中，如果body是JSON，必须设置`Content-Type: application/json`，否则服务器可能无法正确解析请求体。

5.  **在非async函数中使用await**：顶层await（不在async函数内）在模块中支持，但在普通script标签中不支持。确保await在async函数内使用。

6.  **async函数的调用者也需要处理异步**：async函数返回Promise，调用者要么await它，要么.then处理它。如果直接忽略返回值，其中的错误会变成"未捕获的Promise rejection"。

# 六、核心总结
## async/await速查
```javascript
async function demo() {
    try {
        const result = await somePromise();     // 等待Promise
        const parallel = await Promise.all([    // 并行等待
            promise1(), promise2()
        ]);
        return result;
    } catch (error) {
        console.error(error);
    }
}
```

## Fetch速查
```javascript
// GET
const res = await fetch(url);
const data = await res.json();

// POST
const res = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ key: "value" })
});
const data = await res.json();

// 安全封装
async function safeFetch(url) {
    const res = await fetch(url);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
}
```

## JS vs Python异步对照
| 操作 | JS | Python |
|------|-----|--------|
| 声明异步函数 | `async function` | `async def` |
| 等待Promise/Future | `await promise` | `await future` |
| 错误处理 | `try-catch` | `try-except` |
| 并行等待 | `Promise.all([p1, p2])` | `asyncio.gather(t1, t2)` |
| HTTP GET | `fetch(url)` | `requests.get(url)` |
| HTTP POST | `fetch(url, {method, headers, body})` | `requests.post(url, json=data)` |

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
