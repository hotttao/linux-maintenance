# 28. nginx 7层负载均衡和方向代理
26 章我们讲解了 nginx 作为 web 服务的应用，除了 web 服务功能，nginx 还能作为七层的反向代理实现负载均衡功能。本章我们就来讲解 nginx 的另两项主要功能:
1. 反向代理服务器
2. 负载均衡调度器

nginx 是高度模块化的，只要nginx 具有实现了相关协议的模块，就可以作为相关的反向代理服务器。ngx_http_proxy_module 是 http 反向代理模块，ngx_http_fastcgi_module 是 fastcgi 协议的反代模块。

在介绍 LVS 的负载均衡集群时，我们对 LVS 和 nginx 的负载均衡能力就进行的比较。nginx 作为七层的负载均衡器，能获取应用层的报文信息，因此提供了更多的功能。但是由于工作于用户空间，需要通过套接字与客户端和后端服务器进行交互，所以并发能力受到系统套接字数量的限制。

在讲解 nginx 之前，我们再来回顾一下 LB集群的软件方式
1. 四层调度: lvs, nginx(stream module), haproxy(mode tcp)
2. 七层调度: nginx(http_up_stream module), haproxy(mode http)

nginx 的 http 模块，和 stream 模块都具有 up_stream 模块
1. http 的  up_stream 主要是用来负载均衡 http 服务的
2. stream 本身只是一个能基于四层协议的反代模块，stream 的  up_stream 则是用来负载这类服务的