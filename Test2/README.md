# 电控组第二次培训作业

*  **毕译文**
* ***20263006495***

* **非阻塞延时实现流水灯**

* 我在项目中使用了TIM2的更新中断去帮助我实现非阻塞延时的目的
* 在主函数循环中CPU无需执行任何程序

* 1.时钟源配置
* TIM2的时钟源是RC振荡器产生的内部高速时钟信号 PLL倍频后达到72MHz
* TIM2挂在APB1外设总线上 接收到的是72MHz / 2 * 2 =72MHz的系统时钟信号
* 2.时基单元配置
* TIM2设置为了预分频器7200 - 1MHz  ARR 5000 - 1以满足每次申请更新中断的间隔为500ms 计数模式选择向上计数
* 3.中断配置
* 每次计数器中的值更新时，发出中断申请
* 4.NVIC配置
* 因为程序中只有一个中断 无需优先级排序 使能对应的中断通道

*  程序思路
* 先在外部设置一个全局变量LED_NUM并初始化为0 作用是在执行中断回调函数时确定要对哪个LED连接的GPIO Reset（低电平点亮）

* HAL_TIM_PeriodElapsedCallback()
* 中断回调函数中
* 
* 先进行标志位判断 if判断TIM2的状态寄存器对应位是否被挂起 
* 如果被挂起 先全部置为SET
* 再使用switch case语句判断现在LED_NUM是多少以判断让哪个GPIO置低电平
* LED 0->1->2 
* 每次函数末尾LED_NUM++
* 等LED_NUM =2 并执行完后 LED置为-1再++ 从0开始

* 主程序的while循环里不需要执行任何语句
* CPU只在到计数时间时对照NVIC发出的中断请求根据中断向量表找到回调函数的位置并执行
* 其他时间CPU空闲

* *核心代码段*
* void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef * htim)  
{  
  if (htim->Instance == TIM2)  
  {  
    //先让LED全部熄灭  
    HAL_GPIO_WritePin(LED0_GPIO_Port,LED0_Pin,GPIO_PIN_SET);  
    HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);  
    HAL_GPIO_WritePin(LED2_GPIO_Port,LED2_Pin,GPIO_PIN_SET);  
    //选择根据LED_NUM亮起对应的灯并执行操作  
    switch (LED_NUM)  
    {  
      case 0:  
      HAL_GPIO_WritePin(LED0_GPIO_Port,LED0_Pin,GPIO_PIN_RESET);  
      break;  
      case 1:  
      HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);  
      break;  
      case 2:  
      HAL_GPIO_WritePin(LED2_GPIO_Port,LED2_Pin,GPIO_PIN_RESET);  
      LED_NUM = -1; //方便后续LED_NUM++从0开始循环  
      break;  
    }  
  LED_NUM++;  
  }

 


