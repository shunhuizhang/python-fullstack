


# Python全栈入门到实战【数据库篇 15】MySQL多表连接查询与联合查询详解，跨表数据查询全掌握
上一篇《数据库篇 14》中，我们已经掌握了三种核心多表关系的设计方法，理解了笛卡尔积的概念和消除方法。本篇作为数据库篇的第十五篇，我们将学习**多表查询的具体实现语法**，包括内连接、外连接、自连接三种连接查询，以及union、union all联合查询。这些语法是所有跨表数据查询的基础，也是后端开发中每天都会用到的核心技能。

本文为Python全栈开发者与数据库入门者量身打造，采用"语法+韦恩图+实战示例"的结构，清晰讲解每一种查询类型的原理和使用场景，每一个语法都有可直接运行的SQL代码，同时重点对比易混淆的查询方式，即使是完全没有多表查询经验的同学，也能快速掌握跨表数据查询的核心方法。

本节核心学习内容：
1. 多表查询分类：连接查询与联合查询的整体框架
2. 内连接查询：隐式内连接与显式内连接的语法与区别
3. 外连接查询：左外连接与右外连接的原理与使用场景
4. 自连接查询：同一张表的自关联查询方法
5. 联合查询：UNION与UNION ALL的区别与使用
6. 综合实战：多表查询的组合使用与复杂业务场景实现
7. 常见误区：连接条件遗漏、左右外连接混淆等避坑指南
8. 核心总结：多表查询语法速查表，方便开发时快速查阅

# 一、多表查询分类概述
多表查询是指从多张关联的表中查询数据，主要分为两大类：
- **连接查询**：通过表之间的关联条件，将多张表的行拼接在一起进行查询
  - 内连接：查询两张表的交集部分
  - 外连接：查询一张表的全部数据，以及另一张表中匹配的数据
  - 自连接：将一张表与自身进行连接查询
- **联合查询**：将多个查询的结果集纵向合并成一个新的结果集

# 二、内连接查询
内连接查询的是两张表**交集部分**的数据，也就是同时存在于两张表中的记录。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/416d030ddf924b2ebac23840f5e9ac3b.png#pic_center)


## 2.1 两种语法格式
### 1. 隐式内连接
```sql
SELECT 字段列表 FROM 表1, 表2 WHERE 连接条件 ...;
```

### 2. 显式内连接（推荐使用）
```sql
SELECT 字段列表 FROM 表1 [INNER] JOIN 表2 ON 连接条件 ...;
```

> 💡 最佳实践：推荐使用显式内连接，语法更清晰，连接条件和过滤条件分离，可读性更好。

## 2.2 实战示例
基于之前创建的`emp`员工表和`dept`部门表：
```sql
-- 1. 隐式内连接：查询员工姓名及其所在部门名称
SELECT e.name, d.name 
FROM emp e, dept d 
WHERE e.dept_id = d.id;

-- 2. 显式内连接：查询员工姓名、年龄及其所在部门名称
SELECT e.name, e.age, d.name AS dept_name
FROM emp e INNER JOIN dept d 
ON e.dept_id = d.id;

-- 3. 带过滤条件的内连接：查询研发部所有员工的信息
SELECT e.*, d.name AS dept_name
FROM emp e JOIN dept d 
ON e.dept_id = d.id
WHERE d.name = '研发部';
```

# 三、外连接查询
外连接不仅会查询两张表的交集部分，还会查询其中一张表的全部数据。外连接分为左外连接和右外连接两种。

## 3.1 左外连接
查询**左表的所有数据**，以及右表中与左表匹配的数据。如果右表中没有匹配的数据，则显示NULL。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d5d06b401a2b452b96a5c23a9c3400ac.png#pic_center)


**语法格式**：
```sql
SELECT 字段列表 FROM 表1 LEFT [OUTER] JOIN 表2 ON 连接条件 ...;
```

## 3.2 右外连接
查询**右表的所有数据**，以及左表中与右表匹配的数据。如果左表中没有匹配的数据，则显示NULL。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e33da264b3c54f4db2a1c11971be41e4.png#pic_center)


**语法格式**：
```sql
SELECT 字段列表 FROM 表1 RIGHT [OUTER] JOIN 表2 ON 连接条件 ...;
```

## 3.3 实战示例
```sql
-- 先插入一个没有部门的员工，用于演示外连接
INSERT INTO emp(name, age, dept_id) VALUES ('小龙女', 25, NULL);

-- 1. 内连接：不会显示没有部门的小龙女
SELECT e.name, d.name AS dept_name
FROM emp e JOIN dept d ON e.dept_id = d.id;

-- 2. 左外连接：会显示所有员工，包括没有部门的小龙女
SELECT e.name, d.name AS dept_name
FROM emp e LEFT JOIN dept d ON e.dept_id = d.id;

-- 3. 右外连接：会显示所有部门，包括没有员工的部门
SELECT e.name, d.name AS dept_name
FROM emp e RIGHT JOIN dept d ON e.dept_id = d.id;
```

# 四、自连接查询
自连接是将**同一张表与自身进行连接查询**，本质上还是内连接或外连接，只是将一张表当成两张不同的表来使用。

## 4.1 语法格式
```sql
SELECT 字段列表 FROM 表A 别名A JOIN 表A 别名B ON 条件 ...;
```

> ⚠️ 注意：自连接必须给表起不同的别名，否则会出现字段名冲突的错误。

## 4.2 实战示例
查询员工及其上级领导的姓名：
```sql
-- 先给员工表添加上级领导ID字段并更新数据
ALTER TABLE emp ADD COLUMN managerid INT COMMENT '上级领导ID';
UPDATE emp SET managerid = 1 WHERE id IN (2,3);
UPDATE emp SET managerid = 2 WHERE id IN (4,5);

-- 自连接查询：员工姓名及其上级领导姓名
SELECT 
    e1.name AS 员工姓名,
    e2.name AS 上级领导姓名
FROM emp e1 LEFT JOIN emp e2 
ON e1.managerid = e2.id;
```

# 五、联合查询
联合查询用于将**多个查询的结果集纵向合并**成一个新的结果集。

## 5.1 语法格式
```sql
SELECT 字段列表 FROM 表A ...
UNION [ALL]
SELECT 字段列表 FROM 表B ...;
```

## 5.2 UNION与UNION ALL的区别
- `UNION`：合并结果集后会自动去除重复的记录
- `UNION ALL`：合并结果集后不会去除重复记录，效率更高

## 5.3 重要注意事项
- 多个查询的**列数必须保持一致**
- 多个查询的**对应字段类型必须保持一致**
- 最终结果集的列名由第一个查询的列名决定

## 5.4 实战示例
```sql
-- 查询薪资低于8000的员工
SELECT name, salary FROM emp WHERE salary < 8000
UNION
-- 查询年龄大于40岁的员工
SELECT name, age FROM emp WHERE age > 40;

-- 使用UNION ALL，不会去除重复记录
SELECT name, salary FROM emp WHERE salary < 8000
UNION ALL
SELECT name, salary FROM emp WHERE salary < 10000;
```

# 六、综合实战演示
下面我们通过几个复杂的业务场景，演示多表查询的组合使用。

## 6.1 需求说明
基于`student`学生表、`course`课程表和`student_course`中间表，完成以下查询：
1. 查询所有学生的姓名、学号及其选修的课程名称
2. 查询没有选修任何课程的学生
3. 查询所有课程的名称及其选修人数
4. 查询同时选修了Java和MySQL课程的学生

## 6.2 实现代码
```sql
-- 1. 查询所有学生的姓名、学号及其选修的课程名称
SELECT 
    s.name AS 学生姓名,
    s.no AS 学号,
    c.name AS 课程名称
FROM student s
LEFT JOIN student_course sc ON s.id = sc.studentid
LEFT JOIN course c ON sc.courseid = c.id;

-- 2. 查询没有选修任何课程的学生
SELECT s.*
FROM student s
LEFT JOIN student_course sc ON s.id = sc.studentid
WHERE sc.id IS NULL;

-- 3. 查询所有课程的名称及其选修人数
SELECT 
    c.name AS 课程名称,
    COUNT(sc.id) AS 选修人数
FROM course c
LEFT JOIN student_course sc ON c.id = sc.courseid
GROUP BY c.id, c.name;

-- 4. 查询同时选修了Java和MySQL课程的学生
SELECT s.name
FROM student s
JOIN student_course sc1 ON s.id = sc1.studentid
JOIN course c1 ON sc1.courseid = c1.id AND c1.name = 'Java'
JOIN student_course sc2 ON s.id = sc2.studentid
JOIN course c2 ON sc2.courseid = c2.id AND c2.name = 'MySQL';
```

# 七、常见误区与避坑指南
1. **忘记添加连接条件**：这是最常见的错误，会导致笛卡尔积，产生大量无效数据，严重影响查询性能。
2. **混淆左外连接和右外连接**：记住"左外连接查左表全部，右外连接查右表全部"，推荐统一使用左外连接，逻辑更清晰。
3. **自连接没有起别名**：自连接必须给表起不同的别名，否则会出现字段名冲突的错误。
4. **联合查询列数不一致**：多个查询的列数和对应字段类型必须一致，否则会报错。
5. **滥用UNION**：如果不需要去重，优先使用`UNION ALL`，效率更高。
6. **多表查询字段名冲突**：当多张表有相同的字段名时，必须使用"表名.字段名"的方式指定，否则会报错。

# 八、核心总结：多表查询语法速查表
为了方便后续开发时快速查阅，整理了所有多表查询的核心语法速查表：

| 查询类型             | 语法格式                                      | 核心要点                     |
| -------------------- | --------------------------------------------- | ---------------------------- |
| **隐式内连接**       | `SELECT ... FROM 表1,表2 WHERE 条件;`         | 查询交集，语法简洁           |
| **显式内连接**       | `SELECT ... FROM 表1 JOIN 表2 ON 条件;`       | 查询交集，推荐使用，可读性好 |
| **左外连接**         | `SELECT ... FROM 表1 LEFT JOIN 表2 ON 条件;`  | 查询左表全部+交集            |
| **右外连接**         | `SELECT ... FROM 表1 RIGHT JOIN 表2 ON 条件;` | 查询右表全部+交集            |
| **自连接**           | `SELECT ... FROM 表A a JOIN 表A b ON 条件;`   | 同一张表自关联，必须起别名   |
| **联合查询(去重)**   | `SELECT ... UNION SELECT ...;`                | 纵向合并结果集，自动去重     |
| **联合查询(不去重)** | `SELECT ... UNION ALL SELECT ...;`            | 纵向合并结果集，效率更高     |

# 九、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163130625>
