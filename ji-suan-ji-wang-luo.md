### 参考资料

> 参考视频

SPOTO ccna培训视频

#### DNS解析及DNS服务器的搭建

##### 什么是DNS

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

* * 按Level管理域名
  * TLD\(Top Level Domain\)

    * 组织域: .com, .org, .net, .cc
    * 国家域: .cn, .jp, .hk, .tw, .ca, .iq

    * 反向域: IP---&gt;FQDN

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

##### DNS服务器

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
  * negative answer TTL: 否定答案有效时长

> 缓存DNS服务器

不提供任何权威答案,只负责缓存

> 转发DNS服务器

不提供权威答案,也不缓存,只将请求转发出去

> **NOTE:**
>
> DNS服务器是由上级在数据库授权的

##### DNS主机角色

所有域中的主机都有自己的角色,比如有的是邮件服务器,有的是www的服务器等等.

在数据库中的每一个条目叫作一个资源记录,用以区分不同的主机角色.格式:

```csh
$TTL 600;
NAME  TTL(可省略,但必须提供全局变量)    IN    RRT    VALUE
www.baidu.com                       IN    A      1.1.1.1
1.1.1.1                             IN    PTR    www.baidu.com     
```

> 资源记录类型RRT

用来表示这条记录要解析的值在DNS内部是什么角色.

* A
* AAAA
* NS
* MX
* WWW
* PTR

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

```zsh
Zone Name      ---> FQDN

baidu.com.     ---> ns1.baidu.com.(NS记录)
ns1.baidu.com. ---> 119.75.213.61(A记录)

ZoneName    TTL    IN    RRT    
baidu.com.    600    IN    NS    ns1.baidu.com.
```

> MX\(Mail Exchanger\)记录

```zsh
ZONE NAME      ---> FQDN

ZONE NAME     TTL     IN    RRT    PRI    VALUE
baidu.com.     600    IN    MX     10     mail.baidu.com.
mail.baidu.com.600    IN    A             1.1.1.1
```



### ISO 的 OSI 七层模型

![](/assets/OSI七层模型.png)

### 数据在网络中传输时封装

## ![](/assets/数据封装.png)



