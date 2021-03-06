\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/BitArt/archive/2013/05/28/3103917.html)

```
一、协议 


1.空闲状态 


　I2C总线总线的SDA和SCL两条信号线同时处于高电平时，规定为总线的空闲状态。此时各个器件的输出级场效应管均处在截止状态，即释放总线，由两条信号线各自的上拉电阻把电平拉高。 


2.起始位与停止位的定义：
```

*   起始信号：当 SCL 为高期间，SDA 由高到低的跳变；启动信号是一种电平跳变时序信号，而不是一个电平信号。
*   停止信号：当 SCL 为高期间，SDA 由低到高的跳变；停止信号也是一种电平跳变时序信号，而不是一个电平信号。

![](https://images0.cnblogs.com/blog/470909/201305/28163733-521976b552b74ec0b18bb07f4dbc0fb2.png)3.ACK

　　发送器每发送一个字节，就在时钟脉冲 9 期间释放数据线，由接收器反馈一个应答信号。 应答信号为低电平时，规定为有效应答位（ACK 简称应答位），表示接收器已经成功地接收了该字节；应答信号为高电平时，规定为非应答位（NACK），一般表示接收器接收该字节没有成功。 对于反馈有效应答位 ACK 的要求是，接收器在第 9 个时钟脉冲之前的低电平期间将 SDA 线拉低，并且确保在该时钟的高电平期间为稳定的低电平。 如果接收器是主控器，则在它收到最后一个字节后，发送一个 NACK 信号，以通知被控发送器结束数据发送，并释放 SDA 线，以便主控接收器发送一个停止信号 P。

![](https://images0.cnblogs.com/blog/470909/201305/28163733-f2536f0d40864a098760783845adc352.png)

```
   如下图逻辑分析仪的采样结果：释放总线后，如果没有应答信号，sda应该一直持续为高电平，但是如图中蓝色虚线部分所示，它被拉低为低电平，证明收到了应答信号。

这里面给我们的两个信息是：1)接收器在SCL的上升沿到来之前的低电平期间拉低SDA；2)应答信号一直保持到SCL的下降沿结束；正如前文红色标识所指出的那样。
```

```
4.数据的有效性：
```

```
I2C总线进行数据传送时，时钟信号为高电平期间，数据线上的数据必须保持稳定，只有在时钟线上的信号为低电平期间，数据线上的高电平或低电平状态才允许变化。 

我的理解：虽然只要求在高电平期间保持稳定，但是要有一个提前量，也就是数据在SCL的上升沿到来之前就需准备好，因为在前面I2C总线之(一)---概述一文中已经指出，数据是在SCL的上升沿打入到器件(EEPROM)中的。
```

![](https://images0.cnblogs.com/blog/470909/201305/28163734-16f9cd046fe34ad89f098aa2d5179f16.png)

5\. 数据的传送：

　　在 I2C 总线上传送的每一位数据都有一个时钟脉冲相对应（或同步控制），即在 SCL 串行时钟的配合下，在 SDA 上逐位地串行传送每一位数据。数据位的传输是边沿触发。

 二、工作过程

　　总线上的所有通信都是由主控器引发的。在一次通信中，主控器与被控器总是在扮演着两种不同的角色。

1\. 主设备向从设备发送数据

　　主设备发送起始位，这会通知总线上的所有设备传输开始了，接下来主机发送设备地址，与这一地址匹配的 slave 将继续这一传输过程，而其它 slave 将会忽略接下来的传输并等待下一次传输的开始。主设备寻址到从设备后，发送它所要读取或写入的从设备的内部寄存器地址； 之后，发送数据。数据发送完毕后，发送停止位：

写入过程如下：

　　发送起始位

*   发送从设备的地址和读 / 写选择位；释放总线，等到 EEPROM 拉低总线进行应答；如果 EEPROM 接收成功，则进行应答；若没有握手成功或者发送的数据错误时 EEPROM 不产生应答，此时要求重发或者终止。
*   发送想要写入的内部寄存器地址；EEPROM 对其发出应答；
*   发送数据
*   发送停止位.
*   EEPROM 收到停止信号后，进入到一个内部的写入周期，大概需要 10ms，此间任何操作都不会被 EEPROM 响应；(因此以这种方式的两次写入之间要插入一个延时，否则会导致失败，博主曾在这里小坑了一下)

![](https://images0.cnblogs.com/blog/470909/201305/28163734-7ced9db8d19841c795adbe88690f53b0.png)

　　详细：

![](https://images0.cnblogs.com/blog/470909/201305/28163734-81d9914b5c494e658f2996311997ce63.png)

　　需要说明的是：①主控器通过发送地址码与对应的被控器建立了通信关系，而挂接在总线上的其它被控器虽然同时也收到了地址码，但因为与其自身的地址不相符合，因此提前退出与主控器的通信；

2\. 主控器读取数据的过程：

　　读的过程比较复杂，在从 slave 读出数据前，你必须先要告诉它哪个内部寄存器是你想要读取的，因此必须先对其进行写入 (dummy write)：

*   ```
    发送起始位；
    ```
    
*   ```
    发送slave地址+write bit set；
    ```
    
*   ```
    发送内部寄存器地址；
    ```
    
*   ```
    重新发送起始位，即restart；
    ```
    
*   ```
    重新发送slave地址+read bit set；
    ```
    
*   ```
    读取数据
    ```
    
    ```
    主机接收器在接收到最后一个字节后，也不会发出ACK信号。于是，从机发送器释放SDA线，以允许主机发出P信号结束传输。
    ```
    
*   ```
    发送停止位   
    ```
    

```
详细：
```