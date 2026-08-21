# Python全栈入门到实战【进阶篇 5】面向对象高级技巧：@property进阶与方法分类
在上一节中，我们完成了面向对象三大核心特性（封装、继承、多态）的学习，掌握了模块化、可扩展代码的编写方法。但在实际开发中，我们还会遇到更细致的需求：
- 想修改`@property`装饰的“只读属性”，又想保留校验逻辑；
- 想统一操作类属性（如统计对象总数），用实例方法或普通函数不够规范；
- 想在类中定义“工具方法”，不依赖对象和类的属性，无需创建对象就能调用。

本节课我们将学习3个面向对象高级技巧，精准解决以上问题，进一步提升代码的规范性、灵活性和可读性：`@property`装饰器进阶（setter/deleter）、类方法（@classmethod）、静态方法（@staticmethod）。三者各司其职，是编写高质量面向对象代码的必备技能，也是面试高频考点。

本节核心学习内容：
- `@property`进阶：setter装饰器（修改私有属性）、deleter装饰器（删除私有属性）；
- 类方法：@classmethod装饰器、cls参数、作用场景（操作类属性）；
- 静态方法：@staticmethod装饰器、作用场景（工具方法）；
- 核心区分：实例方法、类方法、静态方法的用法与区别；
- 实战案例：整合三大技巧，改造学生管理系统，提升代码规范性；
- 新手避坑：三类方法调用误区、@property进阶用法错误。

# 一、@property进阶：setter与deleter，实现属性的可控修改与删除
在进阶篇2中，我们学习了`@property`的**基础只读用法**，将私有属性伪装成“只读属性”，通过“对象.属性名”优雅访问，禁止直接修改。但实际开发中，我们常常需要**合法修改/删除私有属性**（如修改学生年龄、删除银行卡绑定信息），此时就需要用到`@property`的进阶用法：`@属性名.setter`（修改）和`@属性名.deleter`（删除）。

## 1. 回顾：@property基础只读用法（痛点）
```python
class Student:
    def __init__(self, age):
        self.__age = age  # 私有属性

    @property
    def age(self):
        """只读属性：获取年龄"""
        return self.__age

stu = Student(20)
print(stu.age)  # 20，正常访问
# stu.age = 21  # 报错，只读属性无法直接修改
# del stu.age   # 报错，无法直接删除
```
**痛点**：只读属性无法合法修改/删除，若直接取消私有属性，又会失去数据校验和封装的安全性。

## 2. @property.setter：实现私有属性的可控修改
`@属性名.setter`与基础`@property`配合使用，为“只读属性”添加**修改接口**，同时在setter方法中添加校验逻辑，保证修改的数据合法。

### 语法格式
```python
class 类名:
    def __init__(self):
        self.__私有属性 = 值

    # 基础：获取属性（必须先定义）
    @property
    def 属性名(self):
        return self.__私有属性

    # 进阶：修改属性（装饰器格式固定：@属性名.setter）
    @属性名.setter
    def 属性名(self, 新值):
        # 可选：添加数据校验逻辑
        if 校验条件:
            self.__私有属性 = 新值
        else:
            print("数据不合法，修改失败")
```

### 核心注意事项
1.  必须先定义`@property`（获取方法），才能定义`@属性名.setter`（修改方法），顺序不能颠倒；
2.  两个方法的**名字必须完全一致**（均为属性名，如age）；
3.  setter方法有两个参数：`self`（对象本身）和`新值`（要修改的值）；
4.  修改属性时，直接用`对象.属性名 = 新值`，无需调用方法，兼顾优雅与安全。

### 示例：用setter修改学生年龄（带校验）
```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.__age = age  # 私有属性

    # 获取年龄（基础@property）
    @property
    def age(self):
        return self.__age

    # 修改年龄（@property.setter）
    @age.setter
    def age(self, new_age):
        # 校验：年龄必须是6-25的整数
        if isinstance(new_age, int) and 6 <= new_age <= 25:
            self.__age = new_age
            print(f"年龄修改成功，当前年龄：{self.__age}")
        else:
            print(f"年龄{new_age}不合法，必须是6-25的整数")

# 测试
stu = Student("张三", 20)
print("初始年龄：", stu.age)  # 初始年龄：20（调用@property）

# 修改年龄（调用@age.setter）
stu.age = 21  # 年龄修改成功，当前年龄：21
stu.age = 30  # 年龄30不合法，必须是6-25的整数
stu.age = "22"  # 年龄22不合法，必须是6-25的整数

print("最终年龄：", stu.age)  # 最终年龄：21
```

## 3. @property.deleter：实现私有属性的可控删除
`@属性名.deleter`用于为私有属性添加**删除接口**，可以在删除时添加校验逻辑（如禁止删除核心属性），避免误删除。

### 语法格式
```python
# 必须在@property和@属性名.setter之后定义（若有setter）
@属性名.deleter
def 属性名(self):
    # 可选：添加删除校验逻辑
    print("执行删除操作")
    del self.__私有属性
```

### 示例：用deleter删除学生年龄（带校验）
```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.__age = age

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, new_age):
        if isinstance(new_age, int) and 6 <= new_age <= 25:
            self.__age = new_age
        else:
            print("年龄不合法")

    # 删除年龄（@property.deleter）
    @age.deleter
    def age(self):
        # 校验：禁止删除年龄（核心属性）
        if hasattr(self, "__age"):
            print("警告：年龄是核心属性，禁止删除！")
            # del self.__age  # 注释掉，禁止删除

# 测试
stu = Student("张三", 20)
# 尝试删除年龄（调用@age.deleter）
del stu.age  # 警告：年龄是核心属性，禁止删除！

# 若允许删除，取消注释del self.__age，删除后无法再直接访问
# del stu.age → 执行删除 → print(stu.age) 报错（属性已删除）
```

## 4. @property进阶总结（新手必记）
| 装饰器          | 作用                     | 语法要点                           | 调用方式           |
| --------------- | ------------------------ | ---------------------------------- | ------------------ |
| @property       | 获取私有属性（只读基础） | 无额外参数，必须有返回值           | 对象.属性名        |
| @属性名.setter  | 修改私有属性（可控）     | 有new_value参数，需在@property之后 | 对象.属性名 = 新值 |
| @属性名.deleter | 删除私有属性（可控）     | 无额外参数，需在setter之后（若有） | del 对象.属性名    |

**核心价值**：用“属性操作”的优雅语法，实现“方法级别的校验逻辑”，兼顾简洁与安全，比手动写get/set/delete方法更规范。

# 二、类方法：@classmethod，操作类属性的专属方法
在之前的学习中，我们修改类属性时，直接用`类名.类属性名 = 新值`，但在复杂场景中，类属性的修改需要校验、日志记录等逻辑，此时就需要**类方法**——专门用于操作类属性的方法，不依赖对象，更规范、可维护。

## 1. 为什么需要类方法？
```python
class Car:
    total_cars = 0  # 类属性：汽车总数

# 直接修改类属性（无校验，不规范）
Car.total_cars = -10  # 负数无效，但能修改成功
print(Car.total_cars)  # -10（数据异常）
```
**痛点**：直接修改类属性无校验逻辑，易出现数据异常；若用实例方法修改，又需要创建对象，冗余且不合理。

**类方法的作用**：封装类属性的操作逻辑，无需创建对象就能调用，专门处理与“类”相关的事务（而非对象相关）。

## 2. 类方法基础语法
```python
class 类名:
    类属性 = 值

    # 类方法：@classmethod装饰器，第一个参数是cls
    @classmethod
    def 类方法名(cls, 自定义参数):
        # cls 等价于 “类名”，用于操作类属性
        cls.类属性 = 新值  # 修改类属性
        print(cls.类属性)  # 访问类属性

# 调用类方法：无需创建对象，直接用 类名.类方法名()
类名.类方法名(参数)
```

### 核心注意事项
1.  类方法必须用`@classmethod`装饰器标记，否则就是普通实例方法；
2.  类方法的**第一个参数固定为cls**（代表当前类），类似实例方法的self，但self指向对象，cls指向类；
3.  类方法中**不能访问实例属性**（self.属性），只能访问/修改类属性（cls.属性）；
4.  调用方式：优先用`类名.类方法名()`，也可以用`对象.类方法名()`（不推荐，不符合设计初衷）。

### 示例：用类方法修改汽车总数（带校验）
```python
class Car:
    total_cars = 0  # 类属性：汽车总数

    # 类方法：修改汽车总数（带校验）
    @classmethod
    def set_total_cars(cls, count):
        if isinstance(count, int) and count >= 0:
            cls.total_cars = count
            print(f"汽车总数修改成功，当前总数：{cls.total_cars}")
        else:
            print(f"数量{count}不合法，必须是非负整数")

    # 类方法：获取汽车总数
    @classmethod
    def get_total_cars(cls):
        return cls.total_cars

# 调用类方法：无需创建对象
Car.set_total_cars(10)  # 汽车总数修改成功，当前总数：10
print(Car.get_total_cars())  # 10

# 尝试修改为负数（校验失败）
Car.set_total_cars(-5)  # 数量-5不合法，必须是非负整数

# 也可以用对象调用（不推荐）
car = Car()
car.set_total_cars(15)  # 能执行，但不符合类方法的设计初衷
```

## 3. 类方法的常见场景
1.  操作类属性（修改、查询、重置），添加校验逻辑；
2.  创建类的实例对象（替代构造函数，如简化对象创建）；
3.  统计类的实例对象数量（结合`__init__`）。

### 示例：用类方法统计学生数量
```python
class Student:
    total_students = 0  # 类属性：学生总数

    def __init__(self, name):
        self.name = name
        # 每次创建对象，总数加1（调用类方法）
        Student.add_student()

    # 类方法：增加学生数量
    @classmethod
    def add_student(cls):
        cls.total_students += 1

    # 类方法：重置学生数量
    @classmethod
    def reset_students(cls):
        cls.total_students = 0

# 测试
stu1 = Student("张三")
stu2 = Student("李四")
print(Student.total_students)  # 2

Student.reset_students()
print(Student.total_students)  # 0
```

# 三、静态方法：@staticmethod，独立于类和对象的工具方法
静态方法是类中的“工具方法”，**不依赖于类（cls）和对象（self）**，既不访问类属性，也不访问实例属性，仅实现一个独立的功能（如数据校验、格式转换），相当于“放在类里面的普通函数”。

## 1. 为什么需要静态方法？
```python
# 普通工具函数（独立于类，不够规范）
def is_valid_score(score):
    return isinstance(score, (int, float)) and 0 <= score <= 100

class Student:
    def __init__(self, name, score):
        if is_valid_score(score):
            self.score = score
        else:
            self.score = 0
```
**痛点**：工具函数与类相关（如成绩校验仅用于Student类），但独立于类之外，代码组织不规范，不易维护。

**静态方法的作用**：将与类相关的工具函数“封装到类中”，使代码更整洁、模块化，同时不依赖类和对象，调用灵活。

## 2. 静态方法基础语法
```python
class 类名:
    # 静态方法：@staticmethod装饰器，无固定第一个参数
    @staticmethod
    def 静态方法名(自定义参数):
        # 不访问类属性（cls.属性），不访问实例属性（self.属性）
        工具逻辑代码
        return 结果

# 调用静态方法：无需创建对象，直接用 类名.静态方法名()
类名.静态方法名(参数)
# 也可以用 对象.静态方法名()（不推荐）
```

### 核心注意事项
1.  静态方法必须用`@staticmethod`装饰器标记；
2.  静态方法**没有固定的第一个参数**（无self、无cls）；
3.  静态方法中**不能访问类属性和实例属性**，仅能使用传入的参数和自身定义的局部变量；
4.  调用方式：优先用`类名.静态方法名()`，与类方法一致，无需创建对象。

### 示例：用静态方法校验学生成绩
```python
class Student:
    def __init__(self, name, score):
        # 调用类中的静态方法，校验成绩
        if Student.is_valid_score(score):
            self.score = score
        else:
            self.score = 0
            print("成绩不合法，默认设为0分")

    # 静态方法：校验成绩（工具方法，不依赖类和对象）
    @staticmethod
    def is_valid_score(score):
        return isinstance(score, (int, float)) and 0 <= score <= 100

# 调用静态方法：无需创建对象
print(Student.is_valid_score(90))  # True
print(Student.is_valid_score(105))  # False

# 创建对象（内部调用静态方法）
stu1 = Student("张三", 95)
stu2 = Student("李四", 110)  # 成绩不合法，默认设为0分
print(stu1.score, stu2.score)  # 95 0
```

## 3. 静态方法的常见场景
1.  实现与类相关的工具功能（如数据校验、格式转换、计算）；
2.  功能独立，不依赖类和对象的任何属性；
3.  代码组织：将零散的工具函数封装到对应类中，提升代码整洁度。

### 示例：静态方法实现简单计算
```python
class MathTools:
    # 静态方法：计算两个数的和
    @staticmethod
    def add(a, b):
        return a + b

    # 静态方法：计算两个数的差
    @staticmethod
    def subtract(a, b):
        return a - b

# 调用静态方法，无需创建对象
print(MathTools.add(10, 5))  # 15
print(MathTools.subtract(10, 5))  # 5
```

# 四、核心区分：实例方法、类方法、静态方法（新手必背）
这是新手最容易混淆的知识点，也是面试高频考点，我们通过“定义、参数、作用、调用方式”四大维度，用表格和示例彻底区分三者。

## 1. 三者核心对比表
| 对比维度   | 实例方法                         | 类方法（@classmethod）     | 静态方法（@staticmethod） |
| ---------- | -------------------------------- | -------------------------- | ------------------------- |
| 装饰器     | 无                               | @classmethod               | @staticmethod             |
| 第一个参数 | self（指向当前对象）             | cls（指向当前类）          | 无固定参数                |
| 可访问属性 | 类属性、实例属性                 | 仅类属性                   | 无（仅用传入参数）        |
| 核心作用   | 操作实例属性，实现对象的行为     | 操作类属性，处理类相关事务 | 工具方法，独立功能        |
| 调用方式   | 对象.方法名()（推荐）            | 类名.方法名()（推荐）      | 类名.方法名()（推荐）     |
| 依赖关系   | 依赖对象（必须创建对象才能调用） | 依赖类（无需创建对象）     | 不依赖类和对象            |

## 2. 直观示例：同一个类中的三种方法
```python
class Person:
    species = "人类"  # 类属性

    # 1. 实例方法：操作实例属性
    def __init__(self, name):
        self.name = name  # 实例属性

    def show_name(self):  # 实例方法
        print(f"姓名：{self.name}，物种：{self.species}")  # 访问两者

    # 2. 类方法：操作类属性
    @classmethod
    def change_species(cls, new_species):
        cls.species = new_species
        print(f"物种修改为：{cls.species}")

    # 3. 静态方法：工具方法
    @staticmethod
    def is_adult(age):
        return age >= 18

# 调用实例方法（必须创建对象）
p = Person("张三")
p.show_name()  # 姓名：张三，物种：人类

# 调用类方法（无需创建对象）
Person.change_species("智人")  # 物种修改为：智人
p.show_name()  # 姓名：张三，物种：智人（类属性已修改）

# 调用静态方法（无需创建对象）
print(Person.is_adult(20))  # True
print(Person.is_adult(17))  # False
```

## 3. 新手判断技巧（快速选型）
1.  若需要操作**实例属性**（self.属性），用「实例方法」；
2.  若需要操作**类属性**（cls.属性），用「类方法」；
3.  若既不操作实例属性，也不操作类属性，仅实现独立工具功能，用「静态方法」。

# 五、实战案例：整合三大技巧，改造学生管理系统
结合本节课所有知识点，改造之前的Student类，整合`@property`（setter/deleter）、类方法、静态方法，实现更规范、灵活的学生管理，贴合真实开发场景。

## 需求
1.  私有属性：`__name`（姓名）、`__age`（年龄）、`__score`（成绩）；
2.  `@property`进阶：
    - 用`@property`获取三个私有属性；
    - 用`@age.setter`、`@score.setter`修改年龄、成绩（带校验）；
    - 用`@name.deleter`禁止删除姓名；
3.  类方法：
    - 统计学生总数（`total_students`）；
    - 重置学生总数；
4.  静态方法：
    - 校验成绩合法性（0-100）；
    - 校验年龄合法性（6-25）；
5.  实例方法：展示学生完整信息。

## 完整代码
```python
class Student:
    # 类属性：学生总数
    total_students = 0

    def __init__(self, name, age, score):
        # 调用静态方法校验数据
        if not Student.is_valid_age(age):
            self.__age = 0
            print("年龄不合法，默认设为0")
        else:
            self.__age = age

        if not Student.is_valid_score(score):
            self.__score = 0
            print("成绩不合法，默认设为0")
        else:
            self.__score = score

        self.__name = name
        # 调用类方法，增加学生总数
        Student.add_student()

    # ------------------------------
    # @property 及其进阶（setter/deleter）
    # ------------------------------
    @property
    def name(self):
        return self.__name

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, new_age):
        if Student.is_valid_age(new_age):
            self.__age = new_age
            print(f"年龄修改成功：{self.__age}")
        else:
            print("年龄不合法，修改失败")

    @property
    def score(self):
        return self.__score

    @score.setter
    def score(self, new_score):
        if Student.is_valid_score(new_score):
            self.__score = new_score
            print(f"成绩修改成功：{self.__score}")
        else:
            print("成绩不合法，修改失败")

    @name.deleter
    def name(self):
        print("警告：姓名是核心属性，禁止删除！")

    # ------------------------------
    # 类方法（操作类属性）
    # ------------------------------
    @classmethod
    def add_student(cls):
        cls.total_students += 1

    @classmethod
    def reset_total(cls):
        cls.total_students = 0

    @classmethod
    def get_total(cls):
        return cls.total_students

    # ------------------------------
    # 静态方法（工具方法）
    # ------------------------------
    @staticmethod
    def is_valid_age(age):
        return isinstance(age, int) and 6 <= age <= 25

    @staticmethod
    def is_valid_score(score):
        return isinstance(score, (int, float)) and 0 <= score <= 100

    # ------------------------------
    # 实例方法（展示信息）
    # ------------------------------
    def show_info(self):
        print(f"姓名：{self.name}，年龄：{self.age}，成绩：{self.score}")

# 实战测试
if __name__ == "__main__":
    # 创建学生对象（内部调用静态方法校验、类方法统计）
    print("=== 创建学生 ===")
    stu1 = Student("张三", 20, 92)
    stu2 = Student("李四", 17, 105)  # 成绩不合法，默认设为0
    stu3 = Student("王五", 26, 88)  # 年龄不合法，默认设为0

    # 展示学生信息（实例方法）
    print("\n=== 学生信息 ===")
    stu1.show_info()
    stu2.show_info()
    stu3.show_info()

    # 修改学生信息（@property.setter）
    print("\n=== 修改信息 ===")
    stu1.age = 21  # 年龄修改成功：21
    stu1.score = 95  # 成绩修改成功：95
    stu2.age = 30  # 年龄不合法，修改失败

    # 尝试删除姓名（@name.deleter）
    print("\n=== 尝试删除姓名 ===")
    del stu1.name  # 警告：姓名是核心属性，禁止删除！

    # 类方法统计、重置学生总数
    print("\n=== 学生统计 ===")
    print(f"当前学生总数：{Student.get_total()}")  # 3
    Student.reset_total()
    print(f"重置后学生总数：{Student.get_total()}")  # 0

    # 静态方法独立调用
    print("\n=== 静态方法测试 ===")
    print(Student.is_valid_age(18))  # True
    print(Student.is_valid_score(99.5))  # True
```

## 运行结果
```
=== 创建学生 ===
成绩不合法，默认设为0
年龄不合法，默认设为0

=== 学生信息 ===
姓名：张三，年龄：20，成绩：92
姓名：李四，年龄：17，成绩：0
姓名：王五，年龄：0，成绩：88

=== 修改信息 ===
年龄修改成功：21
成绩修改成功：95
年龄不合法，修改失败

=== 尝试删除姓名 ===
警告：姓名是核心属性，禁止删除！

=== 学生统计 ===
当前学生总数：3
重置后学生总数：0

=== 静态方法测试 ===
True
True
```

# 六、新手避坑大全：三大技巧高频错误
## 避坑1：@property.setter 定义在 @property 之前
```python
class Student:
    def __init__(self, age):
        self.__age = age

    # 错误：setter在property之前
    @age.setter
    def age(self, new_age):
        self.__age = new_age

    @property
    def age(self):
        return self.__age
```
**报错原因**：setter依赖property，必须先定义“获取方法”，才能定义“修改方法”。
**解决方案**：严格按「@property → @属性名.setter → @属性名.deleter」的顺序定义。

## 避坑2：类方法用self参数，静态方法用self/cls参数
```python
class Student:
    @classmethod
    def test_cls(self):  # 错误：类方法第一个参数应为cls
        pass

    @staticmethod
    def test_static(cls):  # 错误：静态方法无固定参数
        pass
```
**报错原因**：类方法固定第一个参数为cls，静态方法无固定参数，混淆了参数规则。
**解决方案**：类方法用cls，静态方法无固定参数，实例方法用self。

## 避坑3：静态方法/类方法访问实例属性
```python
class Student:
    def __init__(self, name):
        self.name = name

    @classmethod
    def show_cls(cls):
        print(cls.name)  # 错误：类方法不能访问实例属性

    @staticmethod
    def show_static(self):
        print(self.name)  # 错误：静态方法不能访问实例属性
```
**报错原因**：类方法仅能访问类属性，静态方法不访问任何属性，实例属性只能通过self访问。
**解决方案**：访问实例属性用实例方法，类属性用类方法，静态方法仅用传入参数。

## 避坑4：修改类属性不用类方法，直接赋值无校验
```python
class Car:
    total_cars = 0

# 错误：直接修改无校验，易出现异常
Car.total_cars = -5
```
**解决方案**：所有类属性的修改的逻辑，都封装到类方法中，通过类方法调用。

## 避坑5：滥用静态方法/类方法
将需要访问实例属性的逻辑，写成静态方法/类方法，导致无法正常获取属性，或代码逻辑混乱。
**解决方案**：按“是否访问属性”选型，严格遵循三者的使用场景。

# 七、核心总结
本节课我们学习了3个面向对象高级技巧，解决了“属性可控修改、类属性操作、工具方法封装”的核心需求，核心要点回顾：
1.  `@property`进阶：
    - 基础用法：只读访问私有属性，语法`@property`+无参方法；
    - 进阶用法：`@属性名.setter`（修改私有属性，带校验）、`@属性名.deleter`（删除私有属性，带校验）；
    - 核心：用属性操作的优雅语法，实现方法级别的安全控制。

2.  类方法（@classmethod）：
    - 语法：`@classmethod`+第一个参数cls（指向类）；
    - 作用：专门操作类属性，处理与类相关的事务，无需创建对象；
    - 场景：类属性修改/统计、创建实例、类级别的工具逻辑。

3.  静态方法（@staticmethod）：
    - 语法：`@staticmethod`+无固定参数方法；
    - 作用：封装与类相关的独立工具方法，不依赖类和对象；
    - 场景：数据校验、格式转换、独立计算等无属性依赖的功能。

4.  三者核心区别（新手必记）：
    - 实例方法：依赖对象，访问实例属性+类属性，用self；
    - 类方法：依赖类，仅访问类属性，用cls；
    - 静态方法：不依赖任何，无固定参数，仅实现工具功能。

5.  最佳实践：
    - 所有私有属性的获取/修改/删除，优先用`@property`及其进阶；
    - 所有类属性的操作，封装到类方法中；
    - 所有独立工具方法，封装到静态方法中，避免零散函数；
    - 不滥用三者，严格按使用场景选型，保持代码逻辑清晰。

掌握这些技巧后，你的面向对象代码将更规范、更灵活、更具可维护性，能够应对更复杂的开发场景。下一节，我们将学习**面向对象高级特性——抽象类与接口**（简化版，贴合新手），进一步提升代码的规范性和扩展性，为后续框架学习打下基础。

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/159505174>
