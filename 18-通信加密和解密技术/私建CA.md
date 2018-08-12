# 18.4 私建 CA
很多时候我们为了测试目的，或者不便让用户去申请证书，我们就需要私建 CA，本节我们就来讲解如何私建 CA。

## 1. 私建 CA 
CA创建的工具有两个，小范围内可直接使用 `openssl` 命令，如果要维护大量的CA，可以使用完全 CA 创建工具 openCA。

那么如何创建 CA？前面我们知道 PKI 公钥基础设施包括如下几个部分:
- 签证机构：CA
- 注册机构：RA
- 证书吊销列表：CRL
- 证书存取库：

所以私建 CA 首先要创建出上述的基础设施，然后才能签发证书。而证书的申请及签发大体上包括以下步骤:
1. 申请方生成申请请求
2. RA 进行核验
3. CA 签署
4. 证书获取

下面我们就分创建 CA，和签发一个证书两个步骤讲解私建 CA的整个过程。

### 1.1 创建公钥基础设施
创建一个私有 CA非常简单，只要在确定配置为 CA 的服务上生成一个自签证书，并为CA提供所需要的目录及文件即可。CA 所需的目录定义在 CA 的配置文件中 `/etc/pki/tls/openssl.cnf`

```
# less /etc/pki/tls/openssl.conf
####################################################################
[ ca ]
default_ca	= CA_default		# The default ca section

####################################################################
[ CA_default ]
dir            = /etc/pki/CA          # CA 的工作目录
certs          = $dir/certs            # 已经签发的证书目录
crl_dir        = $dir/crl              # 已吊销证书的放置目录
database        = $dir/index.txt        # 已经签发证书的索引
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several ctificates with same subject.
new_certs_dir  = $dir/newcerts          # default place for new certs.

certificate    = $dir/cacert.pem       # CA自签证书
serial          = $dir/serial          # 当前序列号，表示新签发证书的编号
crlnumber      = $dir/crlnumber        # 新吊销证书的编号
                                        # must be commented out to leave a V1 CRL
crl            = $dir/crl.pem          # The current CRL
private_key    = $dir/private/cakey.pem # CA 私钥
RANDFILE        = $dir/private/.rand    # private random number file

default_days    = 365                  # 证书默认的有效时长
default_crl_days= 30                    # how long before next CRL
```

### 1.2 证书申请签发查看
`openssl req [options] outfile`
- 作用: 生成证书签署请求
- 选项:
    - `-new`：生成新证书签署请求；
    - `-x509`: 生成自签格式证书，专用于创建私有CA时；
    - `-key`：生成请求时用到的私钥文件,openssl 会自动提取出公钥放置在证书签署请求中；
    - `-out`：生成的请求文件路径，自签证书将直接生成签署过的证书
    - `-days`：证书的有效时长，单位是day；

`openssl ca`
- 作用: CA 签发证书
- 选项:
	- `-in <file$`: 证书签署请求文件路径
	- `-out <file$`: 生成的新证书的保存路径
	- `-days`：证书的有效时长，单位是day；

`openssl  x509`
- 作用: 查看证书信息
- 选项:
	- `-in`：指定输入文件，默认是标准输入。
	- `-out`：指定输出文件，默认是标准输出。
	- `-passin`：指定私钥密码的来源
	- `-seria`：显示序列号。
	- `-subject`：打印项目的DN
	- `-issuer`：打印签发者的DN
	- `-email`：打印email地址
	- `-startdate`：打印开始日期
	- `-enddate`：打印结束日期
	- `-purpose`：打印证书的用途
	- `-dates`：打印开始日期和结束日期
	- `-public`：输出公钥
	- `-fingerprint`：输出证书的指纹
	- `-noout`：没证书输出
	- `-days`: 设置证书的有效期时间，默认30天
	- `-req`：输入是一个证书请求，签名和输出
	- `-CA`：设置CA证书，必须是PEM格式的
	- `-text`：以文本格式输出证书

### 1.3 构建私有CA步骤

```bash
# 1. 生成所需要的文件和目录
$ dir=/etc/pki/CA
$ cd $dir
$ touch {serial,index.txt}
$ echo 01 $ serial
$ mkdir -pv $dir{certs,crl,newcerts}

# 2. CA 自签证书
# 生成私钥
$ (umask 077; openssl genrsa -out /etc/pki/CA/private/cakey.pem 4096)
$ openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3655
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
# 用到证书的主机生成私钥
$ mkdir  /etc/httpd/ssl
$ cd    /etc/httpd/ssl
$ (umask  077; openssl  genrsa -out  /etc/httpd/ssl/httpd.key  2048)

# 生成证书签署请求，(csr - certificate security request)
$ openssl  req  -new  -key  /etc/httpd/ssl/httpd.key  -out /etc/httpd/ssl/httpd.csr  -days  365

# 3.2  将请求通过可靠方式发送给CA主机；
$ scp /etc/httpd/ssl/httpd.csr root@196.168.1.105:/tmp

# 3.3 在CA主机上签署证书 (crt - certificate 的简写) 
$ openssl ca  -in  /tmp/httpd.csr  -out  /etc/pki/CA/certs/httpd.crt  -days  365

# 3.4 查看证书中的信息：
$ openssl  x509  -in /etc/pki/CA/certs/httpd.crt  -noout  -text

# 4. 吊销证书：
# 4.1. 客户端获取要吊销的证书的serial：
$ openssl  x509  -in /etc/pki/CA/certs/httpd.crt  -noout  -serial  -subject

# 4.2 CA主机吊销证书
# 先根据客户提交的serial和subject信息，对比其与本机数据库index.txt中存储的是否一致；
# 吊销, 其中的SERIAL要换成证书真正的序列号；
$ openssl  ca  -revoke  /etc/pki/CA/newcerts/SERIAL.pem

# 4.3. 生成吊销证书的吊销编号（第一次吊销证书时执行）
$ echo  01  $ /etc/pki/CA/crlnumber

# 4.4 更新证书吊销列表
$ openssl  ca  -gencrl  -out  thisca.crl

# 4.5 查看crl文件：
$ openssl  crl  -in  /PATH/FROM/CRL_FILE.crl  -noout  -text
```
