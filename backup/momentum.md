# 靶机信息
此靶机考验对密码的熟悉度，入口不难，只有这么多信息，所以基本可以锁定入口。提权不难，就是看你细不细心，跟之前的提权方式，一般是文件或者程序的提权，这次是内部端口服务，可能会遗漏掉。

# 信息收集
## nmap扫描
```
# Nmap 7.95 scan initiated Thu Jan 15 01:51:01 2026 as: /usr/lib/nmap/nmap --privileged -sT -p- --min-rate 10000 -oA nmapScan/tcPort 192.168.218.238
Nmap scan report for 192.168.218.238
Host is up (0.00076s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:E3:48:9E (VMware)
```
根据以往靶机习惯，靶机入口肯定是在80端口，我就不看了，直接看80端口去

# web信息枚举
扫描目录，只扫到apache的默认页面。
<img width="1753" height="965" alt="Image" src="https://github.com/user-attachments/assets/7cee1f87-2de0-48a3-9224-c9480065bfac" />
网站的js文件有信息，有个参数 **opus-details.php?id=** 试了一下可以利用xss

<img width="1778" height="542" alt="Image" src="https://github.com/user-attachments/assets/53be59b5-4eb0-40c2-b7be-51f18487dff3" />

以及下面的注释信息，跟AES加解密有关，结合xss，感觉跟 **cookie** 有关，并且只有在 id 参数后面加 value 之后才有cookie出现，并且问AI，这个cookie就是AES的密文，尝试解密一下，这个cookie是openssl的加密格式。

```echo "U2FsdGVkX193yTOKOucUbHeDp1Wxd5r7YkoM8daRtj0rjABqGuQ6Mx28N1VbBSZt" | openssl aes-256-cbc -d -a -k SecretPassphraseMomentum -md md5
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
auxerre-alienum##  
```

# 初始shell获取
这还卡了一会，一直登不上去，凭据 **auxerre:auxerre-alienum##  **
懒得找了直接linpeas。基本没可以直接提权的信息，最后重新看标红信息的时候

发现有个6379端口，redis服务。使用redis-cli看一下

```
127.0.0.1:6379> SELECT 0
OK
127.0.0.1:6379> KEYS *
1) "rootpass"
127.0.0.1:6379> GET rootpass
"m0mentum-al1enum##"
127.0.0.1:6379> 
```
额，直接给了。

那简简单单提权了。