# 应用和驱动笔记
## 1.应用
### 1.1常见问题
* 1.1.1 主进程退出时,如何保证子进程退出,而不成为孤儿进程  
```

```
* 1.1.2 孤儿进程如何退出或杀死  
```

```
* 1.1.3 僵尸进程能杀死吗
```

```
* 1.1.4 在设备树中未定义，是否可以直接在应用层使用，定义和不定义的区别是什么
```vim
    1.在设备树中定义
        at24c08@54 {/* 1KB EEPROM */
            compatible = "atmel,24c08";
            reg = <0x54>;
            pagesize = <16>;
        };
    2.不在设备树中定义
```
### 1.2应用编程
* 1.2.1 GPIO
```
    1.对应关系
        公式：32*(n-1)+x
        n：GPIO1-y
        x：IO0-31
    2.GPIO的配置
        导出GPIO
            在/sys/class/gpio目录下可以看到文件export，调用该文件以实现配置
            该文件对所有GPIO编号，从0开始。GPIOn_x的编号为32*(n-1)+x，例如此处用的GPIO2_6的编号为32*1+6=38
            # echo "38" > /sys/class/gpio/export
            回到目录/sys/class/gpio下，可以看到产生了一个新的目录./gpio38，里面包含了该IO口的输入输出设置等配置文件
            注意：export文件只有root写权限，执行上述命令或者以后用C编写的可执行文件要以ROOT身份执行
        设置GPIO的方向（输入输出）
            # echo "out" > /sys/class/gpio/gpio38/direction，设置该GPIO为输出
            # echo "in" > /sys/class/gpio/gpio38/direction，设置该GPIO为输入
        设置GPIO的输出电平
            # echo "1" > /sys/class/gpio/gpio38/value，设置GPIO输出高电平
            # echo "0" > /sys/class/gpio/gpio38/value，设置GPIO输出低电平
        设置中断方式
            # echo  "rising" > /sys/class/gpio/gpio38/edge，设置GPIO为上升沿中断
            # echo  "falling" > /sys/class/gpio/gpio38/edge，设置GPIO为下降沿中断
            # echo  "both" > /sys/class/gpio/gpio38/edge，设置GPIO为双边沿中断
            # echo  "none" > /sys/class/gpio/gpio38/edge，设置GPIO为无中断
        关闭GPIO
            # echo "38" > /sys/class/gpio/unexport，删除GPIO配置文件，可以看到目录gpio38已经被删除
```