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
### 移植
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
### 间隔1ms发送少量数据、582字节数据、1500以上数据
```
    现象：
        间隔1ms发送数据{"msgID":1234,"op":"sw","param":，出现HardFault_Handler
        间隔1ms发送582字节数据时不会出现HardFault_Handler
        间隔1ms发送1600字节数据时？？？
    判断依据及结论：
        应该是tcp_recved(tpcb, p->tot_len)位置不对导致；
        因为接收582字节数据时，程序会过滤掉不处理，就不会占用时间，所以不会出现该现象
        改变位置后，测试一小时，不再出现HardFault_Handler
    新的问题1：
        主函数main中定时器中断计时1s打印一次“im alive”，不再打印
        》》使用自定义计数，发现主函数中也没有打印，感觉main中不再运行一样，但还可以正常断开和连接tcp以及收发数据
        》》怀疑是main函数中有进入死循环
        大约10分钟，主函数不再有打印
        》》 屏蔽gps和读温度，运行20分钟未出现问题
        》》 只屏蔽读温度，运行20分钟未出现问题
        》》 都不屏蔽，运行20分钟（4.14->4.34），也没有出现
        》》 将json解析错误提示打印打开，运行不到20分钟（4.37->4.40），出现看门狗复位
        》》 将栈空间由0x800->调整到0x2000，运行不到20分钟（4.46->4.51），出现看门狗复位
        》》 关闭网络回调中的打印，开启串口接收中断查询和复位后lwip初始化失败延时3s复位，进行测试（5.07->5.49），未出现看门狗复位
        》》》结论：
            应该是在网络回调中不能加打印（偶尔一次应该问题不大，如果是网络远端快速发送数据时，可能会有问题）
    新的问题2：
        gps和相机控制串口是否overrun，是否需要重启下串口
        》》在主函数中开启串口接收中断查询
    新的问题3：
        复位设备，lwip初始化失败
        》》复位后lwip初始化失败延时3s复位设备
    新的问题4：
        是否有必要添加网络长时间未连接，重启设备的功能
            
    《总结》：
        看门狗复位原因
            1.有可能出现HardFault_Handler（滑动窗口位置不对）
            2.主函数中进入死循环（因为主函数中打印全没有了），导致无法喂狗（长期运行出现重启的，感觉是这个原因）
        网络可以ping通但tcp建立失败
            重启时lwip初始化失败，导致可以ping通，但tcp-server没有开启，故连接不上，需要添加初始化失败判断，如果失败则延时3s然后重启设备
            重启测试完成
        串口overrun
            在主函数中开启串口接收中断查询，当获取未开启中断时，重新开启
    《结果》：
        1.调整tcp_recved位置不再出现（第一、第二次过程），即内存不足现象                        《关键点1》
        2.去掉网络回调打印，不再出现主函数运行不到现象                                         《关键点2》

    稳定性测试：
        1.测试工具，1ms发一次数据（正确或者错误数据），测试一小时（6.00开始->）
        2.部署真正的定向服务，进行自动运行测试（期间操作云台，获取温度，查看gps位置信息等）

    第一次过程：
        Assertion "pbuf_free: p->ref > 0" failed at line 650 in ..\LWIP\lwip-1.4.1\src\core\pbuf.c
        Assertion "pbuf_take: invalid buf" failed at line 973 in ..\LWIP\lwip-1.4.1\src\core\pbuf.c
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        low_level_input() err
        <INFO>	[..\LWIP\lwip_app\tcp_server_demo\tcp_server_demo.c, 294, tcp_server_poll]:	close tcp
        ###: HardFault_Handler
    第二次过程：    
    Assertion "memp_malloc: memp properly aligned" failed at line 430 in ..\LWIP\lwip-1.4.1\src\core\memp.c
    Assertion "pbuf_alloc: pbuf p->payload properly aligned" failed at line 250 in ..\LWIP\lwip-1.4.1\src\core\pbuf.c
    Assertion "check p->payload + p->len does not overflow pbuf" failed at line 256 in ..\LWIP\lwip-1.4.1\src\core\pbuf.c
    ###: HardFault_Handler


```
### 间隔1ms发送正确数据包
```
    现象：
        发送正确数据包，不清楚具体间隔，设备会复位
        有时候不会复位，网络能接收数据，发送不出去（应该是tcp_server_user_senddata，进入死循环导致的）
    
    复现方法：
        间隔1ms发送正确数据包，{"msgID":1234,"op":"TH","param":{},"error":0}

    判断：
        1.网络回调中有串口打印时，会出现main中不打印现象；一旦出现，即使停掉网络发送数据，main中还是不打印；但网络连接和收发正常。
        * 有点像堆栈问题（未正确释放），即无法返回到main函数中
        * 有点像一直进中断导致（或者中断标志未正确清除），但如果是未正确清除的话，应该一开始就有
        2.目前复位现象肯定是看门狗喂狗未喂狗或者是喂狗未成功导致（是未喂狗导致）
        3.不正确的数据包（半包、一包半、512字节以上数据包），都测试了没问题；但正确包不行，更像是解析的问题（还是没喂狗导致，有可能是程序飞了，导致进不了主函数喂狗）

    堆栈问题测试（每一个10分钟，调小堆栈，栈：0x400，堆：0x1000，打开网络中回调错误打印）：
        1.发送{"msgID":1234,"op":"sw","param":{"sw1":0,"sw2":0,"sw3":0,"sw4":0,"sw5":0},"error":0}，开关设置包，√
        2.发送{"op":"sw","param":{"sw1":0,"sw2":0,"sw3":0,"sw4":0,"sw5":0},"error":0}，无msgid包，加错误打印时，看门狗喂狗失败
                    如何判断是喂狗失败，还是没有喂狗，应该是没有进入main，即没有喂狗
                    在网络回调中添加错误打印时，出现概率高，大约15分钟出现复位（感觉还是堆栈问题或者一直中断的问题）
        》》 屏蔽主函数中所有东西，只留下打印，看是否不能进主函数（确认不是主函数中进入死循环了）（1.46->1.52复位）
        》》 屏蔽串口中断，看是否不能进主函数（屏蔽gps串口中断）（1.54开始->）
                    第一次不会出现看门狗复位，但是网络可以接收到数据，却发不出去（期间出现粘包现象，当停止发送然后再单次发送时，可以接收到），重新断开再连接，可以接收到数据，且主函数有打印
                    第二次测试还是会出现看门狗复位
                    第三次（2.50开始->2.55）出现看门狗复位
                    第四次（2.56开始->2.59）重新下载程序，出现看门狗复位
                        出现粘包丢掉情况：
                            <ERROR>	[..\HARDWARE\comm_protocol\comm_protocol.c, 1240, prot_main]	rx_data isn't object!!!
                            <ERROR>	[..\HARDWARE\comm_protocol\comm_protocol.c, 1312, prot_main]	root err!!!
                            <ERROR>	[..\HARDWARE\comm_protocol\comm_protocol.c, 1249, prot_main]	msgID isn't exit!!!
                    第五次（2.22->）在异常中断中加上打印，看是哪里引起的，看门狗复位；没有进异常中断
                    第六次（4.04->）更改使stm32往外发送函数tcp_server_user_senddata（取消无意义的内存拷贝）                       《关键点3》
                    第七次（4.41->4.50）开启所有功能后，1ms间隔发送数据，测试是否正常，还是看门狗复位
                    第八次（4.51->4.56）开启所有功能后，1ms间隔发送数据，测试是否正常，还是看门狗复位
                    第九次（4.58->5.05）关闭所有功能后，关闭gps串口中断，1ms间隔发送数据，测试是否正常，未出现
                    第10次（5.06->5.08）关闭所有功能后，打开gps串口中断，1ms间隔发送数据，测试是否正常，出现看门狗复位
                    第11次（）     关闭所有功能后，打开gps串口中断，1ms间隔发送数据，测试是否正常，出现接收到网络数据，但发送不出去，重连后正常
                    第12次（5.18->5.32）关闭所有功能后，打开gps串口中断，1ms间隔发送数据，将网络中断调为（3，0）， 出现复位
                    第13次（5.43->6.00）打开所有功能，关闭gps串口中断，1ms间隔发送数据，将网络中断调为（1，0），还是会出现复位
                    第14次（18.03->18.07）打开所有功能，1ms间隔发送数据，主函数中多加几次喂狗，还是会出现
======似乎解决=======第15次（18.10->第二天10.52【6000】）打开所有功能，1ms间隔发送数据，主函数中多加几次喂狗，去掉网络回调中的打印    《关键点4》
                        该次未出现复位
                        断开连接，（10.53->11.18）使用测试工具（自动查询功能）开始测试（且手动控制云台正常）；出现复位，获取gps时复位
====================6.6，自动查询
                    第1次，120次打印后复位；获取gps时复位
                    第2次，167次打印后复位；获取gps时复位
                    第3次，208开始->536次，没有复位
                    第4次，536次开始->547次，复位；获取gps时复位
                    第5次，只发获取gps命令，1ms间隔；没有复位
                    第6次，（14.01开始）修改打印串口的发送时间（0xffff->0xff），自动查询；复位                                     《关键点5》
                    第7次，（14.43开始）测试在上述基础上，云台发送时，不使用中断发送；未发生复位
                    第8次，（14.43开始->）第7次没有复位，在第7次基础上断开重连测试没有问题，然后16.16[计数567]开始自动查询           《关键点6》
                    》》可以只发送云台控制，来确定是否不会复位（1ms间隔，测试一段时间不会发生复位）

=======================
总结
=======================
    1.HardFault_Handler是tcp_recved(tpcb, p->tot_len)位置不对导致；
    2.运行一段时间后，main中不再打印，看门狗无法喂狗导致复位；
    原因是打印串口输出的超时时间为0xffff太长导致，顺便把云台串口发送数据改为普通发送防止中断影响《导致restart的原因！！！》
    3.去掉网络回调中的串口打印，复现的几率降低还是因为串口输出时，可能会超时导致
    4.与堆栈的关系不大，与json的编解码没有关系
    5.设备重启后，lwip初始化失败后，延时3s重启重新配置网卡
    6.lwip修改tcp_server_user_senddata

=======================
复现
=======================
    打印串口输出的超时时间设为0xffff，云台串口中断发送《开启所有自动查询（gps、温度、云台等）、大概25分钟复现设备重启restart》

=======================
二轮【6.8】
=======================
    1.自动查询，改变云台部分代码处理【9.19开始】，有复位(大约一个半小时,650次)
    2.自动查询，不请求云台【11.40开始-1.36】有复位
    3.自动查询，不请求云台和gps【1.36开始-2.00】有复位
    4.自动查询，不请求云台和gps及机箱温度【2.00开始-2.20】有复位
    5.自动查询，不请求云台和gps及机箱温度【2.20开始-2.50】有复位
    6.自动查询，不请求云台和gps及机箱温度、不请求风扇【2.52开始-3.33】，有复位
    7.自动查询，不请求云台和gps及机箱温度、不请求风扇、不请求干扰模块【3.34开始】 有复位，大概5分钟
    8.自动查询，只请求温度【3.40-5.48】没有复位
    9.自动查询，只请求电源电压【5.50-】没有结果
    10.自动查询，全部开启【6.27】没有结果
    11.自动查询，全部开启【8.06开始】，改了电源电压返回和温度返回，没有的全部赋值为0;跑了377次复位了
 =======================
三轮【6.9】，上面复位有可能是测试软件断开导致
            当有复位时，将测试软件关闭重开测试
            目的：定位到是哪个引起的
========================
    1.【11.00开始】，自动查询，全部开启，看门狗【2s】只一处喂狗；516次时复位
    2.【280次开始->284】,复位；测试工具建立连接，没几次就出现的
    3.【10次开始->90次】还是有复位
    ====================
    4.主函数中只延时打印，其它不处理，确定是那里引起的复位；先不测
    5.将看门狗关闭，业务正常启动【5.25】；（目的确认是main函数中卡死还是在网络中卡死；或者是喂狗太频繁的原因）
        网络操作正常，主函数卡死
    6.将看门狗关闭，业务正常启动【6.00开始只开温度查询，6.54全部开启自动查询】,7点39出现的主函数不运行，确实是主函数卡死 
    ================================
    7.【8.00】添加打印【m1-m11】，查看是哪里卡死；第二天发现串口和测试工具自动退出；
 =======================
三轮【6.10】，已经确认是主函数卡死，但不确定是那里卡死
             通过在主函数中打印指示，确定那里卡死
========================
    1.【9.07开始】

```

## 4.jansson库中的json_real实数构建出现非期望值，很小的值
```
    jansson库中json_real构建时，出现非期望值，以下是传输的值
        {
            "const-0": 4.10763e-270,        错误
            "const-1000": 4.1077e-270,      错误
            "double": 4.10757e-270,         错误
            "double-f": 1000,               正确
            "error": 0,
            "float": 1000,                  正确
            "float-d": 1000,                正确
            "msgID": 1234,
            "op": "c-gps",
            "param": {
                "alt": 0,
                "h": 0,
                "lat": 0,
                "lon": 0,
                "m": 0,
                "ms": 0,
                "s": 0
            },
            "struct-d": 4.10741e-270,       错误
            "struct-d-f": 1000,             正确
            "struct-f": 1000,               正确
            "struct-f-d": 1000              正确
        }

    情景1：结构体
        typedef struct {
            double test;
        }doubleTest;

        typedef struct {
            float test;
        }floatTest;
        test_f.test = 1000;
        test_d.test = 1000;
        json_object_set_new_check(jsonAck, "struct-f", json_real(test_f.test));             正确
        json_object_set_new_check(jsonAck, "struct-f-d", json_real((double)test_f.test));   正确
        json_object_set_new_check(jsonAck, "struct-d", json_real(test_d.test));             错误
        json_object_set_new_check(jsonAck, "struct-d-f", json_real((float)test_d.test));    正确
	
    情景2：float
        float f_test = 1000;
        json_object_set_new_check(jsonAck, "float", json_real(f_test));                     正确
        json_object_set_new_check(jsonAck, "float-d", json_real((double)f_test));           正确
	
    情景3：double
        double d_test = 1000;
        json_object_set_new_check(jsonAck, "double", json_real(d_test));                    错误
        json_object_set_new_check(jsonAck, "double-f", json_real((float)d_test));	        正确
	
    情景4：常量
        json_object_set_new_check(jsonAck, "const-0", json_real(0));	                    错误
        json_object_set_new_check(jsonAck, "const-1000", json_real(1000));                  错误

    注：
        前提需要Options For Target配置页面中，将Floating Point Hardware配置为Single Precision，否则以上都是结果都是错误的
```