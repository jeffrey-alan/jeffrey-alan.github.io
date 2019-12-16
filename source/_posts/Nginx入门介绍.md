---
title: Nginx入门介绍
date: 2019-12-16 16:08:53
tags: 
- 负载均衡中间件
categories: 
- 中间件

---

# Nginx简介

![20191216162951](http://tc.llx-cn.com/20191216162951.png)

Nginx 作为一个强大的Web服务器软件，具有高性能、高并发性和低内存占用的特点。此外，其也能提供强大的反向代理功能。根据 Netcraft 公司统计，世界上最繁忙的网站中有 11.48%使用 Nginx 作为其服务器或者代理服务器。

<!-- more -->

Nginx 由俄罗斯的程序设计师 lgor Sysoev（伊戈尔·赛索耶夫） 所开发，最初供俄罗斯大型的入口网站及搜索引擎 Rambler 使用。其特点是占用内存少，并发能力强（用于解决 C10K 问题），事实上 Nginx 的并发能力确实在同类型的网页服务器中表现较好。

俄罗斯大约有超过20%的虚拟主机采用Nginx作为反向代理服务器。
中国大陆使用 nginx 网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。


基于反向代理的功能，Nginx 作为负载均衡主要有一下几个特点：

* **高性能**：采用C语言直接开发，编译的形式部署
* **高并发**：采用 epoll NIO，效率非常高
* **低内存**：数据结构紧凑、零拷贝
* 配置简单
* 成本低廉
* 支持 Rewrite 重写规则
* 内置健康检查功能
* 节省带宽
* 稳定性高

其他同类Web服务

* **Apache**： Apache HTTP 服务器是一个模块化的服务器，可以运行在几乎所有广泛使用的计算机平台上。其属于应用服务器。Apache支持支持模块多，性能稳定，Apache本身是静态解析，适合静态HTML、图片等，但可以通过扩展脚本、模块等支持动态页面等。 （Apche可以支持PHPcgiperl,但是要使用Java的话，你需要Tomcat在Apache后台支撑，将Java请求由Apache转发给Tomcat处理。） 缺点：配置相对复杂，自身不支持动态页面。
* **Tomcat**： Tomcat是应用（Java）服务器，它只是一个Servlet(JSP也翻译成Servlet)容器，可以认为是Apache的扩展，但是可以独立于Apache运行。

# 正向代理和反向代理

## 正向代理
类似于一个跳板机，通过代理访问外部资源

![20191216161032](http://tc.llx-cn.com/20191216161032.png)

## 反向代理

实际运行方式是指代理服务器来接收 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求的客户端，此时代理服务器对外就表现为一台服务器。

![20191216161054](http://tc.llx-cn.com/20191216161054.png)

## 反向代理作用

* 保证内网安全：可以使用反向代理提供的WAF功能，阻止web攻击。大型网站，通常将反向代理提供给公网访问，而真正的Web服务则是部署在内网中。
* 负载均衡：通过反向代理来优化网站的负载。


![20191216161106](http://tc.llx-cn.com/20191216161106.png)


# 负载均衡策略

## Round Robin （轮询）
最基本的配置就是轮询的方式。根据Nginx配置文件中的顺序，依次将请求分配到不同的后端服务器上。



* 缺省配置就是轮询策略。
* nginx负载均衡支持http和https协议，只需要修改 proxy_pass 后协议即可。
* nginx支持FastCGI、uwsgi、SCGI、memcached的负载均衡，只需要将proxy_pass 改为 fastcgi_pass，uwsgi_pass、scgi_pass、memcached_pass即可
* 此策略适合服务器配置相当，无状态并且短平快的服务使用
* 如果服务器down掉了，会自动剔除服务器。

## weight（权重）

基于权重的负载均衡（Weighted Load Balancing），这种方式下，我们可以配置Nginx 把请求更多地分发到高配置的后端服务器，把相对较少的请求分发到低配置服务器。

```
#动态服务器组
upstream netbase.com {
  server localhost:8080  weight=2; #tomcat 7.0
  server localhost:8081; #tomcat 8.0
  server localhost:8082  backup; #tomcat 8.5
  server localhost:8083  max_fails=3 fail_timeout=20s; #tomcat 9.0
}
```

在该例子中，weight参数用于指定轮询几率，weight的默认值为1,；weight的数值与访问比率成正比，比如Tomcat 7.0被访问的几率为其他服务器的两倍。

* 权重越高分配到需要处理的请求越多。
* 此策略可以与 least_conn 和 ip_hash 结合使用。
* 此策略比较适合服务器的硬件配置差别比较大的情况


## ip_hash

指定负载均衡器按照基于客户端IP的分配方式，这个方法确保了相同的客户端的请求一直发送到相同的服务器，以保证session会话。

这样每个访客都固定访问一个后端服务器，可以解决session不能跨服务器的问题。

```
#动态服务器组
  upstream netbase.com {
    ip_hash;  #保证每个访客固定访问一个后端服务器
    server localhost:8080  weight=2; #tomcat 7.0
    server localhost:8081;               #tomcat 8.0
    server localhost:8082;              #tomcat 8.5
    server localhost:8083  max_fails=3 fail_timeout=20s; #tomcat 9.0
  }
```

注意：

* 在nginx版本1.3.1之前，不能在ip_hash中使用权重（weight）。
* ip_hash不能与backup同时使用。
* 此策略适合有状态服务，比如session。
* 当有服务器需要剔除，必须手动down掉。

## least_conn（最少连接）

把请求转发给连接数较少的后端服务器。轮询算法是把请求平均的转发给各个后端，使它们的负载大致相同；

但是，有些请求占用的时间很长，会导致其所在的后端负载较高。这种情况下，least_conn这种方式就可以达到更好的负载均衡效果。


```
#动态服务器组
upstream netbase.com {
  least_conn;  #把请求转发给连接数较少的后端服务器
  server localhost:8080  weight=2;   #tomcat 7.0
  server localhost:8081;                 #tomcat 8.0
  server localhost:8082 backup;      #tomcat 8.5
  server localhost:8083  max_fails=3 fail_timeout=20s; #tomcat 9.0
}
```

此负载均衡策略适合请求处理时间长短不一造成服务器过载的情况。

## 第三方
### fair

按照服务器端的响应时间来分配请求，响应时间短的优先分配。

```
#动态服务器组
upstream netbase.com {
  server localhost:8080; #tomcat 7.0
  server localhost:8081; #tomcat 8.0
  server localhost:8082; #tomcat 8.5
  server localhost:8083; #tomcat 9.0
  fair;  #实现响应时间短的优先分配
}
```

### url_hash

按照访问 url 的 hash 结果来分配请求，使每个url定向到同一个后端服务器，要配合缓存命中来使用。

同一个资源多次请求，可能会到达不同的服务器上，导致不必要的多次下载，缓存命中率不高，以及一些资源时间的浪费。

而使用url_hash，可以使得同一个url（也就是同一个资源请求）会到达同一台服务器，一旦缓存住了资源，再此收到请求，就可以从缓存中读取。　

```
#动态服务器组
upstream dynamic_zuoyu {
  hash $request_uri;  #实现每个url定向到同一个后端服务器
  server localhost:8080; #tomcat 7.0
  server localhost:8081; #tomcat 8.0
  server localhost:8082; #tomcat 8.5
  server localhost:8083; #tomcat 9.0
}
```



# 代理缓存机制


nginx的http_proxy模块，可以实现类似于Squild的缓存功能

nginx对客户已经访问过的内容在nginx服务器本地建立副本，这样在一段时间内再次访问该数据，就不需要通过nginx服务器再次向后端服务器发出请求，所以能够减少nginx服务器与后端服务器之间的网络流量，减轻网络拥塞，同时还能减小数据传输延迟，提高用户访问速度。


同时，当后端服务器宕机时，nginx服务器上的副本资源还能够回应相关的用户请求，这样能够提高后端服务器的鲁棒性（健壮性）。

![20191216161153](http://tc.llx-cn.com/20191216161153.png)

## 缓存文件放哪里？

| 名称 | 说明 |
| --- | --- |
| proxy_cache_path | 使用该参数指定缓存位置 |
| proxy_cache | 该参数为之前指定的缓存名称 |


proxy_cache_path 有两个必填参数：

第一个参数：缓存目录
第二个参数：keys_zone指定缓存名称和占用内存空间的大小

注意：下面示例中的10m是对内存中缓存内容元数据信息大小的限值，如果想限值缓存总量大小，需要用max_size参数




## 如何指定哪些请求被缓存？

* nginx默认会缓存所有get和head方法的请求结果，缓存的key默认使用请求字符串。
* 自定义key：例如proxy_cache_key hosthostrequest_uri$cookie_user
* 指定请求至少被发送了多少次以上时才缓存，可以防止低频请求被缓存例如 proxy_cache_min_uses 5
* 指定哪些方法的请求被缓存：例如proxy_cache_methods GET HEAD POST

## 缓存的有效期是多久？


默认情况下，缓存内容长期留存，除法缓存的总量超出限制。可以指定缓存有效时间，例如：
//响应状态码为200 302时，10分钟有效
proxy_cache_valid 200 302 10m
//对应任何状态码，5分钟有效
proxy_cache_valid any 5m




## 对于某些请求是否可以不走缓存？

proxy_cache_bypass：该指令响应来自原始服务器而不是缓存

//如果任何一个参数值不为空，或者不等于0，nginx就不会查找缓存，直接进行代理转发
proxy_cache_bypass cookienocachecookienocachearg_nocache $arg_comment



网页的缓存是由HTTP消息头“Cache-control”来控制的，常见的取值有public、private、no-cache、max-age、must-revalidate等，默认为private。其作用根据不同的重新浏览方式分为下面几种情况。



# 通过Lua扩展Nginx


nginx模块需要用C开发，而且必须符合一系列复杂的规则，用C开发模块必须熟悉nginx的源代码，使得开发者望而生畏。

ngx_lua模块通过将lua解释器集成进nginx，可以采用lua脚本实现业务逻辑。

该模块具备以下特性：

* **高并发、非阻塞的处理各种请求**
* **Lua内建协程，可以很好的将异步回调转换成顺序调用的形式**
* **每个协程都有一个独立的全局环境（变量空间），继承全局共享的、只读的comman data**


得益于Lua协程的支持，ngx_lua在处理10000个并发请求时只需要很少的内存，非常适合用于实现可扩展的、高并发的服务。

协程（Coroutine）  进程是资源分配的基本单位，线程是资源调度的基本单位。

协程类似一种多线程，与多线程的区别有：

1. **协程并发os线程，所以创建、切换开销比线程相对要小。**
2. **协程与线程一样有自己的栈、局部变量等，但是协程的栈是在用户进程空间模拟的，所以创建、切换开销很小。**
3. **多线程程序是多个线程并发执行，也就是说在一瞬间有多个控制流在执行。而协程强调的是一种多个协程键协作的关系，只有当一个协程主动放弃执行权，另一个协程才能获得执行权，所以在某一瞬间，多个协程间只有一个在运行。**
4. **由于多个协程只有一个在运行，所以对于临界区的访问不需要加锁，而多线程的情况则必须加锁。**
5. **多线程程序由于有多个控制流，所以程序的行为不可控，而多个协程的执行是由开发者定义的所以是可控的。**


nginx的每个Worker进程都是在epoll或kqueue这样的事件模型上，封装成协程，每个请求都有一个协程进行处理。这正好与Lua内建协程的模型是一致的，所以即使ngx_lua需要执行Lua，相对C有一定的开销，但依然能保证高并发能力。

## Nginx进程模型


nginx采用多进程模型，单Master-多Worker，Master进程主要用了管理Worker进程。


Worker进程采用单线程、非阻塞的事件模型（Event Loop，事件循环）来实现端口的监听及客户端请求的处理和响应，同时Worker还要处理来自Master的信号。Worker进程个数一般设置为机器CPU核数。 


Master进程具体包括如下4个主要功能：


Master进程具体包括如下4个主要功能：
* 接收来自外界的信号
* 向各worker进程发送信号
* 监控worker进程的运行状态
* 当worker进程退出后（异常情况下），会自动重新启动新的worker进程

![20191216161217](http://tc.llx-cn.com/20191216161217.png)



## ngx_lua指令


属于nginx的一部分，它的执行指令都包含在nginx的11个步骤中，相应的处理阶段可以做插入式处理，即可插拔式架构，不过ngx_lua并不是所有阶段都会运行的；

另外指令可以在http、server if、location、location if几个范围进行配置：



## OpenResty


是一个基于nginx与Lua的高性能Web平台，其内部集成了大量精良的Lua库、第三方模块，以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态Web应用、Web服务和动态网关。


**工作原理**：通过汇聚各种设计精良的nginx模块，从而将nginx有效地变成一个强大的通用Web应用平台。这样，Web开发人员和系统工程师可以使用Lua脚本语言调动nginx支持的各种C以及Lua模块，快速构造出足以胜任10K乃至1000K以上单机并发连接的高性能Web应用系统。

**目标**：让你的Web服务直接跑在nginx服务内部，充分利用nginx的非阻塞I/O模型，不仅仅对HTTP客户端请求，甚至对于远程后端诸如MySQL、PostgreSQL、Memcached以及Redis等都进行一致的高性能响应。


* OpenResty是个package，打包了nginx和各种精良的库
* 将简单的转发工作，扩充为可以编写动态脚本
* nginx转变为业务服务器，可以进行增删改查，可以进行业务处理


content_by_lua：内容处理器，接收请求处理并输出响应。

该指令工作在nginx处理流程的content阶段，即内容生产阶段，是所有请求处理阶段中最为重要的阶段，因为这个阶段的指令通常是用来生成HTTP响应内容的