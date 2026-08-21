

# Python全栈入门到实战【基础篇 20】文件操作核心：读取、写入与管理

在前序内容中，我们掌握的变量、列表、字典等数据都存储在内存中，程序关闭后数据会完全丢失。而**文件操作**是实现数据持久化的核心手段，通过将数据写入磁盘文件，可实现数据的长期保存与跨程序共享。文件操作是Python开发的必备技能，广泛应用于日志记录、数据存储、配置文件读写等场景。

本节将系统讲解Python文件操作的全流程，从文件的打开关闭到读写管理，结合异常处理实现健壮的文件操作逻辑：
- 文件的基础操作：`open()`函数、文件打开模式、`close()`方法与`with`上下文管理器；
- 文件的读取操作：`read()`/`readline()`/`readlines()`的用法与适用场景；
- 文件的写入操作：`write()`/`writelines()`的使用与模式选择；
- 文件与目录管理：`os`/`os.path`模块的核心方法；
- 实战案例：实现一个文本文件内容筛选与备份工具。

@[TOC](文章目录)

# 一、文件操作基础：打开与关闭
文件操作的核心流程为**打开文件→操作文件→关闭文件**，其中“打开”与“关闭”是保障文件资源安全的关键步骤。

## 1. 核心函数：`open()` 打开文件
`open()`函数用于打开一个文件，并返回一个文件对象（也称文件句柄），后续的读写操作均基于该对象。
### 基础语法
```python
file_object = open(file_path, mode='r', encoding=None)
```
### 参数说明
| 参数        | 作用                                | 示例                                  |
| ----------- | ----------------------------------- | ------------------------------------- |
| `file_path` | 文件路径（绝对路径/相对路径）       | `'test.txt'`、`'/home/user/test.txt'` |
| `mode`      | 文件打开模式，默认`'r'`（只读）     | 详见下表                              |
| `encoding`  | 文件编码格式，文本文件常用`'utf-8'` | `encoding='utf-8'`                    |

### 常用文件打开模式
| 模式          | 功能说明         | 注意事项                                       |
| ------------- | ---------------- | ---------------------------------------------- |
| `'r'`         | 只读模式（默认） | 文件不存在则抛出`FileNotFoundError`            |
| `'w'`         | 写入模式         | 文件不存在则创建，存在则覆盖原有内容           |
| `'a'`         | 追加模式         | 文件不存在则创建，写入内容追加到文件末尾       |
| `'r+'`        | 读写模式         | 可读可写，文件不存在则报错                     |
| `'w+'`        | 读写模式         | 可读可写，文件不存在则创建，存在则覆盖         |
| `'a+'`        | 读写模式         | 可读可写，写入内容追加到末尾                   |
| `'rb'`/`'wb'` | 二进制模式读写   | 用于图片、视频等二进制文件，无需指定`encoding` |

## 2. 核心方法：`close()` 关闭文件
文件操作完成后，必须调用`close()`方法关闭文件，释放系统资源（如文件句柄、内存缓冲区）。
### 基础示例
```python
# 打开文件
f = open('test.txt', 'r', encoding='utf-8')
# 读取文件内容
content = f.read()
print(content)
# 关闭文件
f.close()
```
### 潜在风险
若文件操作过程中触发异常，`close()`方法可能无法执行，导致资源泄露。解决方案是结合**异常处理**或**上下文管理器**。

## 3. 最佳实践：`with` 上下文管理器
`with`语句是Python提供的语法糖，可自动管理文件资源——进入`with`块时打开文件，退出`with`块时自动关闭文件，无需手动调用`close()`，且能保证异常情况下文件正常关闭。
### 基础语法
```python
with open(file_path, mode, encoding) as file_object:
    # 文件操作代码
    pass
```
### 核心优势
- 自动关闭文件，避免资源泄露；
- 代码更简洁，无需手动写`try-finally`；
- 异常处理更优雅，可结合`try-except`捕获文件操作异常。

### 示例：使用`with`读取文件
```python
try:
    with open('test.txt', 'r', encoding='utf-8') as f:
        content = f.read()
        print(content)
except FileNotFoundError:
    print("错误：文件不存在")
except UnicodeDecodeError:
    print("错误：文件编码格式不匹配")
```

# 二、文件读取操作：获取文件内容
文件读取是最常用的操作之一，Python提供了三种核心读取方法，适用于不同场景。

## 1. `read(size)`：读取指定字节数的内容
- `size`为可选参数，默认读取文件全部内容；
- 适合读取小文件，大文件一次性读取会占用大量内存。
### 示例
```python
# 读取全部内容
with open('test.txt', 'r', encoding='utf-8') as f:
    content = f.read()
    print(content)

# 读取前10个字符
with open('test.txt', 'r', encoding='utf-8') as f:
    content = f.read(10)
    print(content)
```

## 2. `readline()`：逐行读取文件内容
- 每次调用读取文件的一行内容，返回字符串；
- 读取到文件末尾时返回空字符串；
- 适合读取大文件，逐行读取可降低内存占用。
### 示例
```python
# 逐行读取文件
with open('test.txt', 'r', encoding='utf-8') as f:
    while True:
        line = f.readline()
        if not line:  # 读取到末尾，退出循环
            break
        print(line.strip())  # strip()去除换行符和空格
```

## 3. `readlines()`：读取所有行到列表
- 一次性读取文件所有行，返回字符串列表，每行内容为列表的一个元素；
- 适合读取结构清晰的小文件（如配置文件、日志文件）。
### 示例
```python
with open('test.txt', 'r', encoding='utf-8') as f:
    lines = f.readlines()
    # 遍历所有行
    for line in lines:
        print(line.strip())
```

## 4. 三种读取方法对比
| 方法          | 优点                         | 缺点             | 适用场景       |
| ------------- | ---------------------------- | ---------------- | -------------- |
| `read()`      | 代码简洁，一次性获取全部内容 | 大文件占用内存高 | 小文件读取     |
| `readline()`  | 内存占用低                   | 代码稍繁琐       | 大文件逐行处理 |
| `readlines()` | 便于遍历和切片操作           | 大文件占用内存高 | 小文件按行处理 |

# 三、文件写入操作：保存数据到文件
文件写入操作用于将内存中的数据持久化到磁盘，核心方法为`write()`和`writelines()`。

## 1. `write(content)`：写入字符串内容
- `content`必须是字符串类型，非字符串需先转换；
- 写入模式需选择`'w'`（覆盖）或`'a'`（追加）。
### 示例1：覆盖写入（`'w'`模式）
```python
# 覆盖写入：文件不存在则创建，存在则覆盖
with open('output.txt', 'w', encoding='utf-8') as f:
    f.write("Python文件操作\n")
    f.write("这是第二行内容\n")
    # 非字符串需转换
    num = 100
    f.write(f"数字：{num}\n")
```

### 示例2：追加写入（`'a'`模式）
```python
# 追加写入：内容添加到文件末尾
with open('output.txt', 'a', encoding='utf-8') as f:
    f.write("这是追加的内容\n")
```

## 2. `writelines(iterable)`：写入字符串序列
- `iterable`为字符串序列（列表、元组、生成器等）；
- 不会自动添加换行符，需手动在每个字符串末尾添加`'\n'`。
### 示例
```python
lines = ["第一行内容", "第二行内容", "第三行内容"]
# 手动添加换行符
lines_with_newline = [line + '\n' for line in lines]

with open('output.txt', 'w', encoding='utf-8') as f:
    f.writelines(lines_with_newline)
```

## 3. 二进制文件的读写
对于图片、视频、音频等二进制文件，需使用`'rb'`/`'wb'`模式，无需指定`encoding`参数。
### 示例：复制图片文件
```python
# 读取二进制文件
with open('source.jpg', 'rb') as f_src:
    data = f_src.read()

# 写入二进制文件
with open('target.jpg', 'wb') as f_target:
    f_target.write(data)

print("图片复制完成")
```

# 四、文件与目录管理：`os` 与 `os.path` 模块
Python的`os`模块提供了与操作系统交互的功能，可实现目录创建、删除、文件重命名等操作；`os.path`模块则用于处理文件路径相关的问题。

## 1. 核心模块：`os` 常用方法
| 方法                            | 功能说明                         | 示例                              |
| ------------------------------- | -------------------------------- | --------------------------------- |
| `os.getcwd()`                   | 获取当前工作目录                 | `print(os.getcwd())`              |
| `os.chdir(path)`                | 切换工作目录                     | `os.chdir('/home/user')`          |
| `os.listdir(path)`              | 列出指定目录下的所有文件和子目录 | `os.listdir('.')`                 |
| `os.mkdir(path)`                | 创建单级目录                     | `os.mkdir('new_dir')`             |
| `os.makedirs(path)`             | 创建多级目录                     | `os.makedirs('dir1/dir2/dir3')`   |
| `os.remove(path)`               | 删除文件                         | `os.remove('test.txt')`           |
| `os.rmdir(path)`                | 删除空目录                       | `os.rmdir('new_dir')`             |
| `os.rename(old_name, new_name)` | 重命名文件/目录                  | `os.rename('old.txt', 'new.txt')` |

## 2. 核心模块：`os.path` 常用方法
| 方法                         | 功能说明               | 示例                                              |
| ---------------------------- | ---------------------- | ------------------------------------------------- |
| `os.path.abspath(path)`      | 获取文件的绝对路径     | `os.path.abspath('test.txt')`                     |
| `os.path.exists(path)`       | 判断文件/目录是否存在  | `os.path.exists('test.txt')`                      |
| `os.path.isfile(path)`       | 判断是否为文件         | `os.path.isfile('test.txt')`                      |
| `os.path.isdir(path)`        | 判断是否为目录         | `os.path.isdir('new_dir')`                        |
| `os.path.join(path1, path2)` | 拼接路径（跨平台兼容） | `os.path.join('dir1', 'dir2', 'test.txt')`        |
| `os.path.splitext(path)`     | 分离文件名与扩展名     | `os.path.splitext('test.txt') → ('test', '.txt')` |

## 3. 示例：目录与文件操作综合
```python
import os

# 1. 获取当前工作目录
current_dir = os.getcwd()
print(f"当前工作目录：{current_dir}")

# 2. 创建多级目录
target_dir = os.path.join(current_dir, "data", "logs")
if not os.path.exists(target_dir):
    os.makedirs(target_dir)
    print(f"目录创建成功：{target_dir}")

# 3. 在目录中创建文件
file_path = os.path.join(target_dir, "app.log")
with open(file_path, 'w', encoding='utf-8') as f:
    f.write("应用启动日志\n")

# 4. 判断是否为文件
if os.path.isfile(file_path):
    print(f"文件存在：{file_path}")

# 5. 列出目录下的文件
print(f"目录下的文件：{os.listdir(target_dir)}")
```

# 五、实战案例：文本文件内容筛选与备份工具
需求：实现一个工具，读取指定文本文件，筛选出包含指定关键词的行，将结果保存到新文件，并备份原文件。
## 功能需求
1. 支持指定源文件路径、关键词、目标文件路径；
2. 筛选包含关键词的行，忽略大小写；
3. 备份原文件（在原文件名后添加`.bak`后缀）；
4. 结合异常处理，确保程序健壮性。

## 完整代码
```python
import os
import shutil

def filter_file_by_keyword(source_path, target_path, keyword):
    """
    筛选文本文件中包含指定关键词的行，并保存到目标文件
    :param source_path: 源文件路径
    :param target_path: 目标文件路径
    :param keyword: 筛选关键词
    :return: 成功返回True，失败返回False
    """
    try:
        # 1. 备份原文件
        backup_path = source_path + ".bak"
        shutil.copy2(source_path, backup_path)
        print(f"原文件已备份：{backup_path}")

        # 2. 读取源文件，筛选内容
        filtered_lines = []
        with open(source_path, 'r', encoding='utf-8') as f_src:
            for line in f_src:
                # 忽略大小写，判断是否包含关键词
                if keyword.lower() in line.lower():
                    filtered_lines.append(line)

        # 3. 写入目标文件
        with open(target_path, 'w', encoding='utf-8') as f_target:
            f_target.writelines(filtered_lines)

        print(f"筛选完成，共找到{len(filtered_lines)}行包含关键词'{keyword}'的内容")
        print(f"结果已保存到：{target_path}")
        return True

    except FileNotFoundError:
        print(f"错误：源文件不存在 → {source_path}")
        return False
    except PermissionError:
        print(f"错误：无权限访问文件 → {source_path}")
    except Exception as e:
        print(f"错误：{e}")
        return False

# 程序入口
if __name__ == "__main__":
    # 配置参数
    source_file = "data.txt"
    target_file = "filtered_data.txt"
    keyword = "python"

    # 执行筛选
    filter_file_by_keyword(source_file, target_file, keyword)
```

## 代码解析
1. **备份功能**：使用`shutil.copy2()`复制原文件，实现备份；
2. **筛选逻辑**：逐行读取源文件，忽略大小写判断是否包含关键词；
3. **异常处理**：捕获`FileNotFoundError`（文件不存在）、`PermissionError`（权限不足）等常见异常；
4. **跨平台兼容**：使用`os.path`模块处理路径，确保在Windows/Linux/MacOS上均可运行。

## 运行示例
假设`data.txt`内容如下：
```
Python文件操作基础
Java面向对象编程
Python函数进阶
C语言指针详解
Python异常处理
```
运行程序后，`filtered_data.txt`内容为：
```
Python文件操作基础
Python函数进阶
Python异常处理
```
同时生成`data.txt.bak`备份文件。

# 六、总结
本节系统讲解了Python文件操作的核心知识点，实现了数据从内存到磁盘的持久化存储：
1. **文件基础操作**：
   - 使用`open()`函数打开文件，注意选择正确的打开模式；
   - 优先使用`with`上下文管理器，自动管理文件资源，避免泄露；
   - 结合异常处理捕获文件操作中的常见错误。
2. **文件读写操作**：
   - 读取方法：`read()`（小文件）、`readline()`（大文件）、`readlines()`（按行处理）；
   - 写入方法：`write()`（写入字符串）、`writelines()`（写入字符串序列）；
   - 二进制文件读写需使用`'rb'`/`'wb'`模式。
3. **文件与目录管理**：
   - `os`模块实现目录创建、删除、重命名等操作；
   - `os.path`模块处理路径拼接、判断文件/目录是否存在等问题；
   - `shutil`模块提供高级文件操作（如复制、移动）。
4. **实战原则**：
   - 操作文件时必须添加异常处理；
   - 路径处理优先使用`os.path.join()`，保证跨平台兼容；
   - 重要文件操作前先备份，防止数据丢失。

文件操作是Python全栈开发的基础技能，后续的数据库操作、网络爬虫、Web开发等场景均会用到本节知识。下一节，我们将讲解**Python 高级特性：切片、迭代**

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后，专栏内所有文章可永久免费查看，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/158351897>
