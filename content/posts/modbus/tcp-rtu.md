---
title: "Modbus RTU与Modbus TCP的区别"
date: 2022-08-05T08:54:30+08:00
draft: false
tags: ["Modbus"]
categories: ["Modbus","RTU"]
---
# <center>Modbus RTU与Modbus TCP的区别</center>
## Modbus简介    
&emsp;&emsp;Modbus通信协议具有多个变种，支持串口（主要是RS-485总线），以太网多个版本，其中最著名的是Modbus RTU,Modbus ASCII和Modbus TCP三种。
在工业现场一般都是采用Modbus RTU协议，一般而言，大家说的基于串口通信的Modbus通信协议都是指Modbus RTU通信协议。与Modbus RTU协议相比较，Modbus TCP协议则是在RTU协议上加一个MBAP报文头，并且由于TCP是基于可靠连接的服务，RTU协议中的CRC校验码就不再需要，所以在Modbus TCP协议中是没有CRC校验码的，所以就常用一句比较通俗的话来说：Modbus TCP协议就是Modbus RTU协议在前面加上五个0以及一个6，然后去掉两个CRC校验码字节就OK。

## Modbus功能码  
功能码|含义
:--:|:--:
0x01|读线圈
0x02|读离散量输入
0x03|读保持寄存器
0x04|读输入寄存器
0x05|写单个线圈
0x06|写单个保持寄存器
0x10|写多个保持寄存器
0x0F|写多个线圈


## Modbus RTU
&emsp;&emsp;RTU协议中的指令由地址码(一个字节），功能码（一个字节），起始地址（两个字节），数据（N个字节），校验码（两个字节）五个部分组成。数据由数据长度（两个字节，表示的是寄存器个数，假定为M）和数据正文（M乘以2个字节）组成。
````
# 读（0x03），从寄存器地址01 8E 开始读，读4个寄存器00 04
发：01 03 01 8E 00 04 25 DE  
# 08表示数据长度  ，00 01 00 01 00 01 00 01读到的数据
回：01 03 08 00 01 00 01 00 01 00 01 28 D7 

 
# 写（0x10），从寄存器地址 00 20开始写，写一个寄存器 00 01，写入值 00 00
发：00 10 00 20 00 01 02 00 00 AC A0
回：00 10 00 20 00 01 01 D2

````

## Modbus TCP
&emsp;&emsp;Modbus TCP协议是在RTU协议前面添加MBAP报文头，由于TCP是基于可靠连接的服务，RTU协议中的CRC校验码就不再需要，所以在Modbus TCP协议中是没有CRC校验码。  
MBAP报文头：  
事务处理标识|协议标识|长度|单元标识符
:--:|:--:|:--:|:--:
2字节|2字节|2字节|1字节
  
备注标识：  
报文头|备注
:--:|:--:
事务处理标识|可以理解为报文的序列号，一般每次通信之后就要加1以区别不同的通信数据报文
协议标识符|00 00表示ModbusTCP协议
长度|表示接下来的数据长度，单位为字节
单元标识符|可以理解为设备地址

````
# 读取
发：00 00 00 00 00 06 00 03 00 20 00 01 
回：00 00 00 00 00 05 00 03 02 00 00 

发：00 00 00 00 00 06 00 04 00 30 00 01
回：00 00 00 00 00 05 00 04 02 00 08 

 
# 写入
发：00 00 00 00 00 09 00 10 00 20 00 01 02 00 00
回：00 00 00 00 00 06 00 10 00 20 00 01
````


## RTU和TCP对比(16进制发送)
1. 读指令对比(例：0x04)
|MBAP报文头|地址码|功能码|寄存器地址|寄存器数量|CRC校验
:--:|:--:|:--:|:--:|:--:|:--:|:--:
Modbus RTU|无|01|04|00 00|00 16|71 C4
Modbus TCP|00 00 00 00 00 06 01||04|00 00|00 16|无

2. 写指令对比(例：0x10)
|MBAP报文头|地址码|功能码|寄存器地址|寄存器数量|数据长度|正文|CRC校验
:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:
Modbus RTU|无|00|10|00 20|00 01|02|00 00|AC A0
Modbus TCP|00 00 00 00 00 09 00||10|00 20|00 01|02|00 00|无