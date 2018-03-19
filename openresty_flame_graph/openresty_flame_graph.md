# 安装准备

uname -r 查看内核版本号

```
[root@localvm]# uname -r
2.6.32-642.el6.x86_64
```

安装对应的内核debuginfo包，这里面的kernel-devel我在官网没找到，是从iso里面拷贝出来的, 后缀版本号一定要和uname -r自己显示的版本号对应

```
kernel-debuginfo-2.6.32-642.el6.x86_64.rpm
kernel-debuginfo-common-x86_64-2.6.32-642.el6.x86_64.rpm
kernel-devel-2.6.32-642.el6.x86_64.rpm
```

接下来安装systemtap工具，这个工具建议从源码编译安装，避免在抓取数据时出现莫名奇妙的错误，我们的CenOS6.8 yum源中默认使用2.9版本的，所以我们从2.9的源码开始编译安装(我试了>2.9版本，抓取时直接会把系统跑死，具体原因不明)，下载后根据官网的build教程进行安装，安装完成后运行如下命令检测是否安装成功

```
stap -v -e 'probe vfs.read {printf("read performed\n"); exit()}'

产生如下输出：
Pass 1: parsed user script and 110 library script(s) using 158436virt/33228res/2268shr/31592data kb, in 210usr/20sys/232real ms.
Pass 2: analyzed script: 1 probe(s), 1 function(s), 4 embed(s), 0 global(s) using 257092virt/132824res/3312shr/130248data kb, in 1210usr/250sys/2242real ms.
Pass 3: using cached /root/.systemtap/cache/7e/stap_7e5d88cc495559255e74857abaa6cae8_1657.c
Pass 4: using cached /root/.systemtap/cache/7e/stap_7e5d88cc495559255e74857abaa6cae8_1657.ko
Pass 5: starting run.
read performed
Pass 5: run completed in 0usr/0sys/533real ms.
```



由于我们的openresty使用官方预编译的rpm包安装, 默认是没有debuginfo的，所以需要安装相关的-debuginfo rpm包

```
sudo yum install yum-utils
sudo yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo

列出官方源中所有rpm包
sudo yum --disablerepo="*" --enablerepo="openresty" list available

下载需要的包
yum install --downloadonly --downloaddir=/path/yourself
```

目前需要安装的debuginfo包如下:

```
openresty-debuginfo-1.11.2.5-2.el6.x86_64.rpm
openresty-openssl-debuginfo-1.0.2k-1.el6.x86_64.rpm
openresty-pcre-debuginfo-8.40-1.el6.x86_64.rpm
openresty-zlib-debuginfo-1.2.11-3.el6.x86_64.rpm

运行 rpm -ivh xxx --force --nodeps 进行安装
```

# 抓取数据

**抓取C级别数据**

使用openresty项目中的openresty-systemtap-toolkit工具， 和Brendan Gregg's FlameGraph tools

```
git clone https://github.com/openresty/openresty-systemtap-toolkit.git
git clone https://github.com/brendangregg/FlameGraph.git
#开始抓取数据
./openresty-systemtap-toolkit/sample-bt -p 8736 -t 5 -u > a.bt
#转换数据 生成火焰图
./FlameGraph/stackcollapse-stap.pl a.bt > a.cbt
./FlameGraph/flamegraph.pl a.cbt > a.svg
```

**抓取lua级别数据**

由于openresty中使用luajit来运行lua代码，所以必须使用openresty/stapxx工具来抓取数据

```
git clone https://github.com/openresty/stapxx.git
#把stap++目录加入PATH环境变量
export PATH=$PWD:$PATH
#开始抓取数据
./stapxx/samples/lj-lua-stacks.sxx --skip-badvars -x 6949 > a.bt
#过滤数据文件，得到更易读的火焰图数据
./openresty-systemtap-toolkit/fix-lua-bt a.bt > a2.bt
#生成火焰图
./FlameGraph/stackcollapse-stap.pl a2.bt > a.cbt
./FlameGraph/flamegraph.pl a.cbt  > a.svg
```



# 火焰图示例

![a](https://github.com/FMLS/blog/blob/master/openresty_flame_graph/a.avg)

可以看出上面的图中match函数占用了大量的时间, 可以删除发生在common_func.lua文件的_M.split函数调用栈，match函数是lua内置的builtin#85号函数，lua的内置正则函数比较低效，应该尽量避免使用

下图是一个优化后的火焰图，图中没有耗时很长的栈帧，说明没有明显的问题，测试结果的qps也有大幅提升

![normal](https://github.com/FMLS/blog/blob/master/openresty_flame_graph/normal.svg)



# 问题汇总



```
Found exact match for libluajit: /usr/local/openresty/luajit/lib/libluajit-5.1.so.2.1.0
WARNING: Start tracing 6089 (/usr/local/openresty/nginx/sbin/nginx)
WARNING: Please wait for 2 seconds...
WARNING: user string copy fault -14 at 000000006369641a [man error::fault]
ERROR: Skipped too many probes, check MAXSKIPPED or try again with stap -t for more details.
ERROR: MAXACTION exceeded near identifier 'tmp_level' at stapxx-DpI245Th/luajit.stp:565:21
WARNING: Number of errors: 1, skipped probes: 105
WARNING: /usr/local/bin/staprun exited with status: 1
Pass 5: run failed.  [man error::pass5]
```

目前经验是由于cpu占用率过高, 采样时间久导致超过systemtap的MAXACTION阈值保护，建议这种情况下只采样一两秒，多试几次