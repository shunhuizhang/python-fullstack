

# Python全栈入门到实战【进阶篇 14】Python协程编程：async/await吃透超高并发IO密集型任务
上一篇我们吃透了**多进程+多线程的混合并发**（解决CPU+IO混合密集型任务），覆盖了开发中90%的常规并发场景，但面对**超高并发IO密集型任务**：比如「百万级爬虫批量请求」「高并发接口批量调用」「实时消息队列消费」，纯线程/混合并发会遇到**调度开销大、并发数受限、资源利用率低**的问题——而**协程（Coroutine）** 正是解决这类问题的终极方案。

协程作为Python并发编程的轻量级进阶模型，基于`async/await`原生语法实现，由程序自身控制调度，创建/切换开销远低于线程，单进程可轻松支撑10万+级并发，是处理超高并发IO的最优解。本节课作为并发编程体系的核心收尾，专门讲透协程编程：从「为什么用」到「核心概念」，再到「基础语法+企业级实战」，帮你吃透超高并发IO的处理方案，同时完美衔接前序的进程、线程知识点，构建完整的Python并发编程体系。
本节核心学习内容：
- 协程的核心价值：解决进程/线程处理超高并发IO的效率痛点
- 4个核心概念：事件循环、协程函数、await、Task/Future（吃透协程底层）
- 核心设计原则：异步非阻塞，让协程在IO等待时无缝切换
- 3个实战环节：基础语法入门、异步接口调用、异步爬虫（代码可直接复用）
- 性能调优技巧：协程任务批量提交、并发数控制（避免服务压垮）
- 协程vs多进程vs多线程vs混合并发：一张表快速选型（覆盖所有并发场景）
- 新手必避的8个坑：同步阻塞、事件循环关闭、任务未封装等
- 进阶拓展：协程与多进程的结合（协程处理IO+进程利用多核）

# 一、为什么需要协程？（进程/线程处理超高并发IO的致命痛点）
在讲协程之前，我们先明确**什么是超高并发IO密集型任务**：单批次需要处理**万级/十万级甚至百万级**的IO操作（网络请求、接口调用、消息消费），且每个IO操作的计算量极小，核心耗时集中在IO等待阶段。
对于这类任务，纯进程、纯线程甚至混合并发，都会暴露致命问题，而协程则能完美解决。

### 1.1 纯多进程处理超高并发IO：资源直接耗尽
多进程的核心优势是利用多核处理CPU任务，但面对超高并发IO，会出现：
- 进程创建/销毁开销极大，十万级并发需要创建十万个进程，系统内存、CPU调度直接耗尽；
- 每个进程占用独立内存空间，海量进程会导致系统OOM（内存溢出），直接崩溃；
- IO等待阶段进程全程占用CPU核心，多核资源被严重闲置，资源利用率趋近于0。

**直观例子**：用进程处理10万次接口请求（每个请求IO等待0.5秒，计算耗时可忽略），创建10万进程会直接让服务器内存占满，系统卡死。

### 1.2 纯线程池处理超高并发IO：调度开销成瓶颈
线程池的开销远低于进程，是处理常规IO并发的优选，但面对**万级以上超高并发**，会出现：
- 线程的创建/切换由操作系统内核调度，万级线程的调度开销会急剧增加，导致系统卡顿；
- 线程占用的系统资源（栈空间、内核态资源）随数量线性增长，一般单进程线程数超过2000就会出现明显性能下降，无法支撑十万级并发；
- 受GIL锁间接影响，线程调度的内核态开销会进一步放大，超高并发下效率大幅降低。

**直观例子**：用线程池处理10万次接口请求，设置线程数2000，总耗时会达到数十秒，且服务器CPU利用率会因线程调度飙升至100%。

### 1.3 混合并发处理超高并发IO：大材小用，效率低下
混合并发的核心是「进程扛CPU+线程扛IO」，但面对无计算的超高并发IO，会出现：
- 进程的多核优势完全无法发挥，创建多个进程只会增加资源占用，反而降低效率；
- 每个进程内的线程池仍受线程数限制，整体并发数还是被线程资源约束，无法突破万级；
- 多进程+多线程的多层调度，会让超高并发下的调度开销叠加，资源利用率远低于纯协程。

### 1.4 协程的核心优势：轻量级调度，超高并发无压力
协程（也叫**微线程**）是**由Python程序自身控制的用户态轻量级线程**，无需操作系统内核调度，核心遵循**「异步非阻塞」**原则，针对超高并发IO的优势堪称碾压级：
1. **开销极致低**：协程的创建/切换开销仅为线程的1/100~1/1000，单进程可轻松创建10万+协程，内存占用仅为同等数量线程的几十分之一；
2. **调度更高效**：协程的调度由Python的**事件循环**完成，全程在用户态执行，无需内核态/用户态切换，调度耗时可忽略；
3. **IO等待无缝切换**：当某个协程遇到IO等待时，`await`会主动挂起该协程，事件循环会立即调度其他协程执行，**IO等待阶段无任何资源闲置**，资源利用率接近100%；
4. **语法原生支持**：Python3.5+通过`async/await`关键字原生支持协程，搭配内置`asyncio`模块，无需安装第三方库，开发成本低；
5. **可灵活拓展**：协程可与多进程结合，利用多进程突破单进程CPU核心限制，同时利用协程处理超高并发IO，兼顾多核和超高并发。

**直观例子**：用协程处理10万次接口请求（每个请求IO等待0.5秒），单进程单线程即可实现，总耗时仅略大于0.5秒，服务器CPU和内存利用率均保持在合理范围。

# 二、协程的核心概念与设计原则
协程的开发核心是围绕**事件循环**展开，吃透4个核心概念和3个设计原则，就能避免90%的协程开发坑，这也是后续实战的基础。

### 2.1 4个核心概念
协程的所有操作都基于Python内置的`asyncio`模块，核心概念层层递进，理解后就能轻松看懂协程代码：
1. **事件循环（Event Loop）**：协程的**核心调度器**，负责管理所有协程任务的创建、执行、挂起、切换、销毁，是协程运行的基础。可以理解为「协程的管家」，所有协程都必须在事件循环中运行；
2. **协程函数（Coroutine Function）**：用`async def`定义的函数，是协程的基础单元。**调用协程函数不会立即执行**，只会返回一个**协程对象（Coroutine Object）**，需要交给事件循环调度才会执行；
3. **await**：协程的**挂起关键字**，只能用在协程函数内部。当执行到`await 可等待对象`时，协程会主动挂起，释放事件循环的控制权，直到可等待对象执行完成，事件循环才会重新调度该协程继续执行。**可等待对象**包括：协程对象、Task对象、Future对象、异步IO操作（如`asyncio.sleep`）；
4. **Task/Future对象**：**协程的可调度单元**，事件循环实际调度的是Task/Future对象，而非直接的协程对象。
   - **Task对象**：由协程对象封装而来，是最常用的可调度单元，通过`asyncio.create_task()`或`loop.create_task()`创建，支持任务取消、结果获取、状态查看；
   - **Future对象**：是Task对象的父类，更偏向底层，一般用于封装异步操作的结果，开发中直接用Task对象即可。

**核心关系**：协程函数 → 调用返回协程对象 → 封装为Task对象 → 交给事件循环调度执行 → 遇到await挂起并切换其他Task。

### 2.2 3个核心设计原则（必遵守，否则必踩坑）
协程的高效性完全依赖于**异步非阻塞**的执行模式，违背以下原则，协程会退化为同步执行，甚至效率不如线程：
1. **异步非阻塞原则**：所有IO操作必须使用**异步版本**，避免使用同步IO操作（如`requests.get`、`time.sleep`、同步文件读写）。同步IO操作会阻塞整个事件循环，导致所有协程无法切换，变成串行执行；
2. **await必加原则**：所有可等待对象（协程对象、Task对象、异步IO操作）在执行时，**必须加await关键字**。不加await的可等待对象不会被执行，也不会挂起协程，只会创建一个未执行的对象；
3. **单进程单线程原则**：纯协程编程基于**单进程单线程**运行，利用事件循环实现协程的并发切换。无需手动创建多线程/多进程，否则会破坏事件循环的调度，增加不必要的开销（超高并发IO场景下，单进程协程的效率已足够）。

### 2.3 协程的运行流程
以「两个协程异步执行IO操作」为例，直观理解协程的完整运行流程：
1. 定义两个协程函数`async def func1()`和`async def func2()`，内部包含`await asyncio.sleep(1)`（模拟IO等待）；
2. 主协程函数`async def main()`中，通过`asyncio.create_task()`将两个协程对象封装为Task1和Task2；
3. 调用`asyncio.run(main())`，该方法会**自动创建事件循环**，并将main协程交给事件循环调度；
4. 事件循环执行main协程，依次创建Task1和Task2，并将它们加入调度队列；
5. 事件循环调度执行Task1，执行到`await asyncio.sleep(1)`时，Task1被挂起，事件循环获取控制权；
6. 事件循环立即调度执行Task2，执行到`await asyncio.sleep(1)`时，Task2被挂起，事件循环获取控制权；
7. 1秒后，IO等待完成，事件循环依次唤醒Task1和Task2，执行剩余代码；
8. 所有Task执行完成，main协程执行结束，事件循环自动关闭。

**核心关键点**：两个协程的IO等待阶段是**并行的**，总耗时仅1秒，而非串行的2秒，这就是协程的异步并发核心。

# 三、协程的基础实战（从语法到并发，附完整可运行代码）
本节从最基础的协程语法开始，逐步实现协程的单任务执行、多任务并发执行，所有代码均兼容**Python3.7+**（`asyncio.run()`是3.7+新增特性），跨平台（Windows/Linux/Mac），带详细注释，运行结果可复现。

### 环境准备
无需安装第三方库，直接使用Python内置的`asyncio`模块，核心注意点：
- Windows/Linux/Mac运行无差异，无需加`if __name__ == "__main__"`（协程无进程创建的递归问题）；
- 避免在协程中使用同步IO操作，优先使用`asyncio`提供的异步方法（如`asyncio.sleep()`替代`time.sleep()`）。

## 3.1 实战1：第一个协程程序（单任务执行）
**核心目标**：掌握协程函数的定义、协程对象的创建、事件循环的运行，理解`async/await`的基础用法。
```python
import asyncio  # 导入内置协程模块

# --------------- 步骤1：定义协程函数（async def 标识） ---------------
async def first_coroutine(name):
    """第一个协程函数：模拟简单的IO操作"""
    print(f"【协程{name}】开始执行，准备进入IO等待")
    # 模拟IO等待（必须用asyncio.sleep，不能用time.sleep，否则阻塞事件循环）
    await asyncio.sleep(1)  # await 挂起协程，释放事件循环控制权
    print(f"【协程{name}】IO等待完成，执行结束")
    return f"协程{name}执行成功"  # 协程函数可返回结果，后续可获取

# --------------- 步骤2：定义主协程函数（协程的入口，管理所有子协程） ---------------
async def main():
    """主协程函数：所有子协程都在该函数内执行，是协程的统一入口"""
    # 调用协程函数，返回协程对象（不会立即执行）
    coro_obj = first_coroutine("1")
    # 用await执行协程对象，获取返回结果
    result = await coro_obj
    print(f"主协程获取子协程结果：{result}")

# --------------- 步骤3：运行主协程，启动事件循环 ---------------
if __name__ == "__main__":
    # asyncio.run()：Python3.7+新增，自动创建事件循环、运行协程、关闭事件循环
    # 是运行协程的最简方式，开发中优先使用
    asyncio.run(main())
```

### 运行结果
```
【协程1】开始执行，准备进入IO等待
【协程1】IO等待完成，执行结束
主协程获取子协程结果：协程1执行成功
```

### 核心说明（必看，理解基础）
1. 协程函数必须用`async def`定义，普通函数不能使用`await`，也不能被封装为Task对象；
2. 调用协程函数`first_coroutine("1")`仅返回协程对象，**不会执行函数内部代码**，这是和普通函数的核心区别；
3. `await coro_obj`会让事件循环调度执行该协程对象，直到执行完成并返回结果，期间遇到`await`会自动挂起；
4. `asyncio.run(main())`是协程的**标准运行方式**，自动完成「创建事件循环→运行协程→关闭事件循环」，无需手动操作，开发中优先使用；
5. 加`if __name__ == "__main__"`是为了代码规范，协程运行本身无需该语句，跨平台可直接运行。

## 3.2 实战2：多协程任务并发执行（核心！）
**核心目标**：掌握协程的**异步并发核心**，学会用`asyncio.create_task()`封装Task对象，实现多协程的并发执行，这是处理超高并发IO的基础。
```python
import asyncio
import time

# --------------- 步骤1：定义协程函数（模拟IO密集型任务） ---------------
async def io_coroutine(task_id):
    """模拟超高并发IO任务：如接口调用、网络请求"""
    start = time.perf_counter()
    # 模拟IO等待0.5秒（每个任务的IO耗时一致）
    await asyncio.sleep(0.5)
    end = time.perf_counter()
    print(f"【任务{task_id}】执行完成，IO耗时：{round(end - start, 3)}秒")
    return f"任务{task_id}成功"

# --------------- 步骤2：主协程函数（批量创建Task，实现并发） ---------------
async def main():
    # 目标：并发执行100个协程任务（模拟千级IO并发）
    task_num = 100
    print(f"开始并发执行{task_num}个IO协程任务")
    print("=" * 60)
    start_total = time.perf_counter()

    # 方式：批量创建Task对象（事件循环实际调度的是Task）
    tasks = []
    for i in range(1, task_num + 1):
        # 1. 创建协程对象
        coro = io_coroutine(i)
        # 2. 封装为Task对象，加入任务列表（封装后立即被事件循环调度）
        task = asyncio.create_task(coro)
        tasks.append(task)

    # 等待所有Task执行完成，批量获取结果（asyncio.gather()：批量等待可等待对象）
    # *tasks：解包任务列表，传入gather
    results = await asyncio.gather(*tasks)

    # 统计总耗时
    end_total = time.perf_counter()
    print("=" * 60)
    print(f"{task_num}个协程任务全部执行完成")
    print(f"总耗时：{round(end_total - start_total, 3)}秒")
    print(f"成功完成任务数：{len([res for res in results if '成功' in res])}")

# --------------- 步骤3：启动事件循环 ---------------
if __name__ == "__main__":
    asyncio.run(main())
```

### 运行结果（Windows/Linux/Mac一致）
```
开始并发执行100个IO协程任务
============================================================
【任务1】执行完成，IO耗时：0.501秒
【任务2】执行完成，IO耗时：0.501秒
【任务3】执行完成，IO耗时：0.501秒
...（省略中间94个任务）
【任务99】执行完成，IO耗时：0.502秒
【任务100】执行完成，IO耗时：0.502秒
============================================================
100个协程任务全部执行完成
总耗时：0.503秒
成功完成任务数：100
```

### 核心说明（协程并发的关键，必吃透）
1. **`asyncio.create_task(coro)`**：将协程对象封装为Task对象，**封装后立即被事件循环调度**，这是实现并发的核心。如果仅创建协程对象而不封装为Task，不会被事件循环调度；
2. **`asyncio.gather(*tasks)`**：批量等待多个可等待对象（Task/协程对象）执行完成，返回所有结果的列表，结果顺序与任务创建顺序一致，这是批量处理协程任务的首选方式；
3. **并发核心**：100个协程任务的总耗时仅略大于单个任务的IO耗时（0.5秒），而非串行的50秒，这就是协程的**异步并发优势**，超高并发下该优势会更加明显；
4. **任务数无压力**：将`task_num`改为1000、10000，甚至100000，代码无需任何修改，总耗时仍仅略大于0.5秒，且服务器资源占用极低，这是线程/进程无法实现的；
5. **注意并发数控制**：实际开发中，需根据目标服务的性能控制协程并发数（如接口请求并发数设为1000），避免一次性发起海量请求压垮目标服务。

# 四、企业级实战：协程处理经典超高并发IO场景
协程的核心应用场景是**超高并发IO密集型任务**，本节实现两个企业级经典场景：**异步接口调用**和**异步爬虫**，均使用主流的异步第三方库，代码可直接复用，稍作修改即可对接实际业务。

### 前置准备：安装异步第三方库
协程开发中，需要使用**异步版本**的第三方库，替代同步库（如`requests`→`aiohttp`，`pymysql`→`aiomysql`），本次实战安装核心异步库：
```bash
# aiohttp：异步HTTP请求库，用于异步接口调用、异步爬虫（替代requests）
pip install aiohttp==3.9.1  # 指定版本，避免兼容问题
```

## 4.1 实战1：异步批量调用接口（超高并发接口请求）
**场景说明**：批量调用第三方接口（如数据查询、短信发送、消息推送），单批次调用1000次，属于典型的超高并发IO任务，用协程实现异步调用，对比同步调用的效率差异。
**实战代码**（可直接运行，模拟接口调用，替换url即可对接实际接口）：
```python
import asyncio
import time
import aiohttp  # 异步HTTP请求库

# --------------- 配置项 ---------------
TARGET_URL = "https://httpbin.org/get"  # 模拟第三方接口（可替换为实际接口）
CONCURRENT_NUM = 1000  # 协程并发数（根据目标服务性能调整）
HEADERS = {"User-Agent": "Python-asyncio/3.10"}

# --------------- 异步接口调用协程函数 ---------------
async def async_call_api(session, task_id):
    """
    异步调用接口：使用aiohttp的ClientSession发起异步请求
    :param session: aiohttp的会话对象（复用会话，减少连接创建开销）
    :param task_id: 任务ID
    :return: 接口响应结果
    """
    try:
        # 发起异步GET请求（await 挂起，等待响应）
        async with session.get(url=TARGET_URL, headers=HEADERS, timeout=aiohttp.ClientTimeout(total=5)) as response:
            # 解析响应结果（json格式）
            result = await response.json()
            print(f"【任务{task_id}】接口调用成功，状态码：{response.status}")
            return f"任务{task_id}成功，状态码：{response.status}"
    except Exception as e:
        print(f"【任务{task_id}】接口调用失败，错误：{str(e)}")
        return f"任务{task_id}失败，错误：{str(e)}"

# --------------- 主协程函数：批量创建任务，实现异步并发 ---------------
async def main():
    print(f"开始异步批量调用接口，并发数：{CONCURRENT_NUM}")
    print("=" * 80)
    start_time = time.perf_counter()

    # 关键：创建aiohttp的ClientSession对象，复用会话（减少TCP连接创建开销）
    # 避免为每个任务创建一个Session，否则会导致连接数过多
    async with aiohttp.ClientSession() as session:
        # 批量创建Task对象
        tasks = [asyncio.create_task(async_call_api(session, i)) for i in range(1, CONCURRENT_NUM + 1)]
        # 等待所有任务完成，获取结果
        results = await asyncio.gather(*tasks)

    # 统计结果
    end_time = time.perf_counter()
    success_num = len([res for res in results if "成功" in res])
    fail_num = CONCURRENT_NUM - success_num
    print("=" * 80)
    print(f"接口调用完成 | 总任务数：{CONCURRENT_NUM} | 成功：{success_num} | 失败：{fail_num}")
    print(f"协程异步调用总耗时：{round(end_time - start_time, 3)}秒")
    print(f"平均每个任务耗时：{round((end_time - start_time)/CONCURRENT_NUM, 6)}秒")

# --------------- 启动事件循环 ---------------
if __name__ == "__main__":
    # Windows下解决aiohttp的事件循环策略问题（可选，3.8+已修复）
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
    asyncio.run(main())
```

### 运行结果（1000次接口调用）
```
开始异步批量调用接口，并发数：1000
================================================================================
【任务1】接口调用成功，状态码：200
【任务2】接口调用成功，状态码：200
...（省略中间996个任务）
【任务999】接口调用成功，状态码：200
【任务1000】接口调用成功，状态码：200
================================================================================
接口调用完成 | 总任务数：1000 | 成功：1000 | 失败：0
协程异步调用总耗时：2.356秒
平均每个任务耗时：0.002356秒
```

### 效率对比（同步vs异步）
- **协程异步调用**：1000次接口调用，总耗时≈2.3秒，平均每个任务≈0.0023秒；
- **requests同步调用**：1000次接口调用，总耗时≈200秒+，平均每个任务≈0.2秒；
- **效率提升**：约87倍，且并发数越高，协程的效率优势越明显。

### 核心说明（企业级开发必注意）
1. **复用aiohttp.ClientSession**：必须为所有协程任务复用一个Session对象，避免为每个任务创建独立Session，否则会导致TCP连接数过多，被目标服务限制，同时增加连接创建开销；
2. **设置超时时间**：使用`aiohttp.ClientTimeout()`设置接口请求超时时间，避免某个任务的IO等待过长，阻塞整个事件循环；
3. **异常捕获**：必须为异步接口调用添加异常捕获，处理网络异常、接口超时、服务错误等情况，避免单个任务失败导致整个协程程序崩溃；
4. **并发数控制**：根据目标服务的QPS限制调整`CONCURRENT_NUM`，比如目标服务QPS为500，则并发数设为500，避免压垮目标服务；
5. **Windows兼容**：Python3.8以下的Windows系统，需添加`asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())`，解决aiohttp的事件循环策略问题，3.8+已修复。

## 4.2 实战2：异步爬虫（批量爬取网页内容）
**场景说明**：批量爬取网页内容（如新闻资讯、商品信息、数据采集），单批次爬取500个网页，属于典型的超高并发IO爬虫任务，用协程实现异步爬取，效率远高于同步爬虫（如requests+BeautifulSoup）。
**实战代码**（可直接运行，模拟爬虫，替换url和解析逻辑即可对接实际爬虫业务）：
```python
import asyncio
import time
import aiohttp
from lxml import etree  # 解析网页内容（轻量级，替代BeautifulSoup）

# --------------- 配置项 ---------------
BASE_URL = "https://httpbin.org/html"  # 模拟爬取的网页（返回HTML内容）
CRAWL_NUM = 500  # 爬取网页数量
CONCURRENT_NUM = 500  # 协程并发数
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
}

# --------------- 异步爬取网页协程函数 ---------------
async def async_crawl(session, task_id):
    """
    异步爬取网页：获取HTML内容并解析
    :param session: aiohttp的ClientSession对象
    :param task_id: 任务ID
    :return: 爬取结果
    """
    try:
        # 发起异步GET请求，获取HTML内容
        async with session.get(url=BASE_URL, headers=HEADERS, timeout=aiohttp.ClientTimeout(total=10)) as response:
            if response.status == 200:
                # 读取HTML内容（await 挂起，等待读取完成）
                html = await response.text(encoding="utf-8")
                # 解析HTML内容（lxml同步解析，因解析耗时极短，不影响协程并发）
                tree = etree.HTML(html)
                title = tree.xpath("//h1/text()")[0]  # 提取网页标题
                print(f"【爬虫任务{task_id}】爬取成功，标题：{title}")
                return f"爬虫任务{task_id}成功，标题：{title}"
            else:
                print(f"【爬虫任务{task_id}】爬取失败，状态码：{response.status}")
                return f"爬虫任务{task_id}失败，状态码：{response.status}"
    except Exception as e:
        print(f"【爬虫任务{task_id}】爬取失败，错误：{str(e)[:50]}")
        return f"爬虫任务{task_id}失败，错误：{str(e)[:50]}"

# --------------- 主协程函数 ---------------
async def main():
    print(f"开始异步爬取网页，爬取数量：{CRAWL_NUM}，并发数：{CONCURRENT_NUM}")
    print("=" * 80)
    start_time = time.perf_counter()

    # 复用aiohttp.ClientSession
    async with aiohttp.ClientSession() as session:
        # 批量创建Task
        tasks = [asyncio.create_task(async_crawl(session, i)) for i in range(1, CRAWL_NUM + 1)]
        # 等待所有任务完成
        results = await asyncio.gather(*tasks)

    # 统计结果
    end_time = time.perf_counter()
    success_num = len([res for res in results if "成功" in res])
    fail_num = CRAWL_NUM - success_num
    print("=" * 80)
    print(f"爬虫完成 | 总爬取数：{CRAWL_NUM} | 成功：{success_num} | 失败：{fail_num}")
    print(f"协程异步爬虫总耗时：{round(end_time - start_time, 3)}秒")

# --------------- 启动事件循环 ---------------
if __name__ == "__main__":
    # Windows兼容
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
    asyncio.run(main())
```

### 运行结果（500个网页爬取）
```
开始异步爬取网页，爬取数量：500，并发数：500
================================================================================
【爬虫任务1】爬取成功，标题：Herman Melville - Moby-Dick
【爬虫任务2】爬取成功，标题：Herman Melville - Moby-Dick
...（省略中间496个任务）
【爬虫任务499】爬取成功，标题：Herman Melville - Moby-Dick
【爬虫任务500】爬取成功，标题：Herman Melville - Moby-Dick
================================================================================
爬虫完成 | 总爬取数：500 | 成功：500 | 失败：0
协程异步爬虫总耗时：1.892秒
```

### 核心说明（异步爬虫开发必注意）
1. **解析逻辑可同步**：网页解析（如lxml、BeautifulSoup）属于**CPU耗时极短**的操作，即使使用同步解析库，也不会影响协程的并发效率，无需使用异步解析库；
2. **控制爬取速度**：实际爬虫开发中，需添加**延时**（如`await asyncio.sleep(0.1)`）或**限制并发数**，避免爬取速度过快被目标网站封IP；
3. **使用代理IP池**：超高并发爬虫需搭配代理IP池，通过aiohttp设置代理，避免单IP被封；
4. **数据持久化**：解析后的结果可通过**异步数据库库**（如aiomysql、aioredis）实现异步写入，避免同步数据库操作阻塞事件循环。

# 五、协程进阶：协程与多进程的结合（协程IO+进程多核）
纯协程基于**单进程单线程**运行，无法利用多核CPU，而如果协程任务中包含**少量CPU密集型操作**（如网页内容解析、数据处理），单进程的CPU核心会成为瓶颈。
**协程+多进程**的混合方案，完美解决该问题：**用多进程利用多核CPU，每个进程内创建协程处理超高并发IO**，兼顾多核CPU和超高并发IO，是处理「少量CPU+大量超高并发IO」混合任务的最优解。

### 核心设计思路
1. **外层多进程**：创建与CPU核心数相同的进程，利用多核CPU处理少量CPU密集型操作，突破单进程CPU限制；
2. **内层协程**：每个进程内创建独立的事件循环和协程任务，处理超高并发IO操作，发挥协程的轻量级优势；
3. **任务分片**：将总IO任务分片，每个进程处理一部分分片任务，避免进程间的任务竞争。

### 实战代码（协程+多进程，可直接运行）
```python
import asyncio
import time
import aiohttp
from multiprocessing import Pool
import os

# --------------- 配置项 ---------------
TARGET_URL = "https://httpbin.org/get"
TOTAL_TASK_NUM = 4000  # 总IO任务数
CPU_NUM = os.cpu_count()  # CPU核心数，进程数=CPU核心数
TASK_PER_PROCESS = TOTAL_TASK_NUM // CPU_NUM  # 每个进程处理的任务数

# --------------- 异步IO协程函数 ---------------
async def async_io_task(session, task_id):
    try:
        async with session.get(url=TARGET_URL, timeout=aiohttp.ClientTimeout(total=5)) as response:
            await response.json()
            return f"进程{os.getpid()}：任务{task_id}成功"
    except Exception as e:
        return f"进程{os.getpid()}：任务{task_id}失败，错误：{str(e)[:30]}"

# --------------- 每个进程的核心函数：启动协程 ---------------
def process_core(process_id):
    """每个进程执行的函数：创建事件循环，运行协程"""
    # 计算当前进程的任务ID范围
    start_task_id = process_id * TASK_PER_PROCESS + 1
    end_task_id = (process_id + 1) * TASK_PER_PROCESS
    print(f"进程{os.getpid()}（ID：{process_id}）开始处理任务：{start_task_id}-{end_task_id}")

    # 定义进程内的主协程
    async def main_coro():
        async with aiohttp.ClientSession() as session:
            # 批量创建协程Task
            tasks = [asyncio.create_task(async_io_task(session, i)) for i in range(start_task_id, end_task_id + 1)]
            results = await asyncio.gather(*tasks)
            return results

    # 运行协程，获取结果
    # Windows下设置事件循环策略
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
    results = asyncio.run(main_coro())
    return results

# --------------- 主程序：启动多进程 ---------------
if __name__ == "__main__":
    print(f"开始执行协程+多进程混合方案 | 总任务数：{TOTAL_TASK_NUM} | 进程数：{CPU_NUM} | 每个进程任务数：{TASK_PER_PROCESS}")
    print("=" * 80)
    start_time = time.perf_counter()

    # 创建进程池，进程数=CPU核心数
    with Pool(processes=CPU_NUM) as pool:
        # 每个进程分配一个进程ID，批量提交任务
        all_results = pool.map(process_core, range(CPU_NUM))

    # 统计结果
    end_time = time.perf_counter()
    # 展平结果列表
    flat_results = [res for sublist in all_results for res in sublist]
    success_num = len([res for res in flat_results if "成功" in res])
    fail_num = TOTAL_TASK_NUM - success_num

    print("=" * 80)
    print(f"所有任务执行完成 | 总任务数：{TOTAL_TASK_NUM} | 成功：{success_num} | 失败：{fail_num}")
    print(f"协程+多进程总耗时：{round(end_time - start_time, 3)}秒")
```

### 运行结果（8核CPU，4000次IO任务）
```
开始执行协程+多进程混合方案 | 总任务数：4000 | 进程数：8 | 每个进程任务数：500
================================================================================
进程1234（ID：0）开始处理任务：1-500
进程1235（ID：1）开始处理任务：501-1000
进程1236（ID：2）开始处理任务：1001-1500
进程1237（ID：3）开始处理任务：1501-2000
进程1238（ID：4）开始处理任务：2001-2500
进程1239（ID：5）开始处理任务：2501-3000
进程1240（ID：6）开始处理任务：3001-3500
进程1241（ID：7）开始处理任务：3501-4000
================================================================================
所有任务执行完成 | 总任务数：4000 | 成功：4000 | 失败：0
协程+多进程总耗时：2.568秒
```

### 核心说明
1. **进程数=CPU核心数**：严格遵循多进程的配置原则，避免进程数过多导致调度开销增加；
2. **任务分片均匀**：将总IO任务均匀分配给每个进程，避免某个进程任务过多，成为性能瓶颈；
3. **进程内独立协程**：每个进程内创建独立的事件循环和协程任务，进程间无资源共享，避免数据错乱；
4. **适用场景**：「少量CPU密集型操作+大量超高并发IO操作」的混合任务，如「网页爬取+解析」「接口调用+数据处理」；
5. **不适用场景**：纯超高并发IO任务（无任何计算），直接用单进程协程即可，无需多进程，避免不必要的开销。

# 六、协程vs多进程vs多线程vs混合并发：一张表快速选型
结合前序的多进程、多线程、混合并发，以及本次的协程，我们用一张表总结**四种并发模型**的选型依据，覆盖**纯CPU、常规IO、混合IO、超高并发IO**所有场景，新手可直接按表选型，无需再纠结。

| 并发模型      | 核心优势                                     | 核心劣势                                          | 适用场景                                               | 核心配置建议                       |
| ------------- | -------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------ | ---------------------------------- |
| 纯多进程      | 突破GIL，多核并行计算，CPU利用率100%         | 创建/切换开销大，IO处理效率低，并发数受限         | 纯CPU密集型任务（数据计算、大文件解析、循环处理）      | 进程数=CPU核心数                   |
| 纯线程池      | 开销低于进程，低并发IO处理效率高，开发简单   | 受GIL限制，超高并发调度开销大，并发数受限（千级） | 常规IO密集型任务（少量接口调用、小批量文件读写）       | 线程数=CPU核心数*(1+T_io/T_cpu)    |
| 多进程+多线程 | 兼顾多核CPU和常规IO并发，资源利用率高        | 设计稍复杂，超高并发IO仍受线程数限制              | CPU+IO混合密集型任务（90%的常规业务场景）              | 进程数=CPU核心数，线程数=4~8/进程  |
| 纯协程        | 开销极致低，超高并发IO效率碾压级，资源占用低 | 单进程单线程，无法利用多核CPU，不支持同步IO       | 纯超高并发IO密集型任务（万级/十万级接口/爬虫/消息）    | 协程数=根据目标服务性能调整        |
| 协程+多进程   | 兼顾多核CPU和超高并发IO，混合任务效率最优    | 设计稍复杂，需任务分片                            | 少量CPU+大量超高并发IO混合任务（爬虫+解析、接口+处理） | 进程数=CPU核心数，协程数=万级/进程 |

### 选型口诀（新手记牢，覆盖所有场景）
1. **纯计算，用进程**：多核并行，把CPU性能拉满；
2. **常规IO，用线程**：低开销高并发，拒绝资源浪费；
3. **混合任务，进程+线程**：进程扛计算，线程扛常规IO；
4. **超高IO，用协程**：轻量级调度，十万级并发无压力；
5. **计算+超高IO，协程+进程**：多核加轻量，效率最大化。

# 七、新手必避的8个协程坑（重点，避免踩雷）
协程的开发逻辑与同步编程、进程/线程编程差异较大，新手容易因思维固化踩坑，以下是最常踩的8个坑，附详细避坑方案，看完直接绕开99%的协程开发问题。

### 坑1：在协程中使用同步IO操作（最常见）
**问题**：在协程中使用`time.sleep()`、`requests.get()`、同步文件读写等**同步IO操作**，会阻塞整个事件循环，导致所有协程无法切换，退化为串行执行，效率远低于线程；
**避坑**：所有IO操作必须使用**异步版本**，替换方案如下：
- `time.sleep(t)` → `asyncio.sleep(t)`
- `requests` → `aiohttp`
- 同步文件读写 → `aiofiles`
- `pymysql` → `aiomysql`
- `redis` → `aioredis`

### 坑2：调用协程函数不加await，也不封装为Task
**问题**：直接调用协程函数`func()`仅返回协程对象，不加`await`也不封装为`asyncio.create_task(func())`，会导致协程函数**永远不会执行**，且无任何报错；
**避坑**：协程对象的两种正确执行方式：
1. 直接执行：`await 协程对象`（适用于单任务）；
2. 并发执行：`asyncio.create_task(协程对象)`（适用于多任务，核心）。

### 坑3：在普通函数中使用await关键字
**问题**：`await`关键字**只能用在协程函数（async def定义）** 内部，在普通函数（def定义）中使用会直接抛出`SyntaxError`语法错误；
**避坑**：
1. 若函数内需要使用`await`，则将函数改为协程函数（`async def`）；
2. 若普通函数需要调用协程函数，需通过`asyncio.run()`或事件循环调度执行。

### 坑4：多次创建/关闭事件循环，或手动管理事件循环
**问题**：新手手动创建事件循环`loop = asyncio.get_event_loop()`，又手动关闭`loop.close()`，容易出现**事件循环已关闭**「事件循环未运行」等错误，且跨平台兼容性差；
**避坑**：**优先使用`asyncio.run(coro)`**（Python3.7+），该方法会自动完成「创建→运行→关闭」事件循环，无需手动操作，是开发中的标准方式。

### 坑5：aiohttp未复用ClientSession，创建过多连接
**问题**：为每个协程任务创建一个`aiohttp.ClientSession`对象，会导致**TCP连接数过多**，被目标服务限制或封IP，同时增加连接创建开销，降低效率；
**避坑**：**所有协程任务复用一个ClientSession对象**，通过`async with aiohttp.ClientSession() as session`管理，自动关闭连接，减少开销。

### 坑6：未设置IO操作超时时间，导致协程挂起无限期
**问题**：异步IO操作（如aiohttp请求、asyncio.sleep）未设置超时时间，若目标服务无响应，协程会一直挂起，占用事件循环资源，导致其他任务无法执行；
**避坑**：所有异步IO操作都设置**超时时间**：
- aiohttp：`aiohttp.ClientTimeout(total=5)`（5秒超时）；
- 通用：`asyncio.wait_for(协程对象, timeout=5)`（封装协程，设置超时）。

### 坑7：在协程中使用多线程/多进程，未正确处理事件循环
**问题**：新手在协程中手动创建多线程/多进程，会破坏事件循环的调度，导致协程切换异常，甚至程序崩溃；
**避坑**：
1. 纯协程任务无需创建多线程/多进程，单进程单线程即可；
2. 若需要结合多进程，采用**外层多进程+内层协程**的方案，每个进程内独立运行事件循环，进程间无资源共享。

### 坑8：未捕获协程的异常，导致单个任务失败崩溃整个程序
**问题**：协程任务中的异常若未捕获，会导致**整个事件循环崩溃**，所有未执行的协程任务都会终止，且错误信息不明确；
**避坑**：**为所有协程任务添加异常捕获（try/except）**，处理网络异常、超时、服务错误等所有可能的异常，确保单个任务失败不会影响其他任务。

# 八、核心总结
本节课我们掌握了Python**协程编程**的核心内容，这是处理超高并发IO密集型任务的终极方案，也是Python并发编程体系的**最后一块核心拼图**，结合前序的多进程、多线程、混合并发，我们已经覆盖了所有开发中的并发场景。核心要点回顾：
1. **协程的核心价值**：解决进程/线程处理超高并发IO的调度开销大、并发数受限问题，单进程可支撑10万+级IO并发，资源利用率接近100%；
2. **4个核心概念**：事件循环（调度器）、协程函数（async def）、await（挂起关键字）、Task对象（可调度单元），是协程开发的基础；
3. **3个核心设计原则**：异步非阻塞（用异步IO）、await必加（执行可等待对象）、单进程单线程（纯协程），违背则协程失去效率优势；
4. **核心实战用法**：用`asyncio.create_task()`封装Task实现并发，用`asyncio.gather()`批量等待任务，用`aiohttp`实现异步HTTP请求，是企业级开发的核心；
5. **进阶拓展**：协程+多进程的混合方案，兼顾多核CPU和超高并发IO，是处理「少量CPU+大量IO」混合任务的最优解；
6. **选型原则**：纯计算用进程、常规IO用线程、混合任务用进程+线程、超高IO用协程、计算+超高IO用协程+进程；
7. **核心避坑**：禁用同步IO、协程执行必加await/封装Task、复用aiohttp.Session、设置超时时间、捕获所有异常。

至此，我们已经完整掌握了Python并发编程的**四大核心模型**：多进程、多线程池、多进程+多线程、协程（含协程+多进程），覆盖了**纯CPU、常规IO、CPU+IO混合、超高并发IO**所有开发场景，能轻松应对企业开发中的各种并发需求。

**后续拓展方向**：
- 异步Web框架：FastAPI（基于协程），比Flask/Django效率更高，支持高并发接口开发；
- 异步消息队列：aiokafka、aioredis，实现异步消息消费，支撑高并发消息处理；
- 异步爬虫框架：Scrapy_Playwright、aiohttp爬虫框架，实现分布式异步爬虫；
- 协程性能监控：asyncio的任务状态监控、耗时统计，实现协程程序的性能调优。

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、并发编程（进程/线程/协程）、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/160472346>
