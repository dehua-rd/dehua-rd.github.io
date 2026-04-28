# 靶机信息
pwnOS1.0是vulnhub中的一台简单难度的渗透测试靶机类型靶机

# 信息收集
## nmap扫描
```
└─$ nmap -sCV -p22,80,139,445,10000,137 -O --min-rate 10000 192.168.218.208 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-31 20:52 EDT
Nmap scan report for 192.168.218.208 (192.168.218.208)
Host is up (0.00029s latency).

PORT      STATE  SERVICE     VERSION
22/tcp    open   ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
| ssh-hostkey: 
|   1024 e4:46:40:bf:e6:29:ac:c6:00:e2:b2:a3:e1:50:90:3c (DSA)
|_  2048 10:cc:35:45:8e:f2:7a:a1:cc:db:a0:e8:bf:c7:73:3d (RSA)
80/tcp    open   http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.4 (Ubuntu) PHP/5.2.3-1ubuntu6
137/tcp   closed netbios-ns
139/tcp   open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
445/tcp   open   netbios-ssn Samba smbd 3.0.26a (workgroup: MSHOME)
10000/tcp open   http        MiniServ 0.01 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
MAC Address: 00:0C:29:5E:18:C9 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.22
OS details: Linux 2.6.22, Linux 2.6.22 - 2.6.23
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h30m03s, deviation: 3h32m07s, median: 3s
|_nbstat: NetBIOS name: UBUNTUVM, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.26a)
|   Computer name: ubuntuvm
|   NetBIOS computer name: 
|   Domain name: nsdlab
|   FQDN: ubuntuvm.NSDLAB
|_  System time: 2025-08-31T19:53:11-05:00
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```
再使用 **dirsearch** 扫一遍
再使用 vuln 脚本扫描，扫出了一个新的目录 **/icons** 

## gobuster扫描
```
└─$ gobuster dir -u http://192.168.218.208 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x html,php,txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.208
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/php                  (Status: 301) [Size: 334] [--> http://192.168.218.208/php/]
/index                (Status: 200) [Size: 295]
/index.php            (Status: 200) [Size: 295]
/index2.php           (Status: 200) [Size: 156]
/index2               (Status: 200) [Size: 156]
/server-status        (Status: 403) [Size: 314]
/index1.php           (Status: 200) [Size: 1104]
/index1               (Status: 200) [Size: 1104]
/index.php            (Status: 200) [Size: 295]
/index                (Status: 200) [Size: 295]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```

# web初始枚举
再 php 目录下发现了phpmyadmin 但是需要登录凭据，简单爆破一下，先看看别的目录。

<img width="1919" height="849" alt="Image" src="https://github.com/user-attachments/assets/6bd5a029-1cdc-4ee7-b1dc-9fcc72e9c657" />

类似CTF类型，index1 和 index2 页面都包含 GET 类型参数，试试 sql 注入和文件包含，都不行，页面一点变化都没有，先放弃。
又试了一会发现还是没口子，看了看前面的信息收集。发现 10000 端口好像是 web 页面，爆破有次数限制。
搜一下有没有Nday可以利用

<img width="1919" height="779" alt="Image" src="https://github.com/user-attachments/assets/56c86a46-b2e9-4509-b307-41b03d9f5d95" />

有很多，结合上面vuln的扫描

<img width="1919" height="756" alt="Image" src="https://github.com/user-attachments/assets/0651e09e-b67d-4873-805e-b8e77d1c1d36" />

锁定这个文件读取漏洞。

<img width="1919" height="547" alt="Image" src="https://github.com/user-attachments/assets/65a26509-a1df-4415-bef1-b5333445711d" />

可以，出来了。

<img width="1915" height="709" alt="Image" src="https://github.com/user-attachments/assets/127efaec-b08c-4b38-b6b3-470f967b4162" />

权限很高，shadow 文件都可以读。

解密出来一个 **vmware:h4ckm3**

# 提权
ssh 连上来之后，扒拉了一会，感觉有内核漏洞

<img width="1919" height="773" alt="Image" src="https://github.com/user-attachments/assets/535d7ead-4b89-412b-b236-ea265cef0ce4" />

有很多

<img width="1912" height="763" alt="Image" src="https://github.com/user-attachments/assets/ac998dcf-06fc-4685-b44f-7b051e3c5a5b" />

这个范围很靠近，先试试他

<img width="1919" height="778" alt="Image" src="https://github.com/user-attachments/assets/bb40c45b-5600-4889-bb97-3599082150c5" />

成功了，一步成功，爽！

# 反思
感觉靶机还是有难度的，有兔子洞，而且由于版本有点老，可能导致使用google搜不到漏洞，还好提前用vuln扫了一遍，以及爆破凭据的时候，很久很容易放弃，以为有诈，后面看wp原来主页有一个文件包含，我加的 ./ 太少了我就直接跳过了，以后还得记得多尝试两遍。发现登录还有其他手法，如OpenSSl伪随机数生成密钥漏洞碰撞ssh私钥获取初始shell，但是有点复杂，不研究了，提权方法更是多种多样啊，如不安全权限配置加shellshock漏洞提权，还有CGI执行反弹shell提权。这个靶机很有意思，值得深入一波。