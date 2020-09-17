# Linux 杂项学习笔记  
## 1. 添加全局变量  
```vim
    #在profile文件中添加 arm-fsl-linux-gnueabi-gcc 
    export PATH=$PATH:/work/tool/fsl-linaro-toolchain/bin
    #使生效
    $ source /etc/profile
```
## 2. 修改apt-get下载源  
```vim
    $ vi /etc/apt/sources.list
    $ apt-get update
```
## 3. 安装软件方式  
* 在线安装  
```vim
    # 在线安装小火车
    $ sudo apt-get install sl
```
* deb包安装  
```vim
    $ sudo dpkg -i *.deb #或直接双击安装
```
* 源码编译安装
```vim
    $ ./configure
    $ make  
    $ make install
```  
## 4. 应用开机启动
```
    系统启动时需要加载的配置文件
    /etc/profile、/root/.bash_profile
    /etc/bashrc、/root/.bashrc
    /etc/profile.d/*.sh、/etc/profile.d/lang.sh
    /etc/sysconfig/i18n、/etc/rc.local（/etc/rc.d/rc.local）
```
* rc.local
```vim
    #!/bin/sh -e
    #
    # rc.local
    #
    # This script is executed at the end of each multiuser runlevel.
    # Make sure that the script will "exit 0" on success or any other
    # value on error.
    #
    # In order to enable or disable this script just change the execution
    # bits.
    #
    # By default this script does nothing.

    #添加开机启动 strat.sh
    /home/root/start.sh &
    exit 0
```
* /etc/profile.d/
```
    将写好的脚本（.sh文件）放到目录 /etc/profile.d/ 下，系统启动后就会自动执行该目录下的所有shell脚本
```
## 5. 网络应用协议  
* FTP  
```
    文件传输协议(File Transfer Protocol)，默认端口号21
```
```vim
    # 搭建ftp服务
    $ sudo apt-get install vsftpd
    $ sudo sevice vsftpd start      #开启ftp服务
    $ sudo netstat -a | grep ftp    #查询ftp服务 
```
* TFTP  
```
    简单文件传送协议TFTP(Trivial File Transfer Protocol)是一个TCP/IP协议族中一个很小且易于实现的文件传送协议。TFTP也是使用客户服务器方式，但它使用UDP数据报，因此TFTP需要有自己的差错改正措施
    TFTP特点：
    1.每次传送的数据PDU中有512字节的数据，但最后一次可不足512字节
    2.数据PDU也称为文件块(block)，每个块按序编号，从1开始
    3.支持ASCII码或二进制传送
    4.可对文件进行读或写
    5.使用很简单的首部
    6.TFTP只支持文件传输而不支持交互
    7.TFTP没有一个庞大的命令集
    8.没有列目录的功能
    9.也不能对用户进行身份鉴别
```
```vim
    # 搭建tftp服务
    $ sudo apt-get install tftp-hpa tftpd-hpa
    $ sudo vi /etc/default/tftpd-hpa    #修改tftp目录
    $ sudo service tftpd-hpa restart    #重启tftp服务
    $ sudo netstat -a | grep tftp       #查询tftp服务
```
> FTP和TFTP区别  
> 1. 交互使用FTP，TFTP仅允许单向传输的文件  
> 2. FTP提供身份验证，TFTP不提供身份验证  
> 3. FTP 使用已知TCP端口号[20]的数据和[21]用于连接对话框。TFTP用UDP 端口号[69]进行文件传输  
> 4. FTP依赖于TCP，是面向连接并提供可靠的控件。TFTP依赖UDP，需要减少开销, 几乎不提供控件  
* SFTP  
```
    Secure File Transfer Protocol的缩写，安全文件传送协议。可以为传输文件提供一种安全的加密方法
```
> FTP和SFTP区别  
> 1. SFTP与FTP有着几乎一样的语法和功能。SFTP为SSH的一部份，是一种传输档案至Blogger伺服器的安全方式。其实在SSH软件包中，已经包含了一个叫作SFTP(Secure File Transfer Protocol的安全文件传输子系统，SFTP本身没有单独的守护进程，它必须使用sshd守护进程（端口号默认是22）来完成相应的连接操作，所以从某种意义上来说，SFTP并不像一个服务器程序，而更像是一个客户端程序
> 2. SFTP同样是使用加密传输认证信息和传输的数据，所以，使用SFTP是非常安全的。但是，由于这种传输方式使用了加密/解密技术，所以传输效率比普通的FTP要低得多，如果您对网络安全性要求更高时，可以使用SFTP代替FTP  
* SCP
```
    scp是secure copy 的缩写, scp是linux系统下基于ssh登陆进行安全的远程文件拷贝命令
```
> SFTP和SCP区别
> 1. scp是一种基于SSH的协议，可在网络上的主机之间提供文件传输。 使用scp，您可以在主机之间快速传输文件以及基本文件属性，例如访问权限和通过FTP无法可用的时间戳。 该协议使用RCP传输文件和SSH以提供身份验证和加密
> 2. sftp是一种更强大的文件传输协议，也基于SSH。 更像是远程文件管理协议，sftp允许对远程文件（查看目录，删除文件和目录等）进行一系列操作  

> SFTP和SCP选择
> 1. 速度考虑。在传输文件时，scp通常比sftp快得多，尤其是在网络延迟很高的情况下。这是因为scp实现了更高效的传输算法，不需要等待数据包确认[选择scp]  
> 2. 安全性考虑。由于两种协议都都基于SSH，因此它们都提供相同的安全功能，包括密码和数据加密以及公钥验证。[选择scp或sftp]  
> 3. 功能/可用性考虑。scp提供的功能像其名称所暗示的那样：安全地复制文件（Secure copy）。如果您或您的用户将管理文件（包括查看/搜索目录，创建文件夹和组织文件，删除或重命名文件等），sftp是优秀的协议。此外，sftp还支持断点续传，这在网络连接不佳的环境中将大有帮助。[选择sftp]  
> 传输文件大小考虑。scp和sftp都没有文件大小限制。但是，根据文件的大小，scp的文件传输速度可能会有所帮助[选择scp]  
* SSH  
```
    远程控制通信协议(Secure Shell)，默认端口号22，SSH1使用RSA加密密钥，SSH2使用数字签名算法(DSA)密钥保护连接和认证
```
```vim
    # 搭建ssh服务
    $ sudo apt-get install openssh-client #安装ssh客户端
    $ sudo apt-get install openssh-server #安装ssh服务端
    $ sudo /etc/init.d/ssh start    # 启动ssh服务
    $ ps -ef | grep ssh #查看是否启动
    $ vi /etc/ssh/sshd_config   #查看修改配置文件
```
* Telnet  
```
    Telnet协议是TCP/IP协议族中的一员，是Internet远程登录服务的标准协议和主要方式，端口号23
```
[搭建telnet服务](https://jingyan.baidu.com/article/48b558e35e51f97f38c09ae7.html)  
> Telnet和SSH区别  
> 1. Telnet和SSH都是用于远程控制的通信协议  
> 2. SSH是加密的,需要交换密钥;而Telnet是明文的,传输的是明文字符  
* samba  
```
    samba是基于SMB协议（ServerMessage Block，信息服务块）的开源软件，samba也可以是SMB协议的商标。SMB是一种Linux、UNIX系统上可用于共享文件和打印机等资源的协议，这种协议是基于Client/Server型的协议，Client端可以通过SMB访问到Server（服务器）上的共享资源
```
* NFS  
```
    网络文件系统，英文Network File System(NFS)，是由SUN公司研制的UNIX表示层协议(presentation layer protocol)，能使使用者访问网络上别处的文件就像在使用自己的计算机一样，默认端口号：2049
```
```vim
    # 搭建nfs服务
    $ sudo apt install nfs-common           #安装客户端
    $ sudo apt install nfs-kernel-server    #安装服务端
    $ sudo vi /etc/exports                  #修改配置文件
    #添加nfs目录: /home/bhky/work/nfs-dir *(rw,sync,no_root_squash,no_subtree_check)
    #
    $ sudo /etc/init.d/rpcbind restart      #重启服务
    $ sudo /etc/init.d/nfs-kernel-server restart   

    $ mount -t nfs localhost:/home/bhky/work/nfs-dir /mnt   #挂载测试方式1
    $ mount -t nfs ip:/home/bhky/work/nfs-dir /mnt  #挂载测试方式2

```
* NTP  
```
    NTP是Network Time Protocol的缩写，又称为网络时间协议。是用来使计算机时间同步化的一种协议，它可以使计算机对其服务器或时钟源（如石英钟，GPS等等)做同步化，它可以提供高精准度的时间校正（LAN上与标准间差小于1毫秒，WAN上几十毫秒），且可介由加密确认的方式来防止恶毒的协议攻击。
    NTP默认使用的端口号是UDP123
```
```vim
    $ sudo apt-get install ntp        #安装服务端软件
    $ sudo apt-get install ntpdate    #安装客户端软件
    $ vi /etc/ntp.conf                #修改配置文件
    $ netstat -ln | grep 123          #查询是否启动方式1
    $ service ntp status              #查询是否启动方式2
    $ netdate ip    #同步时间方式1
    $ ntpdate-sync  #同步时间方式2(修改/etc/default/ntpdate)
    #时区相关
    #/etc/localtime或者查看/etc/profile中TZ
```
## 6. 设备类型 
```
　　/dev/hd[a-t]：IDE设备
　　/dev/sd[a-z]：SCSI设备
　　/dev/fd[0-7]：标准软驱
　　/dev/md[0-31]：软raid设备
　　/dev/loop[0-7]：本地回环设备
　　/dev/ram[0-15]：内存
　　/dev/null：无限数据接收设备,相当于黑洞
　　/dev/zero：无限零资源
　　/dev/tty[0-63]：虚拟终端
　　/dev/ttyS[0-3]：串口
　　/dev/lp[0-3]：并口
　　/dev/console：控制台
　　/dev/fb[0-31]：framebuffer
　　/dev/cdrom => /dev/hdc
　　/dev/modem => /dev/ttyS[0-9]
　　/dev/pilot => /dev/ttyS[0-9]
　　/dev/random：随机数设备
　　/dev/urandom：随机数设备
```
## 7. 文件颜色 
```
    白色:普通文件 
    蓝色:目录 
    绿色:可执行 
    红色:压缩文件 
    浅蓝色:链接文件
    红色闪烁:链接文件有问题
    黄色:设备文件 
    灰色:其他文件
```
## 8.文件类型
```
    七种类型
    1.普通文件类型
        Linux中最多的一种文件类型, 包括 纯文本文件(ASCII)；二进制文件(binary)；数据格式的文件(data);各种压缩文件.第一个属性为 [-]
    2.目录文件
        就是目录， 能用 # cd 命令进入的。第一个属性为 [d]，例如 [drwxrwxrwx]
    3.块设备文件
        块设备文件 ： 就是存储数据以供系统存取的接口设备，简单而言就是硬盘。例如一号硬盘的代码是 /dev/hda1等文件。第一个属性为 [b]
        字符设备
    4.字符设备文件
        即串行端口的接口设备，例如键盘、鼠标等等。第一个属性为 [c]
    5.套接字文件
        这类文件通常用在网络数据连接。可以启动一个程序来监听客户端的要求，客户端就可以通过套接字来进行数据通信。第一个属性为 [s]，最常在 /var/run目录中看到这种文件类型
    6.管道文件
        FIFO也是一种特殊的文件类型，它主要的目的是，解决多个程序同时存取一个文件所造成的错误。FIFO是first-in-first-out(先进先出)的缩写。第一个属性为 [p]
    7.链接文件
        类似Windows下面的快捷方式。第一个属性为 [l]，例如 [lrwxrwxrwx]
```
```
    -:普通文件
    d:目录文件
    l:链接文件
    b:块设备
    c:字符设备
    s:网络文件
    p:管道文件
```
## 9.文件系统自动挂载
```
/etc/fstab
```
## 10.热插拔
```
```
## 11.应用多版本选择
```
    # arm-linux-gnueabi-gcc-7和arm-linux-gnueabi-gcc-5配置和选择
    update-alternatives --install   #进行优先级配置（优先级数字大则为auto mode）
    update-alternatives --config    #进行选择
```
## 12.运行级别
```
    0： 系统停机（关机）模式，系统默认运行级别不能设置为0，否则不能正常启动，一开机就自动关机。
    1：单用户模式，root权限，用于系统维护，禁止远程登陆，就像Windows下的安全模式登录。
    2：多用户模式，没有NFS网络支持。
    3：完整的多用户文本模式，有NFS，登陆后进入控制台命令行模式。
    4：系统未使用，保留一般不用，在一些特殊情况下可以用它来做一些事情。例如在笔记本电脑的电池用尽时，可以切换到这个模式来做一些设置。
    5：图形化模式，登陆后进入图形GUI模式或GNOME、KDE图形化界面，如X Window系统。
    6：重启模式，默认运行级别不能设为6，否则不能正常启动，就会一直开机重启开机重启
```
## 13.man
```
    1是普通的命令
    2是系统调用,如open,write之类的(通过这个，至少可以很方便的查到调用这个函数，需要加什么头文件)
    3是库函数,如printf,fread
    4是特殊文件,也就是/dev下的各种设备文件
    5是指文件的格式,比如passwd,就会说明这个文件中各个字段的含义
    6是给游戏留的,由各个游戏自己定义
    7是附件还有一些变量,比如向environ这种全局变量在这里就有说明
    8是系统管理用的命令,这些命令只能由root使用,如ifconfig
```
```
    简要说明
    NAME:名字和简短介绍
    SYNOPSIS:命令下达语法
    DESCRIPTION:描述
    EXAMPLES:例子
    SEE ALSO:更多可使用的
```
## 14.kill
```vim
    $ killall -g 进程名 #杀掉同一组的进程
```
## 15.时间格式
```
GMT:格林威治标准时间
UTC:世界协调时间
DST:夏日节约时间
CST:代表４个不同时区：
    UT-6:00（美国）
    UT+9:30（澳大利亚）
    UT+8:00（中国）
    UT-4:00 （古巴）
```
## 16.快速查看宏定义
```
    建一个test.c,把需要查看的宏用上，然后gcc -E test.c > test.info,打开test.info查看即可
```
## 17.配置arm为nfs启动
```
    arm上进行nfs配置(可在boot时直接挂在根文件系统,也可以进入系统后挂载)
    1.首先验证可以手动挂载,确保挂载正确
    2.修改menuconfig
        a.启动网络文件系统功能
        b.开启nfs功能(boot)
        c.网络配置(Networking support->Networking options->IP支持)
    3.nfs挂载(uboot配置)
        printenv    #查看环境变量，按如下配置
        baudrate=115200
        boot=$LOAD && $LOAD $fdtaddr $fdtfile && setenv bootargs $mtdparts $params root=/dev/$BOOTARGS && bootm $loadaddr - $fdtaddr
        boot_emmc=LOAD='ext2load mmc 0:1'; BOOTARGS='mmcblk0p1 ro'; run boot
        《boot_net=LOAD=tftp; BOOTARGS="nfs rw ip=192.168.1.60:192.168.1.221:192.168.1.1:255.255.255.0::eth0:off nfsroot=$serverip:$rootpath,nolock"; run boot》
        boot_rom=LOAD=fsload; BOOTARGS='mtdblock1 ro rootfstype=cramfs,jffs2'; run boot
        boot_sata=dcache off; sata init; LOAD='ext2load sata 0:1'; BOOTARGS='sda1 ro'; run boot
        boot_sd=LOAD='ext2load mmc 1:1'; BOOTARGS='mmcblk1p1 ro'; run boot
        boot_sdmmc=LOAD='ext2load mmc 2:1'; BOOTARGS='mmcblk3p1 ro'; run boot
        boot_usb=usb start; LOAD='ext2load usb 0:1'; BOOTARGS='sda1 ro'; run boot
        bootcmd=run boot_net
        bootdelay=1
        bootfile=/boot/uImage
        ethact=FEC
        ethaddr=00:30:64:14:bf:a2
        ethprime=FEC
        fdt_high=0xffffffff
        fdtaddr=0x18000000
        fdtfile=/boot/lec-imx6q.dtb
        initrd_high=0xffffffff
        《ipaddr=192.168.1.60》
        loadaddr=0x12000000
        mtdparts=mtdparts=spi32765.0:384K(u-boot)ro,-(root)
        《netmask=255.255.255.0》
        params=rootwait console=ttymxc0,115200
        《rootpath=/home/bhky/work/nfs-dir》
        《serverip=192.168.1.221》
        stderr=serial
        stdin=serial
        stdout=serial
```
## 18.i2ctools
```
    # i2cdump、i2cget、i2cset和i2cdetect使用
```
## 19.canutils
```
    # cansend和candump使用
```
## 20.内核添加驱动源码和驱动配置
```
    1.在drivers下面新建一个文件夹 mkdir gpiotest
    2.将驱动源码添加进gpiotest
    3.新建Kconfig和Makfile文件
        Kconfig格式仿照其他Kconfig写
        Makfile格式仿照其他Makfile写
    4.将gpiotest添加进drivers下面的Kconfig
        source "drivers/gpiotest/Kconfig"
    5.将gpiotest添加进drivers下面的Makefile
        obj-$(CONFIG_GPIOTEST)	+= gpiotest/
        通过menuconfig就可以在Device Drivers最后看到 新添加的gpio选项
```
## 21.手动加载/移除模块
```vim
    #方式1
    insmod drv.ko   #安装模块
    rmmod drv.ko    #移除模块
    #方式2
    modprobe drv    #安装模块，模块是放在/lib/modules/`uname -r`/, 然后需要先depmod
    rmmod drv       #移除模块
```
## 22.虚拟文件系统
* /proc/kallsyms
```
    CONFIG_KALLSYMS=y 符号表中包含的所有函数
    CONFIG_KALLSYMS_ALL=y 符号表中包含的所有变量（包括没有用到EXPORT_SYSBOL导出的变量）
    CONFIG_KALLSYMS_EXTRA_PASS=y
```
## 23.运算符优先级
```
    先指针，单双目
    先算术，后移位
    比大小，判相等
    位操作，逻辑跟
    多赋值，逗号底
    单多赋，右到左
```
## 24.inetl网卡(i210)移植
```
	1.lspci 查看是否有pci id为8086:153x
	2.通过EEUPDATE更新i210配置(invm和flash两种),工具在intel官网下载
	3.内核中添加对应的驱动
```
## 25.安装交叉编译工具链
```

```
## 26.ip设置
```
    静态ip
    dhcp
    注：在连接到支持dhcp的路由上时，怎么使用
```
## 27.[stty](https://baike.baidu.com/item/stty)
```
    打印或更改terminal(终端)的设置
```
## 28.485转向
```
    有三种方式实现收发转向
    1.DCE模式
        TX  <---> RO (485转换芯片PIN1)
        RX  <---> DI (485转换芯片PIN4)
        RTS <---> RE/DE (485转换芯片PIN2和PIN3,PIN2和PIN3短接)
        RTS作为普通IO,手动输出高低实现收发转向控制(应用层控制GPIO)
    2.DCE模式(imx6只有在DCE时可以,手册上有说明)
        TX  <---> RO (485转换芯片PIN1)
        RX  <---> DI (485转换芯片PIN4)
        CTS <---> RE/DE (485转换芯片PIN2和PIN3,PIN2和PIN3短接)
        采用硬件流控,驱动层实现自动转向
    3.DTE模式
        RX  <---> RO (485转换芯片PIN1)
        TX  <---> DI (485转换芯片PIN4)
        RTS <---> RE/DE (485转换芯片PIN2和PIN3,PIN2和PIN3短接)
        采用硬件流控,驱动层实现自动转向
```
## 29.文件系统
```
    在嵌入式linux应用中，主要的存储设备为RAM和FLASH。常用的基于存储设备的文件系统类型包括：jffs2，yaffs，cramfs，ramdisk，ramfs等。
    1.jffs2：日志闪存文件系统版本2，用于NOR flash，可读写、支持数据压缩的日志文件系统
    2.yaffs/ubitFS：用于nand flash设计的一种日志型文件系统，不支持数据压缩
    3.Cramfs:只读的压缩文件系统（用的越来越少）
    4.Ramdisk：将部分固定大小的内存当做块设备来使用
    5.Initramfs：将内存当做块设备用（现在用的多）
    6.NFS：网络文件系统 (开发阶段使用)
```
## 30.常见压缩
![tar](./tar.png)  
## 31.TCP流程
![tcp](./tcp.jpg)  
## 32.UDP流程  
![udp](./udp.jpg)  

