

# Python全栈入门到实战【数据库篇 09】MySQL DQL聚合函数与分组查询详解，数据统计分析核心
上一篇《数据库篇 08》中，我们已经掌握了MySQL DQL的基本查询和条件查询，学会了从表中提取符合条件的原始数据。本篇作为数据库篇的第九篇，我们将学习**数据统计分析的核心语法**——聚合函数与分组查询。通过聚合函数可以对一列数据进行纵向计算，通过分组查询可以将数据按指定维度分组统计，这是所有报表生成、数据可视化和业务分析的基础。

本文为Python全栈开发者与数据库入门者量身打造，重点拆解新手最容易混淆的`WHERE`与`HAVING`的区别，每一个函数和语法都有清晰的说明和可直接运行的实战示例，同时整理了常见的坑和避坑指南，即使是完全没有数据统计经验的同学，也能快速掌握数据聚合与分组的核心方法。

本节核心学习内容：
1. 聚合函数概述：作用与核心特点
2. 常用聚合函数：count、max、min、avg、sum详解
3. 分组查询：GROUP BY语法与使用场景
4. 核心难点：WHERE与HAVING的本质区别
5. 完整实战：基于学生表的多维度统计分析
6. 常见误区：count用法、分组字段限制、执行顺序陷阱
7. 核心总结：聚合与分组查询语法速查表，方便开发时快速查阅

# 一、聚合函数概述
聚合函数是DQL中用于**数据统计计算**的函数，它会将一列数据作为一个整体，进行纵向的计算，最终返回一个单一的结果值。

聚合函数的核心特点：
- 只对**非NULL值**进行计算，NULL值会被自动忽略
- 通常与`SELECT`语句配合使用，用于统计整张表或分组后的数据集
- 不能直接在`WHERE`子句中使用（这是新手最容易犯的错误）
- 是所有数据统计、报表生成和业务分析的基础

# 二、常用聚合函数详解
MySQL提供了5个最常用的聚合函数，覆盖了绝大多数数据统计场景：

| 函数          | 功能                       |
| ------------- | -------------------------- |
| `count(字段)` | 统计指定字段的非NULL值数量 |
| `max(字段)`   | 计算指定字段的最大值       |
| `min(字段)`   | 计算指定字段的最小值       |
| `avg(字段)`   | 计算指定字段的平均值       |
| `sum(字段)`   | 计算指定字段的总和         |

## 2.1 基本语法
```sql
SELECT 聚合函数(字段列表) FROM 表名 [WHERE 条件];
```

## 2.2 各聚合函数实战示例
基于之前创建的`student`学生表，演示各个聚合函数的用法：

### 1. count函数：统计数量
count函数是使用频率最高的聚合函数，用于统计符合条件的记录数。它有三种常见的用法：
```sql
-- 1. 统计学生表的总记录数（推荐使用）
SELECT count(*) FROM student;

-- 2. 统计email字段不为NULL的学生数量
SELECT count(email) FROM student;

-- 3. 统计高一1班的学生数量
SELECT count(*) FROM student WHERE class = '高一1班';
```

> 💡 注意：`count(*)`会统计表中所有的记录数，包括NULL值的行；`count(字段)`只会统计该字段不为NULL的记录数。在大多数情况下，推荐使用`count(*)`来统计总记录数。

### 2. max/min函数：求最大值/最小值
```sql
-- 统计学生表中的最大年龄
SELECT max(age) FROM student;

-- 统计学生表中的最小年龄
SELECT min(age) FROM student;

-- 统计高一2班学生的最大年龄
SELECT max(age) FROM student WHERE class = '高一2班';
```

### 3. avg函数：求平均值
```sql
-- 统计所有学生的平均年龄
SELECT avg(age) FROM student;

-- 统计所有男学生的平均年龄
SELECT avg(age) FROM student WHERE gender = '男';
```

### 4. sum函数：求和
```sql
-- 统计所有学生的年龄总和
SELECT sum(age) FROM student;

-- 统计高一3班学生的年龄总和
SELECT sum(age) FROM student WHERE class = '高一3班';
```

### 5. 同时使用多个聚合函数
可以在一个SELECT语句中同时使用多个聚合函数，一次性统计多个指标：
```sql
-- 一次性统计学生总数、最大年龄、最小年龄、平均年龄
SELECT 
    count(*) AS 学生总数,
    max(age) AS 最大年龄,
    min(age) AS 最小年龄,
    avg(age) AS 平均年龄
FROM student;
```

# 三、分组查询
当需要对数据按不同维度进行分类统计时，就需要使用分组查询。例如：统计每个班级的学生人数、统计每个性别的平均年龄等。

## 3.1 基本语法
```sql
SELECT 字段列表 FROM 表名 
[WHERE 分组前过滤条件] 
GROUP BY 分组字段名 
[HAVING 分组后过滤条件];
```

## 3.2 核心难点：WHERE与HAVING的区别
这是分组查询中最容易混淆的知识点，也是面试的高频考点，两者的本质区别体现在两个方面：

| 区别         | WHERE                                               | HAVING                                       |
| ------------ | --------------------------------------------------- | -------------------------------------------- |
| **执行时机** | 分组之前进行过滤，不满足WHERE条件的记录不会参与分组 | 分组之后对结果进行过滤，只保留满足条件的分组 |
| **判断条件** | 不能对聚合函数进行判断                              | 可以对聚合函数进行判断                       |

> 🚨 重要执行顺序：`WHERE` → 聚合函数 → `HAVING`

## 3.3 分组查询注意事项
- 分组之后，查询的字段**只能是聚合函数和分组字段**，查询其他字段没有任何意义
- 可以按多个字段进行分组，多个字段之间用逗号分隔
- `WHERE`和`HAVING`可以同时使用，`WHERE`先过滤原始数据，`HAVING`再过滤分组后的结果

## 3.4 分组查询实战示例
```sql
-- 1. 统计每个班级的学生人数
SELECT class, count(*) AS 学生人数 FROM student GROUP BY class;

-- 2. 统计每个班级的平均年龄
SELECT class, avg(age) AS 平均年龄 FROM student GROUP BY class;

-- 3. 统计每个班级中男女生的人数（按多个字段分组）
SELECT class, gender, count(*) AS 人数 FROM student GROUP BY class, gender;

-- 4. 先过滤再分组：统计年龄大于17岁的学生，每个班级的人数
SELECT class, count(*) AS 人数 FROM student WHERE age > 17 GROUP BY class;

-- 5. 分组后再过滤：统计学生人数大于2人的班级
SELECT class, count(*) AS 人数 FROM student GROUP BY class HAVING count(*) > 2;

-- 6. 组合使用：统计年龄大于17岁的学生中，人数大于1人的班级
SELECT class, count(*) AS 人数 
FROM student 
WHERE age > 17 
GROUP BY class 
HAVING count(*) > 1;
```

# 四、完整实战演示
下面我们通过一个完整的案例，演示聚合函数和分组查询的综合使用。

## 4.1 准备测试数据
首先在PyCharm的Console中执行以下SQL，向`student`表中插入更多测试数据：
```sql
INSERT INTO student (name, age, gender, class, score)
VALUES 
('小明', 18, '男', '高一1班', 85),
('小红', 17, '女', '高一2班', 92),
('王五', 19, '男', '高一1班', 78),
('赵六', 18, '女', '高一3班', 88),
('孙七', 17, '男', '高一2班', 75),
('周八', 19, '女', '高一1班', 95),
('吴九', 18, '男', '高一3班', 82),
('郑十', 17, '女', '高一2班', 90),
('钱十一', 18, '男', '高一1班', 79),
('孙十二', 19, '女', '高一3班', 87);
```

## 4.2 聚合函数综合实战
```sql
-- 1. 统计全班的总分、平均分、最高分、最低分
SELECT 
    sum(score) AS 总分,
    avg(score) AS 平均分,
    max(score) AS 最高分,
    min(score) AS 最低分
FROM student;

-- 2. 统计高一1班的数学成绩统计
SELECT 
    count(*) AS 人数,
    avg(score) AS 平均分,
    max(score) AS 最高分
FROM student WHERE class = '高一1班';
```

## 4.3 分组查询综合实战
```sql
-- 1. 统计每个班级的成绩统计
SELECT 
    class AS 班级,
    count(*) AS 人数,
    avg(score) AS 平均分,
    max(score) AS 最高分,
    min(score) AS 最低分
FROM student 
GROUP BY class;

-- 2. 统计每个性别的成绩统计
SELECT 
    gender AS 性别,
    count(*) AS 人数,
    avg(score) AS 平均分
FROM student 
GROUP BY gender;

-- 3. 统计平均分大于85分的班级，并按平均分降序排列
SELECT 
    class AS 班级,
    avg(score) AS 平均分
FROM student 
GROUP BY class 
HAVING avg(score) > 85
ORDER BY 平均分 DESC;

-- 4. 统计每个班级中，分数大于80分的学生人数
SELECT 
    class AS 班级,
    count(*) AS 优秀人数
FROM student 
WHERE score > 80 
GROUP BY class;
```

# 五、常见误区与避坑指南
1. **在WHERE子句中使用聚合函数**：这是最常见的错误。由于`WHERE`的执行时机早于聚合函数，因此不能在`WHERE`中使用聚合函数进行判断，应该使用`HAVING`。
   ```sql
   -- 错误写法
   SELECT class, count(*) FROM student WHERE count(*) > 2 GROUP BY class;

   -- 正确写法
   SELECT class, count(*) FROM student GROUP BY class HAVING count(*) > 2;
   ```

2. **分组后查询非聚合字段**：分组之后，查询的字段只能是聚合函数和分组字段，查询其他字段没有任何意义，不同数据库的处理方式不同，可能会返回随机值。

3. **count(NULL)的问题**：`count(字段)`会自动忽略NULL值，如果该字段有很多NULL值，统计结果会不准确。统计总记录数推荐使用`count(*)`。

4. **HAVING代替WHERE**：不要用`HAVING`代替`WHERE`，`WHERE`过滤原始数据的效率更高，应该尽量将能在`WHERE`中过滤的条件放在`WHERE`中，减少参与分组的数据量。

5. **聚合函数中使用DISTINCT**：如果需要统计不重复的值，可以在聚合函数中使用`DISTINCT`：
   ```sql
   -- 统计有多少个不同的班级
   SELECT count(DISTINCT class) FROM student;
   ```

# 六、核心总结：聚合与分组查询语法速查表
为了方便后续开发时快速查阅，整理了聚合函数与分组查询的核心语法速查表：

| 操作类型   | 语法格式                                                | 核心要点                     |
| ---------- | ------------------------------------------------------- | ---------------------------- |
| 统计数量   | `SELECT count(*) FROM 表名;`                            | 推荐使用count(*)统计总记录数 |
| 求最大值   | `SELECT max(字段) FROM 表名;`                           | 自动忽略NULL值               |
| 求最小值   | `SELECT min(字段) FROM 表名;`                           | 自动忽略NULL值               |
| 求平均值   | `SELECT avg(字段) FROM 表名;`                           | 自动忽略NULL值               |
| 求和       | `SELECT sum(字段) FROM 表名;`                           | 自动忽略NULL值               |
| 基本分组   | `SELECT 分组字段,聚合函数 FROM 表名 GROUP BY 分组字段;` | 只能查询聚合函数和分组字段   |
| 分组前过滤 | `SELECT ... FROM 表名 WHERE 条件 GROUP BY ...;`         | WHERE不能使用聚合函数        |
| 分组后过滤 | `SELECT ... FROM 表名 GROUP BY ... HAVING 条件;`        | HAVING可以使用聚合函数       |
| 执行顺序   | `WHERE → 聚合函数 → HAVING`                             | 牢记执行顺序，避免语法错误   |

# 七、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/162267273>
