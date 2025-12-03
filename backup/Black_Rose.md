# 靶机信息


# 信息收集
## NMAP扫描
```
└─$ nmap -sCV -p22,80,3306 -O --min-rate 1000 192.168.218.223 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-02 01:21 EST
Nmap scan report for 192.168.218.223 (192.168.218.223)
Host is up (0.0010s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d1:9b:10:88:15:e9:7a:c4:1a:29:07:3c:21:87:c4:ac (RSA)
|   256 e3:50:0b:c9:e8:f1:68:7f:e7:cf:ec:de:7b:b9:20:a1 (ECDSA)
|_  256 55:0e:96:22:cc:50:20:d9:dd:c2:ff:5b:25:d0:d7:2b (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-title: BlackRose
|_Requested resource was login.php
|_http-server-header: Apache/2.4.29 (Ubuntu)
3306/tcp open  mysql   MySQL (unauthorized)
MAC Address: 00:0C:29:C2:40:13 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.81 seconds
```
三个端口，普通的端口，正常打点。

## 初始信息枚举
先看优先级最高的3306端口
登不上去，也没别的信息，那就看80端口。
扫一下目录先
```
/js                   (Status: 301) [Size: 315] [--> http://192.168.218.223/js/]
/css                  (Status: 301) [Size: 316] [--> http://192.168.218.223/css/]
/logout.php           (Status: 302) [Size: 0] [--> 404.php]
/images               (Status: 301) [Size: 319] [--> http://192.168.218.223/images/]
/img                  (Status: 301) [Size: 316] [--> http://192.168.218.223/img/]
/login.php            (Status: 200) [Size: 1463]
/register.php         (Status: 200) [Size: 1559]
/database.php         (Status: 302) [Size: 0] [--> 404.php]
/404.php              (Status: 200) [Size: 21]
/index.php            (Status: 302) [Size: 0] [--> login.php]
/header.php           (Status: 200) [Size: 21]
/footer.php           (Status: 200) [Size: 21]
/vendors              (Status: 301) [Size: 320] [--> http://192.168.218.223/vendors/]
/server-status        (Status: 403) [Size: 280]
/index.php            (Status: 302) [Size: 0] [--> login.php]
```
基本只要登录，注册功能有用
随便注册一个进去，有个whami功能，并且在主页提交功能会显示你不是admin用户，我就猜肯定是让我以admin用户登录，但是想了半天，一直感觉是sql注入，还是打靶少了，找红笔要了点提示，得知是strcmp函数校验的password字段，那就简单了，黑盒情况居然完全没有想到这个。PHP 弱类型 + strcmp 误用。
函数判断成功的条件，肯定是让函数结果返回0，即两者相同，或者让strcmp函数报错，strcmp比较的是字符串类型，如果强行传入其他类型参数，会出错，出错后返回值0，正是利用这点进行绕过。
先传入NULL值，或者在抓包直接将password字段删除，但是应该被实体化了，失败，放弃。然后就是传入数组，将password字段写成password[]，本身password接受的是字符串，但是传入的是一个数组数据，让strcmp函数前后数据类型不一致产生报错，从而绕过。

<img width="1916" height="959" alt="Image" src="https://github.com/user-attachments/assets/e29d8a94-2097-41e5-a5a9-eeb0cfebdb4f" />

成功绕过了。再看看

<img width="1919" height="932" alt="Image" src="https://github.com/user-attachments/assets/3a276bc2-b506-45cf-bfd0-256060105ed7" />

错误的签名，是一个 bcrypt 哈希，使用hashcat爆破一下，是whoami的密文

<img width="1916" height="910" alt="Image" src="https://github.com/user-attachments/assets/b2f49a5a-4241-404f-9c58-12725bae70fc" />

那么初步判断是将command中的命令加密作为签名。

<img width="1919" height="968" alt="Image" src="https://github.com/user-attachments/assets/dbaefdb9-19c8-4b5b-8f61-9b418c383b11" />

确实是这样