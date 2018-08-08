# 6.9 shell-信号捕捉实战
本节我们来学习 bash shell 编程的第九部分如何在 shell 中捕捉信号。shell 中捕捉信号主要是使用 shell 内置的 trap 命令。shell 中的信号捕捉有以下几点需要特别注意。

`15) SIGTERM` 和 `9) SIGKILL` 信号是不可捕捉的，以防止不能终止进程。

bash 中的命令会以子进程运行，因此信号可能会被子进程捕获，执行脚本的进程因此可能无法捕获到信号。但是如果 trap 在子命令之前优先执行，信号则会优先被执行脚本的进程捕获。

程序执行过程中可能会产生很多临时文件或其他数据，正常结束时，这些临时文件都会被清理；但是如果程序执行过程中被 Ctrl-C 终止可能这些临时数据将无法被清除；因此可能需要捕捉 `2) SIGINT` 信号以清除执行程序时产生的临时文件。

## 1. 信号捕捉
#### trap
`trap -l`:
- 作用: 等同于 `kill -l` 列出所有信号

`trap  'COMMAND'  SIGNALS`
- 作用: 指定在接收到信号后将要采取的动作，常见的用途是在脚本程序被中断时完成清理工作
- 常可以进行捕捉的信号：
  - `1) SIGHUP`
  - `2) SIGINT`

```bash
# 表示当shell收到HUP INT PIPE QUIT TERM这几个命令时，当前执行的程序会读取参数“exit 1”，并将它作为命令执行。
trap "exit 1" HUP INT PIPE QUIT TERM
```

## 2. 示例
```bash
#!/bin/bash
#
declare -a hosttmpfiles
trap  'mytrap'  INT

mytrap()  {
  echo "Quit"
  rm -f ${hosttmpfiles[@]}
  exit 1
}


for i in {1..50}; do
  tmpfile=$(mktemp /tmp/ping.XXXXXX)
  if ping -W 1 -c 1 172.16.$i.1 &> /dev/null; then
    echo "172.16.$i.1 is up" | tee $tmpfile
  else
    echo "172.16.$i.1 is down" | tee $tmpfile
  fi
  hosttmpfiles[${#hosttmpfiles[*]}]=$tmpfile
done

rm -f ${hosttmpfiles[@]}
```
