# Python全栈入门到实战【进阶篇 8】面向对象高级补充：枚举类（Enum）——告别魔法数字与硬编码
在上一节中，我们完成了「小型学生管理系统V2.0」的实战开发，整合了封装、继承、多态等所有核心知识点，但在实战中你可能会发现一个细节问题：
- 表示学生状态（如“正常、休学、毕业”）时，用数字`1/2/3`或字符串`"normal"/"suspend"/"graduate"`硬编码；
- 表示成绩等级（如“优秀、良好、及格、不及格”）时，用`90+/80-89/60-79/<60`的区间判断，代码里充斥着零散的数值；
- 这些“魔法数字/字符串”不仅可读性差，还容易因手误（如把`"normal"`写成`"nomal"`）导致bug，且无法限制取值范围。

本节课我们将学习Python中的**枚举类（Enum）** ——一种专门用于定义“固定取值集合”的工具，解决上述痛点，让代码更规范、更易维护。枚举类是面向对象编程中“常量定义”的最佳实践，也是面试中高频提及的基础高级特性，本节课将用新手能看懂的方式，从基础到实战，彻底掌握枚举类的用法。

本节核心学习内容：
- 为什么需要枚举类？拆解魔法数字/字符串的3大痛点；
- 枚举类基础：定义、访问、核心特性（不可变、唯一、迭代）；
- 枚举类进阶：自定义枚举值、枚举方法、枚举比较、IntEnum（整数枚举）；
- 实战整合：将枚举类融入学生管理系统V2.1，优化状态/等级管理；
- 新手避坑：枚举实例化、修改枚举值、重复枚举成员等高频错误；
- 核心对比：枚举类 vs 普通常量 vs 字典，明确适用场景。

# 一、先搞懂：为什么需要枚举类？拆解硬编码的痛点
我们先通过学生管理系统中的真实场景，直观感受“魔法数字/字符串”的问题，再理解枚举类的核心价值。

## 场景：用硬编码表示学生状态，易出错、可读性差
### 反面案例：魔法数字/字符串硬编码
```python
# 1. 魔法数字：1=正常，2=休学，3=毕业（新人接手需查注释，易记错）
def check_student_status(stu_status):
    if stu_status == 1:
        print("学生状态：正常")
    elif stu_status == 2:
        print("学生状态：休学")
    elif stu_status == 3:
        print("学生状态：毕业")
    else:
        print("状态非法")

# 调用时，易手误写数字（如把1写成0）
check_student_status(0)  # 状态非法（无意义的错误）

# 2. 字符串硬编码：易拼错，无法限制取值范围
def set_student_grade(stu_grade):
    if stu_grade == "excellent":
        print("成绩等级：优秀")
    elif stu_grade == "good":  # 手误写成"god"就会出错
        print("成绩等级：良好")
    elif stu_grade == "pass":
        print("成绩等级：及格")
    elif stu_grade == "fail":
        print("成绩等级：不及格")

# 手误导致逻辑错误（无报错，隐藏bug）
set_student_grade("god")  # 无输出，逻辑异常
```

### 硬编码的3大核心痛点
1.  **可读性差**：新人接手代码时，需反复查注释才能知道`1/2/3`对应什么状态，降低开发效率；
2.  **易出错**：数字/字符串手误（如`3`写成`4`、`"excellent"`写成`"excelent"`），编译器无法检测，只会运行时出错；
3.  **无约束**：无法限制取值范围，比如学生状态只能是“正常/休学/毕业”，但硬编码允许传入任意数字/字符串，导致非法值混入。

### 枚举类的解决思路
1.  定义枚举类`StudentStatus`，明确规定状态只能是`NORMAL、SUSPEND、GRADUATE`，且对应固定值；
2.  定义枚举类`GradeLevel`，明确成绩等级的取值范围和判断逻辑；
3.  代码中直接使用枚举成员（如`StudentStatus.NORMAL`），无需记忆数字/字符串，编译器可检测错误，提升可读性和健壮性。

# 二、枚举类基础：定义、访问与核心特性
Python中的枚举类需要借助标准库`enum`模块中的`Enum`类，无需安装第三方库，是Python内置的基础工具，核心语法简单，新手易上手。

## 1. 枚举类的基础定义
```python
# 1. 导入Enum类（固定写法）
from enum import Enum

# 2. 定义枚举类：继承Enum，成员名大写（规范），值可自定义
class 枚举类名(Enum):
    # key = value
    枚举成员1 = 值1
    枚举成员2 = 值2
    枚举成员3 = 值3

# 示例：定义学生状态枚举类
class StudentStatus(Enum):
    NORMAL = 1       # 正常
    SUSPEND = 2      # 休学
    GRADUATE = 3     # 毕业
```

### 核心规范
- 枚举类**必须继承`Enum`**，否则只是普通类；
- 枚举成员名**建议大写**（符合Python常量命名规范）；
- 枚举成员的值可以是数字、字符串、甚至对象（但通常用数字/字符串）；
- 枚举成员**不可重复**（默认），值可重复（需特殊设置，不推荐）。

## 2. 枚举类的核心访问方式（3种常用）
```python
from enum import Enum

class StudentStatus(Enum):
    NORMAL = 1
    SUSPEND = 2
    GRADUATE = 3

# 方式1：通过成员名访问（最常用）
print(StudentStatus.NORMAL)          # StudentStatus.NORMAL
print(StudentStatus.NORMAL.name)     # 成员名：NORMAL
print(StudentStatus.NORMAL.value)    # 成员值：1

# 方式2：通过值访问（反向查找）
print(StudentStatus(1))              # StudentStatus.NORMAL

# 方式3：遍历所有枚举成员（迭代）
for status in StudentStatus:
    print(f"成员名：{status.name}，成员值：{status.value}")
```

### 运行结果
```
StudentStatus.NORMAL
NORMAL
1
StudentStatus.NORMAL
成员名：NORMAL，成员值：1
成员名：SUSPEND，成员值：2
成员名：GRADUATE，成员值：3
```

## 3. 枚举类的核心特性
1. **不可变**：枚举成员的值无法修改（避免运行时篡改），修改会报错；
    ```python
    # 错误：无法修改枚举成员的值
    StudentStatus.NORMAL.value = 0  # 报错：AttributeError: can't set attribute
    ```
2. **唯一性**：枚举成员名不可重复（重复定义会覆盖前一个）；
    ```python
    # 错误：成员名重复（第二个NORMAL会覆盖第一个）
    class StudentStatus(Enum):
        NORMAL = 1
        NORMAL = 2  # 警告：Duplicate member name 'NORMAL'
    ```
3. **类型安全**：枚举成员是独立的类型，与普通数字/字符串不相等（除非显式比较value）；
    ```python
    print(StudentStatus.NORMAL == 1)          # False（类型不同）
    print(StudentStatus.NORMAL.value == 1)    # True（比较值）
    print(StudentStatus.NORMAL is StudentStatus(1))  # True（同一枚举成员）
    ```
4. **可哈希**：可作为字典的键、集合的元素（普通类实例默认也可，但枚举更规范）。

## 4. 基础示例：用枚举类优化学生状态判断
```python
from enum import Enum

class StudentStatus(Enum):
    NORMAL = 1       # 正常
    SUSPEND = 2      # 休学
    GRADUATE = 3     # 毕业

# 优化后的状态检查函数（无魔法数字，可读性高）
def check_student_status(stu_status):
    # 先校验是否为合法枚举成员
    if not isinstance(stu_status, StudentStatus):
        print("状态非法：必须传入StudentStatus枚举成员")
        return
    # 枚举成员判断（直观，无手误风险）
    if stu_status == StudentStatus.NORMAL:
        print("学生状态：正常")
    elif stu_status == StudentStatus.SUSPEND:
        print("学生状态：休学")
    elif stu_status == StudentStatus.GRADUATE:
        print("学生状态：毕业")

# 正确调用（传入枚举成员）
check_student_status(StudentStatus.NORMAL)  # 学生状态：正常

# 错误调用（传入数字，会提示非法）
check_student_status(1)  # 状态非法：必须传入StudentStatus枚举成员

# 正确的反向调用（先转枚举成员）
status = StudentStatus(1)
check_student_status(status)  # 学生状态：正常
```

### 运行结果
```
学生状态：正常
状态非法：必须传入StudentStatus枚举成员
学生状态：正常
```

# 三、枚举类进阶：自定义值、方法与特殊枚举
基础用法只能满足简单场景，实战中我们需要自定义枚举值、添加枚举方法、使用整数枚举（IntEnum）等进阶技巧，进一步提升枚举类的实用性。

## 1. 自定义枚举值（字符串/元组）
枚举值不仅可以是数字，还可以是字符串、元组等，适合存储更丰富的信息，比如成绩等级的“名称+分数区间”。

```python
from enum import Enum

# 枚举值为元组：(等级名称, 最低分, 最高分)
class GradeLevel(Enum):
    EXCELLENT = ("优秀", 90, 100)
    GOOD = ("良好", 80, 89)
    PASS = ("及格", 60, 79)
    FAIL = ("不及格", 0, 59)

# 访问元组值（通过索引）
print(GradeLevel.EXCELLENT.value[0])  # 优秀
print(GradeLevel.EXCELLENT.value[1])  # 90
print(GradeLevel.EXCELLENT.value[2])  # 100

# 封装方法：根据分数获取等级
class GradeLevel(Enum):
    EXCELLENT = ("优秀", 90, 100)
    GOOD = ("良好", 80, 89)
    PASS = ("及格", 60, 79)
    FAIL = ("不及格", 0, 59)

    # 枚举类的实例方法（第一个参数是枚举成员自身）
    @classmethod
    def get_grade_by_score(cls, score):
        """根据分数返回对应的成绩等级枚举成员"""
        if not isinstance(score, (int, float)):
            return None
        # 遍历所有枚举成员，判断分数区间
        for grade in cls:
            min_score = grade.value[1]
            max_score = grade.value[2]
            if min_score <= score <= max_score:
                return grade
        return None

# 测试：根据分数获取等级
score = 95
grade = GradeLevel.get_grade_by_score(score)
if grade:
    print(f"分数{score}对应的等级：{grade.value[0]}")  # 分数95对应的等级：优秀

score = 75
grade = GradeLevel.get_grade_by_score(score)
print(f"分数{score}对应的等级：{grade.value[0]}")  # 分数75对应的等级：及格
```

## 2. 枚举类的方法（实例方法/类方法）
枚举类可以像普通类一样定义实例方法、类方法，用于封装与枚举相关的逻辑，比如上面的`get_grade_by_score`类方法，就是实战中最常用的技巧。

### 核心注意
- 枚举类的**实例方法**：第一个参数是枚举成员自身（类似普通类的self）；
- 枚举类的**类方法**：第一个参数是枚举类（cls），用于封装通用逻辑（如根据条件查找枚举成员）。

## 3. IntEnum：整数枚举（与数字可比较）
普通`Enum`的成员与数字不相等（`StudentStatus.NORMAL == 1` → False），而`IntEnum`是专门的整数枚举类，成员可与整数直接比较，适合需要兼容旧代码（魔法数字）的场景。

```python
from enum import IntEnum

# 继承IntEnum，而非Enum
class StudentStatus(IntEnum):
    NORMAL = 1
    SUSPEND = 2
    GRADUATE = 3

# IntEnum成员可与整数直接比较
print(StudentStatus.NORMAL == 1)  # True（普通Enum是False）
print(StudentStatus.SUSPEND > 1)  # True
print(StudentStatus.GRADUATE < 4) # True
```

### 适用场景
- 旧代码中大量使用魔法数字，逐步替换为枚举类时，用`IntEnum`兼容原有比较逻辑；
- 需要将枚举成员作为整数参与计算（如状态码判断）。

## 4. unique装饰器：强制枚举成员值唯一
默认情况下，枚举类的成员值可以重复（成员名不可重复），用`unique`装饰器可强制值唯一，重复会报错，避免逻辑混乱。

```python
from enum import Enum, unique

# 装饰器：强制值唯一
@unique
class StudentStatus(Enum):
    NORMAL = 1
    SUSPEND = 2
    # 错误：值重复（1已被NORMAL使用）
    GRADUATE = 1  # 报错：ValueError: duplicate values found in <enum 'StudentStatus'>: GRADUATE -> NORMAL
```

### 核心价值
- 避免因值重复导致的反向查找错误（如`StudentStatus(1)`无法确定是NORMAL还是GRADUATE）；
- 提升枚举类的健壮性，适合对值唯一性要求高的场景（如状态码、类型码）。

# 四、实战整合：枚举类融入学生管理系统V2.1
我们将枚举类整合到上一篇的「小型学生管理系统V2.0」中，优化“学生状态管理”和“成绩等级判断”功能，让代码更规范、更易维护，同时保持与原有功能的兼容。

## 需求优化点
1.  为Student类新增`status`属性（学生状态，类型为StudentStatus枚举成员）；
2.  新增“按状态查询学生”功能；
3.  新增“根据成绩自动生成等级”功能；
4.  所有状态/等级相关逻辑，均使用枚举类实现，消除硬编码。

## 完整整合代码（核心修改部分）
```python
# ------------------------------
# 1. 导入必要模块（新增Enum、unique）
# ------------------------------
from abc import ABC, abstractmethod
import os
from enum import Enum, unique

# ------------------------------
# 2. 定义枚举类（新增）
# ------------------------------
# 学生状态枚举（值唯一）
@unique
class StudentStatus(Enum):
    NORMAL = 1       # 正常
    SUSPEND = 2      # 休学
    GRADUATE = 3     # 毕业

    # 类方法：获取状态的中文名称
    @classmethod
    def get_cn_name(cls, status):
        if not isinstance(status, cls):
            return "未知状态"
        cn_names = {
            cls.NORMAL: "正常",
            cls.SUSPEND: "休学",
            cls.GRADUATE: "毕业"
        }
        return cn_names.get(status, "未知状态")

# 成绩等级枚举
class GradeLevel(Enum):
    EXCELLENT = ("优秀", 90, 100)
    GOOD = ("良好", 80, 89)
    PASS = ("及格", 60, 79)
    FAIL = ("不及格", 0, 59)

    # 类方法：根据分数获取等级
    @classmethod
    def get_grade_by_score(cls, score):
        if not isinstance(score, (int, float)):
            return None
        for grade in cls:
            min_s, max_s = grade.value[1], grade.value[2]
            if min_s <= score <= max_s:
                return grade
        return None

# ------------------------------
# 3. 修改Student类（新增status属性）
# ------------------------------
class Person(ABC):
    # 省略原有代码（与上一篇一致）
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    @property
    def name(self):
        return self.__name

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, new_age):
        if self.is_valid_age(new_age):
            self.__age = new_age
            print(f"年龄修改成功，当前年龄：{self.__age}")
        else:
            print("年龄不合法，修改失败（必须是6-25的整数）")

    @abstractmethod
    def show_info(self):
        pass

    @staticmethod
    def is_valid_age(age):
        return isinstance(age, int) and 6 <= age <= 25

class Student(Person):
    STU_ID_PREFIX = "S"

    def __init__(self, name, age, stu_id, score, status=StudentStatus.NORMAL):
        super().__init__(name, age)
        self.__stu_id = stu_id
        if self.is_valid_score(score):
            self.__score = score
        else:
            self.__score = 0
            print(f"成绩{score}不合法，默认设为0分（0-100的整数/小数）")
        # 新增：学生状态（枚举成员）
        if isinstance(status, StudentStatus):
            self.__status = status
        else:
            # 传入数字，自动转为枚举成员
            try:
                self.__status = StudentStatus(status)
            except ValueError:
                self.__status = StudentStatus.NORMAL
                print(f"状态{status}非法，默认设为正常")

    # 新增：status的@property和setter
    @property
    def status(self):
        return self.__status

    @status.setter
    def status(self, new_status):
        if isinstance(new_status, StudentStatus):
            self.__status = new_status
            print(f"状态修改成功：{StudentStatus.get_cn_name(new_status)}")
        else:
            try:
                self.__status = StudentStatus(new_status)
                print(f"状态修改成功：{StudentStatus.get_cn_name(self.__status)}")
            except ValueError:
                print(f"状态{new_status}非法，修改失败")

    # 新增：获取成绩等级
    def get_grade_level(self):
        grade = GradeLevel.get_grade_by_score(self.__score)
        if grade:
            return grade.value[0]
        return "无效成绩"

    # 重写show_info，新增状态和等级
    def show_info(self):
        status_cn = StudentStatus.get_cn_name(self.__status)
        grade_cn = self.get_grade_level()
        print(f"【学生信息】学号：{self.stu_id}，姓名：{self.name}，年龄：{self.age}，成绩：{self.score}，状态：{status_cn}，等级：{grade_cn}")

    # 省略原有静态方法（is_valid_score、is_valid_stu_id）
    @staticmethod
    def is_valid_score(score):
        return isinstance(score, (int, float)) and 0 <= score <= 100

    @staticmethod
    def is_valid_stu_id(stu_id):
        if not isinstance(stu_id, str):
            return False
        return stu_id.startswith(Student.STU_ID_PREFIX) and stu_id[1:].isdigit()

# ------------------------------
# 4. 修改StudentManager类（新增按状态查询）
# ------------------------------
class StudentManager:
    def __init__(self):
        self.students = []

    # 新增：按状态查询学生
    def query_by_status(self):
        print("\n===== 按状态查询学生 =====")
        if not self.students:
            print("暂无学生信息！")
            return
        # 打印状态选项（遍历枚举）
        print("状态选项：")
        for status in StudentStatus:
            print(f"{status.value}. {StudentStatus.get_cn_name(status)}")
        # 输入状态值，转为枚举成员
        while True:
            status_input = input("请输入要查询的状态编号：")
            if status_input.isdigit():
                status_num = int(status_input)
                try:
                    target_status = StudentStatus(status_num)
                    break
                except ValueError:
                    print("状态编号非法，重新输入！")
            else:
                print("状态编号必须是数字，重新输入！")
        # 筛选学生
        filtered_students = [stu for stu in self.students if stu.status == target_status]
        if not filtered_students:
            print(f"未找到{StudentStatus.get_cn_name(target_status)}状态的学生！")
            return
        print(f"{StudentStatus.get_cn_name(target_status)}状态的学生（共{len(filtered_students)}人）：")
        print("-" * 60)
        for stu in filtered_students:
            stu.show_info()
            print("-" * 60)

    # 省略原有方法（add_student/delete_student等，仅修改add_student新增状态输入）
    def add_student(self):
        print("\n===== 添加学生 =====")
        name = input("请输入学生姓名：")
        # 年龄输入（原有逻辑）
        while True:
            age_input = input("请输入学生年龄（6-25）：")
            if age_input.isdigit():
                age = int(age_input)
                if Person.is_valid_age(age):
                    break
                else:
                    print("年龄不合法，重新输入！")
            else:
                print("年龄必须是整数，重新输入！")
        # 学号输入（原有逻辑）
        while True:
            stu_id = input("请输入学生学号（格式：S+数字，如S1001）：")
            if Student.is_valid_stu_id(stu_id):
                if self.find_student_by_id(self.students, stu_id):
                    print("学号已存在，重新输入！")
                else:
                    break
            else:
                print("学号格式不合法，重新输入！")
        # 成绩输入（原有逻辑）
        while True:
            score_input = input("请输入学生成绩（0-100）：")
            try:
                score = float(score_input)
                if Student.is_valid_score(score):
                    break
                else:
                    print("成绩不合法，重新输入！")
            except ValueError:
                print("成绩必须是数字，重新输入！")
        # 新增：状态输入
        print("状态选项：")
        for status in StudentStatus:
            print(f"{status.value}. {StudentStatus.get_cn_name(status)}")
        while True:
            status_input = input("请输入学生状态编号（默认1）：") or "1"
            if status_input.isdigit():
                status_num = int(status_input)
                try:
                    status = StudentStatus(status_num)
                    break
                except ValueError:
                    print("状态编号非法，重新输入！")
            else:
                print("状态编号必须是数字，重新输入！")
        # 创建学生对象
        student = Student(name, age, stu_id, score, status)
        self.students.append(student)
        print("学生添加成功！")
        student.show_info()

    # 省略其他原有方法（delete_student/modify_student/query_student/stats_student/save_data/load_data/find_student_by_id等）
    @classmethod
    def get_total_count(cls, students):
        return len(students)

    @classmethod
    def get_score_stats(cls, students):
        if not students:
            return 0.0, 0.0, 0.0
        scores = [stu.score for stu in students]
        avg_score = sum(scores) / len(scores)
        max_score = max(scores)
        min_score = min(scores)
        return round(avg_score, 2), max_score, min_score

    @staticmethod
    def find_student_by_id(students, stu_id):
        for stu in students:
            if stu.stu_id == stu_id:
                return stu
        return None

    def delete_student(self):
        print("\n===== 删除学生 =====")
        if not self.students:
            print("暂无学生信息，无法删除！")
            return
        stu_id = input("请输入要删除的学生学号：")
        student = self.find_student_by_id(self.students, stu_id)
        if student:
            self.students.remove(student)
            print(f"学号{stu_id}的学生删除成功！")
        else:
            print(f"未找到学号{stu_id}的学生，删除失败！")

    def modify_student(self):
        print("\n===== 修改学生信息 =====")
        if not self.students:
            print("暂无学生信息，无法修改！")
            return
        stu_id = input("请输入要修改的学生学号：")
        student = self.find_student_by_id(self.students, stu_id)
        if not student:
            print(f"未找到学号{stu_id}的学生，修改失败！")
            return

        print(f"当前学生信息：")
        student.show_info()
        while True:
            choice = input("请选择要修改的内容（1.姓名 2.年龄 3.成绩 4.状态 5.取消）：")
            if choice == "1":
                new_name = input("请输入新姓名：")
                student._Person__name = new_name
                print("姓名修改成功！")
                break
            elif choice == "2":
                while True:
                    new_age_input = input("请输入新年龄（6-25）：")
                    if new_age_input.isdigit():
                        new_age = int(new_age_input)
                        student.age = new_age
                        break
                    else:
                        print("年龄必须是整数，重新输入！")
                break
            elif choice == "3":
                while True:
                    new_score_input = input("请输入新成绩（0-100）：")
                    try:
                        new_score = float(new_score_input)
                        student.score = new_score
                        break
                    except ValueError:
                        print("成绩必须是数字，重新输入！")
                break
            elif choice == "4":
                # 新增：修改状态
                print("状态选项：")
                for status in StudentStatus:
                    print(f"{status.value}. {StudentStatus.get_cn_name(status)}")
                while True:
                    new_status_input = input("请输入新状态编号：")
                    if new_status_input.isdigit():
                        new_status_num = int(new_status_input)
                        student.status = new_status_num
                        break
                    else:
                        print("状态编号必须是数字，重新输入！")
                break
            elif choice == "5":
                print("取消修改！")
                break
            else:
                print("输入错误，请重新选择！")

    def query_student(self):
        print("\n===== 查询学生 =====")
        if not self.students:
            print("暂无学生信息！")
            return
        choice = input("请选择查询方式（1.按学号查询 2.查询所有学生 3.按状态查询）：")
        if choice == "1":
            stu_id = input("请输入要查询的学生学号：")
            student = self.find_student_by_id(self.students, stu_id)
            if student:
                print("查询到的学生信息：")
                student.show_info()
            else:
                print(f"未找到学号{stu_id}的学生！")
        elif choice == "2":
            print(f"所有学生信息（共{self.get_total_count(self.students)}人）：")
            print("-" * 60)
            for stu in self.students:
                stu.show_info()
                print("-" * 60)
        elif choice == "3":
            self.query_by_status()
        else:
            print("输入错误，查询失败！")

    def stats_student(self):
        print("\n===== 学生统计 =====")
        if not self.students:
            print("暂无学生信息，无法统计！")
            return
        total = self.get_total_count(self.students)
        avg_score, max_score, min_score = self.get_score_stats(self.students)
        print(f"学生总数：{total}人")
        print(f"成绩平均分：{avg_score}分")
        print(f"成绩最高分：{max_score}分")
        print(f"成绩最低分：{min_score}分")
        # 新增：按状态统计
        print("\n按状态统计：")
        for status in StudentStatus:
            count = len([stu for stu in self.students if stu.status == status])
            print(f"{StudentStatus.get_cn_name(status)}：{count}人")

    def save_data(self, file_path="students.txt"):
        try:
            with open(file_path, "w", encoding="utf-8") as f:
                for stu in self.students:
                    # 保存状态值（数字），便于加载
                    line = f"{stu.stu_id},{stu.name},{stu.age},{stu.score},{stu.status.value}\n"
                    f.write(line)
            print(f"\n数据保存成功！文件路径：{os.path.abspath(file_path)}")
        except Exception as e:
            print(f"数据保存失败！错误信息：{str(e)}")

    def load_data(self, file_path="students.txt"):
        try:
            if not os.path.exists(file_path):
                with open(file_path, "w", encoding="utf-8") as f:
                    pass
                print(f"文件{file_path}不存在，已创建空文件！")
                return
            with open(file_path, "r", encoding="utf-8") as f:
                lines = f.readlines()
                for line in lines:
                    line = line.strip()
                    if not line:
                        continue
                    stu_data = line.split(",")
                    if len(stu_data) != 5:
                        print(f"跳过非法数据行：{line}")
                        continue
                    stu_id, name, age, score, status_num = stu_data
                    try:
                        age = int(age)
                        score = float(score)
                        status_num = int(status_num)
                    except ValueError:
                        print(f"数据类型错误，跳过该行：{line}")
                        continue
                    # 加载状态（转为枚举成员）
                    try:
                        status = StudentStatus(status_num)
                    except ValueError:
                        status = StudentStatus.NORMAL
                    student = Student(name, age, stu_id, score, status)
                    self.students.append(student)
            print(f"数据加载成功！共加载{len(self.students)}名学生信息！")
        except Exception as e:
            print(f"数据加载失败！错误信息：{str(e)}")

# ------------------------------
# 5. 主程序（新增按状态查询选项）
# ------------------------------
def main():
    manager = StudentManager()
    manager.load_data()

    while True:
        print("\n" + "="*60)
        print("        小型学生管理系统V2.1（新增枚举类）")
        print("="*60)
        print("  1. 添加学生        2. 删除学生        3. 修改学生信息")
        print("  4. 查询学生        5. 学生统计        6. 保存数据")
        print("  7. 加载数据        8. 退出系统")
        print("="*60)
        choice = input("请输入您的操作（1-8）：")

        try:
            if choice == "1":
                manager.add_student()
            elif choice == "2":
                manager.delete_student()
            elif choice == "3":
                manager.modify_student()
            elif choice == "4":
                manager.query_student()
            elif choice == "5":
                manager.stats_student()
            elif choice == "6":
                manager.save_data()
            elif choice == "7":
                manager.students.clear()
                manager.load_data()
            elif choice == "8":
                manager.save_data()
                print("感谢使用，系统已退出！")
                break
            else:
                print("输入错误，请输入1-8之间的数字！")
        except Exception as e:
            print(f"操作失败！错误信息：{str(e)}，请重新操作！")

if __name__ == "__main__":
    main()
```

## 实战测试：新增功能验证
### 测试1：添加学生时选择状态
```
===== 添加学生 =====
请输入学生姓名：张三
请输入学生年龄（6-25）：20
请输入学生学号（格式：S+数字，如S1001）：S1001
请输入学生成绩（0-100）：95
状态选项：
1. 正常
2. 休学
3. 毕业
请输入学生状态编号（默认1）：1
学生添加成功！
【学生信息】学号：S1001，姓名：张三，年龄：20，成绩：95.0，状态：正常，等级：优秀
```

### 测试2：按状态查询学生
```
===== 查询学生 =====
请选择查询方式（1.按学号查询 2.查询所有学生 3.按状态查询）：3

===== 按状态查询学生 =====
状态选项：
1. 正常
2. 休学
3. 毕业
请输入要查询的状态编号：1
正常状态的学生（共1人）：
------------------------------------------------------------
【学生信息】学号：S1001，姓名：张三，年龄：20，成绩：95.0，状态：正常，等级：优秀
------------------------------------------------------------
```

### 测试3：学生统计（按状态）
```
===== 学生统计 =====
学生总数：1人
成绩平均分：95.0分
成绩最高分：95.0分
成绩最低分：95.0分

按状态统计：
正常：1人
休学：0人
毕业：0人
```

# 五、新手避坑大全：枚举类高频错误
## 避坑1：枚举类未继承Enum，误以为是枚举
```python
# 错误：未继承Enum，只是普通类
class StudentStatus:
    NORMAL = 1
    SUSPEND = 2

# 无法迭代，无法反向查找
print(StudentStatus(1))  # 报错：TypeError: 'type' object is not callable
```
**解决方案**：所有枚举类必须继承`Enum`（或`IntEnum`），语法：`class StudentStatus(Enum):`。

## 避坑2：直接修改枚举成员的值（不可变）
```python
from enum import Enum

class StudentStatus(Enum):
    NORMAL = 1

# 错误：枚举成员值不可修改
StudentStatus.NORMAL.value = 0  # 报错：AttributeError: can't set attribute
```
**解决方案**：枚举成员是不可变的，若需修改状态，应在业务层（如Student类的status.setter）处理，而非直接修改枚举值。

## 避坑3：枚举成员值重复（用unique装饰器避免）
```python
from enum import Enum

# 错误：值重复，反向查找会出错
class StudentStatus(Enum):
    NORMAL = 1
    ACTIVE = 1  # 值与NORMAL重复

print(StudentStatus(1))  # 输出StudentStatus.NORMAL，ACTIVE被隐藏
```
**解决方案**：用`@unique`装饰器强制值唯一，重复会直接报错，避免逻辑混乱。

## 避坑4：混淆Enum与IntEnum的比较逻辑
```python
from enum import Enum, IntEnum

class Status1(Enum):
    NORMAL = 1

class Status2(IntEnum):
    NORMAL = 1

# 错误：普通Enum与数字不相等
print(Status1.NORMAL == 1)  # False
# 正确：IntEnum与数字相等
print(Status2.NORMAL == 1)  # True
```
**解决方案**：根据场景选择枚举类型——需要与数字兼容用`IntEnum`，否则用普通`Enum`（更安全）。

## 避坑5：枚举成员作为参数时，传入数字而非枚举实例
```python
def check_status(status):
    if status == StudentStatus.NORMAL:
        print("正常")

# 错误：传入数字，类型不匹配
check_status(1)  # 无输出
# 正确：传入枚举实例
check_status(StudentStatus(1))  # 正常
```
**解决方案**：函数参数优先接收枚举实例，而非数字/字符串，若需兼容，可在函数内先转为枚举成员。

# 六、核心总结（枚举类核心价值与适用场景）
本节课我们学习了枚举类（Enum）的基础与进阶用法，并整合到学生管理系统中，解决了硬编码的痛点，核心要点回顾：

1. 枚举类的核心价值：
    - 替代“魔法数字/字符串”，提升代码**可读性**（新人无需查注释）；
    - 限制取值范围，避免非法值，提升代码**健壮性**（编译器可检测错误）；
    - 封装与枚举相关的逻辑（如成绩等级判断），提升代码**可维护性**。

2. 枚举类基础用法（新手必记）：
    - 定义：`from enum import Enum` → `class 枚举类名(Enum): 成员 = 值`；
    - 访问：`枚举类.成员`（获取实例）、`.name`（成员名）、`.value`（成员值）；
    - 进阶：`@unique`强制值唯一、`IntEnum`兼容数字、类方法封装业务逻辑。

3. 枚举类适用场景（新手必背）：
    - 定义固定取值集合（如状态、类型、等级、选项）；
    - 替代常量定义（如`NORMAL = 1`），更规范；
    - 业务逻辑中需要限制取值范围的场景（如学生状态、成绩等级）。

4. 核心对比（枚举类 vs 普通常量 vs 字典）：
    | 方式     | 可读性 | 健壮性 | 可维护性 | 适用场景                 |
    | -------- | ------ | ------ | -------- | ------------------------ |
    | 枚举类   | 高     | 高     | 高       | 固定取值集合，需限制范围 |
    | 普通常量 | 中     | 低     | 中       | 简单常量，无范围限制     |
    | 字典     | 中     | 中     | 中       | 动态取值集合，需灵活修改 |

至此，面向对象的核心高级特性已全部讲解完毕（封装、继承、多态、@property、类方法、静态方法、抽象类、接口、枚举类），你已具备编写高质量面向对象代码的能力。下一节，我们将进入「Python模块与包」的学习，解决“代码模块化、复用、发布”的问题，为后续开发大型项目打下基础。

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、Web开发（Django/Flask框架）、数据库（MySQL/ORM）、网络爬虫、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/160102961>
