## SQL之盲注篇



###### 盲注：页面没有错误回显，不知道数据库具体返回值的情况下，对数据库中的内容进行猜解，实行SQL注入



#### 盲注分类：  布尔盲注 ， 时间盲注 ， 报错盲注（少用）





### 一.  布尔盲注（web页面只返回Ture真，False假两种类型，利用页面返回值不同，逐个猜       解数据）



![Screenshot_20251226_195856_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251226_195856_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



###### 关键函数  ascii（） 美国信息标准交换码，把字母转化成对应数字



为什么要把字母转化成数字？

查询命令可以执行，但不会返回信息到页面

![image-20251228133125939](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228133125939.png)



所以我们可以使用ascii函数

![Screenshot_20251228_133305_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251228_133305_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



我们在  ascii（） 在这个括号里输入我们想要查询的命令

![image-20251228134138451](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228134138451.png)

我们看到命令执行了，但是只能显示结果的第一个字符的ascii值



但是由于在页面中，我们是拿不到回显的，并不知道字符具体的数字是多少，所以我们需要用到二分法去进行测试

![image-20251228134620319](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228134620319.png)

![image-20251228134640571](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228134640571.png)

最终试出来是115

![image-20251228134727046](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228134727046.png)



我们想查看之后的几个字符，可以用 substring（） 或者 substr（）

![image-20251228142023355](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228142023355.png)

![image-20251228142127498](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228142127498.png)

![image-20251228142224433](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228142224433.png)

之后慢慢查询把所有的字符查出来即可

下面附上具体查询步骤，过程比较繁琐

![1766903215481](D:\Huawei Share\Huawei Share\1766903215481.png)





substr 和 limit 的区别

![Screenshot_20251228_142912_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251228_142912_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)









### 二.  时间盲注（web页面只返回一个正常页面，利用页面响应时间不同，逐个猜解数据）



###### 前提是数据库会执行代码，只是不返回页面信息



#### 关键函数：函数  sleep（） 参数为休眠时长，以秒为单位，可以为小数



如图，不管怎么样注入，页面回显都不变

![image-20251228192531598](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228192531598.png)



所以我们可以使用sleep函数，如果指令成功执行，那么页面就会在指定的一段时间后刷新

执行失败，则快速返回

![image-20251228193031352](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228193031352.png)

我们能看到页面在加载中，说明命令正确

![image-20251228193121285](C:\Users\HUAWEI\AppData\Roaming\Typora\typora-user-images\image-20251228193121285.png)

当然也可以这样看，更直观一些，由此可见页面可以使用时间盲注



![Screenshot_20251228_193619_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251228_193619_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)





所以，我们可以通过布尔盲注的方式，判断我们值的大小是否正确，从而一步步确定

![Screenshot_20251228_195142_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251228_195142_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)



之后以此类推即可

![Screenshot_20251228_195526_tv_danmaku_bili_UnitedBizDetailsActivity](D:\Huawei Share\Huawei Share\Screenshot_20251228_195526_tv_danmaku_bili_UnitedBizDetailsActivity.jpg)

