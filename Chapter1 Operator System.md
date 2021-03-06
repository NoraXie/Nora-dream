### 参考资料

> 参考书

操作系统概念  
深入理解计算机系统  
操作系统: 精髓与设计

> 参考视频

清华大学学堂在线公开课: &lt;&lt;操作系统&gt;&gt;  
清华大学学堂在线操作系统实验课: &lt;&lt;操作系统uCore实验&gt;&gt;

### uCore操作系统实验

#### 实验0

> 环境准备

- 可以用实验楼的环境,但速度慢得unbelievable,果断自己装个机了跑 
- Linux图形化环境一台

我用的[Ubuntu 1610](http://mirrors.xmu.edu.cn/ubuntu/releases/), 这是厦门大学的镜像文件服务器,包括各种Linux发行版都有.由于不喜欢直接在mac本上装软件,一般会用Parallel Desktop虚拟机装多个系统,目前装了2台centos,1台ubuntu,1台win7.

- Qemu硬件模拟器(之所以要用图形化Linux,就是因为qemu)
- 清华大学OS课程实验代码库[ucore lab](https://github.com/chyyuu/os_tutorial_lab)
- 清华大学OS课程[WIKI](http://os.cs.tsinghua.edu.cn/oscourse/OS2017spring)

> 知识准备

- Linux基本命令
- Linux包管理工具(用于安装各种应用程序)
- 8086/80386的硬件结构(可以看看清华大学的<<汇编语言>>,国内将底层结构与原理讲得最好的一本书)
- vim编辑器
- 汇编指令
- C语言
- 数据结构
- git版本管理软件
- markdown
- makefile
- gdb调试命令
- qemu
- meld
- diffutil

当然不会也没关系,[实验指导书](https://chyyuu.gitbooks.io/ucore_os_docs/content/)里都有列出常用的工具和命令,用的时候去查也可以.

#### 实验一

> 目的

写一个简单的落后的功能单一的dirt simple boot loader.其sole job就是要从磁盘(assume假设是这样了滴)的第一个扇区开始读取elf格式的kernel.

- 1 生成磁盘文件ucore.img
- 2 qemu装载ucore.img硬盘
- 3 开机启动qemu,装载kernel运行第一个操作系统.

> 如何工作

qemu是一台物理主机.

**1. 给qemu装硬盘**  
生成一个ucore.img镜像文件,模拟物理硬盘.

**2. 生成主引导扇区**  
boot/sign.c代码负责生成512Bytes的主引导扇区bootblock,里面有boot loader代码. bootblock被嵌入ucore.img文件内,从而模拟磁盘的0号扇区内的MBR.file一下bootblock可以看到这个文件类型与相关信息

```ZSH
shell> hbase@hbase:~/ucore_os_lab/labcodes/lab1$ file bin/bootblock
bin/bootblock: DOS/MBR boot sector; partition 1 : ID=0xc5, active 0xc6, start-CHS (0x341,65,7), end-CHS (0x4,12,4), startsector 1835008, 4980736 sectors; partition 2 : ID=0xff, start-CHS (0xff,0,61), end-CHS (0x10,255,51), startsector 1090519040, 42272782 sectors
```

由上面可以知道bootblock是一个MBR的启动扇区,此外,它还记录了ucore.img有两个分区.

**3. 载入内核kernel**  
kernel是elf格式的可执行文件,它被嵌入到ucore.img文件中,存放在紧挨着(onward)主引导扇区的几个扇区内.

> make V=

这个命令可以看到命令执行的过程,从而分析ucore.img文件的生成过程.简单截取关键步骤如图所示.

![](/assets/生成磁盘文件.png)

Step1: 编译各个目录下的.c和.s文件,生成.目标文件  

```ZSH
+ cc kern/init/init.c
+ cc kern/libs/readline.c
+ cc kern/libs/stdio.c
+ cc kern/debug/kdebug.c
+ cc kern/debug/kmonitor.c
+ cc kern/debug/panic.c
+ cc kern/driver/clock.c
+ cc kern/driver/console.c
+ cc kern/driver/intr.c
+ cc kern/driver/picirq.c
+ cc kern/trap/trap.c                     ^
+ cc kern/trap/trapentry.S
+ cc kern/trap/vectors.S
+ cc kern/mm/pmm.c
+ cc libs/printfmt.c
+ cc libs/string.c

```

Step2: ld bin/kern,将bin/kern目录下的所有.o目标文件链接起来,生成可执行文件;ld bin/bootblock,与bin/kern目录一样,生成可执行文件.  

```	zsh
+ ld bin/kernel
+ cc boot/bootasm.S
+ cc boot/bootmain.c
+ cc tools/sign.c
+ ld bin/bootblock
'obj/bootblock.out' size: 472 bytes
build 512 bytes boot sector: 'bin/bootblock' success!
```

Step3: 开始生成ucore.img文件. dd命令生成ucore.img文件也经过了三步.每一步的具体操作如图中Step3所示.

```ZSH
10000+0 records in
10000+0 records out
5120000 bytes (5.1 MB) copied, 0.0138834 s, 369 MB/s
1+0 records in
1+0 records out
512 bytes (512 B) copied, 0.00042473 s, 1.2 MB/s
138+1 records in
138+1 records out
70783 bytes (71 kB) copied, 0.000804386 s, 88.0 MB/s
```

> 练习一  

> - 操作系统镜像文件ucore.img是如何一步一步生成的?详细说明Makefile文件中的每一条相关命令和命令参数的含义,并写出命令执行后的结果
> > 整个生成过程见上面的分析,这里省略

> - 系统上电后的启动过程
> > 2.1 上电开机 BIOS起始地址为CS:EIP=0XF000<4+0XFFF0=0XF0000+OXFFF0=0XFFFF0  
> > 2.2 BIOS跳转至bootloader的起始地址CS:EIP=0X0000<4+OX7C00=OX7C00  
> > 2.3 bootloader加载kernel进内存,起始地址为EIP0X100000 

>> BIOS是烧制计算机上的某个固件上,其起始地址在开机上电初始化CPU时设置的,为CS:EIP.BIOS负责检查CPU,内存,外部IO设备, 一切正常后将EIP寄存器的值设置为0x7c00,将CPU的执行权交给bootloader.  
>> bootasm.s和bootmain.c是bootloader的两个子程序. 它们负责1.初始化全局描述符表,以便通过段寄存器选择子查找对应段的起始地址与长度2.将运行模式从1M的实模式切换到4G的保护模式3.读取磁盘位于主引导扇区之后的扇区,加载kernel进内存.  
>> kernel运行后,整个操作系统就开始正常运行了.

> - 一个被操作系统认为是符合规范的硬盘主导扇区的特征是什么?  

> > 3.1 扇区最后两个字节的内容必须为0X55AA  
> > 3.2 bootloader代码不能超过446Bytes  

> 练习三
 
> - 分析bootloader进入保护模式的过程,并说明为什么要开启A20,以及如何开启A20的.

>  >  bootasm.s对cs,ds,ss,ex等寄存器进行初始化  
>  >  Intel的处理器采用向后兼容的指令体系结构, 因此在80386的机器上也能运行8086的指令. 80386在开机后实际是处于实模式, 有效的地址总线只有16根, 可以使用的地址空间只有1M. 对于内存的管理没有使用保护措施, 这导致任意程序可以直接访问内存, 显然不够安全.因此,需要将CPU切换到保护模式, 将有效地址总线从16根切换到20根, 可访问内存空间提高为4G, 而内存管理也相应改为段机制, 最后加载全启描述符以及设置CRO控制寄存器的保护模式使能位. 整个过程bootasm.s是通过8042寄存器的64h端口开启A20, 将16位地址总线切换到20位; 并通过8042的60端口将A20的最高置1,使能A20.     

>  - 初始化GDT

>  > 通过指定gdt的起始地址和长度完成.

> 练习四  
> 分析bootloader加载ELF模式的OS的过程: bootloader如何读取硬盘扇区?bootloader如何加载elf模式的OS?
>  > bootloader运行时还没有文件系统,要读取磁盘扇区只能通过LBA的program io,即访问磁盘的地址寄存器实现磁盘的读写.80386的主板有两个IDE,第一个IDE通过0x1f0-0x1f7寄存器实现磁盘读写;第二个IDE通过0x170-0x17f完成访问.  
>  > 每次读写磁盘时,先向0x1f7发送一条读写指令,以确定磁盘是否处于忙碌状态.如果不是忙状态,通过0x1f0端口读取数据.   
>  > bootloader读取了磁盘的8个扇区后, 通过判断elf文件的成员变量是否为某个固定值,以确定kernel文件的格式是否合法.  
>  > 合法的elf kernel文件被bootloader指定entry_point加载到内存.此后将CPU的交给kernel.

> 练习五
> 实现函数调用堆栈跟踪函数


### 计算机结构 

> CPU

- 寄存器
- 运算器
- 一级缓存(大概也就是几K的样子)
- 二级缓存(大概是几M的样子)
	
> 外设

- 显卡
	- 接口
	- ROM,存放显卡BIOS程序
- 光驱
	- 接口
	- ROM,存放光驱BIOS程序
- 硬盘
	- 接口
	- ROM,存放硬盘BIOS程序 
- USB接口	
	- 接口
	- ROM,存放USB BIOS程序 
- 网卡
	- 接口
	- ROM,存放网卡BIOS程序
- 键盘
	- 接口
	- ROM,存放键盘BIOS程序 

> 内存

- 地址
	- 用来访问内存空间的索引 
	- 80386的内存空间为:2^32=4G;8086的内存空间为: 2^20=1M
- 物理地址空间
	- CPU提交到地址总线的RAM
	- CPU检测到所有外设的EPROM(可擦除的只读存储器)
- 线性地址空间
	- 虚拟内存管理的产物,这个空间映射出整个物理内存空间,让每个应用程序认为自己独享整个内存地址空间.
- 逻辑地址
	- 应用程序直接使用的地址空间
	
段机制启动,页机制未启动: 逻辑地址->**段机制处理**->线性地址=物理地址  
段机制和页机制都启动: 逻辑地址->**段机制处理**->线性地址->**页机制处理**->物理地址

> 总线

- 控制总线
- 数据总线
- 地址总线
	- Intel 8086
	- Intel 80386

一台计算机的内存空间由CPU寄存器的宽度决定.Intel 8086CPU为16位,所以实际寻址空间是2^16=64K,但Intel用了非常精妙的**段地址:偏移量**方式寻址,以实现20位的寻址空间,这样x8086的内存空间为2^20=1M.当8086的寻址空间为1M时,我们说8086计算机工作在实模式.	

x8086的CS是一个专门用来存储代码段的段地址寄存器,即:只要是放在CS寄存器中的地址,指向的数据是程序代码.EIP寄存器是程序指令指针寄存器,这个寄存器存的是偏移量.x8086的代码段起始地址由CS:EIP组成.

### 计算机的启动

> S1: 电源开启

初始化CPU中的各种寄存器,它们都有自己的默认值,每次开启电源都加载相同的内容.CS: 0XF000H,EIP: 0XFFF0H.CPU通过这两个寄存器计算出第一个指令的地址位于CS:EIP=0XFFFF0H,于是通过控制总线发出一条读指令,并通过地址总线找0XFFFF0这个空间取指令,开始执行0XFFFF0H单元后的指令.0XFFFF0H这单元之后的代码属于BIOS程序.

初始化GDT(全局描述符表),GDT将CPU里所有段寄存器进行了定义.segment selector(段选择子,是一个指向GDT的index).

还有一个重要的寄存器CR0.这个寄存器的高13bits用来表示GDT index,中间用2bits表示系统特权级.2bits可以标识四种特权级,即0,1,2,3.内核工作在ring 0,应用程序工作在ring 3. ring 1和ring 2由于历史原因并没有使用.


> S2: BIOS

BIOS的起始地址为0XFFFF0H,是所有外设ROM的起始地址.主BIOS负责挨个检查各个外设的BIOS程序,以确定有多个外设正常工作.之后根据外设的情况建立引导次序.比如,如果确定有光驱,USB,硬盘三个外设,它会形成一个表格,之后引导次序寻找MBR主引导记录的存放位置.当然我们一般是将MBR存放在硬盘上了.

> S3: MBR

MBR一般存放在硬盘的0号扇区上,一共512Bytes.

BIOS找到MBR后,将其加载到内存(其实就是将EIP指向另一个程序boot loader的起始地址)中,这时EIP=0X7C00H.boot loader负责读硬盘0号扇区之后的几个扇区,以识别硬盘上的文件系统并初始化文件系统,从而去/boot目录下找到kernel代码,将其加载进内存,最终CPU的执行权交给内核.


### Linux磁盘管理

U盘,光盘,软盘,机械式硬盘,固态硬盘,磁带.

> 机械硬盘

![机械硬盘](/assets/磁盘结构1.png)  
![机械硬盘](/assets/磁盘结构2.jpg)

真空: 转速太快,与其它物体碰撞后果严重.用于	防尘.  
钢性盘片: 双面可读写  
机械臂: 可移动    
磁头: 读取电气信号  
磁道Tracks: 同心圆,不同的磁道周长不同,周长越长可以存储的数据越多.  
扇区Sector: 共512Bytes.数据按扇区来管理数据.包括扇区号,磁道号,盘片号等等.这些属于额外开销.  
柱面Cylinder: 所有盘片上同一磁道的构成.  
分区Partition: 磁盘的逻辑边界.不分分区,只能有一个文件系统.   
低级格式化: 磁盘出厂时进行格式化,主要是为了划分磁道,扇区等.但不进行分区.

> 指标

平均寻道时间(Average Seek Time): 指磁头移动到数据所在磁道需要的时间，这是衡量硬盘机械性能的重要指标,越短越好
平均潜伏期(Average Latency): 指当磁头移动到数据所在的磁道以后，等待指定的数据扇区转动到磁头下方的时间
平均访问时间(Average Access Time): 指从读/写指令写出到第一笔数据读/写实际开始所用的平均时间
数据传输率:(Data Transfer Rate): 指硬盘读/写数据的速度,单位为兆字节每秒(MB/s)
转速: 5400rpm/7200rpm/15000rpm, 越大越好  
外道: 比内道读写速率快,因为同样的时间段,磁头扫过的长度外道比内道长.

> 硬盘缓存区

硬盘缓存区也称缓存,作用是平衡内部与外部的数据传输率.为了减少洗衣机的等待时间,硬盘会将读取的资料先存入缓冲区,等全部读完或缓冲区填满后再以接口速率快速向主机发送


> MBR

Master boot record: 主引导记录.

O磁道0扇区共512Bytes,叫主引导记录.不属于操作系统的独立的存储空间,被分成三段.

**0-445Bytes**: boot loader程序,引导磁盘上的操作系统,使之运行起来.

**446-509Bytes**: 每16Bytes构成一个分区,共64Bytes, 因此最多可以有4个分区.

**510-511Bytes**: 一个字节存一个magic number,标识MBR是否有效, 这两个字节的16进制数为: **OX55AAH**.


> 分区

文件存储在磁盘上以柱面为基准,即一个文件放在了同一柱面所有盘面上.因此为了方便文件的存储,将磁盘按柱面进行分区.由MBR可知一个磁盘最多只有四个分区.

- 最多四个主分区
- 3个主分区+1个扩展分区
- 1个扩展分区可以指向多个逻辑分区

问题是,文件具体放在一个分区的什么位置,这个文件有多大,是什么类型的文件?...

这些内容必须有效的管理起来.于是有了一个软件叫**文件系统**.文件系统在管理分区时,将物理分区分成**元数据存储区**和**数据存储区**两个逻辑空间来管理.数据存储区里的最小存储单元是block.元数据区有一个区域---块位图(bitmap): 将所有数据存储区里的所有block在这里建立相应的位.0表示unused,1表示used. 元数据区还有一个区域---文件路径管理区.

于是一个文件在文件系统中是这样的组成: 
一份元数据: 文件名称,文件路径, block的编号,文件有多少个block等.
一份数据: 一个文件的所有block.

哪些block是空闲的,哪些block已经被占用?
每个block的第一位进行标记,为1表示空闲,0表示被占用.但是这样做效率非常低!所以实际在管理block时,在元数据区找一块空间来管理block的占用状态.

> 文件系统初实

- 元数据
	- 块位图: 提高查找空闲block的速度.
	- inode: 一个文件一个inode. inode号,文件属主,属主,blocks. 没有文件名.
	- inode位图: 用来遍历inode条目是否空闲,可提高遍历速度.
- 数据
	- blocks

**NOTE:**目录也是一个文件.所以文件名其实涉及多个文件,如:/etc/passwd,这其中有

- / --- 根inode号,根block中保存有 etc及其inode号
- etc --- etc inode, etc block中保存有 passwd 及其 inode号
- pawwd --- passwd inode号, passwd blocks 

每次如此找文件,有些费力.实际是在第一次遍历结束后建立了一个缓存条目: dentry.第一个dentry都记录了/etc/passwd这种目录对应的所有block.

**创建文件**

/data/test.txt 10k数据 每一个block 2k

- Step1扫描inode位图,找一个空闲的inode,比如inode号为 100
- Step2找到data对应的inode,inode号为99,由inode找到对应的block,block号为11
- Step3在这个11号block号中添加一条数据inode_id:100,name: test.txt
- Step4扫描block位图,找到空闲块后,往block里写数据,并将这个block号保存在100号inode中.

**删除文件**

只需要将文件对应的inode位图与block位图标记为未使用,下次再写数据时,只是覆盖了原来block里的内容.



ls -l中列出的文件次数,表示的硬链接的次数.如果一个文件的硬链接的次数大于2时,这个文件不会真正被删除.

**复制文件**

实际就是一个创建新文件.但是同一个分区剪切,并没有创建新文件,而只是修改了目录文件block里的文件名而已,inode与block的编号都没有变化,因此同一个分区上的文件剪切速度很快.

**创建链接**

/backup/a/m.txt
/backup/b/n.txt
这两个文件可以指向同一个inode.这就是硬链接.

软链接的inode中没有block,但是它存的是一个文件路径.要找这个软链接指向的真实的文件,得按这个路径再去找一个.软链接的权限都是777,因为这个软链接最终指向的文件有自己的权限,用户的访问权限由真实文件的权限决定.

```ZSH
// 格式 ln [-s -v] src dest, 不带选项是硬链接 -s软链接 -v verbose
// 硬链接 
mkdir /backup
cd /backup
cp /etc/rc.d/rc.sysinit ./abc
ls -l
mkdir test
ln abc test/abc2
ls -l
cd test
ls -l
man ls 
cd ..
ls -i
ls -i test/
rm abc
ls -li test/

//软链接 
mv test/abc2 abc
ln -sv /backup/abc /backup/test/abc2
ls -i
ls -li test
```

**硬链接**

- 只能对文件创建,不能对目录创建硬链接. 目录硬链接的次数始终是2.
- 硬链接指向inode
- 不能跨文件系统创建
- 创建硬链接会增加文件被链接的次数

**软链接** 

- 文件和目录都可以创建
- 指向的是一个路径
- 可以跨文件系统
- 不会增加文件被链接的次数
- 其大小为其所指向路径的字符数长度

```ZSH
// du 命令: estimate file space usage
du -sh [目录]
// df 命令: report file system disk space usage
df 
df -h
df -i
df -i -P
```

> 设备文件

Linux的思想之一是**一切皆文件**,所以设备在Linux里也是文件,叫设备文件,都在/dev目录下./dev下的所有文件都有一个设备号(major number)和次设备号(minor number).主设备号用于标识设备类型,次设备号用于标识同一种类型的多个不同设备.设备文件并不真正占用block,只在inode里存储着设备文件主次设备号,所以这类文件没有大小.

设备文件主要目的是设备的访问入口,默认是被内核使用的.每一个设备文件是通过内核与一个实际的物理设备连接起来的.

**块设备b**: 按块为单位,随机访问的设备.常见: 硬盘

**字符设备c**: 按字符为单位,线性访问的设备.常见: 键盘

设备文件的创建: `mknod命令 make block or character special files`

```ZSH
// mknod [OPTION]... NAME TYPE [MAJOR MINOR]
// 创建一个b设备,设备名为mydev, 主设备号66, 次设备号0
mknod mydev c 66 0
// 创建一个b设备,设备名为mydev, 主设备号66, 次设备号1, 设备权限为640
mknod -m 640 mydev2 c 66 1
```

> 硬盘设备

设备文件名

IDE,ATA: 以hd开头(centos6以后也用sd开头)
SATA: 以sd开头
SCSI: 以sd开头
USB: 以sd开头


举栗: 一个处理器可以接一块硬盘,所以双核的处理器可以接四块硬盘.四个硬盘在文件系统中这么定义

- 第一块硬盘: /dev/sda
	- 分区1: /dev/sda1
	- 分区2: /dev/sda2
	- 分区3: /dev/sda3
	- 分区4: /dev/sda4
	- 逻辑分区5: /dev/sda5
	- 依此类推
- 第二块硬盘: /dev/sdb
- 第三块硬盘: /dev/sdc
- 第四块硬盘: /dev/sdd

根据磁盘的MBR可知,一个硬盘最多只能有四个主分区或者是三个主分区与一个扩展分区(指向多个逻辑分区).所以所有的逻辑分区都是从5开始编号.主分区最多到4.

查看系统上的所有磁盘信息`fdisk`	

#### 文件系统

文件系统是属于内核.文件系统需要借助用户空间的进程,然后通过系统调用进入内核空间.

**文件系统分类**

- VFS: 虚拟文件系统,支持众多类型的文件系统
- CIFS: Windows 上的网卡邻居用的文件系统
- FAT16/FAT32/NTFS: Windows
- ISO9660: 光盘
- ext2,ext3,ext4,xfs,reiserfs:
- jfs: IBM的日志文件系统
- NFS: 网络文件系统
- gfs2: 全局文件系统
- ocfs2: 集群文件系统

Linux几乎支持上述所有文件系统,因为有VFS用来管理所有文件系统.但是Linux对NTFS的支持不是很好,最好是不要往NTFS的文件系统里写数据.有了VFS后,用户空间进程只需要通过一个系统调用与VFS交互即可.

每一个分区可以是不同的文件系统,但是最后都要挂载到根目录下.

内核只知道根的位置.


**磁盘格式化**

- 低级格式化: 厂商完成,负责创建磁道,扇区
- 高级格式化: 创建文件系统,这后才可被挂载.

**创建分区(创建文件系统)**

可以用`fdisk /dev/sda `

>fdisk /dev/sda
>> p: 显示当前硬件的分区,包括没保存的改动  
>> n: 创建新分区  
>>> e: 扩展分区
>>> p: 主分区(1-4)
>> d: 删除一个分区  
>> w: 保存退出  
>> t: 修改分区类型  
>> l: 显示所支持的所有类型  







