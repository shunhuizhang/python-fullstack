


# Python全栈入门到实战【数据库篇 07】MySQL DML数据操作详解（增删改），数据库核心操作必掌握
上一篇《数据库篇 06》中，我们已经学会了使用PyCharm作为MySQL的图形化工具，实现了数据库的连接、SQL执行和表数据可视化编辑。本篇作为数据库篇的第七篇，我们将学习**MySQL最核心、最常用的DML数据操纵语言**，掌握数据的添加、修改和删除操作，这是所有后端开发和数据库应用的基础，也是后续学习ORM框架的必备知识。

本文为Python全栈开发者与数据库入门者量身打造，采用"语法+示例+注意事项"的结构，每一个语法都有清晰的格式标注和实战演示，同时重点标注新手最容易踩的坑，即使是完全没有SQL基础的同学，也能快速掌握DML数据操作的核心方法。

本节核心学习内容：
1. DML语言概述：作用与核心分类
2. 添加数据：指定字段、全字段、批量添加三种语法
3. 修改数据：UPDATE语法与条件过滤
4. 删除数据：DELETE语法与使用限制
5. 完整实战：基于用户表的增删改操作演示
6. 常见误区：无条件操作风险、语法错误排查
7. 核心总结：DML语法速查表，方便开发时快速查阅

# 一、DML语言概述
DML（Data Manipulation Language，数据操纵语言）是用于对数据库表中的**数据本身**进行操作的SQL语言，是日常开发中使用频率最高的SQL语句类型。

DML主要包含三个核心语句：
- `INSERT`：向表中添加新的数据行
- `UPDATE`：修改表中已存在的数据
- `DELETE`：删除表中不需要的数据

> ⚠️ 注意：DML操作的是表中的**数据**，而不是表的结构。修改表结构需要使用DDL语言（如`CREATE TABLE`、`ALTER TABLE`等）。

# 二、添加数据（INSERT）
添加数据是最基础的数据库操作，MySQL提供了三种添加数据的方式，分别适用于不同的场景。



## 2.1 给指定字段添加数据
当只需要给表中的部分字段赋值时，使用指定字段的语法，未指定的字段会使用默认值（如果设置了默认值）或NULL。

**语法格式**：
```sql
INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...);
```

**实战示例**：
```sql
-- 向user表中添加一条数据，只指定id、name、age三个字段
INSERT INTO user (id, name, age) VALUES (1, '张三', 20);
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bb08ce8cebae4249be3204f2e7312422.png#pic_center)


## 2.2 给全部字段添加数据

当需要给表中的所有字段都赋值时，可以省略字段名，直接按照表结构中字段的顺序依次赋值。

**语法格式**：
```sql
INSERT INTO 表名 VALUES (值1, 值2, ...);
```

**实战示例**：
```sql
-- 向user表中添加一条数据，给所有字段赋值
INSERT INTO user VALUES (2, '李四', 22);
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1f7b130d484749acb53aec9cda197564.png#pic_center)


## 2.3 批量添加数据

当需要一次性添加多条数据时，使用批量添加语法，比逐条执行INSERT语句效率高得多。

**语法格式1（指定字段批量添加）**：
```sql
INSERT INTO 表名 (字段名1, 字段名2, ...) 
VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);
```

**语法格式2（全字段批量添加）**：
```sql
INSERT INTO 表名 
VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);
```

**实战示例**：
```sql
-- 批量向user表中添加三条数据
INSERT INTO user (id, name, age) 
VALUES (3, '王五', 21), (4, '赵六', 23), (5, '孙七', 20);
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ea75003c659744828b78b0dda08b6a69.png#pic_center)


# 三、修改数据（UPDATE）

修改数据用于更新表中已存在的记录，可以同时修改一个或多个字段的值，通常需要配合`WHERE`条件使用，只修改符合条件的记录。



## 3.1 基本语法
```sql
UPDATE 表名 SET 字段名1 = 值1, 字段名2 = 值2, ... [WHERE 条件];
```

## 3.2 实战示例
```sql
-- 修改id为1的用户的年龄为21岁
UPDATE user SET age = 21 WHERE id = 1;

-- 同时修改多个字段：将id为2的用户的姓名改为"李思思"，邮箱改为"lisisi@example.com"
UPDATE user SET name = '李思思', email = 'lisisi@example.com' WHERE id = 2;

```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ace801159aee43e5a1caed68e66a6f66.png#pic_center)


## 3.3 重要注意事项

> 🚨 **高危警告**：修改语句的条件可以有，也可以没有。**如果没有WHERE条件，则会修改整张表的所有数据**，这是非常危险的操作，生产环境中绝对禁止使用无条件的UPDATE语句！

# 四、删除数据（DELETE）
删除数据用于删除表中不需要的记录，同样需要配合`WHERE`条件使用，只删除符合条件的记录。



## 4.1 基本语法
```sql
DELETE FROM 表名 [WHERE 条件];
```

## 4.2 实战示例
```sql
-- 删除id为5的用户
DELETE FROM user WHERE id = 5;

-- 删除所有年龄大于25岁的用户（谨慎使用！）
DELETE FROM user WHERE age > 25;
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/438495cfb87a4b139ac7d1f1dcd6b4c2.png#pic_center)


## 4.3 重要注意事项

1. 🚨 **高危警告**：DELETE语句的条件可以有，也可以没有。**如果没有WHERE条件，则会删除整张表的所有数据**，生产环境中绝对禁止使用无条件的DELETE语句！
2. DELETE语句**不能删除某一个字段的值**，只能删除整行数据。如果需要清空某个字段的值，应该使用UPDATE语句将其设置为NULL或空字符串：
   ```sql
   -- 正确：清空id为3的用户的邮箱
   UPDATE user SET email = NULL WHERE id = 3;
   ```

# 五、完整实战演示
下面我们基于一个完整的`student`学生表，演示DML增删改操作的完整流程：

## 5.1 准备测试表
首先在PyCharm的Console中执行以下SQL，创建测试用的student表：
```sql
CREATE TABLE student (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20) NOT NULL,
    age INT,
    gender CHAR(1),
    class VARCHAR(20)
);
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1072913ad5d74c6b88cd25c99f8622f2.png#pic_center)


## 5.2 添加数据

```sql
-- 给指定字段添加数据
INSERT INTO student (name, age, class) VALUES ('小明', 18, '高一1班');

-- 全字段添加数据
INSERT INTO student VALUES (2, '小红', 17, '女', '高一2班');

-- 批量添加数据
INSERT INTO student (name, age, gender, class)
VALUES ('小刚', 18, '男', '高一1班'),
       ('小丽', 17, '女', '高一3班'),
       ('小强', 18, '男', '高一2班');
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/236cd40d2bd347028ba99cd49b501bd2.png#pic_center)


## 5.3 修改数据

```sql
-- 将小明的年龄修改为19岁
UPDATE student SET age = 19 WHERE name = '小明';

-- 将高一1班所有学生的班级改为"高一实验班"
UPDATE student SET class = '高一实验班' WHERE class = '高一1班';
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4fdf5bb177ad415392b0472540404067.png#pic_center)


## 5.4 删除数据

```sql
-- 删除id为5的小丽同学
DELETE FROM student WHERE id = 5;

-- 删除所有年龄小于17岁的学生
DELETE FROM student WHERE age < 17;
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/64e532b882474174a67a910dfe859452.png#pic_center)


# 六、常见误区与避坑指南

1. **无条件操作风险**：永远不要在生产环境中执行没有WHERE条件的UPDATE或DELETE语句，否则会导致整张表的数据被修改或删除，造成不可挽回的损失。
2. **字段与值类型不匹配**：字符串类型的值必须用单引号包裹，数字类型的值不需要用单引号。
3. **批量添加语法错误**：批量添加时，每条数据用括号包裹，多条数据之间用逗号分隔，最后一条数据后面不要加逗号。
4. **DELETE与TRUNCATE混淆**：DELETE是删除表中的数据，表结构保留；TRUNCATE是清空整张表并重置自增主键，速度更快，但无法回滚。
5. **主键重复错误**：如果手动指定主键值，要确保主键值不重复，否则会报错。

# 七、核心总结：DML语法速查表
为了方便后续开发时快速查阅，整理了DML增删改操作的核心语法速查表：

| 操作类型     | 语法格式                                          | 核心要点                                        |
| ------------ | ------------------------------------------------- | ----------------------------------------------- |
| 指定字段添加 | `INSERT INTO 表名(字段1,字段2) VALUES(值1,值2);`  | 未指定字段使用默认值或NULL                      |
| 全字段添加   | `INSERT INTO 表名 VALUES(值1,值2,...);`           | 值的顺序必须与表结构一致                        |
| 批量添加     | `INSERT INTO 表名 VALUES(...),(...),(...);`       | 效率远高于逐条添加                              |
| 修改数据     | `UPDATE 表名 SET 字段1=值1,字段2=值2 WHERE 条件;` | 必须加WHERE条件，否则修改全表                   |
| 删除数据     | `DELETE FROM 表名 WHERE 条件;`                    | 必须加WHERE条件，否则删除全表；不能删除单个字段 |

# 八、专栏订阅
> - <font color=black size=3>==专栏优点？==《Python从入门到实战》，专栏内容涵盖：Python基础到高级编程、并发编程（进程/线程/协程）、网络编程（TCP/UDP/Socket）、核心内置/第三方模块、数据库核心实战、Web开发（Django/Flask/FastAPI框架）、数据库（MySQL/ORM/异步数据库）、网络爬虫（同步/异步/分布式）、AI实战、Linux部署运维等全栈核心知识，以项目驱动教学，构建清晰学习路径，适合零基础入门和进阶提升的同学，跟着一步步从入门到精通！专栏地址：<https://blog.csdn.net/zsh_1314520/category_13108073.html></font>
> - <font color=black size=3>==文章是永久吗？== 一次订阅后可永久免费查看专栏内所有文章，后续会持续更新全栈相关内容，第一时间获取最新教程！</font>
> - <font color=black size=3>==有答疑交流群吗？== 订阅专栏后有专属的全栈学习答疑群，群内提供专业问题答疑、和众多学习者抱团取暖，一起沉淀技术、赋能成长！</font>
> - <font color=black size=3>==进群方式？== 订阅专栏后可直接在专栏内申请加入答疑群，或私信博主沟通进群事宜：<https://bbs.csdn.net/topics/620104702></font>
> - <font color=black size=3>==更多干货？== 点赞+收藏+关注博主不迷路！博主博客链接：<https://blog.csdn.net/zsh_1314520?spm=1000.2115.3001.5343>，专注Python全栈技术分享，评论区留言问题会一一回复，助力大家轻松搞定Python全栈！</font>

【原创声明】

除本文原文地址以外，如发现同款内容皆为盗版，本文已收录于《[Python全栈：从入门到实战](https://blog.csdn.net/zsh_1314520/category_13108073.html)》 ，请勿购买盗版文章和专栏，如购买盗版内容不提供任何服务。原文地址：<https://blog.csdn.net/zsh_1314520/article/details/162210211>
