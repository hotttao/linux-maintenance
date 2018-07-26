# 6.3 if 和条件判断
本节我们来学习 bash shell 编程的第三部分条件判断，包括以下内容:
1. 条件测试的实现
2. 测试表达式
  - 数值测试
  - 字符串测试
  - 文件测试
  - 组合测试
3. 条件判断语句 if 和 case

## 1. 条件测试的实现
bash 中测试的实现有两种方式，一是执行命令，并利用命令状态返回值来判断；二是所谓的测试表达式。但是所谓的测试表达式本质上仍然是由特定的测试命令执行，并通过命令状态返回值来判断测试是否满足。条件表达式我的理解只不过是为某些通用的测试目的提供便利。

因此测试是否满足，需要在执行测试命令后，使用 `echo $?` 查看测试结果。后面讲解的 if 语句则会自动判断命令执行状态，做出判断无需通过 `$?`。

bash 中的条件测试有如下三种使用方式，EXPRESSION 表示测试表达式
1. `test EXPRESSION`: bash 命令
2. `[ EXPRESSION ]`:
  - `[]` 是 test 的同义词，使用方式与 test 完全一样
  - 因为是命令，所有 bash 中的元字符如`> < ()` 在表达式中使用时需要转义
  - 命令的操作数不能为空，所以表达式中如果引用变量，需要使用 "$var"，否在当 var 不存在或为空时，会因为缺少操作数报语法错误
  - EXPRESSION两端必须有空白字符，否则为语法错误
3. `[[ EXPRESSION ]]`
  - bash 内置关键字，`[[]]` 是 `[]` 的改进，大多数情况下可等同使用
  - 因为不是命令，所以 bash 的元字符在表达式中使用无需转义
  - 非命令，所以操作数为空时也能自动识别，但是在引用变量时，为防止变量包含空格，仍然建议使用 "$var"
  - EXPRESSION两必须有空白字符，否则为语法错误

需要特别注意的是 bash 是通过 空格分隔运算符和操作数的，因此无论上述哪种形式，运算符和操作数之间必需要有空格。除了字符转义外，`[]` 和 `[[]]` 在支持的测试表达式范围上也略有不同，二者的区别可以参考此篇文章 http://mywiki.wooledge.org/BashFAQ/031  。一个实现 a 和 b 两个字符比较的示例如下。

```
> test a \> b
> [ a \> b ]
> [[ a > b ]]
> echo $?
```

## 2. 测试表达式
### 2.1 数值测试
- `-eq`：是否等于； [ $num1 -eq $num2 ]
- `-ne`：是否不等于；
- `-gt`：是否大于；
- `-ge`：是否大于等于；
- `-lt`：是否小于；
- `-le`：是否小于等于；

### 2.2 字符串测试
- `==`：是否等于；
- `>`：是否大于；
- `<`：是否小于；
- `!=`：是否不等于；
- `=~`：左侧字符串是否能够被右侧的PATTERN所匹配；
- `-z "STRING"`：判断指定的字串是否为空；空则为真，不空则假；
- `-n "STRING"`：判断指定的字符串是否不空；不空则真，空则为假；
- 注意: 字符串比较的操作数，都应该使用引号括住
    - `[ -z "$name" ]`
    - `[ "$name" = "$myname" ]`

### 2.3 文件测试
1. 文件存在测试
    - `-a FILE`: 等同于 `-e FILE`
    - `-e FILE`: 存在则为真；否则则为假；
    - `-s FILE`: 存在并且为非空文件则为值，否则为假；
2. 存在及文件类型测试
    - `-f FILE:` 存在并且为 普通文件，则为真；否则为假；
    - `-d FILE:` 存在并且为 目录文件，则为真；否则为假；
    - `-L/-h FILE`: 存在并且为 符号链接文件，则为真；否则为假；
    - `-b`: 是否存在并且为 块设备文件，则为真；否则为假；
    - `-c`: 是否存在并且为 字符设备文件，则为真；否则为假；
    - `-S`: 是否存在且为 套接字文件，则为真；否则为假；
    - `-p`: 是否存在且为 命名管道文件，则为真；否则为假；
3. 文件权限测试
    - `-r FILE`:在并且对当前用户可读；
    - `-w FILE`:存在并且对当前用户可写；
    - `-x FILE`:存在并且对当前用户可执行；
    - `-g sgid`:存在并且 拥有sgid权限；
    - `-u suid`:存在并且 拥有suid权限；
    - `-k sticky`:存在并且 拥有sticky权限；
4. 丛属关系测试
    - `-t fd`：文件是否打开且与某终端有关
    - `-O FILE` 当前用户是否为文件的属主
    - `-G FILE` 当前用户是否为文件的属组
5. 更改及新旧对比测试
    - `-N FILE`: 文件自从上一次读操作后是否被修改过；
    - `file1 -nt file2`: file1的mtime新于file2则为真，否则为假；
    - `file1 -ot file2`: file1的mtime旧于file2则为真，否则为假；
    - `file1 -ef file2`: 两文件是否是指向同一设备的 inode 的硬链接

### 2.4 组合测试条件
bash 中表示逻辑运算有两种方式，一是使用命令的逻辑运算符，连接两个命令；另一个是表达式的逻辑符号，连接两个表达式。不过 `[]` 与 `[[]]`的使用方式有所不同

1. 逻辑与：
  - `[ condition1 -a condition2 ]`
  - `[[ condition1 && condition2 ]]`
  - `command1 && command2`
2. 逻辑或：
  - `[ condition1 -o condition2 ]`  
  - `[[ condition1 || condition2 ]]`  
  - `command1 || command2`
3. 逻辑非：
  - `[ ! condition ]`
  - `[[ ! condition ]]`
  - `! command`
- eg： `[ -O FILE ] && [ -r FILE ]` 或 `[ -O FILE -a -r FILE ]`


## 3. 条件判断语句
### 3.1 if 语句
#### 语法
if 语句会自动通过判断条件测试命令的执行状态来判断测试条件是否满足
```bash
if condition; then
    pass
else
    pass
fi

if condition
then
    pass
else
    pass
fi

# 将 if 写在一行，命令行中常用
if condition; then command1;command2; else command3; fi

if [[ a > b ]]; then echo "aaaa"; else echo "bbbb"; fi
```

#### 示例
通过参数传递一个用户名给脚本，此用户不存时，则添加之；

```bash
#!/bin/bash
#
if [ $# -lt 1 ]; then
    echo "At least one username."
    exit 2
fi

if grep "^$1\>" /etc/passwd &> /dev/null; then
    echo "User $1 exists."
else
    useradd $1
    echo $1 | passwd --stdin $1 &> /dev/null
    echo "Add user $1 finished."
fi
```

### 3.2 case 语句
#### 语法
```bash
case  $VARAIBLE  in  
PAT1)
    分支1
    ;;
PAT2)
    分支2
    ;;
...
*)
    默认分支
    ;;
esac
```

**case PAT 支持glob风格的通配符**：
- `*`：任意长度的任意字符；
- `?`：任意单个字符；
- `[]`：范围内任意单个字符；
- `a|b`：a或b

#### 示例


## 练习
### 练习1
将当前主机名称保存至hostName变量中；主机名如果为空，或者为localhost.localdomain，则将其设置为www.magedu.com；
```
> hostName=$(hostname)
> [ -z "$hostName" -o "$hostName" == "localhost.localdomain" -o "$hostName" == "localhost" ] && hostname www.magedu.com
```

### 练习2
通过命令行参数给定两个数字，输出其中较大的数值；
```bash
#!/bin/bash
#
if [ $# -lt 2 ]; then
    echo "Two integers."
    exit 2
fi

declare -i max=$1

if [ $1 -lt $2 ]; then
    max=$2
fi

echo "Max number: $max."
```

### 练习3
通过命令行参数给定一个用户名，判断其ID号是偶数还是奇数；

### 练习4
通过命令行参数给定两个文本文件名，如果某文件不存在，则结束脚本执行；都存在时返回每个文件的行数，并说明其中行数较多的文件；
