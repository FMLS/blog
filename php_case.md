# PHP case

记录一些使用php遇到的一些问题

---

## Problem

这两天重新编译安装了php5.6,碰到一个小问题,执行php-cli模式会出现:

```bash
PHP Warning: Module 'pdo_pgsql' already loaded in Unknown on line 0
```

##Cause

其实有两种方法加载php扩展:
1.编译时选定
2.加载动态链接库
这个错误的出现是因为pdo_pgsql已经被我编译进了php，但又在php.ini文件中加载了一次pdo_pgsql.so文件导致。

##Fix:

把php.ini中extension=pdo_pgsql注释掉即可, 输入`php-m`依然可以看到pdo_pgsql扩展已经加载(因为已经静态编译到了php)

　## Problem

最近碰到一个问题，用php写了个测试端口是否联通的接口，该接口在前端调用后需要n秒才能返回，在此期间点击页面上其他按钮没有反应，js是异步处理的，应该不是前端的问题，当用两个浏览器时没有此问题。

##Cause

最终定位到是后端php的session锁问题，当同一用户发起请求，php为了控制并发，是会将用户的请求串行执行的

##Fix

解决方法是用session_write_close（）关闭session写锁，但还是可以读的，如果用redis等存储session是没有这个问题的，猜测应该是利用底层文件的读写锁实现的

---

