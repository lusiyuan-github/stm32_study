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

