

# Python全栈入门到实战【数据库篇 16】MySQL子查询详解，嵌套查询的四种类型与实战
上一篇《数据库篇 15》中，我们已经掌握了内连接、外连接、自连接和联合查询，学会了如何将多张表的数据横向或纵向拼接查询。本篇作为数据库篇的第十六篇，我们将学习**另一种强大的多表查询方式——子查询**。子查询允许我们在一个SQL语句中嵌套另一个SELECT语句，将复杂的查询分解为多个简单的步骤，大幅提升SQL的可读性和灵活性，是解决复杂业务查询的核心技能。

本文为Python全栈开发者与数据库入门者量身打造，系统讲解标量、列、行、表四种类型的子查询，每一种类型都有清晰的语法说明和可直接运行的实战示例，同时覆盖子查询在不同位置的使用方法，重点标注新手最容易踩的坑，即使是完全没有复杂查询经验的同学，也能快速掌握子查询的核心用法。

本节核心学习内容：
1. 子查询概述：概念、作用与核心分类
2. 标量子查询：返回单个值的子查询与常用操作符
3. 列子查询：返回一列多行的子查询与操作符详解
4. 行子查询：返回一行多列的子查询与使用场景
5. 表子查询：返回多行多列的子查询与临时表用法
6. 不同位置的子查询：WHERE、FROM、SELECT之后的子查询
7. 综合实战：复杂业务场景的子查询解决方案
8. 常见误区：子查询的常见错误与性能优化建议
9. 核心总结：子查询语法速查表，方便开发时快速查阅

# 一、子查询概述
## 1.1 基本概念
SQL语句中嵌套SELECT语句，称为**嵌套查询**，又称**子查询**。子查询可以将一个复杂的查询分解为多个简单的步骤，先执行内部的子查询，再将子查询的结果作为外部查询的条件或数据源。

**基本语法格式**：
```sql
SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2);
```

## 1.2 子查询的适用范围
子查询外部的语句可以是`INSERT`、`UPDATE`、`DELETE`、`SELECT`中的任何一个，其中最常用的是与`SELECT`语句结合使用。

## 1.3 子查询的分类
### 按子查询返回结果分类
这是最常用的分类方式，根据子查询返回结果的不同，分为四种类型：
- **标量子查询**：子查询返回的结果是**单个值**（数字、字符串、日期等）
- **列子查询**：子查询返回的结果是**一列（可以是多行）**
- **行子查询**：子查询返回的结果是**一行（可以是多列）**
- **表子查询**：子查询返回的结果是**多行多列**

### 按子查询位置分类
根据子查询在SQL语句中出现的位置，分为三种：
- `WHERE`之后：作为过滤条件使用
- `FROM`之后：作为临时表使用
- `SELECT`之后：作为字段值使用

# 二、标量子查询
标量子查询是最简单的子查询形式，返回的结果是单个值（数字、字符串、日期等）。

## 2.1 常用操作符
标量子查询可以使用所有比较运算符：
`=`、`<>`、`>`、`>=`、`<`、`<=`

## 2.2 实战示例
基于之前创建的`emp`员工表和`dept`部门表：
```sql
-- 1. 查询"研发部"的所有员工信息
-- 分步：先查询研发部的部门ID，再查询该部门的员工
SELECT * FROM emp WHERE dept_id = (SELECT id FROM dept WHERE name = '研发部');

-- 2. 查询比"张无忌"年龄大的员工
SELECT * FROM emp WHERE age > (SELECT age FROM emp WHERE name = '张无忌');

-- 3. 查询最晚入职的员工信息
SELECT * FROM emp WHERE entrydate = (SELECT MAX(entrydate) FROM emp);

-- 4. 查询员工人数比"市场部"多的部门
SELECT d.name, COUNT(e.id) AS emp_count
FROM dept d LEFT JOIN emp e ON d.id = e.dept_id
GROUP BY d.id, d.name
HAVING COUNT(e.id) > (SELECT COUNT(*) FROM emp WHERE dept_id = (SELECT id FROM dept WHERE name = '市场部'));
```

# 三、列子查询
列子查询返回的结果是一列（可以是多行），通常用于判断某个值是否在子查询返回的集合中。

## 3.1 常用操作符
| 操作符   | 描述                                         |
| -------- | -------------------------------------------- |
| `IN`     | 在指定的集合范围之内，多选一                 |
| `NOT IN` | 不在指定的集合范围之内                       |
| `ANY`    | 子查询返回列表中，有任意一个满足即可         |
| `SOME`   | 与`ANY`等同，使用`SOME`的地方都可以使用`ANY` |
| `ALL`    | 子查询返回列表的所有值都必须满足             |

## 3.2 实战示例
```sql
-- 1. 查询"研发部"和"市场部"的所有员工信息
SELECT * FROM emp WHERE dept_id IN (SELECT id FROM dept WHERE name IN ('研发部', '市场部'));

-- 2. 查询不是"财务部"和"总经办"的员工信息
SELECT * FROM emp WHERE dept_id NOT IN (SELECT id FROM dept WHERE name IN ('财务部', '总经办'));

-- 3. 查询比研发部任意一个员工年龄都大的员工
SELECT * FROM emp WHERE age > ANY (SELECT age FROM emp WHERE dept_id = (SELECT id FROM dept WHERE name = '研发部'));

-- 4. 查询比研发部所有员工年龄都大的员工
SELECT * FROM emp WHERE age > ALL (SELECT age FROM emp WHERE dept_id = (SELECT id FROM dept WHERE name = '研发部'));
```

# 四、行子查询
行子查询返回的结果是一行（可以是多列），通常用于判断多个字段是否同时满足某个条件。

## 4.1 常用操作符
`=`、`<>`、`IN`、`NOT IN`

## 4.2 实战示例
```sql
-- 查询和"张无忌"同部门且同年龄的员工信息
SELECT * FROM emp 
WHERE (dept_id, age) = (SELECT dept_id, age FROM emp WHERE name = '张无忌');

-- 查询和"赵敏"同部门且同薪资的员工信息
SELECT * FROM emp 
WHERE (dept_id, salary) IN (SELECT dept_id, salary FROM emp WHERE name = '赵敏');
```

# 五、表子查询
表子查询返回的结果是多行多列，通常用在`FROM`之后，作为临时表使用。

## 5.1 常用操作符
`IN`

## 5.2 重要注意事项
表子查询作为临时表使用时，**必须给临时表起一个别名**，否则会报错。

## 5.3 实战示例
```sql
-- 1. 查询每个部门的平均年龄，然后查询平均年龄大于20的部门
SELECT * FROM (
    SELECT d.name AS dept_name, AVG(e.age) AS avg_age
    FROM dept d LEFT JOIN emp e ON d.id = e.dept_id
    GROUP BY d.id, d.name
) AS temp_dept_avg
WHERE avg_age > 20;

-- 2. 查询每个部门薪资最高的员工信息
SELECT e.*, d.name AS dept_name
FROM emp e
JOIN dept d ON e.dept_id = d.id
JOIN (
    SELECT dept_id, MAX(salary) AS max_salary
    FROM emp
    GROUP BY dept_id
) AS temp_max_salary
ON e.dept_id = temp_max_salary.dept_id AND e.salary = temp_max_salary.max_salary;
```

# 六、不同位置的子查询
## 6.1 WHERE之后的子查询
这是最常用的子查询位置，作为过滤条件使用，支持标量、列、行子查询。
```sql
-- 示例：查询薪资高于公司平均薪资的员工
SELECT * FROM emp WHERE salary > (SELECT AVG(salary) FROM emp);
```

## 6.2 FROM之后的子查询
作为临时表使用，只能使用表子查询，必须给临时表起别名。
```sql
-- 示例：查询学生的姓名及其选修课程的数量
SELECT s.name, temp.course_count
FROM student s
LEFT JOIN (
    SELECT studentid, COUNT(*) AS course_count
    FROM student_course
    GROUP BY studentid
) AS temp ON s.id = temp.studentid;
```

## 6.3 SELECT之后的子查询
作为字段值使用，只能使用标量子查询。
```sql
-- 示例：查询员工姓名及其所在部门名称
SELECT 
    e.name AS emp_name,
    (SELECT name FROM dept WHERE id = e.dept_id) AS dept_name
FROM emp e;
```

# 七、综合实战演示
下面我们通过几个复杂的业务场景，演示子查询的综合使用。

## 7.1 需求说明
基于`student`学生表、`course`课程表和`student_course`中间表，完成以下查询：
1. 查询选修了"Java"课程的学生姓名
2. 查询没有选修任何课程的学生姓名
3. 查询选修了所有课程的学生姓名
4. 查询至少选修了两门课程的学生姓名

## 7.2 实现代码
```sql
-- 1. 查询选修了"Java"课程的学生姓名
SELECT name FROM student 
WHERE id IN (
    SELECT studentid FROM student_course 
    WHERE courseid = (SELECT id FROM course WHERE name = 'Java')
);

-- 2. 查询没有选修任何课程的学生姓名
SELECT name FROM student 
WHERE id NOT IN (SELECT DISTINCT studentid FROM student_course);

-- 3. 查询选修了所有课程的学生姓名
SELECT name FROM student 
WHERE id IN (
    SELECT studentid FROM student_course
    GROUP BY studentid
    HAVING COUNT(*) = (SELECT COUNT(*) FROM course)
);

-- 4. 查询至少选修了两门课程的学生姓名
SELECT name FROM student 
WHERE id IN (
    SELECT studentid FROM student_course
    GROUP BY studentid
    HAVING COUNT(*) >= 2
);
```

# 八、常见误区与避坑指南
1. **子查询忘记加括号**：子查询必须用括号包裹，这是最常见的语法错误。
2. **标量子查询返回多个值**：标量子查询只能返回单个值，如果返回多个值会报错。
3. **表子查询没有起别名**：表子查询作为临时表使用时，必须给临时表起一个别名。
4. **滥用子查询**：子查询虽然灵活，但嵌套过深会导致性能下降。对于简单的多表查询，优先使用连接查询。
5. **IN子查询的性能问题**：当子查询返回的结果集很大时，`IN`子查询的性能会很差，此时可以使用`JOIN`查询代替。
6. **NULL值的影响**：如果子查询的结果中包含NULL值，`NOT IN`操作符会返回空结果，这是一个容易被忽略的陷阱。

# 九、核心总结：MySQL子查询语法速查表
为了方便后续开发时快速查阅，整理了四种子查询的核心语法速查表：

| 子查询类型     | 返回结果 | 常用操作符             | 使用位置      | 示例                                                         |
| -------------- | -------- | ---------------------- | ------------- | ------------------------------------------------------------ |
| **标量子查询** | 单个值   | `=、<>、>、>=、<、<=`  | WHERE、SELECT | `WHERE age > (SELECT age FROM emp WHERE name='张三')`        |
| **列子查询**   | 一列多行 | `IN、NOT IN、ANY、ALL` | WHERE         | `WHERE dept_id IN (SELECT id FROM dept)`                     |
| **行子查询**   | 一行多列 | `=、<>、IN、NOT IN`    | WHERE         | `WHERE (dept_id,age) = (SELECT dept_id,age FROM emp WHERE name='张三')` |
| **表子查询**   | 多行多列 | `IN`                   | FROM          | `FROM (SELECT ...) AS temp`                                  |

# 十、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/163373495>
