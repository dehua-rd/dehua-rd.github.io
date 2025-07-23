# 靶机信息
jarbas是vulnhub中的一台简单类型的渗透测试靶机

# 初始信息探测

## nmap扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/jarbas]
└─$ nmap -sT -p- --min-rate 10000 192.168.218.191 -oA nmapScan/ports
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 08:42 EDT
Nmap scan report for 192.168.218.191
Host is up (0.00098s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy
MAC Address: 00:0C:29:58:A8:3F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 1.72 seconds

┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/jarbas]
└─$ nmap -sU -p- --min-rate 10000 192.168.218.191 -oA nmapScan/udp  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 08:43 EDT
Warning: 192.168.218.191 giving up on port because retransmission cap hit (10).
Nmap scan report for 192.168.218.191
Host is up (0.00081s latency).
Not shown: 65456 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT      STATE SERVICE
33848/udp open  unknown
MAC Address: 00:0C:29:58:A8:3F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 72.87 seconds

```
不要忘了udp的扫描

再进行详解的服务扫描和漏洞扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/jarbas]
└─$ nmap -sV -p22,80,3306,8080 -O 192.168.218.191 -oA nmapScan/detail
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 08:50 EDT
Nmap scan report for 192.168.218.191
Host is up (0.00026s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT
MAC Address: 00:0C:29:58:A8:3F (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.77 seconds


┌──(kali㉿kali)-[~]                                                                                                                                                                  08:48:02 [8/51]
└─$ nmap --script=vuln -p22,80,3306,8080 192.168.218.191 -oA RedteamStu/stageOne/writeup/vulnhub/jarbas/nmapScan/vuln                                                                               
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 08:47 EDT                                                                                                                                     
Nmap scan report for 192.168.218.191                                                                                                                                                                
Host is up (0.00026s latency).                                                                                                                                                                      
                                                                                                                                                                                                    
PORT     STATE SERVICE                                                                                                                                                                              
22/tcp   open  ssh                                                                                                                                                                                  
80/tcp   open  http                                                                                                                                                                                 
|_http-trace: TRACE is enabled
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-sql-injection: 
|   Possible sqli for queries:
|     http://192.168.218.191:80/index_arquivos/?C=N%3BO%3DD%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=S%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=M%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=N%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=S%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=M%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/njarb_data/?C=N%3BO%3DD%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/njarb_data/?C=S%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/njarb_data/?C=M%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/njarb_data/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=D%3BO%3DD%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=N%3BO%3DA%27%20OR%20sqlspider
|     http://192.168.218.191:80/index_arquivos/?C=S%3BO%3DA%27%20OR%20sqlspider
|_    http://192.168.218.191:80/index_arquivos/?C=M%3BO%3DA%27%20OR%20sqlspider
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.218.191
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.218.191:80/
|     Form id: wmtb
|     Form action: /web/submit
|     
|     Path: http://192.168.218.191:80/
|     Form id: 
|     Form action: /web/20020720170457/http://jarbas.com.br:80/user.php
|     
|     Path: http://192.168.218.191:80/
|     Form id: 
|_    Form action: /web/20020720170457/http://jarbas.com.br:80/busca/
| http-enum: 
|_  /icons/: Potentially interesting folder w/ directory listing
3306/tcp open  mysql
8080/tcp open  http-proxy
| http-enum: 
|_  /robots.txt: Robots file
MAC Address: 00:0C:29:58:A8:3F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 39.98 seconds

```

## WEB枚举
