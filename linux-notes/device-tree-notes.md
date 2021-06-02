# 设备树
## 1.背景
> 1.1 linux从3.x开始引入设备树的概念,用于实现驱动代码与设备信息相分离  
> 1.2 Device Tree可以描述的信息包括CPU的数量和类别、内存基地址和大小、总线和桥、外设连接、中断控制器和中断使用情况、GPIO控制器和GPIO使用情况、Clock控制器和Clock使用情况
## 2.设备树
### 2.1 基础
> * **设备树节点举例**  
> 设备树节点信息在Documentation/devicetree/bindings下对应的txt文件，如：dm9000网卡的设备数节点信息在Documentation/devicetree/bindings/net/davicom-dm9000.txt有详细说明，驱动源码在drivers/net/ethernet/davicom/dm9000.c  

> * **常见符号**  
>   /根节点  
>   @如果有设备地址,此符号指定  
>   &引用节点  
>   :节点别名,引用时使用&lable  
>   ,属性名称中可以包含逗号  
>   #不表示注释  
>   ""引号中为字符串  
>   <>尖括号中为32位整形数字  
>   []方括号中为32位十六进制数,0x可以省略  
### 2.2 概念
#### 2.2.1 节点名
```
    有效字符:0-9、a-z、A-Z、,、.、_、+和-
```
#### 2.2.2 路径名
#### 2.2.3 属性格式
```
	属性名字+属性值
	属性名有效字符:0-9、a-z、A-Z、,、.、_、+和-
	属性值
		<empty>:没有值,属性名的存在即可以说明真假消息
		<u32>  :32-bit大端格式整型值
		<u64>  :64-bit大端格式整型值(两个32-bit值)
		<string>:可打印的字符,以null结尾
		<prop-encoded-array>:格式特定于属性
		<phandle>:一个<u32>的值.phandle值是引用devicetree中的另一个节点的方法。可以引用的任何节点都定义了一个具有唯一<u32>值的phandle属性。该数字用于具有phandle值类型的属性的值
		<stringlist>:连接在一起的<string>值
```
#### 2.2.4 属性类型 
* **标准属性**
```
    compatible:兼容属性,值为<stringlist>。推荐的格式是“manufacturer,model”，其中manufacturer是描述制造商名称的字符串(例如股票行情符号)，model指定了模型号。

    model:型号,值为<string>。推荐的格式是:“manufacturer,model”，其中manufacturer是描述制造商名称的字符串，model指定型号。
    phandle:引用,值为<u32>。DTS中的大多数设备树(参见附录A)都不包含显式的phandle属性。当DTS编译成二进制DTB格式时，DTC工具自动插入phandle属性。

    status:状态,值为<string>。"okay":表示设备运行."disabled":当前未运行,以后可以运行(热插拔)."fail":表示设备没有运行,有严重错误,如果不修复不能运行."fail-sss":表示设备没有运行,有严重错误,如果不修复不能运行.该值的sss部分特定于该设备，并指示检测到的错误条件。

    #address-cells和#size-cells:可用于在设备树层次结构中具有子节点的任何设备节点，并描述子设备节点应该如何寻址。#address-cells属性定义用于在子节点reg属性中编码地址字段的<u32>单元的数量,#size-cells属性定义用于在子节点reg属性中编码size字段的<u32>单元的数量。#address-cells和#size-cells属性不是从devicetree中的祖先继承的,应当明确界定。#address-cells(地址个数)默认为<2>,#size-cells(长度个数)默认为<1>

    reg:reg属性描述设备资源在其父总线定义的地址空间中的地址。值为<prop-encoded-array>.通常这意味着内存映射的IO寄存器块的偏移量和长度，但在某些总线类型上可能有不同的含义。根节点定义的地址空间中的地址是CPU实际地址。

    virtual-reg:指定一个有效地址，该地址映射到设备节点的reg属性中指定的第一个物理地址。此属性使引导程序能够为客户端程序提供已设置好的虚拟到物理映射。值为<u32>

    ranges:<empty>或者<prop-encoded-array>. (child-bus-address, parent-bus-address, length)对.提供了一种方法来定义总线的地址空间(子地址空间)和总线节点的父节点的地址空间(父地址空间)之间的映射或转换。如果属性定义为<empty>值，则指定父地址空间和子地址空间相同，不需要地址转换。如果该属性不存在于总线节点中，则假定该节点的子节点与父地址空间之间不存在映射。
    注:reg是偏移量,ranges是映射

    dma-ranges:<empty>或者<prop-encoded-array>. (child-bus-address, parent-bus-address, length)对.用于描述内存映射总线的直接内存访问(DMA)结构，该内存映射总线的devicetree父节点可以通过来自总线的DMA操作访问。它提供了一种方法来定义总线的物理地址空间与总线父级的物理地址空间之间的映射或转换。dma-ranges属性值的格式是任意数量的三元组(子总线地址、父总线地址、长度)。每个指定的三元组描述一个连续的DMA地址范围。
```
* **中断和中断映射**
```
    类似一个有向无环图
    中断源到中断控制器的物理连接使用中断父属性在devicetree中表示。表示中断生成设备的节点包含中断父属性，该属性具有一个phandle值，该值指向设备中断路由到的设备，通常是中断控制器。如果一个中断生成设备没有中断父属性，那么它的中断父属性被认为是它的devicetree父元素。

    每个中断生成设备都包含一个interrupts属性，该属性的值描述该设备的一个或多个中断源。每个源都用称为中断说明符的信息表示。中断说明符的格式和意义是特定于中断域的，即，它依赖于其中断域根节点上的属性。中断域的根使用#interrupt-cells属性来定义编码中断说明符所需的<u32>值的数量。例如，对于一个打开的PIC中断控制器，中断指定器接受两个32位的值，并由中断号和中断的级别/感知信息组成。
    中断域是解释中断说明符的上下文。域的根是(1)中断控制器或(2)中断连接。

    中断控制器是物理设备，需要一个驱动程序来处理通过它路由的中断。它也可能级联到另一个中断域。中断控制器由devicetree中该节点上的中断控制器属性的存在来指定。

    中断连接定义了一个中断域和另一个中断域之间的转换。翻译基于特定于域和特定于总线的信息。域之间的转换使用中断映射属性执行。例如，一个PCI控制器设备节点可以是一个中断连接，它定义了从PCI中断名称空间(INTA、INTB等)到具有中断请求(IRQ)号的中断控制器的转换。

    当遍历中断树到达一个没有interrupts属性的中断控制器节点时，中断树的根被确定，因此没有显式的中断父节点。
```
* **中断生成设备属性**
```
    interrupts:中断,值为<prop-encoded-array>.中断被中断扩展interrupts-extended属性覆盖，通常只应该使用其中一个或另一个。

    interrupt-parent:中断父属性,值为<phandle>.由于中断树中节点的层次结构可能与devicetree不匹配，因此可以使用interrupt-parent属性显式定义中断父节点。该值是中断父进程的phandle。如果一个设备缺少这个属性，它的中断父级被认为是它的devicetree父级。

    interrupts-extended:中断扩展属性,值为<phandle>或者<prop-encoded-array>.中断扩展属性列出由设备生成的中断。当一个设备连接到多个中断控制器时，应该使用扩展中断，而不是中断，因为它用每个中断说明符编码一个父phandle。

    注:中断和中断扩展属性是互斥的。设备节点应该使用一个或另一个，但不能同时使用这两个节点。只有在与不理解中断扩展的软件兼容时，才允许同时使用这两种方法。如果同时存在扩展中断和中断，则扩展中断优先。
```
* **中断控制器属性**
```
    #interrupt-cells:定义为中断域编码中断说明符所需的单元格数量。值为<u32>。
    interrupt-controller:说明此节点为中断控制器节点,值为<empty>。
```
* **中断连结(中断映射)属性**
```
    interrupt-map:值为<prop-encoded-array>.中断映射是nexus节点上的一个属性，它用一组父中断域连接一个中断域，并指定子域中的中断说明符如何映射到各自的父域。
    中断映射是一个表，其中每一行是一个映射条目，由五个组件组成:子单元地址、子中断说明符、父中断、父单元地址、父中断说明符。

    child unit address:正在映射的子节点的单元地址。指定这一点所需的32位单元格数量由子节点所在的总线节点的#address-cells属性描述。

    child interrupt specifier:正在映射的子节点的中断说明符。指定此组件所需的32位单元格数量由该节点的#interrupt-cells属性描述，该节点是包含中断映射属性的nexus节点。

    interrupt-parent:一个<phandle>值，该值指向正在将子域映射到的中断父进程

    parent unit address:中断父域中的单元地址。指定此地址所需的32位单元格数量由中断父字段指向的节点的#address-cells属性描述。

    parent interrupt specifier:父域中的中断说明符。指定此组件所需的32位单元格数量由中断父字段指向的节点的#interrupt-cells属性描述。

    通过对中断映射中的子组件匹配单元地址/中断说明符对，可以在中断映射表上执行查找。由于单元中断说明符中的某些字段可能不相关，所以在执行查找之前应用掩码。这个掩码在中断映射掩码属性中定义
    子节点和中断父节点都需要定义#address-cell和#interrupt-cell属性。如果不需要单元地址组件，则应显式地将#address-cells定义为零。

    interrupt-map-mask:值为<prop-encoded-array>,中断树中的nexus节点指定中断映射掩码属性。此属性指定一个掩码，该掩码应用于正在中断映射属性中指定的表中查找的传入单元中断说明符。

    #interrupt-cells:值为<u32>,中断单元格属性定义为中断域编码中断说明符所需的单元格数量。
```
#### 2.2.5 基本设备结点类型
```
	root node
		#address-cells
		#size-cells
		model
		compatible

	/aliases node 别名结点

	/memory	node 物理内存结点
		device_type:<string>,值为"memory"
		reg:<prop-encoded-array>,地址和长度对
		initial-mapped-area:<prop-encoded-array>(effective address, physical address, size),effective和physical为64-bit,size为32-bit

	/chosen node
		bootargs:为客户端程序指定引导参数的字符串。如果不需要启动参数，该值可能是一个空字符串。
		stdout-path:标准输出路径,老版本为linux,stdout-path。也可以通过bootargs传入(console=ttyS0,115200)
		stdin-path:标准输入路径

	/cpus node
		#address-cells
		#size-cells:值为0

	/cpus/cpu*
		共享MMU的硬件线程通常在一个cpu节点下表示。如果设计了其他更复杂的CPU拓扑，则CPU绑定必须描述地形(例如，不共享MMU的线程)。
		cpu和线程通过一个统一的数字空间进行编号，这个数字空间应该尽可能与中断控制器对cpu /线程的编号匹配。
		device_type:值为"cpu"
		reg:值类型array
		clock-frequency
		time-frequency
		status:okay或者disabled
		enable-method:使能方法
		cpu-release-addr:与enable-method一起使用
		power-isa-version
		power-isa-*
		cache-op-block-size
		reservation-granule-size
		mmu-type
		tlb:translate look-aside buffer
		internal (L1) Cache(可能直接是L2)
		/cpus/cpu*/l?-cache
			compatible
			cache-level
```
#### 2.2.6 设备绑定
* **杂项属性**
```
    clock-frequency
    reg-shift
    label
```
* **串行**
```
    8250 UART, 16550 UART, HDLC device, and BISYNC device,不包括I2C和SPI
    clock-frequency
    current-speed(例:115200)	
    国家半导体16450/16550兼容UART要求
        compatible
        clock-frequency 
        current-speed
        reg
        interrupts
        reg-shift
        virtual-reg
```
* **网络设备**
```
    address-bits(mac地址位数 48)
    local-mac-address
    mac-address(当boot与本地mac地址不一致时,使用)
    max-frame-size(指定一帧数据传输的长度)
    max-speed(10/100/1000)
    phy-connection-type(mii/rmii/gmii/rgmii等)
    phy-handle
```
* **open-PIC**
```
    compatible
    reg
    interrupt-controller
    #interrupt-cells(2)
    #address-cells(0)
```
* **simple-bus**
```
    compatible
    ranges
```