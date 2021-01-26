# stm32_study

<!-- stm32自学笔记 -->

[TOC]



## 零.声明

本文为我自学stm32的一些理解和笔记，仅交流学习目的，无商业用途。

## 一.各文件用处

以 **模板** 文件夹中各个文件为例：

### CORE文件夹

core_cm3.c/h为CM3内核芯片为外设提供进入内核的接口

startup_stm32f10x_**md**.s为启动文件，为c语言的运行建立合适环境，其中md代表中等密度产品，有下述多种型号缩写

| 缩写 |              型号               |
| :--: | :-----------------------------: |
|  vl  |    超值型产品，stm32f100系列    |
|  cl  |  互联型产品，stm32f105/107系列  |
|  xl  | 超高密度产品，stm32f101/103系列 |
|  ld  |    低等密度产品，Flash≤64KB     |
|  md  |  中等密度产品，Flash=64/128KB   |
|  hd  |     高密度产品，Flash≥128KB     |

### OBJ文件夹

存放编译后的文件

### STM32F10x_FWLib文件夹

inc文件夹中存放外设的.h文件

src文件夹中存放外设的.c文件

两个文件夹都属于设备外设函数部分

### SYSTEM文件夹

系统自带较为常用的函数组

### USER文件夹

Listings文件夹存放生成的链接文件

system_stm32f10x.c/h为设置系统时钟和总线时钟

stm32f10x.h为底层文件，包含寄存器地址和结构体定义，用固件库**必需包含**

stm32f10x_it.c/h为写入中断服务函数，函数接口自行在启动文件中查询，写入中断时要打开对NVIC（中断向量控制器）的访问函数

stm32f10x_conf.h被包含在stm32f10x.h中，是用于配置使用了什么外设的头文件

## 二.GPIO的基本控制用法

以几个GPIO操作为例，介绍GPIO一些基本函数

```c
//案例一：点亮一个LED灯（位于PA5）
void LED_Init(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOLED, ENABLE);
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_LED;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;	
	GPIO_Init(GPIOA, &GPIO_InitStructure);	
  	GPIO_SetBits(GPIOLED, GPIO_Pin_LED);
}//只是一个初始化自定义函数，其余模块化编程自行完成，参考文件： 20210120_操作GPIO点灯
```

### GPIO_Init

STM32固件库使用手册的中文翻译版 中page124面

用于定义GPIO的组别，以及要初始化的管脚、输出速率、模式

 

四种输入

上拉输入模式：默认状态读取GPIO为高电平

下拉输入模式：默认状态读取GPIO为低电平

浮空输入模式：为不确定值，阻抗较大，一般用于iic和usart接收端

模拟输入模式：不接上拉或下拉电阻，将电压信号传送到片上外设，如ADC外设

四种输出

普通/复用 推挽输出模式：高电平为3.3v，低电平0v，不需要外接电阻

普通/复用 开漏输出模式：要接上拉电阻才能输出高电平

普通推挽输出用于输出电平为0和3.3v场合

普通开漏输出用于电平不匹配场合，如要输出5v由上拉电阻和电源来提供

------

### RCC_APB2PeriphClockCmd

**STM32固件库使用手册的中文翻译版** 中page208面

用于开启外设时钟，GPIO都是挂载在APB2总线上的，其他外设参考stm32的**时钟树**

------

### GPIO_SetBits/GPIO_ResetBits

**STM32固件库使用手册的中文翻译版** 中page128面

用于控制IO口输出高低电平

```c
//案例二：用按键key（PC13）控制灯的亮灭,按下按键时灯熄灭，平时亮
 int main(void)
{	
	 char i;
   LED_Init();
	 delay_init();
	 KEY_Init();
  
 				//PA.5 输出高  
	while(1)
	{
	  i = GPIO_ReadInputDataBit(GPIOKEY , GPIO_Pin_KEY);
		if(i==i)
			GPIO_SetBits(GPIOLED , GPIO_Pin_LED);
		else if(i==0)
			GPIO_ResetBits(GPIOLED , GPIO_Pin_LED);
	}
}
//main函数
#include "key.h"
void KEY_Init(void)
{


	GPIO_InitTypeDef GPIO_InitStructure;
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOKEY, ENABLE);
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_KEY;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;	

	GPIO_Init(GPIOA, &GPIO_InitStructure);	
  GPIO_SetBits(GPIOKEY, GPIO_Pin_KEY);

}
//key初始化，设为上拉输入模式，按键按下视为输入低电平，详细程序见 20210122_GPIO读取输入
//未写入消抖过程，多专注代码主要实现方向，后续可自行加入。
```

### GPIO_Read Input/Output DataBit

**STM32固件库使用手册的中文翻译版** 中page127面

用于读取指定端口管脚的输入/输出，当GPIO为输入/输出模式时候使用

------

### GPIO_Read Input/Output Data

**STM32固件库使用手册的中文翻译版** 中page127面

读取指定的 GPIO 端口输入/输出

------

GPIO库中主要库函数基本如上，其余函数不懂用法自行百度或查询STM32固件库使用手册

### 总结步骤

配置GPIO的步骤为

（1）设置结构体

（2）打开对应外设时钟总线APB

（3）给结构体传入参数

可对GPIO操作为

（1）写入GPIO状态

（2）读出GPIO状态

## 三.EXTI中断的基本用法

### NVIC中断控制器

为了方便控制大量的中断，引入NVIC中断控制器，用于处理不可屏蔽中断NMI和外部中断EXTI，但系统滴答定时器SYSTICK**不是**由它控制

------

### NVIC_Init

**STM32固件库使用手册的中文翻译版** 中page166面

根据 NVIC_InitStruct 中指定的参数初始化外设 NVIC 寄存器，传入参数为一个结构体，下列对结构体进行分析

------

##### NVIC_IRQChannel

作为NVIC_Init里结构体中一项，用于指定需要配置的中断向量，注意对于GPIO的中断，stm32所有GPIO都引入到外部中断线上，都能进行中断操作，而PAx——PGx都连接到EXTIx上，选择时注意。

------

#### NVIC_PriorityGroupConfig

设置优先级分组：先占优先级和从优先级，共5组，组别数与抢占优先级2的n次方相等，为0时不存在（eg.第0组无抢占优先级，2^4=16个响应优先级

第1组抢占优先级2^1=2，2^3=8个响应优先级）

##### NVIC_IRQChannelPreemptionPriority

作为NVIC_Init里结构体中一项，用于配置抢占优先级，优先级越高编号越小

##### NVIC_IRQChannelSubPriority

作为NVIC_Init里结构体中一项，用于配置响应优先级，优先级越高编号越小

**抢占级别高的能打断级别低的抢占，而在抢占相同时才比较响应优先级**

------

##### NVIC_IRQChannelCmd

要使用此通道中断就让此函数传入参数为ENABLE，失能则DISABLE

------

### EXTI外部中断

配置中断一般有五步

（1）使能EXITx线和时钟第二功能AFIO时钟（AFIO时钟在使用中断或者重映射功能时需要开启，默认复用功能时不需要开启）

（2）配置中断优先级

（3）配置中断线和工作模式

（4）编写中断服务函数

根据以上几点有以下新的基本函数

#### GPIO_EXTILineConfig

**STM32固件库使用手册的中文翻译版** 中page133面

选择 GPIO 管脚用作外部中断线路

------

#### EXTI_Init

**STM32固件库使用手册的中文翻译版** 中page99面

配置中断相关结构体

------

#### 编写中断服务函数

stm32f10x_it.c为专门存放中断服务函数的，默认只有系统异常中断服务函数，其余中断函数在启动文件startup_stm32f10x_hd.s中寻找，找到后在stm32f10x_it.c中写入中断动作，统一为void类型，传入void参数。

```c
//例如
void EXTI0_IRQHandler(void)
```

### 总结步骤

（1）设置NVIC优先级

（2）指定要进行中断GPIO初始化并与EXTI相连接

（3）初始化EXTI

（4）编写中断服务函数

## 四.串口通信USART

串口外设主要有三个部分构成，分别为波特率控制、收发控制和数据存储转移。

### 波特率控制

波特率为每秒传输的二进制位数，通过向寄存器USART_BRR写入参数修改UARTDIV的分频值，用于对APB2/1总线进行分频来作为控制的时序

### 收发控制

通过对三个控制寄存器CR1、CR2、CR3和一个状态寄存器SR来写入控制参数控制发送和接收。

### 数据存储转移

发送数据时，内核或DMA外设将数据从内存写入发送数据寄存器TDR然后由发送控制器将数据从TDR中发送到移位寄存器，通过Tx将数据一位位发出。数据从TDR到移位寄存器会产生TDR已空事件TXE，从移位寄存器到全部发出会产生数据发送完成事件TC，这些都能在SR状态寄存器中查询到

接收数据为逆过程，用到Rx输入到接受移位寄存器，再从移位寄存器到接受数据寄存器RDR中，再由内核指令或DMA读入到内存

### USART初始化配置

#### （1）GPIO的初始化

作为串口通信的GPIO，不是任意引脚，要查看默认复用功能中的引脚进行初始化。参考**STM32F103x8B_DS_CH_V10**的 p17 

初始化的模式配置 参考**STM32中文参考手册_V10**的 p110 中8.1.11 外设的GPIO配置进行

#### （2）USART初始化

##### USART_Init

**STM32固件库使用手册的中文翻译版** 中page346面

```c
USART_HardwareFlowControl;//硬件流在不用到CTS和RTS时候不需要开启
USART_Mode;//在全双工时要都开启
/*
USART_Clock; 
USART_CPOL; 
USART_CPHA; 
USART_LastBit;
一般这几项不用配置
*/

```

##### USART_Cmd

**STM32固件库使用手册的中文翻译版** 中page349面

使能或者失能 USART 外设







## 五.DMA直接存储器读取

DMA是用于减轻CPU工作量的数据存储方式，例如在ADC外设寄存器读取到内存中要占用CPU资源，为了让CPU更多专注于运算和中断之中，引入DMA外设

