---
title: "CANoe CAPL编程"
date: 2020-06-08T08:06:25+08:00
description: Shortcodes sample
menu:
  sidebar:
    name: CANoe CAPL编程
    identifier: CANoeCAPL编程
    parent: CANoe
    weight: 10
hero: hero.svg
mermaid: true
---



## CANoe CAPL编程

### 1. CAPL概述

与Vspy的"C Code Interface"一样；在CANoe的使用中，一样提供了我们进行**二次编程开发**的工具——”**CAPL Browser**”。通过CAPL的编程，我们可以在节点上完成更为复杂的功能需求。操作如下：在CANoe工程的”**Simulation Setup**”界面下的左侧的**网络节点**中，点击**铅笔形状的图标**，进入CAPL编辑界面（若当前节点还没有创建对应的CAPL程序，则此时会先提示输入CAPL程序名，并**保存为.can后缀的文件**）

![](https://cdn.jsdelivr.net/gh/GaoSHF/7011/blogs/24b04cae1ab749738c052210b271225b.png)

#### 1.1 CAPL语言特性

- CAPL（Communication Access Programming Laguage）语言是**类C语言**，语法其实与C语言很相似，但同时又包含了一些C++的特性，如**this指针、事件**等；
- 应用于Vector CAN工具节点的编程，是基于事件建模的语言；
- 可以使用**write()函数**进行调试，用于将调试信息输出到CANoe的write窗口上；
- 通过**output()函数进行指定报文的发送**；
- 通常是通过环境变量事件与CANoe面板进行关联，实现交互；
- **提供调用dll文件的方法**（操作见"关于CAPL中对dll的调用操作"一文）；这样保证了对由其他语言封装好的程序模块的调用；

#### 1.2 CAPL的程序结构

 如下，一个完整的CAPL程序的结构包含了**头文件、全局变量、事件函数、自定义函数**；当然不是每个因素都要有，视具体程序功能确定。

![](https://cdn.jsdelivr.net/gh/GaoSHF/7011/blogs/ae70c67136834e05ac2a1133b3b1fd3f.png)

#### 1.3 CAPL的数据类型

![](https://cdn.jsdelivr.net/gh/GaoSHF/7011/blogs/030a91e452b14b22a88c04204578f0d5.jpg)

#### 1.4 CAPL事件类型概述

 CAPL是基于事件建模的语言，从1.2小节对CAPL的程序结构的介绍也可以看出，关于CAPL的运用主要就是在于熟悉其事件的使用；其常用的事件如下：

![](https://cdn.jsdelivr.net/gh/GaoSHF/7011/blogs/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20210323161403.png)

事件过程语法结构中 蓝色字体表示该程序的关键字，深红色字体表示用户自定义的名称。

{...}内是CAPL程序体，用户可根据需要使用CAPL语言编写。

### 2. CAPL事件类型

#### 2.1 系统事件

在CAPL的系统事件中，有**preStart、start、preStop、stopMeasurement**这4种。我们可以根据需要在相应的系统事件函数接口中定义想要进行的操作；当**工程运行时，下述系统事件的发生顺序依次是**

```c
preStart-->start-->preStop-->stopMeasurement
```

关于系统事件的**定义格式如下**：

```c
on preStart   	  	    /*系统事件，初始化时执行*/
{
 	resetCan();      	/*CAPL接口函数，用于复位CAN控制器*/
}

on start        	    /*系统事件，工程开始时执行*/
{
 	write(“Just A Try”);   /*write()函数将字符串信息在”write”窗口输出*/
}


on preStop    /*系统事件，工程预备停止时执行;发生stopMeasurement事件前面*/
{
  	write("The Project Will Stop!”);
}

on stopMeasurement  		/*系统事件，工程停止时执行*/
{
  	write("The End!\n");
}
```

#### 2.2 CAN控制器事件

**当硬件对CAN控制器检测到相应动作发生时执行**；以BusOff事件为例，格式如下：

```c
on busOff       /*CAN控制器事件：硬件检测到BusOff时执行*/
	{
	  	write("BusOff Error!");
	}
```

#### 2.3 CAN消息事件

通过“on message"定义消息事件，该事件会在指定的报文消息被接收时被调用。关于消息事件的定义格式示例如下：

```c
on message 123         		/*接收到123（10进制）这个ID的报文时执行*/
on message 0x441       		/*接收到0x441（16进制）这个ID的报文时执行*/
on message BCM 	       		/*接收到BCM（工程dbc文件中的报文名）这个报文时执行*/
on message*	      			/*接收到任意报文时都执行（注意*与message之间没有空格）*/
on message 0x300-0x444	 	/*接收到这个范围内的ID报文时执行*/
{
  	write(“Received %x”,this.id);	 /*打印接收到的报文id*/
  	write(“Received Message %d in total!”,count)；
}
```

​    以上是关于消息事件的定义格式，关于**消息的索引及发送操作**我们通过下例介绍：
​    假设VoiceStatus是我们工程dbc文件中定义的一个报文，该报文包括了VoiceType和VoiceOperation这两个信号；其中，VoiceType这个变量占据第1个字节；VoiceOperation占据第2、3个字节；则**关于消息的索引，通过报文的信号(msg.VoiceType这样)去操作如下**：

```c
void TxMsg_VoiceStatus(void) 
{
    message VoiceStatus msg;         /*将工程中dbc中定义的VoiceStatus这条报文取名为msg*/
    msg.VoiceType = @VoiceType;      /*对应赋值给到报文的信号，通过报文别名"msg."调出*/
    msg.VoiceOperation = @VoiceOperation;
    output(msg);                     /*通过output指令发送该报文*/
}
```

也可以直接**通过后接数据类型(msg.byte(0)这样)去操作，此时操作如下**：

```c
void TxMsg_VoiceStatus(void) 
{
    message VoiceStatus msg;         /*将工程中dbc中定义的VoiceStatus这条报文取名为msg*/
    msg.byte(0) = @VoiceType;        /*报文第1个数据字节*/
    msg.word(1) = @VoiceOperation; ; /*报文从第1个字节开始的一个字（2个字节）*/
    output(msg);                     /*通过output指令发送该报文*/
}
```

#### 2.4 键盘事件

 通过”**on key**”定义键盘事件,该事件会在我们**按下指定按键时执行**;关于键盘事件的**定义格式示例如下**：

```c
on key ‘a’      	/*在小写输入法下，按下键盘的’A’键时执行*/
on key ‘A’      	/*在大写输入法下，按下键盘的’A’键时执行*/
on key ‘ ’      	/*按下键盘的空格键时执行，注意单引号中间是有空格的*/
on key 0x20     	/*按下键盘的空格键时执行*/
on key F2      	    /*按下键盘的’F2’键时执行*/
on key Ctrl-F3      	/*同时按下键盘的’Ctrl’键和’F3’键时执行*/
on key*      		/*按下键盘的任意键时都会执行（注意*与key之间没有空格） */
{
  	write(“The Key Is Press”)；
}
```

#### 2.5 时间事件

 通过”**on timer**”定义时间事件；该事件会**在设定的时间到达时执行**。关于时间事件的**定义格式及使用示例如下**：

```c
variables
{
  msTimer Timer1;    		/*在variables中声明一个以ms为单位的定时器变量Timer1*/
}

on start
{
  setTimer(Timer1,100);     /*将Timer1的定时时间设定为100ms，并启动它*/
}

on timer Timer1  	 		/*定义的Timer1时间事件，每100ms执行一次*/
{
  setTimer(Timer1,100);     /*启动下一个周期循环*/
}

on key ‘a‘		 			/*键盘事件，按下键盘’A’键时执行*/
{
  cancelTimer(Timer1);	 	/*停止Timer1这个100ms执行一次的定时器*/
}
```

#### 2.6 错误帧事件

通过”**on errorFrame** ”定义错误帧事件；该事件会**在硬件检测到错误帧时执行**。关于错误帧事件的**定义格式示例如下**：

```c
on errorFrame       /*错误帧事件:硬件检测到错误帧时执行*/
{
  write("The error has occur"); 
}
```

#### 2.7 环境变量事件

通过”**on envVar**”定义环境变量事件；该事件会**在指定的环境变量值有新的输入时执行**（环境变量常常用于关联上一个面板控件，当我们**对控件进行操作时，对应改变关联上的环境变量值；而此时我们在CAPL中关于该环境变量的事件就会被调用**；以此完成交互操作）。关于环境变量事件的**定义格式示例如下**：

```c
on envVar BCM_HightBeamAlarm    /*环境变量事件:指定的环境变量值有输入时执行*/
{
	  byte num=0;
	  num = getValue(this);     /*可以使用getValue(环境变量名/this关键字)获取指定的环境变量的值*/
	  if(num == 1)
	  {
	    write("The envVar is %d",@BCM_HightBeamAlarm);  
	  }
	  else
	  {
	    putValue(this,1);/*使用putValue(环境变量名/this关键字,设定的值)改变指定的环境变量的值；直接赋值的话，格式是@BCM_HightBeamAlarm = 1; */
	    write("Change envVar to %d",@BCM_HightBeamAlarm);
	  }
}
```

关于在CAPL中对环境变量的操作中，**getValue()与putValue()是常用的接口函数**。其函数格式如下，具体介绍及示例也可以通过神键"F1"召唤帮助文档，在"CAPL"相关章节中进行学习。

![](https://cdn.jsdelivr.net/gh/GaoSHF/7011/blogs/cd6b9310a0b043dc8ad454f6202e7199.png)

![](https://cdn.jsdelivr.net/gh/GaoSHF/7011/blogs/5d23097740ac4289b9320257902bcd29.png)

Ps:关于**环境变量的定义是在dbc文件中完成的；CANoe工程导入该dbc文件即可使用其定义的环境变量了**。环境变量的创建如下：

![](https://cdn.jsdelivr.net/gh/GaoSHF/7011/blogs/66b827e5309b4724baf3e79eade62556.jpg)

#### 2.8 系统变量事件

​    通过”**on sysvar**”定义系统变量事件；该事件会**在指定的系统变量值有新的输入时执行**，其格式及使用方法与前一小节的环境变量基本一致；差别只在于环境变量是在dbc文件中定义的；而**系统变量的定义如下**：
​    点击工具栏的”**Environment”下的”System Variables**”；此时界面如下，右键空白处，选择”**New**”进行新建；在弹出的窗口对新建的系统变量进行参数设置。

![](https://cdn.jsdelivr.net/gh/GaoSHF/7011/blogs/d082de1b46d746d3a339172142a556bb.jpg)

关于系统变量事件的定义格式如下：

```c
on sysvar SysVar1	 /*系统变量事件:指定的系统变量值有新的输入时执行*/
	{
	  	write("The SysVar1 is %d",@SysVar1);
	}
```