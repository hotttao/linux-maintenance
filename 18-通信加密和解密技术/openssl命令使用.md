# 18.3 openssl 命令使用
OpenSSL 分为三个组成
1. libencrypto库:加密算法库
2. libssl库:加密模块应用库，实现了ssl及tls
3. openssl多用途命令行工具

openssl 命令行工具有多个子命令，大体上分为如下三类
1. 标准命令(standard)
2. 消息摘要命令（dgst子命令）
3. 加密命令（enc子命令）

本节我们就来讲解常见命令的使用

## 1. openssl 使用概述

```bash
openssl ?
openssl:Error: '?' is an invalid command.

Standard commands   # 可使用的标准子命令                                                
asn1parse         ca                ciphers           cms               
crl               crl2pkcs7         dgst              dh                
dhparam           dsa               dsaparam          ec                
ecparam           enc               engine            errstr            
gendh             gendsa            genpkey           genrsa            
nseq              ocsp              passwd            pkcs12            
pkcs7             pkcs8             pkey              pkeyparam         
pkeyutl           prime             rand              req               
rsa               rsautl            s_client          s_server          
s_time            sess_id           smime             speed             
spkac             ts                verify            version           
x509              

# 消息摘要子命令 dgst, 下面是可用的算法
Message Digest commands (see the `dgst' command for more details)
md2               md4               md5               rmd160            
sha               sha1              

# 对称加密子命令 enc，下面是可用的算法
Cipher commands (see the `enc' command for more details)
aes-128-cbc       aes-128-ecb       aes-192-cbc       aes-192-ecb       
aes-256-cbc       aes-256-ecb       base64            bf                
bf-cbc            bf-cfb            bf-ecb            bf-ofb            
camellia-128-cbc  camellia-128-ecb  camellia-192-cbc  camellia-192-ecb  
camellia-256-cbc  camellia-256-ecb  cast              cast-cbc          
cast5-cbc         cast5-cfb         cast5-ecb         cast5-ofb         
des               des-cbc           des-cfb           des-ecb           
des-ede           des-ede-cbc       des-ede-cfb       des-ede-ofb       
des-ede3          des-ede3-cbc      des-ede3-cfb      des-ede3-ofb      
des-ofb           des3              desx              idea              
idea-cbc          idea-cfb          idea-ecb          idea-ofb          
rc2               rc2-40-cbc        rc2-64-cbc        rc2-cbc           
rc2-cfb           rc2-ecb           rc2-ofb           rc4               
rc4-40            rc5               rc5-cbc           rc5-cfb           
rc5-ecb           rc5-ofb           seed              seed-cbc          
seed-cfb          seed-ecb          seed-ofb          zlib
```


## 2. 标准命令
### 1.1 生成用户密码
`openssl password [OPTIONS] [password]`
- 作用: 生成用户密码
- 参数: password 用户密码，可省略，默认会提示用户输入
- 选项:
	- `-crypt      `: standard Unix password algorithm (default)
	- `-1          `: MD5-based password algorithm
	- `-salt string`: use provided salt
	- `-in file    `: read passwords from file
	- `-stdin      `: read passwords from stdin
	- `-reverse    `: switch table columns

```
$ openssl passwd -1 -salt tabc
Password:                         # 根据提示输入用户密码
$1$tabc$dMfThcS/0AhHbG277/5.Y.
```

### 1.2 生成随机数
`openssl rand OPTIONS num`
- 参数: NUM 表示字节数
- 选项: 
	- `-out file`: write to file
	- `-base64  `: base64 encode output
	- `-hex     `: hex encode output

```
$ openssl rand -base64 10
CyrtOhCwGXySRQ==

$ openssl rand -hex 10
61fc2a72e000622746f4

$ openssl passwd -1 -salt `openssl rand -hex 4`
Password: 
$1$e3a21fb9$Zahip67zta7xJB2QiaVAm0
```

## 2. 加密命令
### 2.1 对称加密
`openssl enc -ciphernam OPTIONS`
- 作用: 使用对称加密算法加密文件
- 选项: 
	- `-e`: 加密
	- `-d`: 解密
	- `-a/-base64`: 使用 base64 编码和解码文件
	- `-ciphernam`: 指定使用的加密算法
	- `-in <file>`: 待加密的明文文件
	- `-out <file>`: 加密后的密文输出路径
	- `-pass <arg>`: 加密使用的密码
	- `-md`: 指定密钥生成的摘要算法，用户输入的口令不能直接作为文件加密的密钥，而是经过摘要算法做转换，此参数指定摘要算法，默认md5
	- `-S`: 在把用户密码转换成加密密钥的时候需要使用盐值，默认盐值随机生成
	- `-salt`: use a salt in the key derivation routines. This is the default


```
# -e 加密
openssl  enc  -e  -des3  -a  -salt  -in fstab  -out fstab.ciphertext

# -d 解密
openssl  enc  -d  -des3  -a  -salt  -out fstab  -in fstab.ciphertext
```

### 2.2 单向加密
`openssl dgst OPTIONS file`
- 作用: 使用单向加密算法，提取摘要信息
- 参数: file 指定提取摘要的文件
- 提取算法:
	- `-md4`
	- `-md5`
	- `-ripemd160`
	- `-sha`
	- `-sha1`
	- `-sha224`
	- `-sha256`
	- `-sha384`
	- `-sha512`
	- `-whirlpool`
- 选项:
	- `-c             `: to output the digest with separating colons
	- `-r             `: to output the digest in coreutils format
	- `-d             `: to output debug info
	- `-hex           `: output as hex dump
	- `-binary        `: output in binary form


```
$ openssl  dgst  -md5  /PATH/TO/SOMEFILE
$ md5sum /path/to/somefile
```

### 2.3 公钥加密
`openssl rsautl`
- 作用: 使用RSA密钥进行加密、解密、签名和验证等运算
- 算法：
	- 加解密: RSA，ELGamal
	- 数字签名：RSA， DSA， ELGamal
    - 密钥交换：DH


### 3. 生成密钥
`openssl genrsa  OPTIONS numbits`
- 作用: 生成私钥
- 参数: numbits 私钥的长度，只能是 1024 的证书倍
- 参数:
	- `-out <file>`: 输出的文件路径
	- `-passout arg`: 指定密钥文件的加密口令，可从文件、环境变量、终端等输入

`openssl rsa [options] <infile >outfile`
- 作用: 管理生成的密钥，rsa 默认输出私钥，通过 -pubout 指定输出公钥
- 选项: 
	- `-in arg     `:输入文件
	- `-out arg    `:输出文件
	- `-passin arg `:指定输入文件的加密口令，可来自文件、终端、环境变量等
	- `-passout arg`:指定输出文件的加密口令，可来自文件、终端、环境变量等
	- `-pubin      `:指定输入文件是公钥
	- `-pubout     `:指定输出文件是公钥
	- `-text       `:以明文形式输出各个参数值
	- `-check      `:检查输入密钥的正确性和一致性

```
# 生成私钥
# shell 中 () 内的命令会在同一个子 shell 中执行
$ (umask 077;  openssl  genrsa  -out  /PATH/TO/PRIVATE_KEY_FILE  NUM_BITS)

# 提出公钥
$ openssl  rsa  -in  /PATH/FROM/PRIVATE_KEY_FILE  -pubout  -out  outputfile
```

## 4. Linux系统上的随机数生成器
Linux 中有如下几个随机数生成器
- `/dev/random`：仅从熵池返回随机数；随机数用尽，阻塞；
- `/dev/urandom`：从熵池返回随机数；随机数用尽，会利用软件生成伪随机数，非阻塞；伪随机数不安全；
- 附注: 熵池中随机数的来源：
    - 硬盘IO中断时间间隔；
    - 键盘IO中断时间间隔；
