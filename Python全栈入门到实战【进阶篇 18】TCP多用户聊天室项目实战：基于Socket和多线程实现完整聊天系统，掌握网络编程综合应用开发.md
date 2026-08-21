# Python全栈入门到实战【进阶篇 18】TCP多用户聊天室项目实战：基于Socket和多线程实现完整聊天系统，掌握网络编程综合应用开发

上一篇我们吃透了**TCP网络编程核心实战**，掌握了面向连接的可靠通信逻辑，能独立开发TCP客户端与服务端程序，解决了粘包等核心坑点。这一篇我们将基于TCP协议，结合面向对象设计，从零到一构建一个**多用户聊天室**项目——这是网络编程最经典的综合实战案例，涵盖TCP通信、多线程并发、线程安全、协议设计、面向对象封装等核心知识，能帮你将前两篇的知识点融会贯通，真正掌握网络编程在真实项目中的应用。

本节课作为网络编程的综合实战篇，专门讲透如何将TCP基础知识落地为完整项目，**从需求分析、协议设计到完整代码实现**，每一步都附详细讲解和逐行注释代码，帮你彻底吃透多用户聊天室的完整开发流程，为后续开发更复杂的网络应用（如即时通讯、游戏服务器、物联网平台）打下坚实基础。

![](https://liaoxuefeng.com/books/python/network/tcp-ip/net.png)

本节核心学习内容：
- 项目概述：多用户聊天室的功能需求和整体架构
- 通信协议设计：自定义文本行协议，明确客户端与服务器的消息格式
- 系统设计：面向对象的类设计（User、Server、ClientHandler、ChatClient）
- 多线程与线程安全：如何用锁保护共享数据，避免竞态条件
- 详细开发步骤：从单客户端通信到完整功能的增量开发全过程
- 完整代码展示：带超详细注释的服务器端和客户端代码
- 常见问题与解决方案：端口占用、粘包、线程泄漏等坑点避坑指南
- 项目扩展建议：图形界面、文件传输、加密通信等进阶方向
- 核心总结：TCP聊天室项目的知识要点与开发心得

# 一、项目概述

## 1.1 项目背景

网络编程是Python全栈开发的核心技能之一，而多用户聊天室是练习网络编程最经典的项目。通过实现一个聊天室，你将深入理解TCP协议的实际应用、多线程并发处理、线程安全以及如何将面向对象设计运用到网络程序中。本项目将实现一个控制台下的多用户聊天室，支持多个用户同时在线、设置昵称、发送公共消息、私聊、查看在线用户等功能。

## 1.2 项目目标

- 实现一个基于TCP协议的多用户聊天室服务器，能够同时处理多个客户端连接。
- 实现一个基于TCP协议的客户端，能够连接服务器并进行聊天。
- 客户端需支持昵称注册、公共聊天、私聊、查看在线用户、退出等功能。
- 服务器需保证多线程环境下共享数据的安全，正确处理客户端异常断开。
- 代码结构清晰，符合面向对象设计原则，便于扩展和维护。

## 1.3 功能需求

**服务器端**
- 监听指定IP和端口（如`127.0.0.1:8888`），接受客户端连接。
- 为每个连接创建一个独立线程，负责与该客户端的所有通信。
- 维护在线用户列表，包含每个用户的昵称及其对应的socket连接。
- 接收客户端消息，解析命令并执行相应操作：
  - `NICK 昵称`：设置昵称，检查唯一性；成功则加入在线列表，回复`NICK_OK`；失败则回复`NICK_ERR`。
  - `MSG 消息`：广播公共消息给所有在线用户，格式为`MSG 发送者 消息`。
  - `PM 目标昵称 消息`：发送私聊消息给指定用户，若目标存在则发送`PM 发送者 消息`，并给发送者回显`PM 对 目标: 消息`；若目标不在线则回复错误信息。
  - `LIST`：返回当前在线用户列表，格式为`LIST 用户1 用户2 ...`。
  - `QUIT`：用户主动退出，清理资源并广播`INFO 用户 离开了聊天室`。
- 处理客户端异常断开（如强制关闭窗口），及时从在线列表中移除该用户，并广播离开消息。
- 所有消息以换行符`\n`结尾，确保接收方能正确分割。

**客户端**
- 连接服务器后，首先提示用户输入昵称，发送`NICK 昵称`给服务器，并根据服务器响应决定是否继续。
- 启动两个线程：
  - **接收线程**：持续接收服务器发来的消息，并根据消息类型（`MSG`、`PM`、`INFO`、`LIST`、`ERROR`等）进行格式化显示。
  - **发送线程（主线程）**：循环读取用户输入，根据输入内容发送相应命令或消息。
- 支持命令：
  - `/list`：查看当前在线用户。
  - `/quit`：退出聊天室。
  - `/msg 昵称 消息`：向指定用户发送私聊消息。
  - 直接输入其他内容则作为公共消息发送。
- 当连接断开时（服务器关闭或网络异常），客户端应能感知并退出。

# 二、通信协议设计

TCP是流式协议，没有消息边界，因此我们需要自定义消息边界。本项目采用最简单的**文本行协议**：每条消息以换行符`\n`结尾。发送时在字符串末尾加上`\n`，接收时逐字节读取直到遇到`\n`。

消息的格式为：`命令 参数1 参数2 ...`，命令与参数之间用空格分隔。以下是完整的命令列表：

| 方向          | 命令                   | 示例                      | 说明                 |
| ------------- | ---------------------- | ------------------------- | -------------------- |
| 客户端→服务器 | `NICK 昵称`            | `NICK Alice`              | 设置昵称             |
| 客户端→服务器 | `MSG 消息`             | `MSG 大家好`              | 发送公共消息         |
| 客户端→服务器 | `PM 目标昵称 消息`     | `PM Bob 你好`             | 发送私聊             |
| 客户端→服务器 | `LIST`                 | `LIST`                    | 请求在线用户         |
| 客户端→服务器 | `QUIT`                 | `QUIT`                    | 退出聊天室           |
| 服务器→客户端 | `NICK_OK`              | `NICK_OK`                 | 昵称设置成功         |
| 服务器→客户端 | `NICK_ERR 提示`        | `NICK_ERR 昵称已被占用`   | 昵称设置失败         |
| 服务器→客户端 | `MSG 发送者 消息`      | `MSG Alice 大家好`        | 广播公共消息         |
| 服务器→客户端 | `PM 发送者 消息`       | `PM Alice 你好`           | 私聊消息             |
| 服务器→客户端 | `LIST 用户1 用户2 ...` | `LIST Alice Bob`          | 在线用户列表         |
| 服务器→客户端 | `INFO 提示`            | `INFO Alice 加入了聊天室` | 系统通知             |
| 服务器→客户端 | `ERROR 错误`           | `ERROR 用户 Bob 不在线`   | 错误提示             |
| 服务器→客户端 | `BYE`                  | `BYE`                     | 服务器主动断开前通知 |

这样设计的好处是简单直观，易于调试，也方便后续扩展。

# 三、系统设计

在动手编码之前，我们先进行系统设计，明确类结构和线程模型，这样后续开发会更有条理。

## 3.1 类设计

为了使代码清晰，我们将使用面向对象设计。主要包含以下类：

- **`User`**：代表一个连接的用户，包含socket、地址、昵称。
- **`Server`**：服务器核心，负责启动监听、管理在线用户列表、提供广播和私聊等方法。
- **`ClientHandler`**（继承 `threading.Thread`）：每个客户端对应的线程，处理该客户端的消息收发和命令解析。
- **`ChatClient`**：客户端主类，负责连接服务器、发送/接收消息、处理用户输入。

它们的关系如下：
- `Server` 拥有一个 `users` 字典（键为socket，值为 `User` 对象）和一个线程锁。
- 当 `Server.accept()` 接受一个新连接时，会创建一个 `User` 对象和一个 `ClientHandler` 线程，并将 `User` 对象和 `Server` 实例传给该线程。
- `ClientHandler` 在运行时，通过持有的 `Server` 实例调用广播、私聊等方法，并利用锁保护共享数据。

## 3.2 多线程与线程安全

- **主线程**：运行 `Server.start()`，不断 `accept()` 新连接，为每个连接创建 `ClientHandler` 线程。
- **工作线程**：每个 `ClientHandler` 负责一个客户端，包括接收消息、解析命令、调用服务器方法。
- **共享数据**：`users` 字典会被所有工作线程并发访问（添加、删除、查找、遍历）。为了保证数据一致性，我们使用 `threading.Lock` 保护对 `users` 的每次读写操作。
- **广播优化**：在广播消息时，我们采用“先加锁复制用户列表，释放锁后再逐个发送”的策略，避免长时间持有锁导致其他线程阻塞。即使发送过程中某个socket已失效，我们捕获异常即可，由对应的线程负责清理。

# 四、详细开发步骤

我们将采用**增量开发**方式，每一步都构建在前一步的基础上，确保每个阶段都有可运行的版本。每一步都包含：**目标**、**具体任务**、**代码实现**、**测试方法**和**小结**。

## 第1步：实现单客户端通信（基础）

### 目标
搭建最基础的TCP通信框架，实现服务器与单个客户端的“一问一答”。通过这一步，你将掌握socket的基本API和文本行协议。

### 具体任务
1. 编写服务器程序：创建TCP socket，绑定地址和端口，监听并接受一个客户端连接；进入循环，每次从客户端读取一行数据（以`\n`结尾），打印并回复一条确认消息。
2. 编写客户端程序：连接服务器，循环读取用户输入，发送给服务器，并接收回复打印；支持输入`quit`退出。

### 代码实现

**server_step1.py**
```python
import socket

def main():
    host = '127.0.0.1'
    port = 8888

    # 1. 创建 TCP socket
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 设置地址复用，避免重启时 "Address already in use"
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 2. 绑定地址和端口
    server_socket.bind((host, port))
    # 3. 开始监听，最大等待连接数为1
    server_socket.listen(1)
    print(f'✅ 服务器启动，监听 {host}:{port}')

    # 4. 接受客户端连接（阻塞）
    client_socket, addr = server_socket.accept()
    print(f'📞 客户端 {addr} 已连接')

    try:
        while True:
            # 5. 逐字节读取，直到遇到换行符，构建一行数据
            data = b''
            while True:
                chunk = client_socket.recv(1)   # 一次读1字节
                if not chunk:                    # 连接关闭
                    break
                if chunk == b'\n':                # 行结束
                    break
                data += chunk
            if not data:                           # 无数据说明客户端关闭连接
                break
            message = data.decode('utf-8')         # 解码为字符串
            print(f'📩 收到消息: {message}')

            # 6. 发送回复，记得加换行符
            reply = '服务器已收到你的消息\n'
            client_socket.sendall(reply.encode('utf-8'))
    except Exception as e:
        print(f'⚠️ 异常: {e}')
    finally:
        # 7. 清理
        client_socket.close()
        server_socket.close()
        print('✅ 服务器关闭')

if __name__ == '__main__':
    main()
```

**client_step1.py**
```python
import socket

def main():
    host = '127.0.0.1'
    port = 8888

    # 1. 创建 TCP socket 并连接服务器
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect((host, port))
    print(f'✅ 已连接到服务器 {host}:{port}')

    try:
        while True:
            msg = input('请输入消息（输入quit退出）: ')
            if msg.lower() == 'quit':
                break

            # 2. 发送消息，加上换行符
            client_socket.sendall((msg + '\n').encode('utf-8'))

            # 3. 接收服务器回复
            data = b''
            while True:
                chunk = client_socket.recv(1)
                if not chunk:
                    break
                if chunk == b'\n':
                    break
                data += chunk
            if data:
                print(f'📩 服务器回复: {data.decode("utf-8")}')
            else:
                print('❌ 服务器已关闭连接')
                break
    except Exception as e:
        print(f'⚠️ 异常: {e}')
    finally:
        client_socket.close()

if __name__ == '__main__':
    main()
```

### 测试方法
1. 打开终端，运行 `python server_step1.py`，等待连接。
2. 打开另一个终端，运行 `python client_step1.py`。
3. 在客户端输入任意消息（例如 `hello`），服务器应打印消息并回复，客户端收到回复。
4. 输入 `quit` 退出客户端，服务器也会退出（因为只接受一个客户端）。

### 小结
- 我们实现了最基本的TCP通信，理解了一次 `recv(1)` 的用法来构建行协议。
- 注意：`recv()` 可能返回空字节表示对端关闭连接，需要正确处理。
- 目前服务器只能处理一个客户端，这是下一步要解决的问题。

## 第2步：支持多客户端（引入多线程）

### 目标
让服务器能同时处理多个客户端，并实现消息广播：一个客户端发送的消息，其他所有客户端都能收到。

### 具体任务
1. 修改服务器主循环，每次 `accept()` 后创建一个新线程（`threading.Thread`）来处理该客户端的通信。
2. 将每个客户端的socket存储在一个全局列表 `clients` 中，以便广播时遍历。
3. 广播逻辑：当一个客户端发送消息时，服务器将该消息发送给列表中除自己以外的所有客户端。
4. 引入线程锁 `threading.Lock` 保护对 `clients` 列表的并发访问（添加、删除、遍历）。

### 代码实现

**server_step2.py**
```python
import socket
import threading

# 全局列表，存储所有客户端 socket
clients = []
clients_lock = threading.Lock()

def handle_client(client_socket, addr):
    """处理单个客户端的线程函数"""
    print(f'📞 新线程处理客户端 {addr}')
    with clients_lock:
        clients.append(client_socket)

    try:
        while True:
            # 逐字节读取一行
            data = b''
            while True:
                chunk = client_socket.recv(1)
                if not chunk:
                    return
                if chunk == b'\n':
                    break
                data += chunk
            if not data:
                break
            message = data.decode('utf-8')
            print(f'📩 [{addr}] {message}')

            # 广播给其他客户端
            with clients_lock:
                # 遍历前复制列表，防止遍历过程中被修改
                for other in list(clients):
                    if other is client_socket:
                        continue
                    try:
                        other.sendall(f'其他用户说: {message}\n'.encode('utf-8'))
                    except:
                        # 发送失败，可能该客户端已断开，交给其线程清理
                        pass
    except Exception as e:
        print(f'⚠️ 处理客户端 {addr} 异常: {e}')
    finally:
        with clients_lock:
            clients.remove(client_socket)
        client_socket.close()
        print(f'❌ 客户端 {addr} 断开连接')

def main():
    host = '127.0.0.1'
    port = 8888

    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((host, port))
    server_socket.listen(5)   # 最大等待连接数设为5
    print(f'✅ 服务器启动，监听 {host}:{port}')

    try:
        while True:
            client_socket, addr = server_socket.accept()
            print(f'📞 接受新连接 {addr}')
            # 创建线程，传入处理函数和参数
            thread = threading.Thread(target=handle_client, args=(client_socket, addr))
            thread.daemon = True   # 守护线程，主线程退出时自动结束
            thread.start()
    except KeyboardInterrupt:
        print('🛑 服务器关闭')
    finally:
        server_socket.close()

if __name__ == '__main__':
    main()
```

客户端代码可以沿用第1步的版本（不需要修改，因为它已经能处理单条消息的收发）。为了测试，我们需要运行多个客户端实例。

### 测试方法
1. 启动服务器：`python server_step2.py`
2. 打开两个或更多终端，分别运行客户端：`python client_step1.py`
3. 在一个客户端输入消息，其他客户端应收到广播消息（发送者自己不会收到）。
4. 关闭一个客户端（直接按Ctrl+C），服务器应打印断开信息，并且其他客户端不受影响。

### 小结
- 我们引入了多线程，使服务器能并发处理多个客户端。
- 使用 `threading.Lock` 保护共享列表，避免了多线程修改时的数据竞争。
- 广播时复制列表，减少锁的持有时间，提高并发性能。
- 但消息格式还很原始（“其他用户说：”），缺少发送者标识，这是下一步要改进的。

## 第3步：添加昵称和命令系统

### 目标
为聊天室引入用户昵称，支持查看在线用户和退出命令。广播消息时带上发送者昵称，让消息更友好。

### 具体任务
1. 将全局列表 `clients` 改为字典 `clients = {socket: nickname}`，方便通过socket查找昵称。
2. 修改客户端连接后的处理逻辑：必须先发送 `NICK 昵称` 命令，服务器检查昵称唯一性，成功后加入字典并回复 `NICK_OK`，否则回复错误并要求重试。
3. 修改广播函数，使其发送格式化的消息，如 `MSG Alice 大家好`。
4. 实现命令解析：当收到以 `/` 开头的行时，识别命令并执行相应操作：
   - `/LIST`：返回当前在线用户列表。
   - `/QUIT`：用户主动退出。
5. 用户退出时，从字典中移除，并广播离开消息。
6. 保证所有对字典的访问都受锁保护。

### 代码实现

**server_step3.py**
```python
import socket
import threading

clients = {}          # {socket: nickname}
clients_lock = threading.Lock()

def broadcast(message, exclude=None):
    """广播消息给所有客户端，exclude 是要排除的 socket"""
    with clients_lock:
        items = list(clients.items())
    for sock, nick in items:
        if sock is exclude:
            continue
        try:
            sock.sendall((message + '\n').encode('utf-8'))
        except:
            pass

def handle_client(client_socket, addr):
    print(f'📞 新线程处理客户端 {addr}')

    # ---- 昵称设置阶段 ----
    nickname = None
    while True:
        try:
            data = b''
            while True:
                chunk = client_socket.recv(1)
                if not chunk:
                    return
                if chunk == b'\n':
                    break
                data += chunk
            line = data.decode('utf-8').strip()
            if not line:
                continue

            if line.startswith('NICK '):
                nickname = line[5:].strip()
                with clients_lock:
                    if nickname in clients.values():
                        client_socket.sendall(b'NICK_ERR 昵称已被占用\n')
                    else:
                        clients[client_socket] = nickname
                        client_socket.sendall(b'NICK_OK\n')
                        break
            else:
                client_socket.sendall(b'ERROR 请先发送 NICK 命令设置昵称\n')
        except:
            return

    # 广播新用户加入
    broadcast(f'INFO {nickname} 加入了聊天室', exclude=client_socket)

    # ---- 主消息循环 ----
    try:
        while True:
            data = b''
            while True:
                chunk = client_socket.recv(1)
                if not chunk:
                    return
                if chunk == b'\n':
                    break
                data += chunk
            if not data:
                break
            line = data.decode('utf-8').strip()
            if not line:
                continue

            # 解析命令
            if line.startswith('/'):
                parts = line.split(' ', 1)
                cmd = parts[0][1:].upper()
                if cmd == 'LIST':
                    with clients_lock:
                        nick_list = ' '.join(clients.values())
                    client_socket.sendall(f'LIST {nick_list}\n'.encode('utf-8'))
                elif cmd == 'QUIT':
                    client_socket.sendall(b'BYE\n')
                    break
                else:
                    client_socket.sendall(b'ERROR 未知命令\n')
            else:
                # 公共消息
                broadcast(f'MSG {nickname} {line}', exclude=None)
    except Exception as e:
        print(f'⚠️ 处理 {nickname} 异常: {e}')
    finally:
        with clients_lock:
            if client_socket in clients:
                del clients[client_socket]
        broadcast(f'INFO {nickname} 离开了聊天室', exclude=None)
        client_socket.close()
        print(f'❌ 客户端 {nickname} 断开连接')

def main():
    host = '127.0.0.1'
    port = 8888

    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((host, port))
    server_socket.listen(5)
    print(f'✅ 服务器启动，监听 {host}:{port}')

    try:
        while True:
            client_socket, addr = server_socket.accept()
            thread = threading.Thread(target=handle_client, args=(client_socket, addr))
            thread.daemon = True
            thread.start()
    except KeyboardInterrupt:
        print('🛑 服务器关闭')
    finally:
        server_socket.close()

if __name__ == '__main__':
    main()
```

客户端需要修改以支持昵称设置和命令输入，这里给出一个初步版本：

**client_step3.py**
```python
import socket
import threading

def main():
    host = '127.0.0.1'
    port = 8888

    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((host, port))
    print(f'✅ 已连接到服务器 {host}:{port}')

    # ---- 昵称设置 ----
    while True:
        nickname = input('请输入昵称: ').strip()
        sock.sendall(f'NICK {nickname}\n'.encode('utf-8'))
        # 接收响应
        data = b''
        while True:
            chunk = sock.recv(1)
            if not chunk:
                print('❌ 服务器断开连接')
                return
            if chunk == b'\n':
                break
            data += chunk
        response = data.decode('utf-8').strip()
        if response == 'NICK_OK':
            print('✅ 昵称设置成功！')
            break
        elif response.startswith('NICK_ERR'):
            print(response)
        else:
            print('服务器错误:', response)

    # ---- 接收线程 ----
    def receive():
        try:
            while True:
                data = b''
                while True:
                    chunk = sock.recv(1)
                    if not chunk:
                        print('❌ 服务器已关闭连接')
                        return
                    if chunk == b'\n':
                        break
                    data += chunk
                line = data.decode('utf-8').strip()
                if line:
                    print(line)   # 简单打印，后续可以格式化
        except:
            pass
        finally:
            sock.close()

    threading.Thread(target=receive, daemon=True).start()

    # ---- 发送主循环 ----
    try:
        while True:
            text = input()
            if text == '/quit':
                sock.sendall(b'/QUIT\n')
                break
            elif text == '/list':
                sock.sendall(b'/LIST\n')
            else:
                sock.sendall((text + '\n').encode('utf-8'))
    except KeyboardInterrupt:
        pass
    finally:
        sock.close()

if __name__ == '__main__':
    main()
```

### 测试方法
1. 启动服务器：`python server_step3.py`
2. 启动客户端A（昵称 Alice），应成功。
3. 启动客户端B（昵称 Bob），应成功。
4. 在 Alice 输入 `/list`，应看到 `LIST Alice Bob`（客户端可能未格式化显示，但能看出列表）。
5. 在 Alice 输入“大家好”，Bob 应收到 `MSG Alice 大家好`。
6. 在 Bob 输入 `/quit`，Bob 退出，Alice 应收到 `INFO Bob 离开了聊天室`。

### 小结
- 实现了用户登录机制，包括昵称唯一性检查。
- 增加了命令解析框架，支持 `/list` 和 `/quit`。
- 广播消息现在包含发送者昵称，消息格式更友好。
- 但客户端显示还很简陋，下一步可以优化显示，并添加私聊功能。

## 第4步：实现私聊功能

### 目标
支持 `/msg 昵称 消息` 命令发送私聊消息。

### 具体任务
1. 在服务器命令解析中添加对 `PM` 命令的处理（客户端输入 `/msg` 时转换为 `PM` 命令发送）。
2. 服务器根据目标昵称找到对应的socket，单独发送消息，并给发送者一个回显。
3. 处理目标用户不在线或发送失败的情况，通知发送者。

### 代码修改
在服务器 `handle_client` 的命令解析部分，添加如下分支：

```python
elif cmd == 'PM' and len(parts) > 1:
    subparts = parts[1].split(' ', 1)
    if len(subparts) == 2:
        target_nick, content = subparts
        with clients_lock:
            target_sock = None
            for sock, nick in clients.items():
                if nick == target_nick:
                    target_sock = sock
                    break
        if target_sock:
            try:
                target_sock.sendall(f'PM {nickname} {content}\n'.encode('utf-8'))
                # 给发送者回显
                client_socket.sendall(f'PM 对 {target_nick}: {content}\n'.encode('utf-8'))
            except:
                client_socket.sendall(f'ERROR 发送给 {target_nick} 失败\n'.encode('utf-8'))
        else:
            client_socket.sendall(f'ERROR 用户 {target_nick} 不在线\n'.encode('utf-8'))
    else:
        client_socket.sendall(b'ERROR PM命令格式: PM 昵称 消息\n')
```

客户端需要增加对 `/msg` 命令的处理，在发送循环中添加：

```python
elif text.startswith('/msg '):
    parts = text.split(' ', 2)
    if len(parts) >= 3:
        sock.sendall(f'PM {parts[1]} {parts[2]}\n'.encode('utf-8'))
    else:
        print('格式错误: /msg 昵称 消息')
```

同时，为了更好地区分消息类型，可以在接收线程中对不同前缀进行格式化显示。例如：

```python
def _display_message(line):
    if line.startswith('MSG '):
        parts = line.split(' ', 2)
        if len(parts) >= 3:
            print(f'[公共] {parts[1]}: {parts[2]}')
        else:
            print(line)
    elif line.startswith('PM '):
        parts = line.split(' ', 2)
        if len(parts) >= 3:
            print(f'[私聊] {parts[1]}: {parts[2]}')
        else:
            print(line)
    elif line.startswith('INFO '):
        print(f'[系统] {line[5:]}')
    elif line.startswith('LIST '):
        users = line[5:].split()
        print(f'在线用户: {", ".join(users)}')
    elif line.startswith('ERROR '):
        print(f'[错误] {line[6:]}')
    else:
        print(line)
```

将接收循环中的 `print(line)` 替换为 `_display_message(line)`。

### 测试方法
1. 启动服务器，两个客户端 Alice 和 Bob。
2. Alice 输入 `/msg Bob 你好`，Bob 应收到 `[私聊] Alice: 你好`，Alice 应收到 `[私聊] 对 Bob: 你好`。
3. Alice 输入 `/msg Tom 你好`，Tom 不在线，Alice 应收到错误提示。

### 小结
- 实现了私聊功能，进一步完善了聊天室的交互性。
- 学习了如何根据昵称查找socket并单独发送消息。
- 客户端显示更加清晰，用户能区分消息类型。

## 第5步：重构为面向对象形式

### 目标
将过程式代码重构为清晰的类结构，提高可读性和可扩展性。

### 具体任务
- 创建 `User` 类，封装客户端socket、地址和昵称。
- 创建 `Server` 类，包含服务器主循环、用户管理、广播和私聊等方法。
- 创建 `ClientHandler` 类（继承 `threading.Thread`），负责单个客户端的通信。
- 将全局变量（`clients`、`lock`）转为 `Server` 的实例变量。
- 调整代码，使各部分通过方法调用交互，保持锁的封装性。

最终代码就是我们提供的完整版本（见后文）。由于重构涉及较大改动，我们直接给出最终代码，并在注释中详细解释每个部分的作用。重构后，代码结构如下：

- `User`：简单的数据类。
- `Server`：
  - `__init__`：初始化socket、users字典、锁。
  - `start()`：启动监听循环，创建 `ClientHandler` 线程。
  - `broadcast(message, exclude)`：广播消息。
  - `send_private(sender_nick, target_nick, content)`：发送私聊。
  - `get_user_by_nick(nickname)`、`remove_user(user)`、`is_nickname_taken(nickname)`、`get_all_nicknames()` 等辅助方法。
- `ClientHandler`（继承 `Thread`）：
  - `__init__(server, user)`：保存server引用和user对象。
  - `run()`：处理昵称设置、主循环、异常处理。
  - `_handle_nickname()`：昵称设置阶段。
  - `_process_command(line)`：命令解析。
  - `_cleanup()`：清理资源。
- `ChatClient`（客户端）：
  - `connect()`、`set_nickname()`、`receive_messages()`、`send_messages()`、`start()` 等方法，与之前类似但封装在类中。

### 测试方法
与第4步相同，所有功能应保持不变。

# 五、完整代码

## server.py（最终版本）
```python
import socket
import threading


class User:
    """用户类：封装客户端连接信息"""

    def __init__(self, sock, addr):
        self.sock = sock  # 客户端的 socket 连接
        self.addr = addr  # 客户端的地址 (IP, 端口)
        self.nickname = None  # 用户的昵称，初始为 None


class Server:
    """服务器类：管理所有客户端连接和消息转发"""

    def __init__(self, host, port):
        self.host = host  # 服务器监听的 IP 地址
        self.port = port  # 服务器监听的端口号
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # 创建 TCP socket
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # 允许地址复用，避免重启时等待
        self.users = {}  # 存储所有在线用户 {socket: User 实例}
        self.lock = threading.Lock()  # 线程锁，保护多线程共享资源的访问安全

    def start(self):
        """启动服务器，进入监听循环"""
        self.sock.bind((self.host, self.port))  # 绑定 IP 和端口
        self.sock.listen(5)  # 开始监听，最多允许 5 个连接排队等待
        print(f'服务器启动，监听 {self.host}:{self.port}')
        try:
            while True:  # 持续接受新连接
                client_sock, addr = self.sock.accept()  # 阻塞等待客户端连接
                user = User(client_sock, addr)  # 创建用户对象,传入socket和地址(IP, 端口)
                handler = ClientHandler(self, user)  # 为该用户创建独立的处理线程,传入服务器实例和用户实例对象
                handler.daemon = True  # 设置为守护线程，主程序退出时自动终止
                handler.start()  # 启动线程
        except KeyboardInterrupt:
            print('服务器关闭')
        finally:
            self.sock.close()  # 关闭服务器 socket

    def broadcast(self, message, exclude=None):
        """
        广播消息给所有在线用户
        :param message: 要发送的消息字符串
        :param exclude: 要排除的 socket（通常是不给自己发消息）
        """
        with self.lock:  # 加锁保护，防止多线程同时修改 users 字典
            users_copy = list(self.users.values())  # 创建副本，避免遍历时被修改
        for user in users_copy:
            if user.sock is exclude:  # 跳过要排除的用户
                continue
            try:
                user.sock.sendall((message + '\n').encode('utf-8'))  # 发送 UTF-8 编码的消息
            except:
                pass  # 如果发送失败（如用户已断开），忽略错误

    def send_private(self, sender_nick, target_nick, content):
        """
        发送私聊消息
        :param sender_nick: 发送者昵称
        :param target_nick: 接收者昵称
        :param content: 消息内容
        """
        target_user = None  # 目标用户对象
        with self.lock:
            for user in self.users.values():
                if user.nickname == target_nick:  # 查找目标用户
                    target_user = user
                    break

        if target_user:  # 如果找到目标用户
            try:
                target_user.sock.sendall(f'PM {sender_nick} {content}\n'.encode('utf-8'))  # 发送给接收者
                # 回显给发送者，确认消息已发送
                sender_user = self.get_user_by_nick(sender_nick)
                if sender_user:
                    sender_user.sock.sendall(f'PM 对 {target_nick}: {content}\n'.encode('utf-8'))
            except:
                # 发送失败，通知发送者
                sender_user = self.get_user_by_nick(sender_nick)
                if sender_user:
                    sender_user.sock.sendall(f'ERROR 发送给 {target_nick} 失败\n'.encode('utf-8'))
        else:
            # 目标用户不在线，通知发送者
            sender_user = self.get_user_by_nick(sender_nick)
            if sender_user:
                sender_user.sock.sendall(f'ERROR 用户 {target_nick} 不在线\n'.encode('utf-8'))

    def get_user_by_nick(self, nickname):
        """根据昵称查找用户对象"""
        with self.lock:
            for user in self.users.values():
                if user.nickname == nickname:
                    return user
        return None  # 未找到返回 None

    def remove_user(self, user):
        """从用户列表中移除用户"""
        with self.lock:
            if user.sock in self.users:
                del self.users[user.sock]

    def is_nickname_taken(self, nickname):
        """检查昵称是否已被其他用户使用"""
        with self.lock:
            # any() 函数：只要有一个用户的昵称匹配就返回 True
            return any(u.nickname == nickname for u in self.users.values())

    def get_all_nicknames(self):
        """获取所有在线用户的昵称列表"""
        with self.lock:
            return [u.nickname for u in self.users.values()]


class ClientHandler(threading.Thread):
    """客户端处理器：每个客户端一个独立线程，处理该客户端的所有通信"""

    def __init__(self, server, user):
        super().__init__()
        self.server = server  # 服务器实例引用
        self.user = user  # 当前处理的用戶对象

    def run(self):
        """线程的主执行方法"""
        # ---- 第一步：处理昵称设置 ----
        if not self._handle_nickname():
            return  # 如果昵称设置失败，直接结束

        # 广播通知其他用户：有人加入了聊天室
        self.server.broadcast(f'INFO {self.user.nickname} 加入了聊天室', exclude=self.user.sock)

        # ---- 第二步：进入主消息循环 ----
        try:
            while True:
                data = b''
                # 逐字节接收数据，直到遇到换行符
                while True:
                    chunk = self.user.sock.recv(1)  # 每次接收 1 字节
                    if not chunk:  # 没有数据表示客户端断开连接
                        return
                    if chunk == b'\n':  # 遇到换行符表示一行消息结束
                        break
                    data += chunk  # 累加数据

                if not data:  # 如果数据为空，跳出循环
                    break

                line = data.decode('utf-8').strip()  # 解码并去除首尾空白字符
                if not line:  # 空行则忽略
                    continue

                self._process_command(line)  # 处理命令或消息
        except Exception as e:
            print(f'处理 {self.user.nickname} 异常：{e}')
        finally:
            self._cleanup()  # 清理资源

    def _handle_nickname(self):
        """
        处理昵称设置阶段
        :return: 成功返回 True，失败返回 False
        """
        while True:
            try:
                data = b''
                # 逐字节接收，直到遇到换行符
                while True:
                    chunk = self.user.sock.recv(1)
                    if not chunk:  # 客户端断开
                        return False
                    if chunk == b'\n':
                        break
                    data += chunk

                line = data.decode('utf-8').strip()

                if line.startswith('NICK '):  # 检查是否是 NICK 命令
                    nickname = line[5:].strip()  # 提取昵称部分（去掉"NICK "前缀）

                    if self.server.is_nickname_taken(nickname):
                        # 昵称已被占用，发送错误消息
                        self.user.sock.sendall('NICK_ERR 昵称已被占用\n'.encode('utf-8'))
                    else:
                        # 昵称可用，保存并继续
                        self.user.nickname = nickname
                        with self.server.lock:
                            self.server.users[self.user.sock] = self.user  # 加入用户列表
                        self.user.sock.sendall(b'NICK_OK\n')  # 发送成功确认
                        return True
                else:
                    # 客户端发送的不是 NICK 命令，提示错误
                    self.user.sock.sendall('ERROR 请先发送 NICK 命令设置昵称\n'.encode('utf-8'))
            except:
                return False  # 发生异常，返回失败

    def _process_command(self, line):
        """
        处理客户端发送的命令或消息
        :param line: 解码后的一行文本
        """
        if line.startswith('/'):  # 以/开头的是命令
            parts = line.split(' ', 1)  # 分割命令和参数
            cmd = parts[0][1:].upper()  # 去掉/并转为大写

            if cmd == 'LIST':  # /LIST - 列出所有在线用户
                nicks = self.server.get_all_nicknames()
                self.user.sock.sendall(f'LIST {" ".join(nicks)}\n'.encode('utf-8'))

            elif cmd == 'QUIT':  # /QUIT - 退出聊天室
                self.user.sock.sendall(b'BYE\n')
                raise Exception('quit')  # 抛出异常触发清理

            elif cmd == 'PM' and len(parts) > 1:  # /PM - 私聊消息
                subparts = parts[1].split(' ', 1)  # 分割目标昵称和消息内容
                if len(subparts) == 2:
                    target, content = subparts
                    self.server.send_private(self.user.nickname, target, content)
                else:
                    self.user.sock.sendall('ERROR PM 命令格式：PM 昵称 消息\n'.encode('utf-8'))
            else:
                self.user.sock.sendall('ERROR 未知命令\n'.encode('utf-8'))
        else:
            # 普通消息，广播给所有人
            self.server.broadcast(f'MSG {self.user.nickname} {line}', exclude=None)

    def _cleanup(self):
        """清理用户断开连接后的资源"""
        if self.user.nickname:  # 如果用户已经设置了昵称（已完成登录）
            self.server.remove_user(self.user)  # 从用户列表中移除
            # 广播通知其他用户
            self.server.broadcast(f'INFO {self.user.nickname} 离开了聊天室', exclude=None)

        self.user.sock.close()  # 关闭 socket 连接
        print(f'客户端 {self.user.addr} 断开连接')


def main():
    """程序入口函数"""
    server = Server('127.0.0.1', 8888)  # 创建服务器实例，监听本地 8888 端口
    server.start()  # 启动服务器


if __name__ == '__main__':
    main()

```

## client.py（最终版本）
```python
import socket
import threading


class ChatClient:
    """聊天室客户端：处理与服务器的连接、消息收发"""

    def __init__(self, host, port):
        self.host = host  # 服务器 IP 地址
        self.port = port  # 服务器端口号
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # 创建 TCP socket
        self.nickname = None  # 保存当前用户的昵称

    def connect(self):
        """连接到服务器"""
        self.sock.connect((self.host, self.port))  # 建立 TCP 连接
        print(f'已连接到服务器 {self.host}:{self.port}')

    def _recv_line(self):
        """
        从服务器接收一行数据（以换行符结束）
        :return: 解码后的字符串，如果连接断开返回 None
        """
        data = b''
        while True:
            chunk = self.sock.recv(1)  # 每次接收 1 字节
            if not chunk:  # 没有数据表示服务器断开了连接
                return None
            if chunk == b'\n':  # 遇到换行符表示一行结束
                break
            data += chunk  # 累加字节
        return data.decode('utf-8').strip()  # 解码为字符串并去除首尾空白

    def set_nickname(self):
        """
        设置昵称，直到成功或连接断开
        :return: 成功返回 True，失败返回 False
        """
        while True:
            nickname = input('请输入昵称：').strip()  # 获取用户输入的昵称
            self.sock.sendall(f'NICK {nickname}\n'.encode('utf-8'))  # 发送 NICK 命令

            response = self._recv_line()  # 等待服务器响应

            if response is None:  # 服务器断开连接
                print('服务器断开连接')
                return False

            if response == 'NICK_OK':  # 昵称设置成功
                self.nickname = nickname
                print('昵称设置成功！')
                return True

            elif response.startswith('NICK_ERR'):  # 昵称已被占用
                print(response)  # 打印错误信息，让用户重新输入

            else:  # 其他未知响应
                print('服务器错误:', response)

    def receive_messages(self):
        """
        持续接收服务器消息的函数（在后台线程运行）
        这个函数会一直运行，直到连接断开
        """
        try:
            while True:
                line = self._recv_line()  # 接收一行消息
                if line is None:  # 连接断开
                    break
                self._display_message(line)  # 显示消息
        except:
            pass  # 发生异常时静默处理
        finally:
            self.sock.close()  # 关闭 socket

    def _display_message(self, line):
        """
        解析并显示服务器发来的消息
        :param line: 服务器发来的一行文本
        """
        if line.startswith('MSG '):  # 公共聊天消息
            parts = line.split(' ', 2)  # 分割成 ['MSG', '昵称', '消息内容']
            if len(parts) >= 3:
                print(f'[公共] {parts[1]}: {parts[2]}')  # 格式：[公共] 小明：大家好
            else:
                print(line)

        elif line.startswith('PM '):  # 私聊消息
            parts = line.split(' ', 2)
            if len(parts) >= 3:
                print(f'[私聊] {parts[1]}: {parts[2]}')  # 格式：[私聊] 小红：你好
            else:
                print(line)

        elif line.startswith('INFO '):  # 系统通知（有人加入/离开）
            print(f'[系统] {line[5:]}')  # 去掉'INFO '前缀，显示剩余内容

        elif line.startswith('LIST '):  # 在线用户列表
            users = line[5:].split()  # 去掉'LIST '并按空格分割
            print(f'在线用户：{", ".join(users)}')  # 用逗号分隔显示

        elif line.startswith('ERROR '):  # 错误消息
            print(f'[错误] {line[6:]}')  # 去掉'ERROR '前缀

        else:  # 其他未知格式的消息
            print(line)

    def send_messages(self):
        """
        处理用户输入并发送到服务器（在主线程运行）
        支持命令：/quit、/list、/msg
        """
        try:
            while True:
                text = input()  # 等待用户输入

                if text == '/quit':  # 退出聊天室
                    self.sock.sendall(b'/QUIT\n')
                    break

                elif text == '/list':  # 查看在线用户
                    self.sock.sendall(b'/LIST\n')

                elif text.startswith('/msg '):  # 发送私聊消息
                    parts = text.split(' ', 2)  # 分割成 ['/msg', '昵称', '消息']
                    if len(parts) >= 3:
                        # 转换为服务器协议格式：PM 昵称 消息
                        self.sock.sendall(f'/PM {parts[1]} {parts[2]}\n'.encode('utf-8'))
                    else:
                        print('格式错误：/msg 昵称 消息')

                else:  # 普通聊天消息（不是命令）
                    self.sock.sendall((text + '\n').encode('utf-8'))

        except (KeyboardInterrupt, EOFError):
            # KeyboardInterrupt: 用户按 Ctrl+C
            # EOFError: 输入流意外结束
            pass
        finally:
            self.sock.close()  # 关闭连接

    def start(self):
        """启动客户端的主流程"""
        self.connect()  # 第 1 步：连接服务器

        if not self.set_nickname():  # 第 2 步：设置昵称
            return  # 如果昵称设置失败，直接退出

        # 第 3 步：启动后台线程接收消息
        recv_thread = threading.Thread(target=self.receive_messages, daemon=True)
        recv_thread.start()

        # 第 4 步：主线程负责发送消息
        self.send_messages()


def main():
    """程序入口函数"""
    client = ChatClient('127.0.0.1', 8888)  # 创建客户端实例，连接本地 8888 端口
    client.start()  # 启动客户端


if __name__ == '__main__':
    main()

```

# 六、常见问题与解决方案

## 1. 地址已被占用
- **现象**：运行服务器时抛出 `OSError: [Errno 98] Address already in use` 或类似错误。
- **原因**：上次服务器关闭后，端口仍处于 TIME_WAIT 状态。
- **解决**：在创建socket后添加 `server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)`，允许地址复用。

## 2. 客户端连接后立即断开
- **现象**：客户端一连接就提示断开，没有进入昵称设置。
- **可能原因**：服务器端在处理昵称时没有正确循环，或者客户端没有发送 NICK 命令。
- **检查**：确保服务器在昵称设置阶段持续读取，客户端在连接后立即发送 NICK。

## 3. 广播时出现 `RuntimeError: dictionary changed size during iteration`
- **现象**：服务器运行时抛出该异常。
- **原因**：在遍历字典时，另一个线程修改了字典（如添加或删除用户）。
- **解决**：遍历前复制字典或列表，如 `list(self.users.items())`，而不是直接遍历原字典。

## 4. 私聊消息发送失败
- **现象**：用户发送私聊消息后，没有收到任何反馈。
- **可能原因**：目标用户可能已经断开，但字典中尚未移除；或者发送时目标socket已不可写。
- **解决**：在发送前检查socket，但更简单的是捕获异常并通知发送者，同时由目标用户对应的线程负责清理。

## 5. 客户端接收线程卡死
- **现象**：客户端可以发送消息，但收不到任何回复，或者退出时 hang 住。
- **可能原因**：`recv(1)` 是阻塞的，如果服务器不发送数据，它会一直等待。但连接断开时 `recv` 会返回空字节，所以应确保退出条件正确。
- **解决**：检查接收循环中是否正确处理了 `recv` 返回空的情况，并退出循环。

## 6. 多个客户端同时发送消息时乱序
- **现象**：接收到的消息顺序混乱。
- **原因**：TCP 保证数据按顺序到达，但多线程情况下，如果服务器在处理广播时先加锁再发送，顺序是确定的。但如果有线程调度延迟，显示顺序可能与发送顺序略有不同，这是正常的。如果需要严格顺序，可以在发送时附加时间戳或序号。

# 七、项目扩展建议

完成基础聊天室后，你可以尝试以下扩展，进一步提升技能：

1. **图形界面**：使用 Tkinter 或 PyQt 为客户端添加 GUI，需要将网络通信与 UI 线程分离，可以使用 `queue.Queue` 传递消息。
2. **文件传输**：扩展协议，支持发送文件。可以单独建立一个 TCP 连接用于文件传输，或在现有连接上使用特殊命令和 base64 编码。
3. **聊天记录**：服务器端将消息写入日志文件，客户端可保存聊天记录到本地。
4. **加密通信**：使用 `ssl` 模块包装 socket，实现加密传输，防止消息被窃听。
5. **多房间**：支持创建/加入不同聊天室，每个房间独立，用户可切换房间。
6. **性能优化**：使用 `select`、`poll` 或 `epoll` 替代多线程，实现单线程异步 I/O，提高并发性能（适合作为进阶练习）。
7. **用户认证**：增加密码登录功能，使用数据库存储用户信息。
8. **Web 界面**：使用 WebSocket 实现浏览器客户端，与 Python 后端通信。

# 八、核心总结

本节课作为网络编程的综合实战篇，我们基于前两篇的 TCP 基础，从零到一构建了一个多用户聊天室项目，核心要点回顾，务必记牢：

1. **项目整体流程**：从单客户端通信开始，逐步引入多线程、昵称系统、命令解析、私聊功能，最后重构为面向对象形式，每一步都确保代码可运行，体现了增量开发的思路。
2. **TCP通信标准流程**：服务端固定7步，客户端固定5步，必须添加端口复用和超时设置，避免基础坑点。
3. **多线程并发模型**：主线程负责监听接入，每个客户端分配独立子线程，子线程必须设为守护线程，避免僵尸线程；使用锁保护共享数据，避免竞态条件。
4. **自定义文本行协议**：以换行符为消息边界，命令和参数用空格分隔，简单易实现，方便调试和扩展。
5. **面向对象设计**：通过 `User`、`Server`、`ClientHandler`、`ChatClient` 类清晰划分职责，提高代码可读性和可维护性。
6. **核心避坑点**：必配端口复用、设置Socket超时、使用守护线程、遍历共享数据时复制副本、捕获异常并清理资源，这5点是TCP多线程开发的基础要求。
7. **IP+端口的实际应用**：服务端绑定 `0.0.0.0` 支持对外访问，客户端无需手动绑定端口，由系统自动分配，完全遵循前两篇的IP/端口规则。

至此，我们已经掌握了TCP网络编程的综合应用能力，能独立开发可靠的多用户聊天室程序，为后续开发更复杂的网络应用（如即时通讯、游戏服务器、物联网平台）打下坚实基础。

# 九、专栏订阅

> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/161491731>
