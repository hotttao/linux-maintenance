# 28.6 nginx线上部署示例

## 1. nginx 线上部署示例
```
user                              nobody nobody;
worker_processes                  4;
worker_rlimit_nofile              51200;
error_log                         logs/error.log  notice;
pid                               /var/run/nginx.pid;
events {
  use                             epoll;
  worker_connections              51200;
}
http {
  server_tokens                   off;
  include                         mime.types;
  proxy_redirect                off;
  proxy_set_header              Host $host;
  proxy_set_header              X-Real-IP $remote_addr;
  proxy_set_header              X-Forwarded-For $proxy_add_x_forwarded_for;
  client_max_body_size          20m;
  client_body_buffer_size       256k;
  proxy_connect_timeout         90;
  proxy_send_timeout            90;
  proxy_read_timeout            90;
  proxy_buffer_size             128k;
  proxy_buffers                 4 64k;
  proxy_busy_buffers_size       128k;
  proxy_temp_file_write_size    128k;
  default_type                    application/octet-stream;
  charset                         utf-8;

  client_body_temp_path           /var/tmp/client_body_temp 1 2;
  proxy_temp_path                 /var/tmp/proxy_temp 1 2;
  fastcgi_temp_path               /var/tmp/fastcgi_temp 1 2;
  uwsgi_temp_path                 /var/tmp/uwsgi_temp 1 2;
  scgi_temp_path                  /var/tmp/scgi_temp 1 2;
  ignore_invalid_headers          on;
  server_names_hash_max_size      256;
  server_names_hash_bucket_size   64;
  client_header_buffer_size       8k;
  large_client_header_buffers     4 32k;
  connection_pool_size            256;
  request_pool_size               64k;
  output_buffers                  2 128k;
  postpone_output                 1460;
  client_header_timeout           1m;
  client_body_timeout             3m;
  send_timeout                    3m;
  log_format main                 '$server_addr $remote_addr [$time_local] $msec+$connection '
                                  '"$request" $status $connection $request_time $body_bytes_sent "$http_referer" '
                                  '"$http_user_agent" "$http_x_forwarded_for"';
  open_log_file_cache               max=1000 inactive=20s min_uses=1 valid=1m;
  access_log                      logs/access.log      main;
  log_not_found                   on;
  sendfile                        on;
  tcp_nodelay                     on;
  tcp_nopush                      off;
  reset_timedout_connection       on;
  keepalive_timeout               10 5;
  keepalive_requests              100;
  gzip                            on;
  gzip_http_version               1.1;
  gzip_vary                       on;
  gzip_proxied                    any;
  gzip_min_length                 1024;
  gzip_comp_level                 6;
  gzip_buffers                    16 8k;
  gzip_proxied                    expired no-cache no-store private auth no_last_modified no_etag;
  gzip_types                      text/plain application/x-javascript text/css application/xml application/json;
  gzip_disable                    "MSIE [1-6]\.(?!.*SV1)";
  upstream tomcat8080 {
    ip_hash;
    server                        172.16.100.103:8080 weight=1 max_fails=2;
    server                        172.16.100.104:8080 weight=1 max_fails=2;
    server                        172.16.100.105:8080 weight=1 max_fails=2;
  }
  server {
    listen                        80;
    server_name                   www.magedu.com;
    # config_apps_begin
    root                          /data/webapps/htdocs;
    access_log                    /var/logs/webapp.access.log     main;
    error_log                     /var/logs/webapp.error.log      notice;
    location / {

      location ~* ^.*/favicon.ico$ {
        root                      /data/webapps;
        expires                   180d;
        break;
      }

      if ( !-f $request_filename ) {
        proxy_pass                http://tomcat8080;
        break;
      }
    }
    error_page                    500 502 503 504  /50x.html;
      location = /50x.html {
      root                        html;
    }
  }
  server {
    listen                        8088;
    server_name                   nginx_status;
      location / {
          access_log                  off;
          deny                        all;
          return                      503;
      }
      location /status {
          stub_status                 on;
          access_log                  off;
          allow                       127.0.0.1;
          allow                       172.16.100.71;
          deny                        all;
      }
  }
}
```
## 2. tornado 配置
```
upstream pyapi_prod {
    server 10.80.85.26:9999 max_fails=3 fail_timeout=20s;
}
upstream pyapi_pre {
    server 10.81.33.246:9999 max_fails=3 fail_timeout=20s;
}
upstream prediction_prod {
    server 10.47.208.181:8083 max_fails=3 fail_timeout=20s;
}
upstream crawlerLink_prod {
    server 10.47.208.181:8086 max_fails=3 fail_timeout=20s;
}
upstream crawlerLink_pre {
    server 10.47.208.181:8087 max_fails=3 fail_timeout=20s;
}
server {
    #include drop.conf;
    listen 80;
    server_name pyapi.internal.enlightent.com pyapi.enlightent.com;
    location /prediction/ {
        proxy_pass http://prediction_prod/;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-SSL-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-HTTPS-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
    location /crawlerLink/ {
        proxy_pass http://crawlerLink_prod/;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-SSL-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-HTTPS-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
    location / {
        proxy_pass http://pyapi_prod;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-SSL-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-HTTPS-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
}
server {
    #include drop.conf;
    listen 80;
    server_name pre.pyapi.internal.enlightent.com pre.pyapi.enlightent.com;
    location / {
        proxy_pass http://pyapi_pre;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-SSL-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-HTTPS-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
    location /prediction/ {
        proxy_pass http://prediction_prod/;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-SSL-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-HTTPS-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
    location /crawlerLink/ {
        proxy_pass http://crawlerLink_pre/;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-SSL-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-HTTPS-Protocol $ssl_protocol;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
}
```
## 3. pycgi
```
<VirtualHost *:8083>
	ServerName pycgi.internal.enlightent.com
	DocumentRoot "/home/yunheadmin/yunhetools/python-cgi/model/prediction_play_times"
	<IfModule alias_module>
		ScriptAlias /cgi-bin/ "/home/yunheadmin/yunhetools/python-cgi/model/prediction_play_times/"
	</IfModule>
	<Directory "/home/yunheadmin/yunhetools/python-cgi/model/prediction_play_times/">
		AllowOverride None
		Options Indexes FollowSymLinks ExecCGI
		Require all granted
		Require host ip
	</Directory>
</VirtualHost>
<VirtualHost *:8084>
	ServerName pycgi.internal.enlightent.com
	DocumentRoot "/home/yunheadmin/yunhetools/python-cgi/weixinartical/prod/weixinartical/monitor"
	<IfModule alias_module>
		ScriptAlias /cgi-bin/ "/home/yunheadmin/yunhetools/python-cgi/weixinartical/prod/weixinartical/monitor/"
	</IfModule>
	<Directory "/home/yunheadmin/yunhetools/python-cgi/weixinartical/prod/weixinartical/monitor/">
		AllowOverride None
		Options Indexes FollowSymLinks ExecCGI
		Require all granted
		Require host ip
	</Directory>
</VirtualHost>
<VirtualHost *:8085>
	ServerName pre.pycgi.internal.enlightent.com
	DocumentRoot "/home/yunheadmin/yunhetools/python-cgi/weixinartical/pre/weixinartical/monitor"
	<IfModule alias_module>
		ScriptAlias /cgi-bin/ "/home/yunheadmin/yunhetools/python-cgi/weixinartical/pre/weixinartical/monitor/"
	</IfModule>
	<Directory "/home/yunheadmin/yunhetools/python-cgi/weixinartical/pre/weixinartical/monitor/">
		AllowOverride None
		Options Indexes FollowSymLinks ExecCGI
		Require all granted
		Require host ip
	</Directory>
</VirtualHost>
<VirtualHost *:8086>
	ServerName pycgi.internal.enlightent.com
	DocumentRoot "/home/yunheadmin/yunhetools/python-cgi/crawlerLink/prod/crawlerLink/dataPycgi"
	<IfModule alias_module>
		ScriptAlias /cgi-bin/ "/home/yunheadmin/yunhetools/python-cgi/crawlerLink/prod/crawlerLink/dataPycgi/"
	</IfModule>
	<Directory "/home/yunheadmin/yunhetools/python-cgi/crawlerLink/prod/crawlerLink/dataPycgi/">
		AllowOverride None
		Options Indexes FollowSymLinks ExecCGI
		Require all granted
		Require host ip
	</Directory>
</VirtualHost>
<VirtualHost *:8087>
	ServerName pre.pycgi.internal.enlightent.com
	DocumentRoot "/home/yunheadmin/yunhetools/python-cgi/crawlerLink/pre/crawlerLink/dataPycgi"
	<IfModule alias_module>
		ScriptAlias /cgi-bin/ "/home/yunheadmin/yunhetools/python-cgi/crawlerLink/pre/crawlerLink/dataPycgi/"
	</IfModule>
	<Directory "/home/yunheadmin/yunhetools/python-cgi/crawlerLink/pre/crawlerLink/dataPycgi/">
		AllowOverride None
		Options Indexes FollowSymLinks ExecCGI
		Require all granted
		Require host ip
	</Directory>
</VirtualHost>
```
