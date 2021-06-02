# stm32_debug_notes
## 1.编译后内存分析
```
Program Size: Code=50924 RO-data=9044 RW-data=7600 ZI-data=39008  

map文件：
    Total RO  Size (Code + RO Data)                59968 (  58.56kB)
    Total RW  Size (RW Data + ZI Data)             46608 (  45.52kB)
    Total ROM Size (Code + RO Data + RW Data)      61260 (  59.82kB)

1.只读长度RO size: Code + RO-data
2.读写长度RW size: RW-data + ZI-data
3.需要存储的最小空间ROM （minimum）size = Code + RO-data + RW-data （即烧/下载程序到FLASH/ROM时，所占用的最小空间）
4.总的存储空间Total ROM Size (Code + RO-Data + RW-Data)这样所写的程序占用的ROM的字节总数，也就是说程序所下载到ROM flash 中的大小。
为什么Rom中还要存RW，因为掉电后RAM中所有数据都丢失了，每次上电RAM中的数据是被重新赋值的，每次这些固定的值就是存储在Rom中的。
为什么不包含ZI段呢，是因为ZI数据都是0，没必要包含，只要程序运行之前将ZI数据所在的区域一律清零即可。包含进去反而浪费存储空间。
5.RAM size: RW-Data + ZI-Data (即程序运行的时，RAM使用的空间)。
这里需要注意，并不一定真的需要这么大的空间，因为子函数中的局部变量只有在函数被调用时才会创建，子函数不会被同时全部调用，所以即使RW-Data + ZI-Data 超过RAM的实际大小也不会有问题。
估算下最多会有多少子函数同时被调用，就可以计算出实际使用的最大RAM。
```
## 2.堆栈
```
堆区（heap）
    一般由程序员使用malloc或new来进行分配，在适当的时候用free或delete来进行释放。若程序员不释放，程序结束时可能由操作系统回收。分配方式类似于数据结构中的链表。
栈区（stack）
    由编译器自动分配和释放，程序员不做干涉。存放函数的参数值、局部变量的值等，其操作方式类似于数据结构中的栈。程序的中断，函数的形式参数传递等都需要STACK来实现。

注意：
    1.堆栈的大小在编译器编译之后是不知道的，只有运行的时候才知道，所以需要注意一点，就是别造成堆栈溢出了，不然就会发生hardfault错误。
    2.所有在处理的函数，包括函数嵌套，递归，等等，都是从这个“栈”里面，来分配的。所以，如果栈大小为2K，一个函数的局部变量过多，比如在函数里面定义一个u8 buf[512]，这一下就占了1/4的栈大小了，再在其他函数里面来搞两下，程序崩溃是很容易的事情，这时候,一般你会进入到hardfault….这是初学者非常容易犯的一个错误.切记不要在函数里面放N多局部变量,尤其有大数组的时候!
    3.STM32的栈，是向下生长的。事实上，一般CPU的栈增长方向，都是向下的。而堆的生长方向，都是向上的。堆和栈，只是他们各自的起始地址和增长方向不同，他们没有一个固定的界限，所以一旦堆栈冲突，系统就到了崩溃的时候了。
    4.程序中的常量，如果没加const也会编译到SRAM里，加了const会被编译到flash中。    
```
## 3.lwip
```
    * udp通信[2021-03-05]
    stm32+lwip[udp]
    1.使用工具与stm32建立udp连接，快速发送[10ms/包，5000字节包数据]，stm32会出现hardfault或者网络ping不通现象
    2.进入hardfault
    大概原因是lwipopts.h配置不对，小数据和大数据配置是有区别的？
    快速收发(1ms一包数据)

    * 发送数据太快
        解决方式？
    * 发送数据太大
        解决方式？
    * 发送数据太快太大
        解决方式？

    3.ping不通
	首先明确ping是icmp协议，是mcu处理的，不是phy芯片[lan8720]
	* 确认ip，netmask和gateway
	* 确认是否进入接收中断
	* 确认是否进入ethernetif_input
	* 确认是否进入low_level_input
	* 确认io配置[因为是100M网，所以速度要匹配]
	 	GPIO_SPEED_FREQ_HIGH和GPIO_SPEED_FREQ_VERY_HIGH
	* 确认是否开启SCB_EnableDCache
	* 确认是否清楚无效缓存，low_level_input和low_level_output
		SCB_CleanInvalidateDCache()和
		SCB_InvalidateDCache_by_Addr()    

    4.开启插拔网线监测线程和状态回调函数
	/*----- Default Value for LWIP_NETIF_STATUS_CALLBACK: 0 ---*/
	#define LWIP_NETIF_STATUS_CALLBACK 1
	/*----- Default Value for LWIP_NETIF_LINK_CALLBACK: 0 ---*/
	#define LWIP_NETIF_LINK_CALLBACK 1
	* 回调函数
		netif_set_link_callback(&gnetif, ethernet_link_status_updated);
	* 插拔网线操作线程	
		osThreadDef(EthLink, ethernet_link_thread, osPriorityAboveNormal, 0, configMINIMAL_STACK_SIZE *2);
```