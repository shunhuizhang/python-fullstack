

# Python全栈入门到实战【数据库篇 12】MySQL常用内置函数详解，数据处理与业务逻辑必备
上一篇《数据库篇 11》中，我们已经掌握了MySQL DCL数据控制语言，学会了用户管理和权限控制，保障了数据库的安全。本篇作为数据库篇的第十二篇，我们将学习**MySQL最常用的内置函数**，包括字符串函数、数值函数、日期函数和流程函数。这些函数可以帮助我们在SQL层面直接处理数据，无需在应用程序中进行额外的转换，大幅提升数据处理效率，是业务开发中不可或缺的技能。

本文为Python全栈开发者与数据库入门者量身打造，详细讲解每一个常用函数的功能、语法和实战用法，每一个函数都有可直接运行的示例代码，同时重点标注新手最容易踩的坑，即使是完全没有数据处理经验的同学，也能快速掌握这些函数的使用方法。

本节核心学习内容：
1. 内置函数概述：作用与核心分类
2. 字符串函数：拼接、大小写转换、填充、截取等常用操作
3. 数值函数：取整、取模、随机数、四舍五入等数学计算
4. 日期函数：获取当前时间、日期计算、日期差等时间处理
5. 流程函数：条件判断、空值处理、多分支逻辑实现
6. 综合实战：基于学生表的多函数组合业务场景
7. 常见误区：索引问题、NULL值处理、日期格式陷阱
8. 核心总结：常用内置函数速查表，方便开发时快速查阅

# 一、MySQL内置函数概述
MySQL提供了大量的内置函数，用于对数据库中的数据进行各种处理和转换。这些函数可以直接嵌入到SQL语句中使用，将数据处理逻辑从应用程序转移到数据库层面，不仅能提升查询效率，还能简化应用代码，降低开发复杂度。

MySQL内置函数主要分为以下四大类，覆盖了绝大多数业务数据处理场景：
- **字符串函数**：专门用于处理字符串类型的数据
- **数值函数**：用于处理数值类型的数据，执行各种数学运算
- **日期函数**：用于处理日期和时间类型的数据
- **流程函数**：用于在SQL语句中实现条件判断和流程控制

# 二、字符串函数
MySQL中内置了丰富的字符串处理函数，是日常开发中使用频率最高的一类函数。

## 2.1 常用字符串函数
| 函数                       | 功能                                                      |
| -------------------------- | --------------------------------------------------------- |
| `CONCAT(S1,S2,...Sn)`      | 字符串拼接，将S1，S2，... Sn拼接成一个字符串              |
| `LOWER(str)`               | 将字符串str全部转为小写                                   |
| `UPPER(str)`               | 将字符串str全部转为大写                                   |
| `LPAD(str,n,pad)`          | 左填充，用字符串pad对str的左边进行填充，达到n个字符串长度 |
| `RPAD(str,n,pad)`          | 右填充，用字符串pad对str的右边进行填充，达到n个字符串长度 |
| `TRIM(str)`                | 去掉字符串头部和尾部的空格                                |
| `SUBSTRING(str,start,len)` | 返回从字符串str从start位置起的len个长度的字符串           |

## 2.2 实战示例
基于之前创建的`student`学生表，演示各个字符串函数的用法：
```sql
-- 1. 字符串拼接：将学生姓名和班级拼接成"姓名-班级"格式
SELECT CONCAT(name, '-', class) AS 学生信息 FROM student;

-- 2. 转换为大写：将学生姓名转换为大写
SELECT UPPER(name) AS 姓名大写 FROM student;

-- 3. 转换为小写：将学生姓名转换为小写
SELECT LOWER(name) AS 姓名小写 FROM student;

-- 4. 左填充：将学生ID统一填充为6位，不足的用0补齐
SELECT LPAD(id, 6, '0') AS 学号 FROM student;

-- 5. 右填充：将学生姓名右填充为10个字符，用空格补齐
SELECT RPAD(name, 10, ' ') AS 姓名 FROM student;

-- 6. 去除空格：去除学生姓名前后的多余空格
SELECT TRIM(name) AS 姓名 FROM student;

-- 7. 字符串截取：截取学生姓名的前1个字符作为姓氏
SELECT SUBSTRING(name, 1, 1) AS 姓氏 FROM student;
```

> ⚠️ 重要注意：MySQL中`SUBSTRING`函数的索引**从1开始**，不是从0开始，这是新手最容易犯的错误。

# 三、数值函数
数值函数用于执行各种数学运算，是数据统计和计算的基础。

## 3.1 常用数值函数
| 函数         | 功能                         |
| ------------ | ---------------------------- |
| `CEIL(x)`    | 向上取整                     |
| `FLOOR(x)`   | 向下取整                     |
| `MOD(x,y)`   | 返回x除以y的余数（模）       |
| `RAND()`     | 返回0~1之间的随机小数        |
| `ROUND(x,y)` | 对x进行四舍五入，保留y位小数 |

## 3.2 实战示例
```sql
-- 1. 向上取整
SELECT CEIL(3.14); -- 结果：4
SELECT CEIL(-3.14); -- 结果：-3

-- 2. 向下取整
SELECT FLOOR(3.14); -- 结果：3
SELECT FLOOR(-3.14); -- 结果：-4

-- 3. 取模运算
SELECT MOD(10, 3); -- 结果：1
SELECT MOD(10, -3); -- 结果：1

-- 4. 生成随机数
SELECT RAND(); -- 结果：0~1之间的随机小数
-- 生成1~100之间的随机整数
SELECT FLOOR(RAND() * 100) + 1;

-- 5. 四舍五入
SELECT ROUND(3.14159, 2); -- 结果：3.14
SELECT ROUND(3.145, 2); -- 结果：3.15
```

# 四、日期函数
日期函数用于处理日期和时间类型的数据，在订单管理、用户注册、日志记录等场景中广泛使用。

## 4.1 常用日期函数
| 函数                                 | 功能                                          |
| ------------------------------------ | --------------------------------------------- |
| `CURDATE()`                          | 返回当前日期（YYYY-MM-DD）                    |
| `CURTIME()`                          | 返回当前时间（HH:MM:SS）                      |
| `NOW()`                              | 返回当前日期和时间（YYYY-MM-DD HH:MM:SS）     |
| `YEAR(date)`                         | 获取指定日期的年份                            |
| `MONTH(date)`                        | 获取指定日期的月份                            |
| `DAY(date)`                          | 获取指定日期的日                              |
| `DATE_ADD(date, INTERVAL expr type)` | 返回一个日期/时间值加上一个时间间隔后的时间值 |
| `DATEDIFF(date1,date2)`              | 返回date1减去date2的天数差                    |

## 4.2 实战示例
```sql
-- 1. 获取当前日期
SELECT CURDATE();

-- 2. 获取当前时间
SELECT CURTIME();

-- 3. 获取当前日期和时间
SELECT NOW();

-- 4. 提取日期的年、月、日
SELECT YEAR(NOW()) AS 年份, MONTH(NOW()) AS 月份, DAY(NOW()) AS 日期;

-- 5. 日期加法：计算当前日期加7天后的日期
SELECT DATE_ADD(NOW(), INTERVAL 7 DAY);
-- 计算当前日期减1个月后的日期
SELECT DATE_ADD(NOW(), INTERVAL -1 MONTH);

-- 6. 计算两个日期之间的天数差
SELECT DATEDIFF('2026-05-01', '2026-04-12'); -- 结果：19
```

# 五、流程函数
流程函数可以在SQL语句中实现条件判断和流程控制，替代应用程序中的部分逻辑，让SQL语句更加强大。

## 5.1 常用流程函数
| 函数                                                         | 功能                                                      |
| ------------------------------------------------------------ | --------------------------------------------------------- |
| `IF(value , t , f)`                                          | 如果value为true，则返回t，否则返回f                       |
| `IFNULL(value1 , value2)`                                    | 如果value1不为空，返回value1，否则返回value2              |
| `CASE WHEN [val1] THEN [res1] ... ELSE [default] END`        | 如果val1为true，返回res1，... 否则返回default默认值       |
| `CASE [expr] WHEN [val1] THEN [res1] ... ELSE [default] END` | 如果expr的值等于val1，返回res1，... 否则返回default默认值 |

## 5.2 实战示例
```sql
-- 1. IF函数：判断学生成绩是否及格
SELECT name, score, IF(score >= 60, '及格', '不及格') AS 成绩等级 FROM student;

-- 2. IFNULL函数：处理空值，将邮箱为空的显示为"未填写"
SELECT name, IFNULL(email, '未填写') AS 邮箱 FROM student;

-- 3. CASE WHEN多分支判断：根据分数划分成绩等级
SELECT 
    name, 
    score,
    CASE 
        WHEN score >= 90 THEN '优秀'
        WHEN score >= 80 THEN '良好'
        WHEN score >= 60 THEN '及格'
        ELSE '不及格'
    END AS 成绩等级
FROM student;

-- 4. CASE expr WHEN：根据性别显示中文
SELECT 
    name,
    CASE gender
        WHEN '男' THEN '男生'
        WHEN '女' THEN '女生'
        ELSE '未知'
    END AS 性别
FROM student;
```

# 六、综合实战演示
下面我们通过一个完整的业务案例，演示多个内置函数的组合使用。

## 6.1 需求说明
基于`student`学生表，完成以下查询需求：
1. 将学生姓名和班级拼接成"姓名(班级)"格式
2. 将学生分数四舍五入保留1位小数
3. 根据分数划分成绩等级（优秀、良好、及格、不及格）
4. 计算学生的入学天数（假设入学日期为2025-09-01）
5. 将邮箱为空的显示为"未填写"

## 6.2 实现代码
```sql
SELECT 
    CONCAT(name, '(', class, ')') AS 学生信息,
    ROUND(score, 1) AS 分数,
    CASE 
        WHEN score >= 90 THEN '优秀'
        WHEN score >= 80 THEN '良好'
        WHEN score >= 60 THEN '及格'
        ELSE '不及格'
    END AS 成绩等级,
    DATEDIFF(NOW(), '2025-09-01') AS 入学天数,
    IFNULL(email, '未填写') AS 邮箱
FROM student;
```

# 七、常见误区与避坑指南
1. **SUBSTRING索引问题**：MySQL中`SUBSTRING`函数的索引从1开始，不是从0开始。例如`SUBSTRING('abc', 1, 1)`返回'a'，而不是'b'。
2. **NULL值运算问题**：任何函数与NULL值运算的结果都是NULL，因此在使用函数时要注意处理NULL值，优先使用`IFNULL`函数进行兜底。
3. **日期格式问题**：MySQL默认的日期格式是`'YYYY-MM-DD'`，如果使用其他格式的日期字符串，需要使用`STR_TO_DATE`函数进行转换。
4. **负数取整问题**：`CEIL(-3.14)`返回-3，`FLOOR(-3.14)`返回-4，这一点和很多编程语言的行为一致，但容易被忽略。
5. **随机数生成问题**：`RAND()`函数每次调用都会生成一个新的随机数，生成指定范围的整数请使用`FLOOR(RAND() * (max - min + 1)) + min`。

# 八、核心总结：MySQL常用内置函数速查表
为了方便后续开发时快速查阅，整理了所有常用内置函数的核心速查表：

| 函数类型       | 函数                                       | 功能                      |
| -------------- | ------------------------------------------ | ------------------------- |
| **字符串函数** | `CONCAT(S1,S2,...Sn)`                      | 字符串拼接                |
|                | `LOWER(str)`                               | 转小写                    |
|                | `UPPER(str)`                               | 转大写                    |
|                | `LPAD(str,n,pad)`                          | 左填充                    |
|                | `RPAD(str,n,pad)`                          | 右填充                    |
|                | `TRIM(str)`                                | 去除首尾空格              |
|                | `SUBSTRING(str,start,len)`                 | 字符串截取（索引从1开始） |
| **数值函数**   | `CEIL(x)`                                  | 向上取整                  |
|                | `FLOOR(x)`                                 | 向下取整                  |
|                | `MOD(x,y)`                                 | 取模                      |
|                | `RAND()`                                   | 生成0~1随机数             |
|                | `ROUND(x,y)`                               | 四舍五入保留y位小数       |
| **日期函数**   | `CURDATE()`                                | 返回当前日期              |
|                | `CURTIME()`                                | 返回当前时间              |
|                | `NOW()`                                    | 返回当前日期和时间        |
|                | `YEAR(date)`                               | 获取年份                  |
|                | `MONTH(date)`                              | 获取月份                  |
|                | `DAY(date)`                                | 获取日                    |
|                | `DATE_ADD(date, INTERVAL expr type)`       | 日期加减                  |
|                | `DATEDIFF(date1,date2)`                    | 计算日期差（date1-date2） |
| **流程函数**   | `IF(value,t,f)`                            | 单条件判断                |
|                | `IFNULL(value1,value2)`                    | 空值处理                  |
|                | `CASE WHEN ... THEN ... ELSE ... END`      | 多条件判断                |
|                | `CASE expr WHEN ... THEN ... ELSE ... END` | 等值判断                  |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163054717>
