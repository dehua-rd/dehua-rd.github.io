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

下面有一串怪异的文本，直接问ai帮助整理一下
>I think this is the way to root!
>... we have a lot of work to do, stop playing around!