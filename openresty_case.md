# OpenResty Case

在此记录一些使用openresty遇到的一些问题

### lua_code_cache on/off 所带来的一些影响

之前一直以为lua_code_cache 这个命令所带来的影响只有性能开销，其实不然， 为了完成项目中的一个需求：以syslog的格式转发client上传的各种日志到syslog服务器，用到了cloudflare的lua-resty-logger-socket模块，这个模块的功能就是做发送，本来用这个模块是为了批量发送的功能，使用时发现除非我把发送buffer大小限制到一条日志大小以下，才可以发送成功，不然感觉缓冲区buffer永远填不满一样发不出来，其实开始也好奇，这个模块不用类似shared_dict这样的存储就实现了记录buffer大小的功能，后来才发现这是一个初级的openresty知识：***在openresty中，一个变量位于lua module的top-level（也就是最外层), 在lua_code_cache on的模式下,变量值是会被缓存的!*** 这个点看似初级，其实应该包括我很多人都不知道, 如果 lua_code_cache off了， 那么这种变量不会被缓存，因为OR会给每个请求重新创建lua执行环境，这也就是为什么那个buffer填不满，永远触发不了报文发送，那个buffer计数器每次都被刷新了！get 到这个点后，我们也可以把这种变量当做一种*缓存*使用, 用模块中的“全局变量”做优化, 这也体现了OR简单背后的复杂，用起来虽然方便，高效，但背后需要深挖理解的东西有很多(后来应为syslog协议的限制，一个udp包必须是一条syslog日志，就没用)

---

### attempt to yield across c-call boundary (2017-05-26)

这两天在写模块时测试时遇到了一个错误: **lua entry thread aborted: runtime error: attempt to yield across c-call boundary** ，背景是代码中需要调用编译好的.so文件对数据进行处理，把处理好的数据用lua-resty-http(非官方维护)的非阻塞接口发送出去，但是在http.request_uri接口抛出了上面的错误, 一开始以为是我用ffi库调用了外部动态链接库文件，导致在整个上下文环境中不能让lua携程yield了，吓的我不轻了，如果真是这样那我的技术方案就行不通了，后来各种google， 在stackoverflow和google group上都有人遇到过类似的问题, 是因为luajit 实现的require是一种c接口,然后只要你的代码require了其他文件，那么在本文件中的top-level位置不能有会直接产生yield的调用,你必须把会产生yield的调用封装到自己写的lua function中，这样就可以绕过这个坑, 有人写了纯lua的require来避过这个问题, 我我个人感觉最好不要用，因为后期不好维护, 可以先这么解决，等待官方后期的改动看是否可以解决这个问题. 至于产生这个问题的深层原因，后边再补充把.

---

### ngx.timer.at的错误用法！ (2017-07-16)

又碰到坑了！项目中有段代码是记录日志用的，并且向外部发送syslog, 为了不影响请求响应的时间，当然要在**log phase**阶段, 去做这个事情, 但是在**log phase**阶段是不能用udp()这样的调用去用非阻塞socket的, 如何解决呢? 春哥给出了解决办法: *利用ngx.timer.at创建一个定时器开启一个**轻线程**去执行类似这些调用, 绕过去!*  ok, 如此确实可以使nginx不报错, 顺利执行我们的想法, 发送日志用的我们第一个case中提到的模块*lua-resty-logger-socket*, 现在看这个模块开发还是很不完善的,在我的场景里日志需要拆分成单条发送, 所以我暴力地写下了类似如下代码:

```lua
for log in logs do
    --发送每条日志
    logger.log(log)
end
```

然而这个库里面实现如我们所说的用的**ngx.timer.at**, 我把库的发送buffer的限制在每条发送, 所以不应该有buffer的积累, 但是抓包发现一个udp报文中有多条日志.后来经大牛指导, 发现是使用这个库的姿势错啦! ngx.timer.at在nginx底层是把一个job放在了一颗红黑树中，然后下次nginx事件循环时去拿到job执行, 在一个请求中执行上面的代码是**不会立即开启轻线程执行的**, 拆分的日志会挤压在第一个job中，　后边的job就拿不到日志了, 这个问题是我的使用场景出现的bug，看官可能不知所云，但重点强调的是**ngx.timer.at**这个调用的使用姿势，和底层原理