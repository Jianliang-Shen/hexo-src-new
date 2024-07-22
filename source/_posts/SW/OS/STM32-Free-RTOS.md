---
title: STM32 Free RTOS实战
date: 2020-09-03 09:10:27
index_img: /img/post_pics/index_img/freertos.png
tags: 
    - RTOS
categories: 
    - OS
---
使用的平台：秉火STM32 Cortex-M3内核开发板，Free RTOS v8.2.3。

<!-- more -->
## 多任务流水灯
```bash
.
├── Doc
│   └── readme.txt
├── FreeRTOS      //OS依赖目录
│   ├── inc
│   │   ├── croutine.h
│   │   ├── ...   //头文件
│   └── src
│       ├── croutine.c
│       ├── ...   //源码文件
├── Libraries     //库文件
├── Project       //工程文件
└── User
    ├── led
    │   ├── bsp_led.c
    │   └── bsp_led.h
    ├── main.c    //源码
    ├── stm32f10x_conf.h
    ├── stm32f10x_it.c
    └── stm32f10x_it.h
```
创建多任务：
```c
int main ( void )
{	
	xTaskCreate( vTaskStart, "Task Start", 512, NULL, 1, NULL );
    vTaskStartScheduler();
}

void vTaskStart( void * pvParameters )
{
	LED_Init ();

	xTaskCreate( vTaskLed1, "Task Led1", 512, NULL, 1, NULL );
	xTaskCreate( vTaskLed2, "Task Led2", 512, NULL, 1, NULL );
	xTaskCreate( vTaskLed3, "Task Led3", 512, NULL, 1, NULL );

	vTaskDelete( NULL );	
}

void vTaskLed1( void * pvParameters )
{
	while(1)
	{
		macLED1_ON ();			 
		vTaskDelay( 1000 );
		macLED1_OFF ();	
		vTaskDelay( 1000 );
	}
}
```
上面依次对应主函数创建一个任务并开始调度，任务中又新建了三个任务，每个任务负责一个灯的开启和关闭。
## 任务管理-挂起与恢复
```c
static void vTaskLed1( void * pvParameters ) //LED闪烁任务，循环执行
{
	TickType_t xLastWakeTime;	
	while(1)                                                    
	{
		macLED1_ON ();			                                     
		vTaskDelayUntil(&xLastWakeTime, 200 / portTICK_RATE_MS); 
		macLED1_OFF ();		                                       	
		vTaskDelayUntil(&xLastWakeTime, 200 / portTICK_RATE_MS);
	}
	
}


static void vTaskKey( void * pvParameters )   //按键控制挂起或恢复任务
{
	uint8_t ucKey1Press = 0, ucKey2Press = 0;	
	while(1)
	{
		if( Key_Scan ( macKEY1_GPIO_PORT, macKEY1_GPIO_PIN, 1, & ucKey1Press ) )      //KEY1按下，任务挂起
		{			
			vTaskSuspend( xHandleTaskLed1 );                                           
		}
		else if( Key_Scan ( macKEY2_GPIO_PORT, macKEY2_GPIO_PIN, 1, & ucKey2Press )) //KEY2按下，任务恢复
		{		
			vTaskResume( xHandleTaskLed1 );                                          
		}
		vTaskDelay( 20 / portTICK_RATE_MS );                                         
	}
}
```
现象：开机闪烁，按下按键1将停止，按下按键2恢复。
## 消息队列
```c
/************************************ 创建队列 *********************************/
static xQueueHandle xQueue;

/********************************** 任务函数声明 *******************************/
static void vTaskStart  ( void * pvParameters );
static void vTaskSender ( void * pvParameters );
static void vTaskReceive( void * pvParameters );

/********************************** 任务函数定义 *******************************/
static void vTaskStart( void * pvParameters )                            //起始任务
{
	LED_Init ();	     //初始化LED
	USARTx_Config ();   //初始化串口
	printf("Test Queue");

	/* 创建消息队列 */
	xQueue = xQueueCreate(5, sizeof(uint32_t) );
	if(xQueue != NULL)                                                      //创建成功则创建另外两个任务
	{
		/* 创建发送1和发送2任务 */
		xTaskCreate( vTaskSender, "Task Sender1", 512, ( void *)100, 1, NULL);  //传递的参数为向队列中添加的参数
		xTaskCreate( vTaskSender, "Task Sender2", 512, ( void *)200, 1, NULL);
		
		/* 创建接收任务 */
		xTaskCreate( vTaskReceive, "Task Receiver", 512, NULL, 2, NULL );
	}	
	else                                                                      //创建失败
	{
		while(1);
	}
	vTaskDelete( NULL );                                                      //删除
}
static void vTaskSender( void * pvParameters )
{
	uint32_t lValueToSend;
	portBASE_TYPE xStatus;

	lValueToSend = ( uint32_t ) pvParameters;                  //获取参数（地址传递）
	while(1)
	{
		xStatus = xQueueSendToBack( xQueue, &lValueToSend, 0 );  //阻塞写入队列	
		if( xStatus != pdPASS )                                  
		{
		  printf("数据不能发送到数据队列\r\n");
		}	
		taskYIELD();                                             //放弃时间片，让出CPU
	}
}
static void vTaskReceive( void * pvParameters )
{
	uint32_t lReceivedValue;
	portBASE_TYPE xStatus;

	while(1)
	{
		xStatus = xQueueReceive( xQueue, &lReceivedValue, portMAX_DELAY );//阻塞读取
		if(xStatus==pdPASS)                                           
		{
			printf( "接收的数据是： %d\r\n", lReceivedValue );               
		}
		else                                                            
		{
			printf("没有接收的消息.\r\n");
		}
	}
}
/************************************* 主函数 **********************************/
int main ( void )
{	
	xTaskCreate( vTaskStart, "Task Start", 512, NULL, 1, NULL );       
	vTaskStartScheduler(); 
}
```
输出结果：
![](/img/post_pics/os/queue.PNG)

## 二值信号量--同步数据读写
```c
static SemaphoreHandle_t  xSemaphore = NULL;
xSemaphore = xSemaphoreCreateBinary();
//发送信号量
xSemaphoreGive( xSemaphore );
//接收信号量
xSemaphoreTake( xSemaphore, portMAX_DELAY );
```
## 计数信号量
```c
xSemaphore = xSemaphoreCreateCounting(5, 5);  //总共5个停车位，目前有5个可用（计数初值为5）

static void vTaskKey1( void * pvParameters ) 
{
	BaseType_t xResult;
	uint8_t ucKey1Press = 0;
	while(1)
	{
		if( Key_Scan ( macKEY1_GPIO_PORT, macKEY1_GPIO_PIN, 1, & ucKey1Press ) ) 
		{
            xResult = xSemaphoreTake( xSemaphore,0);    //接收信号量，信号值减1                  
            taskENTER_CRITICAL();
            if ( xResult == pdPASS )                      
                printf ( "\r\nKEY1被单击：成功申请到停车位。\r\n" );
            else
                printf ( "\r\nKEY1被单击：不好意思，现在停车场已满！\r\n" );
            taskEXIT_CRITICAL();
		}
		vTaskDelay( 20 / portTICK_RATE_MS );  
	}
	
}

static void vTaskKey2( void * pvParameters ) 
{
	BaseType_t xResult;
	uint8_t ucKey2Press = 0;
	while(1)
	{
		if( Key_Scan ( macKEY2_GPIO_PORT, macKEY2_GPIO_PIN, 1, & ucKey2Press ) ) 
		{
			xResult = xSemaphoreGive( xSemaphore );    //发送信号量，信号值加1
			taskENTER_CRITICAL();
			if ( xResult == pdPASS ) 
				printf ( "\r\nKEY2被单击：释放1个停车位。\r\n" );	
		    else
			    printf ( "\r\nKEY2被单击：但已无车位可以释放！\r\n" );
			taskEXIT_CRITICAL();
		}
		vTaskDelay( 20 / portTICK_RATE_MS );  	
	}
}
```

运行结果：
```bash
计数信号量模拟停车场管理测试
KEY1被单击：成功申请到停车位。
KEY1被单击：成功申请到停车位。
KEY1被单击：成功申请到停车位。
KEY1被单击：成功申请到停车位。
KEY1被单击：成功申请到停车位。
KEY1被单击：不好意思，现在停车场已满！
KEY2被单击：释放1个停车位。
KEY2被单击：释放1个停车位。
KEY2被单击：释放1个停车位。
KEY2被单击：释放1个停车位。
KEY2被单击：释放1个停车位。
KEY2被单击：但已无车位可以释放！
```

## 互斥量--同步数据读写
```c
static SemaphoreHandle_t  xSemaphore = NULL;
xSemaphore = xSemaphoreCreateMutex();
//发送信号量
xSemaphoreGive( xSemaphore );
//接收信号量
xSemaphoreTake( xSemaphore, portMAX_DELAY );
```

## 事件标志组--按键组合
```c
static EventGroupHandle_t xCreatedEventGroup = NULL;
xCreatedEventGroup = xEventGroupCreate();  //创建互斥信号量

//KEY1按下置位松开清零
xEventGroupSetBits(xCreatedEventGroup, 0x01);   //bit0置位
xEventGroupClearBits(xCreatedEventGroup, 0x01); //bit0清零

//KEY2按下置位松开清零
xEventGroupSetBits(xCreatedEventGroup, 0x02);   //bit1置位
xEventGroupClearBits(xCreatedEventGroup, 0x02); //bit1清零

xEventGroupWaitBits(xCreatedEventGroup,                              //事件标志组对象
				    0x03,                                            //等待bit0和bit1
					pdTRUE,                                          //事件发生后两位清零
					pdTRUE,                                          //等待两位均置位
					portMAX_DELAY); 	                             //无期限等待
//上面函数执行后，点亮LED3
```
运行现象：KEY1红灯，KEY2绿灯，KEY3蓝灯，按下KEY1或者KEY2红灯或者绿灯亮，同时按下后红灯、绿灯、蓝灯均亮起，松开后只剩下蓝灯。


## 软件定时器
```c
/********************************** 内核对象句柄 *******************************/
static TimerHandle_t xTimers[3] = {NULL};

static void vTimerCallback( xTimerHandle pxTimer )
{
	uint32_t ulTimerID;	
	ulTimerID = (uint32_t) pvTimerGetTimerID( pxTimer );                   //获取软件定时器的ID
	switch ( ulTimerID )
	{
		case 0:
			macLED1_TOGGLE ();                                             //切换LED1的亮灭状态
			break;
		case 1:
			macLED2_TOGGLE ();                                             //切换LED1的亮灭状态
			break;
		case 2:
			macLED3_TOGGLE ();                                             //切换LED1的亮灭状态
			break;	
    default:
        break;			
	}	
}

static void vTaskTmr( void * pvParameters )                                //Led3任务
{
	uint8_t i;
	for( i=0; i<3; i++ )
	{
	/* 创建软件定时器 */
	xTimers[i] = xTimerCreate("Timer",                                      //命名定时器
                                (i+1)*1000/portTICK_RATE_MS,                //定时的滴答数
                                pdTRUE,                                     //周期性
                                (void * const)i,                            //定时器ID
                                vTimerCallback);                            //回调函数
	}
	for( i=0; i<3; i++ )
	{
        /* 启动软件定时器 */								 
        xTimerStart(xTimers[i],                                             //软件定时器的句柄
                    portMAX_DELAY);									        //如果无法立即启动的堵塞滴答数
	}  	
	vTaskDelete( NULL );                                                    //删除定时器任务自身	
}
```
## 多内核对象测试
使用内核对象集合进行统一处理，具体实现方式在创建队列、互斥等对象后，将对象放入集合并在一个任务中统一处理：
```c
/********************************** 内核对象句柄 *******************************/
static SemaphoreHandle_t xSemaphore = NULL;
static xQueueHandle      xQueue1    = NULL;
static xQueueHandle      xQueue2    = NULL;
static xQueueSetHandle   xQueueSet  = NULL;
/********************************** 任务函数定义 *******************************/
static void vTaskStart( void * pvParameters )                       //起始任务
{
    ......
	/* 创建二值信号量 */
	xSemaphore = xSemaphoreCreateBinary();                                    //按键信号量

	/* 创建两个消息队列 */
	xQueue1 = xQueueCreate( 10, sizeof(uint32_t) );	                          //消息队列1
	xQueue2 = xQueueCreate( 10, sizeof(uint32_t) );                           //消息队列2

	/* 创建一个内核对象集合 */
	xQueueSet = xQueueCreateSet( 1 );                                       

	
	/* 添加内核对象到对象集合 xQueueSet */
    xQueueAddToSet(xSemaphore, xQueueSet);
	xQueueAddToSet(xQueue1,    xQueueSet);
	xQueueAddToSet(xQueue2,    xQueueSet);

    ......	
}

static void vTaskPend(void *pvParameters)
{
	QueueSetMemberHandle_t xActivatedMember;
	uint32_t ulQueueMsgValue;
	uint8_t  ucQueueMsgValue;

	while(1)
	{
	    /* 等待多个内核对象 */
		xActivatedMember = xQueueSelectFromSet(xQueueSet, portMAX_DELAY);

		/* 判断接收哪个内核对象 */
		if(xActivatedMember == xQueue1)                                    //如果接收到的对象是xQueue1
		{
		    /* 接收xQueue1的消息 */
			xQueueReceive(xActivatedMember, &ulQueueMsgValue, 0);
			printf("接收到消息队列1接收到消息内容为 %d\r\n", ulQueueMsgValue);
		}
		else if(xActivatedMember == xQueue2)                               //如果接收到的对象是xQueue2
		{
		    /* 接收xQueue2的消息 */
			xQueueReceive(xActivatedMember, &ucQueueMsgValue, 0);
			printf("接收到消息队列2接收到消息内容为 %d\r\n", ucQueueMsgValue);  
		}
		else if(xActivatedMember == xSemaphore)                             //如果接收到的对象是xSemaphore
		{
			/* 接收信号量 */	
			if( xSemaphoreTake(xActivatedMember, 0) == pdPASS )
			macLED1_TOGGLE();                                               //反转LED1的亮灭状态              
		}
	}
}
```
## 任务间通信--IPC
```c

/************************************ 任务句柄 *********************************/
static TaskHandle_t xHandleTaskWait = NULL;

static void vTaskWait( void * pvParameters )     //定义接收任务消息任务
{
	BaseType_t xResult;
	uint32_t   ulNotifiedValue;
	while(1)                                                  
	{
		xResult = xTaskNotifyWait(0x00000000,         //函数执行前保留任务消息数据的所有位
                                    0xFFFFFFFF,       //函数执行后清除任务消息数据的所有位
                                    &ulNotifiedValue, //接收任务消息数据
                                    portMAX_DELAY);   //等到有任务消息到来为止	
		if(xResult == pdPASS)
		{
			printf("接收到的任务消息数据为： %d\r\n", ulNotifiedValue);
		}
		else
		{
			macLED1_TOGGLE();                         //反转LED1的亮灭状态
		}		
	}
}

static void vTaskPost( void * pvParameters )     //定义发送任务消息任务
{
	uint32_t ucCount = 0;
	while(1)
	{
		 /* 发送任务消息 */
		xTaskNotify(xHandleTaskWait,                 //目标任务的任务句柄
					ucCount++,                       //任务消息数据
      		        eSetValueWithOverwrite);         //如果目标任务没有及时接收，上次的数据会被覆盖
		vTaskDelay( 2000 / portTICK_RATE_MS );       //每2s发送一次任务消息
	}
}
```
运行现象：
```bash
任务消息测试
接收到的任务消息数据为： 0
接收到的任务消息数据为： 1
接收到的任务消息数据为： 2
接收到的任务消息数据为： 3
接收到的任务消息数据为： 4
接收到的任务消息数据为： 5
接收到的任务消息数据为： 6
......
```
## 任务间通信--中断
通过按键中断获取信息，主要将发送放入中断处理函数当中：
```c
/**
  * @brief  EXTI 中断服务函数
  * @param  无
  * @retval 无
  */
void macEXTI_INT_FUNCTION (void)
{
	static portBASE_TYPE xHigherPriorityTaskWoken;
	static uint32_t ucCount = 0;

	if(EXTI_GetITStatus(macEXTI_LINE) != RESET)           //确保是否产生了EXTI Line中断
	{
		/* 发送任务消息 */
		xTaskNotifyFromISR(xHandleTaskWait,               //目标任务的任务句柄
							ucCount++,                    //任务消息数据
							eSetValueWithoutOverwrite,    //如果目标任务没有及时接收，上次的数据会被覆盖	
							&xHigherPriorityTaskWoken);   //返回目标任务的优先级是否高于当前运行任务	

		EXTI_ClearITPendingBit(macEXTI_LINE);             //清除中断标志位	
	}
	portYIELD_FROM_ISR( xHigherPriorityTaskWoken );       //切换到更高优先级的任务
}
```
## 内存管理
```c
/********************************** 任务函数定义 *******************************/
static void vTaskStart( void * pvParameters )                       //定义起始任务函数
{
	char * pMallocMem;
	/* 初始化 */
	LED_Init ();	        //初始化 LED
	Key_Initial ();         //初始化按键
	USARTx_Config ();       //初始化 USART1
	printf ( "\r\n内存管理测试\r\n" );

	/* 测试动态申请和释放内存块 */
	pMallocMem = (char *)pvPortMalloc(50);                             //申请动态内存块
	printf( "%s\r\n", ( char * ) pvParameters );                       //打印任务函数参数	
	strcpy( pMallocMem, pvParameters );                                //拷贝任务函数参数到动态内存块
	printf( "%s\r\n", pMallocMem );                                    //打印动态内存块里的字符串
	vPortFree(pMallocMem);                                             //如果动态内存块不再使用，应该释放它
	
	vTaskDelete( NULL );                                               //删除起始任务自身
	
	
}
```