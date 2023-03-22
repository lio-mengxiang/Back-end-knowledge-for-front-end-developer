## 前言

题目均是真题，职位为Node.js开发、全栈（node.js + react）、大前端（需要会node.js的前端职位），最近6个月的面试题（大小厂都有，总体来说，小厂问的题都是抄的大厂的题），关于nginx的。


感慨一下：其实在后端运维这一层，k8s本身的很多功能，可以说把以前的很多技术都简化了，比如你做架构，要什么自动扩缩容，滚动升级（热更新），在k8s里面都是很简单的配置。

在以前，很多高级的技术实现都要分散在其它的技术栈。比如nginx的负载均衡，加上在遇到机器挂掉，设置将流量负载到另一台机器，我们就需要额外的学习这些知识，而现在有了k8s，我们在运维这块集中学习k8s即可（仅仅熟练使用api就可以在小公司为所欲为了）

本文收录在[node.js大前端面试题的github项目中]()

## 看你简历上写着使用过nginx，它在你们业务里承担什么角色，为什么要用？

答：

首先，我曾经经历过一个小型公司，需要从单机架构到集群架构的转变，需要一个实现负载均衡和反向代理功能的软件，加上nginx可以非常轻松的解决掉很多前端常见的需求，比如gzip压缩，跨域等等问题，所以选择了ng。

#### 面试官: 你说到了单机架构到集群架构，单机架构有有什么缺点，让你们产生这样的架构转变。

- 单点故障问题，如果服务崩溃，会造成所有服务不可用，服务稳定性极低。集群可以解决这个问题，也就是具有高可用性。
- 耦合度太高，所有服务在一台服务器上，或者说都在一份代码里，可拓展性就比较低
- 单节点并发能力有限，用户量增多，如果流量大了，可能需要负载均衡，让多个服务器来分担流量，集群可以解决这个问题，在流量大的时候，自动扩缩容，也就是具有高可拓展性（借助k8s）

#### 面试官: 你们有没有遇到一些坑，比如单机架构的某个功能，在集群架构下就没法用了

有，比如说：

- session。单体架构比如session可以存在内存里（不建议这么做，因为非常容易造成内存占用过高），在集群环境，这样的方案无法跨服务共享session。

    - 可以使用redis实现分布式的session。
- 定时任务。单体架构的定时任务可能会在每台机器上都搞一遍，所以可能需要单独起一个定时任务的服务，独立部署。
    - 可以使用k8s来解决，k8s有专门的定时器控制器。

## 你提到你们使用了nginx的反向代理功能，那什么是正向代理，什么是反向代理？

首先代理是指，客户端和服务端之间有一个代理服务器，其中代理服务器承担什么样的角色，决定了它是正向代理还是反向代理。

正向代理总结就一句话：代理端代理的是客户端。比如我们翻墙。
反向代理总结一句话：反向代理就是服务端。比如用户访问我们的nginx。


## 你还提到说，你们使用nginx实现了负载均衡，负载均衡有哪些算法

题外话：其实怎么说呢，k8s改变了以前的很多应用的使用方式，k8s本身自带负载均衡，所以如果使用k8s的话，nginx更多的是承担了反向代理的作用，还有动静资源分离，对于负载均衡的功能交给k8s更适合。

nginx的upstream（用来实现负载均衡的字段配置）目前支持的5种方式的分配

1、轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

```
upstream backserver { 
server 192.168.0.14; 
server 192.168.0.15; 
} 
```

2、指定权重

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

```
upstream backserver { 
server 192.168.0.14 weight=8; 
server 192.168.0.15 weight=10; 
} 
```

3、IP绑定 ip_hash

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

```
upstream backserver { 
ip_hash; 
server 192.168.0.14:88; 
server 192.168.0.15:80; 
} 
```

4、url_hash（第三方）

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

```
upstream backserver { 
server squid1:3128; 
server squid2:3128; 
hash $request_uri; 
hash_method crc32; 
} 
```

## Nginx为什么采用多进程结构而不是多线程

- 更安全

主要是因为ng要保持高可用性，多线程结构中，多个线程间是共享内存空间的，如果一个第三方模块的代码导致内存空间发生某些错误时，会导致整个ng进程挂掉。而采用多进程就不会出现这样的
- 性能更好

并且对比Apache。

**Apache** **:** 创建多个进程或线程，而每个进程或线程都会为其分配 cpu 和内存（线程要比进程小的多，所以 worker 支持比 perfork 高的并发），并发过大会榨干服务器资源。

**Nginx:** 采用单线程来异步非阻塞处理请求（管理员可以配置 Nginx 主进程的工作进程的数量）(epoll)，不会为每个请求分配 cpu 和内存资源，节省了大量资源，同时也减少了大量的 CPU 的上下文切换。所以才使得 Nginx 支持更高的并发。

##  Nginx怎么处理请求的？

（官方写的处理请求有11个阶段，但是我认为实际上理解以下7个阶段就足够了）

一个请求过来，nginx会：

1.  Read Request Headers：解析请求头。（知晓是哪个server，也就是哪个ip和端口号去处理）
1.  Identify Configuration Block：识别由哪一个 location 进行处理，匹配 URL。
1.  Apply Rate Limits：判断是否限速。例如可能这个请求并发的连接数太多超过了限制，或者 QPS 太高。
1.  Perform Authentication：连接控制，验证请求。例如可能根据 Referrer 头部做一些防盗链的设置，或者验证用户的权限。
1.  Generate Content：生成返回给用户的响应。为了生成这个响应，做反向代理的时候可能会和上游服务（Upstream Services）进行通信，然后这个过程中还可能会有些子请求或者重定向，那么还会走一下这个过程（Internal redirects and subrequests）。
1.  Response Filters：过滤返回给用户的响应。比如压缩响应，或者对图片进行处理。
1.  Log：记录日志。

##  Nginx 是如何实现高并发的？

1.  事件驱动（主要是网络I/O的读事件和写事件）
    - nginx跟node.js很相似的一点，就是也有事件循环，目的是使用异步非阻塞的I/O模式去解决请求高并发的问题
    - 我们需要知道，事件其实是操作系统的内核产生的，然后给应用去是使用，但是例如ng有100万个连接，只有某些连接产生数据的时候，才会把这些数据给ng，nginx（node.js）怎么知道这个事件是给ng的？正常思路就是我把这100万个连接相关的fd都遍历一遍，哪些产生数据就给ng，这样无疑是低效的。ng目前和node都是使用epoll来解决
    - epoll之所以**高效**, 是因为存在这样一个现实，在建立大量TCP连接的服务端，真正处于**活跃状态**的TCP连接的数量很少，而select 或者poll 每次要读写数据的时候都需要把所有的TCP连接丢给操作系统判断是否可以读写，因此浪费了大量时间。而eopll会维护一个正在活跃状态的fd的数据结构，正样遍历消耗就减少非常多。

3.  多进程模型：Nginx 采用多进程/线程模型。
    - 正如上面说的，比如在多进程架构中，每个进程都有自己独立的内存空间，这就意味着一个进程的崩溃不会影响其他进程的正常运行。

5.  负载均衡：Nginx 作为反向代理服务器。
    - 可以通过负载均衡算法将请求分发到多个后端服务器上，避免单个服务器的过载，提高整个系统的并发处理能力。
7.  静态资源缓存。
    - Nginx 可以将静态资源缓存到内存或磁盘中，避免每次请求都要重新生成或读取资源，提高了访问速度和并发能力。而且nginx还有零拷贝技术，在传输静态资源的时候直接从例如磁盘读到网卡，而不用经过内存。

  


## nginx作为反向代理服务器的优点是什么?

1.  高性能：nginx采用了异步非阻塞的工作方式，可以在不占用过多资源的情况下处理大量的并发请求，从而保证了服务器的高性能。此外，nginx还支持多线程和负载均衡等功能，可以进一步提高服务器的性能。
1.  高可靠性：nginx具有较好的稳定性和可靠性，可以有效防止服务器因为突发流量而崩溃。nginx还支持动态配置，可以实时地对服务器进行调整，从而保证服务器的稳定性和可靠性。
1.  负载均衡：nginx可以实现负载均衡，将请求分发到不同的服务器上进行处理，从而提高了系统的并发处理能力和可扩展性。
1.  缓存机制：nginx支持缓存机制，可以将常用的数据缓存在内存中，从而加快数据的访问速度，降低服务器的负载。
1.  安全性：nginx具有较好的安全性，可以对HTTP请求进行访问控制和防御攻击，如DDoS攻击、SQL注入等，从而保障服务器的安全性。
1. 方便水平拓展，比如新增别的服务，直接在ng的代理配置文件中加入一条规则即可

## 如何实现nginx的动静分离，为什么要这么做？

目的是静态资源一般都不会改变（前端目前使用不同的content hash作为打包后文件的名的一部分，所以可以放心大胆的把这些文件缓存起来）。可以让我们提供的web服务更加高效。

配置动静分离很简单，主要是匹配静态资源，然后设置缓存响应头即可（比如 expires，catch-control）。

```
location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css){
  root   /soft/nginx/static_resources;
  expires 7d;
}
```

注意这里的nginx的expires 7d;会在响应头里加入expires，catch-control两个响应头。

还需要注意，expire是把缓存放到了浏览器里，nginx本身也可以缓存静态文件，需要设置proxy_cache



## Nginx目录结构有哪些？

-   /etc/nginx/: nginx的主要配置文件存放目录。
-   /etc/nginx/nginx.conf: nginx的主要配置文件，包含了http、server、location等模块的配置。
-   /etc/nginx/conf.d/: 存放其他配置文件的目录。
-   /var/log/nginx/: 存放nginx日志的目录。
-   /var/cache/nginx/: 存放nginx缓存文件的目录。
-   /usr/share/nginx/: 存放nginx默认网页文件的目录。
-   /usr/share/nginx/html: 存放默认主页文件的目录。

## Nginx如何实现热更新

Nginx支持热更新配置文件和重载（reload）服务进程，而不需要停止服务。

以下是Nginx进行热更新的步骤：

1.  编辑新的配置文件

通过修改Nginx的配置文件来更新服务器设置。您可以通过编辑配置文件进行更改，例如更改端口号、添加新的虚拟主机等。

2.  检查配置文件是否正确

在应用新的配置文件之前，需要检查配置文件是否正确，以避免出现错误。您可以使用以下命令检查配置文件的语法：

```
sudo nginx -t
```

3.  重载Nginx服务

如果新的配置文件没有错误，则可以重新加载Nginx服务以使更改生效。使用以下命令重新加载Nginx服务：

```
sudo nginx -s reload
```

##  Nginx配置文件nginx.conf有哪些属性模块?
- main: 全局配置
    - event: 配置工作模式以及连接数
        - 比如 epoll和worker_connections
    - http: http模块相关配置
        - server 虚拟主机配置，可以有多个
            - location 路由规则，表达式
            - upstream 配置集群（负载均衡的规则）


## 如何用Nginx解决前端跨域问题？


```
location / {
  # 允许跨域的请求，可以自定义变量$http_origin，*表示所有
  add_header 'Access-Control-Allow-Origin' *;
  # 允许携带cookie请求
  add_header 'Access-Control-Allow-Credentials' 'true';
  # 允许跨域请求的方法：GET,POST,OPTIONS,PUT
  add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS,PUT';
  # 允许请求时携带的头部信息，*表示所有
  add_header 'Access-Control-Allow-Headers' *;
  # 允许发送按段获取资源的请求
  add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
  # 一定要有！！！否则Post请求无法进行跨域！
  # 在发送Post跨域请求前，会以Options方式发送预检请求，服务器接受时才会正式请求
  if ($request_method = 'OPTIONS') {
      add_header 'Access-Control-Max-Age' 1728000;
      add_header 'Content-Type' 'text/plain; charset=utf-8';
      add_header 'Content-Length' 0;
      # 对于Options方式的请求返回204，表示接受跨域请求
      return 204;
  }
}

```

## nginx限流怎么做的？

1.  限制请求速率：可以使用Nginx的“ngx_http_limit_req_module”模块来限制每个IP地址的请求速率。该模块基于“漏桶算法”实现，可以在指定时间窗口内限制客户端请求的数量。例如：

```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=5r/s;

server {
    location / {
        limit_req zone=mylimit burst=10 nodelay;
        ...
    }
}
```

上述配置将限制每个IP地址的请求速率为每秒5个请求，如果突发请求超过10个，则Nginx将返回“503 Service Unavailable”的HTTP错误代码。

2.  限制并发连接数：可以使用Nginx的“ngx_http_limit_conn_module”模块来限制每个IP地址的并发连接数。该模块基于“令牌桶算法”实现，可以在指定时间窗口内限制客户端的并发连接数。例如：

```
limit_conn_zone $binary_remote_addr zone=mylimit:10m;

server {
    location / {
        limit_conn mylimit 5;
        ...
    }
}
```

上述配置将限制每个IP地址的并发连接数为5个，如果超过限制则Nginx将返回“503 Service Unavailable”的HTTP错误代码。


## 在nginx中，如何使用未定义的服务器名称来阻止处理请求？

在Nginx中，可以使用`server_name`指令来定义服务器名称，如果收到的请求不匹配任何定义的服务器名称，则默认使用`server_name`为`_`的配置。如果你想阻止处理这些未定义的请求，可以在`server_name`为`_`的配置中添加一个特殊的指令`return`，将HTTP状态码设置为`444`，它表示连接被关闭，这将终止连接并阻止Nginx处理请求。例如：

```
markdownCopy code
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name _;

    return 444;
}
```

在上面的示例中，配置了一个默认的服务器块，`server_name`为`_`，并使用`return`指令返回状态码为`444`。这将导致Nginx关闭连接并不响应未定义服务器名称的请求。请注意，如果你的Nginx版本低于0.8.48，那么将无法使用状态码`444`，你需要使用其他状态码如`400`或`403`。


##  怎么限制浏览器访问？

nginx怎么限制浏览器访问？

如果你想要限制浏览器访问你的Nginx服务器，可以使用`$http_user_agent`变量来检查HTTP请求中的User-Agent头部，以识别浏览器类型，然后使用`deny`指令来拒绝来自指定浏览器的请求。以下是一个示例Nginx配置文件：

```
perlCopy code
server {
    listen 80;
    server_name example.com;

    if ($http_user_agent ~* (Firefox|Chrome)) {
        return 403;
    }

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

在上面的示例中，`$http_user_agent`变量用于检查HTTP请求中的User-Agent头部，如果它匹配`Firefox`或`Chrome`，则会返回403 Forbidden错误，否则允许访问`/`路径下的文件。使用`if`指令来匹配User-Agent头部并返回错误响应，这样就可以限制浏览器访问。但是，由于`if`指令在性能上的影响，建议在高流量的生产环境中避免使用。


## Nginx 如何开启压缩？

这个太简单了，就写一个简单的示例吧。

```
http{
  # 开启压缩机制
  gzip on;
  # 指定会被压缩的文件类型(也可自己配置其他类型)
  gzip_types text/plain application/javascript text/css application/xml text/javascript image/jpeg image/gif image/png;
  # 设置压缩级别，越高资源消耗越大，但压缩效果越好
  gzip_comp_level 5;
  # 在头部中添加Vary: Accept-Encoding（建议开启）
  gzip_vary on;
  # 处理压缩请求的缓冲区数量和大小
  gzip_buffers 16 8k;
  # 对于不支持压缩功能的客户端请求不开启压缩机制
  gzip_disable "MSIE [1-6]\."; # 低版本的IE浏览器不支持压缩
  # 设置压缩响应所支持的HTTP最低版本
  gzip_http_version 1.1;
  # 设置触发压缩的最小阈值
  gzip_min_length 2k;
  # 关闭对后端服务器的响应结果进行压缩
  gzip_proxied off;
}
```

##  什么是C10K问题?

C10K问题是指如何在单个服务器上处理超过10,000个并发连接的问题，其中“C”表示并发连接数，而“10K”表示10,000。Apache

## 如何在Nginx中获得当前的时间?

在Nginx中获取当前时间可以使用Nginx内置的变量$time_local。这个变量包含了当前时间的本地表示（通常是格式化为“日/月/年 时:分:秒”的字符串）。

## 用Nginx服务器解释-s的目的是什么?

在Nginx服务器中，使用-s选项可以指定要向服务器发送的信号。这些信号是指令Nginx服务器执行某些特定的操作，而不是向服务器发送HTTP请求。

常用的信号包括：

-   stop：停止Nginx服务器。
-   quit：优雅地停止Nginx服务器。
-   reload：重新加载Nginx配置文件，以便应用最新更改。

## nginx如果location中配置了access_log, 同一server中也配置了access_log以谁为准

以location中为准，简单来说就是，存储值的指令，在子配置不存在时，直接使用父配置块，如果子配置存在时，直接覆盖父配置块

## nginx中的listen，只能指定为端口吗，比如80

错误，还可以指定ip+端口，甚至unix的socket地址。

## 你对nginx做过什么优化

- 增大cpu的利用率
    - 启用多进程的worker（比如等于cpu核心数）
    - 使用缓存，比如提前压缩好文件，减少cpu的压缩时的时间
- 增大内存的利用率
    - 使用零拷贝，不走内存，直接将磁盘数据发送到网卡
- 增大磁盘I/O利用率
    - Nginx可以通过缓存来减少磁盘I/O，从而提高性能
- 增大网络带宽的利用率
    - 启用TCP优化：可以通过调整Nginx的TCP参数，如增大TCP窗口大小
    - 启用HTTP/2：HTTP/2协议可以通过多路复用技术，同时处理多个请求，从而提高网络带宽利用率。
    - 启用gzip压缩：使用gzip压缩可以减少传输的数据量，从而提高网络带宽利用率
    - 通常Nginx作为代理服务，负责分发客户端的请求，那么建议开启`HTTP`长连接，用户减少握手的次数，降低服务器损耗，具体如下：

```
upstream xxx {
    # 长连接数
    keepalive 32;
    # 每个长连接提供的最大请求数
    keepalived_requests 100;
    # 每个长连接没有新的请求时，保持的最长时间
    keepalive_timeout 60s;
}
```

  

## 大文件上传，经过nginx会有什么问题吗？

在某些业务场景中需要传输一些大文件，但大文件传输时往往都会会出现一些`Bug`，比如文件超出限制、文件传输过程中请求超时等，那么此时就可以在`Nginx`稍微做一些配置，先来了解一些关于大文件传输时可能会用的配置项：

|           配置项           |              释义              |
| :---------------------: | :--------------------------: |
|  `client_max_body_size` |         设置请求体允许的最大体积         |
| `client_header_timeout` |       等待客户端发送一个请求头的超时时间      |
|  `client_body_timeout`  |         设置读取请求体的超时时间         |
|   `proxy_read_timeout`  | 设置请求被后端服务器读取时，`Nginx`等待的最长时间 |
> 上述配置仅是作为代理层需要配置的，因为最终客户端传输文件还是直接与后端进行交互，这里只是把作为网关层的`Nginx`配置调高一点，调到能够“容纳大文件”传输的程度。

  



## nginx location的匹配规则是什么？


1.  精确匹配（使用`=`符号开头）的location指令优先级最高。
1.  正则表达式匹配（使用`~`或`~*`符号开头）的location指令优先级次之。
1.  前缀匹配（使用`^~`符号开头）的location指令优先级次于正则表达式匹配。
1.  普通匹配（不使用任何符号）的location指令优先级最低。

如果有多个location指令匹配同一个请求，Nginx将选择优先级最高的匹配项来处理请求。


## nginx配置过https吗
以下是腾讯云建议的https配置：
```
#设定虚拟主机配置
server {
  #侦听443端口，这个是ssl访问端口
  listen    443;
  #定义使用 访问域名
  server_name  XXX.com;
  #定义服务器的默认网站根目录位置
  root /web/www/website/dist;  

  #设定本虚拟主机的访问日志
  access_log  logs/nginx.access.log  main;

  # 这些都是腾讯云推荐的配置，直接拿来用就行了，只是修改证书的路径，注意这些路径是相对于/etc/nginx/nginx.conf文件位置
  ssl on;
  ssl_certificate 1_XXX.com_bundle.crt;
  ssl_certificate_key 2_XXX.com.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
  ssl_prefer_server_ciphers on;

  #默认请求
  location / {     
  root /web/www/website/dist;      
      #定义首页索引文件的名称
      index index.html;
  }
}
```

这里解释一下ssl相关的配置：
1.  `ssl on;`: 启用 SSL。
1.  `ssl_certificate`: 指定 SSL 证书的路径。
1.  `ssl_certificate_key`: 指定 SSL 证书密钥的路径。这是服务器私钥的位置。
1.  `ssl_session_timeout`: 指定 SSL 会话的超时时间。在这个例子中，会话超时时间为 5 分钟。
1.  `ssl_protocols`: 指定 SSL 协议的版本。为什么腾讯云建议是TLS的一些版本呢。这是因为这些协议都是目前被广泛支持的安全协议版本，它们具有良好的安全性和兼容性。比如SSLv2和v3都爆出严重的安全漏洞了。
1.  `ssl_ciphers`: 指定 SSL 加密套件的名称。
1.  `ssl_prefer_server_ciphers`: 启用服务器端的加密套件优先级。

其实我们主要做的就是把证书和私钥配好即可

## root和alias的区别

在nginx中，`root`和`alias`都是用来指定静态资源文件所在的目录的指令。

1.  `root`指令：`root`指令是指定根目录，表示请求的文件相对于根目录的路径。它是默认的情况下使用的指令，一般情况下在location或server中只需要使用root就可以了。

举例：假设根目录为`/usr/share/nginx/html`，当客户端请求`http://example.com/test.html`时，Nginx会在`/usr/share/nginx/html/test.html`中寻找该文件。

```
server {
    listen 80;
    server_name example.com;
    root /usr/share/nginx/html;
}
```

2.  `alias`指令：`alias`指令可以指定某个路径下的资源映射到另外一个路径下，它可以用来隐藏实际的资源路径，从而保证资源的安全性。当使用`alias`指令时，Nginx会将匹配到的location路径中匹配到的部分替换为指定的路径。

举例：假设需要将`/files`目录下的所有文件都映射到`/var/www/files`目录下，可以使用`alias`指令进行配置。

```
location /files {
    alias /var/www/files/;
}
```

在上面的例子中，如果客户端请求`http://example.com/files/test.html`，Nginx会在`/var/www/files/test.html`中寻找该文件。

