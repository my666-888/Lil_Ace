## SQL注入本地环境配置与使用

###### 引言：本章所有操作均在本地环境下进行，涉及基础语法和基础sql注入



软件下载略过，环境基础安装略过



### mysql增删改



![a601b0cce690adf5b6e8f90ffcc29a4d](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2025-12\9e20f478899dc29eb19741386f9343c8\a601b0cce690adf5b6e8f90ffcc29a4d.jpg)

数据库：一个按照数据结构来组织，储存和管理数据的仓库



Linux系统下使用命令行操作，在本地上的话直接输入即可

 ~#mysql -u root -p    输入密码登录数据库

 show databases;       查看数据库

create database employees charset utf8;   创建数据库employees并且选择字符集

drop database employees;     删除数据库

use employees;   Linux下进入数据库，本地点一下就好



![image-20251219200436007](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251219200436007.png)



![57e1bd6b71af05b4fa56629838d99f53](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2025-12\9e20f478899dc29eb19741386f9343c8\57e1bd6b71af05b4fa56629838d99f53.jpg)

上两图是本地环境和Linux环境的区别，记得倒数第二行不要有逗号



show full columns from employee;      查看数据表信息

select * from employee;           查看数据表列表

drop table employee;                删除数据表

rename table employee to user;         改名

alter table user character set utf8;            修改字符级









![Screenshot_20251219_202110_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251219_202110_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



写入数据执行后，可以看到写入内容即为成功

![image-20251219202255158](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251219202255158.png)



alter table user add salary decimal(8,2);        增加一列内容

update user set salary=5000;            修改工资为5000

update user set name='bingbing' where id=1;     修改id=1的行name位bingbing     如果没有where限制则全部改

alter table user drop salary;         删除列

delete from user where job='it';          删除行

delete from user;        删除表





## 数据库查询



![Screenshot_20251219_203653_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251219_203653_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



#### 基本查询语句：

select * from users where id=1;             select+列名（*代表所有）from+表名 where+条件语句     

where后可以写成  id=‘1’ 或 id in （'3'）

select * from users where id=(select id from users where username=('admin'));    

子查询，优先执行（）内查询语句



#### 查询参数指令：

##### union

select id from users union select email_id from emails;             查询的同时合并数据显示

![image-20251219205644879](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251219205644879.png)

前面先显示，后面后显示

注： 

select * from users where id=6 union select * from emails where id=6;

这样会报错，因为联合查询前后表格列数必须相等    从表中我们看到，users有三列信息，emails有两列信息

select * from users where id=6 union select *，3 from emails where id=6;      3为填充列，第三列具体什么内容不管

![image-20251219210504609](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251219210504609.png)







##### group by   （分组）

GROUP BY 子句的根本用途是数据分组，但在SQL注入中，它常被安全人员忽略的一个作用是替代 ORDER BY 来探测查询结果的列数。由于许多Web应用防火墙（WAF）对 ORDER BY 的过滤较为严格，而对 GROUP BY 的检测可能不那么严密，因此攻击者可以利用 GROUP BY 进行绕过。

不过，这种方法并不十分可靠。原因在于 GROUP BY 本身存在一个限制：当按某列分组时，如果 SELECT 查询的其他列中存在重复值（例如，用户表中有多个同名但ID不同的记录），并且数据库处于严格模式下，那么查询就会因语义错误而报错。这种特性可能干扰列数的准确判断。

正因如此，在注入时，GROUP BY 通常不作为首选的列数探测方法，但它却恰恰因为其“不完美”的特性和相对宽松的过滤规则，在某些特定场景下成为一种有效的绕过手段。



一般用二分法判断数据表列数：

以下拿环境中的users表来举例，已知表中有三列数据

select * from users where id=1 group by 2;                group by 后面接的数字就是待测的列表列数，用二分法逐步查询即可

![image-20251220005633674](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220005633674.png)



只有寻找边界值，即一共有多少列，只要数字不超过列数则不会有任何异常





##### order by （默认按照升序来排列，数字后面加上   desc    则改为降序）

select * from users order by 2;                    对第二列数据进行升序排序

![image-20251220010113209](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220010113209.png)

select * from users order by 3;

![image-20251220010148439](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220010148439.png)

仔细看清区别        同时，如果数字超过列数，则一样报错





##### limit   （限制输出的内容数量，一般用于限数显示报错反馈信息）

select * from users limit 1,3 ;         限制为从第1行开始显示三行（实际上是从1开始数）

![image-20251220010724601](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220010724601.png)





##### and    和    or      ‘与’和‘或’



and  要同时满足前后两个条件

or     满足一个即可

and 和 or 通常用来判断它到底是字符型还是数字型，or 在post提交注入可能会用来作为万能密码

![f00fcaef6e2411c1f4cd277b93c9d2a8](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2025-12\9e20f478899dc29eb19741386f9343c8\f00fcaef6e2411c1f4cd277b93c9d2a8.jpg)









#### 常用函数



##### group_concat    （作用：把多行数据显示成一行）

在做union注入和其他报错注入的时候经常用到，因为有时候页面回显它被设定只能回显一行数据，我们用此函数就可绕过限制

![image-20251220114349083](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220114349083.png)





##### select  database()      查询当前数据库的名字

##### select  version()          查询当前数据库的版本（防火墙绕过，如注释符绕过）







### SQL注入基础

![173e229a9ee1a704699f63c68953f611](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2025-12\9e20f478899dc29eb19741386f9343c8\173e229a9ee1a704699f63c68953f611.jpg)



##### 什么是注入:

所谓SQL注入，就是通过把SQL命令插入到WEB表单提交或输入域名或页面请求的查询字符串，最终到达欺骗服务器执行恶意的SQL命令，从而进一步得到相应的数据信息。  总而言之，就是通过构造一条精巧的语句，来查询想要得到的信息



##### 注入分类：

按照查询字段来分类，分为  字符型  和  数字型       

也可以按照注入方法来分

###### 字符型：当输入的参数为字符串时，称为字符型。

###### 数字型：当输入的参数为整形时，可以认为是数字型注入。

###### 注入方式：union 注入  ，报错注入，布尔注入，时间注入



##### 什么是注入点：

注入点就是可以实行注入的地方，通常是一个访问数据库的链接，如页面中的注入点 input the ID

![image-20251220173130983](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220173130983.png)

记得将版本切换回php5

![image-20251220173338312](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220173338312.png)



##### 如何判断是字符型注入还是数字型注入   

###### 使用 and 1=1 和 and 1=2  来判断

less-1  提交 and 1=1 和 提交and 1=2 都能正常显示页面，则不可能是数字型注入，即为字符型注入

![image-20251220173928937](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220173928937.png)

less-2 提交 and 1=2 条件无法满足，语句无法被数据库查询到，网页无法正常显示，判断为数字型注入

![image-20251220174140673](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220174140673.png)

1. 数字型注入

· SQL 查询中，参数是直接嵌入到 SQL 中的，没有单引号包裹。
  ```sql
  SELECT * FROM users WHERE id = 1
  ```
· 如果输入 1 and 1=1，会变成：
  ```sql
  SELECT * FROM users WHERE id = 1 and 1=1
  ```
  逻辑永远成立，页面正常。
· 如果输入 1 and 1=2：
  ```sql
  SELECT * FROM users WHERE id = 1 and 1=2
  ```
  逻辑永远为假，页面通常会返回空或异常（因为查询无结果）。

2. 字符型注入

· 参数被单引号包裹。
  ```sql
  SELECT * FROM users WHERE name = 'admin'
  ```
· 如果输入 admin' and '1'='1，会变成：
  ```sql
  SELECT * FROM users WHERE name = 'admin' and '1'='1'
  ```
  逻辑成立，页面正常。
· 如果输入 admin' and '1'='2：
  ```sql
  SELECT * FROM users WHERE name = 'admin' and '1'='2'
  ```
  逻辑不成立，但如果 SQL 执行了，结果为空，页面可能异常也可能只是无数据。



在字符型注入中，and 1=2 可能因为sql语句被提前闭合或者注释掉，导致条件被忽略，and 的功能无法实现，页面正常显示



###### 下面讲一个小技巧，如果是数字型，则支持数学运算，字符型则不行

![image-20251220175223552](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220175223552.png)

这是  id=2  的结果，可以看到减法不起作用，判断为字符型

![25809e92f69b65699dd01bc30434f088](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2025-12\9e20f478899dc29eb19741386f9343c8\25809e92f69b65699dd01bc30434f088.png)

这是  id = 1 的结果 ，减法起作用了，则判断为数字型

（不建议使用加号，因为加号可能会被理解成空格）



##### 闭合方式     ‘    “     ’）  ”） 其他...........

##### 如何判断闭合方式：  less-1 输入  ?id=1‘   （字符型）

![image-20251220180235500](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220180235500.png)

分析报错：''1'' LIMIT 0,1'

'         '1'  ' LIMIT 0,1     '      找出引号之间的对应关系，发现 1 后面多了一个 ’  ，由此引发报错。

（有时候不一定会报错，要通过 页面回显 正确还是错误来判断



##### 闭合的作用

手动提交闭合引号，结束前一段查询语句，后面即可加入其他语句，查询需要的参数

不需要的语句可以用注释符号 ‘ --+ ’ 或  ‘#’  或  ‘ %23 '  注释掉  （要看具体环境）

![image-20251220181200117](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220181200117.png)

在图中所示，--+ 就把内容   ' LIMIT 0,1  给注释掉了，使代码正常运行





##### union联合注入

提交   ?id=1' union select database()  --+

![image-20251220192943244](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220192943244.png)

使用  --+  注释掉后则可以再中间插入语句了

此时查询失败，因为  union  前后对应的列数不一样，建议最好先确认列数

注：

如果使用 ?id=1 union select 1,2,3

此时整个注入内容被当作一个字符串，而不是SQL语句：

会被识别为：SELECT * FROM users WHERE id='1 union select 1,2,3'

不能正常查询



##### 用 group by 二分法判断默认页面数据列数量    （order by 有同样效果）

?id=1' group by 4(3,2,...) --+



##### 查找回显位

或者用  ?id=1' union select 1,2,3,4.... --+   当union前后列数相同时就不会报错了   这里的1,2,3...  没有意义 。1,100,10 也行

![image-20251220194535148](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220194535148.png)

此时我们可以看到，页面只回显了第一行的数据，union后面的内容没有显示            到小皮里面测试一下（留意版本）

![image-20251220195052066](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220195052066.png)

为了不显示前面的数据，只显示我们想要的数据，我们可以把前面的 1 改为 -1

![image-20251220195350568](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220195350568.png)

![image-20251220195430691](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220195430691.png)

负数或者0都可以达到此效果(数据库查不到的数字都行)   ， 以此拿到我们的回显位 ，然后插入函数或其他命令  

?id=-1' union select 1,version(),database() --+

![image-20251220195812615](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251220195812615.png)



###### 总结：

###### 1.查找注入点

###### 2.判断是字符型还是数字型注入   and 1=1 1=2 或  2-1

###### 3.如果字符型，找到闭合方式， ’  “  ‘）  ”）     数字型跳过这一步

###### 4.判断查询列数长短  ，group by   order by 

###### 5.查询回显位  第一个id要改为查不到的，如负数

