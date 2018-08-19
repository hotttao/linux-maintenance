# 21.5 LAMP部署示例
本节我们就来部署两个经典的 php 开源项目作为演示部署 LAMP 的示例

## 1. httpd部署示例

### 1.1 wordpress 部署
```bash
# wordpress 配置
unzip wordpress-2.9.2.zip
cp -a wordpress /www/htdoc
cd /www/htdoc/wordpress
cp wp-config-sample.php wp-config.php
vim wp-config.php # 更改mysql 连接的账号，密码，数据库

# mariadb 配置
mysql
mysql> grant all on wpdb.*  to  "wpuser"@"localhost"  identified by "wppasswd"
mysql> grant all on wpdb.*  to  "wpuser"@"127.0.0.1"  identified by "wppasswd"
mysql> create datebase wpdb
mysql> flush privileges

vim /etc/my.cnf.d/server.cnf
skip-name-resolve=ON
```

### 1.2 phpMyAdmin 部署
```bash
# 系统环境
yum install  php-mbstring

# phpMyAdmin 配置
unzip phpMyAdmin-version.zip
cp -a phpMyAdmin-version /www/htdoc/
cd /www/htdoc
ln -sv phpMyAdmin-version  pma
cd pma
cp config.sample.inc.php config.inc.php
vim config.inc.php # 修改 $cfg['blowfish_secret'] = ''

# mysql 账号配置:
mysql
mysql> set password for 'root'@'localhost' = PASSWORD('mageedu')
mysql> set password for 'root'@'127.0.0.1' = PASSWORD('mageedu')
mysql> flush privileges
```
