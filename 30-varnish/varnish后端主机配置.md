# 30.5 varnish 后端主机配置
在讲解完 varnish 的缓存配置之后，我们来看看如何配置后端服务器，包括后端服务器组的定义，调度算法，以及健康状态检测。

## 1. 后端主机配置
### 1.1 定义后端主机
定义后端主机使用 `backend` 关键字，在定义主机的同时，我们可以设置后端主机的多项属性。

```
backend BE_NAME {
  .host =                       
  .port =
  .probe =                      # 指定健康状态检测方法
  .connect_timeout = 0.5s;
  .first_byte_timeout = 20s;
  .between_bytes_timeout = 5s;
  .max_connections = 50;
}
```

### 1.2 读写分离
```
# 定义后端主机
backend default {
  .host = "172.16.100.6";
  .port = "80";
}

backend appsrv {
  .host = "172.16.100.7";
  .port = "80";
}

# 对后端主机进行读写分离
sub vcl_recv {
  if (req.url ~ "(?i)\.php$") {
    set req.backend_hint = appsrv;
  } else {
    set req.backend_hint = default;
  }

  ...
}
```

### 1.2 定义服务器组
定义后端服务器组，并配置调度算法需要导入 varnish 的 `Director` 模块，使用 `import directors；`

```
# 1. 导入模块
import directors;    # load the directors

# 2. 定义后端主机
backend server1 {
  .host =
  .port =
}
backend server2 {
  .host =
  .port =
}

# 3. 服务器组定义，并指定调度算法
sub vcl_init {
  new GROUP_NAME = directors.round_robin();
  GROUP_NAME.add_backend(server1);
  GROUP_NAME.add_backend(server2);
}

# 4. 使用服务器组
sub vcl_recv {
  # send all traffic to the bar director:
  set req.backend_hint = GROUP_NAME.backend();
}

# 5. 基于cookie的session sticky：
sub vcl_init {
  new h = directors.hash();
  h.add_backend(one, 1);   // backend 'one' with weight '1'
  h.add_backend(two, 1);   // backend 'two' with weight '1'
}

sub vcl_recv {
  // pick a backend based on the cookie header of the client
  set req.backend_hint = h.backend(req.http.cookie);
}
```

### 1.3 健康状态检测
后端服务器的健康状态检测有两种方式
1. 使用专用 probe 关键词，定义检测方式，可复用
2. 在使用 backend 定义后端主机时通过 `.probe` 单独指定

#### probe 配置参数
```
probe PB_NAME  {
  .url                ：检测时要请求的URL，默认为”/";
  .request            ：发出的具体请求；
    .request =
      "GET /.healthtest.html HTTP/1.1"
      "Host: www.magedu.com"
      "Connection: close"
  .window             ：基于最近的多少次检查来判断其健康状态；
  .threshold          ：最近.window次检查中至有.threshhold定义的次数是成功的；
  .interval           ：检测频度；
  .timeout            ：超时时长；
  .expected_response  ：期望的响应码，默认为200；
}

```

#### 示例
```
probe check {
  .url = "/.healthcheck.html";
  .window = 5;
  .threshold = 4;
  .interval = 2s;
  .timeout = 1s;
}

backend default {
  .host = "10.1.0.68";
  .port = "80";
  .probe = {
    .url = "/.healthcheck.html";
    .window = 5;
    .threshold = 4;
    .interval = 2s;
    .timeout = 1s;
  }
}

backend appsrv {
  .host = "10.1.0.69";
  .port = "80";
  .probe = check;
}
```
