# Python全栈入门到实战【进阶篇 3】面向对象之继承：代码复用与功能扩展
在上一节中，我们学习了面向对象的**封装特性**，通过私有属性、私有方法与`@property`装饰器，实现了数据的隐藏与安全访问，让代码更健壮、更规范。但在实际开发中，我们常常会遇到**多个类拥有相同的属性与方法**的场景，如果在每个类中重复定义相同代码，会导致代码冗余、维护成本剧增、修改一处需全局同步。

面向对象的**第二大核心特性——继承**，正是为解决**代码复用**与**功能扩展**而生。继承允许我们定义一个**父类（基类）**，封装通用的属性与方法，再定义**子类（派生类）** 继承父类，直接拥有父类所有公开的成员，同时可以扩展自身独有的功能，或重写父类的逻辑。

继承是大型项目中**代码复用、模块化扩展、逻辑分层**的核心手段，也是理解多态、框架设计的前置基础。本节我们从“为什么用继承”出发，由浅入深掌握单继承、多层继承、方法重写、`super()`调用父类成员，以及继承中的私有成员规则，写出高复用、易扩展的面向对象代码。

本节核心学习内容：
- 为什么需要继承？告别重复代码，理解继承的核心价值；
- 继承基础语法：父类、子类定义，单继承的使用；
- 多层继承：层级化的代码复用与扩展；
- 方法重写（覆盖）：子类自定义父类已有逻辑；
- `super()`函数：子类中调用父类的属性与方法；
- 继承中的私有成员：父类私有属性/方法的访问规则；
- 实战案例：整合继承、重写、`super`实现业务分层；
- 新手避坑：继承高频错误与最佳实践。

# 一、先搞懂：为什么需要继承？告别代码冗余
我们先通过一个**无继承、重复编码**的反面案例，直观感受代码冗余的痛点，再理解继承的必要性。

## 场景：定义学生类、教师类、员工类
三个类都有**姓名、年龄**属性，都有**展示信息**的方法，只有各自独有的属性/方法不同。

### 无继承：重复编写代码（冗余严重）
```python
# 学生类
class Student:
    def __init__(self, name, age, stu_id):
        self.name = name
        self.age = age
        self.stu_id = stu_id

    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}，学号：{self.stu_id}")

# 教师类
class Teacher:
    def __init__(self, name, age, subject):
        self.name = name
        self.age = age
        self.subject = subject

    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}，科目：{self.subject}")

# 员工类
class Staff:
    def __init__(self, name, age, department):
        self.name = name
        self.age = age
        self.department = department

    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}，部门：{self.department}")
```

### 核心问题
1. **代码严重冗余**：`name`、`age`属性、`show_info`结构在三个类中重复编写；
2. **维护成本极高**：如果要修改`show_info`的输出格式，必须修改三个类；
3. **扩展性差**：新增一个“管理员类”，又要复制粘贴通用代码。

### 继承的解决思路
1. 抽取**所有类的通用属性与方法**，封装为一个**父类（Person）**；
2. 学生、教师、员工作为**子类**，继承父类，直接拥有通用成员；
3. 子类只需要编写**自己独有的属性与方法**，或修改父类已有逻辑。

这就是继承的核心价值：**一次定义，多处复用，一处修改，全局生效**。

# 二、继承的基础语法：单继承
## 1. 核心概念
- **父类（基类/超类）**：被继承的类，封装通用的属性与方法；
- **子类（派生类）**：继承父类的类，拥有父类公开成员，可扩展、重写。

## 2. 单继承语法
单继承指**一个子类只继承一个父类**，是最常用、最安全、最易维护的继承方式。
```python
# 父类
class 父类名:
    通用属性与方法

# 子类：继承父类
class 子类名(父类名):
    子类独有的属性与方法
```

## 3. 基础示例：使用继承重构上述案例
```python
# 父类：人类，封装通用属性与方法
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}")

# 子类：学生类，继承Person
class Student(Person):
    # 子类独有属性：学号
    def __init__(self, name, age, stu_id):
        # 调用父类构造方法，初始化通用属性
        super().__init__(name, age)
        self.stu_id = stu_id

    # 子类重写展示方法
    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}，学号：{self.stu_id}")

# 子类：教师类，继承Person
class Teacher(Person):
    def __init__(self, name, age, subject):
        super().__init__(name, age)
        self.subject = subject

    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}，科目：{self.subject}")

# 测试
stu = Student("张三", 20, "S1001")
tea = Teacher("李老师", 35, "Python全栈")
stu.show_info()
tea.show_info()
```

**运行结果**：
```
姓名：张三，年龄：20，学号：S1001
姓名：李老师，年龄：35，科目：Python全栈
```

可以看到：子类**无需重复编写**`name`、`age`与基础`show_info`，代码量直接减少50%以上。

## 4. 继承的本质
子类会**自动拥有父类中所有非私有的属性与方法**，私有成员除外（下文详细讲解）。
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        print(f"{self.name}在吃饭")

class Student(Person):
    pass

# 子类无任何代码，直接拥有父类所有公开成员
stu = Student("小明", 18)
print(stu.name, stu.age)  # 小明 18
stu.eat()  # 小明在吃饭
```

# 三、多层继承：层级化的复用与扩展
继承不仅支持**单层继承**，还支持**多层继承**：
`祖父类 → 父类 → 子类`

父类继承祖父类，子类继承父类，子类会**拥有所有上层父类的公开成员**，实现层级化的功能分层。

## 示例：多层继承
```python
# 祖父类：最基础的生物类
class Creature:
    def breathe(self):
        print("生物会呼吸")

# 父类：人类，继承生物类
class Person(Creature):
    def __init__(self, name):
        self.name = name

    def speak(self):
        print(f"{self.name}会说话")

# 子类：学生，继承人类
class Student(Person):
    def study(self):
        print(f"{self.name}在学习")

# 子类拥有三层所有公开方法
stu = Student("小红")
stu.breathe()  # 来自祖父类
stu.speak()    # 来自父类
stu.study()    # 自身独有
```

**运行结果**：
```
生物会呼吸
小红会说话
小红在学习
```

**注意**：多层继承层级不宜过深（一般不超过3层），过深会导致逻辑混乱、可读性下降。

# 四、方法重写（覆盖）：子类自定义父类逻辑
继承不是单纯的“复制代码”，子类可以**重新定义父类中已有的方法**，实现自己的业务逻辑，这就是**方法重写（Override）**。

## 1. 方法重写规则
1. 子类方法的**方法名、参数列表**与父类完全一致；
2. 调用时，**优先执行子类的重写方法**，而非父类原方法；
3. 常用于：父类逻辑不满足子类需求，需要定制化修改。

## 2. 基础示例：方法重写
```python
class Person:
    def work(self):
        print("人类需要工作")

class Student(Person):
    # 重写父类的work方法
    def work(self):
        print("学生的工作是学习")

class Teacher(Person):
    # 重写父类的work方法
    def work(self):
        print("老师的工作是教书")

p = Person()
s = Student()
t = Teacher()

p.work()  # 人类需要工作
s.work()  # 学生的工作是学习（执行子类重写方法）
t.work()  # 老师的工作是教书（执行子类重写方法）
```

## 3. 重写`__init__`构造方法
构造方法也可以重写，子类`__init__`通常会先调用父类`__init__`，再初始化自身属性，这是最标准的写法。
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Student(Person):
    # 重写构造方法
    def __init__(self, name, age, stu_id):
        # 先调用父类构造，初始化通用属性
        super().__init__(name, age)
        # 再初始化子类独有属性
        self.stu_id = stu_id

    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}，学号：{self.stu_id}")

stu = Student("赵四", 22, "S2002")
stu.show_info()
```

# 五、super()函数：子类中调用父类成员
当子类重写了父类方法后，**仍然想使用父类的原逻辑**，就需要使用`super()`函数。

`super()`的核心作用：**在子类中，调用父类的方法（包括构造方法、普通方法）**。

## 1. 语法
```python
# 调用父类构造方法
super().__init__(参数)

# 调用父类普通方法
super().方法名(参数)
```

## 2. 场景1：构造方法中调用父类`__init__`（最常用）
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Student(Person):
    def __init__(self, name, age, stu_id):
        # 调用父类构造，复用初始化逻辑
        super().__init__(name, age)
        self.stu_id = stu_id
```

## 3. 场景2：普通方法中调用父类原方法
```python
class Person:
    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}")

class Student(Person):
    def __init__(self, name, age, stu_id):
        super().__init__(name, age)
        self.stu_id = stu_id

    def show_info(self):
        # 先执行父类的show_info，输出通用信息
        super().show_info()
        # 再输出子类独有信息
        print(f"学号：{self.stu_id}")

stu = Student("小刘", 19, "S3003")
stu.show_info()
```

**运行结果**：
```
姓名：小刘，年龄：19
学号：S3003
```

这是**最优雅的复用方式**：保留父类逻辑，扩展子类逻辑。

# 六、继承中的私有成员：子类不能直接访问
我们在封装章节学习了**双下划线私有成员**，在继承中，私有成员有严格的访问规则：

**父类中以`__`开头的私有属性、私有方法，子类无法直接访问、无法重写**。

## 1. 私有属性：子类不能直接访问
```python
class Person:
    def __init__(self, name):
        self.name = name
        self.__age = 30  # 私有属性

class Student(Person):
    def show(self):
        print(self.name)
        # print(self.__age)  # 报错，无法访问父类私有属性

stu = Student("小张")
stu.show()
# print(stu.__age)  # 外部也无法访问
```

## 2. 私有方法：子类不能直接调用、不能重写
```python
class Person:
    def __run(self):
        print("父类私有跑步方法")

class Student(Person):
    def test(self):
        # self.__run()  # 报错，无法调用父类私有方法
        pass
```

## 3. 正确访问方式：通过父类公开接口访问
父类私有成员，只能通过**父类自身的公开方法**访问，子类间接调用公开方法即可。
```python
class Person:
    def __init__(self, name):
        self.name = name
        self.__age = 30

    # 公开接口，获取私有属性
    def get_age(self):
        return self.__age

class Student(Person):
    def show(self):
        print(f"姓名：{self.name}，年龄：{self.get_age()}")

stu = Student("小王")
stu.show()  # 姓名：小王，年龄：30
```

**核心规则**：**私有成员不参与继承暴露，封装安全边界不会被继承打破**。

# 七、实战案例：校园人员管理系统（继承+重写+super）
我们用一个完整实战，整合本节所有知识点，实现**父类通用逻辑 + 子类定制化扩展**，贴近真实开发场景。

## 需求
1. 父类`Person`：通用属性`name、age`，通用方法`show_info、work`；
2. 子类`Student`：独有`stu_id、score`，重写`work`，扩展`show_info`；
3. 子类`Teacher`：独有`subject、salary`，重写`work`，扩展`show_info`；
4. 所有子类通过`super()`复用父类逻辑，不重复编码。

## 完整代码
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def show_info(self):
        print(f"【通用信息】姓名：{self.name}，年龄：{self.age}")

    def work(self):
        print("人类的核心工作是创造价值")

class Student(Person):
    def __init__(self, name, age, stu_id, score):
        # 复用父类构造
        super().__init__(name, age)
        self.stu_id = stu_id
        self.score = score

    # 重写工作方法
    def work(self):
        print("学生的工作是学习知识、提升能力")

    # 扩展展示信息
    def show_info(self):
        # 调用父类展示
        super().show_info()
        print(f"【学生信息】学号：{self.stu_id}，成绩：{self.score}")

class Teacher(Person):
    def __init__(self, name, age, subject, salary):
        super().__init__(name, age)
        self.subject = subject
        self.salary = salary

    def work(self):
        print("老师的工作是教书育人、传递知识")

    def show_info(self):
        super().show_info()
        print(f"【教师信息】科目：{self.subject}，薪资：{self.salary}")

# 测试
if __name__ == "__main__":
    print("===== 学生信息 =====")
    stu = Student("陈华", 20, "S9527", 92)
    stu.show_info()
    stu.work()

    print("\n===== 教师信息 =====")
    tea = Teacher("周明", 40, "Python全栈", 15000)
    tea.show_info()
    tea.work()
```

## 运行结果
```
===== 学生信息 =====
【通用信息】姓名：陈华，年龄：20
【学生信息】学号：S9527，成绩：92
学生的工作是学习知识、提升能力

===== 教师信息 =====
【通用信息】姓名：周明，年龄：40
【教师信息】科目：Python全栈，薪资：15000
老师的工作是教书育人、传递知识
```

# 八、新手避坑大全：继承高频错误
## 避坑1：子类重写方法时，参数列表与父类不一致
```python
class Person:
    def work(self, place):
        print(f"在{place}工作")

class Student(Person):
    # 错误：参数个数不同，不是规范重写
    def work(self):
        print("学习")
```
**解决方案**：规范重写要求**方法名、参数个数、参数含义**保持一致。

## 避坑2：子类`__init__`忘记调用`super().__init__`
```python
class Person:
    def __init__(self, name):
        self.name = name

class Student(Person):
    def __init__(self, name, stu_id):
        # 忘记super().__init__(name)
        self.stu_id = stu_id

stu = Student("小李", "S001")
print(stu.name)  # 报错，没有name属性
```
**解决方案**：子类重写`__init__`时，**第一行优先调用父类构造方法**。

## 避坑3：试图访问/重写父类私有成员
```python
class Person:
    def __init__(self):
        self.__age = 20

class Student(Person):
    def show(self):
        # print(self.__age)  # 报错
        pass
```
**解决方案**：通过父类公开`get`方法访问私有属性，不强行突破封装边界。

## 避坑4：多层继承层级过深，导致逻辑混乱
建议继承层级**不超过3层**，过多层级会让代码难以调试、阅读、维护。

## 避坑5：误以为子类可以修改父类的类属性
子类继承父类类属性，但**子类修改自身类属性不会影响父类**，二者相互独立。

# 九、核心总结
本节我们系统学习了面向对象的**第二大核心特性——继承**，彻底解决了代码冗余、复用性差的问题，核心要点回顾：

1. **继承的核心价值**：
   - 抽取通用代码到父类，子类直接继承，**减少冗余、提升复用**；
   - 子类可扩展独有功能，或重写父类逻辑，**高可扩展、易维护**。

2. **基础语法**：
   - 单继承：`class 子类(父类):`，最常用、最安全；
   - 多层继承：`祖父类→父类→子类`，子类拥有所有上层公开成员；
   - 子类自动拥有父类**非私有**的属性与方法。

3. **方法重写**：
   - 子类定义与父类**同名、同参**的方法，覆盖父类逻辑；
   - 调用时优先执行子类重写方法，满足定制化需求。

4. **super()函数**：
   - 子类中调用父类的构造方法、普通方法；
   - 标准写法：子类`__init__`第一行`super().__init__(参数)`；
   - 实现“保留父类逻辑 + 扩展子类逻辑”。

5. **继承与私有**：
   - 父类`__`开头的私有成员，子类**无法直接访问、无法重写**；
   - 只能通过父类**公开接口**间接访问，保证封装安全。

6. **最佳实践**：
   - 优先使用**单继承**，避免复杂多继承；
   - 继承层级≤3层，避免逻辑混乱；
   - 重写方法保持参数一致，使用`super`复用父类逻辑。

继承是面向对象**代码复用与分层设计**的基石，掌握继承后，我们就可以写出模块化、可扩展、易维护的大型项目代码。

下一节，我们将学习面向对象**第三大核心特性——多态**，理解“一个接口，多种实现”的设计思想，完成面向对象三大特性的最后一块拼图。

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/159348978>
