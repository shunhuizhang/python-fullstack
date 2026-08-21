

# Python全栈入门到实战【数据库篇 13】MySQL约束详解，数据完整性与一致性的核心保障
上一篇《数据库篇 12》中，我们已经掌握了MySQL常用内置函数，学会了在SQL层面直接处理字符串、数值、日期和流程逻辑。本篇作为数据库篇的第十三篇，我们将学习**数据库设计的核心——约束**。约束是作用于表中字段上的规则，用于限制存储在表中的数据，保证数据库中数据的正确性、有效性和完整性，是所有关系型数据库设计的基础。

本文为Python全栈开发者与数据库入门者量身打造，详细讲解非空、唯一、主键、默认、检查、外键六大约束的语法和使用方法，重点拆解外键约束的核心原理和级联操作，每一个知识点都有可直接运行的实战示例，同时整理了常见的设计误区和避坑指南，即使是完全没有数据库设计经验的同学，也能快速掌握约束的使用，设计出规范、健壮的数据库表结构。

本节核心学习内容：
1. 约束概述：概念、目的与六大核心分类
2. 单表约束：非空、唯一、主键、默认、检查约束详解（每个语法后附示例）
3. 实战案例：根据业务需求创建带约束的表结构
4. 外键约束：概念、语法与父子表关系
5. 级联操作：外键的删除与更新行为详解
6. 完整实战：部门表与员工表的外键关联设计
7. 常见误区：约束设计的常见错误与最佳实践
8. 核心总结：MySQL约束语法速查表，方便开发时快速查阅

# 一、约束概述
## 1.1 基本概念
约束是**作用于表中字段上的规则**，用于限制存储在表中的数据。例如：限制用户的年龄不能为负数、限制用户的手机号不能重复、限制每个用户都有唯一的ID等。

## 1.2 约束的目的
保证数据库中数据的**正确性、有效性和完整性**，防止非法数据的插入和修改，从数据库层面保障数据质量。

## 1.3 约束的分类
MySQL提供了6种常用的约束，覆盖了绝大多数数据校验场景：

| 约束                     | 描述                                                     | 关键字        |
| ------------------------ | -------------------------------------------------------- | ------------- |
| 非空约束                 | 限制该字段的数据不能为null                               | `NOT NULL`    |
| 唯一约束                 | 保证该字段的所有数据都是唯一、不重复的                   | `UNIQUE`      |
| 主键约束                 | 主键是一行数据的唯一标识，要求非空且唯一                 | `PRIMARY KEY` |
| 默认约束                 | 保存数据时，如果未指定该字段的值，则采用默认值           | `DEFAULT`     |
| 检查约束(8.0.16版本之后) | 保证字段值满足某一个条件                                 | `CHECK`       |
| 外键约束                 | 用来让两张表的数据之间建立连接，保证数据的一致性和完整性 | `FOREIGN KEY` |

> ⚠️ 重要注意：约束是作用于表中字段上的，可以在**创建表**的时候添加约束，也可以在**修改表**的时候添加约束。

# 二、单表约束详解
单表约束是作用于单个表内部字段的约束，是最基础也是最常用的约束类型。

## 2.1 非空约束（NOT NULL）
限制字段的值不能为NULL。

**语法格式**：
```sql
-- 创建表时添加非空约束
CREATE TABLE 表名(
    字段名 数据类型 NOT NULL,
    ...
);

-- 修改表时添加非空约束
ALTER TABLE 表名 MODIFY 字段名 数据类型 NOT NULL;

-- 删除非空约束
ALTER TABLE 表名 MODIFY 字段名 数据类型;
```

**实战示例**：
```sql
-- 创建用户表，要求姓名不能为空
CREATE TABLE user(
    id INT,
    name VARCHAR(20) NOT NULL, -- 姓名不能为空
    age INT
);

-- 测试：插入数据时不指定姓名，会报错
INSERT INTO user(id, age) VALUES(1, 18); -- 报错：Column 'name' cannot be null

-- 正确插入：必须指定姓名
INSERT INTO user(id, name, age) VALUES(1, '张三', 18); -- 成功

-- 修改表：将age字段也改为非空
ALTER TABLE user MODIFY age INT NOT NULL;

-- 删除非空约束：允许age字段为空
ALTER TABLE user MODIFY age INT;
```

## 2.2 唯一约束（UNIQUE）
限制字段的值不能重复，可以为NULL（NULL可以有多个）。

**语法格式**：
```sql
-- 创建表时添加唯一约束
CREATE TABLE 表名(
    字段名 数据类型 UNIQUE,
    ...
);

-- 修改表时添加唯一约束
ALTER TABLE 表名 ADD CONSTRAINT 约束名 UNIQUE(字段名);

-- 删除唯一约束
ALTER TABLE 表名 DROP INDEX 约束名;
```

**实战示例**：
```sql
-- 创建用户表，要求手机号不能重复
CREATE TABLE user(
    id INT,
    name VARCHAR(20),
    phone VARCHAR(11) UNIQUE -- 手机号唯一
);

-- 测试：插入第一条数据成功
INSERT INTO user(id, name, phone) VALUES(1, '张三', '13800138000'); -- 成功

-- 测试：插入相同手机号，会报错
INSERT INTO user(id, name, phone) VALUES(2, '李四', '13800138000'); -- 报错：Duplicate entry '13800138000' for key 'phone'

-- 正确插入：使用不同的手机号
INSERT INTO user(id, name, phone) VALUES(2, '李四', '13800138001'); -- 成功

-- 修改表：为name字段添加唯一约束
ALTER TABLE user ADD CONSTRAINT uk_user_name UNIQUE(name);

-- 删除唯一约束
ALTER TABLE user DROP INDEX uk_user_name;
```

## 2.3 主键约束（PRIMARY KEY）
主键是一行数据的唯一标识，**自动包含非空约束和唯一约束**，一张表只能有一个主键。

**语法格式**：
```sql
-- 创建表时添加主键约束
CREATE TABLE 表名(
    字段名 数据类型 PRIMARY KEY AUTO_INCREMENT, -- AUTO_INCREMENT表示自动增长
    ...
);

-- 修改表时添加主键约束
ALTER TABLE 表名 ADD PRIMARY KEY(字段名);

-- 删除主键约束
ALTER TABLE 表名 DROP PRIMARY KEY;
```

**实战示例**：
```sql
-- 创建用户表，id为主键，自动增长
CREATE TABLE user(
    id INT PRIMARY KEY AUTO_INCREMENT, -- id为主键，自动增长
    name VARCHAR(20),
    phone VARCHAR(11)
);

-- 测试：插入数据时可以不指定id，自动生成
INSERT INTO user(name, phone) VALUES('张三', '13800138000'); -- id自动为1
INSERT INTO user(name, phone) VALUES('李四', '13800138001'); -- id自动为2

-- 查看数据，id已经自动生成
SELECT * FROM user;

-- 修改表：为没有主键的表添加主键
-- 先创建一个没有主键的表
CREATE TABLE temp_user(
    id INT,
    name VARCHAR(20)
);
-- 添加主键
ALTER TABLE temp_user ADD PRIMARY KEY(id);

-- 删除主键
ALTER TABLE temp_user DROP PRIMARY KEY;
```

> 💡 最佳实践：几乎所有的表都应该有一个主键，通常使用自增整数作为主键，性能最好。

## 2.4 默认约束（DEFAULT）
保存数据时，如果未指定该字段的值，则自动使用默认值。

**语法格式**：
```sql
-- 创建表时添加默认约束
CREATE TABLE 表名(
    字段名 数据类型 DEFAULT 默认值,
    ...
);

-- 修改表时添加默认约束
ALTER TABLE 表名 ALTER 字段名 SET DEFAULT 默认值;

-- 删除默认约束
ALTER TABLE 表名 ALTER 字段名 DROP DEFAULT;
```

**实战示例**：
```sql
-- 创建用户表，status字段默认为'1'
CREATE TABLE user(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20),
    status CHAR(1) DEFAULT '1' -- 状态默认为1
);

-- 测试：插入数据时不指定status，自动使用默认值
INSERT INTO user(name) VALUES('张三'); -- status自动为'1'

-- 查看数据
SELECT * FROM user;

-- 修改表：为age字段添加默认值18
ALTER TABLE user ADD COLUMN age INT;
ALTER TABLE user ALTER age SET DEFAULT 18;

-- 测试：插入数据时不指定age，自动为18
INSERT INTO user(name) VALUES('李四'); -- age自动为18

-- 删除默认约束
ALTER TABLE user ALTER age DROP DEFAULT;
```

## 2.5 检查约束（CHECK）
保证字段的值满足指定的条件，MySQL 8.0.16版本之后才支持。

**语法格式**：
```sql
-- 创建表时添加检查约束
CREATE TABLE 表名(
    字段名 数据类型 CHECK(条件),
    ...
);

-- 修改表时添加检查约束
ALTER TABLE 表名 ADD CONSTRAINT 约束名 CHECK(条件);

-- 删除检查约束
ALTER TABLE 表名 DROP CHECK 约束名;
```

**实战示例**：
```sql
-- 创建用户表，要求年龄必须在0到120之间
CREATE TABLE user(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20),
    age INT CHECK(age > 0 AND age <= 120) -- 年龄必须大于0且小于等于120
);

-- 测试：插入合法年龄，成功
INSERT INTO user(name, age) VALUES('张三', 18); -- 成功

-- 测试：插入非法年龄（负数），报错
INSERT INTO user(name, age) VALUES('李四', -1); -- 报错：Check constraint 'user_chk_1' is violated

-- 测试：插入非法年龄（超过120），报错
INSERT INTO user(name, age) VALUES('王五', 121); -- 报错：Check constraint 'user_chk_1' is violated

-- 修改表：添加性别检查约束，只能是'男'或'女'
ALTER TABLE user ADD COLUMN gender CHAR(1);
ALTER TABLE user ADD CONSTRAINT ck_user_gender CHECK(gender IN ('男', '女'));

-- 删除检查约束
ALTER TABLE user DROP CHECK ck_user_gender;
```

# 三、实战案例：创建带约束的表结构
根据以下业务需求，完成表结构的创建：

| 字段名 | 字段含义   | 字段类型    | 约束条件                  | 约束关键字                    |
| ------ | ---------- | ----------- | ------------------------- | ----------------------------- |
| id     | ID唯一标识 | int         | 主键，并且自动增长        | `PRIMARY KEY, AUTO_INCREMENT` |
| name   | 姓名       | varchar(10) | 不为空，并且唯一          | `NOT NULL, UNIQUE`            |
| age    | 年龄       | int         | 大于0，并且小于等于120    | `CHECK`                       |
| status | 状态       | char(1)     | 如果没有指定该值，默认为1 | `DEFAULT`                     |
| gender | 性别       | char(1)     | 无                        | -                             |

**实现代码**：
```sql
CREATE TABLE user(
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT 'ID唯一标识',
    name VARCHAR(10) NOT NULL UNIQUE COMMENT '姓名',
    age INT CHECK(age > 0 AND age <= 120) COMMENT '年龄',
    status CHAR(1) DEFAULT '1' COMMENT '状态',
    gender CHAR(1) COMMENT '性别'
) COMMENT '用户表';
```

# 四、外键约束
外键约束是用来让两张表的数据之间建立连接，从而保证数据的一致性和完整性的约束。

## 4.1 基本概念
- **父表（主表）**：被引用的表，包含主键字段
- **子表（从表）**：引用父表的表，包含外键字段

例如：员工表（子表）的`dept_id`字段引用部门表（父表）的`id`字段，这样就保证了员工的部门ID必须是部门表中存在的ID，不会出现无效的部门ID。

## 4.2 添加外键约束
### 方式1：创建表时添加外键
```sql
CREATE TABLE 表名(
    字段名 数据类型,
    ...
    [CONSTRAINT] [外键名称] FOREIGN KEY(外键字段名) REFERENCES 主表(主表列名)
);
```

### 方式2：修改表时添加外键
```sql
ALTER TABLE 表名 ADD CONSTRAINT 外键名称 FOREIGN KEY(外键字段名) REFERENCES 主表(主表列名);
```

## 4.3 删除外键约束
```sql
ALTER TABLE 表名 DROP FOREIGN KEY 外键名称;
```

## 4.4 外键的删除/更新行为
当父表中的数据被删除或更新时，子表中外键字段的行为可以通过以下参数设置：

| 行为          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| `NO ACTION`   | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。（与`RESTRICT`一致） |
| `RESTRICT`    | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。（与`NO ACTION`一致） |
| `CASCADE`     | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有，则也删除/更新外键在子表中的记录。 |
| `SET NULL`    | 当在父表中删除对应记录时，首先检查该记录是否有对应外键，如果有则设置子表中该外键值为null（这就要求该外键允许取null）。 |
| `SET DEFAULT` | 父表有变更时，子表将外键列设置成一个默认的值（InnoDB不支持）。 |

**语法格式**：
```sql
ALTER TABLE 表名 ADD CONSTRAINT 外键名称 
FOREIGN KEY (外键字段) REFERENCES 主表名(主表字段名) 
ON UPDATE CASCADE ON DELETE CASCADE;
```

# 五、完整实战演示：部门与员工表设计
下面我们通过一个完整的案例，演示外键约束的使用和级联操作。

## 5.1 创建部门表（父表）
```sql
CREATE TABLE dept(
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '部门ID',
    name VARCHAR(20) NOT NULL UNIQUE COMMENT '部门名称'
) COMMENT '部门表';

-- 插入测试数据
INSERT INTO dept(name) VALUES ('研发部'), ('市场部'), ('财务部'), ('销售部'), ('总经办');
```

## 5.2 创建员工表（子表）
```sql
CREATE TABLE emp(
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '员工ID',
    name VARCHAR(20) NOT NULL COMMENT '员工姓名',
    age INT COMMENT '年龄',
    job VARCHAR(20) COMMENT '职位',
    salary INT COMMENT '薪资',
    entrydate DATE COMMENT '入职日期',
    managerid INT COMMENT '上级领导ID',
    dept_id INT COMMENT '部门ID'
) COMMENT '员工表';

-- 插入测试数据
INSERT INTO emp(name, age, job, salary, entrydate, managerid, dept_id)
VALUES 
('金庸', 66, '总裁', 20000, '2000-01-01', NULL, 5),
('张无忌', 20, '项目经理', 12500, '2005-12-05', 1, 1),
('杨逍', 33, '开发', 8400, '2000-11-03', 2, 1),
('韦一笑', 48, '开发', 11000, '2002-02-05', 2, 1),
('常遇春', 43, '开发', 10500, '2004-09-07', 3, 1);
```

## 5.3 添加外键约束并设置级联操作
```sql
ALTER TABLE emp ADD CONSTRAINT fk_emp_dept_id 
FOREIGN KEY (dept_id) REFERENCES dept(id)
ON UPDATE CASCADE ON DELETE CASCADE;
```

## 5.4 测试级联操作
```sql
-- 测试级联更新：将部门表中id为1的部门ID改为10
UPDATE dept SET id = 10 WHERE id = 1;
-- 查看员工表，会发现所有dept_id为1的员工自动变为10
SELECT * FROM emp;

-- 测试级联删除：删除部门表中id为10的部门
DELETE FROM dept WHERE id = 10;
-- 查看员工表，会发现所有dept_id为10的员工自动被删除
SELECT * FROM emp;
```

# 六、常见误区与避坑指南
1. **主键只能有一个**：一张表只能有一个主键，但主键可以由多个字段组成（联合主键），不推荐使用联合主键。
2. **外键字段类型必须一致**：子表的外键字段类型必须和父表的主键字段类型完全一致，否则会报错。
3. **级联操作的风险**：`CASCADE`级联删除非常危险，生产环境中尽量不要使用级联删除，应该通过业务逻辑来控制数据的删除。
4. **检查约束的版本问题**：MySQL 8.0.16之前的版本不支持`CHECK`约束，虽然语法不会报错，但不会生效。
5. **自增主键的特点**：自增主键只会递增，不会回滚。如果插入失败，自增计数器仍然会加1，导致主键不连续。
6. **唯一约束允许NULL**：唯一约束允许字段的值为NULL，并且可以有多个NULL值，因为NULL不等于任何值，包括NULL本身。

# 七、核心总结：MySQL约束语法速查表
为了方便后续开发时快速查阅，整理了所有约束的核心语法速查表：

| 约束类型 | 关键字              | 作用                           | 语法示例                                   |
| -------- | ------------------- | ------------------------------ | ------------------------------------------ |
| 非空约束 | `NOT NULL`          | 限制字段不能为NULL             | `name VARCHAR(20) NOT NULL`                |
| 唯一约束 | `UNIQUE`            | 限制字段值不能重复             | `phone VARCHAR(11) UNIQUE`                 |
| 主键约束 | `PRIMARY KEY`       | 一行数据的唯一标识，非空且唯一 | `id INT PRIMARY KEY AUTO_INCREMENT`        |
| 默认约束 | `DEFAULT`           | 未指定值时使用默认值           | `status CHAR(1) DEFAULT '1'`               |
| 检查约束 | `CHECK`             | 限制字段值满足条件             | `age INT CHECK(age > 0)`                   |
| 外键约束 | `FOREIGN KEY`       | 建立两张表的关联               | `FOREIGN KEY(dept_id) REFERENCES dept(id)` |
| 级联更新 | `ON UPDATE CASCADE` | 父表更新时子表自动更新         | `ON UPDATE CASCADE`                        |
| 级联删除 | `ON DELETE CASCADE` | 父表删除时子表自动删除         | `ON DELETE CASCADE`                        |

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163079972>
