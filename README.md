# PostgreSQL
Learning notes based on the official PostgreSQL documentation.

本书分为几个部分。针对不同的用户群体、经验不同的阶段
加州大学伯克利分校计算机科学系基因 ORDBMS

其他的资料 Wiki  FAQ list, TODO list  Mailing Lists

uses a client/server model
PostgreSQL session 由一下协同进程组成 
server process 数据库文件，接受连接，代表客户端执行操作
user's client (frontend) 文本 图形 应用程序 维护工具

$ createdb mydb command not found 
createdb: error: connection to server on socket "/tmp/.s.PGSQL.5432" failed: No such file or directory
Is the server running locally and accepting connections on that socket?
未启动，或者它没有在 createdb 期望连接的位置进行监听
createdb: error: connection to server on socket "/tmp/.s.PGSQL.5432" failed: FATAL:  role "joe" does not exist
 -U 选项或设置 PGUSER 环境变量来指定
createdb: error: database creation failed: ERROR:  permission denied to create database

单单$ createdb $ dropdb mydb

客户端会尝试使用当前系统用户名 服务器进程由postgres用户运行

$ psql mydb可以不提供名字

SELECT version(); SELECT current_date; SELECT 2 + 2;

\h \q  \?

Understanding the New SQL: A Complete Guide
A Guide to the SQL Standard: A User's Guide to the Standard Database Language SQL

\i 命令从指定文件中读取命令
psql 的 -s 选项使您进入单步模式

管理存储在关系中的数据的系统。关系本质上是数学中对表的称呼。
以表格形式存储数据似乎理所当然，但还有其他多种组织数据库的方式。

每张表都是由若干行（row）构成的、具有名称的数据集合。
每一行都具有相同的一组命名列（column），并且每一列都有特定的数据类型。
列具有固定的排列顺序，但保证表中各行的排列顺序，除开指定排序规则。

标识符被双引号括起来以保留大小写

\conninfo
SELECT current_database();

除了在 SQL 标准中需要支持的特殊情况外，类型名称在语法中不是关键字。

DROP TABLE tablename;

INSERT INTO weather
    (city, temp_lo, temp_hi, prcp, date)
VALUES
    ('San Francisco', 46, 50, 0.25, '1994-11-27');

COPY 命令更大更快，但是灵活性差了点
/var/lib/pgsql
\d weather

SELECT * FROM weather;
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;

SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;

SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;

SELECT * FROM weather
    ORDER BY city;

SELECT * FROM weather
    ORDER BY city, temp_lo;

SELECT DISTINCT city
    FROM weather
    ORDER BY city;

查询可以同时访问多张表，也可以通过特定的方式访问同一张表，使该表中的多行数据能够同时参与处理。同时访问多张表（或者同一张表的多个实例）的查询，称为连接查询（join query）。通过一个表达式指定哪些行应当彼此配对。例如，如果要返回所有天气记录及其对应城市的位置信息，数据库就需要将 weather 表中每一行的 city 列，与 cities 表中所有行的 name 列进行比较，并选出这两个值相匹配的行对。  SELECT *
FROM weather
JOIN cities ON city = name;

SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.location
    FROM weather JOIN cities ON weather.city = cities.name;

SELECT *
    FROM weather, cities
    WHERE city = name;

扫描 weather 表，并针对其中的每一行，查找 cities 表中与之匹配的行。如果找不到匹配的行，就用一些“空值”代替 cities 表中相应列的值。
这种查询称为外连接（outer join）。前面介绍的连接则称为内连接（inner join）。

SELECT *
FROM weather
LEFT OUTER JOIN cities ON weather.city = cities.name;

右外连接（right outer join）和全外连接（full outer join）自连接（Self Join）

Aggregate Functions聚合函数能够从多行输入中计算出单一结果。计算一组行的计数（count）、总和（sum）、平均值（avg）、最大值（max）和最小值（min）。

SELECT max(temp_lo) FROM weather;

因为 WHERE 子句决定哪些行将参与聚合计算。因此，显然必须先执行 WHERE 子句，再计算聚合函数的结果。
SELECT city FROM weather WHERE temp_lo = max(temp_lo);-- 错误的写法
SELECT city
FROM weather
WHERE temp_lo = (SELECT max(temp_lo) FROM weather);

可以查询每个城市的天气记录数量，以及该城市观测到的最高最低气温：SELECT city, count(*), max(temp_lo)
FROM weather
GROUP BY city;

可以使用 HAVING 子句对分组后的结果进行筛选：SELECT city, count(*), max(temp_lo)
FROM weather
GROUP BY city
HAVING max(temp_lo) < 40;

可以查询每个城市的天气记录数量，以及该城市观测到的最高最低气温：SELECT city, count(*), max(temp_lo)
FROM weather
GROUP BY city;

可以使用 HAVING 子句对分组后的结果进行筛选：SELECT city, count(*), max(temp_lo)
FROM weather
GROUP BY city
HAVING max(temp_lo) < 40;

只关心城市名称以字母 S 开头的城市，可以这样查询：  
SELECT city, count(*), max(temp_lo)
FROM weather
WHERE city LIKE 'S%'  -- (1)
GROUP BY city;

1. WHERE 在分组和聚合计算之前筛选输入行。 它决定哪些行能够参与后续的聚合计算。
2. HAVING 在分组和聚合计算之后筛选分组结果。 它决定哪些分组能够保留在最终的查询结果中。

除了 WHERE，我们还可以使用 FILTER 来选择参与聚合计算的行。FILTER 是一种针对单个聚合函数设置的选项。  
SELECT city,
       count(*) FILTER (WHERE temp_lo < 45),
       max(temp_lo)
FROM weather
GROUP BY city;
FILTER 只会从它所附加的那个聚合函数的输入中排除不符合条件的行，而不会影响其他聚合函数。  

UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';

DELETE 命令从表中删除行 DELETE FROM weather WHERE city = 'Hayward';
DELETE FROM tablename;

存储和访问数据的基础知识--> SQL 的一些更高级的特性，简化管理并防止数据丢失或损坏-->了解一些 PostgreSQL 扩展。

确保没有人能够向 weather 表中插入在 cities 表中没有匹配项的行。这被称为维护数据的引用完整性。
CREATE TABLE cities (
        name     varchar(80) primary key,
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(name),
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);
INSERT INTO weather VALUES ('Berkeley', 45, 53, 0.0, '1994-11-28');
ERROR:  insert or update on table "weather" violates foreign key constraint "weather_city_fkey"
DETAIL:  Key (city)=(Berkeley) is not present in table "cities".

通过将事务中的 SQL 命令包裹在 BEGIN 和 COMMIT 命令中
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
-- etc etc
COMMIT;
ROLLBACK 

使用 SAVEPOINT 定义保存点后，如果需要，你可以使用 ROLLBACK TO 回滚到该保存点。

无论是释放还是回滚到某个保存点，都会自动释放在该保存点之后定义的所有保存点。
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Wally';
COMMIT;

ROLLBACK TO 是重新获得事务块控制权的唯一方法，该事务块因错误或系统原因进入中止状态，除了完全回滚并重新开始之外。

Window Functions
Inheritance

Part III. Server Administration
安装、配置、用户和数据库管理以及维护任务。

18
如何设置和运行数据库服务器，以及它与操作系统的交互。

在执行任何操作之前，先在磁盘上初始化一个数据库集簇。（SQL标准使用的术语是目录集簇。）
数据库集簇将包含一个名为postgres的数据库，该数据库旨在作为工具、用户和第三方应用程序使用的默认数据库。
每个集簇中还会创建另外两个数据库，分别名为template1和template0。顾名思义，它们将用作后续创建数据库的模板。

使用 initdb 对其进行初始化。


Chapter 20. Client Authentication
根据（客户端）主机地址、数据库和用户来选择，
客户端认证由一个配置文件控制pg_hba.conf存储在数据库集群的数据目录中HBA 代表基于主机的认证。
重新读取该文件

