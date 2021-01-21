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
}//只是一个启动函数，其余模块化编程自行完成，或参考文件： 20210120_操作GPIO点灯
```

### GPIO_Init

STM32固件库使用手册的中文翻译版 中page124面

用于定义GPIO的组别，以及要初始化的管脚、输出速率、模式

[^模式]: 推挽输出不需要上拉电阻而开漏输出需要上拉电阻

------

### RCC_APB2PeriphClockCmd

**STM32固件库使用手册的中文翻译版** 中page208面

用于开启外设时钟，GPIO都是挂载在APB2总线上的，其他外设参考stm32的**时钟树**

------

### GPIO_SetBits\GPIO_ResetBits

**STM32固件库使用手册的中文翻译版** 中page128面

用于控制IO口输出高低电平

