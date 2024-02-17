---
title:  直流电机的驱动和PWM调速
date: 2024/2/17 24:00:00  # 文章发表时间
tags:
thumbnail: https://lixudongself.eu.org/hexo_image/post19.png
---

51单片机驱动直流电机与 PWM 调速是通过使用 51 单片机来控制直流电机的转速和方向。51 单片机通过控制电机的电流来实现驱动，并通过生成 PWM 信号来调节电机的转速。使用 PWM 调速可以使得直流电机的转速精确可控，并且减少了电机的功率损耗。在 51 单片机的控制系统中，这两种技术都是常见的应用。

一般的直流电机有两个电极，当电极正接时，电机正转，当电极反接时，电机反转。除直流电机外，常见的电机还有步进电机、舵机、无刷电机、空心杯电机等。

## 一、硬件电路

电机属于大功率负载，如果直接接在i/o口，会损坏单片机硬件。因此需要在单片机和电机之间加入驱动电路，常见的是直接驱动和h桥驱动。

![](https://lixudongself.eu.org/hexo_image/post15.png)

<center>直接驱动</center>

![](https://lixudongself.eu.org/hexo_image/post16.png)

<center>H桥驱动</center>

这里我们使用单片机开发板自带的资源ULN2002D进行驱动。

<img src="https://lixudongself.eu.org/hexo_image/post17.png" style="zoom:50%;" />

​		ULN2003D 是一种高电流可溶性继电器驱动芯片，它通常用于驱动直流电机。ULN2003D 的主要作用是将较低电压的控制信号转换为较高电压的驱动信号，从而可以通过驱动直流电机。ULN2003D 具有高电流输出能力，使用它可以减少控制系统的电路复杂度和功率损耗，并且可以额外提供驱动电流。ULN2003D 在驱动直流电机时主要用于将低电压的控制信号转换为高电压的驱动信号，从而提高驱动能力，减少电路复杂度和功率损耗。

<img src="https://lixudongself.eu.org/hexo_image/post18.png" >

​		他的内部结构与直接驱动的电路基本相同。内置达林顿管和续流二极管，工作电压高，工作电流大，灌电流可达500mA，并且能够在关态时承受50V 的电压，输出还可以在高负载电流并行运行。

​		这里我们使用P1.0控制OUT1来间接控制电机。

## 二、PWM调速

​		PWM 调速是一种用来控制直流电机转速的方法。它的原理是通过不断地开关直流电源来调节电机的转速。

​		PWM 调速的具体实现方法是通过生成一个频率固定的、占空比可变的 PWM 信号来控制直流电机的转速。占空比是指 PWM 信号在一个周期内的工作时间与总时间的比值。如果占空比越高，电机的转速就越快；如果占空比越低，电机的转速就越慢。PWM 调速的优点在于可以精确控制电机的转速，并且还可以减少电机的功率损耗。

![](https://lixudongself.eu.org/hexo_image/post19.png)

重要参数：

​	频率 = 1 / TS 占空比= TON / TS 精度= 占空比变化步距

​	这里我们使用单片机内部的资源，定时器进行pwm控制。

​	首先配置定时器，使用51单片机小工具快速生成定时器初始化函数。

```c
void Timer0Init(void)        //10毫秒@11.0592MHz
{
    TMOD &= 0xF0;        //设置定时器模式
    TMOD |= 0x01;        //设置定时器模式
    TL0 = 0xA4;        //设置定时初值
    TH0 = 0xFF;        //设置定时初值
    TF0 = 0;        //清除TF0标志
    TR0 = 1;        //定时器0开始计时
    ET0=1;        //中断初始化【程序生成的函数内没有这个】
    EA=1;                    //中断初始化
    PT0=0;                //中断初始化
}
```

​		在主函数中添加定时器的中断函数。我们提前定义一个比较值Compare用来与计数值比较，这个值会响应按键的动作进行加减，从而实现按键控制速度。

​		计数器内设置count定时自增，当count<compare时将P1.0口（Motor）置1，反之置0使其停止运行。当这一过程执行的速度很快，就会产生pwm方波。

![](https://lixudongself.eu.org/hexo_image/post20.png)

```c
void Timer0Routine() interrupt 1        //中断
{
    TL0 = 0xA4;        //设置定时初值
    TH0 = 0xFF;        //设置定时初值
    Count++;                                        //每100us打断一次，计数器加1
    if(Count<Compare)                            
    {
        Motor=1;
    }
    else
    {
        Motor=0;
    }
    Count%=100;
}
```

## 三、按键选择

​		这里我们使用单片机开发板上的独立按键模块。原理图如下。

<img src="https://lixudongself.eu.org/hexo_image/post21.png" alt="img" style="zoom:50%;" />



​		我们需要设定程序，对独立按键的四个i/o口进行扫描，实时读取按键信息，将按下的按键值传回函数。

```c
unsigned char Key()
{
    unsigned char KeyNumber=0;
    if(P3_1==0){Delay(20);while(P3_1==0);Delay(20);KeyNumber=1;}
    if(P3_0==0){Delay(20);while(P3_0==0);Delay(20);KeyNumber=2;}
    if(P3_2==0){Delay(20);while(P3_2==0);Delay(20);KeyNumber=3;}
    if(P3_3==0){Delay(20);while(P3_3==0);Delay(20);KeyNumber=4;}
    return KeyNumber;
}
```

​		在主函数中我们要将传回的按键值与他对应的转速进行绑定。为了方便，我们只使用按键1对电机转速进行控制。

```c
        KeyNum=Key();
        if(KeyNum==1)
        {
            Speed++;
            Speed%=5;        //当speed超过4将其值0，从而实现转速选择的循环
            if(Speed==0){Compare=0;}
            if(Speed==1){Compare=50;}
            if(Speed==2){Compare=70;}
            if(Speed==3){Compare=90;}
            if(Speed==4){Compare=100;}
        }
```

​		为了方便查看当前转速等级，我们引入数码管来显示转速等级。

​		数码管驱动文件如下。

```c
#include <REGX52.H>
#include "Delay.h"
 
unsigned char smgText[]={0x3F,0x06,0x5B,0x4f,0x66,0x6D,0x7D,0x07,0x7F,0x6F};      
 
void smg(unsigned char location,number)
{
    switch(location)                                                              //location显示的数字
        {
            case 1:P2_4=1;P2_3=1;P2_2=1;break;
            case 2:P2_4=1;P2_3=1;P2_2=0;break;
            case 3:P2_4=1;P2_3=0;P2_2=1;break;
            case 4:P2_4=1;P2_3=0;P2_2=0;break;
            case 5:P2_4=0;P2_3=1;P2_2=1;break;
            case 6:P2_4=0;P2_3=1;P2_2=0;break;
            case 7:P2_4=0;P2_3=0;P2_2=1;break;
            case 8:P2_4=0;P2_3=0;P2_2=0;break;
        }
        P0=smgText[number];        //number显示的数字
        Delay(1);
        P0=0x00;
    }
```

​		在主函数中我们只需要输入smg(1,Speed);即可将速度数字显示在数码管第一位。

## 四、程序总览

​		本次项目为了使程序调理情绪便于理解，我使用了模块化编程。将按键程序、延迟程序、数码管程序、定时器程序分别进行封装。

​		主函数完整内容如下：

```c
#include <REGX52.H>
#include "Key.h"
#include "Delay.h"
#include "smg.h"
#include "Timer0.h"
 
sbit Motor=P1^0;
unsigned char KeyNum,Speed;
unsigned char Compare,Count;
 
void main()
{
    Motor=0;
    Timer0Init();
    
    for(;;)
    {
        KeyNum=Key();
        if(KeyNum==1)
        {
            Speed++;
            Speed%=5;
            if(Speed==0){Compare=0;}
            if(Speed==1){Compare=50;}
            if(Speed==2){Compare=70;}
            if(Speed==3){Compare=90;}
            if(Speed==4){Compare=100;}
        }
        smg(1,Speed);
    }
}
 
 
void Timer0Routine() interrupt 1        //中断
{
    TL0 = 0xA4;        //设置定时初值
    TH0 = 0xFF;        //设置定时初值
    Count++;                                        //每100us打断一次，计数器加1
    if(Count<Compare)                           
    {
        Motor=1;
    }
    else
    {
        Motor=0;
    }
    Count%=100;
}
```

​		独立按键程序如下：

```c
#include <REGX52.H>
#include "Delay.h"
 
 
unsigned char Key()
{
    unsigned char KeyNumber=0;
    if(P3_1==0){Delay(20);while(P3_1==0);Delay(20);KeyNumber=1;}
    if(P3_0==0){Delay(20);while(P3_0==0);Delay(20);KeyNumber=2;}
    if(P3_2==0){Delay(20);while(P3_2==0);Delay(20);KeyNumber=3;}
    if(P3_3==0){Delay(20);while(P3_3==0);Delay(20);KeyNumber=4;}
    return KeyNumber;
}
```

​		延迟函数如下：

```c
#include <intrins.h>
 
void Delay(unsigned int n)
 
{
    unsigned int i,j;
    for(j=n;j>0;j--)
    for(i=112;i>0;i--);
}
```

​		数码管程序如下：

```c
#include <REGX52.H>
#include "Delay.h"
 
unsigned char smgText[]={0x3F,0x06,0x5B,0x4f,0x66,0x6D,0x7D,0x07,0x7F,0x6F};      
 
void smg(unsigned char location,number)
{
    switch(location)                                                              //location显示的数字
        {
            case 1:P2_4=1;P2_3=1;P2_2=1;break;
            case 2:P2_4=1;P2_3=1;P2_2=0;break;
            case 3:P2_4=1;P2_3=0;P2_2=1;break;
            case 4:P2_4=1;P2_3=0;P2_2=0;break;
            case 5:P2_4=0;P2_3=1;P2_2=1;break;
            case 6:P2_4=0;P2_3=1;P2_2=0;break;
            case 7:P2_4=0;P2_3=0;P2_2=1;break;
            case 8:P2_4=0;P2_3=0;P2_2=0;break;
        }
        P0=smgText[number];        //number显示的数字
        Delay(1);
        P0=0x00;
    }
```

​		定时器程序如下：

```c
#include <REGX52.H>
void Timer0Init(void)        //10毫秒@11.0592MHz
{
    TMOD &= 0xF0;        //设置定时器模式
    TMOD |= 0x01;        //设置定时器模式
    TL0 = 0xA4;        //设置定时初值
    TH0 = 0xFF;        //设置定时初值
    TF0 = 0;        //清除TF0标志
    TR0 = 1;        //定时器0开始计时
    ET0=1;        //中断初始化【程序生成的函数内没有这个】
    EA=1;                    //中断初始化
    PT0=0;                //中断初始化
}
 
 
//主函数中断示例
//void Timer0Routine() interrupt 1        //中断
//{
//    TL0 = 0x00;                                                //设置定时初值，让定时器重新开始计时            
//    TH0 = 0xDC;                                                //设置定时初值【注意：这里的定时初值要与函数Timer0Init内的初值一样】
//    Timer0Count++;                                        //每10ms打断一次，计数器加1
//    if(Timer0Count>=100)                            //当计数器到100，即经过1秒后让秒加一，然后时分做出响应变化
//    {
//                
//    
//    }
```

​		每个程序文件单独存放在c文件中，并存在与其同名的h头文件将函数定义在其中。

​		以定时器函数为例，他的头函数将这样定义！！

```c
#ifndef __TIMER0_H__
#define __TIMER0_H__
 
void Timer0Init(void);
 
#endif
```

本文为学习笔记，欢迎大家在评论区友善讨论，指出我的不足！

更多51相关问题：

[【51单片机】简易加减法计算器](http://t.csdn.cn/ko6yU)

[【51单片机】路口红黄绿交通灯设计](http://t.csdn.cn/NuPzk)