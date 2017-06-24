### 目录

* [参考资料](#参考资料)
* [网络安全](#网络安全)
  * [网络通信安全问题](#网络通信安全问题)
  * [加密](#加密)
  * [Openssl](#openssl)  
  * [SSH](#ssh)
* [DNS解析及DNS服务器的搭建](#DNS解析及DNS服务器的搭建)
  * [什么是DNS](#什么是DNS)
  * [DNS服务器](#DNS服务器)
  * [DNS主机角色](#DNS主机角色)
  * [资源记录类型RRT](#资源记录类型RRT)
  * [还有一些概念](#还有一些概念)
  * [DNS服务器搭建](#DNS服务器搭建)
* [ISO的OSI七层模型](#ISO的OSI七层模型)
* [数据在网络中传输时封装](#数据在网络中传输时封装)

### 参考资料 {#参考资料}

> 参考视频

SPOTO ccna培训视频

### 网络安全 {#网络安全}

#### 网络通信安全问题 {#网络通信安全问题}

Alice与Bob通过互联网中的两台主机建立通信.这两台从未建立过连接的主机之间通信,存在哪些问题?

* 机密性:网络协议只支持明文传输数据
* 完整性:数据可能在传输途中被修改
* 身份认证:银行等系统需要明确双方身份

#### 加密 {#加密}

为确保网络中所传输数据的机密性,加密成了必要的解决方式.同时加密后的数据还需要无差别的解密回原来的文本.

其中加密通过加密算法实现.加密算法可以在网络中公开,但是密钥必须保密.因为算法比较难实现,是复杂的数学原理抽象出来的一类问题的解决方法,无疑保证算法的绝对私有是不尽可能的.

> 对称加密

这种加密方式的数据安全依赖密钥,加密算法完全公开.即每一次通信使用的算法相同,密钥不尽相同.这在小范围通信中是可行的,但是这种加密方式依然存在一些问题.比如:

主机A与B通信,默认使用一种叫md5的加密算法.每次通信时A用string对明文加密,B用string对接收密文进行解密.

一天之中A事实上要与上百甚至上千的主机通信,如果每次都用同一个密钥加密与解密,这种加密似乎变得没什么意义,密钥也变得有些人尽皆知了.

这种情况下,要保证每次同一种算法的数据安全,A不得不针对不同的主机使用不同的密钥.显然这又让事情变得复杂了,互联网中主机浩如烟海,保存和管理这么多的密钥似乎也有些不可理喻了. 

这种方式能解决数据机密性问题,但无法保证密钥的有效管理.

> 单向加密

提取数据特征码,从而保证数据完整性.特点

* 输入相同,输出必然相同.
* 雪崩效应: 输入的微小变化, 引起结果的巨大差异
* 定长输出: 无论原始数据多长,结果的长度都是相同的
* 不可逆: 无法根据特征码,还原回原来的数据

主机A与B通信,不对数据进行加密,但是会将这段数据提取出一段特征码,附加在明文之后在网络中传输.B在接收到数据后,通过算法计算明文的特征码,然后比较两端的特征码.如果特征码相同,则可以保证数据完整性.

这种加密方式存在问题这样一些问题

* 中间人攻击(man-in-middle):  中间人拿到明文后,修改明文,自己用相同算法计算特征码,之后发送出去.
* 特征码在传输过程中由于电气性问题导致传输到目标主机后不正确
* 明文被修改了,最终接收方的数据也不正确

因此单向加密只能解决数据完整性问题,却无法保证数据的机密性,同时还存在中间人攻击的问题.而要解决中间人攻击这种问题,这个过程中的特征码也需要加密后传输.

但是两台主机如何约定密钥呢?如果要网络中传输,密钥也就不再机密了.但是双方还是要交换密钥,这需要网络协议支持,最早的协议叫Diff-hellman算法.这种交换有一个名称中Internet Key Exchange,即IKE.

> Diff-hellman算法

A与B通信,A与B在网络中传输两个大素数g,p

A自己随机生成一个数:x  
B自己随机生成一个数:y

A在本机计算: (g^x)%p,并传输给B  
B在本机计算: (g^y)%p,并会输给A

A最终得到: x, (g^y)%p  
B最终得到: y, (g^x)%p

A在本地计算得到密钥: (g^xy)%p  
B在本地计算得到密钥: (g^yx)%p

从而保证了密钥交换的安全,同时也让用户从繁杂的密码管理中解脱出来.从此密钥不需要两台主机事先约定好,但是这也产生另一个问题,无法获知通信双方的身份.这个问题依赖非对称加密算法.

> 非对称加密算法

这种算法会产生一对密钥,分别是公钥(Public Key)与私钥(Secret Key).而公钥是通过某种方式从私钥中提取出来的.通过公钥加密有明文,必须用相应的私钥才能解密.

公钥人尽皆知,私钥只有生成公钥私钥对的家伙才知道.

于是用A主机私钥加密的明文,只能用与之对应的公钥才能解密,如果接收方能解密, 说明这个数据的发送方一定是A.

另一方面,A用接收方B的公钥加密明文,B接收后可以用自己的私钥解密.其它人中间截取了也无法解密,因为不知道B的私钥.所以这种密钥对也可实现数据机密性.但是它们很少用来进行数据加密,因为密钥对太长,加密与解密都非常慢.

通过分析可见,这种密钥对通常在网络中进行身份认证.

同时也发现,鱼与熊掌不可兼得.用对方公钥加密,可以保证数据机密性;用息私钥加密,可以保证数据发送方的身份.但是不能同时保证数据机密性,同时还能保证数据发送方的身份.这需要另一种机制保全鱼与熊掌.

> 身份验证与完整性双保险

A与C在互联网中通信,中间有个B总是时不时的截取A和C的通信数据,乱改一通后伪装成发送方.为防止B这样的Bitch,于是A与B决定这么做.

A与B各自生成自己的密钥对.

A首先将原数据提取出一段特征码,从而保证数据完整性.然后将这段特征码用自己的私钥加密后发送给C.

此时,C通过A的公钥解密,如果能解密,说明发送方的确是A,于是开始处理数据.

若中间有B还截取了数据,由于特征码是用A的私钥加密的,于是B可以解密,之后要将窜改后的数据伪装成A发送给C时,只能用自己的私钥加密这段明文和特征码.于是C在接收时,用A的公钥无法解密,于是丢弃掉.

> 第三方证书颁发机构

所有主机将自己的公钥发送给第三方机构,由这个机构对公钥做公证,以告诉接收方,这个公钥是合法且有效的.发证机构的权威性在于自己也有公钥私钥对,机构用自己的私钥将数据特征码加密,加密后的数据即证书.

PKI: public key infrastructure.

#### Openssl

> SSL是什么

Netscape为了能够让http协议能够加密数据后在网络中传输.在应用层与传输层之间加了一个库SSL---Secure Socket Layer.这让应用层所有的明文协议都可以通过调用SSL的库后将数据加密后通过tcp传输.

TLS与SSL的不同在于,TLS是国际标准化组织维护,是一个完全开放的Transport Layer Sercurity.

> SSL会话

HTTP协议是基于TCP的,因此需要三次握手,之后要走SSL.

Step1: Client发起请求到Server.
Step2: Server与Client



#### DNS解析及DNS服务器的搭建 {#DNS解析及DNS服务器的搭建}

##### 什么是DNS {#什么是DNS}

> 域名

如:www.baidu.com,这是一个主机名, 它实际上有一个明确的定义叫Full Qualified Domain Name,即完全限定域名.这个主机名对应的真正域名是.com和.baidu.com.由些可知,其实域名,是指一定范围内的主机名的集合.

Domain Name Service\(DNS\)名称解析服务, 实际上是在进行名称转换,背后有一个数据查询的过程,以找到两种目标名称的相应转换.而DNS是将FQDN转换成IP地址的一个服务.如:

```
119.75.213.61 ---> www.baidu.com
211.98.71.195 ---> www.bilibili.com
```

之所以存在主机名和IP地之间名称转换的问题,可以这么理解. 人的思考方式总是与计算机不同的,对于计算机而言,所有的语言最终都不过是数字,因此计算机之间通信用以区分彼此的是IP地址.但人类也需要通过自己能理解的方式去区分主机,用IP地址显然不得记忆,因些人们通过主机名来识别互联网中的每一台主机.

> nsswitch框架: /etc/nsswitch.conf是这个框架的具体体现

给名称解析提供一个平台,而实际完成名称解析操作的是具体的一个个的店铺.这个框架依靠linux系统调用库工具帮助具体实现名称解析的程序完成名称解析.这两个库是如下

* libnss\_files.so
* libnss\_dns.so

> /etc/nsswitch.conf中有一行内容为hosts: files   dns

这里的files是libnss\_files去/etc/hosts文件,通过这个文件完成主机名称到ip地址的映射关系;dns则是指dns服务.

所以当我们去请求一个主机名时,系统会调用一个库函数实现主机名到ip的转换,这个过程由stub resolvers完成.这个过程可以通过网际协议ping来查看.

```zsh
➜  ~ ping http://www.bilibili.com/
PING http://www.bilibili.com/ (211.98.71.195): 56 data bytes
64 bytes from 211.98.71.195: icmp_seq=0 ttl=250 time=3.399 ms
64 bytes from 211.98.71.195: icmp_seq=1 ttl=250 time=4.362 ms
64 bytes from 211.98.71.195: icmp_seq=2 ttl=250 time=6.390 ms
64 bytes from 211.98.71.195: icmp_seq=3 ttl=250 time=6.101 ms
64 bytes from 211.98.71.195: icmp_seq=4 ttl=250 time=4.439 ms
```

> 早期**主机名---&gt;IP**的方法,通过hosts文件,这时候的主机较少

```txt
IPADDR            FQDN                Aliases
211.98.71.195     www.bilibili.com    www
```

> IANA和ICANN

IANA负责维护主机名与IP地址的映射关系.IANA即互联网名称分配机构.IANA通过ftp服务器保存一个hosts文件.所有的主机网络中访问其它主机时,会先去IANA的ftp服务器上下载最新的hosts文件,从而拿到要请求主机对应的IP地址.

* 周期性任务更新hosts文件
* Server: 由发送的请求目标主机在服务器端更新,之后将目标主机名对应的IP发送给客户端
  * 当时的条件,请求达到1KW条时,导致服务器不堪重负,查询也慢.
* 分布式数据库
  * 按Level管理域名
  * 自顶向下为: 根,顶级,二级...
  * 上级仅知道其直接下级
  * 下级仅知道根的位置
  * 按Level管理域名
  * TLD\(Top Level Domain\)
	  * 组织域: .com, .org, .net, .cc
  	  * 国家域: .cn, .jp, .hk, .tw, .ca, .iq
  	  * 反向域: IP--->FQDN

但随着互联网中主机数量增大,后来民间成立了一个ICANN替代了IANA的作用.

> 名称解析分类

* 正向解析: FQDN---&gt;IP
* 反向解析: IP---&gt;FQDN
* 一个IP可以对应多个FQDN 
* 一个FQDN也可以对应多个IP, 这是DNS的高级功能,可能实现负载均衡,提高服务器的负载能力

> 查询

* 递归: 只发出一次请求
* 迭代: 发出多次请求
* 所有DNS请求都是两段式: 前半段递归,后半段迭代
* 查询结果会缓存下来

> DNS服务器

* 接受本地客户端的查询请求\(递归\)
* 外部客户端请求: 请求权威答案的
  * 肯定答案: TTL
  * 否定答案: TTL
* 外部客户端请求非权威答案: 非权威答案,不给递归查询
* 根或.com的域不会给任何人递归,安全起见

> /etc/resolv.conf

这个文件中的nameserver IP必须是允许递归的服务器地址,而stub resolver又是一个要求必须有应答的递归请求.因些,一旦写成不允许递归的IP, stub resolver将得不到应答.

##### DNS服务器 {#DNS服务器}

不同的域名层级实际就是由一台一台的域名服务器组织起来的.

> 根服务器

全球一共有A-M排列的13组根服务器,共504台.其中中国北京有四台根镜像.域名格式为a.root-servers.net.

> DNS服务器主从结构

如果一台DNS服务器,意味着这个台DNS服务器管理的所有主机都将不可达.这是十分危险的,从而有必要提供安全机制,以防止DNS服务器挂掉后造成不可想象的后果.

* MASTER DNS服务器: 数据修改
* SLAVE DNS服务器: 请求数据同步,只有有需要时才会pull数据
  * serials number: 版本号
  * refresh: 多长时间检查一次MASTER的版本号
  * retry: 重试时间
  * expire: 认为MASTER挂掉的时间
  * negative answer TTL: 否定答案缓存时长

> 缓存DNS服务器

不提供任何权威答案,只负责缓存

> 转发DNS服务器

不提供权威答案,也不缓存,只将请求转发出去

> **NOTE:**
>
> DNS服务器是由上级在数据库授权的

##### DNS主机角色 {#DNS主机角色}

所有域中的主机都有自己的角色,比如有的是邮件服务器,有的是www的服务器等等.

在数据库中的每一个条目叫作一个资源记录,每条记录中的RRT\(资源记录类型\)字段用以区分这条记录最终要转换成什么.格式:

```csh
$TTL 600;
NAME  TTL(可省略,但必须提供全局变量)    IN    RRT    VALUE
www.baidu.com                       IN    A      1.1.1.1
1.1.1.1                             IN    PTR    www.baidu.com
```

##### 资源记录类型RRT {#资源记录类型RRT}

用来表示这条记录要解析的值在DNS内部是什么角色.

* A
* AAAA
* NS
* MX
* WWW
* PTR
* @: 用来表示域名zone name

> A记录

```zsh
FQDN           ---> IPv4
www.baidu.com. ---> 119.75.213.61
```

> AAAA记录

```zsh
FQDN           ---> IPv6
www.baidu.com. ---> 2001:503:ba3e::2:30
```

> NS记录

在一个zone内,允许有多台NS服务器,而这些服务器的主从关系是通过配置文件加以区分.

```zsh
Zone Name      ---> FQDN

baidu.com.     ---> ns1.baidu.com.(NS记录)
ns1.baidu.com. ---> 119.75.213.61(A记录)

ZoneName      TTL    IN    RRT   VALUE     
baidu.com.    600    IN    NS    ns1.baidu.com.
ns1.baidu.com.600    IN    A     1.1.1.1
```

> SOA:起始授权记录

由于一个zone内,允许有多台NS服务器,**主从之间如何同步数据,以及起始授权对象是谁**,这个申明通过SOA记录负责完成.这条记录必须出现在数据文件的第一条.

邮箱格式正常为 nora@126.com, 但是在资源记录中要写成nora.126.com.因为在资源记录中这个@是一个通配符,代表zone name. 

```zsh
# MASTER FQDN 是主DNS服务器主机全限定主名称
# 主从之间请求的几个参数可以用的单位有: H, M, D, W, 默认是S
# serial number最长不能超过10位,一般使用时间戳,数据文件每修改一次,serialnumber也要修改一次
# 这个资源数据文件中注释用`;`开头
$TTL 600;
ZONE NAME    IN		RRT		MASTER FQDN    ADMINISTRATOR_MAILBOX (
										 serial number
										 refresh
										 retry
										 expire
										 negative answer)
baidu.com    IN		SOA		ns1.baidu.com. admin.baidu.com. (
										 201706241641 ; 版本号
										 1H
										 1M
										 1W
										 1D )
```

> MX\(Mail Exchanger\)

最终指向某个域内邮件服务器的主机名,一般会有多台,由优先级字段决定使用中的是哪台.优先级从0-99,数字越小优先级越高.  同时也要像NS一样,要给出这台邮件服务器对应的A记录,以告诉请求者这台服务器的IP地址.

```zsh
ZONE NAME      ---> FQDN

ZONE NAME         TTL     IN    RRT    PRI    VALUE
baidu.com.        600     IN    MX     10     mail.baidu.com.
mail.baidu.com.   600     IN    A             1.1.1.1
```

> PTR记录: 反向解析,交换IP转换成FQDN

```zsh
IP             ---> FQDN
1.1.1.1    600    IN    PTR    www.baidu.com
```

> CNAME: Canonical Name 正式名称

```ZSH
FQDN			  ---> FQDN
zone name			TTL		IN		RRT		VALUE
www2.baidu.com.	600  	IN		CNAME	www.baidu.com.
```

##### 还有一些概念 {#还有一些概念}
* 域: Domain是一个逻辑概念.
* 区域: Zone是一个物理概念. 
 
> **NOTE:**  
> Zone是一个物理概念.一个域名在DNS解析的过程中依赖解析数据文件,正向解析依赖正向解析文件,反向解析依赖反向解析文件,正向解析文件即正向区域,反向解析文件即反向区域.这两个区域构成的空间即为域.但是域与区域之间没有必然的包含关系.因为域由上级授权,授权通过区域完成,因此域来自于区域文件.同时,正向域与反向域的授权是分开进行的.正向区域的zone name与域名相同,但是反向区域的zone name却是与域名有很大的区别.

```ZSH
# 注册了一个域名 xpercent.com
# 又从ICANN获得了一个网段 10.211.55.0/24
# xpercent.com域中 域名解析服务器主机为 10.211.55.2
# xpercent.com域中 还有一台MX邮件服务器, 主机IP为 10.211.55.3
# 从上级获取授权, 写一个授权记录
# 授权记录
Zone Name			IN		RRT		VALUE
xpercent.com.		IN		NS		ns1.xpercent.com.
ns1.xpercent.com.IN		A		10.211.55.2

# 正向区域文件
Zone Name			 IN		RRT		FQDN	ADMIN_MAILBOX 
xpercent.com. 	 IN		SOA		ns1.xpercent.com. mail.xpercent.com. (
								201706241641 ; 版本号
								1H
								5M
								1W
								1D )
xpercent.com. 	 IN 	MX		10.211.55.3
www.xpercent.com. IN	A		10.211.55.2

注意,soa的zone name写成@,@知道自己处于哪个域内,这是通过配置文件实现的.
而此后的所有记录zone name都可以将域名去掉后只写前半段.**不可加上`.`**

# 反向区域文件
# ZONE NAME是网段地址的逆序且加上.in-addr.arpa后缀
Zone Name					  	 IN		RRT		VALUE
55.211.55.in-addr.arpa. 	 IN		SOA		
2.55.211.10.in-addr.arpa.   IN 	PTR 	www.xpercent.com.
# 上面的PTR记录zone name可简写为2
```

* 区域传送的类型
	* 完全区域传送: axfr
	* 增量区域传送: ixfr
* 主从区域类型
	* 主区域: 主DNS服务器 MASTER
	* 从区域: 从DNS服务器 SLAVE
	* 提示区域: 自己找不到的,根据hint找到根的位置 HINT
	* 转发区域: 添加一条可递归的DNS服务器 FORWAR

##### DNS服务器搭建 {#DNS服务器搭建}

> **规划**  

* 注册域名: xpercent.me 
* 网段号: 10.211.55.0/24  
* 搭一台缓冲DNS服务器: 10.211.55.1 负责解析注册域的所有FQDN工作
* www.xpercent.me 10.211.55.1, 10.211.55.3    
* mail.xpercent.me 10.211.55.2 
* ftp.xpercent.me www.xpercent.me  

> BIND安装

```ZSH
shell> rpm -qa | grep "^bind"
shell> rpm install -y bind-lib, bind-utils, bind
shell> rpm -qlv bind # 查看bind最后的安装目录及配置文件等
# 文件列表如下
drwxr-x---    2 root  named  0 May  9 21:43 /etc/named
-rw-r-----    1 root  named  984 Nov 20 2015 /etc/named.conf
-rw-r--r--    1 root  named  2389 May  9 21:43 /etc/named.iscdlv.key
-rw-r-----    1 root  named  931 Jun 21  2007 /etc/named.rfc1912.zones
-rw-r--r--    1 root  named  487 Jul 19  2010 /etc/named.root.key
# 守护进程脚本
-rwxr-xr-x    1 root  root   7992 May  9 21:43 /etc/rc.d/init.d/named
# 守护进程脚本的配置文件
-rw-r--r--    1 root    root 1695 May  9 21:43 /etc/sysconfig/named
# rndc文件
-rw-r-----    1 root    named  0 May  9 21:43 /etc/rndc.conf
-rw-r-----    1 root    named  0 May  9 21:43 /etc/rndc.key
# 二进制程序
-rwxr-xr-x    2 root    root   550744 May  9 21:43 /usr/sbin/named
-rwxr-xr-x    1 root    root   28464 May  9 21:43 /usr/sbin/named-checkconf
-rwxr-xr-x    1 root    root   24416 May  9 21:43 /usr/sbin/named-
checkzone
# 区域数据文件
-rw-r-----    1 root  named  3171 Jan 11  2016 /var/named/named.ca
-rw-r-----    1 root  named  152 Dec 15  2009 /var/named/named.empty
-rw-r-----    1 root  named  152 Jun 21  2007 /var/named/named.localhost
-rw-r-----    1 root  named  168 Dec 15  2009 /var/named/named.loopback
```


> BIND相关文件

* /etc/named 
* /etc/named.conf BIND进程的工作属性,区域的定义等主配置文件,相当于已经安装caching-server
* /etc/rndc.key
	* rndc: 远程域名控制进程
	* rndc.key: 是远程管理使用的密钥
	* 配置文件其实是: /etc/rndc.conf
* /var/named: 区域数据文件存储位置
* /etc/rc.d/named: start|stop|restart|reload|status|configtest
* /var/log/message: dns运行时日志文件
* named: 二进制程序,位于/usr/sbin/named
* named-checkconfig: 检查区域配置文件的语法
* named-checkzone: 检查区域配置文件的语法 
* named.ca: 13组根结点
* named.localhost: 将localhost解析成127.0.0.1及::1
* named.loopback: 本地主机名的正反向解析

> DNS监听的端口

* UDP 53, 默认的查询请求协议
* TCP 53, 从服务器同步数据时用TCP
* TCP 953, RNDC监听的端口

> 数据文件注意事项

格式: 全文所有括号前后有空格,每一个完整语句必须以`;`结束.

```ZSH
ZONE "ZONE NAME" IN {
	type master|slave|hint|forward
	file "相对option中的directory而言"
};
```

* options: 全局选项,对每一个zone有效
	* directory: 定义数据目录,是个相对路径,文件中其它目录都相对这个路径而言
	* recursion: 是否给所接收的请求进行递归查询 
* ZONE: 
	* 根区域: type hint;
	* 主区域: type master;
	* 从区域: type slave;
* include: 

> 搭建流程

![](/assets/DNS服务器搭建过程.png)

* 用到的一些命令及基说明

```ZSH
shell> mv /etc/named.conf /etc/named.conf.backup
shell> cp /etc/named.conf.backup /etc/named.conf -p
shell> named-checkconfig
shell> named-checkzone "." /var/named/named.ca
# named-checkzone "zone name" filename
shell> named-checkzone "localhost" /var/named/named.localhost
shell> named-checkzone "loopback" /var/named/named.loopback
shell> getenforce # 确认selinux是否开启,开启的话要关闭
shell> setenforce 0 # 临时性的关闭selinux
shell> servcie named start
shell> tail /var/log/message
shell> netstat -npl#p父进程id/l列表/n以ip:port形式显示进程/t:tcp/u:udp
shell> ping www.baidu.com # 能ping能表示服务器搭建成功
shell> dig -t "RT" "ZONE NAME"
shell> dig -t NS	xpercent.me
shell> dig -t A www.xpercent.me.
shell> dig -t MX mail.xpercent.me.
shell> dig -x IP

# dig: domain information gropher,在域名服务器系统中查看相关信息
shell> dig -t NS . # 查找要域的所有DNS服务器,前提是运行dig进程的主机必须要能访问互联网
shell> dig -t NS . @A.ROOT-SERVERS.NET. # 直接通过A.ROOT-SERVERS.NET.这个域查找根域的信息
```

* DNS服务器配置文件 /etc/named.conf

```js
options {
	directory "/var/named";
};
zone "." IN {
	type hint;
	file "named.ca";
};
zone "localhost" IN {
	type master;
	file "named.localhost";
};
zone "loopback" IN {
	type master;
	file "named.loopback";
};
zone "xpercent.me" IN {
	type master;
	file "xpercent.me.zone";
};
zone "55.211.10.in-addr.arpa" IN {
	type master;
	file "10.211.55.zone";
};
```

* 正向区域文件 /var/named/xpercent.me.zone

```ZSH
$TTL 86400;
@			IN		SOA	ns1.xpercent.me.		admin.xpercent.me. (
					201706241800
					1H
					5M
					1W
					1D )
			IN		NS      	ns1
			IN		MX  10		mail
ns1			IN		A			10.211.55.1
www			IN		A			10.211.55.1
mail		IN		A			10.211.55.2
www			IN		A			10.211.55.3
ftp			IN		CNAME		www
```

* 反向区域文件 /var/named/10.211.55.1.zone

```ZSH
$TTL 86400;
@	IN	SOA	ns1.xpercent.me.	admin.xpercent.me. (
					201706241800
					1H
					5M
					1W
					1D )
	IN	NS	ns1.xpercent.me. ; 这里的FQDN必须写到`.`,
1	IN	PTR	www.xpercent.me.
2	IN	PTR	mail.xpercent.me.
3	IN	PTR	www.xpercent.me.

```

* /etc/resolv.conf

```ZSH
# 这里的nameserver必须指向一台可以联网的主机
nameserver 10.211.55.200
```

* 测试一下

```ZSH
# 正向测试
[root@os1 named]# dig -t A www.xpercent.me
; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.62.rc1.el6_9.2 <<>> -t A www.xpercent.me
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55993
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;www.xpercent.me.		IN	A

;; ANSWER SECTION:
www.xpercent.me.	86400	IN	A	10.211.55.1
www.xpercent.me.	86400	IN	A	10.211.55.3

;; AUTHORITY SECTION:
xpercent.me.		86400	IN	NS	www.xpercent.me.

;; Query time: 0 msec
;; SERVER: 10.211.55.200#53(10.211.55.200)
;; WHEN: Sat Jun 24 07:52:18 2017
;; MSG SIZE  rcvd: 79

# 反向测试
[root@os1 named]# dig -x 10.211.55.1
; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.62.rc1.el6_9.2 <<>> -x 10.211.55.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50309
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; QUESTION SECTION:
;1.55.211.10.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
1.55.211.10.in-addr.arpa. 86400	IN	PTR	www.xpercent.me.

;; AUTHORITY SECTION:
55.211.10.in-addr.arpa.	86400	IN	NS	ns1.xpercent.me.

;; ADDITIONAL SECTION:
ns1.xpercent.me.	86400	IN	A	10.211.55.1

;; Query time: 0 msec
;; SERVER: 10.211.55.200#53(10.211.55.200)
;; WHEN: Sat Jun 24 08:48:14 2017
;; MSG SIZE  rcvd: 105
```

### DNS主从复制 

> 泛域名解析

> 递归查询 

```ZSH
[root@os1 named]# dig -t A www.baidu.com
; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.62.rc1.el6_9.2 <<>> -t A www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8910
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 5, ADDITIONAL: 5

;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; ANSWER SECTION:
www.baidu.com.		1200	IN	CNAME	www.a.shifen.com.
www.a.shifen.com.	300	IN	A	111.13.100.91
www.a.shifen.com.	300	IN	A	111.13.100.92

;; AUTHORITY SECTION:
a.shifen.com.		1145	IN	NS	ns1.a.shifen.com.
a.shifen.com.		1145	IN	NS	ns2.a.shifen.com.
a.shifen.com.		1145	IN	NS	ns3.a.shifen.com.
a.shifen.com.		1145	IN	NS	ns5.a.shifen.com.
a.shifen.com.		1145	IN	NS	ns4.a.shifen.com.

;; ADDITIONAL SECTION:
ns1.a.shifen.com.	1145	IN	A	61.135.165.224
ns2.a.shifen.com.	1145	IN	A	180.149.133.241
ns3.a.shifen.com.	1145	IN	A	61.135.162.215
ns4.a.shifen.com.	545	IN	A	115.239.210.176
ns5.a.shifen.com.	1145	IN	A	119.75.222.17

;; Query time: 45 msec
;; SERVER: 10.211.55.200#53(10.211.55.200)
;; WHEN: Sat Jun 24 09:32:16 2017
;; MSG SIZE  rcvd: 260

```

> 迭代查询

**Step1**

```ZSH
[root@os1 named]# dig +norecurse -t www.baidu.com
;; Warning, ignoring invalid type www.baidu.com

; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.62.rc1.el6_9.2 <<>> +norecurse -t www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2486
;; flags: qr ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 13

;; QUESTION SECTION:
;.				IN	NS

;; ANSWER SECTION:
.			517857	IN	NS	e.root-servers.net.
.			517857	IN	NS	m.root-servers.net.
.			517857	IN	NS	k.root-servers.net.
.			517857	IN	NS	i.root-servers.net.
.			517857	IN	NS	j.root-servers.net.
.			517857	IN	NS	f.root-servers.net.
.			517857	IN	NS	l.root-servers.net.
.			517857	IN	NS	c.root-servers.net.
.			517857	IN	NS	h.root-servers.net.
.			517857	IN	NS	a.root-servers.net.
.			517857	IN	NS	g.root-servers.net.
.			517857	IN	NS	b.root-servers.net.
.			517857	IN	NS	d.root-servers.net.

;; ADDITIONAL SECTION:
a.root-servers.net.	517857	IN	A	198.41.0.4
a.root-servers.net.	517857	IN	AAAA	2001:503:ba3e::2:30
b.root-servers.net.	517857	IN	A	192.228.79.201
b.root-servers.net.	517857	IN	AAAA	2001:500:200::b
c.root-servers.net.	517857	IN	A	192.33.4.12
c.root-servers.net.	517857	IN	AAAA	2001:500:2::c
d.root-servers.net.	517857	IN	A	199.7.91.13
d.root-servers.net.	517857	IN	AAAA	2001:500:2d::d
e.root-servers.net.	517857	IN	A	192.203.230.10
e.root-servers.net.	517857	IN	AAAA	2001:500:a8::e
f.root-servers.net.	517857	IN	A	192.5.5.241
f.root-servers.net.	517857	IN	AAAA	2001:500:2f::f
g.root-servers.net.	517857	IN	A	192.112.36.4

;; Query time: 0 msec
;; SERVER: 10.211.55.200#53(10.211.55.200)
;; WHEN: Sat Jun 24 09:34:19 2017
;; MSG SIZE  rcvd: 508
```
**Step2**

```ZSH
[root@os1 named]# dig +norecurse -t A www.baidu.com @a.gtld-servers.net.

; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.62.rc1.el6_9.2 <<>> +norecurse -t A www.baidu.com @a.gtld-servers.net.
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24622
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 5, ADDITIONAL: 5

;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; AUTHORITY SECTION:
baidu.com.		172800	IN	NS	dns.baidu.com.
baidu.com.		172800	IN	NS	ns2.baidu.com.
baidu.com.		172800	IN	NS	ns3.baidu.com.
baidu.com.		172800	IN	NS	ns4.baidu.com.
baidu.com.		172800	IN	NS	ns7.baidu.com.

;; ADDITIONAL SECTION:
dns.baidu.com.		172800	IN	A	202.108.22.220
ns2.baidu.com.		172800	IN	A	61.135.165.235
ns3.baidu.com.		172800	IN	A	220.181.37.10
ns4.baidu.com.		172800	IN	A	220.181.38.10
ns7.baidu.com.		172800	IN	A	119.75.219.82

;; Query time: 113 msec
;; SERVER: 192.5.6.30#53(192.5.6.30)
;; WHEN: Sat Jun 24 09:35:41 2017
;; MSG SIZE  rcvd: 201
```
**Step3**

```ZSH
[root@os1 named]# dig +norecurse -t A www.baidu.com @dns.baidu.com
; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.62.rc1.el6_9.2 <<>> +norecurse -t A www.baidu.com @dns.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17196
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 5, ADDITIONAL: 5

;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; ANSWER SECTION:
www.baidu.com.		1200	IN	CNAME	www.a.shifen.com.

;; AUTHORITY SECTION:
a.shifen.com.		1200	IN	NS	ns5.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns3.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns2.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns1.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns4.a.shifen.com.

;; ADDITIONAL SECTION:
ns1.a.shifen.com.	1200	IN	A	61.135.165.224
ns2.a.shifen.com.	1200	IN	A	180.149.133.241
ns3.a.shifen.com.	1200	IN	A	61.135.162.215
ns4.a.shifen.com.	1200	IN	A	115.239.210.176
ns5.a.shifen.com.	1200	IN	A	119.75.222.17

;; Query time: 7 msec
;; SERVER: 202.108.22.220#53(202.108.22.220)
;; WHEN: Sat Jun 24 09:36:45 2017
;; MSG SIZE  rcvd: 228
```
**总结**
一条命令跟踪整个解析过程.

```ZSH
[root@os1 named]# dig +trace -t A www.baidu.com
; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.62.rc1.el6_9.2 <<>> +trace -t A www.baidu.com
;; global options: +cmd
.			517565	IN	NS	k.root-servers.net.
.			517565	IN	NS	d.root-servers.net.
.			517565	IN	NS	c.root-servers.net.
.			517565	IN	NS	l.root-servers.net.
.			517565	IN	NS	i.root-servers.net.
.			517565	IN	NS	a.root-servers.net.
.			517565	IN	NS	g.root-servers.net.
.			517565	IN	NS	f.root-servers.net.
.			517565	IN	NS	m.root-servers.net.
.			517565	IN	NS	e.root-servers.net.
.			517565	IN	NS	j.root-servers.net.
.			517565	IN	NS	b.root-servers.net.
.			517565	IN	NS	h.root-servers.net.
;; Received 508 bytes from 10.211.55.200#53(10.211.55.200) in 5 ms

com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
;; Received 491 bytes from 192.33.4.12#53(192.33.4.12) in 3170 ms

baidu.com.		172800	IN	NS	dns.baidu.com.
baidu.com.		172800	IN	NS	ns2.baidu.com.
baidu.com.		172800	IN	NS	ns3.baidu.com.
baidu.com.		172800	IN	NS	ns4.baidu.com.
baidu.com.		172800	IN	NS	ns7.baidu.com.
;; Received 201 bytes from 192.48.79.30#53(192.48.79.30) in 116 ms

www.baidu.com.		1200	IN	CNAME	www.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns2.a.shifen.com.	
a.shifen.com.		1200	IN	NS	ns1.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns5.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns4.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns3.a.shifen.com.
;; Received 228 bytes from 220.181.38.10#53(220.181.38.10) in 17 ms
```

> 数据传送

数据传送发生在主从DNS服务器结构的,所以前提是的至少有两台DNS服务器.然后申明主从身份.注意,数据传送可以用来分析服务器系统的结构, 这是不安全的做法.应该将允许传送的主机做一些限制.通过allow-transfer中列出的名单.
* 全量传送: axfr
* 增量传送: ixfr

### 基于OpenSS的HTTPS服务配置

Http协议是明文的.为了在网络中安全的传送数据, 使用https.
http是一个协议,而https是一个C/S模式的服务进程,默认端口是443.


### ISO的OSI七层模型 {#ISO的OSI七层模型}

![](/assets/OSI七层模型.png)

### 数据在网络中传输时封装 {#数据在网络中传输时封装}

![](/assets/数据封装.png)

