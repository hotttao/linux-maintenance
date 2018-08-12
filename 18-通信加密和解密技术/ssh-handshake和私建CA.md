# 18.3 ssl handshake 和私建 CA
第一节我们学习了通信加密的基础知识，常见的加密解密算法，ssl/tls 协议。 。还概括性的介绍了在已知公钥和基于公钥基础设施实现安全通信的过程，ssl/tls 就是用来规范这个过程如何进行的的协议，本节我们就来详细讲解 ssl 会话建立的过程。在这之后我们会介绍如何使用 openssl 手动构建 CA，并签发证书。


## 1. SSL会话：
ssl 会话创建需要三个步骤:
1. 客户端向服务器端索要并验正证书；
2. 双方协商生成“会话密钥”；
3. 双方采用“会话密钥”进行加密通信；

SSL Handshake Protocol(ssl 握手协议) 就是用来规范客户端与服务器端如何协商生成会话密钥。如下图所示，其分成了四个阶段
![ssl_handshake](../images/18/ssl_handshake.png)

#### 第一阶段：ClientHello
客户端将向服务器端发送以下信息:
- 支持的协议版本，比如tls 1.2；
- 客户端生成一个随机数，稍后用户生成“会话密钥”
- 支持的加密算法，比如AES、3DES、RSA；
- 支持的压缩算法；

#### 第二阶段：ServerHello
服务器端将向客户端发送以下信息
- 确认使用的加密通信协议版本，比如tls 1.2；
- 服务器端生成一个随机数，稍后用于生成“会话密钥”
- 确认使用的加密方法；
- 服务器证书；

#### 第三阶段-Client：
客户端接收到服务器的证书后，验正服务器证书，在确认无误后取出其公钥；（发证机构、证书完整性、证书持有者、证书有效期、吊销列表）。发送以下信息给服务器端：
- 一个随机数；
- 编码变更通知，表示随后的信息都将用双方商定的加密方法和密钥发送；
- 客户端握手结束通知；

#### 第四阶段-Server
服务器端收到客户端发来的第三个随机数pre-master-key后，计算生成本次会话所有到的“会话密钥”；向客户端发送如下信息：
- 编码变更通知，表示随后的信息都将用双方商定的加密方法和密钥发送；
- 服务端握手结束通知；


## 2. 私建 CA 
前面我们知道 PKI 公钥基础设施包括如下几个部分:
- 签证机构：CA
- 注册机构：RA
- 证书吊销列表：CRL
- 证书存取库：

私建 CA 首先要创建出上述的基础设施，然后才能签发证书。而证书的申请及签发大体上包括以下步骤:
1. 申请方生成申请请求
2. RA 进行核验
3. CA 签署
4. 证书获取

下面我们就分创建 CA，和签发一个证书两个步骤讲解私建 CA的整个过程。

### 2.1 创建公钥基础设施
- 方法: 在确定配置为CA的服务上生成一个自签证书，并为CA提供所需要的目录及文件即可；
- 配置文件： /etc/pki/tls/openssl.cnf
- CA管理目录: /etc/pki/CA

```
# /etc/pki/tls/openssl.conf
dir            = /etc/pki/CA          # Where everything is kept
certs          = $dir/certs            # Where the issued certs are kept
crl_dir        = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several ctificates with same subject.
new_certs_dir  = $dir/newcerts        # default place for new certs.

certificate    = $dir/cacert.pem      # The CA certificate
serial          = $dir/serial          # The current serial number
crlnumber      = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl            = $dir/crl.pem          # The current CRL
private_key    = $dir/private/cakey.pem# The private key
RANDFILE        = $dir/private/.rand    # private random number file

default_days    = 365                  # how long to certify for
default_crl_days= 30                    # how long before next CRL
```


#### 构建私有CA步骤
```
# 1. 生成所需要的文件和目录
> dir=/etc/pki/CA
> cd $dir
> touch {serial,index.txt}
> echo 01 > serial
> mkdir -pv $dir{certs,crl,newcerts}

# 2. CA 自签证书
# 生成私钥
> (umask 077; openssl genrsa -out /etc/pki/CA/private/cakey.pem 4096)
> openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3655
    # new：生成新证书签署请求；
    # x509：生成自签格式证书，专用于创建私有CA时；
    # key：生成请求时用到的私钥文件；
    # out：证书的保存路径
    # days：证书的有效时长，单位是day；


# 3. 发证
# 3.1 要用到证书的主机生成证书请求：
# 3.2 把请求文件传输给 CA
# 3.3 CA 验证证书合法性，签署证书，并将证书发还给请求者

# 3.1 步骤：（以httpd为例）
# 用到证书的主机生成私钥；
> mkdir  /etc/httpd/ssl
> cd    /etc/httpd/ssl
> (umask  077; openssl  genrsa -out  /etc/httpd/ssl/httpd.key  2048)

# 生成证书签署请求
> openssl  req  -new  -key  /etc/httpd/ssl/httpd.key  -out /etc/httpd/ssl/httpd.csr  -days  365

# 3.2  将请求通过可靠方式发送给CA主机；
> scp /etc/httpd/ssl/httpd.csr root@196.168.1.105:/tmp

# 3.3 在CA主机上签署证书；
> openssl ca  -in  /tmp/httpd.csr  -out  /etc/pki/CA/certs/httpd.crt  -days  365

# 3.4 查看证书中的信息：
> openssl  x509  -in /etc/pki/CA/certs/httpd.crt  -noout  -text

# 4. 吊销证书：
# 4.1. 客户端获取要吊销的证书的serial：
> openssl  x509  -in /etc/pki/CA/certs/httpd.crt  -noout  -serial  -subject

# 4.2 CA主机吊销证书
# 先根据客户提交的serial和subject信息，对比其与本机数据库index.txt中存储的是否一致；
# 吊销, 其中的SERIAL要换成证书真正的序列号；
> openssl  ca  -revoke  /etc/pki/CA/newcerts/SERIAL.pem

# 4.3. 生成吊销证书的吊销编号（第一次吊销证书时执行）
> echo  01  > /etc/pki/CA/crlnumber

# 4.4 更新证书吊销列表
> openssl  ca  -gencrl  -out  thisca.crl

# 4.5 查看crl文件：
> openssl  crl  -in  /PATH/FROM/CRL_FILE.crl  -noout  -text
```
