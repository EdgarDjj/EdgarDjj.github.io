# MySQL57学习笔记

# MySQL57
## 概念
**SQL：Structure Query Language (结构化查询语言)**

- 数据（Data）
  数据库中存储的基本对象[图、文、声、像]
- 数据库（DataBase）
  人们收集并取出一个应用所需要的大量数据之后，应将其保存起来，以供进一步加工处理，进一步抽取有用信息。
  数据库就是[长期]存储在计算机内有[组织]的，可[共享]的[大量]数据集合。
- 数据库管理系统（DataBase Management System）
  主要功能：
  1. 数据定义 DDL 数据描述语言
  2. 数据操作 DML：查询，更新 数据操作语言
  3. 数据库的运行，管理：保证数据的安全性，完整性
- 数据系统（Database System）
  <!-- more -->

##### DBMS的产生和发展：  

1. 什么是数据库管理
   - 分类、组织、编码、存储、检索、维护
2. 数据库管理技术的发展过程

- 人工管理
  - 数据不保存
  - 应用程序管理数据
  - 数据不共享
  - 数据不具有独立性
- 文件系统
  - 数据可以长期保存
  - 由文件系统管理数据
  - 数据共享性差，冗余度大
  - 数据独立性差[数据的逻辑结构改变，就必须修改应用程序]
- 数据库系统
  - 数据结构化
  - 数据的共享性高，冗余度低，易扩充
  - 数据的独立性高
  - 数据又DBMS统一管理和控制

##### DBMS的数据控制功能：

1. 数据的安全性保护
2. 数据的完整性检查
3. 并发控制
4. 数据库恢复

##### 数据模型(通常由数据结构、数据操作和完整性约束三部分组成)

作用：抽象、表示、处理现实世界的信息

- 概念模型（信息模型）  
  按用户的观点来对数据和信息建模，主要用于数据库设计
- 逻辑和物理模型
  - 逻辑模型：
        - 层次模型
        - 网状模型
        - 关系模型
        - 面向对象模型
        - 对象关系模型 

**组成要素：**

- 数据结构[对系统的静态特性的描述]  
  数据结构描述数据库的组成对象以及对象之间的联系。
  描述的内容：
  - 与对象的类型、用户、性质有关的
  - 与数据之间联系有关的对象
- 数据操作[对系统的动态特性的描述]
  - 将对数据库中各种对象（型）的实力（值）允许执行的操作的集合，包括操作及有关的操作规则。
- 数据的完整性约束条件[一组完整性规则]
  - 完整性规则用以限定符合数据模型的数据库状态以及状态的变化，以保证数据的正确、有效、相容。

### 分类

* SQL
  - DQL: 数据查询语言 SELECT
  - DML: 数据操作语言 INSERT UPDATE DELETE
  - DDL: 数据定义语言 CREATE DROP

## MySQL指令学习

(环境变量的配置名为 MySQL57)

##### 客户端启动与关闭

    net start MySQL57
    net stop MySQL57

##### 登录和注销

**登录**

>mysql -u[**数据库用户名**] -p[**对应的密码**] -- 用于登录自己本机的MySQL数据库  
>mysql -u[**数据库用户名**] -p[**对应的密码**] -p[**数据库服务端口号**] -h[**数据库服务器的IP地址**] --用于登录云服务器

**退出**  

>\q;  
>exit;  
>quit;

    mysql -uroot -p[密码]
    //连接云服务器
    mysql -uroot -p[密码] -h[ip地址]
    -- 例:
    云服务器
    IP:212.64.91.102   
    PORT:3306  
    USERNAME:root  
    PASSWORD:gaozhu  
    
    mysql -uroot -pgaozhu -h212.64.91.102

## SQL命令

### DQL

显示哪些线程正在运行

    SHOW PROCESSLIST 

显示系统变量信息

    SHOW VARIABLES

查看当前数据库服务器上所有的数据库  
    

    SHOW DATABASES;
    SHOW SECHEMAS;

**SELECT** 用于展示信息，展示select之后的内容，并且展示为结果集（格式）。  
**FROM** 用于指定数据源  

    SELECT [LIST] FROM [TABLE_NAME]


**查询语句**  
SELECT：指定数据源  
FROM：筛选内容  
WHERE：定制并且展示  
GROUP BY: 分组内容 HAVING: 分组条件
ORDER: 数据筛选
LIMIT: 子句，限制结果数量子句
**FROM > WHERE > GROUP BY > (HAVING) > SELECT > ORDER BY > LIMIT**


>FROM table1 left join table2 on 将table1和table2中的数据产生笛卡尔积，生成Temp1
>JOIN table2 所以先是确定表，再确定关联条件
>ON table1.column = table2.columu 确定表的绑定条件 由Temp1产生中间表Temp2
>WHERE 对中间表Temp2产生的结果进行过滤 产生中间表Temp3
>GROUP BY 对中间表Temp3进行分组，产生中间表Temp4
>HAVING 对分组后的记录进行聚合 产生中间表Temp5
>SELECT 对中间表Temp5进行列筛选，产生中间表 Temp6
>DISTINCT 对中间表 Temp6进行去重，产生中间表 Temp7
>ORDER BY 对Temp7中的数据进行排序，产生中间表Temp8
>LIMIT 对中间表Temp8进行分页，产生中间表Temp9

排序：ASC【缺省值】（从小到大），DESC（从大到小）

##### 函数  

- **数值函数**

>abs(x)          -- 绝对值 abs(-10.9) = 10
>format(x, d)    -- 格式化千分位数值 format(1234567.456, 2) = 1,234,567.46
>ceil(x)         -- 向上取整 ceil(10.1) = 11
>floor(x)        -- 向下取整 floor (10.1) = 10
>round(x)        -- 四舍五入去整
>mod(m, n)       -- m%n m mod n 求余 10%3=1
>pi()            -- 获得圆周率
>pow(m, n)       -- m^n
>sqrt(x)         -- 算术平方根
>rand()          -- 随机数
>truncate(x, d)  -- 截取d位小数

- **时间日期函数**

>now(), current_timestamp();     -- 当前日期时间
>current_date();                 -- 当前日期
>current_time();                 -- 当前时间
>date('yyyy-mm-dd hh:ii:ss');    -- 获取日期部分
>time('yyyy-mm-dd hh:ii:ss');    -- 获取时间部分
>date_format('yyyy-mm-dd hh:ii:ss', '%d %y %a %d %m %b %j'); -- 格式化时间
>unix_timestamp();               -- 获得unix时间戳
>from_unixtime();                -- 从时间戳获得时间

- **字符串函数**

>length(string)          -- string长度，字节展示的长度字节数
>char_length(string)     -- string的字符个数
>substring(str, position [,length])      -- 从str的position开始,取length个字符
>replace(str ,search_str ,replace_str)   -- 在str中用replace_str替换search_str
>instr(string ,substring)    -- 返回substring首次在string中出现的位置
>concat(string [,...])   -- 连接字串
>charset(str)            -- 返回字串字符集
>lcase(string)           -- 转换成小写
>left(string, length)    -- 从string2中的左边起取length个字符
>load_file(file_name)    -- 从文件读取内容
>locate(substring, string [,start_position]) -- 同instr,但可指定开始位置
>lpad(string, length, pad)   -- 重复用pad加在string开头,直到字串长度为length
>ltrim(string)           -- 去除前端空格
>repeat(string, count)   -- 重复count次
>rpad(string, length, pad)   --在str后用pad补充,直到长度为length
>rtrim(string)           -- 去除后端空格
>strcmp(string1 ,string2)    -- 逐字符比较两字串大小

- 流程函数

>case when [condition] then result [when [condition] then result ...] [else result] end   多分支
>if(expr1,expr2,expr3)  双分支。

- **聚合函数**

>count()
>sum();
>max();
>min();
>avg();
>group_concat()

- 其他常用函数
  md5();
  default();


以定义划分  

- 预定义函数（MySQL已经定义好的函数）
- 自定义函数（由我们自己根据实际的开发所需要定义的函数）  

以功能进行划分

- 单行函数
- 多行函数（聚合函数，分组函数）

**单行函数**  

    length（）展示的长度是字节数  
    concat（）字符拼接
    upper（）将字符转换为大写
    lower（）将字符转化位小写
    substr（）字符的截取 &nbsp;substr([某一字符串]， [索引], [截取长度]) //若没有填写截取长度，从某一Number到最后的内容 
    instr（）查找某一字符第一次出现的位置  
    trim（）去除字符的内容

**字符相关的函数**
    -- 获取字符的长度 length() 展示的长度是字节数

    SELECT length('nizhdsasa'),length('及你太美');
    
    -- 查询所有员工的名字，按照员工的名字长短正序排序
    
    SELECT e_first_name,e_last_name
    FROM t_employees
    ORDER BY length(e_first_name) DESC;
    
    -- concat() 字符拼接
    SELECT concat('你好','我好','大家好');
    
    -- 展示所有员工表中人的名字
    SELECT concat(e_first_name,' ',e_last_name) FROM t_employees;
    
    -- 讲字符中的内容转换为大写
    SELECT
    concat(('dsadfadsjfhdskljafASFdsfdsafsdaASFdsa'),lower('DSFSADFDSAdfasf'));
    
    -- 截取第10个字母到最后的内容
    SELECT substr('abcdefghijklmnopqrst', 10);
    SELECT substr('abcdefghijklmnopqrst', 10, 5);


    SELECT instr('abcdefghoijklmn', 'a');
    
    -- 去除 你好啊 两端的空格
    SELECT trim('      dasfsa       ');
    
    SELECT trim('a' FROM 'AAAAAaaaa');
    
    -- 在'abc'左边补齐一堆0值得整个字符德内容长度达到10
    SELECT lpad('abc', 10, '0');
    -- 在右边 
    SELECT rpad('abc', 10, '0');
    
    SELECT rpad('abcdefghijklmnopqrst', 10, '0');
    
    -- 轻轻的我走了，正如我轻轻地来了，我挥一挥衣袖不带走一片云彩。把这段内容中的’轻轻’ 替换为 ‘骚骚’
    SELECT
    replace('轻轻的我走了，正如我轻轻地来了，我挥一挥衣袖不带走一片云彩。','轻轻','骚骚');

**数值相关的函数**
    -- 绝对值
    SELECT abs(-100);

    -- 四舍五入
    SELECT round(3.1415926);
    
    -- 向上取整向下取整
    SELECT ceil(0.81111111);
    SELECT floor(32131.4324324354655456);
    
    -- 取模
    SELECT mod(10, 3), 10%3;
    
    -- 截断小数
    SELECT truncate(3.1415926,3);

**日期相关的函数**
序号 | 格式符 | 功能
|-|-|-
1|%Y|四位数组成的年份
2|%y|两位数组成的年份
3|%m|月份（01,02,03,04...,11,12）
4|%c|月份（1,2,3,4,5...11,12）
5|%d|日（01,02....）
6|%H|小时（24小时制）
7|%h|小时（12小时制）
8|%i|分钟（00,01...59）
9|%s|秒钟（00,01...59）

    -- 获取当前时间(获取的是数据库的时间）
    SELECT now();
    
    -- 获取当前的日期 [日期：年月日][时间：时分秒]
    SELECT curdate(),curtime();
    
    -- yyyy-MM-dd HH:mm:ss 分离时间字段
    SELECT 
    year('2023-6-12 12:12:32'),
    month('2023-6-12 12:12:32'),
    monthname('2023-6-12 12:12:32'),
    day('2023-6-12 12:12:32'),
    minute('2023-6-12 12:12:32'),
    second('2023-6-12 12:12:32');
    
    -- 比较两个时间的差值
    SELECT
    datediff('2000-5-17 00:00:00', now());
    
    -- 将当前的日期以中文的形式展现出来
    SELECT
    data_format(now(),'%Y年%c月%d日%H时%i分%s秒');
    
    -- 将字符串的形式的时间表达转换为时间对象
    SELECT
    str_to_date('');


#### 连接查询(join)

**连接查询:**

1. 内连接
2. 外连接
   * 左外连接
   * 右外连接
3. 全外连接
   **将多个表的字段进行连接，可以指定连接条件。**

- 内连接(inner join)

  - 默认就是内连接，可省略inner。

  - 只有数据存在时才能发送连接。即连接结果不能出现空行。
    on 表示连接条件。其条件表达式与where类似。也可以省略条件（表示条件永远为真）
    也可用where表示连接条件。
    还有 using, 但需字段名相同。 using(字段名)

  - 交叉连接 cross join
    即，没有条件的内连接。
     select * from tb1 cross join tb2;  

         /*
             使用lesson_db1数据库
             查询所有有部门有职位的员工的名字，及其部门名称和其职位名称。
         */
         SHOW CREATE TABLE t_employees;
         -- SQL92
         SELECT E.e_first_name,D.d_name,j_title
         FROM t_employees E,t_jobs J,t_department D
         WHERE E.e_department_id=D.d_id
         AND E.e_job_id=j_id;
         -- SQL99
         SELECT GROUP_CONCAT(TE.e_first_name,' ',TE.e_last_name),TD.d_name,TE.e_job_id,TJ.j_title
         FROM t_employees TE
         INNER JOIN t_department TD ON TE.e_department_id=TD.d_id
         INNER JOIN t_jobs TJ ON TE.e_job_id = TJ.j_id
         GROUP BY e_department_id;

- 外连接(outer join)

  - 如果数据不存在，也会出现在连接结果中。

  - 左外连接 left join
    如果数据不存在，左表记录会出现，而右表为null填充

          -- 展示所有的女生的姓名，如果有男朋友的也展示其那男朋友的名字，否则不展示。
          SELECT g_name,b_name
          FROM
          t_girls g LEFT JOIN t_boys b ON g.g_boyfriend_id=b.b_id;

  - 右外连接 right join
    如果数据不存在，右表记录会出现，而左表为null填充

- 自然连接(natural join)
  自动判断连接条件完成连接。
  相当于省略了using，会自动查找相同字段名。
  natural join
  natural left join
  natural right join

- 自连接

      -- 展示一个员工的员工名字和他领导的名字
      SELECT ta.e_first_name,tb.e_first_name
      FROM t_employees ta
      LEFT JOIN t_employees tb ON ta.e_manager_id=tb.e_id;

#### 子查询

在一个语句中，嵌套查询语句。  
a -> b (b) 是一个查询语句  

**结果集的形态**

1. 标量形态：有且只有一个值得叫做标量形态
2. 列形态：一列数据，多行。
3. 行形态：多行多列的结果集。


### DML

##### INSERT

>select语句获得的数据可以用insert插入。

1. 可以省略对列的指定，要求 values () 括号内，提供给了按照列顺序出现的所有字段的值。或者使用set语法。
   `INSERT INTO tbl_name SET field=value,...；`
2. 可以一次性使用多个值，采用(), (), ();的形式。
   `INSERT INTO tbl_name VALUES (), (), ();`
3. 可以在列值指定时，使用表达式。
   `INSERT INTO tbl_name VALUES (field_value, 10+10, now());`
4. 可以使用一个特殊值 DEFAULT，表示该列使用默认值。
   `INSERT INTO tbl_name VALUES (field_value, DEFAULT);`
5. 可以通过一个查询的结果，作为需要插入的值。
   `INSERT INTO tbl_name SELECT ...;`

##### UPDATE

**UPDATE 表名 SET 字段名=新值[, 字段名=新值] [更新条件]**
多表连接更改

    UPDATE t_employees te
    LEFT JOIN t_jobs tj on te.e_job_id=tj.j_id
    LEFT JOIN t_department td on te.e_department_id=td.d_id
    SET te.e_first_name='高柱',tj.j_title='sapo',td.d_name='sh'
    WHere te.e_id='102';

##### DELETE

**DELETE FROM 表名[ 删除条件子句]**

>没有条件子句，则会删除全部

多表连接删除

    DELETE tg,tb
    FROM t_girls tg
    LEFT JOIN t_boys tb on tg.g_boyfriend_id=tb.b_id
    WHERE g_name='周芷若';

### DDL

##### 库的创建

- 创建库
  `CREATE DATABASE[ IF NOT EXISTS] 数据库名 数据库选项`

  * 数据库选项：
    CHARACTER SET charset_name
    COLLATE collation_name

  > 创建数据库并指定字符集(加上``解决关键字冲突问题)
  > CREATE DATABASE `INT` CHARACTER SET utf8;

- 删除库

  * 删除
    DROP FUNCTION [IF EXISTS] function_name;

- 更改库
  只能更库选项，不可更改名字（以前可以）
  `ALTER DATABASE djj CHARACTER SET GBK;`

##### 数据类型

1. **数值类型**

   - a. 整型  

   | 类型      | 字节  | 范围（有符号位）   |                   |
   | --------- | ----- | ------------------ | ----------------- |
   | tinyint   | 1字节 | -128 ~ 127         | 无符号位：0 ~ 255 |
   | smallint  | 2字节 | -32768 ~ 32767     |                   |
   | mediumint | 3字节 | -8388608 ~ 8388607 |                   |
   | int       | 4字节 |                    |                   |
   | bigint    | 8字节 |                    |                   |

    int(M)  M表示总位数

    - 默认存在符号位，unsigned 属性修改
    - 显示宽度，如果某个数不够定义字段时设置的位数，则前面以0补填，zerofill 属性修改(则变为无符号形式)
      例：int(5)   插入一个数'123'，补填后为'00123'
    - 在满足要求的情况下，越小越好。
    - 1表示bool值真，0表示bool值假。MySQL没有布尔类型，通过整型0和1表示。常用tinyint(1)表示布尔型。

   - b. 浮点型

   | 类型           | 字节  | 范围 |
   | -------------- | ----- | ---- |
   | float(单精度)  | 4字节 |      |
   | double(双精度) | 8字节 |      |

    浮点型既支持符号位 unsigned 属性，也支持显示宽度 zerofill 属性。
    不同于整型，前后均会补填0.
    定义浮点型时，需指定总位数和小数位数。
    float(M, D)     double(M, D)
    M表示总位数，D表示小数位数。
    M和D的大小会决定浮点数的范围。不同于整型的固定范围。
    **插入的数据小数位大于D，四舍五入，大于5则不可插入报错，小于5的则会被忽略。**
        M既表示总位数（不包括小数点和正负号），也表示显示宽度（所有显示符号均包括）。
        支持科学计数法表示。
        浮点数表示近似值。

   - c. 定点数
     decimal -- 可变长度
      decimal(M, D)   M也表示总位数，D表示小数位数。
      保存一个精确的数值，不会发生数据的改变，不同于浮点数的四舍五入。
      将浮点数转换为字符串来保存，每9位数字保存为4个字节。

2. **字符串类型**

   - a. char, varchar
     * char    定长字符串，速度快，但浪费空间
     * varchar 变长字符串，速度慢，但节省空间
   - varchar(M)&nbsp;&nbsp;&nbsp;M表示能存储的最大长度，此长度是字符数，非字节数。
     不同的编码，所占用的空间不同。
      char,最多255个字符，与编码无关。
      varchar,最多65535字符，与编码有关。
      一条有效记录最大不能超过65535个字节。
       utf8 最大为21844个字符，gbk 最大为32766个字符，latin1 最大为65532个字符

    >varchar 是变长的，需要利用存储空间保存 varchar 的长度，如果数据小于255个字节，则采用一个字节来保存长度，反之需要两个字节来保存。
    > varchar 的最大有效长度由最大行大小和使用的字符集确定。
    > 最大有效长度是65532字节，因为在varchar存字符串时，第一个字节是空的，不存在任何数据，然后还需两个字节来存放字符串的长度，所以有效长度是64432-1-2=65532字节。

    >例：若一个表定义为 CREATE TABLE tb(c1 int, c2 char(30), c3 varchar(N)) charset=utf8; 问N的最大值是多少？ 答：(65535-1-2-4-30*3)/3

   - b. blob, text ----------
     blob 二进制字符串（字节字符串）
       tinyblob, blob, mediumblob, longblob
      text 非二进制字符串（字符字符串）
       tinytext, text, mediumtext, longtext
      text 在定义时，不需要定义长度，也不会计算总长度。
      text 类型在定义时，不可给default值

    - c. binary, varbinary ----------
      类似于char和varchar，用于保存二进制字符串，也就是保存字节字符串而非字符字符串。
       char, varchar, text 对应 binary, varbinary, blob.

3. **日期时间类型**
   一般用整型保存时间戳，因为PHP可以很方便的将时间戳进行格式化。

   | 类型      | 字节数 | 意义       | 时间范围                                   |
   | --------- | ------ | ---------- | ------------------------------------------ |
   | datetime  | 8字节  | 日期及时间 | 1000-01-01 00:00:00 到 9999-12-31 23:59:59 |
   | date      | 3字节  | 日期       | 1000-01-01 到 9999-12-31                   |
   | timestamp | 4字节  | 时间戳     | 19700101000000 到 2038-01-19 03:14:07      |
   | time      | 3字节  | 时间       | -838:59:59 到 838:59:59                    |
   | year      | 1字节  | 年份       | 1901 - 2155                                |

##### 表的创建

    CREATE TABLE IF NOT EXISTS `t_person`(
    `p_id` INT COMMENT '人员编号',
    `p_name` VARCHAR(20) COMMENT '人员姓名',
    `p_sex` ENUM('male','female') COMMENT '人员性别', -- 存储的值要么是男，要么是女，如果是其他的值则无法保存。
    `p_height` FLOAT(5,2) COMMENT '身高',
    `p_weight` FLOAT(5,2) COMMENT '体重',
    `p_birthday` DATETIME COMMENT '出生日期'
    )CHARACTER SET UTF8 COMMENT '人员信息表';

- **非空约束** NOT NULL和NULL

  >-- 非空约束：约束在插入新的数据的时候，被约束的字段必须给出值
  >NOT NULL该项不能为空

- **默认约束** DEFAULT

  >-- 默认约束：如果在插入数据的过程中，如果没有给t_person2t_person2这个字段值，那么其会采用默认值(null)。

- **重复约束** UNIQUE

  >某个字段的值，不可重复出现。

- **检查约束** CHECK（MySQL不支持）

  >指的是给字段设定规则，要求其插入的值的过程中必须满足这个要 求。

- **主键约束** PRIMARY KEY

  >- 能唯一标识记录的字段，可以作为主键。
  >- 一个表只能有一个主键。
  >- 主键具有唯一性。
  >- 声明字段时，用 primary key 标识。
  >  也可以在字段列表之后声明
  >  例：create table tab ( id int, stu varchar(10), primary key (id));
  >- 主键字段的值不能为null。
  >- 主键可以由多个字段共同组成。此时需要在字段列表后声明的方法。
  >  例：create table tab ( id int, stu varchar(10), age int, primary key (stu, age));

- **外键约束** FOREIGN KEY

  >被外键约束的列，其数据来源需要来源于主表。用于限制主表与从表数据完整性。
  >只支持列约束写法。
  > 语法：
  >`FOREIGN KEY` (外键字段） `REFERENCES` 主表名 (关联字段) [主表记录删除时的动作] [主表记录更新时的动作]
  >每个外键都有一个名字可以通过`CONSTRAINT`指定。
  >此时需要检测一个从表的外键需要约束为主表的已存在的值。外键在没有关联的情况下，可以设置为null.前提是该外键列，没有not null。

  >外键约束：外键表插入数据的时候，引用的值必须首先在主表中存在。不然无法插入。
  >删除表的时候，必须先删除外键表，不然主键表无法被删除
  >逐渐表中，已经被外键表引用了的数据，除非先删除外键表中引用的数据，不然逐渐表中被引用的数据无法被直接删除。且外键字段与关联字段数据类型和字段长度必须相同。

列约束写法：将约束的内容直接的写在列的定义上。  
列约束的写法，支持六种约束，除了、

1. 检查约束（MySQL不支持）
2. 将字段的约束定义在列的定义上
3. 列级约束支持所有的约束（六种）
4. 外键约束本身不支持列约束的语法（MySQL）只能在表约束的语法中写入。

表约束写法：

1. 可添加复合主键  
2. 表约束可以支持外键约束
3. 默认 非空 写在表级约束上无效

##### 表的更改

- **修改表本身的选项**
      `ALTER TABLE 表名 表的选项`
      eg: ALTER TABLE 表名 ENGINE=MYISAM;

  - 修改表本身的选项
    ALTER TABLE 表名 表的选项
    eg: ALTER TABLE 表名 ENGINE=MYISAM;

  - 对表进行重命名
    RENAME TABLE 原表名 TO 新表名
    RENAME TABLE 原表名 TO 库名.表名 （可将表移动到另一个数据库）
    -- RENAME可以交换两个表名

  - 修改表的字段结构
    ` ALTER TABLE 表名 操作名`

     - 操作名

       | 操作                                       | 作用                                                         |
       | ------------------------------------------ | ------------------------------------------------------------ |
       | ADD[ COLUMN] 字段定义                      | -- 增加字段                                                  |
       | ADD PRIMARY KEY(字段名)                    | -- 创建主键                                                  |
       | ADD UNIQUE [索引名] (字段名)               | -- 创建唯一索引                                              |
       | ADD INDEX [索引名] (字段名)                | -- 创建普通索引                                              |
       | DROP[ COLUMN] 字段名                       | -- 删除字段                                                  |
       | MODIFY[ COLUMN] 字段名 字段属性            | -- 支持对字段属性和约束进行修改，不能修改字段名(所有原有属性也需写上) |
       | CHANGE[ COLUMN] 原字段名 新字段名 字段属性 | -- 支持对字段名修改                                          |
       | DROP PRIMARY KEY                           | -- 删除主键(删除主键前需删除其AUTO_INCREMENT属性)            |
       | DROP INDEX 索引名                          | -- 删除索引                                                  |
       | DROP FOREIGN KEY 外键                      | -- 删除外键                                                  |

SHOW VARIABLES LIKE '%auto_increment%';

### 视图

##### 什么是视图：

    >视图是一个虚拟表，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。
    视图具有表结构文件，但不存在数据文件。
    对其中所引用的基础表来说，视图的作用类似于筛选。定义视图的筛选可以来自当前或其它数据库的一个或多个表，或者其它视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。
    视图是存储在数据库中的查询的sql语句，它主要出于两种原因：安全原因，视图可以隐藏一些数据，如：社会保险基金表，可以用视图只显示姓名，地址，而不显示社会保险号和工资数等，另一原因是可使复杂的查询易于理解和使用。

##### 视图作用:

1. 简化业务逻辑
2. 对客户端隐藏真实的表结构
   **视图与表的区别？**
3. 表会占用实际空间，视图并没有占用物理空间（视图中的数据）。
4. 视图可以增删改，但看情况。

##### 视图操作

- 创建视图
  `CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement`

    - 视图名必须唯一，同时不能与表重名。
    - 视图可以使用select语句查询到的列名，也可以自己指定相应的列名。
    - 可以指定视图执行的算法，通过ALGORITHM指定。
    - column_list如果存在，则数目必须等于SELECT语句检索的列数

- 视图算法(ALGORITHM)

  | ALGORITHM | 作用         | 其他                                           |
  | --------- | ------------ | ---------------------------------------------- |
  | MERGE     | 合并         | 将视图的查询语句，与外部查询需要先合并再执行！ |
  | TEMPTABLE | 临时表       | 将视图执行完毕后，形成临时表，再做外层查询！   |
  | UNDEFINED | 未定义(默认) | 指的是MySQL自主去选择相应的算法。              |

- 查看结构
  SHOW CREATE VIEW view_name

- 删除视图

  - 删除视图后，数据依然存在。
  - 可同时删除多个视图。
    DROP VIEW [IF EXISTS] view_name ...

- 修改视图结构

  - 一般不修改视图，因为不是所有的更新视图都会映射到表上。
    ALTER VIEW view_name [(column_list)] AS select_statement

### 事务（transaction）

- 什么是事务？

  事务是逻辑上的一组操作，要么都执行，要么都不执行。

  - 自动事务

    只要你对数据库执行了任何一条DQL或DML语句，那么每一行语句，都会被打包为一个事务并且提交。

  - 声明式事务

    由用户自行定义的事务，里面可能包含一行或者多行事务。

- 事务提交与回滚

  - 事务的创建

    - 隐式事务

      隐式事务是指一个事务并没有明显的开启和结束的标记。
      例如INSERT,UPDATE,DELETE等。

    - 显示事务

      我们自行定义的事务内容，包含多条语句。

  - 事务开启

    START TRANSACTION;
      开启事务后，所有被执行的SQL语句均被认作当前事务内的SQL语句。

  - 事务提交

    事务提交
      COMMIT;

  - 事务回滚

    事务回滚
      ROLLBACK;
      如果部分操作发生问题，映射到事务开启前。

    - 保存点

      SAVEPOINT 保存点名称 -- 设置一个事务保存点
        ROLLBACK TO SAVEPOINT 保存点名称 -- 回滚到保存点
        RELEASE SAVEPOINT 保存点名称 -- 删除保存点

- 事务的特性（ACID）

  - 原子性（Atomicity）

    事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。

  - 一致性（Consistency）

    事务前后数据的完整性必须保持一致。

       - 事务开始和结束时，外部数据一致
         - 在整个事务过程中，操作是连续的

  - 隔离性（Isolation）

    多个用户并发访问数据库时，一个用户的事务不能被其它用户的事物所干扰，多个并发事务之间的数据要相互隔离。

  - 持久性（Durablity）

    一个事务一旦被提交，它对数据库中的数据改变就是永久性的。

- 事务的实现

  事务的实现

      1. 要求是事务支持的表类型
      2. 执行一组相关的操作前开启事务
      3. 整组操作完成后，都成功，则提交；如果存在失败，选择回滚，则会回到事务开始的备份点。

- 事务的原理

- 注意

  1. 数据定义语言（DDL）语句不能被回滚，比如创建或取消数据库的语句，和创建、取消或更改表或存储的子程序的语句。
    2. 事务不能被嵌套

### 存储引擎

SHOW ENGINES;
-InnoDB是MYSQL在Window中默认的存储引擎。

- InnoDB自动提交特性

  InnoDB自动提交特性设置
    SET autocommit = 0|1;  0表示关闭自动提交，1表示开启自动提交。

    - 如果关闭了，那普通操作的结果对其他客户端也不可见，需要commit提交后才能持久化数据操作。
    - 也可以关闭自动提交来开启事务。但与START TRANSACTION不同的是，
        SET autocommit是永久改变服务器的设置，直到下次再次修改该设置。(针对当前连接)
          而START TRANSACTION记录开启前的状态，而一旦事务提交或回滚后就需要再次开启事务。(针对当前事务)

### 事物隔离级别

MySQL的隔离级别为可重复读级别。

- READ-UNCOMMITTED（读取未提交）

  最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。

- READ-COMMITTED（读取已提交）

  允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。

- REPEATABLE-READ（可重复读）

  对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

- SERIALIZABLE（串行化）

  最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

### 并发事务带来的问题

- 脏读（Dirty read）

  当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。

- 丢失修改（Lost to modify）

  指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。	
  例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。

- 不可重复读（Unrepeatable read）

  指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。

- 幻读（Phantom read）

  幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

- 不可重复读与幻读的区别

  不可重复读的重点是修改，幻读的重点在于新增或者删除。
  例1（同样的条件, 你读取过的数据, 再次读取出来发现值不一样了 ）：事务1中的A先生读取自己的工资为 1000的操作还没完成，事务2中的B先生就修改了A的工资为2000，导 致A再读自己的工资时工资变为 2000；这就是不可重复读。
  例2（同样的条件, 第1次和第2次读出来的记录数不一样 ）：假某工资单表中工资大于3000的有4人，事务1读取了所有工资大于3000的人，共查到4条记录，这时事务2 又插入了一条工资大于3000的记录，事务1再次读取时查到的记录就变为了5条，这样就导致了幻读。


### 杂项

1. 可用反引号（`）为标识符（库名、表名、字段名、索引、别名）包裹，以避免与关键字重名！中文也可以作为标识符！
2. 每个库目录存在一个保存当前数据库的选项文件db.opt。
3. 注释：
   单行注释 # 注释内容
   多行注释 /* 注释内容 */
   单行注释 -- 注释内容     (标准SQL注释风格，要求双破折号后加一空格符（空格、TAB、换行等）)
4. 模式通配符：
   _   任意单个字符
   %   任意多个字符，甚至包括零字符
   单引号需要进行转义 \'
5. CMD命令行内的语句结束符可以为 ";", "\G", "\g"，仅影响显示结果。其他地方还是用分号结束。delimiter 可修改当前对话的语句结束符。
6. SQL对大小写不敏感
7. 清除已有语句：\c
