


# Python全栈入门到实战【数据库篇 08】MySQL DQL基本查询与条件查询详解，数据库最常用操作
上一篇《数据库篇 07》中，我们已经掌握了MySQL DML数据操纵语言，学会了数据的添加、修改和删除操作。本篇作为数据库篇的第八篇，我们将学习**数据库中使用频率最高的DQL数据查询语言**，从最基础的单表查询开始，掌握基本查询和条件查询的核心语法，这是所有数据统计、分析和展示的基础，也是后端开发中最核心的数据库技能。

本文为Python全栈开发者与数据库入门者量身打造，采用"语法+表格+实战示例"的结构，每一个关键字和运算符都有清晰的说明和可直接运行的代码，同时重点标注新手最容易混淆的概念和常见错误，即使是完全没有SQL基础的同学，也能快速掌握DQL查询的核心方法。

本节核心学习内容：
1. DQL语言概述：作用与重要性
2. 基本查询：多字段查询、设置别名、去除重复记录
3. 条件查询：WHERE子句语法与使用场景
4. 运算符详解：比较运算符与逻辑运算符的完整用法
5. 完整实战：基于学生表的查询操作演示
6. 常见误区：SELECT *滥用、NULL值判断、模糊查询陷阱
7. 核心总结：DQL基本查询语法速查表，方便开发时快速查阅

# 一、DQL语言概述
DQL（Data Query Language，数据查询语言）是用于从数据库表中**提取符合条件的数据**的SQL语言，是所有SQL语句中使用频率最高的部分，占日常数据库操作的80%以上。

DQL的核心特点：
- 只查询数据，**不会修改数据库中的任何数据**
- 支持灵活的条件过滤，只返回需要的数据
- 支持对查询结果进行排序、分组、聚合等复杂处理（后续文章讲解）
- 是所有数据展示、统计分析、报表生成的基础

> ⚠️ 注意：DQL查询不会改变表中的原始数据，只是将数据从数据库中读取出来返回给客户端。

# 二、基本查询
基本查询是最简单的DQL语句，用于从表中查询指定的字段或所有字段的数据，不需要任何条件过滤。

## 2.1 查询多个字段
可以查询表中的一个或多个指定字段，多个字段之间用逗号分隔。如果需要查询表中的所有字段，可以使用通配符`*`代替所有字段名。

**语法格式**：
```sql
-- 查询指定字段
SELECT 字段1, 字段2, 字段3 ... FROM 表名;

-- 查询所有字段
SELECT * FROM 表名;
```

**实战示例**（基于之前创建的`student`学生表）：
```sql
-- 查询学生表中的id、name、age三个字段
SELECT id, name, age FROM student;

-- 查询学生表中的所有字段
SELECT * FROM student;
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f7112837252a4b6d9c61c7a6fe9099db.png#pic_center)


> 💡 最佳实践：在生产环境中，尽量避免使用`SELECT *`查询所有字段。只查询需要的字段可以减少数据传输量，提升查询效率，同时避免暴露不必要的敏感字段。

## 2.2 设置别名
在查询时，可以为字段或表设置别名，让查询结果的列名更易读，或者简化复杂的表名和字段名。

**语法格式**：
```sql
SELECT 字段1 [AS 别名1], 字段2 [AS 别名2] ... FROM 表名 [AS 表别名];
```

**说明**：
- `AS`关键字可以省略，直接在字段名后面写别名即可
- 别名可以包含中文，但建议使用英文或拼音
- 如果别名包含空格或特殊字符，需要用单引号或双引号包裹

**实战示例**：
```sql
-- 为name字段设置别名"学生姓名"，为age字段设置别名"年龄"
SELECT name AS 学生姓名, age AS 年龄 FROM student;

-- 省略AS关键字，直接设置别名
SELECT name 学生姓名, age 年龄 FROM student;

-- 为表设置别名，简化查询语句
SELECT s.id, s.name, s.age FROM student s;
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3eebab31e8ab4cd1baaf1a3cf2ce438d.png#pic_center)


## 2.3 去除重复记录

当查询结果中存在重复的记录时，可以使用`DISTINCT`关键字去除重复的行，只保留唯一的记录。

**语法格式**：
```sql
SELECT DISTINCT 字段列表 FROM 表名;
```

**实战示例**：
```sql
-- 查询学生表中所有的班级，去除重复的班级名称
SELECT DISTINCT class FROM student;

-- 查询学生表中不同班级和性别的组合，去除重复
SELECT DISTINCT class, gender FROM student;
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a188a35e81ea412baee21011522272b5.png#pic_center)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5068664a0a9b4634ae907c2078634393.png#pic_center)


> ⚠️ 注意：`DISTINCT`关键字会作用于它后面的所有字段，只有当所有字段的值都相同时，才会被认为是重复记录。

# 三、条件查询
基本查询会返回表中的所有记录，但在实际开发中，我们通常只需要查询符合特定条件的数据。这时就需要使用`WHERE`子句来添加查询条件，只返回满足条件的记录。

## 3.1 基本语法
```sql
SELECT 字段列表 FROM 表名 WHERE 条件列表;
```

**执行顺序**：
1. 先执行`FROM 表名`，确定要查询的表
2. 再执行`WHERE 条件列表`，过滤出符合条件的记录
3. 最后执行`SELECT 字段列表`，提取指定的字段

## 3.2 条件运算符
条件查询支持两种类型的运算符：比较运算符和逻辑运算符，用于构建各种复杂的查询条件。

### 比较运算符
比较运算符用于比较两个值的大小或相等性，返回布尔值（真或假）。

| 比较运算符            | 功能                                           |
| --------------------- | ---------------------------------------------- |
| `>`                   | 大于                                           |
| `>=`                  | 大于等于                                       |
| `<`                   | 小于                                           |
| `<=`                  | 小于等于                                       |
| `=`                   | 等于                                           |
| `<>` 或 `!=`          | 不等于                                         |
| `BETWEEN ... AND ...` | 在某个范围之内（包含最小值和最大值）           |
| `IN(...)`             | 在IN之后的列表中的值，多选一                   |
| `LIKE 占位符`         | 模糊匹配（`_`匹配单个字符，`%`匹配任意个字符） |
| `IS NULL`             | 是NULL                                         |

### 逻辑运算符
逻辑运算符用于组合多个比较条件，实现更复杂的逻辑判断。

| 逻辑运算符    | 功能                         |
| ------------- | ---------------------------- |
| `AND` 或 `&&` | 并且（多个条件同时成立）     |
| `OR` 或 `||`  | 或者（多个条件任意一个成立） |
| `NOT` 或 `!`  | 非，不是（取反）             |

## 3.3 条件查询实战示例
下面我们基于`student`学生表，演示各种条件查询的用法：

### 比较运算符示例
```sql
-- 查询年龄大于18岁的学生
SELECT * FROM student WHERE age > 18;

-- 查询年龄在17到19岁之间的学生（包含17和19）
SELECT * FROM student WHERE age BETWEEN 17 AND 19;

-- 查询班级是"高一1班"或"高一2班"的学生
SELECT * FROM student WHERE class IN ('高一1班', '高一2班');

-- 查询姓名以"小"开头的学生（模糊查询）
SELECT * FROM student WHERE name LIKE '小%';

-- 查询姓名包含"明"字的学生
SELECT * FROM student WHERE name LIKE '%明%';

-- 查询姓名是两个字的学生（_匹配单个字符）
SELECT * FROM student WHERE name LIKE '__';

-- 查询邮箱为NULL的学生
SELECT * FROM student WHERE email IS NULL;

-- 查询邮箱不为NULL的学生
SELECT * FROM student WHERE email IS NOT NULL;
```

### 逻辑运算符示例
```sql
-- 查询年龄大于18岁且性别为男的学生
SELECT * FROM student WHERE age > 18 AND gender = '男';

-- 查询班级是"高一1班"或者年龄小于17岁的学生
SELECT * FROM student WHERE class = '高一1班' OR age < 17;

-- 查询不是"高一3班"的所有学生
SELECT * FROM student WHERE NOT class = '高一3班';

-- 组合多个条件：查询年龄在17到19岁之间且性别为女的学生
SELECT * FROM student WHERE age BETWEEN 17 AND 19 AND gender = '女';
```

# 四、完整实战演示
下面我们通过一个完整的案例，演示从准备数据到执行各种查询的完整流程。

## 4.1 准备测试数据
首先在PyCharm的Console中执行以下SQL，向`student`表中插入更多测试数据：
```sql
INSERT INTO student (name, age, gender, class, email)
VALUES 
('张三', 18, '男', '高一1班', 'zhangsan@example.com'),
('李四', 17, '女', '高一2班', 'lisi@example.com'),
('王五', 19, '男', '高一1班', 'wangwu@example.com'),
('赵六', 18, '女', '高一3班', 'zhaoliu@example.com'),
('孙七', 17, '男', '高一2班', 'sunqi@example.com'),
('周八', 19, '女', '高一1班', 'zhouba@example.com'),
('吴九', 18, '男', '高一3班', NULL),
('郑十', 17, '女', '高一2班', NULL);
```

## 4.2 基本查询实战
```sql
-- 1. 查询所有学生的姓名和班级
SELECT name, class FROM student;

-- 2. 查询所有不同的班级
SELECT DISTINCT class FROM student;

-- 3. 查询学生的姓名和年龄，设置别名
SELECT name AS 学生姓名, age AS 学生年龄 FROM student;
```

## 4.3 条件查询实战
```sql
-- 1. 查询高一1班的所有学生
SELECT * FROM student WHERE class = '高一1班';

-- 2. 查询年龄大于等于18岁的男学生
SELECT * FROM student WHERE age >= 18 AND gender = '男';

-- 3. 查询姓名以"张"或"李"开头的学生
SELECT * FROM student WHERE name LIKE '张%' OR name LIKE '李%';

-- 4. 查询年龄在17到18岁之间，且邮箱不为空的学生
SELECT * FROM student WHERE age BETWEEN 17 AND 18 AND email IS NOT NULL;

-- 5. 查询不是高一2班的所有女学生
SELECT * FROM student WHERE NOT class = '高一2班' AND gender = '女';
```

# 五、常见误区与避坑指南
1. **SELECT *的滥用**：`SELECT *`会查询表中的所有字段，不仅效率低，还可能暴露敏感数据。生产环境中必须明确指定需要查询的字段。
2. **NULL值的判断**：判断字段是否为NULL必须使用`IS NULL`或`IS NOT NULL`，不能使用`= NULL`或`!= NULL`，因为NULL与任何值比较的结果都是NULL。
3. **模糊查询的占位符**：`_`只能匹配单个字符，`%`可以匹配任意个字符（包括0个字符）。如果需要匹配`_`或`%`本身，需要使用转义字符`\`。
4. **逻辑运算符的优先级**：`NOT`的优先级高于`AND`，`AND`的优先级高于`OR`。如果不确定优先级，建议使用括号明确运算顺序。
5. **字符串比较的大小写**：MySQL默认不区分大小写，如果需要区分大小写，可以使用`BINARY`关键字：`SELECT * FROM student WHERE BINARY name = 'ZhangSan';`

# 六、核心总结：DQL基本查询语法速查表
为了方便后续开发时快速查阅，整理了DQL基本查询和条件查询的核心语法速查表：

| 查询类型     | 语法格式                                | 核心要点                           |
| ------------ | --------------------------------------- | ---------------------------------- |
| 查询指定字段 | `SELECT 字段1,字段2 FROM 表名;`         | 多个字段用逗号分隔                 |
| 查询所有字段 | `SELECT * FROM 表名;`                   | 生产环境尽量避免使用               |
| 设置别名     | `SELECT 字段 AS 别名 FROM 表名;`        | AS关键字可以省略                   |
| 去除重复记录 | `SELECT DISTINCT 字段列表 FROM 表名;`   | 作用于所有字段                     |
| 条件查询     | `SELECT 字段列表 FROM 表名 WHERE 条件;` | 先过滤再提取字段                   |
| 范围查询     | `WHERE 字段 BETWEEN 最小值 AND 最大值;` | 包含两端的值                       |
| 多选一查询   | `WHERE 字段 IN (值1,值2,值3);`          | 匹配列表中的任意一个值             |
| 模糊查询     | `WHERE 字段 LIKE '匹配模式';`           | `_`匹配单个字符，`%`匹配任意个字符 |
| NULL值查询   | `WHERE 字段 IS NULL;`                   | 不能使用`= NULL`                   |
| 逻辑与       | `WHERE 条件1 AND 条件2;`                | 多个条件同时成立                   |
| 逻辑或       | `WHERE 条件1 OR 条件2;`                 | 多个条件任意一个成立               |
| 逻辑非       | `WHERE NOT 条件;`                       | 取反                               |

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/162233307>
