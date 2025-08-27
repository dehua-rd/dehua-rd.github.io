# 靶机信息
insanity是vulnhub中的一台困难难度的渗透测试靶机

# 信息收集
## nmap扫描
```
└─$ nmap -sCV -p21,22,80 192.168.218.202 -oA nmapScan/details                   
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-26 20:47 EDT
Nmap scan report for 192.168.218.202 (192.168.218.202)
Host is up (0.00034s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: ERROR
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.218.148
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 85:46:41:06:da:83:04:01:b0:e4:1f:9b:7e:8b:31:9f (RSA)
|   256 e4:9c:b1:f2:44:f1:f0:4b:c3:80:93:a9:5d:96:98:d3 (ECDSA)
|_  256 65:cf:b4:af:ad:86:56:ef:ae:8b:bf:f2:f0:d9:be:10 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/7.2.33)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.2.33
|_http-title: Insanity - UK and European Servers
MAC Address: 00:0C:29:04:2E:81 (VMware)
Service Info: OS: Unix
```
此靶机只有tcp端口开放

# 初始信息枚举
## FTP枚举
```
ftp 192.168.218.202
Connected to 192.168.218.202.
220 (vsFTPd 3.0.2)
Name (192.168.218.202:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> binary
200 Switching to Binary mode.
ftp> ls
229 Entering Extended Passive Mode (|||19853|).
ftp: Can't connect to `192.168.218.202:19853': 没有到主机的路由
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0               6 Apr 01  2020 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> passive
Passive mode: on; fallback to active mode: on.
ftp> ls
229 Entering Extended Passive Mode (|||14927|).
ftp: Can't connect to `192.168.218.202:14927': 没有到主机的路由
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> quit
221 Goodbye.
```
可以匿名登录但是啥也没有

## gobuster扫描
```
└─$ gobuster dir -u http://192.168.218.202/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.202/
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
/js                   (Status: 301) [Size: 234] [--> http://192.168.218.202/js/]
/css                  (Status: 301) [Size: 235] [--> http://192.168.218.202/css/]
/img                  (Status: 301) [Size: 235] [--> http://192.168.218.202/img/]
/data                 (Status: 301) [Size: 236] [--> http://192.168.218.202/data/]
/news                 (Status: 301) [Size: 236] [--> http://192.168.218.202/news/]
/index.php            (Status: 200) [Size: 31]
/index.html           (Status: 200) [Size: 22263]
/fonts                (Status: 301) [Size: 237] [--> http://192.168.218.202/fonts/]
/webmail              (Status: 301) [Size: 239] [--> http://192.168.218.202/webmail/]
/phpmyadmin           (Status: 301) [Size: 242] [--> http://192.168.218.202/phpmyadmin/]
/monitoring           (Status: 301) [Size: 242] [--> http://192.168.218.202/monitoring/]
/licence              (Status: 200) [Size: 57]
/phpinfo.php          (Status: 200) [Size: 85293]
/index.html           (Status: 200) [Size: 22263]
/index.php            (Status: 200) [Size: 31]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```
> /webmail 又是这个松鼠邮箱
> /phpmyadmin 可以 anonymous 无密码登录但是啥也没有
> /monitoring 是一个登录页面

# web信息枚举
看主页，好像就是一个静态页面，找了半天也没找到可以利用的，再扫一下目录，二级目录也扫。
先看看monitoring 这个登录页面，没有啥信息，爆破一下，放后台。
在扫 /news 的时候，扫出来了很多，是一个 BluditCMS ，枚举一下。

<img width="1916" height="921" alt="Image" src="https://github.com/user-attachments/assets/4c5708f1-7c4b-4044-a002-acdf414da654" />

翻翻找找了很久，依旧没有找到入口点。

