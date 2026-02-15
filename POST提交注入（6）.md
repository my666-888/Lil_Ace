### POST提交注入



#### 一.POST union 注入

我们先看 less-11 的源代码

![image-20260118212359494](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260118212359494.png)

所以这时候我们就能理解他的工作原理了

我们此时在前面注入username的时候，构造命令 admin' or 1=1 #

这里我们用单引号提前闭合，然后用 # 把后面的限制注释掉，此时我们就永远能登录成功，因为命令永远为真

![image-20260119224908027](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260119224908027.png)

结果大概就这样了





#### 二. POST提交报错注入

适用于提交之后没有回显的题目

用less-13举例

![image-20260119225929995](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20260119225929995.png)

可以看到其闭合方式为  ‘） 

所以我们尝试构造命令 admin') or 1=1 #   ,就登录成功了

这里我们使用floor报错

![510429968b818ec9165028ff81765da0](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\510429968b818ec9165028ff81765da0.jpg)

![cfbb8655cf7b62c2f9b05f9a90aad4ff](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\cfbb8655cf7b62c2f9b05f9a90aad4ff.jpg)







#### 三. POST 时间，布尔以及DNSLog盲注

布尔盲注：

![6b0e86b8d79769fa1b4cdea77f2fbb0b](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\6b0e86b8d79769fa1b4cdea77f2fbb0b.jpg)

![561ffab097add503f2cf31cc4f9212ba](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\561ffab097add503f2cf31cc4f9212ba.jpg)

![afa25cb474b75df35486bb624b927fe7](C:\Users\HUAWEI\Documents\xwechat_files\wxid_mtx5ad017bjn22_5d30\temp\RWTemp\2026-01\9e20f478899dc29eb19741386f9343c8\afa25cb474b75df35486bb624b927fe7.jpg)

后面像时间盲注和DNSLog都大同小异，就不展示了