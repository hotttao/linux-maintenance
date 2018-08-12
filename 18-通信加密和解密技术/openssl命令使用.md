# 18.2 openssl 命令使用
OpenSSL 分为三个组成
1. libencrypto库:加密算法库
2. libssl库:加密模块应用库，实现了ssl及tls
3. openssl多用途命令行工具

本节我们来讲解 openssl 这个命令行工具的使用

## 1. openssl 命令
openssl 有多个子命令，分为三类：
1. 标准命令(standard)
2. 消息摘要命令（dgst子命令）
3. 加密命令（enc子命令）

```
# 生成用户密码
openssl  passwd  -1  -salt  SALT
# 随机数生成
openssl  rand  -hex  NUM
# 加密
openssl  enc  -e  -des3  -a  -salt  -in fstab  -out fstab.ciphertext
# 解密
openssl  enc  -d  -des3  -a  -salt  -out fstab  -in fstab.ciphertext
# 摘要
openssl  dgst  -md5  /PATH/TO/SOMEFILE
md5sum /path/to/somefile
# 生成私钥
(umask 077;  openssl  genrsa  -out  /PATH/TO/PRIVATE_KEY_FILE  NUM_BITS)
# 提出公钥
openssl  rsa  -in  /PATH/FROM/PRIVATE_KEY_FILE  -pubout  -out  outputfile
```

### 4.1 标准命令：
#### 生成用户密码：
- 工具：passwd, openssl  passwd
- `openssl  passwd  -1  -salt  SALT`

#### 生成随机数：
- 工具：openssl  rand
- eg: 
    - `openssl  rand  -hex  NUM`
    - `openssl  rand  -base  NUM`
- 参数:
    - NUM: 表示字节数

### 4.2 加密命令
#### 对称加密：
- 工具：openssl  enc,  gpg
- 支持算法：3des, aes, blowfish, towfish
- enc命令：
    - 加密：~]# `openssl  enc  -e  -des3  -a  -salt  -in fstab  -out fstab.ciphertext`
    - 解密：~]# `openssl  enc  -d  -des3  -a  -salt  -out fstab  -in fstab.ciphertext`

#### 单向加密：
- 工具：openssl dgst, md5sum, sha1sum, sha224sum, ...
- dgst命令：

```
openssl  dgst  -md5  /PATH/TO/SOMEFILE
md5sum /path/to/somefile
```

#### 公钥加密：
1. 加密解密：
    - 算法：RSA，ELGamal
    - 工具：openssl  rsautl, gpg
2. 数字签名：
    - 算法：RSA， DSA， ELGamal
        - DSA: Digital  Signature Algorithm
        - DSS: Digital Signature Standard
3. 密钥交换：
    - 算法：DH
4. 生成密钥：shell 中 () 内的命令会在同一个子 shell 中执行
    - 生成私钥： `(umask 077;  openssl  genrsa  -out  /PATH/TO/PRIVATE_KEY_FILE  NUM_BITS)`
    - 提出公钥： `openssl  rsa  -in  /PATH/FROM/PRIVATE_KEY_FILE  -pubout  -out  outputfile`
        - rsa 默认输出私钥，通过 -pubout 指定输出公钥

### 4.3 Linux系统上的随机数生成器：
- /dev/random：仅从熵池返回随机数；随机数用尽，阻塞；
- /dev/urandom：从熵池返回随机数；随机数用尽，会利用软件生成伪随机数，非阻塞；伪随机数不安全；
- 附注: 熵池中随机数的来源：
    - 硬盘IO中断时间间隔；
    - 键盘IO中断时间间隔；
