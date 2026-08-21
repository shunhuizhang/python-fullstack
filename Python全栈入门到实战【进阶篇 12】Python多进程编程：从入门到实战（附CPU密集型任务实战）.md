

# Python全栈入门到实战【进阶篇 12】Python多进程编程：从入门到实战（附CPU密集型任务实战）
在上一节中，我们掌握了线程池的核心用法，它能高效应对**IO密集型场景**（爬虫、文件处理、接口调用）。但面对“大量计算、复杂循环、数据处理”等**CPU密集型任务**时，线程池依然会受Python GIL锁的限制，无法充分利用多核CPU——而**多进程编程**，正是突破GIL锁、最大化利用多核资源、解决CPU密集型任务并发的核心方案。

本节课聚焦**多进程**，从“新手能懂的痛点”到“企业级实战”，一次性讲透：**为什么用→怎么用→通信同步→实战场景→选型对比**，全程搭配可直接复用的代码示例，新手也能快速上手，完美衔接线程、线程池的知识点。

本节核心学习内容：
- 多进程核心价值：彻底突破GIL锁，充分利用多核CPU（大白话讲透）
- 进程创建的3种方式：Process类/进程池Pool/fork（覆盖所有场景）
- 多进程通信：3种核心方式（Queue/Pipe/Manager）详解（附代码）
- 多进程同步：避免资源竞争（Lock/Semaphore同步机制）
- 实战：CPU密集型任务批量计算（对比单线程/多线程，直观见效率）
- 多进程vs多线程vs线程池：一张表分清不同场景该用谁
- 新手必避的6个坑：进程开销、通信陷阱、资源释放

@[toc](文章目录)

# 一、为什么需要多进程？（GIL锁的“死穴”）
我们已经多次提到GIL锁，但在多进程学习前，必须再明确它的核心局限——这也是多进程存在的意义：
1.  GIL锁本质：同一时刻，Python解释器只允许**一个线程**执行字节码，即使是多核CPU，线程也只能“交替执行”，无法真正并行；
2.  线程池的局限：线程池本质还是“多线程”，依然受GIL锁限制，面对CPU密集型任务，多核CPU的性能被完全浪费；
3.  多进程的突破：每个进程都有独立的Python解释器和内存空间，各自拥有独立的GIL锁，互不干扰——也就是说，多核CPU可以同时执行多个进程，真正实现“并行计算”，彻底释放多核性能。

举个直观例子：用单线程、线程池、多进程分别执行“计算100万次平方和”的任务（CPU密集型）：
- 单线程：耗时8秒（单核执行，全程占用1个CPU核心）
- 线程池（8线程）：耗时7.8秒（受GIL限制，依然单核执行，线程切换还会增加少量开销）
- 多进程（8进程）：耗时1.2秒（利用8核CPU并行执行，效率提升6倍以上）

简单来说：**多进程的核心价值，就是突破GIL锁，让CPU密集型任务实现真正的并行，最大化利用多核资源**。

# 二、核心概念回顾与补充
在学习多进程前，我们先回顾+补充3个核心概念，避免和线程、线程池混淆：
1.  进程：独立的“程序实例”，拥有独立的内存空间、Python解释器，每个进程都有自己的GIL锁，互不干扰；
2.  线程：进程内的执行单元，共享进程的内存和资源，受同一个GIL锁限制；
3.  多进程vs多线程：
    - 多线程：适合IO密集型（等待时间长，CPU空闲），开销小、切换快，但受GIL限制；
    - 多进程：适合CPU密集型（计算时间长，CPU满载），开销大、切换慢，但能突破GIL，利用多核；
4.  进程的生命周期：创建→就绪→运行→阻塞→终止（和线程类似，但进程的创建/销毁开销远大于线程）。

Python中实现多进程的核心模块有两个：
- `multiprocessing`：Python3内置模块，支持跨平台（Windows、Linux、Mac），功能全面，推荐新手优先使用；
- `os.fork()`：仅支持Linux/Mac，基于系统fork机制创建进程，Windows不支持，适合Linux环境开发。

# 三、进程创建的3种方式（覆盖所有场景，最简实现）
和多线程类似，多进程也有多种创建方式，不同方式对应不同场景，我们逐一讲解，优先掌握前2种（跨平台、常用）。

## 方式1：直接使用multiprocessing.Process类（推荐，简洁灵活）
用法和`threading.Thread`高度相似，新手可快速上手，适合创建少量进程、自定义进程逻辑的场景。

```python
from multiprocessing import Process
import time
import os  # 用于获取进程ID，区分不同进程

# 定义进程要执行的任务函数
def process_task(name, num):
    """模拟CPU密集型任务：计算num次平方和"""
    print(f"进程{name}（PID：{os.getpid()}）开始执行，计算{num}次平方和")
    # 模拟CPU密集型计算（无IO操作，全程占用CPU）
    total = 0
    for i in range(num):
        total += i * i
    print(f"进程{name}执行完成，计算结果：{total}（父进程PID：{os.getppid()}）")
    return total

# 1. 创建进程对象（和Thread类用法几乎一致）
if __name__ == "__main__":  # Windows系统必须加这行，避免进程创建异常
    # 获取CPU核心数，用于合理设置进程数
    cpu_count = os.cpu_count()
    print(f"当前机器CPU核心数：{cpu_count}")

    # 创建3个进程（建议进程数 ≤ CPU核心数，避免切换开销）
    p1 = Process(target=process_task, args=("p1", 10000000), name="进程1")
    p2 = Process(target=process_task, args=("p2", 10000000), name="进程2")
    p3 = Process(target=process_task, args=("p3", 10000000), name="进程3")

    # 2. 启动进程（和线程start()用法一致）
    start_time = time.time()
    p1.start()
    p2.start()
    p3.start()

    # 3. 等待进程执行完成（join()，和线程用法一致）
    p1.join()
    p2.join()
    p3.join()

    end_time = time.time()
    print(f"\n所有进程执行完毕，总耗时：{round(end_time - start_time, 2)}秒")
```

**运行结果：**

```
当前机器CPU核心数：16
进程p2（PID：20168）开始执行，计算10000000次平方和
进程p1（PID：7764）开始执行，计算10000000次平方和
进程p3（PID：9552）开始执行，计算10000000次平方和
进程p1执行完成，计算结果：333333283333335000000（父进程PID：17536）
进程p2执行完成，计算结果：333333283333335000000（父进程PID：17536）
进程p3执行完成，计算结果：333333283333335000000（父进程PID：17536）

所有进程执行完毕，总耗时：0.75秒
```

### 核心说明

1.  `if __name__ == "__main__":`：**Windows系统必须加这行**！因为Windows系统创建进程时，会重新导入当前模块，不加这行会导致进程无限递归创建，抛出异常；Linux/Mac可省略，但建议统一加上，保证跨平台兼容性。
2.  进程ID相关：
    - `os.getpid()`：获取当前进程的ID（唯一标识）；
    - `os.getppid()`：获取当前进程的父进程ID（创建当前进程的进程）；
3.  Process类核心参数：
    - `target`：进程要执行的任务函数；
    - `args`：传给任务函数的位置参数（元组，即使只有一个参数也要加逗号）；
    - `name`：进程名（可选，默认格式为`Process-1`、`Process-2`）；
    - `daemon`：是否为守护进程（和线程守护进程类似，主线程退出时，守护进程强制终止，需在start()前设置）；
4.  进程的启动与关闭：
    - `start()`：启动进程（真正创建进程，调用任务函数），不可重复调用；
    - `join(timeout=None)`：主线程等待目标进程完成，timeout为超时时间；
    - `terminate()`：强制终止进程（类似“杀进程”，不推荐轻易使用，可能导致资源泄漏）；
    - `is_alive()`：判断进程是否存活（返回True/False）。

## 方式2：进程池Pool（批量CPU密集型任务首选）
和线程池类似，进程池（`multiprocessing.Pool`）用于管理多个进程，实现进程复用，避免频繁创建/销毁进程的高额开销——适合**批量执行CPU密集型任务**（如批量计算、数据处理），用法和线程池高度相似，降低学习成本。

### 2.1 基础用法：apply/apply_async（提交单个任务）
```python
from multiprocessing import Pool
import time
import os

# 任务函数（CPU密集型：计算num的阶乘）
def factorial_task(num):
    """计算num的阶乘（模拟CPU密集型任务）"""
    print(f"进程{os.getpid()}开始计算{num}的阶乘")
    result = 1
    for i in range(1, num+1):
        result *= i
    time.sleep(0.5)  # 模拟少量耗时（实际可删除，仅为演示）
    print(f"进程{os.getpid()}计算完成，{num}的阶乘：{result}")
    return result

if __name__ == "__main__":
    # 1. 创建进程池（processes：最大进程数，建议 ≤ CPU核心数）
    with Pool(processes=os.cpu_count()) as pool:
        # 2. 提交任务（两种方式）
        # 方式1：apply() 阻塞式提交（提交一个任务，等待完成后再提交下一个，不推荐）
        # res1 = pool.apply(factorial_task, args=(5,))
        # print(f"apply方式结果：{res1}")

        # 方式2：apply_async() 异步式提交（非阻塞，批量提交，推荐）
        start_time = time.time()
        # 批量提交5个任务，返回Future对象（和线程池submit()返回值类似）
        futures = [pool.apply_async(factorial_task, args=(i,)) for i in [5, 6, 7, 8, 9]]

        # 3. 获取任务结果（get() 阻塞式获取，可设置超时）
        results = [future.get(timeout=3) for future in futures]

        end_time = time.time()
        print(f"\n所有任务执行完毕，总耗时：{round(end_time - start_time, 2)}秒")
        print(f"批量任务结果：{results}")
```

**运行结果：**

```
进程924开始计算5的阶乘
进程5376开始计算6的阶乘
进程16752开始计算7的阶乘
进程17796开始计算8的阶乘
进程21644开始计算9的阶乘
进程924计算完成，5的阶乘：120
进程5376计算完成，6的阶乘：720
进程16752计算完成，7的阶乘：5040
进程17796计算完成，8的阶乘：40320
进程21644计算完成，9的阶乘：362880

所有任务执行完毕，总耗时：1.01秒
批量任务结果：[120, 720, 5040, 40320, 362880]
```

### 2.2 简化用法：map/map_async（批量任务首选）

如果任务函数相同、参数不同，用`map()`/`map_async()`更简洁，类似线程池的`map()`方法，自动分配进程执行任务，按参数顺序返回结果。

```python
from multiprocessing import Pool
import time
import os

# 任务函数（CPU密集型：计算num的平方和）
def sum_square_task(num):
    total = 0
    for i in range(num):
        total += i * i
    print(f"进程{os.getpid()}完成计算，1~{num-1}平方和：{total}")
    return total

if __name__ == "__main__":
    # 创建进程池，processes设为CPU核心数
    with Pool(processes=os.cpu_count()) as pool:
        # 批量提交任务（参数列表为[1000000, 2000000, ..., 5000000]）
        start_time = time.time()
        # map() 阻塞式，自动分配任务，按参数顺序返回结果
        results = pool.map(sum_square_task, [1000000, 2000000, 3000000, 4000000, 5000000])
        end_time = time.time()

        print(f"\n批量任务结果：{results}")
        print(f"总耗时：{round(end_time - start_time, 2)}秒")
```

**运行结果：**

```
进程5148完成计算，1~999999平方和：333332833333500000
进程20396完成计算，1~1999999平方和：2666664666667000000
进程12248完成计算，1~2999999平方和：8999995500000500000
进程17600完成计算，1~3999999平方和：21333325333334000000
进程8428完成计算，1~4999999平方和：41666654166667500000

批量任务结果：[333332833333500000, 2666664666667000000, 8999995500000500000, 21333325333334000000, 41666654166667500000]
总耗时：0.62秒
```

### 核心说明

1.  进程池核心参数：`processes`（最大进程数），建议设置为**CPU核心数**（过多进程会导致系统调度开销增加，反而降低效率）；
2.  两种提交方式对比：
    - 阻塞式（apply/map）：提交任务后，主线程等待任务完成，无法并行执行其他逻辑，不推荐；
    - 异步式（apply_async/map_async）：提交任务后，主线程继续执行，任务在进程池中并行执行，推荐用于批量任务；
3.  进程池关闭：`with`语句会自动调用`pool.close()`（关闭进程池，不再接受新任务）和`pool.join()`（等待所有任务完成），无需手动关闭；
4.  结果获取：`apply_async()`返回`AsyncResult`对象，需用`get(timeout=None)`获取结果；`map()`直接返回结果列表（按参数顺序）。

## 方式3：fork()创建进程（仅Linux/Mac支持）
`os.fork()`是基于Unix/Linux系统的进程创建机制，Windows系统不支持（会抛出异常），适合Linux/Mac环境下的底层开发，新手了解即可。

```python
import os
import time

if __name__ == "__main__":
    print(f"父进程（PID：{os.getpid()}）开始执行")

    # fork() 创建子进程：调用一次，返回两次（父进程返回子进程PID，子进程返回0）
    pid = os.fork()

    if pid == 0:
        # 子进程执行逻辑（pid=0表示当前是子进程）
        print(f"子进程（PID：{os.getpid()}）创建成功，父进程PID：{os.getppid()}")
        time.sleep(2)
        print("子进程执行完成，退出")
    else:
        # 父进程执行逻辑（pid为子进程ID）
        print(f"父进程创建子进程，子进程PID：{pid}")
        # 等待子进程完成
        os.wait()
        print("父进程执行完成，退出")
```

### 核心说明
- `os.fork()`：调用一次，返回两次——父进程返回子进程的PID（非0整数），子进程返回0；
- 子进程会复制父进程的所有内存空间、代码、数据，父子进程互不干扰；
- 缺点：仅支持Linux/Mac，无法跨平台；手动管理父子进程复杂，不适合新手和批量任务。

# 四、多进程通信：3种核心方式（附代码，实战必备）
多进程拥有独立的内存空间，无法像线程那样直接共享全局变量——因此，我们需要专门的“通信机制”，实现多进程之间的数据传递。Python提供3种核心通信方式，覆盖不同场景：

## 方式1：Queue（队列，最常用，安全便捷）

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/44a54a4ee17949438f3f986e6c0449cf.png#pic_center)

`multiprocessing.Queue`是线程安全、进程安全的队列，用于实现“生产者-消费者”模型，适合**多进程之间批量传递数据**（如进程A生产数据，进程B/C消费数据），用法和线程的`queue.Queue`高度相似。

```python
from multiprocessing import Process, Queue
import time
import random

# 生产者进程：生产数据，放入队列
def producer(queue, name):
    for i in range(5):
        # 生产随机数据
        data = random.randint(1, 100)
        queue.put(data)  # 放入队列（队列满时阻塞）
        print(f"生产者{name}生产数据：{data}，队列当前大小：{queue.qsize()}")
        time.sleep(random.uniform(0.1, 0.3))  # 模拟生产耗时

# 消费者进程：从队列取数据，处理数据
def consumer(queue, name):
    while True:
        try:
            # 取出数据（timeout=2：2秒没数据则抛异常，退出循环）
            data = queue.get(timeout=2)
            print(f"消费者{name}消费数据：{data}，队列剩余：{queue.qsize()}")
            time.sleep(random.uniform(0.2, 0.5))  # 模拟消费耗时
        except:
            print(f"消费者{name}：队列空，停止消费")
            break

if __name__ == "__main__":
    # 1. 创建进程安全的队列（maxsize：队列最大容量，可选）
    q = Queue(maxsize=3)

    # 2. 创建生产者和消费者进程
    p1 = Process(target=producer, args=(q, "P1"))
    p2 = Process(target=producer, args=(q, "P2"))
    c1 = Process(target=consumer, args=(q, "C1"))
    c2 = Process(target=consumer, args=(q, "C2"))

    # 3. 启动进程（先启动生产者，再启动消费者）
    p1.start()
    p2.start()
    c1.start()
    c2.start()

    # 4. 等待生产者完成，再等待队列消费完毕
    p1.join()
    p2.join()
    q.close()  # 关闭队列，不再接受新数据
    c1.join()
    c2.join()

    print("所有生产/消费任务完成")
```

**运行结果：**

```
生产者P1生产数据：35，队列当前大小：1
消费者C1消费数据：35，队列剩余：1
生产者P2生产数据：99，队列当前大小：1
消费者C2消费数据：99，队列剩余：0
生产者P1生产数据：40，队列当前大小：1
消费者C2消费数据：40，队列剩余：0
生产者P2生产数据：78，队列当前大小：1
生产者P1生产数据：87，队列当前大小：2
消费者C2消费数据：78，队列剩余：1
消费者C1消费数据：87，队列剩余：0
生产者P1生产数据：34，队列当前大小：1
生产者P2生产数据：91，队列当前大小：2
生产者P1生产数据：13，队列当前大小：3
消费者C1消费数据：34，队列剩余：3
生产者P2生产数据：61，队列当前大小：3
消费者C2消费数据：91，队列剩余：2
生产者P2生产数据：19，队列当前大小：3
消费者C1消费数据：13，队列剩余：2
消费者C2消费数据：61，队列剩余：1
消费者C2消费数据：19，队列剩余：0
消费者C1：队列空，停止消费
消费者C2：队列空，停止消费
所有生产/消费任务完成
```

### 核心说明

- `Queue`是跨平台的，支持Windows/Linux/Mac；
- 核心方法：`put(item)`（放入数据）、`get(timeout=None)`（取出数据）、`qsize()`（获取队列大小）、`close()`（关闭队列）；
- 优点：线程安全、进程安全，无需手动加锁，用法简单，适合批量数据传递；
- 缺点：仅支持父子进程、同进程池内的进程通信，不支持无亲缘关系的进程通信。

## 方式2：Pipe（管道，高效，适合双向通信）

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8893a7c43dac499d81665f8313320e08.png#pic_center)

`multiprocessing.Pipe`用于实现**两个进程之间的双向通信**，效率比Queue高（无需中间缓存），适合“两个进程一对一通信”的场景（如进程A发送指令，进程B执行并返回结果）。

```python
from multiprocessing import Process, Pipe
import time

# 进程1：发送数据，接收响应
def process_a(conn):
    # 发送数据给进程B
    send_data = ["Python", "多进程", "Pipe通信"]
    for data in send_data:
        conn.send(data)
        print(f"进程A发送：{data}")
        time.sleep(1)

    # 接收进程B的响应
    for _ in range(len(send_data)):  # 根据发送数据的数量接收响应
        try:
            recv_data = conn.recv()
            print(f"进程A接收：{recv_data}")
        except:
            print("进程A：接收完毕，关闭连接")
            break
    print("进程A：接收完毕，关闭连接")
    # 关闭连接
    conn.close()

# 进程2：接收数据，返回响应
def process_b(conn):
    count = 0  # 计数器
    total_messages = 3  # 已知发送的数据总数

    while count < total_messages:
        try:
            # 接收进程A的数据
            recv_data = conn.recv()
            print(f"进程B接收：{recv_data}")
            # 返回响应
            conn.send(f"已收到：{recv_data}")
            time.sleep(0.5)
            count += 1  # 接收成功后计数加一
        except:
            print("进程B：接收过程中发生异常")
            break

    print("进程B：接收完毕，关闭连接")
    conn.close()

if __name__ == "__main__":
    # 1. 创建Pipe管道，返回两个连接对象（conn_a和conn_b，双向通信）
    conn_a, conn_b = Pipe()

    # 2. 创建两个进程，分别传入两个连接对象
    p1 = Process(target=process_a, args=(conn_a,))
    p2 = Process(target=process_b, args=(conn_b,))

    # 3. 启动进程
    p1.start()
    p2.start()

    # 4. 等待进程完成
    p1.join()
    p2.join()

    print("Pipe双向通信完成")

```

**运行结果：**

```
进程A发送：Python
进程B接收：Python
进程A发送：多进程
进程B接收：多进程
进程A发送：Pipe通信
进程B接收：Pipe通信
进程B：接收完毕，关闭连接
进程A接收：已收到：Python
进程A接收：已收到：多进程
进程A接收：已收到：Pipe通信
进程A：接收完毕，关闭连接
Pipe双向通信完成
```

### 核心说明

- `Pipe()`：创建管道，返回两个`Connection`对象（conn1, conn2），两个对象可双向通信（既能发送，也能接收）；
- 核心方法：`send(item)`（发送数据）、`recv()`（接收数据）、`close()`（关闭连接）；
- 优点：效率高，适合两个进程一对一通信，支持双向传递；
- 缺点：仅支持两个进程通信，不支持多个进程共享，且不是进程安全的（多个进程同时操作一个连接会导致数据错乱）。

## 方式3：Manager（管理器，支持多进程共享数据）
`multiprocessing.Manager`用于实现**多个进程共享数据**（如共享列表、字典、变量），底层通过网络通信实现，支持跨进程共享复杂数据结构，适合“多个进程需要读写同一组数据”的场景。

```python
from multiprocessing import Process, Manager
import time

# 任务函数：修改共享字典和列表
def update_shared_data(shared_dict, shared_list, name):
    print(f"进程{name}开始修改共享数据")
    # 修改共享字典
    shared_dict[name] = f"进程{name}的数据"
    # 修改共享列表
    shared_list.append(f"进程{name}添加的数据")
    # 修改共享变量（需用Manager的Value/Array）
    # 此处省略，后续补充Value/Array用法
    time.sleep(1)
    print(f"进程{name}修改完成，共享字典：{shared_dict}，共享列表：{shared_list}")

if __name__ == "__main__":
    # 1. 创建Manager管理器
    with Manager() as manager:
        # 2. 创建共享数据（字典、列表）
        shared_dict = manager.dict()  # 共享字典
        shared_list = manager.list()  # 共享列表

        # 3. 创建3个进程，同时修改共享数据
        p1 = Process(target=update_shared_data, args=(shared_dict, shared_list, "P1"))
        p2 = Process(target=update_shared_data, args=(shared_dict, shared_list, "P2"))
        p3 = Process(target=update_shared_data, args=(shared_dict, shared_list, "P3"))

        # 4. 启动进程
        p1.start()
        p2.start()
        p3.start()

        # 5. 等待进程完成
        p1.join()
        p2.join()
        p3.join()

        # 6. 查看最终的共享数据
        print(f"\n最终共享字典：{shared_dict}")
        print(f"最终共享列表：{shared_list}")
```

**运行结果：**

```
进程P3开始修改共享数据
进程P1开始修改共享数据
进程P2开始修改共享数据
进程P3修改完成，共享字典：{'P3': '进程P3的数据', 'P1': '进程P1的数据', 'P2': '进程P2的数据'}，共享列表：['进程P3添加的数据', '进程P1添加的数据', '进程P2添加的数据']
进程P1修改完成，共享字典：{'P3': '进程P3的数据', 'P1': '进程P1的数据', 'P2': '进程P2的数据'}，共享列表：['进程P3添加的数据', '进程P1添加的数据', '进程P2添加的数据']
进程P2修改完成，共享字典：{'P3': '进程P3的数据', 'P1': '进程P1的数据', 'P2': '进程P2的数据'}，共享列表：['进程P3添加的数据', '进程P1添加的数据', '进程P2添加的数据']

最终共享字典：{'P3': '进程P3的数据', 'P1': '进程P1的数据', 'P2': '进程P2的数据'}
最终共享列表：['进程P3添加的数据', '进程P1添加的数据', '进程P2添加的数据']
```

### 补充：共享变量（Value/Array）

如果需要共享单个变量（如计数器、数字），用`Manager.Value`；共享数组（如整数数组、浮点数数组），用`Manager.Array`：

```python
from multiprocessing import Process, Value

def update_shared_value(shared_num):
    for _ in range(100000):
        with shared_num.get_lock():  # 使用Value自带的锁
            shared_num.value += 1

if __name__ == "__main__":
    shared_num = Value('i', 0)  # 直接创建共享变量

    p1 = Process(target=update_shared_value, args=(shared_num,))
    p2 = Process(target=update_shared_value, args=(shared_num,))

    p1.start()
    p2.start()
    p1.join()
    p2.join()

    print(f"共享变量最终值：{shared_num.value}")  # 预期200000
```

### 核心说明
- Manager支持的共享数据类型：dict（字典）、list（列表）、Value（单个变量）、Array（数组）、Lock（锁）等；
- 优点：支持多个进程共享数据，用法简单，可共享复杂数据结构；
- 缺点：效率比Queue/Pipe低（底层网络通信），多进程同时读写需加锁，避免数据错乱。

# 五、多进程同步：避免资源竞争（Lock/Semaphore）
多进程共享数据（如Manager创建的共享字典、列表）时，多个进程同时读写会导致“数据错乱”（和线程的竞态条件类似）——这时候就需要“同步机制”，保证同一时刻只有一个进程执行读写操作。

核心同步工具：`multiprocessing.Lock`（锁）、`multiprocessing.Semaphore`（信号量），用法和线程的同步工具完全一致。

## 1. Lock（锁，最常用）
保证同一时刻只有一个进程执行“加锁代码块”，解决多进程共享数据的错乱问题。

```python
from multiprocessing import Process, Manager, Lock
import time

# 任务函数：加锁修改共享计数器
def add_count(shared_num, lock, name):
    for _ in range(100000):
        # 加锁：确保同一时刻只有一个进程修改共享变量
        lock.acquire()
        try:
            shared_num.value += 1
        finally:
            # 释放锁：必须放在finally，避免代码报错导致锁未释放
            lock.release()
    print(f"进程{name}执行完成，当前计数器：{shared_num.value}")

if __name__ == "__main__":
    with Manager() as manager:
        # 创建共享计数器和锁
        shared_num = manager.Value('i', 0)
        lock = Lock()

        # 创建2个进程，同时修改共享计数器
        p1 = Process(target=add_count, args=(shared_num, lock, "P1"))
        p2 = Process(target=add_count, args=(shared_num, lock, "P2"))

        start_time = time.time()
        p1.start()
        p2.start()
        p1.join()
        p2.join()
        end_time = time.time()

        print(f"\n最终计数器值：{shared_num.value}")  # 准确输出200000
        print(f"总耗时：{round(end_time - start_time, 2)}秒")
```

**运行结果：**

```
进程P2执行完成，当前计数器：199985
进程P1执行完成，当前计数器：200000

最终计数器值：200000
总耗时：15.04秒
```

### 简化写法（with lock）

和线程锁一样，推荐用`with lock:`简化加锁/释放操作，更安全、简洁：
```python
def add_count(shared_num, lock, name):
    for _ in range(100000):
        with lock:  # 自动加锁、释放锁
            shared_num.value += 1
```

## 2. Semaphore（信号量，控制并发数）
用于控制“同时执行某个操作的进程数”，比如限制最多3个进程同时访问某个资源（如文件、数据库）。

```python
from multiprocessing import Process, Semaphore
import time
import os

# 任务函数：模拟访问共享资源
def access_resource(semaphore, name):
    # 获取信号量（最多3个进程同时获取）
    semaphore.acquire()
    print(f"进程{name}（PID：{os.getpid()}）开始访问共享资源")
    time.sleep(2)  # 模拟访问耗时
    print(f"进程{name}访问完成，释放资源")
    # 释放信号量
    semaphore.release()

if __name__ == "__main__":
    # 创建信号量，允许最多3个进程同时访问
    semaphore = Semaphore(3)

    # 创建5个进程，同时尝试访问资源
    processes = [Process(target=access_resource, args=(semaphore, f"P{i+1}")) for i in range(5)]

    for p in processes:
        p.start()
    for p in processes:
        p.join()

    print("所有进程访问完成")
```

**运行结果：**

```
进程P2（PID：12384）开始访问共享资源
进程P1（PID：11232）开始访问共享资源
进程P5（PID：11952）开始访问共享资源
进程P2访问完成，释放资源
进程P4（PID：15116）开始访问共享资源
进程P1访问完成，释放资源
进程P3（PID：18212）开始访问共享资源
进程P5访问完成，释放资源
进程P4访问完成，释放资源
进程P3访问完成，释放资源
所有进程访问完成
```

# 六、实战：CPU密集型任务批量计算（直观见效率）

我们用“批量计算多个大数字的质因数”（典型CPU密集型任务），对比**单进程、多进程、线程池**的效率差异，直观感受多进程的优势，代码可直接复用。

```python
from multiprocessing import Pool
from concurrent.futures import ThreadPoolExecutor
import time
import os

# 核心任务：计算一个数的所有质因数（CPU密集型）
def get_prime_factors(num):
    """计算num的所有质因数，返回列表"""
    if num < 2:
        return []  # 非法输入直接返回空列表
    factors = []
    # 从2开始遍历，直到num的平方根
    while num % 2 == 0:
        factors.append(2)
        num = num // 2
    # 遍历奇数，寻找质因数
    i = 3
    while i * i <= num:
        while num % i == 0:
            factors.append(i)
            num = num // i
        i += 2
    # 如果剩余num大于2，也是质因数
    if num > 2:
        factors.append(num)
    return factors

# 1. 单进程执行（对比基准）
def single_process_task(nums):
    start_time = time.perf_counter()  # 更高精度的时间测量
    results = []
    for num in nums:
        try:
            results.append(get_prime_factors(num))
        except Exception as e:
            print(f"处理 {num} 时发生错误: {e}")
            results.append([])
    end_time = time.perf_counter()
    print(f"【单进程】耗时：{round(end_time - start_time, 6)}秒")
    return results

# 2. 多进程执行（进程池）
def multi_process_task(nums):
    start_time = time.perf_counter()
    with Pool(processes=os.cpu_count()) as pool:
        results = pool.map(get_prime_factors, nums)
    end_time = time.perf_counter()
    print(f"【多进程】耗时：{round(end_time - start_time, 6)}秒")
    return results

# 3. 线程池执行（对比，看GIL锁影响）
def thread_pool_task(nums):
    start_time = time.perf_counter()
    with ThreadPoolExecutor(max_workers=os.cpu_count()) as executor:
        results = list(executor.map(get_prime_factors, nums))
    end_time = time.perf_counter()
    print(f"【线程池】耗时：{round(end_time - start_time, 6)}秒")
    return results

if __name__ == "__main__":
    # 定义更大规模的任务数据（增加任务复杂度）
    nums = [1234567890123456789] * 100000  # 重复100000次相同任务以放大性能差异

    print(f"当前CPU核心数：{os.cpu_count()}")
    print("=" * 50)

    # 分别执行三种方式，对比效率
    single_process_task(nums)
    thread_pool_task(nums)
    multi_process_task(nums)

    print("=" * 50)
    print("结论：对于大规模CPU密集型任务，多进程效率通常优于单进程和线程池")
```

### 实战结果预期（8核CPU）
```
当前CPU核心数：16
==================================================
【单进程】耗时：8.831497秒
【线程池】耗时：10.355401秒
【多进程】耗时：1.693512秒
==================================================
结论：对于大规模CPU密集型任务，多进程效率通常优于单进程和线程池
```

### 核心价值
- 该实战完美体现多进程的核心优势：突破GIL锁，利用多核CPU并行计算，效率提升6~8倍；
- 代码可直接复用：将`get_prime_factors`替换为自己的CPU密集型任务（如数据计算、模型训练），即可快速实现并发；
- 对比清晰：让新手直观理解“不同场景该用哪种并发方案”，避免误用线程池处理CPU密集型任务。

# 七、多进程vs多线程vs线程池：一张表选型
学完多进程、多线程、线程池后，很多新手会困惑“什么时候用哪个”，这里用一张表彻底分清，覆盖所有常见场景：

| 特性/场景          | 多进程（multiprocessing）    | 多线程（threading）  | 线程池（ThreadPoolExecutor） |
| ------------------ | ---------------------------- | -------------------- | ---------------------------- |
| GIL锁限制          | 无（突破GIL）                | 有                   | 有（本质是多线程）           |
| 核心优势           | 利用多核CPU，并行计算        | 切换快、开销小       | 复用线程、控制并发、简化管理 |
| 核心劣势           | 开销大、切换慢、通信复杂     | 无法利用多核         | 受GIL限制，无法处理CPU密集型 |
| 内存空间           | 独立内存（互不干扰）         | 共享内存             | 共享内存                     |
| 通信难度           | 较高（需Queue/Pipe/Manager） | 较低（共享全局变量） | 较低（共享全局变量）         |
| 适用场景           | CPU密集型（计算、数据处理）  | 简单IO密集型         | 复杂IO密集型（批量任务）     |
| 进程/线程管理      | 复杂（需手动管理或用进程池） | 复杂（需手动管理）   | 简单（自动管理线程）         |
| 推荐优先级（新手） | CPU密集型首选                | 不推荐（优先线程池） | IO密集型首选                 |

### 选型口诀
1.  看任务类型：CPU密集型→多进程；IO密集型→线程池；
2.  看任务复杂度：简单IO任务→线程池；复杂CPU任务→多进程；
3.  看资源限制：内存紧张→线程池（开销小）；多核空闲→多进程（利用多核）。

# 八、新手避坑大全（重点，避免踩雷）
1.  **Windows系统必加`if __name__ == "__main__":`**：否则会导致进程无限递归创建，抛出`RuntimeError`，这是新手最常踩的坑；
2.  **进程数不要超过CPU核心数**：过多进程会导致系统调度开销剧增，反而降低效率，CPU密集型任务建议设为“CPU核心数”；
3.  **多进程不要共享全局变量**：多进程有独立内存空间，全局变量修改后互不同步，需用Manager/Queue/Pipe实现通信；
4.  **共享数据必须加锁**：用Manager共享字典、列表时，多进程同时读写会导致数据错乱，需用Lock保证同步；
5.  **进程开销远大于线程**：不要频繁创建/销毁进程（如循环创建1000个进程），优先用进程池复用进程；
6.  **Pipe仅支持两个进程通信**：多个进程同时操作一个Pipe连接，会导致数据错乱，多进程通信优先用Queue；
7.  **避免进程泄漏**：用Process类创建进程时，需确保调用`join()`等待进程完成，或设置`daemon=True`，避免进程成为“僵尸进程”；
8.  **fork()仅支持Linux/Mac**：Windows系统用fork()会抛出异常，跨平台开发优先用Process类或进程池。

# 九、核心总结
本节课我们掌握了Python多进程的核心知识，完美衔接之前的线程、线程池，核心要点回顾：
1.  **多进程核心价值**：突破Python GIL锁，充分利用多核CPU，解决CPU密集型任务的并发问题；
2.  **进程创建3种方式**：
    - 常用场景：Process类（少量进程）、进程池Pool（批量CPU密集型任务）；
    - 底层场景：fork()（仅Linux/Mac，新手了解即可）；
3.  **多进程通信3种方式**：
    - 首选Queue（多进程批量通信，安全便捷）；
    - 一对一通信：Pipe（高效，双向）；
    - 多进程共享数据：Manager（共享字典/列表/变量）；
4.  **同步机制**：共享数据需用Lock保证同步，控制并发数用Semaphore；
5.  **选型原则**：CPU密集型→多进程；IO密集型→线程池；简单并发→多线程；
6.  **核心避坑**：Windows系统加`if __name__ == "__main__":`、进程数不超CPU核心数、共享数据加锁。

多进程、多线程、线程池，共同构成了Python并发编程的“三驾马车”——掌握这三者的选型和用法，就能高效应对企业开发中所有的并发场景（IO密集型、CPU密集型）。下一节我们将学习多进程与多线程的结合使用（混合并发），应对更复杂的实际场景。

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/160363879>
