# 靶机信息
KioptrixVM3也是一个vulnhub中的简单类型的渗透测试靶机。

# 信息收集
## NMAP扫描

```
└─$ nmap -sCV -p22,80 -O --min-rate 10000 192.168.218.199 -oA nmapScan/details 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-10 19:44 EST
Nmap scan report for 192.168.218.199 (192.168.218.199)
Host is up (0.00053s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:e3:f6:dc:2e:22:5d:17:ac:46:02:39:ad:71:cb:49 (DSA)
|_  2048 9a:82:e6:96:e4:7e:d6:a6:d7:45:44:cb:19:aa:ec:dd (RSA)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
|_http-title: Ligoat Security - Got Goat? Security ...
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
MAC Address: 00:0C:29:DD:CD:12 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.26 seconds
```
只扫描出两个开放端口，没什么干扰，直接开始枚举信息。

## 初始信息枚举
先挂着扫一下目录
```
└─$ gobuster dir -u http://192.168.218.199/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.199/
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
/modules              (Status: 301) [Size: 359] [--> http://192.168.218.199/modules/]
/cache                (Status: 301) [Size: 357] [--> http://192.168.218.199/cache/]
/data                 (Status: 403) [Size: 326]
/gallery              (Status: 301) [Size: 359] [--> http://192.168.218.199/gallery/]
/style                (Status: 301) [Size: 357] [--> http://192.168.218.199/style/]
/core                 (Status: 301) [Size: 356] [--> http://192.168.218.199/core/]
/index.php            (Status: 200) [Size: 1819]
/phpmyadmin           (Status: 301) [Size: 362] [--> http://192.168.218.199/phpmyadmin/]
/update.php           (Status: 200) [Size: 18]
/server-status        (Status: 403) [Size: 335]
/index.php            (Status: 200) [Size: 1819]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```
看一下主页，扒拉一下。

<img width="1919" height="972" alt="Image" src="https://github.com/user-attachments/assets/410dbd1d-b40c-4989-93bb-2063621bcf7c" />

登录页面是一个CMS，搜一下已知漏洞。搜到一推，包括文件包含，sql注入，代码执行都可以利用。

# 获取初始权限
直接 Sqlmap 梭哈

```
Database: gallery                                                                                                                                                                          
[7 tables]
+----------------------+
| dev_accounts         |
| gallarific_comments  |
| gallarific_galleries |
| gallarific_photos    |
| gallarific_settings  |
| gallarific_stats     |
| gallarific_users     |
+----------------------+
```
一共七个表，先看看users表

```
Database: gallery
Table: gallarific_users
[1 entry]
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+
| userid | email   | photo   | website | joincode | lastname | password | username | usertype  | firstname | datejoined | issuperuser |
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+
| 1      | <blank> | <blank> | <blank> | <blank>  | User     | n0t7t1k4 | admin    | superuser | Super     | 1302628616 | 1           |
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+

```
登录试试，登不进去，再看看accounts表

```Database: gallery
Table: dev_accounts
[2 entries]
+----+---------------------------------------------+------------+
| id | password                                    | username   |
+----+---------------------------------------------+------------+
| 1  | 0d3eccfb887aabd50f243b3f155c0f85 (Mast3r)   | dreg       |
| 2  | 5badcaf789d3d1d09794d8f021f40f0e (starwars) | loneferret |
+----+---------------------------------------------+------------+
```
奇了个怪也登不上去

<img width="1914" height="961" alt="Image" src="https://github.com/user-attachments/assets/b98ea329-a6de-4279-9534-d66ae3123331" />

是ssh的用户，登录一下，两个用户都可以登录。

# 提权
试试两个用户的权限
```
dreg@Kioptrix3:~$ sudo -l
[sudo] password for dreg: 
Sorry, user dreg may not run sudo on Kioptrix3.
dreg@Kioptrix3:~$ su starwars
Unknown id: starwars
dreg@Kioptrix3:~$ su loneferret 
Password: 
loneferret@Kioptrix3:/home/dreg$ sudo -l
User loneferret may run the following commands on this host:
    (root) NOPASSWD: !/usr/bin/su
    (root) NOPASSWD: /usr/local/bin/ht
loneferret@Kioptrix3:/home/dreg$ 
```
是一个ht程序，直接搜索这个编辑器的用法，按F2打开文件/etc/passwd，修改uid，加用户等等很多提权方法，我因为不小心将uid改正1了，导致sudo用不了了，只能重装靶机了，算了已经是最后一步了，不重来了

<img width="1919" height="822" alt="Image" src="https://github.com/user-attachments/assets/5d6751b7-ea15-45bd-8b6e-2e58db578d4a" />

到这里就提权结束了。

其实一开始我是直接命令执行打进来的，但是不知道怎么提权才想着sql注入看看有什么别的路子，看来sql注入才是靶机想让你走的那条路。