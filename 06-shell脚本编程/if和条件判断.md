# 6.3 if 和条件判断

## 1. bash 算术运算
常见的算术运算符包括 `+，-，*，/,  **, %`，bash中实现算术运算有如下几种方式:
1. `let var=3+4` --  不会打印输出，只能使用变量进行保存
1. `let count+=2`
2. `var=$[3+4]`
3. `var=$((3+4))`
4. `var=$(expr 3 \* 4)`: 运算符和操作数之间必须使用空格分割
5. `$RANDOM`: bash 内置的随机数生成器，表示 1-32767 的随机数
   - `echo $[$RANDOM%60]`
6. 注意：乘法符号在有些场景中需要使用转义符；

## 5. 条件测试
测试表达式:
1. test 1 -gt 3
2. [ 1 -gt 3 ] -- 命令
3. [[ 1 -gt 3 ]] -- bash 内置关键字

### 数值测试
1. -gt
2. -ge

### 字符串测试
1. ==
2. >
3. <
4. !=
5. =~：左侧字符串是否被右侧的 pattern 所匹配，只能用于 [[ ]] 中
6. -z "String": 测试字符串是否为空，空为真
7. -n "String"，非空为真
8. note: 字符串比较的操作数，都应该使用引号括住
    - [ -z "$name" ]
    - [ "$name" = "$myname" ]
    - [[ "$name" =~ ^o.* ]]


### 文件测试
存在
- -a FILE
- -e FILE: 存在则为真；否则则为假；
- -s FILE: 存在并且为非空文件则为值，否则为假；

文件类型
- -f FILE: 存在并且为普通文件，则为真；否则为假；
- -d FILE: 存在并且为目录文件，则为真；否则为假；
- -L/-h FILE: 存在并且为符号链接文件，则为真；否则为假；
- -b: 块设备
- -c: 字符设备
- -S: 套接字文件
- -p: 命名管道

权限:
- -r FILE
- -w FILE
- -x FILE
- -g sgid
- -u suid
- -k sticky

文件是否打开:
- -t fd：文件是否打开且与某终端有关
- -N FILE 文件自上次读取之后是否被修改过
- -O FILE 当前用户是否为文件的属主
- -G FILE 当前用户是否为文件的属组

双目测试
- file1 -nt file2: file1的mtime新于file2则为真，否则为假；
- file1 -ot file2：file1的mtime旧于file2则为真，否则为假；
- file1 -ef file2： 两文件是否指向同一设备的 inode


### 组合测试条件
- 与：[ condition1 -a condition2 ]  或  condition1 && condition2
- 或：[ condition1 -o condition2 ]  或  condition1 || condition2
- 非：[ -not condition ]  或  [ ! condition ]    或  ! condition

## 2. bash 编程之用户交互
read [-ers] [变量名 ...]
- 作用:从标准输入读取一行并将其分为不同的域
- [-a 数组]
- [-d 分隔符]
- [-i 缓冲区文字]
- [-n 读取字符数]
- [-N 读取字符数]
- [-p 提示符]
- [-t 超时]
- [-u 文件描述符]
- eg:
    - read -p "enter a disk filename"  -t  5  name
    - [  -z  "$name"  ]   &&  name="value"

## 3. bash 调试
- bash  -n  script.sh  -- 检查脚本语法错误
- bash  -x  script.sh  --  单步执行，显示代码执行的详细过程


## 2. if语句
### 语法
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
```

### 示例
```bash
#!/bin/bash
if [ $# -lt 1 ]; then  # 表达式不会有返回值，所以要加 []
    echo 'one least '
    exit 1
fi

if id $1> /dev/null; then  # id 命令直接有返回值，无须 []
    echo "$1 exits"
    exit 0
else
    useradd $1
    [ $? -eq 0 ] && echo "$1"|passwd --stdin $1 &> /dev/null && echo "add user $1"
fi
```
