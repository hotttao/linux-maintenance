# 26.1 IO模型
为了应对"C10K" 问题，

## 1. IO模型：
- 同步/异步: 关注的是被调用者，如何通知调用者 -- 消息通知机制
- 阻塞/非阻塞: 关注的是调用者如何等待结果
### 1.1 IO 模型类别
1. 同步阻塞
2. 同步非阻塞
3. IO复用：select(), poll():
4. 事件驱动I/O
5. 异步IO


### 1.2 httpd 的IO 模型
1. 多进程模型：prefork, 一个进程响应一个用户请求，并发使用多个进程实现；
2. 多线程模型：worker, 一个进程生成多个线程，一个线程响应一个用户请求；并发使用多个线程实现；n进程，n*m个线程；
3. 事件模型：event, 一个线程响应多个用户请求，基于事件驱动机制来维持多个用户请求；

## 2. 网站访问统计
术语:
1. PV: page view
2. UV: usere view

浏览器:
1. 对每个域名的并发请求数量存在上线
2. 通过将资源放置在多个域名下，能够增加浏览器并发请求的数量，以提高页面访问速度


### 3. IO 模型
- 阻塞/非阻塞: 程序的执行模式
- 同步/异步: 应用程序与操作系统的交互方式
- IO 事件模型: `select` + `poll` + `epool`
- 参考连接:
  - https://songlee24.github.io/2016/07/19/explanation-of-5-IO-models/
  - https://blog.csdn.net/wuzhengfei1112/article/details/78242004
  - https://blog.csdn.net/lijinqi1987/article/details/71214974
