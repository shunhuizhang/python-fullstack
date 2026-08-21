
# Python全栈入门到实战【JavaScript篇 11】DOM操作进阶，动态构建与批量渲染网页元素
上一篇《JavaScript篇 10》中，我们学习了DOM操作的基础——获取元素、修改内容和样式。但你可能会想：如果用户发布了新评论，我如何把这个新评论"追加"到评论列表的末尾？如果用户点了删除按钮，我如何把这个元素从页面上"移除"？如果后端返回了100条数据，我如何高效地把它们全部渲染出来？

这些场景需要的不是"修改已有元素"，而是**动态创建和删除元素**——这才是真实项目中DOM操作的核心。一个现代Web应用的页面不是"一次性画好"的，而是由JS根据数据**动态构建**出来的。本篇将系统学习DOM的增删改查进阶操作，包括创建元素、追加/插入/删除节点、操作表单、以及批量渲染时的性能优化技巧。

本节核心学习内容：
1.  创建和追加元素：createElement + appendChild + insertBefore
2.  删除和替换元素：removeChild、remove、replaceChild
3.  克隆节点：cloneNode
4.  操作表单元素：value、checked、selectedIndex
5.  批量渲染性能优化：DocumentFragment一次性插入
6.  获取元素尺寸和位置：getBoundingClientRect与offset系列
7.  综合实战：动态新增/删除列表项、批量生成表格

# 一、创建和追加元素
## 1.1 createElement：创建元素
`document.createElement()`创建一个新的HTML元素节点，但它**不会**自动出现在页面上——需要手动插入到DOM树中的某个位置。

```javascript
// 创建一个 div 元素
const div = document.createElement("div");

// 设置属性和内容
div.className = "new-box";
div.id = "box-001";
div.textContent = "我是新创建的元素";
div.setAttribute("data-id", "1001");

// 设置样式
div.style.padding = "20px";
div.style.backgroundColor = "#f0f0f0";

console.log(div); // <div class="new-box" id="box-001">...</div>
// 但此时页面上还看不到它！
```

## 1.2 appendChild：追加到末尾
`parent.appendChild(child)`将新元素追加到父元素的**最后面**：

```javascript
// 获取父容器
const container = document.querySelector("#list");

// 创建新元素
const newItem = document.createElement("li");
newItem.textContent = "新添加的项";

// 追加到容器末尾
container.appendChild(newItem);
```

## 1.3 insertBefore：插入到指定位置
`parent.insertBefore(newNode, referenceNode)`将新元素插入到指定参考节点的**前面**：

```javascript
const list = document.querySelector("#list");
const items = list.querySelectorAll("li");

// 创建新元素
const newItem = document.createElement("li");
newItem.textContent = "插入到第二个元素前面";

// 插入到第二个元素之前
list.insertBefore(newItem, items[1]);
```

**Python类比**：类似`list.insert(index, item)`，但JS是基于DOM引用而不是基于索引。

## 1.4 现代替代方案：insertAdjacentHTML / insertAdjacentElement
这两个方法更灵活，可以指定插入位置：

```javascript
const container = document.querySelector("#content");

// insertAdjacentHTML：直接用HTML字符串插入
container.insertAdjacentHTML("beforeend", "<p>追加到末尾</p>");
container.insertAdjacentHTML("afterbegin", "<p>追加到开头</p>");
container.insertAdjacentHTML("beforebegin", "<p>在元素之前</p>");
container.insertAdjacentHTML("afterend", "<p>在元素之后</p>");

// insertAdjacentElement：插入DOM元素
const newDiv = document.createElement("div");
container.insertAdjacentElement("beforeend", newDiv);
```

**位置参数说明**：
```
beforebegin  →  <div id="container">
afterbegin   →      <p>在开头插入</p>
                     ...原有内容...
beforeend    →      <p>在末尾插入</p>
afterend     →  </div>
```

# 二、删除和替换元素
## 2.1 removeChild：通过父元素删除
```javascript
const parent = document.querySelector("#list");
const child = parent.querySelector(".item-to-remove");
parent.removeChild(child);
```

## 2.2 remove()：直接删除自己（推荐）
ES6引入了`remove()`方法，元素可以直接删除自己，不需要通过父元素：

```javascript
const element = document.querySelector(".item-to-remove");
element.remove();
```

## 2.3 实战：删除一个列表中的某一项
```html
<ul id="task-list">
    <li>
        学习Python <button class="delete-btn">删除</button>
    </li>
    <li>
        学习JavaScript <button class="delete-btn">删除</button>
    </li>
    <li>
        学习MySQL <button class="delete-btn">删除</button>
    </li>
</ul>

<script>
    document.querySelectorAll(".delete-btn").forEach(btn => {
        btn.addEventListener("click", function() {
            // this 指向被点击的按钮
            const li = this.parentElement; // 找到按钮所在的<li>
            li.remove(); // 删除这个<li>
        });
    });
</script>
```

# 三、克隆节点
`cloneNode(deep)`方法用于复制一个节点。参数`deep`表示是否深拷贝（克隆所有子节点）：

```javascript
const template = document.querySelector(".card-template");

// 浅克隆：只拷贝元素自身，不拷贝子节点
const shallowCopy = template.cloneNode(false);

// 深克隆：拷贝元素及其所有子节点（99%的情况用这个）
const deepCopy = template.cloneNode(true);

// 克隆后修改内容，再追加到页面
deepCopy.querySelector(".title").textContent = "新的卡片标题";
document.querySelector(".card-list").appendChild(deepCopy);
```

克隆是创建重复结构的模板（如卡片列表、表格行）的高效方式。

# 四、操作表单元素
表单元素（input、select、textarea等）的属性和普通元素不同，有自己独特的API。

## 4.1 获取和设置表单值
```javascript
const input = document.querySelector("#username");
const textarea = document.querySelector("#bio");
const checkbox = document.querySelector("#agree");
const radio = document.querySelector("input[name=gender]:checked");

// 文本输入：使用 value
console.log(input.value);       // 获取输入值
input.value = "张三";           // 设置输入值

// 文本域
textarea.value = "个人简介";

// 复选框和单选框：使用 checked
console.log(checkbox.checked);  // true/false
checkbox.checked = true;        // 勾选

// 获取选中的单选按钮值
const gender = document.querySelector("input[name=gender]:checked")?.value;
```

## 4.2 下拉选择框（select）
```html
<select id="city">
    <option value="beijing">北京</option>
    <option value="shanghai" selected>上海</option>
    <option value="guangzhou">广州</option>
</select>
```

```javascript
const select = document.querySelector("#city");

// 获取选中的值
console.log(select.value); // "shanghai"

// 设置选中项
select.value = "guangzhou";

// 获取选中的索引
console.log(select.selectedIndex); // 2

// 获取选中的option元素
const selectedOption = select.options[select.selectedIndex];
console.log(selectedOption.textContent); // "广州"
```

## 4.3 禁用和启用表单元素
```javascript
const input = document.querySelector("#username");
const btn = document.querySelector("#submit");

// 禁用
input.disabled = true;
btn.disabled = true;

// 启用
input.disabled = false;
```

# 五、批量渲染性能优化
## 5.1 频繁操作DOM的性能问题
当需要渲染大量元素时，如果每个元素都单独操作一次DOM（appendChild），会导致**页面重排（reflow）次数过多**，性能严重下降：

```javascript
// ❌ 性能差：每次都操作DOM
const list = document.querySelector("#list");
for (let i = 0; i < 1000; i++) {
    const li = document.createElement("li");
    li.textContent = `第 ${i + 1} 项`;
    list.appendChild(li); // 每次循环都操作一次DOM → 1000次重排！
}
```

## 5.2 解决方案1：DocumentFragment
`DocumentFragment`是一个**虚拟的DOM节点容器**，在它上面操作不会触发页面重排。先把所有元素添加到Fragment中，最后一次性插入页面——**只触发一次重排**。

```javascript
// ✓ 性能好：使用DocumentFragment
const list = document.querySelector("#list");
const fragment = document.createDocumentFragment(); // 虚拟容器

for (let i = 0; i < 1000; i++) {
    const li = document.createElement("li");
    li.textContent = `第 ${i + 1} 项`;
    fragment.appendChild(li); // 操作虚拟容器，不触发重排
}

list.appendChild(fragment); // 一次性插入，只触发一次重排
```

## 5.3 解决方案2：innerHTML（直接生成字符串）
对于非常简单的批量渲染，拼接字符串后一次性设置`innerHTML`是最快的方式：

```javascript
// ✓ 又快又简洁（仅限简单结构、数据可信）
const list = document.querySelector("#list");
let html = "";

for (let i = 0; i < 1000; i++) {
    html += `<li>第 ${i + 1} 项</li>`;
}

list.innerHTML = html; // 只操作一次DOM
```

> **注意**：innerHTML不能安全地插入用户输入的内容（XSS风险），且已存在的事件监听器会丢失。对于复杂结构或需要绑定事件的场景，使用DocumentFragment更安全。

## 5.4 三种方式对比
| 方式 | 性能 | 安全 | 保留事件 | 适用场景 |
|------|------|------|---------|----------|
| 逐个appendChild | 差 | ✓ | ✓ | 几个元素的小批量操作 |
| DocumentFragment | 好 | ✓ | ✓ | 大量元素，需要绑定事件 |
| innerHTML拼接字符串 | 最好 | ⚠️ | ✗ | 大量简单结构，数据可信 |

# 六、获取元素尺寸和位置
## 6.1 getBoundingClientRect()：推荐
返回元素相对于**视口（浏览器可见区域）**的位置和大小：

```javascript
const box = document.querySelector(".box");
const rect = box.getBoundingClientRect();

console.log(rect.top);     // 元素顶部距视口顶部的距离
console.log(rect.left);    // 元素左边距视口左边的距离
console.log(rect.bottom);  // 元素底部距视口顶部的距离
console.log(rect.right);   // 元素右边距视口左边的距离
console.log(rect.width);   // 元素宽度（含border和padding）
console.log(rect.height);  // 元素高度
console.log(rect.x);       // 同left
console.log(rect.y);       // 同top
```

**用途**：判断元素是否在可见区域（懒加载）、获取元素相对于视口的位置（tooltip定位）、拖拽计算等。

## 6.2 offset系列：相对于定位父元素
```javascript
const box = document.querySelector(".box");

// 相对于最近的定位祖先元素（position不为static的祖先）
console.log(box.offsetTop);
console.log(box.offsetLeft);

// 元素自身尺寸（含border）
console.log(box.offsetWidth);
console.log(box.offsetHeight);

// 最近的定位祖先元素
console.log(box.offsetParent);
```

## 6.3 client系列：内容和padding区域
```javascript
const box = document.querySelector(".box");

// 内容+padding（不含border和滚动条）
console.log(box.clientWidth);
console.log(box.clientHeight);

// 边框宽度
console.log(box.clientTop);
console.log(box.clientLeft);
```

## 6.4 scroll系列：滚动相关
```javascript
const box = document.querySelector(".box");

// 内容总高度（包括溢出不可见的部分）
console.log(box.scrollWidth);
console.log(box.scrollHeight);

// 当前滚动位置
console.log(box.scrollTop);
console.log(box.scrollLeft);

// 设置滚动位置
box.scrollTop = 100;
box.scrollTo({ top: 200, behavior: "smooth" }); // 平滑滚动
```

# 七、综合实战：动态列表管理
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>动态列表管理</title>
    <style>
        body { font-family: sans-serif; max-width: 500px; margin: 40px auto; padding: 0 20px; }
        h2 { color: #333; }

        .input-group {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        .input-group input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 14px;
        }
        .input-group button {
            padding: 10px 20px;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        .todo-list {
            list-style: none;
            padding: 0;
        }
        .todo-list li {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px;
            border-bottom: 1px solid #eee;
            transition: all 0.3s;
        }
        .todo-list li:hover {
            background: #f8f8f8;
        }
        .todo-list li.completed span {
            text-decoration: line-through;
            color: #999;
        }
        .todo-list .delete-btn {
            padding: 4px 12px;
            background: #e74c3c;
            color: white;
            border: none;
            border-radius: 3px;
            cursor: pointer;
            font-size: 12px;
        }
        .todo-list .delete-btn:hover {
            background: #c0392b;
        }
        .empty-tip {
            text-align: center;
            color: #999;
            padding: 30px;
        }
        .stats {
            margin-top: 20px;
            font-size: 14px;
            color: #666;
        }
    </style>
</head>
<body>
    <h2>任务列表</h2>

    <div class="input-group">
        <input type="text" id="task-input" placeholder="输入新任务...">
        <button id="add-btn">添加</button>
    </div>

    <ul id="todo-list" class="todo-list">
        <li class="empty-tip">暂无任务，添加一个吧</li>
    </ul>

    <div class="stats">
        共 <span id="total-count">0</span> 个任务，
        已完成 <span id="completed-count">0</span> 个
    </div>

    <script>
        const todoList = document.querySelector("#todo-list");
        const taskInput = document.querySelector("#task-input");
        const addBtn = document.querySelector("#add-btn");
        const totalCount = document.querySelector("#total-count");
        const completedCount = document.querySelector("#completed-count");

        let tasks = []; // 任务数据

        // 更新统计
        function updateStats() {
            totalCount.textContent = tasks.length;
            const completed = tasks.filter(t => t.completed).length;
            completedCount.textContent = completed;
        }

        // 渲染列表
        function render() {
            // 清空列表
            todoList.innerHTML = "";

            if (tasks.length === 0) {
                todoList.innerHTML = '<li class="empty-tip">暂无任务，添加一个吧</li>';
                updateStats();
                return;
            }

            // 使用DocumentFragment批量渲染
            const fragment = document.createDocumentFragment();

            tasks.forEach((task, index) => {
                const li = document.createElement("li");
                if (task.completed) li.classList.add("completed");

                li.innerHTML = `
                    <span style="cursor:pointer" data-index="${index}">${task.text}</span>
                    <button class="delete-btn" data-index="${index}">删除</button>
                `;

                // 点击文字切换完成状态（事件委托将在下篇统一学习，这里使用直接绑定）
                const span = li.querySelector("span");
                span.addEventListener("click", () => {
                    tasks[index].completed = !tasks[index].completed;
                    render();
                });

                // 点击删除按钮
                const delBtn = li.querySelector(".delete-btn");
                delBtn.addEventListener("click", () => {
                    tasks.splice(index, 1);
                    render();
                });

                fragment.appendChild(li);
            });

            todoList.appendChild(fragment);
            updateStats();
        }

        // 添加任务
        function addTask() {
            const text = taskInput.value.trim();
            if (!text) {
                alert("请输入任务内容");
                return;
            }
            tasks.push({ text, completed: false });
            taskInput.value = "";
            taskInput.focus();
            render();
        }

        // 绑定事件
        addBtn.addEventListener("click", addTask);
        taskInput.addEventListener("keydown", (e) => {
            if (e.key === "Enter") addTask();
        });

        // 初始渲染
        render();
    </script>
</body>
</html>
```

# 八、常见误区与避坑指南
1.  **创建了元素但忘记插入DOM**：`createElement`只是创建了一个内存中的DOM节点，必须通过`appendChild`、`insertBefore`等方法插入到页面上才能看到。

2.  **appendChild移动已有节点而非复制**：如果你用`appendChild`插入一个已经存在于页面上的元素，它会被**移动**（从原来位置移到新位置），而不是复制。如果要保留原位置的元素，需要先用`cloneNode`克隆。

3.  **循环中逐个操作DOM导致性能问题**：渲染10个元素逐条appendChild没问题，但渲染1000个时性能会明显下降。大量数据渲染使用`DocumentFragment`或拼接`innerHTML`一次性插入。

4.  **innerHTML会清除已有的事件监听器**：`element.innerHTML = newHTML`会销毁旧元素上的所有事件处理器。如果你需要在已有的元素上保留事件，不要用innerHTML替换整个内容。

5.  **getBoundingClientRect的坐标是相对视口的**：滚动页面后，同一个元素的`getBoundingClientRect().top`会变化。如果需要相对文档的绝对位置，需要加上`window.scrollY`。

6.  **忘记form元素用value获取值**：input/select/textarea的值通过`.value`获取，不是通过`.textContent`。

# 九、核心总结：DOM进阶速查表
## 创建与操作
| 操作 | 方法 |
|------|------|
| 创建元素 | `document.createElement("div")` |
| 插入末尾 | `parent.appendChild(child)` |
| 插入指定位置之前 | `parent.insertBefore(new, ref)` |
| 插入HTML | `element.insertAdjacentHTML("beforeend", html)` |
| 删除自身 | `element.remove()` |
| 通过父删除 | `parent.removeChild(child)` |
| 克隆 | `element.cloneNode(true)` |

## 表单操作
| 元素类型 | 获取值 | 设置值 |
|----------|--------|--------|
| input[text] | `input.value` | `input.value = "xx"` |
| textarea | `textarea.value` | 同上 |
| checkbox | `checkbox.checked` | `checkbox.checked = true` |
| select | `select.value` | `select.value = "xx"` |
| 禁用 | `el.disabled = true` | |

## 尺寸与位置
| 需求 | 方法 |
|------|------|
| 相对视口位置 | `element.getBoundingClientRect()` |
| 自身尺寸（含border） | `element.offsetWidth` / `offsetHeight` |
| 内容+padding | `element.clientWidth` / `clientHeight` |
| 滚动位置 | `element.scrollTop` / `scrollLeft` |

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维、前端开发、JavaScript全栈等核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】
除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：待发布
