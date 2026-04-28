# 靶机信息
CTF4是vulnhub中的一台简单类型的渗透测试靶机。

# 信息收集
```
# Nmap 7.95 scan initiated Fri Aug 22 04:40:12 2025 as: /usr/lib/nmap/nmap --privileged -sCV --min-rate 10000 -p22,25,80 -oA nmapScan/tcports 192.168.218.205
Nmap scan report for 192.168.218.205 (192.168.218.205)
Host is up (0.00030s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 10:4a:18:f8:97:e0:72:27:b5:a4:33:93:3d:aa:9d:ef (DSA)
|_  2048 e7:70:d3:81:00:41:b8:6e:fd:31:ae:0e:00:ea:5c:b4 (RSA)
25/tcp open  smtp    Sendmail 8.13.5/8.13.5
| smtp-commands: ctf4.sas.upenn.edu Hello [192.168.218.148], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.5 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp open  http    Apache httpd 2.2.0 ((Fedora))
|_http-server-header: Apache/2.2.0 (Fedora)
| http-robots.txt: 5 disallowed entries 
|_/mail/ /restricted/ /conf/ /sql/ /admin/
|_http-title:  Prof. Ehks 
MAC Address: 00:0C:29:62:1B:0C (VMware)
Service Info: Host: ctf4.sas.upenn.edu; OS: Unix
```
有三个tcp端口开放


# web初始枚举
```
/images               (Status: 301) [Size: 318] [--> http://192.168.218.205/images/]
/admin                (Status: 301) [Size: 317] [--> http://192.168.218.205/admin/]
/inc                  (Status: 301) [Size: 315] [--> http://192.168.218.205/inc/]
/pages                (Status: 301) [Size: 317] [--> http://192.168.218.205/pages/]
/mail                 (Status: 301) [Size: 316] [--> http://192.168.218.205/mail/]
/calendar             (Status: 301) [Size: 320] [--> http://192.168.218.205/calendar/]
/index.html           (Status: 200) [Size: 3479]
/usage                (Status: 301) [Size: 317] [--> http://192.168.218.205/usage/]
/sql                  (Status: 301) [Size: 315] [--> http://192.168.218.205/sql/]
/conf                 (Status: 500) [Size: 618]
/restricted           (Status: 401) [Size: 481]
/robots.txt           (Status: 200) [Size: 104]
```
目录扫描的结果

然后一个功能一个功能查看
`/admin` `/mail` `/restricted `这三个功能是需要登录的，看主页有非常多的人名，使用 **cewl**将其全部收集起来准备爆破。
在 **admin** 页面前端看到一个JS代码中含有输入参数过滤功能，使用 bp 重新放包绕过，发现有报错

<img width="1531" height="832" alt="Image" src="https://github.com/user-attachments/assets/66073f40-22ec-4217-98ae-0eb60d6ce7a7" />

使用 sqlmap 跑一遍，拿到一组凭据

<img width="1475" height="365" alt="Image" src="https://github.com/user-attachments/assets/84b49e91-54e8-4e2c-b764-72820e7ce12a" />

这一组凭据中的用户名密码，可以登录到上述的三个需要登录的目录

<img width="1919" height="516" alt="Image" src="https://github.com/user-attachments/assets/1ee550f7-9525-41c7-990c-1e5e721d3d60" />

这是 /restricted 登录上去之后的样式，只有两个txt文件，分别是 /mail 和 /admin 的使用说明
再登录 /admin 看看凭据中除了 admin 都可以登录上去，登录上去之后只有一个提交博客的功能，提交一个xss上去，可以执行

<img width="1919" height="394" alt="Image" src="https://github.com/user-attachments/assets/597e8701-4191-4003-98e4-2e52ee2e3afa" />

先放一边，开始我搜索到 webmail 的一个命令执行漏洞，所有优先先看一下这个，xss先放下。

尝试很多遍自动化脚本使用和手动尝试，均无果，我一直坚信是这个口子进去的，所以导致我在这卡了非常久，知道最后的偷看wp，才发现可以直接使用那个凭据ssh登录了，我晕。

可以使用 **dstevens** 这个账户登录，但是由于这个靶机很老，现在的很多工具不支持这个算法，我是使用 putty 登录上去的

# 提权
提权非常简单，因为登录上去的用户 sudo 可以 （ALL）ALL
那就一步 sudo su 就提权了


# 反思
没什么难度，就是被这个 webmail 给迷惑住了，导致被卡，想困难了，没那么难。