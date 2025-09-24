[toc]

# 程序配置与外设

## 1. 程序配置

### 1-1 `Systick`

​	1ms中断一次,回调函数为`HAL_SYSTICK_Callback`

### 1-2 USART串口通信

#### 1-2-1 简介

​	略

#### 1-2-2 引脚配置

​	由于USART1是STM32F4数据线自带的,所以引脚可以忽略

#### 1-2-3 基础代码

1. 全局变量

```c
// 定义接受长度
#define USART_RX_BUF_LEN 25 
// 接收状态和缓冲区
uint8_t USART_RX_BUF[USART_RX_BUF_LEN] ;
```

2. setup初始化

```c
// 接收中断初始化
HAL_UARTEx_ReceiveToIdle_IT(&huart1, USART_RX_BUF, USART_RX_BUF_LEN);
```

3. 接收函数中断初始化

```c
// USART串口通信
void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *huart, uint16_t Size)
{
	if (huart->Instance == USART1)
	{
		// 1.添加结束符确保字符串安全
		if (Size > 0 && Size < USART_RX_BUF_LEN) 
		{
			USART_RX_BUF[Size] = '\0';
		}

		// 2.使用字符串匹配代替固定长度检查
		if (strstr((char *)USART_RX_BUF, "send") != NULL)
		{
			HAL_GPIO_TogglePin(LED0_GPIO_Port, LED0_Pin);
		}

		// 3. 发送回显（添加\r\n+正确长度+状态检查）
		const char *echo_str = "Hello!"; // 不换行的回显字符串
		uint16_t echo_len = strlen(echo_str); // 计算长度
	
		if (huart1.gState == HAL_UART_STATE_READY) 
		{ 
			HAL_UART_Transmit_IT(&huart1, (uint8_t *)echo_str, echo_len); // 检查UART是否就绪
		}

		// 4.重启接收前清除缓存区
		memset(USART_RX_BUF, 0, USART_RX_BUF_LEN);
	
		// 5.再次接收
		HAL_UARTEx_ReceiveToIdle_IT(&huart1, USART_RX_BUF, USART_RX_BUF_LEN);
	}
}
```

## 2. 外设配置

### 2-1 OLED

#### 2-1-1 简介

* 采用江协科技OLED库(v1.2版本),进行了STM32F1标准库到STM32F4HAL库的移植

* 采用软件模拟IIC形式,节省资源

#### 2-1-2 引脚配置

| 引脚号 | 初始电平 | GPIO模式 | 上下拉 | 引脚速度 |   标签   |
| :----: | :------: | :------: | :----: | :------: | :------: |
|  PB8   |   LOW    |    PP    |   NO   |   HIGH   | OLED_SCL |
|  PB9   |   LOW    |    PP    |   NO   |   HIGH   | OLED_SDA |

#### 2-1-3 基础代码

* include

* setup:

```c
// OLED初始化
OLED_Init() ;
```

* while:

```c
// OLED刷新(必要,建议放在最后)
OLED_Update() ;
```



### 2-2 Key

#### 2-2-1 简介

* 采用江协科技Key库(编程技巧2版本),进行了简单移植
* 按键有多种模式(单击,双击,长按,按下瞬间,松开瞬间等),函数调用非常方便
* ==本代码配置了6个按键,其中有2个系统板载按键(没有加入到Key库中),还有4个接线按键,如果想要更改引脚配置,记得在函数库中修改引脚编号==

#### 2-2-2 引脚配置

|    引脚号    | GPIO模式 | 上下拉 | 标签(注意大小写) |
| :----------: | :------: | :----: | :--------------: |
|  PA-3,4,6,7  |   输入   |  上拉  |    KEY1,2,3,4    |
| PF-6,7(板载) |   输入   |  上拉  |      Key0,1      |

#### 2-2-3 基础代码

* include
* 不需要初始化
* while:

```c
if (Key_Check(KEY_1 , KEY_SINGLE))
{
    HAL_GPIO_TogglePin(LED1_GPIO_Port , LED1_Pin) ;
}
```

* 1ms定时器回调函数

```c
// 功能: 按键检测
Key_Tick() ;
```

