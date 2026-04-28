# 靶机信息
CTF7是vulnhub中的一台简单难度的渗透测试靶机

# 初始信息收集

## nmap扫描
```
nmap -sCV -sT -p$ports --min-rate 10000 192.168.218.207 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-25 04:46 EDT
Nmap scan report for 192.168.218.207 (192.168.218.207)
Host is up (0.00025s latency).

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 5.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 41:8a:0d:5d:59:60:45:c4:c4:15:f3:8a:8d:c0:99:19 (DSA)
|_  2048 66:fb:a3:b4:74:72:66:f4:92:73:8f:bf:61:ec:8b:35 (RSA)
80/tcp    open  http        Apache httpd 2.2.15 ((CentOS))
|_http-title: Mad Irish Hacking Academy
|_http-server-header: Apache/2.2.15 (CentOS)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
139/tcp   open  netbios-ssn Samba smbd 3.5.10-125.el6 (workgroup: MYGROUP)
901/tcp   open  http        Samba SWAT administration server
|_http-title: 401 Authorization Required
| http-auth: 
| HTTP/1.0 401 Authorization Required\x0D
|_  Basic realm=SWAT
8080/tcp  open  http        Apache httpd 2.2.15 ((CentOS))
| http-title: Admin :: Mad Irish Hacking Academy
|_Requested resource was /login.php
|_http-open-proxy: Proxy might be redirecting requests
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.2.15 (CentOS)
10000/tcp open  http        MiniServ 1.610 (Webmin httpd)
|_http-title: Login to Webmin
| http-robots.txt: 1 disallowed entry 
|_/
MAC Address: 00:0C:29:9D:12:A9 (VMware)

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Unix (Samba 3.5.10-125.el6)
|   Computer name: 192
|   NetBIOS computer name: 
|   Domain name: 168.218.207
|   FQDN: 192.168.218.207
|_  System time: 2025-08-10T11:28:06-04:00
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: -14d15h18m59s, deviation: 2h49m45s, median: -14d17h19m01s
```

只开放了部分 tcp 端口

## gobuster扫描
```
gobuster dir -u http://192.168.218.207/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.207/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/js                   (Status: 301) [Size: 315] [--> http://192.168.218.207/js/]
/css                  (Status: 301) [Size: 316] [--> http://192.168.218.207/css/]
/contact              (Status: 200) [Size: 5017]
/register             (Status: 200) [Size: 6591]
/img                  (Status: 301) [Size: 316] [--> http://192.168.218.207/img/]
/inc                  (Status: 301) [Size: 316] [--> http://192.168.218.207/inc/]
/newsletter           (Status: 200) [Size: 4037]
/assets               (Status: 301) [Size: 319] [--> http://192.168.218.207/assets/]
/db                   (Status: 200) [Size: 3904]
/about                (Status: 200) [Size: 4910]
/webalizer            (Status: 301) [Size: 322] [--> http://192.168.218.207/webalizer/]
/index.php            (Status: 200) [Size: 6058]
/profile              (Status: 200) [Size: 3977]
/usage                (Status: 403) [Size: 288]
/webmail              (Status: 301) [Size: 320] [--> http://192.168.218.207/webmail/]
/backups              (Status: 301) [Size: 335] [--> http://192.168.218.207/backups/?action=backups]
/signup               (Status: 200) [Size: 4783]
/header               (Status: 200) [Size: 3904]
/footer               (Status: 200) [Size: 3904]
/default              (Status: 200) [Size: 6058]
/read                 (Status: 302) [Size: 1] [--> /readings]
/recovery             (Status: 200) [Size: 4807]
/phpinfo              (Status: 200) [Size: 58677]
/trainings            (Status: 200) [Size: 4218]
/index.php            (Status: 200) [Size: 6058]
/readingroom          (Status: 200) [Size: 4037]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```
有很多，先找重要优先级的看看。看 /inc 有目录遍历。没有翻出什么有用的

还有个 8080 端口也扫一下
```
gobuster dir -u http://192.168.218.207:8080/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.207:8080/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/logout.php           (Status: 302) [Size: 0] [--> /login.php]
/login.php            (Status: 200) [Size: 2861]
/inc                  (Status: 301) [Size: 323] [--> http://192.168.218.207:8080/inc/]
/docs                 (Status: 301) [Size: 324] [--> http://192.168.218.207:8080/docs/]
/users.php            (Status: 302) [Size: 5225] [--> /login.php]
/usage                (Status: 403) [Size: 290]
/index.php            (Status: 302) [Size: 2539] [--> /login.php]
/phpmyadmin           (Status: 301) [Size: 330] [--> http://192.168.218.207:8080/phpmyadmin/]
/feedback.php         (Status: 302) [Size: 3026] [--> /login.php]
/newsletters.php      (Status: 302) [Size: 2839] [--> /login.php]
/reservations.php     (Status: 302) [Size: 3413] [--> /login.php]
/trainings.php        (Status: 302) [Size: 2897] [--> /login.php]
/readings.php         (Status: 302) [Size: 4726] [--> /login.php]
/index.php            (Status: 302) [Size: 2539] [--> /login.php]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```


# web信息枚举
有不一样的目录，大多是重定向到 login 页面
输入单引号，造成了报错

<img width="1919" height="505" alt="Image" src="https://github.com/user-attachments/assets/0bc48681-e587-46b0-9246-826e16908581" />

`admin' or 1=1 -- -`  直接进

<img width="1919" height="562" alt="Image" src="https://github.com/user-attachments/assets/e4858377-e3ee-4214-8751-46c6ce01f10c" />

我刚刚在**80**端口注册的账户。在这个功能点提交了一个信息，留言处是一段 **xss弹窗**

<img width="1919" height="960" alt="Image" src="https://github.com/user-attachments/assets/8006c93c-7061-4596-8f10-e428c79d8a88" />

然后在**8080**端口登录上去之后看到了这条信息，并触发上面的xss弹窗。

<img width="1919" height="851" alt="Image" src="https://github.com/user-attachments/assets/847e9d3e-6671-404a-804d-92871b4441e8" />

然后在 Users 界面看到很多用户名，但是没有密码泄露，也爆破一下字典，但是没成功

<img width="1919" height="827" alt="Image" src="https://github.com/user-attachments/assets/ce337903-770c-4206-a057-9acab1bb1673" />

再翻找一下别的功能，在这看到一个上传功能。

<img width="1919" height="889" alt="Image" src="https://github.com/user-attachments/assets/7f72e308-c718-40c9-8500-e7bac8753125" />

好像没什么限制，直接上php反向连接。

<img width="1919" height="810" alt="Image" src="https://github.com/user-attachments/assets/e15a5079-5832-49a7-bed2-22c7352e7331" />

打开 **nc** 点击刚刚上传的文件就可以了就可以 getshell 了。

# 提权
扒拉了一会没发现什么，搜索 password 文件的时候，发现 phpmyadmin 忘看了，扒拉**config**目录发现配置文件允许**root**无密码登录，登录进去之后发现**website**库中的**users**表中有凭据信息。但是有md5加密，都破解出来了
```
e22f07b17f98e0d9d364584ced0e3c18:my2cents
0d9ff2a4396d6939f80ffe09b1280ee1:transformersrule
2146bf95e8929874fc63d54f50f1d2e3:turtles77
3a24d81c2b9d0d9aaf2f10c6c9757d4e:LosAngelesLakers
4773408d5358875b3764db552a29ca61:Jets4Ever
4cb9c8a8048fd02294477fcb1a41191a:changeme
5d93ceb70e2bf5daa84ec3d0cd2c731a:qwer1234
9f80ec37f8313728ef3e2f218c79aa23:Shelly2012
b2a97bcecbd9336b98d59d9324dae5cf:chuck33
ed2539fe892d2c52c42a440354e8e3d5:madrid
```
爆破了一下，好像都是可以登录的，然后ssh登录一下，登录第一个 brian 用户 `sudo -l` 发现是（ALL）ALL，然后`sudo su`秒了。

# 反思
这个靶机总的来说不难，重点在于信息收集，以及初始枚举，站点没什么防护，找到入口就可以拿下，但开放的端口比较多，可访问的资源也比较多，容易迷路被卡，后面的提权部分也可能迷路，反正我是被卡了一会，像找 suid文件，找passwd文件等等，要不是在找passwd文件的时候出现phpmyadmin目录，我可能还会被卡更久。