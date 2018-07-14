# 17.1 sed命令应用与实战

## 1. sed命令：
### 1.1 文本处理三剑客：
- grep, egrep, fgrep：文本过滤器
- sed：Stream EDitor，流编辑器，行
- awk：文本格式化工具，报告生成器

### 1.2 sed 使用
**sed [OPTION]...  'script'  [input-file] ...**
1. options：
    - -n：不输出模式空间中的内容至屏幕；
    - -e script, --expression=script：多点编辑；
    - -f  /PATH/TO/SED_SCRIPT_FILE 每行一个编辑命令；
    - -r, --regexp-extended：支持使用扩展正则表达式；
    - -i[SUFFIX], --in-place[=SUFFIX]：直接编辑原文件 ；
    - eg: `sed  -e  's@^#[[:space:]]*@@'  -e  '/^UUID/d'  /etc/fstab`
1. script：
    - 格式: 地址定界+命令，多个命令使用 ";" 隔开
    - 地址定界：
        - 空地址：对全文进行处理；
        - 单地址：
            - \#：指定行；
            - /pattern/：被此模式所匹配到的每一行；
        - 地址范围
            - \#,\#：从哪一行开始，哪一行结束
            - \#,+\#：从哪一行开始，往下几行
            - \#，/pat1/
            - /pat1/,/pat2/
            - $：最后一行；
        - 步进：~
            - 1~2：所有奇数行
            - 2~2：所有偶数行
            - eg： sed -n "1~2p"  /etc/fstab
    - 编辑命令：
        - d：删除；
        - p：显示模式空间中的内容；
        - a  \text：在行后面追加文本“text”，支持使用\n实现多行追加；
        - i  \text：在行前面插入文本“text”，支持使用\n实现多行插入；
        - c  \text：把匹配到的行替换为此处指定的文本“text”；
        - w /PATH/TO/SOMEFILE：保存模式空间匹配到的行至指定的文件中；
        - r  /PATH/FROM/SOMEFILE：读取指定文件的内容至当前文件被模式匹配到的行后面；文件合并；
            - eg: sed '6r /home/tao/log.csv' /etc/fstab
        - =：为模式匹配到的行打印行号；
        - !：条件取反；位于编辑命令之前，表示对地址定界之前的行
            - 地址定界!编辑命令；
            - eg: sed '/^UUID/!d'  /etc/fstab
        - s///：查找替换，其分隔符可自行指定，常用的有s@@@, s###等；
            - 替换标记：
                - g：全局替换；
                - w /PATH/TO/SOMEFILE：将替换成功的结果保存至指定文件中；
                - p：显示替换成功的行；
            - eg:
                - sed '/^UUID/s/UUID/uuid/g' /etc/fstab
                - sed 's@r..t@&er@'    /etc/passwd  -- & 表示后项引用
                - sed 's@r..t@&er@p' /etc/passwd  -- 仅仅打印被替换的行
    - 高级编辑命令：
        - h：把模式空间中的内容覆盖至保持空间中；
        - H：把模式空间中的内容追加至保持空间中；
        - g：把保持空间中的内容覆盖至模式空间中；
        - G：把保持空间中的内容追加至模式空间中；
        - x：把模式空间中的内容与保持空间中的内容互换；
        - n：覆盖读取匹配到的行的下一行至模式空间中；
        - N：追加读取匹配到的行的下一行至模式空间中；
        - d：删除模式空间中的行；
        - D：删除多行模式空间中的所有行；

### 1.3 示例：
```
# 删除/boot/grub/grub2.cfg文件中所有以空白字符开头的行的行首的所有空白字符；
> sed  's@^[[:space:]]\+@@' /etc/grub2.cfg

# 删除/etc/fstab文件中所有以#开头的行的行首的#号及#后面的所有空白字符；
> sed  's@^#[[:space:]]*@@'  /etc/fstab

# 输出一个绝对路径给sed命令，取出其目录，其行为类似于dirname；
> echo "/var/log/messages/" | sed 's@[^/]\+/\?$@@'
> echo "/var/log/messages" | sed -r 's@[^/]+/?$@@'
```

```
sed  -n  'n;p'  FILE：显示偶数行；
sed  '1!G;h;$!d'  FILE：逆序显示文件的内容；
sed  ’$!d'  FILE：取出最后一行；
sed  '$!N;$!D' FILE：取出文件后两行；
sed '/^$/d;G' FILE：删除原有的所有空白行，而后为所有的非空白行后添加一个空白行；
sed  'n;d'  FILE：显示奇数行；
sed 'G' FILE：在原有的每行后方添加一个空白行；
sed -n '1!G;h;$p': 逆序显示文件的内容；
```
