# 30.6 varnish 日志查看
varnish 的日志存放在特定的内存区域中，分为计数器和日志信息两个部分，查看日志需要专门的工具。本节我们来学习这部分命令的使用。

## 1. varnishstat
`varnishstat options`
- 作用: 显示 varnish 缓存的计数器
- 选项:
    - `-1`: 一次输出所有统计计数
    - `-f field_list `: 列出特定的统计字段，field 支持通配符， `^` 开头表示取反
    - `-l`: 列出可供 `-f` 选择的所有字段      
    - `-n varnish_name`: 从哪个 varnish 实例获取日志    
    - `-V`: 显示 varnish 的版本信息
    - `-w delay`: 间隔输出，可结合`-1, -x , -j` 使用
    - `-x`: 以 xml 格式输出
    - `-j`: 以 json 格式输出

```
varnishstat -1 -f MAIN.cache_hit -f MAIN.cache_miss
varnishstat -l -f MAIN -f MEMPOOL
```

## 2. varnishtop
`varnishtop options`
- 作用: 对 varnish 日志排序后输出
- 选项:
    - ` [-1]`:                      运行一次，输出所有日志排序后的结果
    - ` [-b]`:                      Only display backend records
    - ` [-c]`:                      Only display client records
    - ` [-f]`:                      First field only
    - ` [-i taglist]`:              Include tags
    - ` [-I <[taglist:]:regex>]`:   Include by regex
    - ` [-x taglist]`:              Exclude tags
    - ` [-X <[taglist:]:regex>]`:   Exclude by regex
    - ` [-V]`:                      Version



## 3. varnishlog
`varnishlog options`
- 作用: 显示 varnish 日志

## 4. varnishncsa
`varnishncsa`
- 作用: 以类似 httpd combined 格式显示 varnish 日志


## 5. 日志记录服务
varnish 的日志只会记录在内存中，空间不够用时，新的日志就会覆盖老的日志。因此需要定期执行 `varnishlog` 或  `varnishncsa` 保存日志。yum 安装已经自动为我们配置了 Unit file，启用下面两个服务之一即可:
1. `/usr/lib/systemd/system/varnishlog.service`
2. `/usr/lib/systemd/system/varnishncsa.service`
 
