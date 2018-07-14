# 17.2 awk使用与实战
## 2. awk 进阶详解
### 2.1 命令格式
**gawk [options] 'program' FILE ...**
- program: PATTERN{ACTION STATEMENTS}，语句之间用分号分隔
    - PATTERN：地址范围
    - ACTION STATEMENTS: 编程语法表达式
- options：
    - -F：指明输入时用到的字段分隔符；
    - -v var=value: 自定义变量；
- 特性:
    - 支持使用：变量（内置变量、自定义变量）、循环、条件、数组
    - 引用变量的值，不需要以\$开头，所有以\$开头的变量，是用于引用字段的值；

### 2.2  PATTERN 选择地址范围
1. empty：空模式，匹配每一行；
2. /regular expression/：仅处理能够被此处的模式匹配到的行；
3. relational expression:
    - 关系表达式；结果有“真”有“假”；结果为“真”才会被处理；
    - 真：结果为非0值，非空字符串；
4. line ranges：行范围，
    - startline,endline：/pat1/,/pat2/
    - 注意： 不支持直接给出数字的格式
    - eg: awk -F: '(NR>=2&&NR<=10){print \$1}' /etc/passwd
5. BEGIN/END模式
    - BEGIN{}: 仅在开始处理文件中的文本之前执行一次；
    - END{}：仅在文本处理完成之后执行一次；

### 2.3 ACTION 语法表达式
- Expressions
- Control statements：if, while等；
- Compound statements：组合语句；
- input statements
- output statements

```
练习：
# 1、显示gid小于500的组；
> gawk -F: '\$3<500{print \$1}' /etc/group

# 2、显示默认shell为nologin的用户；
> gawk -F: '\$7~/nologin\$/{print \$1}' /etc/passwd

# 3、显示eth0网卡配置文件的配置信息，只显示=号后的内容；
> gawk -F= '{print \$2}' /etc/sysconfig/network-scripts/ifcfg-eth0

# 4、显示/etc/sysctl.conf文件定义的内核参数的参数名；
> awk -F= '/^[^#]/{print \$1}' /etc/sysctl.conf

# 5、显示eth0网卡的ip地址；
> ifconfig eth0 | awk -F: '/inet addr/{print \$2}' | awk '{print \$1}'
> ifconfig eth0 | awk 'BEGIN{FS="[ :]+"}/inet addr/{print \$4}'
```

### 2.4 awk 编程语法
#### 2.4.1 输出
1. print
    - print item1, item2, ...
    - 要点：
        - 逗号分隔符；
        - 输出的各item可以字符串，也可以是数值；当前记录的字段、变量或awk的表达式；
        - 如省略item，相当于print \$0;
2. printf命令
    - 格式化输出：printf FORMAT, item1, item2, ...
        - FORMAT必须给出;
        - 不会自动换行，需要显式给出换行控制符，\n
        - FORMAT中需要分别为后面的每个item指定一个格式化符号；
    - 格式符：
        - %c: 显示字符的ASCII码；
        - %d, %i: 显示十进制整数；
        - %e, %E: 科学计数法数值显示；
        - %f：显示为浮点数；
        - %g, %G：以科学计数法或浮点形式显示数值；
        - %s：显示字符串；
        - %u：无符号整数；
        - %%: 显示%自身；
    - 修饰符：
        - \#[.\#]：第一个数字控制显示的宽度；第二个#表示小数点后的精度；
            %3.1f
        - -: 左对齐
        - +：显示数值的符号

#### 2.4.2  变量
变量名区分字符大小写；
1. 内建变量
    - FS：input field seperator，默认为空白字符；
    - OFS：output field seperator，默认为空白字符；
    - RS：input record seperator，输入时的换行符；
    - ORS：output record seperator，输出时的换行符；-
    - NF：number of field，字段数量
        - {print NF}: 打印当前行列数
