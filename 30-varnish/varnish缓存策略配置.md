# 30.4 varnish缓存策略配置

## 3. VCL 域
### 3.1 vcl_recv

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


缓存对象的修剪：purge, ban
  (1) 能执行purge操作
    sub vcl_purge {
      return (synth(200,"Purged"));
    }

  (2) 何时执行purge操作
    sub vcl_recv {
      if (req.method == "PURGE") {
        return(purge);
      }
      ...
    }

  添加此类请求的访问控制法则：
    acl purgers {
      "127.0.0.0"/8;
      "10.1.0.0"/16;
    }

    sub vcl_recv {
      if (req.method == "PURGE") {
        if (!client.ip ~ purgers) {
          return(synth(405,"Purging not allowed for " + client.ip));
        }
        return(purge);
      }
