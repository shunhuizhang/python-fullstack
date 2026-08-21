

# Python全栈入门到实战【进阶篇 9】四大常用设计模式：工厂/单例/装饰器/观察者
在前面的章节中，我们完成了面向对象核心特性、枚举类等高级语法的学习。而在真实企业开发中，**设计模式**是写出**可复用、可扩展、低耦合**代码的必备思想，它是前人总结的、针对特定场景的最佳实践。

本节课我们不讲复杂业务场景，完全使用**通用极简示例**，一次性讲透 Python 最常用的 **4 大设计模式**：工厂模式、单例模式、装饰器模式、观察者模式。不讲花里胡哨的理论，只讲：**是什么→解决什么问题→怎么写→用在哪**，新手也能直接看懂、直接复用。

本节核心学习内容：
- 工厂模式：统一创建对象，告别分散、混乱的实例化代码
- 单例模式：保证一个类永远只有一个实例，全局共享、数据一致
- 装饰器模式：不修改原代码，动态给函数/方法增加功能
- 观察者模式：一对多通知，状态一变，所有依赖自动更新
- 每种模式的**最简代码**+**新手避坑**+**适用场景**
- 四大模式横向对比，一眼知道该用谁

@[toc]()

# 一、工厂模式：统一造对象的“生产线”
## 1. 为什么需要工厂模式？（痛点拆解）
如果直接在代码中到处用`类名()`创建对象，会出现这些问题：
- 创建逻辑分散，参数校验、类型判断的代码重复写，维护成本极高；
- 新增一类对象时，要修改大量业务代码，违反“开放-封闭原则”；
- 容易因参数顺序、格式错误导致隐蔽bug，且难以排查。

## 2. 工厂模式核心思想
定义一个专门的“工厂类”，将所有对象的创建逻辑封装在其中，外部代码只需告诉工厂“想要什么类型的对象”，无需关心对象的创建细节（如参数校验、初始化配置）。

## 3. 工厂模式最简实现（通用电子产品示例）
```python
# 1. 定义基础产品类（抽象产品）
class Product:
    def show_info(self):
        """展示产品信息（子类可重写）"""
        pass

# 2. 定义具体产品类
class Phone(Product):
    def show_info(self):
        print("产品类型：手机，核心功能：通讯")

class Laptop(Product):
    def show_info(self):
        print("产品类型：笔记本电脑，核心功能：办公")

# 3. 核心：工厂类（统一创建对象）
class ProductFactory:
    @staticmethod
    def create_product(product_type):
        """
        统一创建产品对象
        :param product_type: 产品类型（phone/laptop）
        :return: 对应类型的产品实例
        """
        # 集中校验参数
        if product_type not in ["phone", "laptop"]:
            raise ValueError(f"不支持创建{product_type}类型的产品")
        
        # 集中创建对象
        if product_type == "phone":
            return Phone()
        elif product_type == "laptop":
            return Laptop()

# 4. 使用工厂创建对象（无需关心内部逻辑）
phone = ProductFactory.create_product("phone")
laptop = ProductFactory.create_product("laptop")

phone.show_info()  # 输出：产品类型：手机，核心功能：通讯
laptop.show_info() # 输出：产品类型：笔记本电脑，核心功能：办公
```

## 4. 工厂模式核心价值
- **低耦合**：对象创建与业务代码分离，修改创建逻辑只需改工厂类；
- **易维护**：参数校验、初始化逻辑集中管理，避免代码重复；
- **易扩展**：新增产品类型时，只需在工厂类中新增分支，无需修改业务代码。

## 5. 新手避坑指南
- 工厂类只负责“创建对象”，不要把业务逻辑写进工厂，避免职责过重；
- 简单场景用“简单工厂”即可，无需过度设计成“抽象工厂”；
- 工厂方法中要做好参数校验，避免返回非法对象。

# 二、单例模式：全局只一个实例
## 1. 为什么需要单例模式？（痛点拆解）
对于配置管理、数据库连接池、日志管理器这类类，如果被多次实例化，会出现：
- 多个实例各自维护数据，导致数据不一致（如修改了一个实例的配置，另一个实例未同步）；
- 重复创建昂贵资源（如数据库连接），浪费系统内存和性能；
- 全局状态混乱，难以统一管理。

## 2. 单例模式核心思想
确保某个类在程序运行期间，无论被实例化多少次，都只会创建**一个实例对象**，所有调用者共享这个实例。

## 3. 单例模式最简实现（重写__new__方法）
```python
class ConfigManager:
    """全局配置管理器（单例模式）"""
    # 类属性：存储唯一实例
    _instance = None
    # 类属性：标记是否已初始化，避免多次执行__init__
    _is_initialized = False

    # 核心：重写__new__方法，控制实例创建
    def __new__(cls, *args, **kwargs):
        # 如果实例不存在，创建新实例；否则返回已有实例
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        # 确保初始化逻辑只执行一次
        if not ConfigManager._is_initialized:
            # 模拟加载全局配置
            self.config = {
                "debug": True,
                "timeout": 30
            }
            ConfigManager._is_initialized = True

    # 示例方法：获取配置
    def get_config(self, key):
        return self.config.get(key)

# 测试单例模式
# 创建第一个实例
config1 = ConfigManager()
print(config1.get_config("timeout"))  # 输出：30

# 修改配置
config1.config["timeout"] = 60

# 创建第二个实例（实际是同一个）
config2 = ConfigManager()
print(config2.get_config("timeout"))  # 输出：60（共享修改后的配置）

# 验证实例是否相同
print(f"两个实例是否为同一个：{config1 is config2}")  # 输出：True
```

## 4. 单例模式核心价值
- **数据一致**：全局唯一实例，所有调用者共享同一套数据，避免状态混乱；
- **节省资源**：避免重复创建数据库连接、文件句柄等昂贵资源；
- **简化操作**：无需通过全局变量传递状态，直接调用单例实例即可。

## 5. 新手避坑指南
- 仅重写`__new__`不控制`__init__`，会导致初始化逻辑多次执行，需加标记位；
- 多线程场景下，需给`__new__`方法加线程锁，避免创建多个实例；
- 单例类不要设计成“万能类”，只负责核心功能（如配置管理），避免职责过重。

# 三、装饰器模式：不动原代码，增强功能
## 1. 为什么需要装饰器模式？（痛点拆解）
如果直接修改函数/方法的代码来增加功能（如日志、计时、权限校验），会出现：
- 功能代码与业务代码耦合，违背“单一职责原则”；
- 相同的增强逻辑（如日志）需要在多个函数中重复写，代码冗余；
- 后续想要移除增强功能，需修改原有业务代码，风险高。

## 2. 装饰器模式核心思想
在不修改原有函数/类代码的前提下，通过“装饰器”动态为其添加新功能，增强逻辑与业务逻辑完全分离，且可灵活叠加、移除。

## 3. 装饰器模式最简实现（计时+日志示例）
```python
import time

# 1. 定义计时装饰器（增强功能：统计函数执行耗时）
def timer_decorator(func):
    def wrapper(*args, **kwargs):
        # 增强逻辑：记录开始时间
        start_time = time.time()
        # 执行原函数
        result = func(*args, **kwargs)
        # 增强逻辑：计算并打印耗时
        cost_time = round(time.time() - start_time, 2)
        print(f"函数{func.__name__}执行耗时：{cost_time}秒")
        return result
    return wrapper

# 2. 定义日志装饰器（增强功能：记录函数调用信息）
def log_decorator(func):
    def wrapper(*args, **kwargs):
        # 增强逻辑：记录调用信息
        call_time = time.strftime("%Y-%m-%d %H:%M:%S")
        print(f"[{call_time}] 调用函数：{func.__name__}，参数：{args}")
        # 执行原函数
        result = func(*args, **kwargs)
        # 增强逻辑：记录返回结果
        print(f"[{call_time}] 函数{func.__name__}返回结果：{result}")
        return result
    return wrapper

# 3. 应用装饰器（叠加使用，无侵入增强）
@log_decorator  # 先执行
@timer_decorator # 后执行
def calculate_sum(n):
    """计算1到n的和（模拟业务函数）"""
    total = 0
    for i in range(1, n+1):
        total += i
    return total

# 4. 调用函数（自动触发增强功能）
calculate_sum(100000)
```

### 运行结果
```
[2026-02-10 17:00:00] 调用函数：calculate_sum，参数：(100000,)
函数calculate_sum执行耗时：0.01秒
[2026-02-10 17:00:00] 函数calculate_sum返回结果：5000050000
```

## 4. 装饰器模式核心价值
- **无侵入增强**：不修改原有业务代码，仅通过`@装饰器名`添加功能；
- **复用性高**：一个装饰器可应用到多个函数，避免增强逻辑重复；
- **灵活可控**：想要移除增强功能，只需删除`@装饰器名`，无任何副作用；
- **可叠加**：多个装饰器可叠加使用，实现多维度功能增强。

## 5. 新手避坑指南
- 装饰器的`wrapper`函数必须接收`*args, **kwargs`，否则会导致原函数参数传递失败；
- 多层装饰器的执行顺序是“靠近函数的先执行”；
- 装饰器要保留原函数的返回值，避免业务逻辑出错。

# 四、观察者模式：一对多通知，状态联动
## 1. 为什么需要观察者模式？（痛点拆解）
如果将状态变更的响应逻辑直接写死在被观察对象中，会出现：
- 响应逻辑与被观察对象高度耦合，新增响应逻辑需修改被观察对象代码；
- 无法动态添加/移除响应逻辑，扩展性差；
- 被观察对象需要知道所有响应逻辑的细节，违背“单一职责原则”。

## 2. 观察者模式核心思想
定义“被观察者（主题）”和“观察者”两类角色：
- 被观察者维护一个观察者列表，提供“注册/取消”观察者的方法；
- 当被观察者的状态发生变化时，自动通知所有已注册的观察者；
- 观察者收到通知后，执行自身的更新逻辑，与被观察者完全解耦。

## 3. 观察者模式最简实现（气象站+显示屏示例）
```python
# 1. 定义被观察者基类（主题）
class Subject:
    def __init__(self):
        self._observers = []  # 存储所有观察者

    def attach(self, observer):
        """注册观察者"""
        if observer not in self._observers:
            self._observers.append(observer)
            print(f"观察者{observer.name}已注册")

    def detach(self, observer):
        """取消观察者"""
        if observer in self._observers:
            self._observers.remove(observer)
            print(f"观察者{observer.name}已取消")

    def notify(self, msg):
        """通知所有观察者"""
        for observer in self._observers:
            observer.update(msg)

# 2. 定义观察者基类
class Observer:
    def __init__(self, name):
        self.name = name

    def update(self, msg):
        """收到通知后的更新逻辑（子类必须重写）"""
        pass

# 3. 定义具体被观察者（气象站）
class WeatherStation(Subject):
    def __init__(self):
        super().__init__()
        self._temperature = 0.0  # 温度（核心状态）

    @property
    def temperature(self):
        return self._temperature

    @temperature.setter
    def temperature(self, new_temp):
        self._temperature = new_temp
        # 状态变更，通知所有观察者
        self.notify(f"温度更新：{new_temp}℃")

# 4. 定义具体观察者（显示屏）
class IndoorDisplay(Observer):
    def update(self, msg):
        print(f"【{self.name}】{msg}（室内环境）")

class OutdoorDisplay(Observer):
    def update(self, msg):
        print(f"【{self.name}】{msg}（室外环境），请注意增减衣物")

# 5. 使用观察者模式
# 创建被观察者（气象站）
weather_station = WeatherStation()

# 创建观察者（显示屏）
indoor_display = IndoorDisplay("室内显示屏")
outdoor_display = OutdoorDisplay("室外显示屏")

# 注册观察者
weather_station.attach(indoor_display)
weather_station.attach(outdoor_display)

# 修改温度（自动通知观察者）
print("\n=== 修改温度为25℃ ===")
weather_station.temperature = 25.0

# 取消一个观察者
print("\n=== 取消室外显示屏 ===")
weather_station.detach(outdoor_display)

# 再次修改温度（仅室内显示屏收到通知）
print("\n=== 修改温度为30℃ ===")
weather_station.temperature = 30.0
```

### 运行结果
```
观察者室内显示屏已注册
观察者室外显示屏已注册

=== 修改温度为25℃ ===
【室内显示屏】温度更新：25.0℃（室内环境）
【室外显示屏】温度更新：25.0℃（室外环境），请注意增减衣物

=== 取消室外显示屏 ===
观察者室外显示屏已取消

=== 修改温度为30℃ ===
【室内显示屏】温度更新：30.0℃（室内环境）
```

## 4. 观察者模式核心价值
- **解耦**：被观察者与观察者完全分离，各自可独立修改、扩展；
- **灵活扩展**：新增观察者只需定义新的观察者类并注册，无需修改被观察者；
- **动态管理**：可随时注册/取消观察者，适配不同场景需求；
- **一对多联动**：一个状态变更可触发多个观察者的响应，无需手动调用。

## 5. 新手避坑指南
- 不再使用的观察者要及时调用`detach`取消注册，避免内存泄漏；
- 观察者的`update`方法不要写耗时逻辑，否则会阻塞所有通知；
- 被观察者的通知逻辑要保证健壮性，避免单个观察者报错导致整体通知失败。

# 五、四大设计模式横向对比
| 设计模式   | 核心作用                     | 典型适用场景                                       |
| ---------- | ---------------------------- | -------------------------------------------------- |
| 工厂模式   | 统一创建对象，解耦创建逻辑   | 多类型对象创建（如电子产品、支付方式）、插件化开发 |
| 单例模式   | 全局唯一实例，数据/资源共享  | 配置管理、数据库连接池、日志管理器、缓存管理器     |
| 装饰器模式 | 无侵入增强功能，不修改原代码 | 日志记录、性能统计、权限校验、参数校验             |
| 观察者模式 | 一对多通知，状态变更自动广播 | 消息推送、事件监听、实时数据展示、监控告警         |

# 六、新手避坑大全
1. **所有模式通用**：不要为了“用模式”而用模式，简单场景优先写简洁代码，模式是解决复杂问题的工具；
2. **工厂模式**：工厂类只负责对象创建，不要包含业务逻辑，避免职责过重；
3. **单例模式**：多线程场景下需给`__new__`加线程锁，防止创建多个实例；
4. **装饰器模式**：装饰器要保留原函数的参数和返回值，避免破坏业务逻辑；
5. **观察者模式**：及时注销无用观察者，避免内存泄漏和无效通知。

# 七、核心总结
本节课我们用通用极简示例，掌握了Python中最常用的四大设计模式，核心要点回顾：
1. **工厂模式**：核心是“统一创建对象”，解决创建逻辑分散、易出错的问题；
2. **单例模式**：核心是“全局唯一实例”，解决多实例数据不一致、资源浪费的问题；
3. **装饰器模式**：核心是“无侵入增强功能”，解决功能增强与业务逻辑耦合的问题；
4. **观察者模式**：核心是“一对多解耦通知”，解决状态变更与响应逻辑耦合的问题。

设计模式的本质是“解决特定问题的固定思路”，不用死记硬背，记住“创建混乱用工厂、全局唯一用单例、增强功能用装饰器、消息通知用观察者”即可。下一节我们将学习**Python 多线程编程**。

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/160136474>
