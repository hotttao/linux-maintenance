# 26. nginx入门及企业实战应用
nginx 是一个 web 服务器，同时还能作为 http 协议的反向代理服务器。相比于 http，nginx 使用了更先进的 IO 模型，异步通信以及进程间通信的技术，能支持更多的并发请求，具有更高的性能和稳定性。本章我们首先来学习如何使用 nginx 配置一个 web serve，nginx 的反向代理功能我们留在 28 章再来介绍。本章内容包括
1. IO 事件模型
2. nginx 框架与配置
3. nginx web 服务配置

有关 web 的基础概念和 http 协议的内容将不再此累述，大家可以回看以下几个章节:
- [20.1 web基础概念](20-web-apache/web基础概念.md)
- [20.2 http协议基础](20-web-apache/http协议基础.md)
- [20.3 http协议进阶](20-web-apache/http协议进阶.md)
