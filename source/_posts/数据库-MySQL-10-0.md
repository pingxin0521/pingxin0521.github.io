---
title: MySQL SQL 权限(八)
date: 2019-05-16 13:08:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---

### 用户权限

MariaDB/MySQL中的user由用户名和主机名构成，如"root@localhost"，同用户名但不同主机名对MySQL/MariaDB来讲是不同的，也就是说"root@localhost"和"root@127.0.0.1"是不同的用户，尽管它们都是本机的root。

<!--more-->

#### CREATE USER

在对 MySQL 的日常管理和实际操作中，为了避免用户恶意冒名使用 root 账号控制数据库，通常需要创建一系列具备适当权限的账号，应该尽可能地不用或少用 root 账号登录系统，以此来确保数据的安全访问。 

可以使用 CREATE USER 语句来创建一个或多个 MySQL 账户，并设置相应的口令。

语法格式：

```
-- mysql5.7以上
CREATE USER <用户名> [ IDENTIFIED ] BY [ PASSWORD ] <口令>
```

语法说明如下：

1. <用户名>
   指定创建用户账号，格式为 'user_name'@'host_name'。这里user_name是用户名，host_name为主机名，即用户连接 MySQL 时所在主机的名字。若在创建的过程中，只给出了账户的用户名，而没指定主机名，则主机名默认为“%”，表示一组主机。
   
   `username@host`表示授予的用户以及允许该用户登录的IP地址。其中Host有以下几种类型：
   
   - **localhost：**只允许该用户在本地登录，不能远程登录。
   - **%：**允许在除本机之外的任何一台机器远程登录。
   - **192.168.52.32：**具体的IP表示只允许该用户从特定IP登录。
   
   ![MC8kWj.png](https://s2.ax1x.com/2019/11/06/MC8kWj.png)
   
2. PASSWORD
   可选项，用于指定散列口令，即若使用明文设置口令，则需忽略PASSWORD关键字；若不想以明文设置口令，且知道 PASSWORD() 函数返回给密码的散列值，则可以在口令设置语句中指定此散列值，但需要加上关键字PASSWORD。

3. IDENTIFIED BY子句
   用于指定用户账号对应的口令，若该用户账号无口令，则可省略此子句。

4. <口令>
   指定用户账号的口令，在IDENTIFIED BY关键字或PASSWOED关键字之后。给定的口令值可以是只由字母和数字组成的明文，也可以是通过 PASSWORD() 函数得到的散列值。

使用 CREATE USER 语句应该注意以下几点： 

-  如果使用 CREATE USER 语句时没有为用户指定口令，那么 MySQL 允许该用户可以不使用口令登录系统，然而从安全的角度而言，不推荐这种做法。
-  使用 CREATE USER 语句必须拥有 MySQL 中 MySQL 数据库的 INSERT 权限或全局 CREATE USER 权限。
-  使用 CREATE USER 语句创建一个用户账号后，会在系统自身的 MySQL 数据库的 user 表中添加一条新记录。若创建的账户已经存在，则语句执行时会出现错误。
-  新创建的用户拥有的权限很少。他们可以登录 MySQL，只允许进行不需要权限的操作，如使用 SHOW 语句查询所有存储引擎和字符集的列表等。

 如果两个用户具有相同的用户名和不同的主机名，MySQL 会将他们视为不同的用户，并允许为这两个用户分配不同的权限集合。

 【实例 1】使用 CREATE USER 创建一个用户，用户名是 james，密码是 tiger，主机是 localhost。输入的 SQL 语句和执行过程如下所示。 

```
mysql> CREATE USER 'james'@'localhost'
    -> IDENTIFIED BY 'tiger';
Query OK, 0 rows affected (0.12 sec)
```

#### RENAME USER

可以使用 RENAME USER 语句修改一个或多个已经存在的 MySQL 用户账号。

语法格式：

```
RENAME USER <旧用户> TO <新用户>
```

语法说明如下： 

-  <旧用户>：系统中已经存在的 MySQL 用户账号。
-  <新用户>：新的 MySQL 用户账号。

 使用 RENAME USER 语句时应该注意以下几点：

- RENAME USER 语句用于对原有的 MySQL 账户进行重命名。

- 若系统中旧账户不存在或者新账户已存在，则该语句执行时会出现错误。

-  要使用 RENAME USER 语句，必须拥有 MySQL 中的 MySQL 数据库的 UPDATE 权限或全局 CREATE USER 权限。

【实例 1】使用 RENAME USER 语句将用户名 james 修改为 jack，主机是 localhost。输入的 SQL 语句和执行过程如下所示。 

```
mysql> RENAME USER james@'localhost'
    -> TO jack@'localhost';
Query OK, 0 rows affected (0.03 sec)
```

####  修改用户口令

可以使用 SET PASSWORD 语句修改一个用户的登录口令。

 语法格式： 

```
SET PASSWORD [ FOR <用户名> ] =
{
    PASSWORD('新明文口令')
    | OLD_PASSWORD('旧明文口令')
    | '加密口令值'
}
```

 语法说明如下。 

-  FOR 子句：可选项。指定欲修改口令的用户。
-  PASSWORD('新明文口令')：表示使用函数 PASSWORD() 设置新口令，即新口令必须传递到函数 PASSWORD() 中进行加密。
-  加密口令值：表示已被函数 PASSWORD() 加密的口令值。

>  注意：PASSWORD() 函数为单向加密函数，一旦加密后不能解密出原明文。

 使用 SET PASSWORD 语句应注意以下几点： 

-  在 SET PASSWORD 语句中，若不加上 FOR 子句，表示修改当前用户的口令。若加上 FOR 子句，表示修改账户为 user 的用户口令。
-  user 必须以 'user_name'@'host_name' 的格式给定，user_name 为账户的用户名，host_name 为账户的主机名。
-  该账户必须在系统中存在，否则语句执行时会出现错误。
-  在 SET PASSWORD 语句中，只能使用选项 PASSWORD('新明文口令') 和加密口令值中的一项，且必须使用其中的一项。

 【实例 2】使用 SET 语句将用户名为 jack 的密码修改为 lion，主机是 localhost。输入的 SQL 语句和执行过程如下所示。 

```
mysql> SET PASSWORD FOR 'jack'@'localhost'=
    -> PASSWORD('lion');
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> update mysql.user set password = password('lion') where user = 'jack' and host = '%';
mysql>  flush privileges;
```

**普通用户自己修改自己的密码**

```
set password=password('password');
```



#### DROP USER

MySQL 数据库中可以使用 DROP USER 语句来删除一个或多个用户账号以及相关的权限。

语法格式：

```
DROP USER <用户名1> [ , <用户名2> ]…
```

 使用 DROP USER 语句应该注意以下几点： 

-  DROP USER 语句可用于删除一个或多个 MySQL 账户，并撤销其原有权限。
-  使用 DROP USER 语句必须拥有 MySQL 中的 MySQL 数据库的 DELETE 权限或全局 CREATE USER 权限。
-  在 DROP USER 语句的使用中，若没有明确地给出账户的主机名，则该主机名默认为“%”。

>  注意：用户的删除不会影响他们之前所创建的表、索引或其他数据库对象，因为 MySQL 并不会记录是谁创建了这些对象。

 【实例】使用 DROP USER 语句删除用户'jack'@'localhost'。输入的 SQL 语句和执行过程如下所示。 

```
mysql> DROP USER 'jack'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

### GRANT

当成功创建用户账户后，还不能执行任何操作，需要为该用户分配适当的访问权限。可以使用 SHOW GRANT FOR 语句来查询用户的权限。

> 注意：新创建的用户只有登录 MySQL 服务器的权限，没有任何其他权限，不能进行其他操作。

USAGE ON*.* 表示该用户对任何数据库和任何表都没有权限。 

对于新建的 MySQL 用户，必须给它授权，可以用 GRANT 语句来实现对新建用户的授权。

语法格式： 

```
GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user  [IDENTIFIED [BY [PASSWORD] 'password'][WITH with_option [with_option]

object_type:
    TABLE
  | FUNCTION
  | PROCEDURE

priv_level:
    *
  | *.*
  | db_name.*
  | db_name.tbl_name
  | tbl_name
  | db_name.routine_name

with_option:  
    GRANT OPTION
  | MAX_QUERIES_PER_HOUR count
  | MAX_UPDATES_PER_HOUR count
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count
  | MAX_STATEMENT_TIME time 
```

 语法说明如下： 

语法说明如下：

1. <列名>
   可选项。用于指定权限要授予给表中哪些具体的列。
2. ON 子句
   用于指定权限授予的对象和级别，如在 ON 关键字后面给出要授予权限的数据库名或表名等。
3. <权限级别>
   用于指定权限的级别。可以授予的权限有如下几组： 
   -  列权限，和表中的一个具体列相关。例如，可以使用 UPDATE 语句更新表 students 中 student_name 列的值的权限。
   -  表权限，和一个具体表中的所有数据相关。例如，可以使用 SELECT 语句查询表 students 的所有数据的权限。
   -  数据库权限，和一个具体的数据库中的所有表相关。例如，可以在已有的数据库 mytest 中创建新表的权限。
   -  用户权限，和 MySQL 中所有的数据库相关。例如，可以删除已有的数据库或者创建一个新的数据库的权限。

对应地，在 GRANT 语句中可用于指定权限级别的值有以下几类格式： 

- ` *`：表示当前数据库中的所有表。
- ` *.*`：表示所有数据库中的所有表。
- ` db_name.*`：表示某个数据库中的所有表，db_name 指定数据库名。
-  db_name.tbl_name：表示某个数据库中的某个表或视图，db_name 指定数据库名，tbl_name 指定表名或视图名。
-  tbl_name：表示某个表或视图，tbl_name 指定表名或视图名。
-  db_name.routine_name：表示某个数据库中的某个存储过程或函数，routine_name 指定存储过程名或函数名。
-  TO 子句：用来设定用户口令，以及指定被赋予权限的用户 user。若在 TO  子句中给系统中存在的用户指定口令，则新密码会将原密码覆盖；如果权限被授予给一个不存在的用户，MySQL 会自动执行一条 CREATE USER  语句来创建这个用户，但同时必须为该用户指定口令。

 GRANT语句中的<权限类型>的使用说明如下： 

1. 授予数据库权限时，<权限类型>可以指定为以下值：

-  SELECT：表示授予用户可以使用 SELECT 语句访问特定数据库中所有表和视图的权限。
-  INSERT：表示授予用户可以使用 INSERT 语句向特定数据库中所有表添加数据行的权限。
-  DELETE：表示授予用户可以使用 DELETE 语句删除特定数据库中所有表的数据行的权限。
-  UPDATE：表示授予用户可以使用 UPDATE 语句更新特定数据库中所有数据表的值的权限。
-  REFERENCES：表示授予用户可以创建指向特定的数据库中的表外键的权限。
-  CREATE：表示授权用户可以使用 CREATE TABLE 语句在特定数据库中创建新表的权限。
-  ALTER：表示授予用户可以使用 ALTER TABLE 语句修改特定数据库中所有数据表的权限。
-  SHOW VIEW：表示授予用户可以查看特定数据库中已有视图的视图定义的权限。
-  CREATE ROUTINE：表示授予用户可以为特定的数据库创建存储过程和存储函数的权限。
-  ALTER ROUTINE：表示授予用户可以更新和删除数据库中已有的存储过程和存储函数的权限。
-  INDEX：表示授予用户可以在特定数据库中的所有数据表上定义和删除索引的权限。
-  DROP：表示授予用户可以删除特定数据库中所有表和视图的权限。
-  CREATE TEMPORARY TABLES：表示授予用户可以在特定数据库中创建临时表的权限。
-  CREATE VIEW：表示授予用户可以在特定数据库中创建新的视图的权限。
-  EXECUTE ROUTINE：表示授予用户可以调用特定数据库的存储过程和存储函数的权限。
-  LOCK TABLES：表示授予用户可以锁定特定数据库的已有数据表的权限。
-  ALL 或 ALL PRIVILEGES：表示以上所有权限。

2. 授予表权限时，<权限类型>可以指定为以下值：

-  SELECT：授予用户可以使用 SELECT 语句进行访问特定表的权限。
-  INSERT：授予用户可以使用 INSERT 语句向一个特定表中添加数据行的权限。
-  DELETE：授予用户可以使用 DELETE 语句从一个特定表中删除数据行的权限。
-  DROP：授予用户可以删除数据表的权限。
-  UPDATE：授予用户可以使用 UPDATE 语句更新特定数据表的权限。
-  ALTER：授予用户可以使用 ALTER TABLE 语句修改数据表的权限。
-  REFERENCES：授予用户可以创建一个外键来参照特定数据表的权限。
-  CREATE：授予用户可以使用特定的名字创建一个数据表的权限。
-  INDEX：授予用户可以在表上定义索引的权限。
-  ALL 或 ALL PRIVILEGES：所有的权限名。

3. 授予列权限时，<权限类型>的值只能指定为 SELECT、INSERT 和 UPDATE，同时权限的后面需要加上列名列表 column-list。

4. 最有效率的权限是用户权限。

   授予用户权限时，<权限类型>除了可以指定为授予数据库权限时的所有值之外，还可以是下面这些值： 

-  CREATE USER：表示授予用户可以创建和删除新用户的权限。
-  SHOW DATABASES：表示授予用户可以使用 SHOW DATABASES 语句查看所有已有的数据库的定义的权限。

【实例】使用 GRANT 语句创建一个新的用户 testUser，密码为 testPwd。用户 testUser 对所有的数据有查询、插入权限，并授予 GRANT 权限。输入的 SQL 语句和执行过程如下所示。 

```
mysql> GRANT SELECT,INSERT ON *.*
    -> TO 'testUser'@'localhost'
    -> IDENTIFIED BY 'testPwd'
    -> WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.05 sec)
```

 使用 SELECT 语句查询用户 testUser 的权限，如下所示。 

```
mysql> SELECT Host,User,Select_priv,Grant_priv
    -> FROM mysql.user
    -> WHERE User='testUser';
+-----------+----------+-------------+------------+
| Host      | User     | Select_priv | Grant_priv |
+-----------+----------+-------------+------------+
| localhost | testUser | Y           | Y          |
+-----------+----------+-------------+------------+
1 row in set (0.01 sec)
```

### REVOKE

MySQL 数据库中可以使用 REVOKE 语句删除一个用户的权限，此用户不会被删除。

revoke表示收回权限，注意revoke无法收回usage权限。

语法格式有两种形式，如下所示：

1. 第一种：

   ```
    REVOKE <权限类型> [ ( <列名> ) ] [ , <权限类型> [ ( <列名> ) ] ]…
   ON <对象类型> <权限名> FROM <用户1> [ , <用户2> ]…
   ```

2.  第二种：

   ```
    REVOKE ALL PRIVILEGES, GRANT OPTION
   FROM user <用户1> [ , <用户2> ]…
   ```

   语法说明如下： 

   -  REVOKE 语法和 GRANT 语句的语法格式相似，但具有相反的效果。
   -  第一种语法格式用于回收某些特定的权限。
   -  第二种语法格式用于回收特定用户的所有权限。
   -  要使用 REVOKE 语句，必须拥有 MySQL 数据库的全局 CREATE USER 权限或 UPDATE 权限。

【实例】使用 REVOKE 语句取消用户 testUser 的插入权限，输入的 SQL 语句和执行过程如下所示。 

```
mysql> REVOKE INSERT ON *.*
    -> FROM 'testUser'@'localhost';
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT Host,User,Select_priv,Insert_priv,Grant_priv
    -> FROM mysql.user
    -> WHERE User='testUser';
+-----------+----------+-------------+-------------+------------+
| Host      | User     | Select_priv | Insert_priv | Grant_priv |
+-----------+----------+-------------+-------------+------------+
| localhost | testUser | Y           | N           | Y          |
+-----------+----------+-------------+-------------+------------+
1 row in set (0.00 sec)
```

**mysql 8.0语句**

```
-- 创建用户的操作已经不支持grant的同时创建用户的方式，需先创建用户再进行授权
create user  'admin'@'%' identified by 'admin123';
grant all on *.* to 'admin'@'%' ;
flush privileges;

-- 修改密码
ALTER USER 'admin'@'%' IDENTIFIED WITH MYSQL_NATIVE_PASSWORD BY '新密码';

-- 修改加密方式
-- 修改密码为用不过期
ALTER USER 'root'@'%' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; 
-- 修改密码并指定加密规则为mysql_native_password
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

```

### 权限验证

在MariaDB/MySQL服务器启动后会**载入权限表到内存中**，当用户要连接服务器，会读取权限表来验证和分配权限，即在内存中进行权限的读取和写入。

MariaDB/MySQL中的权限系统经过两步验证：

1.合法性验证：验证user是否合法，合法者允许连接服务器，否则拒绝连接。

2.权限验证和分配：对通过合法性验证的用户分配对数据库中各对象的操作权限。

MariaDB/MySQL中的权限表都存放在mysql数据库中。MySQL5.6以前，权限相关的表有user表、db表、host表、tables_priv表、columns_priv表、procs_priv表(存储过程和函数相关的权限)。从MySQL5.6开始，host表已经没有了。MariaDB中虽然有host表，但却不用。

那这些表有什么用，和权限又有什么关系呢？

- User的一行记录代表一个用户标识

- db的一行记录代表对数据库的权限

- table_priv的一行记录代表对表的权限

- column_priv的一行记录代表对某一列的权限

![MCmlJx.png](https://s2.ax1x.com/2019/11/06/MCmlJx.png)

这几个表用的最多的是user表。user表主要分为几个部分：用户列、权限列、安全列、资源控制列以及杂项列，最需要关注的是用户列和权限列。其中权限列又分为普通权限(上表中红色字体)和管理权限列，如select类的为普通权限，super权限为管理权限。且可以看到，db表中的权限全都是普通权限，user表中除了db表中具有的普通权限还有show_db_pirv和create_tablespace_priv，除此之外还有几个管理员权限。也就是说，db中没有的权限是无法授予到指定数据库的。例如不能授予super权限给test数据库。

另外，usage权限在上表中没有列出，因为该权限是所有用户都有的权限，它只用来表示能否登录数据库，**它的一个特殊功能是grant仅指定该权限的时候不会影响现有权限，也就是说可以拿grant来修改密码而不影响现有权限。**

需要说明的是，从user表到db表再到tables_priv表最后是columns_priv表，它们的权限是逐层细化的。user表中的普通权限是针对所有数据库的，例如在user表中的select_priv为Y，则对所有数据库都有select权限；db表是针对特定数据库中所有表的，如果只有test数据库中有select权限，那么db表中就有一条记录test数据库的select权限为Y，这样对test数据库中的所有表都有select权限，而此时user表中的select权限就为N(因为为Y的时候是所有数据库都有权限)；同理tables_priv表也一样，是针对特定表中所有列的权限；columns_priv则是针对特定列的权限。

所以对于已经通过身份合法性验证的用户的权限读取和分配的机制如下：

**1.读取uesr表，看看user表是否有对应为Y的权限列，有则分配。**

**2.读取db表，看看db表中是否有哪个数据库分配了对应的权限。**

**3.读取tables_priv表，看看哪些表中有对应的权限。**

**4.读取columns_priv表，看看对哪些具体的列有什么权限。**

例如，为某一用户授予test数据库的select权限。可以看到user表中的select_priv为N，而db表中的select为Y。

```
CREATE USER 'long'@'192.168.100.1' IDENTIFIED BY '123456';
GRANT  SELECT ON test.* TO 'long'@'192.168.100.1';
SELECT host,user,select_priv FROM mysql.user where host='192.168.100.1';
SELECT * FROM mysql.db where host='192.168.100.1' \G
```

**图解认证和权限分配的两个阶段**

![MC3coF.png](https://s2.ax1x.com/2019/11/06/MC3coF.png)

**权限生效时机**

在服务器启动时读取权限表到内存中，从此时开始权限表生效。

之后使用grant、revoke、set password 等命令也会隐含的刷新权限表到内存中。

另外，使用显式的命令flush privileges或mysqladmin flush-privileges或mysqladmin reolad也会将上述几张权限表重新刷到内存中以供后续的身份验证和权限验证、分配。

