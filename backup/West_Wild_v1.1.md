# 靶机信息

West_Wild_v1.1是vulnhub中的一台简单难度的渗透测试靶机


# 靶机初始枚举

端口扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/West_Wild]
└─$ sudo nmap -sT -p$port -sCV -O $ip -oA nmapScan/detail                              
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-29 13:51 EDT
Nmap scan report for 192.168.218.197
Host is up (0.00036s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 6f:ee:95:91:9c:62:b2:14:cd:63:0a:3e:f8:10:9e:da (DSA)
|   2048 10:45:94:fe:a7:2f:02:8a:9b:21:1a:31:c5:03:30:48 (RSA)
|   256 97:94:17:86:18:e2:8e:7a:73:8e:41:20:76:ba:51:73 (ECDSA)
|_  256 23:81:c7:76:bb:37:78:ee:3b:73:e2:55:ad:81:32:72 (ED25519)
80/tcp  open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
MAC Address: 00:0C:29:A4:0C:C8 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 1 hop
Service Info: Host: WESTWILD; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: WESTWILD, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-07-29T17:52:08
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: westwild
|   NetBIOS computer name: WESTWILD\x00
|   Domain name: \x00
|   FQDN: westwild
|_  System time: 2025-07-29T20:52:08+03:00
|_clock-skew: mean: -59m29s, deviation: 1h43m55s, median: 30s

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.08 seconds
```
# SMB枚举

匿名连接SMB
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/West_Wild]
└─$ smbclient  //192.168.218.197/wave
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul 30 01:18:56 2019
  ..                                  D        0  Thu Aug  1 19:02:20 2019
  FLAG1.txt                           N       93  Mon Jul 29 22:31:05 2019
  message_from_aveng.txt              N      115  Tue Jul 30 01:21:48 2019

                1781464 blocks of size 1024. 285212 blocks available
smb: \> get FLAG1.txt 
getting file \FLAG1.txt of size 93 as FLAG1.txt (18.2 KiloBytes/sec) (average 18.2 KiloBytes/sec)
smb: \> get message_from_aveng.txt 
getting file \message_from_aveng.txt of size 115 as message_from_aveng.txt (28.1 KiloBytes/sec) (average 22.6 KiloBytes/sec)
smb: \> exit
```
解码flag
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/West_Wild]
└─$ cat FLAG1.txt | base64 -d
Flag1{Welcome_T0_THE-W3ST-W1LD-B0rder}
user:wavex
password:door+open
```
获得一组凭据和flag1  **wavex:door+open**
直接就ssh登录进去了
```
wavex@WestWild:~/wave$ cat message_from_aveng.txt 
Dear Wave ,
Am Sorry but i was lost my password ,
and i believe that you can reset  it for me . 
Thank You 
Aveng 
```
有个消息说aveng忘记密码了。让我们帮助重置

用linpeas扫一下
```
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                                                                                    
/home/wavex                                                                                                                                                                                         
/run/lock
/run/shm
/run/user/1001
/tmp
/tmp/tmux-1001
/usr/share/av/westsidesecret
/usr/share/av/westsidesecret/ififoregt.sh
/var/crash
/var/lib/php5
/var/spool/samba
/var/tmp
```
扫完在这一栏有个有趣的文件 **ififoregt.sh** 还是 **.sh** 
```
wavex@WestWild:~$ cat /usr/share/av/westsidesecret/ififoregt.sh
 #!/bin/bash 
 figlet "if i foregt so this my way"
 echo "user:aveng"
 echo "password:kaizen+80"
```
里面有aveng的密码

```
wavex@WestWild:~/wave$ su aveng
Password: 
aveng@WestWild:/home/wavex/wave$ cd 
aveng@WestWild:~$ id
uid=1000(aveng) gid=1000(aveng) groups=1000(aveng),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(sambashare),114(lpadmin)
aveng@WestWild:~$ sudo -l
[sudo] password for aveng: 
Matching Defaults entries for aveng on WestWild:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User aveng may run the following commands on WestWild:
    (ALL : ALL) ALL
aveng@WestWild:~$ bash
aveng@WestWild:~$ sudo bash
root@WestWild:~# 
```
横向和纵向提权一步到位了

# 反思
属于是入门级别的靶机了，除了在横向移动的时候，可能会被卡一会。