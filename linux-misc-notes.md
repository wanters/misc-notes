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
    $ apt-get update        //将所有包的来源更新，也就是提取最新的包信息
    $ apt-get upgrade -y    //将系统中旧版本的包升级成最新的
    $ apt-get dist-upgrade -y   //
```
## 3. [安装/卸载软件方式](https://www.cnblogs.com/kinwing/p/11829546.html)  
### 安装
* 在线安装  
```vim
    // 在线<新装>安装小火车
    $ sudo apt-get install sl
    // <重装>小火车
    $ sudo apt-get install --reinstall sl
    // <更新>当前版本的包而不是安装新的版本
    $ sudo apt-get install --only-upgrade sl
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

### 卸载[配置文件]
* remove只删除程序文件，保留相关的配置文件
```vim
    $ sudo apt-get remove vim
```
* purge同时删除程序文件及其配置文件
```vim
    $ sudo apt-get purge vim
    # 同上
    $ sudo apt-get remove --purge vim
```
* 删除自动安装的软件包(由于依赖而安装的)
```vim
    $ sudo apt-get autoremvoe
```
### 查看已安装软件  
* 模拟执行并输出结果  
```vim
    # 模拟执行并输出结果
    $ sudo apt-get -s install tree
    # 安装指定版本
    $ sudo apt-get install tree=1.7.0-5
```
* 查看已安装的软件  
```
    $ dpkg -l
```
### 搜索软件可安装版本
```vim
    $ sudo apt-cache search g++ | grep g++
    //使用 apt 安装的 g++ 编译器和相关库的版本只能选择大的版本号如 6 ，而无法指定具体的小版本号 6.3.0，如笔者安装的 g++-6 的版本实际为 g++-6.4.0.
    $ sudo apt install g++-n g++-n-multilib        //安装对应的 g++ 编译器和库  
```
### 总结
| apt 命令 | 取代的命令 | 命令的功能 |
| --- | --- | --- |
|apt install |apt-get install |安装软件包 |
|apt remove  |apt-get remove  |移除软件包 |
|apt purge   |apt-get purge   |移除软件包及配置文件| 
|apt update  |apt-get update  |刷新存储库索引 |
|apt upgrade |apt-get upgrade |升级所有可升级的软件包| 
|apt autoremove |apt-get autoremove |自动删除不需要的包| 
|apt full-upgrade |apt-get dist-upgrade |在升级软件包时自动处理依赖关系| 
|apt search |apt-cache search |搜索应用程序| 
|apt show   |apt-cache show   |显示装细节| 
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
* update-rc.d  
[a.启动流程](https://blog.csdn.net/u011857683/article/details/81395893)  
[b.启动脚本编写模板](https://blog.csdn.net/weixin_34146805/article/details/89826513)
```
    service、chkconfig和update-rc.d的关系
    * 三者都是对/etc/init.d文件下的脚本或者服务进行操作的命令
    * service是即时生效，重启后失效
    * chkconfig是当前不生效，重启之后才生效的命令(相当于开机自启动)
    * update-rc.d是ubuntu中替代chkconfig

    下面详细说明update-rc.d命令
    * update-rc.d [-n] [-f] name remove 用于移除脚本
      update-rc.d [-n] name default/start/stop [NN | SS KK] level . 用于添加脚本(注意最后的句点)
      NN表示执行序号（0-99），SS表示启动时的执行序号，KK表示关机时的执行序号，SS、KK主要用于在脚本直接的执行顺序上有依赖关系的情况下。
      -n：不做任何事情，只显示将要做的。（预览、做测试）
      -f：强制移除符号连接，即使 /etc/init.d/script-name 仍然存在
    * 凡是以Kxx开头的，都以stop为参数来调用
      凡是以Sxx开头的，都以start为参数来调用
    * rcS.d、rc5.d、rc.local启动顺序
        (1) 启动Boot Manager
        (2) 加载系统内核，启动init进程，init进程是Linux的根进程，所有的系统进程都是它的子进程
        (3) init进程读取“/etc/inittab”文件中的信进入inittab中预设的运行级别，按顺序运行该运行级别对应文件夹(init*.d)下的脚本。
            脚本通常以“start”参数启动，并指向一个系统中的程序。
            通常情况下，“/etc/rcS.d/”目录下的启动脚本首先被执行，然后是“/etc/rcN.d/”目录。
            例如您设定的运行级别为3,那么它对应的启动目录为“/etc/rc3.d/”。
        (4) 根据“/etc/rcS.d/”文件夹中对应的脚本启动Xwindow服务“xorg” 。
            Xwindow为Linux下的图形用户界面系统。
        (5) 启动登录管理器，等待用户登录。
    * /etc/profile /etc/rc.d/rc.local /etc/inittab, /etc/init.d/rcS和/etc/profile、~/.bash_profile启动顺序
        是先加载环境变量还是先启动脚本？？？
        /etc/inittab,/etc/init.d/rcS和/etc/profile三者的顺序就是：
            /etc/inittab > /etc/init.d/rcS > /etc/profile
        注意：
            rcX.d是执行启动脚本的【应用】
            profile是配置环境变量的，当然也可以执行应用【变量，当然也可以是脚本的方式定义变量，如：在profile.d下】
    * Ubuntu系统架构关于启动项大致分为四类，每一类都分为系统级和用户级
        (1) 第一类upstart，或者叫job，由init管理，配置文件目录/etc/init，~/.init
        (2) 第二类叫service，由rc.d管理，配置文件目录/etc/init.d，以及/etc/rc.local文件
        (3) 第三类叫cron，由contab管理，使用crontab进行配置
        (4) 第四类叫startup，由xdg管理，配置文件目录/etc/xdg/autostart，以及~/.config/autostart
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
> 3. FTP 使用已知TCP端口号[20]的数据和[21]用于连接对话框。TFTP用UDP端口号[69]进行文件传输  
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
    #cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
```
    注意
    1.ntpd和ntpdate的区别
    2.来回链路的时延Sigma：
        Sigma = (t4-t1)-(t3-t2)
    因此，假设来回网络链路是对称的，即传输时延相等，那么可以计算客户端与服务器之间的时间误差Delta为：
        Delta = t2-t1-Sigma/2=((t2-t1)+(t3-t4))/2

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
* /etc/mtab
```
1.文件格式
    /etc/mtab 的格式和/etc/fstab 是一样的.但这个文件不能算是用户配置文件,
    他是由系统维护的.和/etc/fstab 的区别在于, fstab 是系统启动时需挂载的文
    件系统列表,而 mtab 是系统当前已挂载的文件系统列表,它由系统维护,在用户
    执行了 mount 或者 umount 命令后自动更新.用户不应该对此文件作任何修改
2.安全性
    /etc/mtab 的默认权限仍然是 644
3.相关命令
    mount
    umount
    smbmount
```
* /etc/fstab
```
1.文件格式
    /etc/fstab 记载了系统启动时自动挂载的文件系统。一行为一条记录。每条记
    录有 6 个字段，字段间用空格或者 tab 键分开。这六个字段分别是：设备名称，
    挂载点（除交换分区为 swap 外，都必须是一个存在的目录名），文件系统类型，
    mount 选项，是否需要 dump（1 表示需要，0 表示不需要），在 reboot 期间 fsck
    检查的顺序（激活文件系统设定为 1，其余文件系统设定为 2，若设定为 0 表示
    该文件系统不需要被检查）。
    在 linux 和 windows 共存时，也许想开机自动挂载 windows 分区，那么就可以
    在这个文件里加上相应的记录。
    某些时候对硬盘分区作了调整以后，这里也需要做一些相应的修改。否则会出
    现一些问题。
    可用的 mount 选项：
    async
        对该文件系统的所有 I/O 操作都异步执行
    ro
        该文件系统是只读的
    rw
        该文件系统是可读可写的
    atime
        更新每次存取 inode 的存取时间
    auto
        可以使用 -a 选项 mount
    defaults
        使用预设的选项：rw,suid,dev,exec,auto,nouser,async
    dev
        解释在文件系统上的字符或区块设备
    exec
        允许执行二进制文件
    noatime
        不要在这个文件系统上更新存取时间
    noauto
        这个文件系统不能使用 -a 选项来 mount
    nodev
        不要解释在文件系统上的字符或区块设备
    noexec
        不允许在 mounted 文件系统上执行任何的二进制文件。这个选项对于具有包含
        非它自己的二进制结构的文件系统服务器而言非常有用
    nosuid
        不允许 setuid 和 setgid 位发生作用。（这似乎很安全，但是在安装 suidperl
        后，同样不安全）。
    nouser
        限制一般非 root 用户 mount 文件系统
    remount
        尝试重新 mount 已经 mounted 的文件系统。这通常是用来改变文件系统的 mount
        标志，特别是让只读的文件系统变成可擦写的
    suid
        允许 setuid 和 setgid 位发生作用
    sync
        文件系统的所有 I/O 同步执行
    user
        允许一般非 root 用户 mount 文件系统。这个选项会应用 noexec,nosuid,nodev
        这三个选项（除非在命令行上有指定覆盖这些设定的选项）。
2.安全性
    /etc/fstab 的默认权限是 644,所有者和所有组均为 root
3.相关命令
    mount
    df
    列举计算机当前“可以安装”的文件系统。这非常重要，因为计算机引导时将
    运行 mount -a 命令，该命令负责安装 fstab 的倒数第二列中带有“1”标记
    的每一个文件系统。
```
* /etc/mtools.conf 
```
    DOS 类型的文件系统上所有操作（创建目录、复制、格式化等等）的配置
```
## 10.热插拔
* [udev实现热插拔](https://www.cnblogs.com/linhaostudy/archive/2017/11/12/7820518.html)
>```
> usb（网卡、u盘等）和sd卡
>```
* [udev、mdev和devfs之间的区别](https://blog.csdn.net/wade_510/article/details/72083189)
```
    mdev是udev的简化版本，是busybox中所带的程序，最适合用在嵌入式系统，而udev一般用在PC上的linux中，相对mdev来说要复杂些。
    devfs是2.4内核引入的，而在2.6内核中却被udev所替代，他们有着共同的优点，只是devfs中存在一些未修复的BUG，作者也停止了对他的维护，最显著的一个区别，采用devfs时，当一个并不存在的设备结点时，他却还能自动的加载对应的设备驱动，而udev则不能加载，因为加载浪费了资源。

    dev和mdev是两个使用uevent 机制处理热插拔问题的用户空间程序，两者的实现机理不同。
    udev是基于netlink 机制的，它在系统启动时运行了一个deamon程序udevd，通过监听内核发送的uevent来执行相应的热拔插动作，包括创建/删除设备节点，加载/卸载驱动模块等等。
    mdev是基于uevent_helper机制的，它在系统启动时修改了内核中的uevnet_helper变量（通过写/proc/sys/kernel/hotplug），值为“/sbin/mdev”。这样内核产生uevent时会调用uevent_helper所指的用户级程序，也就是mdev，来执行相应的热拔插动作。udev使用的netlink机制在有大量uevent的场合效率高，适合用在PC 机上。

```
> 思考：热插拔(udev)和文件系统自动挂载(fstab)之间的关系
## 11.[update-alternatives](https://www.jianshu.com/p/4d27fa2dce86)
```
    1.alternatives 的管理目录 /etc/alternatives
    2.查看软件候选项
        $ update-alternatives --display arm-linux-gnueabi-gcc
    3.选择软件候选项
        $ update-alternatives --config arm-linux-gnueabi-gcc
    4.添加一个命令的link值
        $ update-alternatives --install /usr/bin/python python /usr/bin/python2.7 2
    5.删除一个命令的link值
        $ update-alternatives –remove python /usr/bin/python2.7
```
* 语法
```
$ update-alternatives --help
用法：update-alternatives [<选项> ...] <命令>

命令：
  --install <链接> <名称> <路径> <优先级>
    [--slave <链接> <名称> <路径>] ...
                           在系统中加入一组候选项。
  --remove <名称> <路径>   从 <名称> 替换组中去除 <路径> 项。
  --remove-all <名称>      从替换系统中删除 <名称> 替换组。
  --auto <名称>            将 <名称> 的主链接切换到自动模式。
  --display <名称>         显示关于 <名称> 替换组的信息。
  --query <名称>           机器可读版的 --display <名称>.
  --list <名称>            列出 <名称> 替换组中所有的可用候选项。
  --get-selections         列出主要候选项名称以及它们的状态。
  --set-selections         从标准输入中读入候选项的状态。
  --config <名称>          列出 <名称> 替换组中的可选项，并就使用其中哪一个，征询用户的意见。
  --set <名称> <路径>      将 <路径> 设置为 <名称> 的候选项。
  --all                    对所有可选项一一调用 --config 命令。

<链接> 是指向 /etc/alternatives/<名称> 的符号链接。(如 /usr/bin/pager)
<名称> 是该链接替换组的主控名。(如 pager)
<路径> 是候选项目标文件的位置。(如 /usr/bin/less)
<优先级> 是一个整数，在自动模式下，这个数字越高的选项，其优先级也就越高。
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
## 25.[交叉编译工具链](https://blog.csdn.net/guodeqiangde/article/details/78239408)
* **当前使用版本**
```
    gcc:
        系统版本：Ubuntu 14.04.6 LTS
        gcc版本：gcc (Ubuntu 4.8.4-2ubuntu1~14.04.4) 4.8.4
    arm-linux-gcc->arm-linux-gnueabihf-gcc[adlink]:
        系统版本：Ubuntu 14.04.6 LTS
        arm-linux-gcc版本: gcc version 4.9.1 20140710 (prerelease) (crosstool-NG linaro-1.13.1-4.9-2014.07 - Linaro GCC 4.9-2014.07)
        文件：gcc-linaro-arm-linux-gnueabihf-4.9-2014.07_linux
        编译内核：4.1.15-1.0.0
    arm-poky-linux-gnueabi-gcc[advantech]:
        系统版本：Ubuntu 14.04.6 LTS
        arm-poky-linux-gnueabi-gcc版本：gcc version 5.3.0 (GCC)
        文件：fsl-imx-x11-2.1.tar.bz2
        编译内核：4.1.15-2.0.0
    arm-linux-gcc->arm-fsl-linux-gnueabi-gcc[norco]:
        系统版本：Ubuntu 14.04.6 LTS
        arm-fsl-linux-gnueabi-gcc版本：gcc version 4.6.2 20110630 (prerelease) (Freescale MAD -- Linaro 2011.07 -- Built at 2011/08/10 09:20)
        文件：fsl-linaro-toolchain
        编译内核：未开放
```
* **命名规则**
```
交叉编译工具链的命名规则为：arch [-vendor] [-os] [-(gnu)eabi]
* arch – 体系架构，如ARM，MIPS
* vendor – 工具链提供商
* os – 目标操作系统
* eabi – 嵌入式应用二进制接口（Embedded Application Binary Interface）

根据对操作系统的支持与否，ARM GCC可分为支持和不支持操作系统，如
* arm-none-eabi：这个是没有操作系统的，自然不可能支持那些跟操作系统关系密切的函数，比如fork(2)。他使用的是newlib这个专用于嵌入式系统的C库。
* arm-none-linux-eabi：用于Linux的，使用Glibc
```
* **各版本arm-gcc区别**

```
armcc
    ARM 公司推出的编译工具，功能和 arm-none-eabi 类似，可以编译裸机程序（u-boot、kernel），但是不能编译 Linux 应用程序。armcc一般和ARM开发工具一起，Keil MDK、ADS、RVDS和DS-5中的编译器都是armcc，所以 armcc 编译器都是收费的

arm-eabi-gcc
    Android ARM 编译器

arm-none-eabi-gcc
    （ARM architecture，no vendor，not target an operating system，complies with the ARM EABI）
    用于编译 ARM 架构的裸机系统（包括 ARM Linux 的 boot、kernel，不适用编译 Linux 应用 Application），一般适合 ARM7、Cortex-M 和 Cortex-R 内核的芯片使用，所以不支持那些跟操作系统关系密切的函数，比如fork(2)，他使用的是 newlib 这个专用于嵌入式系统的C库

arm-none-linux-gnueabi-gcc
    1.arm-none-linux-gnueabi-gcc是 Codesourcery 公司（目前已经被Mentor收购）基于GCC推出的的ARM交叉编译工具。可用于交叉编译ARM系统中所有环节的代码，包括裸机程序、u-boot、Linux kernel、filesystem和App应用程序
    (ARM architecture, no vendor, creates binaries that run on the Linux operating system, and uses the GNU EABI)

    2.主要用于基于ARM架构的Linux系统，可用于编译 ARM 架构的 u-boot、Linux内核、linux应用等。arm-none-linux-gnueabi基于GCC，使用Glibc库，经过 Codesourcery 公司优化过推出的编译器。arm-none-linux-gnueabi-xxx 交叉编译工具的浮点运算非常优秀。一般ARM9、ARM11、Cortex-A 内核，带有 Linux 操作系统的会用到

arm-none-uclinuxeabi-gcc 和 arm-none-symbianelf-gcc
    arm-none-uclinuxeabi 用于uCLinux，使用Glibc
    arm-none-symbianelf 用于symbian

```
* **ABI和EABI**
```
ABI：二进制应用程序接口(Application Binary Interface (ABI) for the ARM Architecture)。在计算机中，应用二进制接口描述了应用程序（或者其他类型）和操作系统之间或其他应用程序的低级接口。

EABI：嵌入式ABI。嵌入式应用二进制接口指定了文件格式、数据类型、寄存器使用、堆积组织优化和在一个嵌入式软件中的参数的标准约定。开发者使用自己的汇编语言也可以使用 EABI 作为与兼容的编译器生成的汇编语言的接口。

两者主要区别是，ABI是计算机上的，EABI是嵌入式平台上（如ARM，MIPS等)
```
* **arm-linux-gnueabi-gcc 和 arm-linux-gnueabihf-gcc**
```
两个交叉编译器分别适用于 armel 和 armhf 两个不同的架构，armel 和 armhf 这两种架构在对待浮点运算采取了不同的策略（有 fpu 的 arm 才能支持这两种浮点运算策略）。
arm-linux-gnueabi-gcc: 使用hard硬件浮点模式
arm-linux-gnueabi-gcc：使用softfp模式

其实这两个交叉编译器只不过是 gcc 的选项 -mfloat-abi 的默认值不同。gcc 的选项 -mfloat-abi 有三种值 soft、softfp、hard（其中后两者都要求 arm 里有 fpu 浮点运算单元，soft 与后两者是兼容的，但 softfp 和 hard 两种模式互不兼容）：
soft： 不用fpu进行浮点计算，即使有fpu浮点运算单元也不用，而是使用软件模式。
softfp： armel架构（对应的编译器为 arm-linux-gnueabi-gcc ）采用的默认值，用fpu计算，但是传参数用普通寄存器传，这样中断的时候，只需要保存普通寄存器，中断负荷小，但是参数需要转换成浮点的再计算。
hard： armhf架构（对应的编译器 arm-linux-gnueabihf-gcc ）采用的默认值，用fpu计算，传参数也用fpu中的浮点寄存器传，省去了转换，性能最好，但是中断负荷高。
```
* **安装方式**
```
1.Linux解压版：在Linux主机（如Ubuntu、RedHat等）直接解压即可使用。推荐方式！
2.Linux安装版：在Linux主机下执行后按照提示安装后使用。
3.Windows解压版：在Windows系统下解压后使用，但是需要MingW32。
4.Windows安装版：在Windows系统下安装后使用。
5.RPM安装版：RedHat系统安装包，新版本不提供该类安装包。
6.源码版：交叉编译器源代码，一般很少用到
```
* **安装/卸载**
```vim
    //安装arm-linux-gnueabi-gcc或arm-linux-gnueabidf-gcc
    $ sudo apt-get install gcc-arm-linux-gnueabi 
    $ sudo apt-get install gcc-arm-linux-gnueabihf
    //安装arm-linux-gnueabi-g++或arm-linux-gnueabidf-g++
    $ sudo apt-get install g++-arm-linux-gnueabi
    $ sudo apt-get install g++-arm-linux-gnueabihf

    //卸载arm-linux-gnueabi-gcc或arm-linux-gnueabidf-gcc
    $ sudo apt-get remove --purge gcc-arm-linux-gnueabi 
    $ sudo apt-get remove --purge gcc-arm-linux-gnueabihf  
    //卸载arm-linux-gnueabi-g++或arm-linux-gnueabidf-g++
    $ sudo apt-get remove --purge g++-arm-linux-gnueabi
    $ sudo apt-get remove --purge g++-arm-linux-gnueabihf  

注：
    这样安装一般是当前系统支持的最新版本，当然可以根据实际版本需求，进行压缩包解压安装，添加到系统路径即可
```
* **下载地址**
> * [GCC Releases](https://gcc.gnu.org/releases.html)
> * [linaro](https://www.linaro.org)
> * [arm](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads)
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
* **API**
![tcp](./tcp.jpg)  
* **TCP三次握手四次挥手**
![tcp握手](./tcp_handshake.jpg)
```
1.三次握手
    首先Client端发送连接请求报文，Server端接受连接后回复ACK报文，并为这次连接分配资源。
    Client端接收到ACK报文后也向Server端发生ACK报文，并分配资源，这样TCP连接就建立了。

2.数据传输
    两者之间可以传输数据了

3.四次挥手
    注：中断连接端可以是Client端，也可以是Server端。
    假设Client端发起中断连接请求，也就是发送FIN报文。Server端接到FIN报文后，意思是说"我Client端没有数据要发给你了"，
    但是如果你还有数据没有发送完成，则不必急着关闭Socket，可以继续发送数据。所以你先发送ACK，"告诉Client端，你的请求
    我收到了，但是我还没准备好，请继续你等我的消息"。这个时候Client端就进入FIN_WAIT状态，继续等待Server端的FIN报文。
    当Server端确定数据已发送完成，则向Client端发送FIN报文，"告诉Client端，好了，我这边数据发完了，准备好关闭连接了"。
    Client端收到FIN报文后，"就知道可以关闭连接了，但是他还是不相信网络，怕Server端不知道要关闭，所以发送ACK后进入
    TIME_WAIT状态，如果Server端没有收到ACK则可以重传。Server端收到ACK后，"就知道可以断开连接了"。Client端等待了2MSL
    后依然没有收到回复，则证明Server端已正常关闭，那好，我Client端也可以关闭连接了。Ok，TCP连接就这样关闭了！
```
## 32.UDP流程  
![udp](./udp.jpg)  
## 33.Visual Studio 2013
* 使用
```
    1.文件->项目->Vistual C++->空项目
    2.添加头文件
        项目右键->属性->配置属性->C/C++->常规->附加包含目录
    3.添加lib库
        项目右键->属性->配置属性->链接器->常规->附加包含目录
    4.添加源/头文件
        源文件右键->添加->新建项->cpp/h文件
    5.boost库
        boost_1_55_0-msvc-12.0-64.exe
```
## 34.嵌入式框架(总结)
![linux嵌入式框架](./嵌入式框架/linux嵌入式框架.png)  
* [学习linux应用编程](https://blog.csdn.net/yychuyu/article/details/80636496)
```vim
对于Linux应用的学习，主要有六部分：1. 环境搭建；2. 基本操作；3. 系统编程；4. 网络编程；5. 数据库编程，6. Shell编程。下面一一详细介绍。
1. 环境搭建
作为Linux工程师，毋庸置疑一定需要Linux环境。对于Linux环境的获取，我们通常有两种方式：
* 将电脑整体安装为Linux系统；
* 在电脑里安装一个虚拟机，跑Linux电脑；
* Window+Linux双系统。
得到Linux环境后还不够，还要知道如何配置、如何远程连接Linux电脑、如何与Linux电脑互传文件、如何在主机上阅读Linux电脑中的代码，等等。

2. 基本操作
众所周知，Linux很少或几乎没有界面，所有的操作几乎都可以通过命令行来完成。对于运维人员来说，需要掌握相当大量的Linux命令。而对于应用、驱动方向的人员来说，只需掌握一些基本的常用的命令即可。对于这部分很多人建议看 「鸟哥的私房菜」 ，但我觉得这个更适合运维人员，我们无需掌握那么多命令。

3. 系统编程
在学系统编程之前，一定要先学习Makefile，这会为后续的学习提高很大效率。之后的系统编程，主要有几大块：IO编程、进程、线程、进程间通讯（包括管道、信号、信号量、共享内存等）。这几部分学完了，基本也就差不多了。

4. 网络编程
网络编程主要就是socket，poll，epoll，以及对TCP/IP的理解，同时要学会高并发式服务器的编写。进程池，线程池的理解。

5. 数据库编程
数据库的内容其实并不属于Linux，但在项目中经常要用到。这部分主要要学会数据库的基本操作，以及如何写一套接口去操作数据库。

6. Shell编程
Shell是Linux下的脚本语言，功能虽然不如高级语言强大，但它可能做很多事，在某些场合甚至比高级语言要方便得多。当然除了Shell脚本，还有Python脚本。

Linux应用编程书籍推荐：
* UNIX环境高级编程。简称APUE，号称程序员的圣经。它不是一本API字典，它还讲述了很多操作系统的细节，内存，文件系统等方面，是一本难得的好书。但是它起点有点高，不适合初学者。
* Linux程序设计。如果觉得APUE有点难入门的话，可以选择此书进行入门。
* Unix/Linux系统编程手册。这本书号称是一本超越APUE的书，它是一本比较新的书，里面新增了APUE所没有的Linux/Unix新特性。而且对于一些概念性的东西讲的确实比APUE好。但至于能否超载APUE，还有待历史的考验。
* UNIX 网络编程。也是一本非常经典的书，主要是网络编程方向的。简称UNP
* MySQL必知必会。本书在Amazon上长期排在数据库销售榜首，建议想快速了解数据库原理和MySQL的新手阅读。快餐性质，简洁明快，小开本，而且很薄，比较好阅读。
* Linux Shell脚本攻略。这本书很薄很精华，它追求的不是全，而是精，所以用它来入门再适合不过了。

学完以上六部分，基本就有能力完成Linux环境下的应用编程了。当然，在有些场合我们可能还需要用到Python脚本。像我公司的项目部分脚本就是用Python完成的。对于Python的入门，可以参考 「简明Python教程」。但如果想进一步提高的话，那就需要阅读大量书籍了。对于Linux层级的脚本应用，掌握一些基础的足够了。
```
* [学习linux内核](https://blog.csdn.net/xzjj2007/article/details/48438311)
```
* 了解操作系统基本概念  
    如果不会，可以学习《操作系统：设计与实现》Andrew S.Tanenbaum 写的那本。以MINIX为例子讲解操作系统的概念。非常推荐。
* 了解Linux内核机制  
    有了操作系统的基本概念以后，可以了解Linux的机制了。推荐《Linux内核设计与实现》Robert Love 写的。这本书从概念上讲解了Linux有什么，他们是怎么运行的。这本书要反复认真看透。
* 研究Linux内核源码  
    有了Linux内核的了解，还需要具体研究Linux内核源码。最经典的就是《深入理解Linux内核》Daniel P. Bovet 写的。学习这本书的时候，要对着内核代码看着学。这本书学起来相当费力了，那么多多代码要研究。不过这本书如果学明白了，恭喜你，Linux内核你已经很熟悉了。
* 设备驱动方向学习 
    如果要开发设备驱动，可以学习《linux设备驱动程序》O’Reilly出版社的。这本作为驱动的入门是很好的资料。另外还有一本《精通 Linux 驱动程序开发》也是不错的教材，可以参考着看。学习驱动，免不了要学习一些硬件的协议和资料，研究哪个就找到相应的硬件文档，把硬件的工作原 理搞明白。这些就不细说了。
* 网络方向学习  
    网络部分，学些Linux网络部分就学习《深入理解LINUX网络技术内幕》。这本书把Linux的网络部分讲的非常清晰透彻。但是通常不做这方面的工作研究，也不用研究这么深，毕竟现在国内相关职位较少。
* 嵌入式开发 
    现在Linux相关的工作，多集中在一些嵌入式开发领域，arm，mips等，要学习以下这些体系架构的的资料，了解CPU的设计和工作方式。 ARM就看对应的芯片手册，讲的很细致。MIPS就看 《see mips run》，有一二两版，两版内容有些差异，推荐都看。
* 补充一点经验 
    不要认为Linux很庞大，很复杂，就觉的很难学。任何东西认真学下来都是能学会的，看你的恒心和毅力了。另外，不要走弯路，不要看 市面上讲什么Linux0.11的那些书，直接学你要学的东西。就像学C语言看什么谭浩强一样，弯路走了，力气没少花，还严重影响学习效果。
你问的内核，多给你说几句应用编程，有时候经常会需要的： 
    1. 学习Linux应用编程，建议看《unix环境高级编程》，把里面的例子都做一遍，会对整个Linux编程有系统都认识。 
    2. 针对Linux，有本 《Linux系统编程》，学完上一本，这本很快看一遍就懂了。主要是针对Linux具体懂一些内容，讲的挺全了，很实用。 
    3. Linux网络编程，系统的学习一下《unix网络编程.卷1,套接字联网api》，基本上网络应用相关的程序就都没问题了。
```
* [linux学习之路](https://blog.csdn.net/Rock_Ling/article/details/79569995)
* 嵌入式从入门到实战项目完整课程
```
    01 - Linux C语言
    02 - Linux C语言_高级
    03 - 数据结构全攻略
    04 - 嵌入式Linux下文件IO精讲
    05 - Linux并发程序设计你该这么学
    06 - Linux网络编程必修篇
    07 - 嵌入式数据库之Sqlite3
    08 - 在线词典综合实战
    09 - 精通ARM体系结构及接口技术
    10 - 全面掌握嵌入式系统移植 
    11 - 嵌入式内核及驱动开发（初级）
    12 - 嵌入式内核及驱动开发（高级）
    13 - 嵌入式项目实战
    14 - 精通STM32开发
    15 - ZigBee系统开发
    16 - 蓝牙4.0 BLE
    17 - RFID开发与应用
    18 - LoRa开发与应用
    19 - NB-IOT技术实践开发
    20 - WIFI开发与应用
```
## 35.[cgi](https://github.com/boutell/cgic.git)
![cgi](./cgi.png)  
## 36.字节对齐
* [字节对齐背景知识](https://www.cnblogs.com/clover-toeic/p/3853132.html)
```vim
    字节对齐的问题主要就是针对结构体
```
* 对齐准则说法1  
```
    1.数据类型自身的对齐值：char型数据自身对齐值为1字节，short型数据为2字节，int/float型为4字节，double型为8字节。
    2.结构体或类的自身对齐值：其成员中自身对齐值最大的那个值。
    3.指定对齐值：#pragma pack (value)时的指定对齐值value。
    4.数据成员、结构体和类的有效对齐值：自身对齐值和指定对齐值中较小者，即有效对齐值=min{自身对齐值，当前指定的pack值}。
```
* 对齐准则说法2
```
    1.结构体变量的首地址能够被其最宽基本类型成员的大小所整除；
    2.结构体每个成员相对结构体首地址的偏移量(offset)都是成员大小的整数倍，如有需要编译器会在成员之间加上填充字节(internal adding)；
    3.结构体的总大小为结构体最宽基本类型成员大小的整数倍，如有需要编译器会在最末一个成员之后加上填充字节{trailing padding}。
```
* 字节对齐方式
> * 方式1  
> 使用伪指令#pragma pack(n)：C编译器将按照n个字节对齐；  
> 使用伪指令#pragma pack()： 取消自定义字节对齐方式。
```vim
    //按2字节对齐
    #pragma pack(2)  //指定按2字节对齐
    struct C{
        char  b;
        int   a;
        short c;
    };
    #pragma pack()   //取消指定对齐，恢复缺省对齐
```
```vim
    //按8字节对齐
    #pragma pack(8)
    struct D{
        char  b;
        short a;
        char  c;
    };
    #pragma pack()
    //虽然#pragma pack(8)，但依然按照两字节对齐，所以sizeof(struct D)的值为6。因为：对齐到的字节数 = min｛当前指定的pack值，最大成员大小｝
```
> * 方式2(GCC特有语法)  
> \_\_attribute__((aligned (n)))：让所作用的结构成员对齐在n字节自然边界上。如果结构体中有成员的长度大于n，则按照最大成员的长度来对齐  
> \_\_attribute__ ((packed))：取消结构在编译过程中的优化对齐，按照实际占用字节数进行对齐
```vim
    //按1字节对齐(GCC)
    #define GNUC_PACKED __attribute__((packed))
    struct C{
        char  b;
        int   a;
        short c;
    }GNUC_PACKED;
```
## 37.STM32-lwip
```
初始化流程
1.分配内存
	ETH_Mem_Malloc
	lwip_comm_mem_malloc
	lwip_comm_default_ip_set（获取默认ip、netmask、gatway）
2.设置mac
	读st芯片唯一id作为mac地址
	硬复位lan8720
	初始化ETH（配置mac和开启中断及自动协商）
		HAL_ETH_Init
3.初始化lwip
	lwip_init
	转换ip、netmask和gateway
		IP4_ADDR
	网卡名称
	netif_add
		添加网卡
		初始化ip、netmask和gateway及硬件初始化
        ethernetif_init
            etharp_output
            low_level_output
            low_level_init
        ethernet_input     
            接收数据处理
    ethernetif_input[数据接收]
        low_level_input（读取dma传输数据长度和对应的数据内容pbuf）
        ethernet_input接收到数据处理
        HAL_ETH_BuildRxDescriptors 接收下一包数据
	netif_set_default
		设定默认网卡
	netif_set_up
		启动网卡
    DMA传输
    缓存问题
        SCB_CleanInvalidateDCache   
注意事项
	1.没有插网线时上电
	2.连续连接断开（看stm32的tcp收发是否正常）
	3.正常连接，拔网线->插网线
		无心跳时，插拔可正常通信
		心跳（3秒一次，一次poll是0.5秒）
```
## 38.查看/修改系统主频
```
1.cat /proc/cpuinfo
2.cat /sys/devices/system/cpu/cpu0/cpufreq/cpu_cur_freq
   //手动修改主频
   echo userspace > 	//开启用户权限
   echo 792000 > 		//设置主频
```
## 39.存储
```
     emmc
     ssd
     sdio 
     eeprom 
     ufs 
     ddr
```
## 40.ps使用
```
    # 查看某个进程创建的线程
    $ ps -T -p <pid>
```
## 41.websocket
* [RFC 6455 - The WebSocket Protocol](https://blog.csdn.net/u011645307/article/details/76682705)
## 42.ARM体系架构
* [ARM CPU 架构](https://www.cnblogs.com/zhangjiankun/p/4852749.html)
![arm-cpu架构](arm-cpu架构.png)
* [ARM体系架构总结](https://blog.csdn.net/frank_zyp/article/details/84646051)
* [ARM架构处理器全解析](https://blog.csdn.net/FunkyFrog821951259/article/details/79896210)
## 43.进程间通信持续性
* [IPC持续性](https://zhuanlan.zhihu.com/p/37872762)
* [进程间通信](http://www.cppblog.com/tankzhouqiang/archive/2011/07/04/150085.html)
![进程间通信持续性](进程间通信持续性.jpg)
## 44.linux kernel map
![linux-kernel-map](linux-kernel-map.png)
## 45.[环境变量](https://www.freecplus.net/ebfb46a0f8014f59a16c78ec8de73468.html)
* 环境变量含义
```
    程序（操作系统命令和应用程序）的执行都需要运行环境，这个环境是由多个环境变量组成的。
```
* 环境变量的分类
```
1）按生效的范围分类。
系统环境变量：公共的，对全部的用户都生效。
用户环境变量：用户私有的、自定义的个性化设置，只对该用户生效。

2）按生存周期分类。
永久环境变量：在环境变量脚本文件中配置，用户每次登录时会自动执行这些脚本，相当于永久生效。
临时环境变量：使用时在Shell中临时定义，退出Shell后失效。
```
* 常用环境变量
```
1）PATH
可执行程序的搜索目录，可执行程序包括Linux系统命令和用户的应用程序，PATH变量的具体用法本文后面的章节中有详细的介绍。
2）LANG
Linux系统的语言、地区、字符集，LANG变量的具体用法本文后面的章节中有详细的介绍。
3）HOSTNAME
服务器的主机名。
4）SHELL
用户当前使用的Shell解析器。
5）HISTSIZE
保存历史命令的数目。
设置历史命令时间戳
export HISTTIMEFORMAT='%F %T '
6）USER
当前登录用户的用户名。
7）HOME
当前登录用户的主目录。
8）PWD
当前工作目录。
9）LD_LIBRARY_PATH
C/C++语言动态链接库文件搜索的目录，它不是Linux缺省的环境变量，但对C/C++程序员来说非常重要，具体用法本文后面的章节中有详细的介绍。
```
* 系统环境变量
```
系统环境变量对全部的用户生效，设置系统环境变量有三种方法。
1）在/etc/profile文件中设置。
用户登录时执行/etc/profile文件中设置系统的环境变量。但是，Linux不建议在/etc/profile文件中设置系统环境变量。
2）在/etc/profile.d目录中增加环境变量脚本文件，这是Linux推荐的方法。
/etc/profile在每次启动时会执行 /etc/profile.d下全部的脚本文件。/etc/profile.d比/etc/profile好维护，不想要什么变量直接删除 /etc/profile.d下对应的 shell 脚本即可。
3）在/etc/bashrc文件中设置环境变量。
该文件配置的环境变量将会影响全部用户使用的bash shell。但是，Linux也不建议在/etc/bashrc文件中设置系统环境变量。
```
* 用户环境变量
```
用户环境变量只对当前用户生效，设置用户环境变量也有多种方法
1）.bash_profile（推荐首选）
当用户登录时执行，每个用户都可以使用该文件来配置专属于自己的环境变量。
2）.bashrc
当用户登录时以及每次打开新的Shell时该文件都将被读取，不推荐在里面配置用户专用的环境变量，因为每开一个Shell，该文件都会被读取一次，效率肯定受影响
3）.bash_logout
当每次退出系统（退出bash shell）时执行该文件
4）.bash_history
保存了当前用户使用过的历史命令
```
* 环境变量脚本文件的执行顺序
```
/etc/profile->/etc/profile.d->/etc/bashrc->用户的.bash_profile->用户的.bashrc
```
* 环境变量的生效
```
1）在Shell下，用export设置的环境变量对当前Shell立即生效，Shell退出后失效。

2）在脚本文件中设置的环境变量不会立即生效，退出Shell后重新登录时才生效，或者用source命令让它立即生效，例如：
    $ source /etc/profile
```
## 46./etc/syslog
```
syslogd 是一种守护进程，它负责记录（写到磁盘）从其它程序发送到系统
的消息。这个服务尤其常被某些守护进程所使用，这些守护进程不会有另外
的方法来发出可能有问题存在的信号或向用户发送消息。
1.文件格式
/etc/syslog.conf 是 syslog 守护程序的配置文件.syslog 守护程序为记录
来自运行于系统之上的程序的消息提供了一种成熟的客户机 -服务器机制。
syslog 接收来自守护程序或程序的消息，根据优先级和类型将该消息分类，
然后根据由管理员可配置的规则将它写入日志。结果是一个健壮而统一的管
理日志的方法。
这个文件由一条条的规则组成.每条规则应该写在一行内.但是如果某行以
反斜线 \ 结尾的话,他的下个物理行将被认为与此行同属于一行.空白行和
以 # 开始的行将被忽略.
每条规则都是下面这种形式:
facility.priority[;facility.priority .....] action
facility和priority之间用一个英文句点分隔.他们的整体称为selector.
每条规则可以有多个 selector,selector 之间用分号隔开. 而 selector 和
action 之间则用空格或者 tab 隔开.
facility 指定 syslog 功能，主要包括以下这些：
auth 由 pam_pwdb 报告的认证活动。
authpriv 包括特权信息如用户名在内的认证活动
cron 与 cron 和 at 有关的信息。
daemon 与 inetd 守护进程有关的信息。
kern 内核信息，首先通过 klogd 传递。
lpr 与打印服务有关的信息。
mail 与电子邮件有关的信息
mark syslog 内部功能用于生成时间戳
news 来自新闻服务器的信息
syslog 由 syslog 生成的信息
user 由用户程序生成的信息
uucp 由 uucp 生成的信息
local0----local7 与自定义程序使用，例如使用 local5 做为 ssh 功能
* 通配符代表除了 mark 以外的所有功能
priority 指定消息的优先级. 与每个功能对应的优先级是按一定顺序排列
的，emerg 是最高级，其次是 alert，依次类推。缺省时，在
/etc/syslog.conf 记录中指定的级别为该级别和更高级别。如果希望使用
确定的级别可以使用两个运算符号！(不等)和=。
user.=info
表示告知 syslog 接受所有在 info 级别上的 user 功能信息。
可用的 syslog 优先级如下:
emerg 或 panic 该系统不可用
alert 需要立即被修改的条件
crit 阻止某些工具或子系统功能实现的错误条件
err 阻止工具或某些子系统部分功能实现的错误条件
warning 预警信息
notice 具有重要性的普通条件
info 提供信息的消息
debug 不包含函数条件或问题的其他信息
none 没有重要级，通常用于排错
* 所有级别，除了 none
action 字段所表示的活动具有许多灵活性，特别是，可以使用名称管道的作
用是可以使 syslogd 生成后处理信息。
syslog 主要支持以下 action
file
指定文件的绝对路径,如: /var/log/messages . log 信息将写到此文件中
terminal 或 printer
完全的串行或并行设备标志符,如/dev/console . log 信息将送到此设备
@host
远程的日志服务器. log 信息将送到此日志服务器
username
发送信息给指定用户
named pipe
指定使用 mkfifo 命令来创建的 FIFO 文件的绝对路径。
如果对此文件作了改动, 想要使改动生效，您需要向 syslog 守护程序通知
所做的更改。向它发送 SIGHUP 是个正确的办法，您可以用 killall 命令
轻松地做到这一点：
# killall -HUP syslogd
2.安全性
您应该清楚如果 syslogd 写的日志文件还不存在的话，程序将创建它们。
无论您当前的 umask 如何设置，该文件将被创建为可被所有用户读取。如
果您关心安全性，那么您应该用 chmod 命令将该文件设置为仅 root 用户
可读写。此外，可以用适当的许可权配置 logrotate 程序（在下面描述）
以创建新的日志文件。syslog 守护程序始终会保留现有日志文件的当前属
性，因此一旦创建了文件，您就不需要担心它。
3.相关命令
logrotate
klogd
syslogd
dmesg
```
## 47.重要的配置文件
```
启动引导程序配置文件
LILO /etc/lilo.conf
GRUB /boot/grub/menu.lst
系统启动文件核脚本
主启动控制文件 /etc/inittab
SysV 启动脚本的位置 /etc/init.d、/etc/rc.d/init.d 或/etc/rc.d
SysV 启动脚本链接的位置 /etc/init.d/rc?.d、/etc/rc.d/rc?.d 或/etc/rc?.d
本地启动脚本 /etc/rc.d/rc.local、/etc/init.d/boot.local 或/etc/rc.boot 里的文件
网络配置文件
建立网络接口的脚本 /sbin/ifup
保存网络配置数据文件的目录 /etc/network、/etc/sysconfig/network 和
/etc/sysconfig/network-scripts
保存解析 DNS 服务的文件 /etc/resolv.conf
DHCP 客户端的配置文件 /etc/dhclient.conf
超级服务程序配置文件和目录
inetd 配置文件 /etc/inetd.conf
TCP Wrappers 配置文件 /etc/hosts.allow 和/etc/hosts.deny
xinetd 配置文件 /etc/xinetd.conf 和/etc/xinetd.d 目录里的文件
硬件配置
内核模块配置文件 /etc/modules.conf
硬件访问文件
Linux 设备文件 /dev 目录里
保存硬件和驱动程序数据的文件 /proc 目录里
扫描仪配置文件
SANE 主配置 /etc/sane.d/dll.conf
特定扫描仪的配置文件 /etc/sane.d 目录里以扫描仪型号命名的文件
打印机配置文件
BSD LPD 核 LPRng 的本地打印机主配置文件 /etc/printcap
CUPS 本地打印机主配置和远程访问受权文件 /etc/cups/cupsd.conf
BSD LPD 远程访问受权文件 /etc/hosts.lpd
LPRng 远程访问受权文件 /etc/lpd.perms
文件系统
文件系统表 /etc/fstab
软驱装配点 /floppy、/mnt/floppy 或/media/floppy
光驱装配点 /cdrom、/mnt/cdrom 或/media/cdrom
shell 配置文件
bash 系统非登录配置文件 /etc/bashrc、/etc/bash.bashrc 或/etc/bash.bashrc.local
bash 系统登录文件 /etc/profile 和/etc/profile.d 里的文件
bash 用户非登录配置文件 ~/.bashrc
bash 用户登录配置文件 ~/.profile
XFree86 配置文件核目录
XFree86 主配置文件 /etc/XF86config、/etc/X11/XF86Config 或/etc/X11/XF86Config-4
字体服务程序配置文件 /etc/X11/fs/config
Xft 1.x 配置文件 /etcX11/XftConfig
Xft 2.0 配置文件 /etc/fonts/fonts.conf
字体目录 /usr/X11R6/lib/X11/fonts 和/usr/share/fonts
Web 服务程序配置文件
Apache 主配置文件 /etc/apache、/etc/httpd 或/httpd/conf 里的 httpd.conf 或 httpd2.conf 文
件
MIME 类型文件 与 Apache 主配置文件在同一目录里的 mime.types 或 apache-mime.types
文件服务程序配置文件
ProFTPd 配置文件 /etc/proftpd.conf
vsftpd 配置文件 /etc/vsftpd.conf
NFS 服务程序的输出定义文件 /etc/exports
NFS 客户端装配的 NFS 输出 /etc/fstab
Samba 配置文件 /etc/samba/smb.conf
Samba 用户配置文件 /etc/samba/smbpasswd
邮件服务程序配置文件
sendmail 主配置文件 /etc/mail/sendmail.cf
sendmail 源配置文件 /etc/mail/sendmail.mc 或/usr/share/sendmail/cf/cf/linux.smtp.mc 或
其他文件
Postfix 主配置文件 /etc/postfix/main.cf
Exim 主配置文件 /etc/exim/exim.cf
Procmail 配置文件 /etc/procmailrc 或~/.procmailrc
Fetchmail 配置文件 ~/.fetchmailrc
远程登录配置文件
SSH 服务程序配置文件 /etc/ssh/sshd_config
SSH 客户端配置文件 /etc/ssh/ssh_config
XDM 配置文件 /etc/X11/xdm 目录下
GDM 配置文件 /etc/X11/gdm 目录下
VNC 服务程序配置文件 /usr/X11R6/bin/vncserver 启动脚本和~/.vnc 目录里的文件
其他服务程序配置文件
DHCP 服务程序配置文件 /etc/dhcpd.conf
BIND 服务程序配置文件 /etc/named.conf 和/var/named/
NTP 服务程序配置文件 /etc/ntp.conf
```
> ***特别注意：*** Linux中没有一个标准的配置文件格式。在 Linux 中，每个程序员都可以自由选择他或她喜欢的配置文件格式。可以选择的格式很多，从 /etc/shells 文件（它包含被一个换行符分开的 shell 的列表），到 Apache 的复杂的 /etc/httpd.conf 文件。
## 48.Makefile
```
    make -p #查看隐式规则
    gcc -Mm #查看依赖
```
## 49.加密算法
* [加密算法比较](https://blog.csdn.net/chenze666/article/details/79730753)
## 50.linux可重入、异步信号安全和线程安全
* [linux可重入、异步信号安全和线程安全](https://blog.csdn.net/u012317833/article/details/39546377)
```
1.可重入函数
    一个可重入的函数简单来说就是可以被中断的函数，也就是说，可以在这个函数执行的任何时刻中断它，
    转入OS调度下去执行另外一段代码，而返回控制时不会出现什么错误。可重入（reentrant）函数可以
    由多于一个任务并发使用，而不必担心数据错误。相反，不可重入（non-reentrant）函��不能由超过
    一个任务所共享，除非能确保函数的互斥（或者使用信号量，或者在代码的关键部分禁用中断）。
    
    可重入函数可以在任意时刻被中断，稍后再继续运行，不会丢失数据。可重入函数要么使用本地变量，
    要么在使用全局变量时保护自己的数据。

    信号安全，其实也就是异步信号安全，是说线程在信号处理函数当中，不管以任何方式调用你的这个函数
    如果不死锁不修改数据，那就是信号安全的。因此，我认为可重入与异步信号安全是一个概念。

2.线程安全
    一个函数被称为线程安全的，当且仅当被多个并发线程反复的调用时，它会一直产生正确的结果。

    有一类重要的线程安全函数，叫做可重入函数，其特点在于它们具有一种属性：当它们被多个线程
    调用时，不会引用任何共享的数据。

    尽管线程安全和可重入有时会（不正确的）被用做同义词，但是它们之间还是有清晰的技术差别的。
    可重入函数是线程安全函数的一个真子集。

    可重入与线程安全的区别及联系
        可重入函数：重入即表示重复进入，首先它意味着这个函数可以被中断，其次意味着它除了使用
        自己栈上的变量以外不依赖于任何环境（包括static），这样的函数就是可重入，可以允许有该
        函数的多个副本在运行，由于它们使用的是分离的栈，所以不会互相干扰。

        可重入函数是线程安全函数，但是反过来，线程安全函数未必是可重入函数。

    实际上，可重入函数很少，APUE 10.6 节中描述了Single UNIX Specification说明的可重入的函数，只有115个；
    APUE 12.5节中描述了POSIX.1中不能保证线程安全的函数，只有89个。

    信号就像硬件中断一样，会打断正在执行的指令序列。信号处理函数无法判断捕获到信号的时候，进程在何处运行。
    如果信号处理函数中的操作与打断的函数的操作相同，而且这个操作中有静态数据结构等，当信号处理函数返回的
    时候（当然这里讨论的是信号处理函数可以返回），恢复原先的执行序列，可能会导致信号处理函数中的操作覆盖了
    之前正常操作中的数据。

【不可重入的几种情况】
    使用静态数据结构，比如getpwnam，getpwuid
        如果信号发生时正在执行getpwnam，信号处理程序中执行getpwnam可能覆盖原来getpwnam获取的旧值
    调用malloc或free
        如果信号发生时正在malloc（修改堆上存储空间的链接表），信号处理程序又调用malloc，会破坏内核的数据结构
    使用标准IO函数
        因为好多标准IO的实现都使用全局数据结构，比如printf(文件偏移是全局的)
    函数中调用longjmp或siglongjmp
        信号发生时程序正在修改一个数据结构，处理程序返回到另外一处，导致数据被部分更新

    即使对于可重入函数，在信号处理函数中使用也需要注意一个问题就是errno 。一个线程中只有一个errno 变量，信号处理函数中使用的可重入函数也有可能 会修改errno 。例如，read 函数是可重入的，但是它也有可能会修改errno 。因此，正确的做法是在信号处理函数开始，先保存errno；在信号处 理函数退出的时候，再恢复errno。 

    例如，程序正在调用printf 输出，但是在调用printf 时，出现了信号，对应的信号处理函数也有printf 语句，就会导致两个printf 的输出混杂在一起。
    如果是给printf加锁的话，同样是上面的情况就会导致死锁。对于这种情况，采用的方法一般是在特定的区域屏蔽一定的信号。

【屏蔽信号的方法】
    signal(SIGPIPE, SIG_IGN); 
        忽略一些信号
    sigprocmask()
        只为单线程定义的
    pthread_sigmask()
        pthread_sigmasks可以在多线程中使用
    现在看来信号异步安全和可重入的限制似乎是一样的，所以这里把它们等同看待

【特别注意】
    函数名字后面加上"_r" ，以表明这个版本是可重入的（对于线程可重入，也就是说是线程安全的，但并不是说对于信号处理函数
    也是可重入的，或者是异步信号安全的）。

【多线程程序中常见的疏忽性问题】
    1.将指针作为新线程的参数传递给调用方栈。
    2.在没有同步机制保护的情况下访问全局内存的共享可更改状态。
    3.两个线程尝试轮流获取对同一对全局资源的权限时导致死锁。其中一个线程控制第一种资源，另一个线程控制第二种资源。其中一个线程放弃之前，任何一个线程都无法继续操作。
    4.尝试重新获取已持有的锁（递归死锁）。
    5.在同步保护中创建隐藏的间隔。如果受保护的代码段包含的函数释放了同步机制，而又在返回调用方之前重新获取了该同步机制，则将在保护中出现此间隔。结果具有误导性。对于调用方，表面上看全局数据已受到保护，而实际上未受到保护。
    6.将UNIX 信号与线程混合时，使用sigwait(2) 模型来处理异步信号。
    7.调用setjmp(3C) 和longjmp(3C) ，然后长时间跳跃，而不释放互斥锁。
    8.从对*_cond_wait() 或*_cond_timedwait() 的调用中返回后无法重新评估条件。

3.总结
    判断一个函数是不是可重入函数，在于判断其能否可以被打断，打断后恢复运行能够得到正确的结果。（打断执行的指令序列并不改变函数的数据）
    判断一个函数是不是线程安全的，在于判断其能否在多个线程同时执行其指令序列的时候，保证每个线程都能够得到正确的结果。
    如果一个函数对多个线程来说是可重入的，则说这个函数是线程安全的，但这并不能说明对信号处理程序来说该函数也是可重入的。
    如果函数对异步信号处理程序的重入是安全的，那 就可以说函数是"异步-信号安全"的。

    三者的关系
        可重入函数必然是线程安全函数和异步信号安全函数，线程安全函数不一定是可重入函数。

     可重入与线程安全的区别体现在能否在signal处理函数中被调用的问题上，可重入函数在signal处理函数中可以被安全调用，
     因此同时也是Async-Signal-Safe Function；而线程安全函数不保证可以在signal处理函数中被安全调用，如果通过设置
     信号阻塞集合等方法保证一个非可重入函数不被信号中断，那么它也是Async-Signal-Safe Function。

     在C语言中，局部变量是在栈上分配的。因此，任何未使用静态数据或其他共享资源的函数都是线程安全的。

最后让我们来构想一个线程安全但不可重入的函数:
    假设函数func() 在执行过程中需要访问某个共享资源，因此为了实现线程安全，在使用该资源前加锁，在不需要资源解锁。
    假设该函数在某次执行过程中，在已经获得资源锁之后，有异步信号发生，程序的执行流转交给对应的信号处理函数；再假设
    在该信号处理函数中也需要调用函数func()，那么func()在这次执行中仍会在访问共享资源前试图获得资源锁，然而我们知道
    前一个func() 实例已然获得该锁，因此信号处理函数阻塞；另一方面，信号处理函数结束前被信号中断的线程是无法恢复执行
    的，当然也没有释放资源的机会，这样就出现了【线程和信号处理函数之间的死锁局面】。
    因此，func() 尽管通过加锁的方式能保证线程安全，但是由于函数体对共享资源的访问，因此是非可重入。

    可以对整个函数库加锁，但可能会遇到性能瓶颈

```
## 51.缓冲类型
```
三种类型：全缓冲、行缓冲、不带缓冲
总结：
    1.通常磁盘上的文件是全缓冲区的
    2.标准输入和标准输出通常是行缓冲的
    3.指向终端设备的流通常是行缓冲，而指向文件时，则是全缓冲，可使用fflush()进行直接输出
    4.为了尽可能显示错误信息，标准错误是不带缓冲的
```
## 52.日志文件系统
```
    文件系统要解决的一个关键问题是怎样防止掉电或系统崩溃造成数据损坏，在此类意外事件中，导致文件系统损坏的根本原因在于写文件不是原子操作，
    因为写文件涉及的不仅仅是用户数据，还涉及元数据(metadata)包括 Superblock、inode bitmap、inode、data block bitmap等，所以写操作无法
    一步完成，如果其中任何一个步骤被打断，就会造成数据的不一致或损坏。

    举一个简化的例子，我们对一个文件进行写操作，要涉及以下步骤：
        从data block bitmap中分配一个数据块；
        在inode中添加指向数据块的指针；
        把用户数据写入数据块。

    如果步骤2完成了，3未完成，结果是数据损坏，因为该文件认为数据块是自己的，但里面的数据其实是垃圾；
    如果步骤2完成了，1未完成，结果是元数据不一致，因为该文件已经把数据块据为己有，然而文件系统却还认为该数据块未分配、随后又可能会把该数据块
    分配给别的文件、造成数据覆盖；
    如果步骤1完成了、2未完成，结果就是文件系统分配了一个数据块，但是没有任何文件用到这个数据块，造成空间浪费；
    如果步骤3完成了，2未完成，结果就是用户数据写入了硬盘数据块中，但白写了，因为文件不知道这个数据块是自己的。

日志文件系统(Journal File System)就是为解决上述问题而诞生的。

    它的原理是在进行写操作之前，把即将进行的各个步骤（称为transaction）事先记录下来，保存在文件系统上单独开辟的一块空间上，这就是所谓的日志(journal)
    ，也被称为write-ahead logging，日志保存成功之后才进行真正的写操作、把文件系统的元数据和用户数据写进硬盘（称为checkpoint），这样万一写操作的过程
    中掉电，下次挂载文件系统之前把保存好的日志重新执行一遍就行了（术语叫做replay），避免了前述的数据损坏场景。

    有人问如果保存日志的过程中掉电怎么办？最初始的想法是把一条日志的数据一次性写入硬盘，相当于一个原子操作，然而这并不可行，因为硬盘通常以512字节为单
    位进行操作，日志数据一超过512字节就不可能一次性写入了。所以实际上是这么做的：给每一条日志设置一个结束符，只有在日志写入成功之后才写结束符，如果一
    条日志没有对应的结束符就会被视为无效日志，直接丢弃，这样就保证了日志里的数据是完整的。

    一条日志在它对应的写操作完成之后就没用了，占用的硬盘空间就可以释放。保存日志的硬盘空间大小是有限的，被循环使用，所以日志也被称为circular log。

至此可以总结一下日志文件系统的工作步骤了：
    Journal commit : 在一条日志保存好之后，写入结束符；
    Checkpoint : 进行真正的写操作，把元数据(metadata)和用户数据(user data)写入文件系统；
    Free : 回收日志占用的硬盘空间。
    
    以上方式把用户数据(user data)也记录在日志中，称为Data Journaling，Linux EXT3文件系统就支持这种方式，这种方式存在效率问题：
    就是每一个写操作涉及的元数据(metadata)和用户数据(user data)实际上都要在硬盘上写两次，一次写在日志里，一次写在文件系统上。元
    数据倒也罢了，用户数据通常比较大，拷贝几个GB的电影文件也要乘以2实在是降低了效率。

    一个更高效的方式是Metadata Journaling，不把用户数据(user data)记录在日志中，它防止数据损坏的方法是先写入用户数据(user data)、
    再写日志，即在上述”Journal write”之前先写用户数据，这样就保证了只要日志是有效的，那么它对应的用户数据也是有效的，一旦发生掉电故
    障，最坏的结果也就是最后一条日志没记完，那么对应的用户数据也会丢，效果与Data Journaling丢弃日志一样，重要的是文件系统的一致性和
    完整性是有保证的。
    
    Metadata Journaling又叫Ordered Journaling，大多数文件系统都采用这种方式。像Linux EXT3文件系统也是可以选择Data Journaling
    还是Ordered Journaling的。
```
## 53.[5万字、97 张图总结操作系统核心知识点](https://blog.csdn.net/qq_36894974/article/details/107330720)
### 1.目的
```
    学习操作系统我们能够有效的解决并发问题，并发几乎是互联网的重中之重了，这也从侧面说明了学习操作系统的重要性。

    学习操作系统的重点不是让你从头制造一个操作系统，而是告诉你「操作系统是如何工作的」，能够让你对计算机底层有所
    了解，打实你的基础。

    编程：「Data structures + Algorithms = Programming」
```
### 2.CPU
```
    运算器和控制器共同组成了CPU
```   
### 3.进程和线程相关内容
![进程和线程](./进程和线程相关内容.jpg)
### 4.进程
```
    进程创建
        * UNIX中，仅有一个系统调用来创建一个新的进程，这个系统调用就是 fork。这个调用会创建一个与调用进程相关的副本。
        在 fork 后，一个父进程和子进程会有相同的内存映像，相同的环境字符串和相同的打开文件。
        * 在 Windows 中，情况正相反，一个简单的 Win32 功能调用 CreateProcess，会处理流程创建并将正确的程序加载到新
        的进程中。这个调用会有 10 个参数，包括了需要执行的程序、输入给程序的命令行参数、各种安全属性、有关打开的文
        件是否继承控制位、优先级信息、进程所需要创建的窗口规格以及指向一个结构的指针，在该结构中新创建进程的信息被
        返回给调用者。「在 Windows 中，从一开始父进程的地址空间和子进程的地址空间就是不同的」。
    进程终止
        * 正常退出(自愿的) ：多数进程是由于完成了工作而终止。当编译器完成了所给定程序的编译之后，编译器会执行一个系统
        调用告诉操作系统它完成了工作。这个调用在 UNIX 中是 exit ，在 Windows 中是 ExitProcess。
        * 错误退出(自愿的)：比如执行一条不存在的命令，于是编译器就会提醒并退出。
        * 严重错误(非自愿的)
        * 被其他进程杀死(非自愿的) ：某个进程执行系统调用告诉操作系统杀死某个进程。在 UNIX 中，这个系统调用是 kill。
        在 Win32 中对应的函数是 TerminateProcess（注意不是系统调用）。
    进程层次结构
        * 在 UNIX 中，进程和它的所有子进程以及子进程的子进程共同组成一个进程组。当用户从键盘中发出一个信号后，该信号被
        发送给当前与键盘相关的进程组中的所有成员（它们通常是在当前窗口创建的所有活动进程）。每个进程可以分别捕获该信
        号、忽略该信号或采取默认的动作，即被信号 kill 掉。整个操作系统中所有的进程都隶属于一个单个以 init 为根的进程
        树。
        * 相反，Windows 中没有进程层次的概念，Windows 中所有进程都是平等的，唯一类似于层次结构的是在创建进程的时候，
        父进程得到一个特别的令牌（称为句柄），该句柄可以用来控制子进程。然而，这个令牌可能也会移交给别的操作系统，这样
        就不存在层次结构了。而在 UNIX 中，进程不能剥夺其子进程的进程权。（这样看来，还是 Windows 比较渣）。
    进程状态（内核中切换）
        运行态，运行态指的就是进程实际占用 CPU 时间片运行时
        就绪态，就绪态指的是可运行，但因为其他进程占用CPU正在运行而处于就绪状态
        阻塞态，除非某种外部事件发生，否则进程不能运行，此时CPU被占用，资源未准备好
    进程实现
        操作系统为了执行进程间的切换，会维护着一张表，这张表就是进程表(process table)。每个进程占用一个进程表项。该
        表项包含了进程状态的重要信息，包括程序计数器、堆栈指针、内存分配状况、所打开文件的状态、账号和调度信息，以及其
        他在进程由运行态转换到就绪态或阻塞态时所必须保存的信息
```
### 5.线程
```
    线程使用
        * 多线程之间会共享同一块地址空间和所有可用数据的能力，这是进程所不具备的
        * 线程要比进程更轻量级，由于线程更轻，所以它比进程更容易创建，也更容易撤销。在许多系统中，创建一个线程要比创建
        一个进程快 10 - 100 倍。
        *第三个原因可能是性能方面的探讨，如果多个线程都是 CPU密集型(CPU-bound)的，那么并不能获得性能上的增强，但是如果
        存在着大量的计算和大量的 I/O 处理，拥有多个线程能在这些活动中彼此重叠进行，从而会加快应用程序的执行速度
    线程模型
        * 线程不像是进程那样具备较强的独立性。同一个进程中的所有线程都会有完全一样的地址空间，这意味着它们也共享同样的全
        局变量。由于每个线程都可以访问进程地址空间内每个内存地址，「因此一个线程可以读取、写入甚至擦除另一个线程的堆栈」。
        * 线程之间的状态转换和进程之间的状态转换是一样的，每个线程都会有自己的堆栈
    线程系统调用
        * 线程创建 thread_create
        * 线程退出 thread_exit
        * 线程回收 thread_jion
        * 线程调度 thread_yield
    POSIX线程
        POSIX线程通常称为 pthreads是一种独立于语言而存在的执行模型，以及并行执行模型。
        pthread_create	创建一个新线程
        pthread_exit	结束调用的线程
        pthread_join	等待一个特定的线程退出
        pthread_yield	释放 CPU 来运行另外一个线程
        pthread_attr_init	创建并初始化一个线程的属性结构
        pthread_attr_destory	删除一个线程的属性结构
    线程实现
    主要有三种实现方式
        * 在用户空间中实现线程；
            把整个线程包放在用户空间中，内核对线程一无所知，它不知道线程的存在。所有的这类实现都有同样的通用结构
            线程在运行时系统之上运行，运行时系统是管理线程过程的集合，包括前面提到的四个过程：pthread_create, 
            pthread_exit, pthread_join 和 pthread_yield。
        * 在内核空间中实现线程；
            当某个线程希望创建一个新线程或撤销一个已有线程时，它会进行一个系统调用，这个系统调用通过对线程表的更新
            来完成线程创建或销毁工作。
            内核中的线程表持有每个线程的寄存器、状态和其他信息。这些信息和用户空间中的线程信息相同，但是位置却被放
            在了内核中而不是用户空间中。另外，内核还维护了一张进程表用来跟踪系统状态。
            所有能够阻塞的调用都会通过系统调用的方式来实现，当一个线程阻塞时，内核可以进行选择，是运行在同一个进程
            中的另一个线程（如果有就绪线程的话）还是运行一个另一个进程中的线程。但是在用户实现中，运行时系统始终运
            行自己的线程，直到内核剥夺它的 CPU 时间片（或者没有可运行的线程存在了）为止。
        * 在用户和内核空间中混合实现线程。
            结合用户空间和内核空间的优点，设计人员采用了一种内核级线程的方式，然后将用户级线程与某些或者全部内核线
            程多路复用起来
            在这种模型中，编程人员可以自由控制用户线程和内核线程的数量，具有很大的灵活度。采用这种方法，内核只识别
            内核级线程，并对其进行调度。其中一些内核级线程会被多个用户级线程多路复用。
```
### 6.进程间通信
```
    进程是需要频繁的和其他进程进行交流的。下面我们会一起讨论有关 进程间通信(Inter Process Communication, IPC) 
    的问题，大致来说，进程间的通信机制可以分为 6 种：信号(signal)、管道(pipe)、共享内存(shared memory)、
    命名管道/先入先出队列(FIFO)、消息队列(message queue)和套接字(socket)。
    5.1信号(signal)
        * 信号是 UNIX 系统最先开始使用的进程间通信机制，因为 Linux 是继承于 UNIX 的，所以 Linux 也支持信号机制，通过
        向一个或多个进程发送异步事件信号来实现，信号可以从键盘或者访问不存在的位置等地方产生；信号通过 shell 将任务
        发送给子进程。可以在 Linux 系统上输入 kill -l 来列出系统使用的信号。
        * 进程可以选择忽略发送过来的信号，但是有两个是不能忽略的：SIGSTOP 和 SIGKILL 信号。
        SIGSTOP 信号会通知当前正在运行的进程执行关闭操作。
        SIGKILL 信号会通知当前进程应该被杀死。
        除此之外，进程可以选择它想要处理的信号，进程也可以选择阻止信号，如果不阻止，可以选择自行处理，也可以选择进行内核处理。
        如果选择交给内核进行处理，那么就执行默认处理。
        * 操作系统会中断目标程序的进程来向其发送信号、在任何非原子指令中，执行都可以中断，如果进程已经注册了信号处理程序，那么
        就执行进程，如果没有注册，将采用默认处理的方式。
        * 进程检查是否收到信号的时机是：
            一个进程在即将从内核态返回到用户态时；
            或者，在一个进程要进入或离开一个适当的低调度优先级睡眠状态时。   
            内核处理一个进程收到的信号的时机是在一个进程从内核态返回用户态时。
    5.2管道(pipe)
        Linux 系统中的进程可以通过建立管道 pipe 进行通信，只能用于亲缘进程间通信，类似打开文件一样
        读写一端关闭时，会出现什么情况？
    5.3共享内存(shared memory)
        * 两个进程之间还可以通过共享内存进行进程间通信，其中两个或者多个进程可以访问公共内存空间。两个进程的共享工作是通过共享内存
        完成的，一个进程所作的修改可以对另一个进程可见(很像线程间的通信)。
        * 使用流程
            创建共享内存段或者使用已创建的共享内存段(shmget())
            将进程附加到已经创建的内存段中(shmat())
            从已连接的共享内存段分离进程(shmdt())
            对共享内存段执行控制操作(shmctl())
    5.4命名管道/先入先出队列(FIFO)
        * 先入先出队列 FIFO 通常被称为 命名管道(Named Pipes)，命名管道的工作方式与常规管道非常相似，但是确实有一些明显的区别。
        未命名的管道没有备份文件：操作系统负责维护内存中的缓冲区，用来将字节从写入器传输到读取器。一旦写入或者输出终止的话，缓冲
        区将被回收，传输的数据会丢失。相比之下，命名管道具有支持文件和独特API，命名管道在文件系统中作为设备的专用文件存在。当所
        有的进程通信完成后，命名管道将保留在文件系统中以备后用。
        * 命名管道具有严格的 FIFO 行为，写入的第一个字节是读取的第一个字节，写入的第二个字节是读取的第二个字节，依此类推。
        * 阻塞模式如果一端关闭了管道,而另一端向管道中写或者读都会出现问题。
            read返回0(文件结束符)，write给线程产生SIGPIPE信号(默认行为是终止进程)
        非阻塞模式的FIFO行为模式发生了改变。
            当我们以只读方式打开FIFO时，描述符均会成功返回。
            当我们以只写方式打开FIFO时，如果FIFO没有读端则会返回一个ENXIO错误而不会阻塞。
            当我们read一个空FIFO时，如果此时FIFO有写端则会返回EAGAIN，如果没有则返回0。
            当我们write一个FIFO时，没有读端则产生一个SIGPIPE信号。
        * FIFO的长度是否可以设置？
    5.5消息队列(message queue)
        消息队列是用来描述内核寻址空间内的内部链接列表。可以按几种不同的方式将消息按顺序发送到队列并从队列中检索消息。
        每个消息队列由 IPC 标识符唯一标识。消息队列有两种模式，一种是[严格模式]，严格模式就像是FIFO先入先出队列似的，消息顺序发送，顺序读取。
        还有一种模式是[非严格模式]，消息的顺序性不是非常重要。
    5.6套接字(socket)
        * socket 提供端到端的双向通信。一个套接字可以与一个或多个进程关联。就像管道有命令管道和未命名管道一样。
        套接字也有两种模式，套接字一般用于两个进程之间的网络通信，网络套接字需要来自诸如TCP（传输控制协议）或较低级别UDP（用户数据报协议）等
        基础协议的支持。
        * 套接字种类
            * 顺序包套接字(Sequential Packet Socket)：此类套接字为最大长度固定的数据报提供可靠的连接。此连接是双向的并且是顺序的。
            * 数据报套接字(Datagram Socket)：数据包套接字支持双向数据流。数据包套接字接受消息的顺序与发送者可能不同。
            * 流式套接字(Stream Socket)：流套接字的工作方式类似于电话对话，提供双向可靠的数据流。
            * 原始套接字(Raw Socket)：可以使用原始套接字访问基础通信协议。
```
### 7.调度（如何查看当前系统使用的是哪一种调度算法）
```
        当一个计算机是多道程序设计系统时，会频繁的有很多进程或者线程来同时竞争 CPU 时间片。当两个或两个以上的进程/线程处于就绪状态时，
        就会发生这种情况。如果只有一个 CPU 可用，那么必须选择接下来哪个进程/线程可以运行。操作系统中有一个叫做 调度程序(scheduler) 的
        角色存在，它就是做这件事儿的，该程序使用的算法叫做 调度算法(scheduling algorithm) 。
    6.1调度算法分类
        毫无疑问，不同的环境下需要不同的调度算法。之所以出现这种情况，是因为不同的应用程序和不同的操作系统有不同的目标。也就是说，在不同
        的系统中，调度程序的优化也是不同的。这里有必要划分出三种环境
        * 批处理(Batch): 商业领域
        * 交互式(Interactive): 交互式用户环境
        * 实时(Real time)
    批处理中的调度
        * 先来先服务(first-come,first-serverd)
            ** 最简单的【非抢占式】调度算法的设计就是 先来先服务(first-come,first-serverd)。当第一个任务从外部进入系统时，将会立即启动并允许运行
            任意长的时间。它不会因为运行时间太长而中断。当其他作业进入时，它们排到就绪队列尾部。当正在运行的进程阻塞，处于等待队列的第一个进
            程就开始运行。当一个阻塞的进程重新处于就绪态时，它会像一个新到达的任务，会排在队列的末尾，即排在所有进程最后。
            ** 这个算法的强大之处在于易于理解和编程，在这个算法中，一个单链表记录了所有就绪进程。要选取一个进程运行，只要从该队列的头部移走一个
            进程即可；要添加一个新的作业或者阻塞一个进程，只要把这个作业或进程附加在队列的末尾即可。这是很简单的一种实现。
        * 最短作业优先(Shortest Job First)
            ** 批处理中，第二种调度算法是 最短作业优先(Shortest Job First)，我们假设运行时间已知。例如，一家保险公司，因为每天要做类似的工作，
            所以人们可以相当精确地预测处理 1000 个索赔的一批作业需要多长时间。当输入队列中有若干个同等重要的作业被启动时，调度程序应使用最短优先
            作业算法。
            ** 需要注意的是，在所有的进程都可以运行的情况下，最短作业优先的算法才是最优的。
        * 最短剩余时间优先(Shortest Remaining Time Next)
            ** 最短作业优先的【抢占式】版本被称作为最短剩余时间优先(Shortest Remaining Time Next) 算法。使用这个算法，调度程序总是选择剩余运行时间最短的那个进程运行。    
    交互式系统中的调度
        * 轮询调度（round-robin）
            ** 一种最古老、最简单、最公平并且最广泛使用的算法就是轮询算法(round-robin)，小插曲：round robin 来源于法语ruban rond（round ribbon），意思是环形丝带。每个进程都会被分配一个时间段，称为时间片(quantum)，在这个时间片内允许进程运行。如果时间片结束时进程还在运行的话，则抢
            占一个 CPU 并将其分配给另一个进程。如果进程在时间片结束前阻塞或结束，则 CPU 立即进行切换。轮询算法比较容易实现。调度程序所做的就是维护一
            个可运行进程的列表，当一个进程用完时间片后就被移到队列的末尾。
        * 优先级调度(priority scheduling)
            ** 轮询调度假设了所有的进程是同等重要的。但事实情况可能不是这样。例如，在一所大学中的等级制度，首先是院长，然后是教授、秘书、后勤人员，最
            后是学生。这种将外部情况考虑在内就实现了优先级调度(priority scheduling)。它的基本思想很明确，每个进程都被赋予一个优先级，优先级高的进程
            优先运行。
        * 多级队列（时间片）
            ** 最早使用优先级调度的系统是 CTSS(Compatible TimeSharing System)。CTSS 在每次切换前都需要将当前进程换出到磁盘，并从磁盘上读入一个新进程。为 CPU 密集型进程设置较长的时间片比频繁地分给他们很短的时间要更有效（减少交换次数）。另一方面，如前所述，长时间片的进程又会影响到响应时间，解决办法是设置【优先级类。】属于最高优先级的进程运行一个时间片，次高优先级进程运行 2 个时间片，再下面一级运行 4 个时间片，以此
            类推。当一个进程用完分配的时间片后，它被移到下一类。
        * 最短进程优先
            ** 最短进程优先是根据进程过去的行为进行推测，并执行估计运行时间最短的那一个。
        * 保证调度
            ** 一种完全不同的调度方法是对用户做出明确的性能保证。一种实际而且容易实现的保证是：若用户工作时有 n 个用户登录，则每个用户将获得 CPU 处
            理能力的 1/n。类似地，在一个有 n 个进程运行的单用户系统中，若所有的进程都等价，则每个进程将获得 1/n 的 CPU 时间。
        * 彩票调度
            ** 对用户进行承诺并在随后兑现承诺是一件好事，不过很难实现。但是存在着一种简单的方式，有一种既可以给出预测结果而又有一种比较简单的实现方式
            的算法，就是 彩票调度(lottery scheduling)算法。
            ** 其基本思想是为进程提供各种系统资源（例如 CPU 时间）的彩票。当做出一个调度决策的时候，就随机抽出一张彩票，拥有彩票的进程将获得该资源。
            在应用到 CPU 调度时，系统可以每秒持有 50 次抽奖，每个中奖者将获得比如 20 毫秒的 CPU 时间作为奖励。
        * 公平分享调度（多用户）
            ** 到目前为止，我们假设被调度的都是各个进程自身，而不用考虑该进程的拥有者是谁。结果是，如果用户 1 启动了 9 个进程，而用户 2 启动了一个进
            程，使用轮转或相同优先级调度算法，那么用户 1 将得到 90 % 的 CPU 时间，而用户 2 将之得到 10 % 的 CPU 时间。
            ** 为了阻止这种情况的出现，一些系统在调度前会把进程的拥有者考虑在内。在这种模型下，每个用户都会分配一些CPU 时间，而调度程序会选择进程并
            强制执行。因此如果两个用户每个都会有 50% 的 CPU 时间片保证，那么无论一个用户有多少个进程，都将获得相同的 CPU 份额。
    实时系统中的调度
        ** 实时系统(real-time) 是一个时间扮演了重要作用的系统。实时系统可以分为两类，硬实时(hard real time) 和 软实时(soft real time) 系统，前者意味着必须要满足绝对的截止时间；后者的含义是虽然不希望偶尔错失截止时间，但是可以容忍。
        ** 实时系统中的事件可以按照响应方式进一步分类为周期性(以规则的时间间隔发生)事件或 非周期性(发生时间不可预知)事件。一个系统可能要响应多个周期
        性事件流，根据每个事件处理所需的时间，可能甚至无法处理所有事件。例如，如果有 m 个周期事件，事件 i 以周期 Pi 发生，并需要 Ci 秒 CPU 时间处理
        一个事件，需满足可以处理负载的条件。
```
### 8.内存管理
![内存管理](./内存管理相关内容.png)
#### [重定位与位置无关码](https://blog.csdn.net/cherisegege/article/details/80708143)
```
    1.如果要使多个应用程序同时运行在内存中，必须要解决两个问题：保护和重定位。
        ** 第一种解决方式是用保护密钥标记内存块，并将执行过程的密钥与提取的每个存储字的密钥进行比较。这种方式只能解决第一种问题（破坏操作系统），但是不能解决多进程在内存中同时运行的问题。
        ** 还有一种更好的方式是创造一个存储器抽象：地址空间(the address space)。就像进程的概念创建了一种抽象的 CPU 来运行程序，地址空间也创建了一种抽象内存供程序使用。
   
    基址寄存器和变址寄存器
        ** 最简单的办法是使用动态重定位(dynamic relocation)技术，它就是通过一种简单的方式将每个进程的地址空间映射到物理内存的不同区域。
        ** 还有一种方式是使用基址寄存器和变址寄存器。
            基址寄存器：存储数据内存的起始位置
            变址寄存器：存储应用程序的长度
            每当进程引用内存以获取指令或读取、写入数据时，CPU 都会自动将基址值添加到进程生成的地址中，然后再将其发送到
            内存总线上。同时，它检查程序提供的地址是否大于或等于变址寄存器 中的值。如果程序提供的地址要超过变址寄存器的
            范围，那么会产生错误并中止访问。
    
    2.交换技术
        在程序运行过程中，经常会出现内存不足的问题。
        针对上面内存不足的问题，提出了两种处理方式：
        ** 最简单的一种方式就是交换(swapping)技术，即把一个进程完整的调入内存，然后在内存中运行一段时间，再把它放回磁盘。空闲进程会存储在磁盘中，所以这些进程在没有运行时不会占用太多内存。
        ** 另外一种策略叫做虚拟内存(virtual memory)，虚拟内存技术能够允许应用程序部分的运行在内存中。
    2.1 swapping
        交换过程
            * 在装载过程中需要被重新定位，或者在交换程序时通过软件来执行；或者在程序执行期间通过硬件来重定位。基址寄存器和变址寄存器就适用于这种情况。
            * 交换在内存创建了多个空闲区(hole)，内存会把所有的空闲区尽可能向下移动合并成为一个大的空闲区。这项技术称为
            内存紧缩(memory compaction)。但是这项技术通常不会使用，因为这项技术会消耗很多 CPU 时间。
        空闲内存管理
            * 在进行内存动态分配时，操作系统必须对其进行管理。大致上说，有两种监控内存使用的方式
                位图(bitmap)
                空闲列表(free lists)
            * 使用位图的存储管理使用位图方法时，内存可能被划分为小到几个字或大到几千字节的分配单元。每个分配单元对应于
            位图中的一位，0 表示空闲， 1 表示占用（或者相反）。
            位图提供了一种简单的方法在固定大小的内存中跟踪内存的使用情况，因为「位图的大小取决于内存和分配单元的大
            小」。这种方法有一个问题是，当决定为把具有 k 个分配单元的进程放入内存时，内容管理器(memory manager) 必须
            搜索位图，在位图中找出能够运行 k 个连续 0 位的串。在位图中找出指定长度的连续 0 串是一个很耗时的操作，这是
            位图的缺点。（可以简单理解为在杂乱无章的数组中，找出具有一大长串空闲的数组单元）
            * 维护一个记录已分配内存段和空闲内存段的链表，段会包含进程或者是两个进程的空闲区域。可用上面的图 c 「来表示内存的使用情况」。链表中的每一项都可以代表一个 空闲区(H) 或者是进程(P)的起始标志，长度和下一个链表项的位置。



```
## 54.[进程状态](https://blog.csdn.net/sdkdlwk/article/details/65938204)
```
    Linux进程状态解析 之 R、S、D、T、Z、X (主要有三个状态)
    此状态通过ps可查
```
## 55.[Linux I/O 调度算法](https://www.cnblogs.com/reaperhero/p/10261625.html)
```
    Linux I/O 调度算法，IO调度器的总体目标是希望让磁头能够总是往一个方向移动,移动到底了再往反方向走,这恰恰就是现实生活中的电梯模型,所以IO调度器也被叫做电梯 (elevator)。而相应的算法也就被叫做电梯算法。而Linux中IO调度的电梯算法有好几种,一个叫做as(Anticipatory),一个叫做 cfq(Complete Fairness Queueing),一个叫做deadline,还有一个叫做noop(No Operation).具体使用哪种算法我们可以在启动的时候通过内核参数elevator来指定。

    * 查看当前系统支持的IO调度算法
        # dmesg | grep -i scheduler
    * 查看当前系统的I/O调度方法
        # cat /sys/block/sda/queue/scheduler
    
```
## 56.repo
* [repo安装](https://blog.csdn.net/qq_33160790/article/details/78193314)
```vim
    $ git clone https://git.dev.tencent.com/codebug8/repo.git # repo下载，可以去其它git仓库也可以
```
* [repo使用](https://blog.csdn.net/nwpushuai/article/details/78778602)
