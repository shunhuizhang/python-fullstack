# Python全栈入门到实战【进阶篇 6】面向对象高级特性：抽象类与接口
在上一节中，我们学习了`@property`进阶、类方法与静态方法，进一步提升了面向对象代码的规范性和灵活性。但在实际开发中，我们还会遇到一个核心问题：**父类被误实例化，或子类未按要求重写父类的核心方法**，导致代码逻辑混乱、功能异常。

比如我们定义了`Person`父类，核心方法是`work`，本意是让`Student、Teacher`等子类重写该方法，但新手可能会直接实例化`Person`对象，或子类忘记重写`work`，导致调用时执行父类空逻辑，引发bug。

本节课我们将学习面向对象高级特性——**抽象类与接口**，无需深入复杂的底层原理，重点掌握“抽象类作为模板、强制子类重写核心方法”的核心用法，从源头避免上述问题，同时为后续学习框架（如Django、Flask）中的接口设计打下基础。

本节核心学习内容：
- 为什么需要抽象类？解决“父类误实例化、子类未重写核心方法”的痛点；
- 抽象类基础：`abc`模块、抽象方法（`@abstractmethod`）、抽象类的核心规则；
- 抽象类与普通父类的区别：一张表格彻底分清；
- 接口模拟：Python中无真正接口，用“全抽象方法的抽象类”模拟接口；
- 实战案例：整合抽象类、接口，改造校园人员管理系统，规范子类实现；
- 新手避坑：抽象类实例化、子类未重写抽象方法等高频错误。

# 一、先搞懂：为什么需要抽象类？告别混乱的父类与子类
我们先通过一个“无抽象类”的反面案例，直观感受新手常踩的坑，再理解抽象类的核心价值。

## 场景：用Person作为父类，Student、Teacher作为子类，要求所有子类必须实现work方法
### 无抽象类：父类可实例化，子类可省略重写，逻辑混乱
```python
class Person:
    def work(self):
        """父类核心方法，本意是让子类重写"""
        pass  # 空逻辑，无实际功能

class Student(Person):
    # 忘记重写work方法，继承父类空逻辑
    def study(self):
        print("学生学习")

class Teacher(Person):
    # 正确重写work方法
    def work(self):
        print("教师教书")

# 痛点1：父类被误实例化（无意义，父类本应是模板）
person = Person()
person.work()  # 无任何输出，逻辑冗余

# 痛点2：子类未重写work，调用时执行空逻辑（隐藏bug）
stu = Student()
stu.work()  # 无任何输出，不符合预期
stu.study()  # 学生学习

# 痛点3：无法强制子类重写work，新手易遗漏
```

### 核心问题
1.  父类`Person`是“模板类”，本意是规范子类，却能被直接实例化，无实际业务意义；
2.  子类可随意省略重写父类核心方法（如`Student`未重写`work`），导致调用时功能异常；
3.  代码可读性差，其他开发者无法快速区分“父类是否可实例化”“子类必须重写哪些方法”。

### 抽象类的解决思路
1.  定义`Person`为**抽象类**，禁止实例化（不能创建`Person`对象）；
2.  将`work`定义为**抽象方法**，强制所有子类必须重写该方法，不重写则无法创建子类对象；
3.  抽象类作为“规范模板”，仅定义核心方法的名称和参数，不实现具体逻辑（或只实现通用逻辑）。

这就是抽象类的核心价值：**规范子类实现，强制子类重写核心方法，禁止父类误实例化，让代码逻辑更清晰、更健壮**。

# 二、抽象类基础：语法与核心规则
Python中没有内置的抽象类语法，需要借助标准库中的`abc`模块（Abstract Base Class，抽象基类），核心是两个东西：`ABC`类（抽象类的父类）、`@abstractmethod`装饰器（标记抽象方法）。

## 1. 抽象类的基础语法
```python
# 1. 导入必要的类和装饰器（固定写法）
from abc import ABC, abstractmethod

# 2. 定义抽象类：继承ABC类
class 抽象类名(ABC):
    # 3. 定义抽象方法：用@abstractmethod装饰，无具体实现（或空实现）
    @abstractmethod
    def 抽象方法名(self, 可选参数):
        # 可选：仅写注释，不写具体逻辑（强制子类重写）
        pass

    # 4. 抽象类中可包含普通方法、类方法、静态方法（非必须）
    def 普通方法名(self):
        # 有具体实现，子类可直接继承使用
        print("抽象类中的普通方法")
```

## 2. 抽象类的3个核心规则
1.  抽象类**必须继承`ABC`类**，否则无法被识别为抽象类；
2.  抽象类中**至少有一个抽象方法**（用`@abstractmethod`装饰）；
3.  抽象类**不能直接实例化**（不能创建对象），子类必须重写抽象类中**所有**抽象方法，才能实例化；若子类未重写所有抽象方法，子类也会变成抽象类，无法实例化。

## 3. 基础示例：用抽象类改造Person类
```python
from abc import ABC, abstractmethod

# 抽象类：继承ABC，不能实例化
class Person(ABC):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    # 抽象方法：强制子类重写
    @abstractmethod
    def work(self):
        pass  # 无具体实现，仅作为规范

    # 抽象类中的普通方法：子类可直接继承
    def show_basic_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}")

# 子类1：Student，重写抽象方法work
class Student(Person):
    def __init__(self, name, age, stu_id):
        super().__init__(name, age)
        self.stu_id = stu_id

    # 必须重写抽象方法work，否则无法创建Student对象
    def work(self):
        print(f"{self.name}（学生）：专注学习，学号：{self.stu_id}")

# 子类2：Teacher，重写抽象方法work
class Teacher(Person):
    def __init__(self, name, age, subject):
        super().__init__(name, age)
        self.subject = subject

    def work(self):
        print(f"{self.name}（教师）：讲授{self.subject}，认真备课")

# 测试抽象类的核心规则
# 1. 抽象类不能实例化（报错）
# person = Person("张三", 20)  # 报错：TypeError: Can't instantiate abstract class Person with abstract method work

# 2. 子类未重写抽象方法，无法实例化（报错）
# class Admin(Person):
#     pass
# admin = Admin("王叔", 40)  # 报错：TypeError: Can't instantiate abstract class Admin with abstract method work

# 3. 子类重写抽象方法，可正常实例化
stu = Student("张三", 20, "S1001")
tea = Teacher("李老师", 35, "Python全栈")

# 调用重写的抽象方法
stu.work()  # 张三（学生）：专注学习，学号：S1001
tea.work()  # 李老师（教师）：讲授Python全栈，认真备课

# 调用抽象类中的普通方法（继承使用）
stu.show_basic_info()  # 姓名：张三，年龄：20
tea.show_basic_info()  # 姓名：李老师，年龄：35
```

## 4. 抽象类与普通父类的核心区别
用表格清晰区分，避免混淆，这是新手最容易踩的坑之一：

| 对比维度 | 抽象类（继承ABC，有抽象方法）              | 普通父类                                       |
| -------- | ------------------------------------------ | ---------------------------------------------- |
| 实例化   | 不能直接实例化（报错）                     | 可以直接实例化                                 |
| 子类要求 | 必须重写所有抽象方法，否则无法实例化       | 可重写父类方法，也可继承使用，无强制要求       |
| 核心作用 | 作为“规范模板”，强制子类实现核心方法       | 封装通用逻辑，实现代码复用                     |
| 方法类型 | 可包含抽象方法、普通方法、类方法、静态方法 | 仅包含普通方法、类方法、静态方法（无抽象方法） |

**新手判断技巧**：若父类是“模板”，仅用于规范子类，不希望被实例化，就用抽象类；若父类有具体逻辑，需要被实例化，就用普通父类。

# 三、接口模拟：Python中如何实现“接口”？
在Java、C#等语言中，有“接口”（Interface）的概念：**接口中没有任何方法实现，全是抽象方法**，用于定义“子类必须实现的方法规范”，但不能包含普通方法、类方法等。

而Python中**没有真正的接口**，我们通常用“**全是抽象方法的抽象类**”模拟接口——即抽象类中所有方法都是`@abstractmethod`装饰的抽象方法，无任何具体实现，仅作为“方法规范”。

## 1. 接口模拟的基础语法
```python
from abc import ABC, abstractmethod

# 模拟接口：全是抽象方法，无任何具体实现
class 接口名(ABC):
    @abstractmethod
    def 方法1(self):
        pass

    @abstractmethod
    def 方法2(self):
        pass

# 子类必须重写接口中的所有抽象方法，才能实例化
class 子类名(接口名):
    def 方法1(self):
        # 具体实现
        pass

    def 方法2(self):
        # 具体实现
        pass
```

## 2. 接口模拟的核心场景
当我们需要“**多个不同类，必须实现相同的一组方法**”时，用接口模拟规范它们，比如：校园中的所有人员（学生、教师、管理员），必须实现“工作（work）”和“展示信息（show_info）”两个方法，就可以定义一个接口。

### 示例：模拟Workable接口，规范所有人员的工作与展示功能
```python
from abc import ABC, abstractmethod

# 模拟接口：所有人员必须实现的两个方法
class Workable(ABC):
    @abstractmethod
    def work(self):
        """工作方法：所有人员必须实现"""
        pass

    @abstractmethod
    def show_info(self):
        """展示信息方法：所有人员必须实现"""
        pass

# 学生类：实现Workable接口，必须重写两个抽象方法
class Student(Workable):
    def __init__(self, name, stu_id):
        self.name = name
        self.stu_id = stu_id

    def work(self):
        print(f"{self.name}（学号：{self.stu_id}）：学习Python全栈")

    def show_info(self):
        print(f"【学生】姓名：{self.name}，学号：{self.stu_id}")

# 教师类：实现Workable接口，必须重写两个抽象方法
class Teacher(Workable):
    def __init__(self, name, subject):
        self.name = name
        self.subject = subject

    def work(self):
        print(f"{self.name}：讲授{self.subject}课程")

    def show_info(self):
        print(f"【教师】姓名：{self.name}，科目：{self.subject}")

# 管理员类：实现Workable接口，必须重写两个抽象方法
class Admin(Workable):
    def __init__(self, name, department):
        self.name = name
        self.department = department

    def work(self):
        print(f"{self.name}（{self.department}）：负责校园日常管理")

    def show_info(self):
        print(f"【管理员】姓名：{self.name}，部门：{self.department}")

# 测试：所有子类都实现了接口的两个方法
people_list = [
    Student("张三", "S1001"),
    Teacher("李老师", "Python全栈"),
    Admin("王叔", "后勤保障部")
]

for person in people_list:
    person.show_info()
    person.work()
    print("-" * 30)
```

**运行结果**：
```
【学生】姓名：张三，学号：S1001
张三（学号：S1001）：学习Python全栈
------------------------------
【教师】姓名：李老师，科目：Python全栈
李老师：讲授Python全栈课程
------------------------------
【管理员】姓名：王叔，部门：后勤保障部
王叔（后勤保障部）：负责校园日常管理
------------------------------
```

## 3. 抽象类与接口（模拟）的区别
这是面试高频考点，用简单的话和表格区分，不用死记硬背：
- 抽象类：**可以有抽象方法，也可以有普通方法、类方法**（有具体实现），核心是“规范子类+复用通用逻辑”；
- 接口（模拟）：**只有抽象方法，无任何具体实现**，核心是“仅规范子类，不提供任何复用逻辑”。

| 对比维度 | 抽象类                                          | 接口（Python模拟）                                    |
| -------- | ----------------------------------------------- | ----------------------------------------------------- |
| 方法类型 | 抽象方法 + 普通方法/类方法/静态方法             | 仅抽象方法                                            |
| 核心作用 | 规范子类 + 复用通用逻辑                         | 仅规范子类，无复用                                    |
| 继承场景 | 子类与父类有“is-a”关系（如Student is a Person） | 子类与接口有“can-do”关系（如Student can do Workable） |

**判断技巧**：若需要“规范子类+复用代码”，用抽象类；若只需要“规范子类，不复用任何代码”，用接口模拟。

# 四、实战案例：整合抽象类与接口，改造校园人员管理系统
结合本节课知识点，整合前几篇的`@property`、类方法、静态方法，用抽象类+接口模拟，改造校园人员管理系统，实现“规范子类、强制重写、代码复用”的核心需求，贴合真实开发场景。

## 需求
1.  定义抽象类`Person`（继承ABC）：
    - 封装通用属性`name、age`（私有属性，用`@property`访问）；
    - 包含抽象方法`show_info`（展示信息）、`work`（工作）；
    - 包含普通方法`verify_age`（校验年龄，子类可直接继承）；
2.  定义接口`Scoreable`（模拟）：
    - 包含抽象方法`get_score`（获取成绩）、`set_score`（修改成绩）；
3.  子类实现：
    - `Student`：继承`Person`，实现`Scoreable`接口，重写所有抽象方法；
    - `Teacher`：继承`Person`，不实现`Scoreable`（教师无成绩），重写`show_info、work`；
    - `Admin`：继承`Person`，不实现`Scoreable`，重写`show_info、work`；
4.  类方法：统计各类人员总数；
5.  静态方法：校验成绩合法性。

## 完整代码
```python
from abc import ABC, abstractmethod

# ------------------------------
# 1. 模拟接口：Scoreable（仅学生需要实现，有成绩相关方法）
# ------------------------------
class Scoreable(ABC):
    @abstractmethod
    def get_score(self):
        """获取成绩（抽象方法，必须重写）"""
        pass

    @abstractmethod
    def set_score(self, new_score):
        """修改成绩（抽象方法，必须重写）"""
        pass

# ------------------------------
# 2. 抽象类：Person（规范所有人员，复用通用逻辑）
# ------------------------------
class Person(ABC):
    # 类属性：统计各类人员总数
    total_students = 0
    total_teachers = 0
    total_admins = 0

    def __init__(self, name, age):
        self.__name = name
        self.__age = age
        # 校验年龄（普通方法，复用）
        self.verify_age()

    # @property：访问私有属性
    @property
    def name(self):
        return self.__name

    @property
    def age(self):
        return self.__age

    # 普通方法：校验年龄（子类可直接继承，无需重写）
    def verify_age(self):
        if not (isinstance(self.__age, int) and 6 <= self.__age <= 60):
            print(f"警告：{self.__name}的年龄{self.__age}不合法（6-60岁）")

    # 抽象方法：展示信息（所有子类必须重写）
    @abstractmethod
    def show_info(self):
        pass

    # 抽象方法：工作（所有子类必须重写）
    @abstractmethod
    def work(self):
        pass

    # 类方法：统计各类人员总数
    @classmethod
    def get_total_count(cls):
        total = cls.total_students + cls.total_teachers + cls.total_admins
        return f"总人数：{total}，学生：{cls.total_students}人，教师：{cls.total_teachers}人，管理员：{cls.total_admins}人"

    # 静态方法：校验成绩合法性（工具方法）
    @staticmethod
    def is_valid_score(score):
        return isinstance(score, (int, float)) and 0 <= score <= 100

# ------------------------------
# 3. 子类1：Student（继承Person，实现Scoreable接口）
# ------------------------------
class Student(Person, Scoreable):
    def __init__(self, name, age, stu_id, score):
        super().__init__(name, age)
        self.__stu_id = stu_id
        # 校验成绩
        if Person.is_valid_score(score):
            self.__score = score
        else:
            self.__score = 0
            print(f"警告：{self.name}的成绩不合法，默认设为0分")
        # 类属性自增
        Person.total_students += 1

    @property
    def stu_id(self):
        return self.__stu_id

    # 实现Scoreable接口的抽象方法：获取成绩
    def get_score(self):
        return self.__score

    # 实现Scoreable接口的抽象方法：修改成绩
    def set_score(self, new_score):
        if Person.is_valid_score(new_score):
            self.__score = new_score
            print(f"{self.name}的成绩修改成功，当前成绩：{self.__score}")
        else:
            print("成绩不合法，修改失败")

    # 重写Person的抽象方法：展示信息
    def show_info(self):
        print(f"【学生】姓名：{self.name}，年龄：{self.age}，学号：{self.stu_id}，成绩：{self.__score}")

    # 重写Person的抽象方法：工作
    def work(self):
        print(f"{self.name}（学生）：专注学习，当前成绩：{self.__score}分")

# ------------------------------
# 4. 子类2：Teacher（继承Person，不实现Scoreable）
# ------------------------------
class Teacher(Person):
    def __init__(self, name, age, subject, salary):
        super().__init__(name, age)
        self.__subject = subject
        self.__salary = salary
        Person.total_teachers += 1

    @property
    def subject(self):
        return self.__subject

    @property
    def salary(self):
        return self.__salary

    def show_info(self):
        print(f"【教师】姓名：{self.name}，年龄：{self.age}，科目：{self.subject}，薪资：{self.salary}元")

    def work(self):
        print(f"{self.name}（教师）：讲授{self.subject}课程，月薪{self.salary}元")

# ------------------------------
# 5. 子类3：Admin（继承Person，不实现Scoreable）
# ------------------------------
class Admin(Person):
    def __init__(self, name, age, department):
        super().__init__(name, age)
        self.__department = department
        Person.total_admins += 1

    @property
    def department(self):
        return self.__department

    def show_info(self):
        print(f"【管理员】姓名：{self.name}，年龄：{self.age}，部门：{self.department}")

    def work(self):
        print(f"{self.name}（管理员）：负责{self.department}日常管理工作")

# ------------------------------
# 实战测试
# ------------------------------
if __name__ == "__main__":
    print("=== 创建校园人员对象 ===")
    # 学生（实现Scoreable接口）
    stu1 = Student("张三", 20, "S1001", 92)
    stu2 = Student("李四", 17, "S1002", 105)  # 成绩不合法，默认0分
    # 教师
    tea1 = Teacher("李老师", 35, "Python全栈", 15000)
    # 管理员（年龄不合法）
    admin1 = Admin("王叔", 65, "后勤保障部")  # 年龄不合法警告

    print("\n=== 展示所有人员信息 ===")
    people_list = [stu1, stu2, tea1, admin1]
    for person in people_list:
        person.show_info()
        person.work()
        # 学生额外调用Scoreable接口的方法
        if isinstance(person, Student):
            print(f"当前成绩：{person.get_score()}")
            person.set_score(95)
        print("-" * 40)

    print("\n=== 人员统计 ===")
    print(Person.get_total_count())

    # 测试抽象类不能实例化（注释掉，否则报错）
    # person = Person("测试", 30)  # 报错：Can't instantiate abstract class Person with abstract methods show_info, work
```

## 运行结果
```
=== 创建校园人员对象 ===
警告：李四的成绩不合法，默认设为0分
警告：王叔的年龄65不合法（6-60岁）

=== 展示所有人员信息 ===
【学生】姓名：张三，年龄：20，学号：S1001，成绩：92
张三（学生）：专注学习，当前成绩：92分
当前成绩：92
张三的成绩修改成功，当前成绩：95
----------------------------------------
【学生】姓名：李四，年龄：17，学号：S1002，成绩：0
李四（学生）：专注学习，当前成绩：0分
当前成绩：0
李四的成绩修改成功，当前成绩：95
----------------------------------------
【教师】姓名：李老师，年龄：35，科目：Python全栈，薪资：15000元
李老师（教师）：讲授Python全栈课程，月薪15000元
----------------------------------------
【管理员】姓名：王叔，年龄：65，部门：后勤保障部
王叔（管理员）：负责后勤保障部日常管理工作
----------------------------------------

=== 人员统计 ===
总人数：4，学生：2人，教师：1人，管理员：1人
```

# 五、新手避坑大全：抽象类与接口高频错误
## 避坑1：忘记继承ABC类，误以为定义了抽象类
```python
from abc import abstractmethod

# 错误：未继承ABC，不是抽象类
class Person:
    @abstractmethod
    def work(self):
        pass

# 可以实例化（无报错），违背抽象类初衷
person = Person()
```
**报错原因**：抽象类必须继承`ABC`类，否则`@abstractmethod`装饰器无效，无法禁止实例化。
**解决方案**：所有抽象类都要继承`ABC`，语法：`class 抽象类名(ABC):`。

## 避坑2：抽象类实例化（新手最常犯）
```python
from abc import ABC, abstractmethod

class Person(ABC):
    @abstractmethod
    def work(self):
        pass

# 错误：抽象类不能实例化
person = Person()  # 报错：TypeError: Can't instantiate abstract class Person with abstract method work
```
**报错原因**：抽象类是“模板类”，核心作用是规范子类，禁止直接实例化。
**解决方案**：仅实例化子类（需重写所有抽象方法），不实例化抽象类。

## 避坑3：子类未重写所有抽象方法，尝试实例化
```python
from abc import ABC, abstractmethod

class Person(ABC):
    @abstractmethod
    def work(self):
        pass

    @abstractmethod
    def show_info(self):
        pass

# 错误：子类只重写了一个抽象方法，未重写show_info
class Student(Person):
    def work(self):
        print("学习")

# 无法实例化
stu = Student()  # 报错：Can't instantiate abstract class Student with abstract method show_info
```
**报错原因**：子类必须重写抽象类中**所有**抽象方法，缺一不可，否则子类也会被视为抽象类，无法实例化。
**解决方案**：子类重写抽象类中的每一个抽象方法，确保无遗漏。

## 避坑4：混淆抽象类与接口，接口中写普通方法
```python
from abc import ABC, abstractmethod

# 错误：模拟接口中写了普通方法（接口应全是抽象方法）
class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

    # 错误：接口不能有普通方法
    def show_info(self):
        print("展示信息")
```
**问题原因**：接口（模拟）的核心是“仅规范方法，无任何实现”，普通方法会破坏接口的纯粹性。
**解决方案**：模拟接口中只定义抽象方法，普通方法放到抽象类中，用于复用。

## 避坑5：抽象方法中写具体实现，违背抽象类初衷
```python
from abc import ABC, abstractmethod

class Person(ABC):
    @abstractmethod
    def work(self):
        # 错误：抽象方法不应有具体实现（强制子类重写，父类实现无意义）
        print("工作")
```
**问题原因**：抽象方法的作用是“定义规范”，具体实现应由子类根据自身需求编写，父类实现会导致子类继承后无需重写，违背抽象类的核心目的。
**解决方案**：抽象方法仅写注释（说明方法作用），不写具体逻辑，或仅写`pass`。

# 六、核心总结
本节课我们学习了面向对象高级特性——抽象类与接口（新手简化版），重点解决了“父类误实例化、子类未重写核心方法”的痛点，核心要点回顾：
1.  抽象类的核心价值：**规范子类实现，强制子类重写核心方法，禁止父类误实例化**，同时可复用通用逻辑。
2.  抽象类基础语法（新手必记）：
    - 导入：`from abc import ABC, abstractmethod`；
    - 定义：抽象类继承`ABC`，抽象方法用`@abstractmethod`装饰；
    - 规则：抽象类不能实例化，子类必须重写所有抽象方法才能实例化。
3.  接口模拟（Python专属）：
    - Python无真正接口，用“全是抽象方法的抽象类”模拟；
    - 核心作用：仅规范子类方法，不提供任何代码复用。
4.  核心区别（新手必背）：
    - 抽象类：有抽象方法+普通方法，规范+复用；
    - 接口（模拟）：仅抽象方法，仅规范，不复用。
5.  最佳实践：
    - 若父类是“模板”，需规范子类+复用代码，用抽象类；
    - 若只需规范子类方法，不复用代码，用接口模拟；
    - 抽象方法不写具体实现，普通方法写通用逻辑；
    - 子类必须重写所有抽象方法，避免报错。

掌握抽象类与接口的简化用法后，你编写的面向对象代码将更规范、更健壮，能够应对中小型项目的模块化开发需求，也为后续学习框架中的接口设计、多态进阶打下基础。

下一节，我们将学习**面向对象实战：小型项目开发（学生管理系统V2.0）**，整合前6篇的所有知识点（封装、继承、多态、@property、类方法、抽象类等），实现一个完整、可运行的小型项目，让你真正做到“学以致用”。

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/159725160>
