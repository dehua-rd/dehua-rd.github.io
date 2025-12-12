# 靶机信息
这个靶机不难，不存在什么漏洞利用，完全考验对于信息的收集能力。如果找不到，就回过头再看看吧。

# 信息收集
## Nmap扫描
```
nmap -sCV -p22,80 -O --min-rate 1000 192.168.218.226 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-11 20:33 EST
Nmap scan report for 192.168.218.226
Host is up (0.00083s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 5e:b8:ff:2d:ac:c7:e9:3c:99:2f:3b:fc:da:5c:a3:53 (RSA)
|   256 a8:f3:81:9d:0a:dc:16:9a:49:ee:bc:24:e4:65:5c:a6 (ECDSA)
|_  256 4f:20:c3:2d:19:75:5b:e8:1f:32:01:75:c2:70:9a:7e (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 00:0C:29:68:05:EF (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
只有两个端口开放，很简明，不会一眼懵

那就直接看80端口了。

## web信息枚举
```
dirsearch -u http://192.168.218.226
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/RedteamStu/stageThree/writeup/deathnote/reports/http_192.168.218.226/_25-12-11_20-39-33.txt

Target: http://192.168.218.226/

[20:39:33] Starting: 
[20:39:35] 403 -  280B  - /.ht_wsr.txt                                      
[20:39:35] 403 -  280B  - /.htaccess.bak1                                   
[20:39:35] 403 -  280B  - /.htaccess.orig                                   
[20:39:35] 403 -  280B  - /.htaccess.save                                   
[20:39:35] 403 -  280B  - /.htaccess.sample
[20:39:35] 403 -  280B  - /.htaccess_orig
[20:39:35] 403 -  280B  - /.htaccess_sc                                     
[20:39:35] 403 -  280B  - /.htaccess_extra
[20:39:36] 403 -  280B  - /.htaccessBAK
[20:39:36] 403 -  280B  - /.htaccessOLD2
[20:39:36] 403 -  280B  - /.htaccessOLD
[20:39:36] 403 -  280B  - /.htm                                             
[20:39:36] 403 -  280B  - /.html
[20:39:36] 403 -  280B  - /.htpasswd_test                                   
[20:39:36] 403 -  280B  - /.htpasswds
[20:39:36] 403 -  280B  - /.httr-oauth
[20:39:37] 403 -  280B  - /.php                                             
[20:40:04] 301 -  319B  - /manual  ->  http://192.168.218.226/manual/       
[20:40:04] 200 -  201B  - /manual/index.html                                
[20:40:13] 200 -   68B  - /robots.txt                                       
[20:40:14] 403 -  280B  - /server-status                                    
[20:40:14] 403 -  280B  - /server-status/                                   
[20:40:24] 200 -    2KB - /wordpress/wp-login.php                           
[20:40:24] 200 -    6KB - /wordpress/ 
```
是一个wp站点，先看优先级最高的**robots.txt**，是提示信息

<img width="1391" height="405" alt="Image" src="https://github.com/user-attachments/assets/20471ff9-5ccc-485d-af1d-42c160f918be" />

图片加载错误，直接看内容，发现是一段话

<img width="1898" height="933" alt="Image" src="https://github.com/user-attachments/assets/94c7b670-be4c-412b-b5b7-2f470dab07b4" />

>i am Soichiro Yagami, light's father
>i have a doubt if L is true about the assumption that light is kira
>i can only help you by giving something important
>login username : user.txt
>i don't know the password.
>find it by yourself 
>but i think it is in the hint section of site
他提示了一个用户名：**user.txt**，但是密码得自己找。

那就看看wp目录，先扫一下所有的注释，没看到隐藏信息，扫一波目录】

```
dirsearch -u http://192.168.218.226/wordpress/
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/RedteamStu/stageThree/writeup/deathnote/reports/http_192.168.218.226/_wordpress__25-12-11_21-34-44.txt

Target: http://192.168.218.226/

[21:34:44] Starting: wordpress/
[21:34:47] 403 -  280B  - /wordpress/.ht_wsr.txt                            
[21:34:48] 403 -  280B  - /wordpress/.htaccess.bak1                         
[21:34:48] 403 -  280B  - /wordpress/.htaccess.orig                         
[21:34:48] 403 -  280B  - /wordpress/.htaccess.sample
[21:34:48] 403 -  280B  - /wordpress/.htaccess_orig                         
[21:34:48] 403 -  280B  - /wordpress/.htaccess.save
[21:34:48] 403 -  280B  - /wordpress/.htaccess_extra
[21:34:48] 403 -  280B  - /wordpress/.htaccess_sc
[21:34:48] 403 -  280B  - /wordpress/.htaccessBAK
[21:34:48] 403 -  280B  - /wordpress/.htaccessOLD
[21:34:48] 403 -  280B  - /wordpress/.htaccessOLD2
[21:34:48] 403 -  280B  - /wordpress/.htm                                   
[21:34:48] 403 -  280B  - /wordpress/.html
[21:34:48] 403 -  280B  - /wordpress/.htpasswds                             
[21:34:48] 403 -  280B  - /wordpress/.httr-oauth
[21:34:48] 403 -  280B  - /wordpress/.htpasswd_test                         
[21:34:49] 403 -  280B  - /wordpress/.php                                   
[21:35:24] 301 -    0B  - /wordpress/index.php  ->  http://192.168.218.226/wordpress/
[21:35:25] 404 -   30KB - /wordpress/index.php/login/                       
[21:35:28] 200 -    7KB - /wordpress/license.txt                            
[21:35:42] 200 -    3KB - /wordpress/readme.html                            
[21:35:57] 301 -  331B  - /wordpress/wp-admin  ->  http://192.168.218.226/wordpress/wp-admin/
[21:35:57] 400 -    1B  - /wordpress/wp-admin/admin-ajax.php                
[21:35:57] 302 -    0B  - /wordpress/wp-admin/  ->  http://deathnote.vuln/wordpress/wp-login.php?redirect_to=http%3A%2F%2F192.168.218.226%2Fwordpress%2Fwp-admin%2F&reauth=1
[21:35:57] 200 -  514B  - /wordpress/wp-admin/install.php                   
[21:35:57] 200 -    0B  - /wordpress/wp-config.php
[21:35:57] 409 -    3KB - /wordpress/wp-admin/setup-config.php
[21:35:57] 301 -  333B  - /wordpress/wp-content  ->  http://192.168.218.226/wordpress/wp-content/
[21:35:57] 200 -    0B  - /wordpress/wp-content/                            
[21:35:57] 200 -   84B  - /wordpress/wp-content/plugins/akismet/akismet.php 
[21:35:57] 500 -    0B  - /wordpress/wp-content/plugins/hello.php           
[21:35:57] 200 -  424B  - /wordpress/wp-content/upgrade/                    
[21:35:57] 200 -  484B  - /wordpress/wp-content/uploads/                    
[21:35:57] 301 -  334B  - /wordpress/wp-includes  ->  http://192.168.218.226/wordpress/wp-includes/
[21:35:57] 200 -    0B  - /wordpress/wp-includes/rss-functions.php
[21:35:57] 200 -    4KB - /wordpress/wp-includes/                           
[21:35:57] 200 -    0B  - /wordpress/wp-cron.php
[21:35:57] 200 -    2KB - /wordpress/wp-login.php                           
[21:35:57] 302 -    0B  - /wordpress/wp-signup.php  ->  http://deathnote.vuln/wordpress/wp-login.php?action=register
[21:35:58] 405 -   42B  - /wordpress/xmlrpc.php 
```
都翻一下，最后在**uploads**下找到了两个文本文件，文件名让我确定就是这个了。

<img width="1918" height="965" alt="Image" src="https://github.com/user-attachments/assets/94c0fd32-d004-4507-a79e-2a7ab4ebeab4" />

然后收集起来这两个文件用来爆破ssh，开始以为就是上面说的用户名，但是爆破不出来，索性就把txt里面的都爆破了。

```
hydra -L user.txt -P pass.txt ssh://192.168.218.226
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-12-11 21:39:13
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 2074 login tries (l:17/p:122), ~130 tries per task
[DATA] attacking ssh://192.168.218.226:22/
[STATUS] 255.00 tries/min, 255 tries in 00:01h, 1822 to do in 00:08h, 13 active
[STATUS] 223.00 tries/min, 669 tries in 00:03h, 1411 to do in 00:07h, 10 active
[22][ssh] host: 192.168.218.226   login: l   password: death4me
[STATUS] 200.00 tries/min, 1400 tries in 00:07h, 680 to do in 00:04h, 10 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-12-11 21:50:18
```
成功得到一组凭据 **l:death4me**

# 登录ssh
正常登录了ssh，没有sudo，就上linpeas扫一遍，试了一下困难性最高的内核提权，没有成功。
然后细细分析一下linpeas的输出，发现了数据库开放了，并且有在wp的配置文件中找到用户名密码，直接登录看看。

<img width="1898" height="757" alt="Image" src="https://github.com/user-attachments/assets/4a0d0e78-45a3-4812-b60d-41e70b8fcd6d" />

直接查询最敏感的信息表，密码被加密了，爆破了一会没有爆破出来，可能不是，再找找。

<!-- Failed to upload "image.png" -->

在kira的ssh认证文件中看到了 **l** 的公钥，那么直接使用自己的私钥登录上去。

```
kira@deathnote:~$ cat kira.txt 
cGxlYXNlIHByb3RlY3Qgb25lIG9mIHRoZSBmb2xsb3dpbmcgCjEuIEwgKC9vcHQpCjIuIE1pc2EgKC92YXIp
kira@deathnote:~$ base64 -d
^C
kira@deathnote:~$ echo cGxlYXNlIHByb3RlY3Qgb25lIG9mIHRoZSBmb2xsb3dpbmcgCjEuIEwgKC9vcHQpCjIuIE1pc2EgKC92YXIp | base64 -d
please protect one of the following 
1. L (/opt)
2. Misa (/var)
```
什么意思？看一下
```
kira@deathnote:/opt$ ls -la
total 12
drwxr-xr-x  3 root root 4096 Aug 29  2021 .
drwxr-xr-x 18 root root 4096 Jul 19  2021 ..
drwxr-xr-x  4 root root 4096 Aug 29  2021 L
kira@deathnote:/opt$ cd L/
kira@deathnote:/opt/L$ ls -la
total 16
drwxr-xr-x 4 root root 4096 Aug 29  2021 .
drwxr-xr-x 3 root root 4096 Aug 29  2021 ..
drwxr-xr-x 2 root root 4096 Aug 29  2021 fake-notebook-rule
drwxr-xr-x 2 root root 4096 Aug 29  2021 kira-case
kira@deathnote:/opt/L$ cd fake-notebook-rule/
kira@deathnote:/opt/L/fake-notebook-rule$ ls -a
.  ..  case.wav  hint
kira@deathnote:/opt/L/fake-notebook-rule$ ls -la
total 16
drwxr-xr-x 2 root root 4096 Aug 29  2021 .
drwxr-xr-x 4 root root 4096 Aug 29  2021 ..
-rw-r--r-- 1 root root   84 Aug 29  2021 case.wav
-rw-r--r-- 1 root root   15 Aug 29  2021 hint
kira@deathnote:/opt/L/fake-notebook-rule$ cat hint
use cyberchef

kira@deathnote:/opt/L/fake-notebook-rule$ cat case.wav 
63 47 46 7a 63 33 64 6b 49 44 6f 67 61 32 6c 79 59 57 6c 7a 5a 58 5a 70 62 43 41 3d
kira@deathnote:/opt/L/fake-notebook-rule$ cat 
case.wav  hint      
kira@deathnote:/opt/L/fake-notebook-rule$ cat case.wav | xxd -r -p
cGFzc3dkIDoga2lyYWlzZXZpbCA=kira@deathnote:/opt/L/fake-notebook-rule$ cat case.wav | xxd -r -p | base64 -d
passwd : kiraisevil kira@deathnote:/opt/L/fake-notebook-rule$ 
```

```
kira@deathnote:/opt/L$ ls
fake-notebook-rule  kira-case
kira@deathnote:/opt/L$ cd kira-case/
kira@deathnote:/opt/L/kira-case$ ls
case-file.txt
kira@deathnote:/opt/L/kira-case$ la -la
-bash: la: command not found
kira@deathnote:/opt/L/kira-case$ ls -la
total 12
drwxr-xr-x 2 root root 4096 Aug 29  2021 .
drwxr-xr-x 4 root root 4096 Aug 29  2021 ..
-rw-r--r-- 1 root root  295 Aug 29  2021 case-file.txt
kira@deathnote:/opt/L/kira-case$ cat case-file.txt 
the FBI agent died on December 27, 2006

1 week after the investigation of the task-force member/head.
aka.....
Soichiro Yagami's family .


hmmmmmmmmm......
and according to watari ,
he died as other died after Kira targeted them .


and we also found something in 
fake-notebook-rule folder .
```

```
kira@deathnote:/var$ cat misa
it is toooo late for misa 
```
ok，解出来一个密码，是kira的密码，直接sudo提权成功。

