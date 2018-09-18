# 30.4 varnish缓存策略配置
前面我们讲解了 VCL 的语法，并通过示例讲解了一部分 varnish 缓存的配置。本节我们来看看 varnish 内置的缓存策略，然后着重来看看如何对缓存进行修剪。

## 1. VCL 域默认配置
### 1.1 vcl_recv

```
# vcl_recv 默认配置
sub vcl_recv {
  if (req.method == "PRI") {
    /* We do not support SPDY or HTTP/2.0 */
    return (synth(405));
  }
  if (req.method != "GET" &&
  req.method != "HEAD" &&
  req.method != "PUT" &&
  req.method != "POST" &&
  req.method != "TRACE" &&
  req.method != "OPTIONS" &&
  req.method != "DELETE") {
    /* Non-RFC2616 or CONNECT which is weird. */
    return (pipe);
  }

  if (req.method != "GET" && req.method != "HEAD") {
    /* We only deal with GET and HEAD by default */
    return (pass);
  }
  if (req.http.Authorization || req.http.Cookie) {
    /* Not cacheable by default */
    return (pass);
  }
    return (hash);
  }
}
```

## 2. 缓存修剪
varnish 让缓存过期有多种方式，最常见的是通过 purge, 和ban
1. purge: 是在 varnish 内置监视一个特殊的 PURGE 请求方法，并对请求的资源进行缓存请求
2. ban: 可以使用类似 purge 的方式，更常用的实在 varnishadm 内使用正则表达式对特定一组资源进行缓存处理

### 2.1 使用 purge
```
# 1. 添加修剪操作的访问控制
acl purgers {
  "127.0.0.0"/8;
  "10.1.0.0"/16;
}

# 2. 如何执行修剪，即能执行purge操作
sub vcl_purge {
  return (synth(200,"Purged"));
}

# 3.何时执行purge操作
sub vcl_recv {
  if (req.method == "PURGE") {
    if (!client.ip ~ purgers) {   # 访问控制
      return(synth(405,"Purging not allowed for " + client.ip));
    }
    return(purge);
}
```

### 2.2 使用 Banning
在 varnishadm 中使用 ban 子命令，可以使用正则表达式批量修剪
- command: `ban <field> <operator> <arg>`
- eg: `ban req.url ~ ^/javascripts`


在配置文件中定义，使用ban()函数；

```
if (req.method == "BAN") {
  ban("req.http.host == " + req.http.host + " && req.url == " + req.url);
  # Throw a synthetic page so the request won't go to the backend.
  return(synth(200, "Ban added"));
}
```
