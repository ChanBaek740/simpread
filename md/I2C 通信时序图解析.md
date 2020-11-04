\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/Tangledice/p/7622794.html)

一、I2C 协议简介
----------

  I2C 通讯协议 (Inter－Integrated Circuit) 是由 Phiilps 公司开发的，由于它引脚少，硬件实现简单，可扩展性强，不需要 USART、CAN 等通讯协议的外部收发设备，现在被广泛地 使用在系统内多个集成电路 (IC) 间的通讯。   
  关于 I2C 协议的更多内容，可阅读《I2C 总线协议》，本博文主要分析 I2C 波形图，对于 I2C 的基础知识不在做介绍。

二、I2C 协议标准代码
------------

### 2.1 起始信号 & 停止信号

  起始信号：当 SCL 线是高电平时 SDA 线从高电平向低电平切换。   
  停止信号：当 SCL 线是高电平时 SDA 线由低电平向高电平切换。

#### 2.1.1 起始信号代码

```
void I2C\_Start(void)
{
    I2C\_SDA\_High();     //SDA=1
    I2C\_SCL\_High();     //SCL=1
    I2C\_Delay();
    I2C\_SDA\_Low();
    I2C\_Delay();
    I2C\_SCL\_Low();
    I2C\_Delay();
}
```

#### 2.1.2 停止信号代码

```
void I2C\_Stop(void)
{
    I2C\_SDA\_Low();
    I2C\_SCL\_High();
    I2C\_Delay();
    I2C\_SDA\_High();
    I2C\_Delay();
}
```

### 2.2 发送一个字节

  CPU 向 I2C 总线设备发送一个字节（8bit）数据

```
u8 I2C\_SendByte(uint8\_t Byte)
{
    uint8\_t i;

    /\* 先发送高位字节 \*/
    for(i = 0 ; i < 8 ; i++)
    {
        if(Byte & 0x80)
        {
            I2C\_SDA\_High();
        }
        else
        {
            I2C\_SDA\_Low();
        }
        I2C\_Delay();
        I2C\_SCL\_High();
        I2C\_Delay();
        I2C\_SCL\_Low();
        I2C\_Delay();

        if(i == 7)
        {
            I2C\_SDA\_High();                     /\* 释放SDA总线 \*/
        }
        Byte <<= 1;                             /\* 左移一位  \*/

        I2C\_Delay();
    }
}
```

### 2.3 读取一个字节

  CPU 从 I2C 总线设备上读取一个字节（8bit 数据）

```
u8 I2C\_ReadByte(void)
{
    uint8\_t i;
    uint8\_t value;

    /\* 先读取最高位即bit7 \*/
    value = 0;
    for(i = 0 ; i < 8 ; i++)
    {
        value <<= 1;
        I2C\_SCL\_High();
        I2C\_Delay();
        if(I2C\_SDA\_READ())
        {
            value++;
        }
        I2C\_SCL\_Low();
        I2C\_Delay();
    }

    return value;
}
```

### 2.4 应答

#### 2.4.1 CPU 产生一个 ACK 信号

```
void I2C\_Ack(void)
{
    I2C\_SDA\_Low();
    I2C\_Delay();
    I2C\_SCL\_High();
    I2C\_Delay();
    I2C\_SCL\_Low();
    I2C\_Delay();

    I2C\_SDA\_High();
}
```

#### 2.4.2 CPU 产生一个非 ACK 信号

```
void I2C\_NoAck(void)
{
    I2C\_SDA\_High();
    I2C\_Delay();
    I2C\_SCL\_High();
    I2C\_Delay();
    I2C\_SCL\_Low();
    I2C\_Delay();
}
```

#### 2.4.3 CPU 产生一个时钟，并读取器件的 ACK 应答信号

```
uint8\_t I2C\_WaitToAck(void)
{
    uint8\_t redata;

    I2C\_SDA\_High();
    I2C\_Delay();
    I2C\_SCL\_High();
    I2C\_Delay();

    if(I2C\_SDA\_READ())
    {
        redata = 1;
    }
    else
    {
        redata = 0;
    }
    I2C\_SCL\_Low();
    I2C\_Delay();

    return redata;
}
```

三、I2C 通信时序图解析
-------------

  有了上边的 I2C 总线标准代码的基础，下面我们进入本博文所要讲解的内容，怎么分析 I2C 的时序图，以 O2Micro 的 OZ9350 为例，OZ9350 是一款模拟前端（AFE）的 IC 器件。是一款性价比不错的电源管理芯片，由于其通讯是通过 I2C 来进行通讯的，所以这里用 OZ9350 的 I2C 通讯做例子进行讲解。

### 3.1 写数据

  首先我们先来看一下写数据的时序图，如下图所示：   
![](https://i.imgur.com/IL0Ha8b.png)  将上图中的写数据时序图进行分解，经分解后如下图所示：   
![](https://i.imgur.com/VBHwFOr.png)  
  结合 I2C 总线协议的知识，我们可以知道 OZ9350 的 I2C 写数据由一下 10 个步骤组成。   
  第一步，发送一个起始信号。   
  第二步，发送 7bit 从机地址，即 OZ9350 的地址。此处需要注意，发送数据时，无法发送 7bit 数据，此处发送了 7bit 地址 + 1bit 读写选择位，即发送 7bit+r/w。最低位为 1 表示读，为 0 表示写。   
  第三步，产生一个 ACK 应答信号，此应答信号为从机器件产生的应答。   
  第四步，发送寄存器地址，8bit 数据。   
  第五步，产生一个 ACK 应答信号，此应答信号为从机器件产生的应答。   
  第六步，发送一个数据，8bit 数据。   
  第七步，产生一个 ACK 应答信号，此应答信号为从机器件产生的应答信号。   
  第八步，发送一个 CRC 校验码，此 CRC 校验值为 2、4、6 步数据产生的校验码。   
  第九步，既可以发送一个应答信号，也可以发送一个无应答信号，均有从机器件产生。   
  第十步，发送一个停止信号。   
  接下来，按照以上是个步骤，可以写出 OZ9350 的 i2c 写数据的函数。代码如下：

```
u8 I2C\_WriteBytes(void)
{
    I2C\_Start();                    //1

    I2C\_SendByte(Slaver\_Addr | 0);  //2
    I2C\_WaitToAck();                //3

    I2C\_SendByte(Reg\_Addr);         //4
    I2C\_WaitToAck();                //5

    I2C\_SendByte(data);             //6
    I2C\_WaitToAck();                //7

    I2C\_SendByte(crc);              //8
    I2C\_WaitToAck();                //9

    I2C\_Stop();                     //10
}
```

### 3.2 读数据

  读数据的时序图如下图所示：   
![](https://i.imgur.com/HQ6HIUT.png)  
  读数据的时序图经分解后如下图所示：   
![](https://i.imgur.com/AjsozXQ.png)  
  通过分解后的时序图，可以看到 OZ9350 的读数据由以下 13 个步骤组成。   
  第一步，发送一个起始信号。   
  第二步，发送 7bit 从机地址，即 OZ9350 的地址。此处需要注意，发送数据时，无法发送 7bit 数据，此处发送了 7bit 地址 + 1bit 读写选择位，即发送 7bit+r/w。最低位为 1 表示读，为 0 表示写。   
  第三步，产生一个 ACK 应答信号，此应答信号为从机器件产生的应答。   
  第四步，发送寄存器地址。   
  第五步，产生一个 ACK 应答信号，此应答信号为从机器件产生的应答。   
  第六步，再次发送一个骑士信号。   
  第七步，发送 7bit 从机地址，即 OZ9350 的地址。此处需要注意，发送数据时，无法发送 7bit 数据，此处发送了 7bit 地址 + 1bit 读写选择位，即发送 7bit+r/w。最低位为 1 表示读，为 0 表示写。   
  第八步，产生一个 ACK 应答信号，此应答信号为从机器件产生的应答。   
  第九步，读取一个字节 (8bit) 的数据。   
  第十步，产生一个 ACK 应答信号，此应答信号为 CPU 产生。   
  第十一步，读取一个 CRC 校验码。   
  第十二步，产生一个 NACK 信号。此无应答信号由 CPU 产生。   
  第十三步，产生一个停止信号。   
  接下来，由以上分析步骤，可以写出 OZ9350 的 I2C 读数据代码。如下所示：

```
u8 I2C\_ReadBytes(void)
{
    u8 data;
    u8 crc;

    I2C\_Start();                    //1

    I2C\_SendByte(Slaver\_Addr | 0);  //2
    I2C\_WaitToAck();                //3

    I2C\_SendByte(Reg\_Addr);         //4
    I2C\_WaitToAck();                //5

    I2C\_Start();                   //6

    I2C\_SendByte(Slaver\_Addr | 1);  //7 1-读
    I2C\_WaitToAck();                //8

    data=I2C\_ReadByte();            //9

    I2C\_Ack();                      //10

    crc=I2C\_ReadByte();             //11

    I2C\_NoAck();                    //12

    I2C\_Stop();                     //13
}
```

四、结语
----

  关于怎样分析 I2C 通信的时序图，在理解原理的基础上还需要自己多动手练习，只有这样才能熟练掌握。如果在博文中出现错误之处，还望指正。