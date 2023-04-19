#RT-Thread学习
##时间：2023.03.13
##第一章：了解RT-Thread
###初时RT-Thread
提供的代码目录的认识：  
不细说，资料和书上都有的！   
   
1. components/FinSH:
FinSH 是 RT-Thread 的命令行组件，提供一套供用户在命令行调用的操作接口，主要用于调试或查看系统信息。它可以使用串口 / 以太网 / USB 等与 PC 机进行通信。 
2. 文件目录和工程目录，提供的源码并非一一对应，但实际上就是一样的！

###RT-Thread启动过程

    int $Sub$$main(void)
    {
    	rt_hw_interrupt_disable();//关中断
        rtthread_startup();//启动RTT
    	return 0;
    }
	rtthread_startup();->rt_application_init();->tid = rt_thread_create("main", main_thread_entry, RT_NULL,RT_MAIN_THREAD_STACK_SIZE, RT_MAIN_THREAD_PRIORITY, 20);
一开始进入submain(),启动RTT，rtthread_startup这个函数里面有很多初始化，下面的是一部分，可以看到里面有rt_application_init函数，就是这个函数里面包含主函数中的Main。

    int rtthread_startup(void)
    {
    rt_hw_board_init();rt_show_version();rt_system_timer_init();    
    /* scheduler system initialization */
    rt_system_scheduler_init();  
    /* create init_thread */
    rt_application_init(); 
    /* timer thread initialization */
    rt_system_timer_thread_init();
    /* start scheduler */
    rt_system_scheduler_start();
    }

##时间：2023.03.14
###RT-Thread下载问题
1. RT-Thread Studio中ST-LINK下载报错“Old ST-LINK firmware version“解决
一般是STT_Studio是最新软件，ST-link用的比较久，升级一下固件就行。推荐直接去ST官网下载最新的ST-LINK的固件升级版本，链接地址：<https://www.st.com/en/development-tools/stsw-link007.html>  
因为在正点原子资料里面的可能并非是最新版本的。

2. ST-LINK error (DEV_USB_COMM_ERR)解决  
升级之后，又出现这个问题！我又反复升级了ST-LINK，到低版本，又到高版本。然后就莫名其妙的好了！！这就有点玄学了！目前原因不详
注意：好像必须要管理员方式运行
3. 又找到了一个解决办法！解决这个问题的思路。
首先，是ST_LINLK的问题，那就升级版本。下载了几把后，又出现第二个问题，打开调试，发现是ST-LINK链接问题！把代码设置一个中断！然后下载，做一个引导作用，可以看到下载器的指示灯在闪，说明可以下载了，然后在关闭调试，就可以正常下载了！！


###终端窗口串口不显示问题
1. 可以下载之后又出现这个问题，使用Studio建立工程，和视频里面的一样，main函数里面应该会在窗口上循环输出HELLO！！
2. 解决：这个问题，我刚学可搞不定！郭老师出面帮我解决的。调试，首先看一下问题出现在哪，然后发现代码卡在一个地方运行不下去了，模糊听到，需要跳转到C++代码上，我这里没有写C++代码，不会跳转，所以卡在了这里！！把该代码屏蔽即可！！（这算是这个软件的BUG！！）
3. 最终解决问题：在startup_stm32l475xx.S文件中，第97行bl __libc_init_array这行代码注释掉！！

###如何学习一个函数
功能、形参、返回值

###Studio中跳转快捷键：①ctrl+跳转字 ②F3
##第二章：内核基础
内核是操作系统最基础也是最重要的部分。下图为 RT-Thread 内核架构图，内核处于硬件层之上，内核部分包括内核库、实时内核实现。 
![](/Picture/RT-Threadneihekuangjia.png) 
####实时内核的实现包括：  
☐ 对象管理
☐ 线程管理及调度器
☐ 线程间通信管理
☐ 时钟管理
☐ 内存管理
☐ 设备管理  
内核最小的资源占用情况是 3KB ROM，1.2KB RAM

####RTT内核启动流程：
RT-Thread 支持多种平台和多种编译器，而 rtthread_startup()函数是 RT-Thread 规定的统一启动入口。一般执行顺序是：系统先从启动文件开始运行，然后进入 RT-Thread 的启动rtthread_startup()，最后进入用户入口 main()。

###创建和删除线程
####创建一个动态线程
	rt_thread_t rt_thread_create(const char *name,
                             void (*entry)(void *parameter),//函数指针
                             void       *parameter,//上面函数的参数
                             rt_uint32_t stack_size,//栈
                             rt_uint8_t  priority,//优先级
                             rt_uint32_t tick);//时间片，时钟节拍
	//define RT_TICK_PER_SECOND 1000   Tick每秒1000次，一次的时间为1ms
如何根据上面的结构体去创建一个线程，就是上面说的学习一个函数，知道功能、形参、返回值嘛 
可以看出第二个形参值是最难懂的！！传什么参数，要传一个函数进去！！ 

    rt_thread_t th1_ptr=NULL;
    void th_entry(void *parameter)
    {
      while(1){
         rt_kprintf("rt_current_thread running...\n");
         rt_thread_mdelay(1000);
      }
    }
    rt_thread_t是什么？是一个结构体指针：typedef struct rt_thread *rt_thread_t;就是创建线程的一个返回值，返回值是一个结构体类型。
	void (*entry)(void *parameter),//函数指针。传参之前还要定义一个函数指针，如上，这个函数里面写什么？该线程需要做的事情！里面的参数就是第三个参数，函数的入口地址。
	根据上面的解释初始化如下：
    th1_ptr =  rt_thread_create("rtt_demo",th_entry,NULL,1024,20,5);
	//删除线程（创建动态线程和删除线程是一对的！！）
	rt_err_t rt_thread_delete(rt_thread_t thread)
##时间：2023.03.15
####创建一个静态线程
	rt_err_t rt_thread_init(struct rt_thread *thread,//结构体指针，这里需要定义一个rt_thread指针变量，然后取地址传进来
                        const char       *name,
                        void (*entry)(void *parameter),
                        void             *parameter,
                        void             *stack_start,//栈的起始地址
                        rt_uint32_t       stack_size,//栈的大小，保存栈的信息
                        rt_uint8_t        priority,
                        rt_uint32_t       tick)
	返回值rt_err_t：@return the operation status, RT_EOK on OK, -RT_ERROR on error

	void th2_entry(void *parameter)
    {
    	int i=0;
    	for (i=0;i<5;i++)
    	{
        	rt_kprintf("th2 running...\n");
        	rt_thread_mdelay(1000);
    	}
    }
	根据上面的解释初始化如下：
	ret = rt_thread_init(&th2,"th2_demo",th2_entry,RT_NULL,rt_stack,sizeof(rt_stack),19,5);
	//脱离线程(初始化线程和这个脱离现程是一对的)
	rt_err_t rt_thread_detach(rt_thread_t thread)
动态线程与静态线程的区别是：动态线程是系统自动从动态内存堆上分配栈空间与线程句柄（初始化 heap 之后才能使用 create 创建动态线程），静态线程是由用户分配栈空间与线程句柄。  

####启动线程
	rt_err_t rt_thread_startup(rt_thread_t thread)
	注：当调用这个函数时，将把线程的状态更改为就绪状态，并放到相应优先级队列中等待调度。
	如果新启动的线程优先级比当前线程优先级高，将立刻切换到这个线程。
	rt_err_t:可以看到这个返回值可不是什么结构体变量啥的，不需要定义一个结构体变量来接收值。
	rt_thread_startup(th1_ptr);//动态线程的开启
	rt_thread_startup(&th2);//静态线程的开启
	rt_thread_t th1_ptr=NULL;//这个是结构体指针变量struct rt_thread th2;//这个是结构体变量//rt_thread_t等价于struct rt_thread*
这里需要注意：因为两个线程本身的定义不同，里面的参数也不同，他们不是同一类型的参数。  
启动线程函数：rt_thread_startup(rt_thread_t thread)可以看出这里面的形参就需要的是结构体指针变量。  
注：当调用这个函数时，将把线程的状态更改为就绪状态，并放到相应优先级队列中等待调度。如果新启动的线程优先级比当前线程优先级高，将立刻切换到这个线程。	
##第三章：线程管理
RT-Thread是支持多任务的操作系统，多任务是通过多线程的方式实现。线程是任务的载体，是RTT中最基本的调度单位。  
线程在运行的时候，它自己会认为独占CPU运行  
线程执行时的运行环境称为上下文，具体来说就是各个变量和数据，包括所有的寄存器变量、堆栈、内存信息等。  
线程相关的操作包括：创建/初始化、启动、运行、删除/脱离。

    typedef struct rt_timer *rt_timer_t;//重命名！！rt_timer_t等价于struct rt_timer *
获得当前线程：

	rt_thread_t rt_thread_self(void)
让出处理器资源：

	rt_err_t rt_thread_yield(void)
线程睡眠：  

	rt_err_t rt_thread_sleep(rt_tick_t tick)
	rt_err_t rt_thread_delay(rt_tick_t tick)
	rt_err_t rt_thread_mdelay(rt_int32_t ms)
控制线程函数：

	rt_err_t rt_thread_control(rt_thread_t thread, int cmd, void *arg)
设置和删除idle线程hook函数：

	rt_err_t rt_thread_idle_sethook(void (*hook)(void))
	rt_err_t rt_thread_idle_delhook(void (*hook)(void))
	注意：空闲线程是一个线程状态永远为就绪态的线程，因此设置的钩子函数必须保证空闲线程在任何时刻都不会处于挂起状态，例如 rt_thread_delay()，rt_sem_take() 等可能会导致线程挂起的函数都不能使用。
设置调度器hook函数：
	
	void rt_scheduler_sethook(void (*hook)(struct rt_thread *from, struct rt_thread *to))


##第四章：时钟管理
####创建动态和静态定时器中问题
这个和创建线程的时候差不多的  

	rt_timer_t tm1 ;//动态线程
	struct rt_timer tm2 ;//静态线程
	typedef struct rt_timer *rt_timer_t;你在看看这个！！前面也说了rt_timer_t等价于struct rt_timer *	
	这里一个定义的是结构体指针，一个是结构体！！创建动态定时器时，需要返回一个结构体类型的指针变量。

定时器相关函数的使用：控制定时器：rt_timer_control();  

	rt_err_t rt_timer_control(rt_timer_t timer, int cmd, void *arg)
	//注意第二和第三的参数！！示例代码：之前设置tm2定时器，3000ms溢出，现在修改RT_TIMER_CTRL_SET_TIME，改为1000ms；之前是周期循环，下面修改为打印10次后，单次循环。
	void tm2_callback(void *parameter)
	{
    	flag++;
    	if (flag==10) {
       	 	rt_timer_control(&tm2, RT_TIMER_CTRL_SET_ONESHOT, NULL);//打印10次后，单次循环。
       	 	flag=0;
    	}
    	rt_tick_t timeout=1000;
    	rt_timer_control(&tm2, RT_TIMER_CTRL_SET_TIME, (void*)&timeout);//改为1000ms,注意需要强制转换
    	rt_kprintf("[%u]tm2_callback running...\n",rt_tick_get());//rt_tick_get()，rt_kprintf函数的使用，应用；学习！！
	}  

##时间：2023.03.16
##第四章：线程间同步
###创建动态信号量和静态信号量
	/****************信号量的创建**和前面的一样创建一个线程、创建一个定时器**********************/
	//还不是太明白，这里说清楚！！
	rt_sem_t sem1;//动态信号量创建的返回值；是一个结构体指针
	struct rt_semaphore sem2;//结构体变量
	int main(void)
	{
	    int ret;
	    sem1 = rt_sem_create("sem1", 1, RT_IPC_FLAG_FIFO);
	//rt_sem_t rt_sem_create(const char *name, rt_uint32_t value, rt_uint8_t flag)
	    if (sem1==RT_NULL) {//和下面的不一样，可以参考返回值理解！！
	        LOG_E("sem1 rt_sem_create failed...\n");
	    }
	
	    LOG_D("sem1 rt_sem_create successed...\n");
	
	    ret = rt_sem_init(&sem2, "sem2", 5, RT_IPC_FLAG_FIFO);
	//rt_err_t rt_sem_init(rt_sem_t sem,const char *name,rt_uint32_t value,rt_uint8_t  flag)
	    if( ret < 0 )
	    {
	        LOG_E("sem2 rt_sem_init failed...\n");
	        return ret;
	    }
	    LOG_D("sem2 rt_sem_init successed...\n");
	    return 0;	    
	}
####详解：
①为什么两种创建方式定义的变量不同？一个是结构体变量，一个是结构指针。  
他们都是根据各自函数的需求定义的相关变量，在动态信号量创建时，他需要返回一个结构体类型的指针；在静态信号量创建时，他需要传入一个结构体类型的指针，所以，需要定义一个结构体变量，然后取地址传进来！！  
在静态信号量创建时，他需要传入一个结构体类型的指针，这里就不能定义一个结构体指针类型了，这里如过你定义一个结构体指针类型了，那分配空间只有一个指针的大小，根本就放不下这个结构体的成员变量了！！所以这里应该按照他的结构体类型，定义一个结构体变量，然后取地址传入参数！！  
②结构体类型变量和结构体类型指针变量有什么区别？结构体类型变量相当于房子，结构体类型指针变量相当于房子的钥匙。  
③判断创建是否成功的条件并不一样，这里可以参考返回值来理解！！
####静态线程&&动态线程的区别（都是一样的）
区别：
①动态线程不需要输入栈的起始地址，不需要定义线程的控制块，只要指出线程栈的大小。  
②静态线程的线程控制块和线程栈都需要静态地定义出来，而动态线程则不需要提前定义出来，是运行的时候自动分配的。 

###互斥锁/互斥量
	/***************创建动态和静态互斥锁*****************************/
	rt_mutex_t mutex1;
	struct rt_mutex mutex2;	
	int main(void)
	{
	    int ret;
	    mutex1 = rt_mutex_create("mutex_1",RT_IPC_FLAG_FIFO );
	    if (mutex1==RT_NULL) {
	        LOG_E("rt_mutex_create failed...\n");
	        return -ENOMEM;
	    }
	
	    LOG_D("rt_mutex_create successed...\n");
	
	    ret = rt_mutex_init(&mutex2, "mutex_2", RT_IPC_FLAG_FIFO);
	    if (ret < 0) {
	        LOG_E("rt_mutex_init failed...\n");
	        return -ENOMEM;
	    }
	
	    LOG_D("rt_mutex_init successed...\n");	
	    //rt_mutex_delete(mutex1);
	    //rt_mutex_detach(&mutex2);	
	    return 0;
	}
####互斥量实例
思路：创建两个线程th1_entry，th2_entry，让这两个线程都处理flag12，这样就可能出现竞态，然后使用互斥锁来解决这个问题。

	void th1_entry(void *parameter)
	{
	    while(1){
	        flag1++;
	        rt_thread_mdelay(1000);
	        flag2++;
	
	    }
	}
	void th2_entry(void *parameter)
	{
	    while(1){
	        flag1++;
	        flag2++;        
	        rt_kprintf("flag1=%d,flag2=%d\n",flag1,flag2);
	        rt_thread_mdelay(1000);
	    }
	}
	//flag2还没+线程就被挂起了，然后就直接打印了，然后线程2被挂起，flag2加加和上一次的flag1相等，但是flag1又加加了，所以每一次打印flag2比flag1要小1
	//使用互斥锁来解决这个问题
	void th1_entry(void *parameter)
	{
	    while(1){
	        rt_mutex_take(mutex1, RT_WAITING_FOREVER);//上锁
	        flag1++;
	        rt_thread_mdelay(1000);
	        flag2++;
	        rt_mutex_release(mutex1);//用完解锁
	    }
	}

###事件集
事件集被设置的是一个32位的数，也就是说一个事件集最多可以有三十二个事件，下面的rt_uint32_t set；  

	//创建一个事件集；不多说，这里和创建一个线程一样
	event1 = rt_event_create("set1", RT_IPC_FLAG_FIFO);
	if(event1 == RT_NULL){
	    LOG_E("set1 rt_event_create is failed...\n");
	    return -ENOMEM;
	}
	LOG_D("set1 rt_event_create is successed...\n");
包含的函数，发送事件和接受事件等等，了解函数功能、形参 、返回值 

	①rt_event_t rt_event_create(const char *name, rt_uint8_t flag)//创建和删除
	rt_err_t rt_event_delete(rt_event_t event)//创建和删除
	重点：rt_uint8_t flag参数，它的入口参数@param flag the flag of event RT_IPC_FLAG_FIFO 和 RT_IPC_FLAG_PRIO
	②rt_err_t rt_event_init(rt_event_t event, const char *name, rt_uint8_t flag)
	rt_err_t rt_event_detach(rt_event_t event)

	③rt_err_t rt_event_send(rt_event_t event, rt_uint32_t set)
	//This function will send an event to the event object, if there are threadssuspended on event object, it will be waked up.
	//传参：事件的结构体指针，集合的值（也就是被设置的第几个事件）
	④rt_err_t rt_event_recv(rt_event_t   event,//事件集的结构体指针
	                       rt_uint32_t  set,//集合的值（也就是被设置的第几个事件）
	                       rt_uint8_t   option,//option the receive option, either RT_EVENT_FLAG_AND or 
						   RT_EVENT_FLAG_OR should be set. RT_EVENT_FLAG_CLEAR 
	                       rt_int32_t   timeout,//timeout the waiting time  RT_WAITING_FOREVER RT_WAITING_NO
	                       rt_uint32_t *recved)
####事件集实例代码
//实例：创建三个线程，用一个事件集去规定三个线程的执行顺序！！
//用一个事件集进行多个线程的同步

	void th1_entry(void *parameter)
	{
	    while(1){
	        rt_event_recv(event1, EVENT_FLAGS_1,  RT_EVENT_FLAG_CLEAR | RT_EVENT_FLAG_AND, RT_WAITING_FOREVER, NULL);
	        rt_kprintf("th1_entry...\n");
	        rt_thread_mdelay(1000);
	        rt_event_send(event1,EVENT_FLAGS_2);
	    }
	}
	void th2_entry(void *parameter)
	{
	    while(1){
	        rt_event_recv(event1, EVENT_FLAGS_2,  RT_EVENT_FLAG_CLEAR | RT_EVENT_FLAG_AND, RT_WAITING_FOREVER, NULL);
	        rt_kprintf("th2_entry...\n");
	        rt_thread_mdelay(1000);
	        rt_event_send(event1,EVENT_FLAGS_3);
	    }
	}
    
	rt_thread_startup(th1);//别忘了需要开启线程
    rt_event_send(event1,EVENT_FLAGS_1);//这里必须要先发送一个事件，要触发线程里面的事件执行，不然进不去线程

没有这个事件集去操作的话，打印结果可以看到三个线程随机执行的，有了这个事件集的规范，三个线程就必须顺序执行。

##时间：2023.03.17
###RT-thread官方视频内容
####软件仿真，纯软件仿真
在MDK中如何操作？注意事项？


##通用外设接口
###UART串口
####写应用层串口编写实例中遇到的问题：
	../applications/main.c:447:36: error: 'RT_SERIAL_CONFIG_DEFAULT' undeclared here (not in a function)
	//报错原因：没有包含'RT_SERIAL_CONFIG_DEFAULT'这个变量的头文件！！添加#include <serial.h>
	../applications/main.c:16:20: fatal error: serial.h: No such file or directory
	//报错原因：serial.h找不到文件，就是说没有包含路径！！添加serial.h路径
	E:\Pandora_STM32L475\rt-thread\include\drivers/serial.h:122:26: error: field 'completion' has incomplete type
	//这个没有包含头文件，这里面有个坑，不用去找缺少那个头文件的，因为这里有好几处报错，都需要添加头文件，一个一个去找去添加太麻烦
	了，这个系统里给你写好了<redevice.h>头文件，你在<serial.h>.h文件中添加这一个就行，里面帮你全部包含了！！！

##时间：2023.03.19
之前学的是使用Studio软件配置生成的相关代码，来实现RT-Thread的内核相关功能！！GCC编译的，RTT官方使用的是MDK方法，软件仿真实现展示相关功能的，两个代码都需要看懂，才会移植！！
对比两者的工程框架！

###MDK ARM软件使用的是什么编译器，RT-Thread Studio使用的是什么编译器？  
ARM编译工具链包含：  
ARM编译器(armcc)
ARM汇编器(armasm)
ARM链接器(armLink)
ARM工具(armar & fromelf)
RT-Thread Studio使用的是GNU GCC编译器。GCC（GNU Compiler Collection）是由 GNU 开发的编程语言编译器。 GCC最初代表“GNU C Compiler”，当时只支持C语言。 后来又扩展能够支持更多编程语言，包括 C++、Fortran 和Java 等。   
GCC编译工具链（toolchain），是指以GCC编译器为核心的一整套工具。它主要包含以下三部分内容：  
gcc-core：即GCC编译器，用于完成预处理和编译过程，把C代码转换成汇编代码。  
Binutils ：除GCC编译器外的一系列小工具包括了链接器ld，汇编器as、目标文件格式查看器readelf等。  
glibc：包含了主要的 C语言标准函数库，C语言中常常使用的打印函数printf、malloc函数就在glibc 库中。  

###回调函数的含义及作用
解释什么是回调函数、它们有什么好处、为什么要使用它们等等问题（涉及函数指针概念，熟知函数指针！！）
####什么是回调函数？
__回调函数__就是一个通过函数指针调用的函数。如果你把函数的指针（地址）作为参数传递给另一个函数，当这个指针被用为调用它所指向的函数时，我们就说这是回调函数。  
当程序跑起来时，一般情况下，应用程序（application program）会时常通过API调用库里所预先备好的函数。但是有些库函数（library function）却要求应用先传给它一个函数，好在合适的时候调用，以完成目标任务。这个被传入的、后又被调用的函数就称为回调函数（callback function）。  
__实例：__打个比方，有一家旅馆提供叫醒服务，但是要求旅客自己决定叫醒的方法。可以是打客房电话，也可以是派服务员去敲门，睡得死怕耽误事的，还可以要求往自己头上浇盆水。 __这里，“叫醒”这个行为是旅馆提供的，相当于库函数，但是叫醒的方式是由旅客决定并告诉旅馆的，也就是回调函数。而旅客告诉旅馆怎么叫醒自己的动作，也就是把回调函数传入库函数的动作，称为登记回调函数（to register a callback function）。__  
####为什么要使用回调函数？
因为可以把调用者与被调用者分开。调用者不关心谁是被调用者，所有它需知道的，只是存在一个具有某种特定原型、某些限制条件（如返回值为int）的被调用函数。
####总结：

> 1. 回调函数是一个很有用，也很重要的概念。当发生某种事件时，系统或其他函数将会自动调用你定义的一段函数。  
> 2. 回调函数就相当于一个__中断处理函数__，由系统在符合你设定的条件时自动调用。为此，你需要做三件事：1，声明；2，定义；3，设置触发条件，就是在你的函数中把你的回调函数名称转化为地址作为一个参数，以便于系统调用。  
> 3. 所谓回调函数就是按照一定的形式由你定义并编写实现内容，当发生某种事件时，而由系统或其它函数来调用的函数。使用回调函数实际上就是在调用某个函数时，将自己编写的一个函数的地址作为参数传递给那个函数。而那个函数在需要的时候，也就是某种事情发生的时候，利用传递的函数地址调用回调函数，这时你可以利用这个机会在回调函数中处理消息或完成一定的操作。回调函数只能是全局函数，或者是静态函数，因为这个函数只是在这个类中使用，所以为了维护类的完整性，我们用类的静态成员函数来做回调函数。  

###ROM和RAM概念、区别：
RAM：随机存取存储器（英语：Random Access Memory，缩写：RAM），也叫主存，是与CPU直接交换数据的内部存储器。计算机和手机中一般把其叫做（运行）内存，它的速度要比硬盘快得多，所以用运行程序在RAM中，而存放运行时不用的数据则在硬盘中，什么时候需要数据，便把数据从硬盘中拿到内存，但同时RAM断电会丢失数据，所以我们电脑如果断电了就会丢失原来正在运行的数据。 
 
ROM:（只读内存(Read-Only Memory)简称）英文简称ROM。ROM所存数据，一般是装入整机前事先写好的，整机工作过程中只能读出，而不像随机存储器那样能快速地、方便地加以改写。  
__区分:__无论是电脑还是手机， __容量小的那个一定是内存RAM__，容量大的一定是存储（闪存）ROM。  

Flash 存储器（FLASH EEPROM）又称闪存，快闪。它是EEPROM的一种。它结合了ROM和RAM的长处。不仅具备电子可擦除可编辑（EEPROM）的性能，还不会断电丢失数据同时可以快速读取数据。它于EEPROM的最大区别是，FLASH按扇区（block）操作，而EEPROM按照字节操作。FLASH的电路结构较简单，同样容量占芯片面积较小，成本自然比EEPROM低，因此适合用于做程序存储器。

###句柄
什么是句柄？  
句柄（Handle）是一个是用来标识对象或者项目的标识符，可以用来描述窗体、文件等，值得注意的是句柄不能是常量。  

个人认为：句柄就是健康码，它的作用就是让你找到所需的对象去到了何处，因为你始终都可以根据健康码找到对象的所在地，然后找到对象给他做核酸检测。健康码（句柄值）是国家（操作系统）分给你的，你没有自定义的权利。

在操作系统中，我们想要操作一个对象，就要知道它的地址，但是对象的内存地址总是变化，因为在windows系统中的内存管理一般会将当前处于空闲状态的对象的内存释放掉，当需要访问的时候再重新提交分配物理内存，从而导致对象的物理地址是变化的。此时windows就搞了一个玩意—句柄，句柄用来管理对象的地址（句柄表），不管对象的地址如何变化，我都可以通过访问句柄来拿到对象的实时地址，进而操作对象。句柄值是操作系统给的，你不能定义。

> static rt_device_t serial;                /* 串口设备句柄 */  
> 这里的这个结构体指针变量不是很熟悉了吗！！它就是句柄！！

##Env开发工具
Env 是 RT-Thread 推出的开发辅助工具，针对基于 RT-Thread 操作系统的项目工程，提供编译构建环境、图形化系统配置及软件包管理功能。其内置的 menuconfig 提供了简单易用的配置剪裁工具，可对内核、组件和软件包进行自由裁剪，使系统以搭积木的方式进行构建。

##时间：2023.3.20
<font color=Blue>Test</font>，效果为Test。
###rt-thread studio 无法使用 stlink 仿真调试问题
这个问题和串口终端无法显示原因一致！！  

    解决方法：  
	在startup_stm32l475xx.S文件中，第97行bl __libc_init_array这行代码注释掉！！
	注：startup_stm32l475xx.S文件路径为rtt_demo_io\libraries\CMSIS\Device\ST\STM32L4xx\Source\Templates\gcc

###I/O 设备模型框架
RT-Thread 提供了一套简单的 I/O 设备模型框架，如下图所示，它位于硬件和应用程序之间，共分成三层，从上到下分别是 I/O 设备管理层、设备驱动框架层、设备驱动层。
![](\Picture\IOshebiequdong.png)  

应用程序通过 I/O 设备管理接口获得正确的设备驱动，然后通过这个设备驱动与底层 I/O 硬件设备进行数据（或控制）交互。

I/O 设备管理层实现了对设备驱动程序的封装。应用程序通过图中的"I/O设备管理层"提供的标准接口访问底层设备，设备驱动程序的升级、更替不会对上层应用产生影响。这种方式使得设备的硬件操作相关的代码能够独立于应用程序而存在，双方只需关注各自的功能实现，从而降低了代码的耦合性、复杂性，提高了系统的可靠性。

设备驱动框架层是对同类硬件设备驱动的抽象，将不同厂家的同类硬件设备驱动中相同的部分抽取出来，将不同部分留出接口，由驱动程序实现。

设备驱动层是一组驱使硬件设备工作的程序，实现访问硬件设备的功能。它负责创建和注册 I/O 设备，对于操作逻辑简单的设备，可以不经过设备驱动框架层，直接将设备注册到 I/O 设备管理器中  

##C语言基础
结构体
指针  
链表  
数据结构

物联网相关的软件包：Paho MQTT、WebClient、mongoose、WebTerminal 等等。  

##时间：2023.03.21
###I/O 设备模型框架
> ☐ 应用程序通过 I/O 设备管理接口获得正确的设备驱动，然后通过这个设备驱动与底层 I/O 硬件设备进行交互。  
☐ I/O 设备管理层实现了对设备驱动程序的封装。  
☐ 设备驱动框架层是对同类硬件设备驱动的抽象，将不同厂家的同类硬件设备驱动中相同的部分抽取出来，将不同部分留出接口，由驱动程序实现。  
☐ 设备驱动层是一组驱使硬件设备工作的程序，实现访问硬件设备的功能。 

####创建和注册IO设备
驱动层负责创建设备实例，并注册到 I/O 设备管理器中   

	rt_device_t rt_device_create(int type, int attach_size)//创建设备，参数int type：设备类型，attach_size大小,返回值：结构体
	void rt_device_destroy(rt_device_t dev)//当一个动态创建的设备不再需要使用时可以通过如下函数来销毁
	rt_err_t rt_device_register(rt_device_t dev,           //设备注册
                            const char *name,
                            rt_uint16_t flags)
	rt_err_t rt_device_unregister(rt_device_t dev)// 设备注销后的，设备将从设备管理器中移除，也就不能再通过设备查找搜索到该
	设备.注销设备不会释放设备控制块占用的内存.
####访问IO设备
应用程序通过 I/O 设备管理接口来访问硬件设备，当设备驱动实现后，应用程序就可以访问该硬件，I/O 设备管理接口与 I/O 设备的操作方法的映射关系下图所示:  
![](/Picture/IOshebeifangwen.png)

##ADC设备驱动实例
访问ADC设备,应用程序通过 RT-Thread 提供的 ADC 设备管理接口来访问 ADC 硬件
###相关接口如下所示：
	
	rt_device_t rt_device_find(const char* name);//查找 ADC 设备
	
	rt_err_t rt_adc_enable(rt_adc_device_t dev, rt_uint32_t channel) //使能 ADC 通道
	@dev ADC 设备句柄 @channel ADC 通道
		
	rt_uint32_t rt_adc_read(rt_adc_device_t dev, rt_uint32_t channel); //读取 ADC 通道采样值
	@dev ADC 设备句柄 @channel ADC 通道
	
	rt_err_t rt_adc_disable(rt_adc_device_t dev, rt_uint32_t channel);//关闭 ADC 通道

###如何使用ADC驱动
> 
1. rt_device_t rt_device_find(const char* name);//查找 ADC 设备。在使用查找指令之前，需要先开启ADC相关驱动
2. 大部分都在board.c和board.h里面的文件中去修改：像IIC、SPI、USART、PWM等等，都是在这里去要求你进行开启相关驱动的操作。只有先开启了才能查找，应用驱动。
 * STEP 1, open adc driver framework support in the RT-Thread Settings file  
 * STEP 2, define macro related to the adc  
 such as     #define BSP_ USING_ADC1
 * STEP 3, copy your adc init function from stm32xxxx_hal_msp.c generated by stm32cubemx to the end of board.c file  
 such as     void HAL_ADC_MspInit(ADC_HandleTypeDef* hadc)
 * STEP 4, modify your stm32xxxx_hal_config.h file to support adc peripherals. define macro related to the peripherals  
 such as     #define HAL_ADC_MODULE_ENABLED

	STEP3：STM32Cube MX生成代码
	void HAL_ADC_MspInit(ADC_HandleTypeDef* hadc)
	{
	  GPIO_InitTypeDef GPIO_InitStruct = {0};
	  if(hadc->Instance==ADC1)
	  {
	  /* USER CODE BEGIN ADC1_MspInit 0 */
	
	  /* USER CODE END ADC1_MspInit 0 */
	    /* Peripheral clock enable */
	    __HAL_RCC_ADC_CLK_ENABLE();
	
	    __HAL_RCC_GPIOA_CLK_ENABLE();
	    /**ADC1 GPIO Configuration
	    PA4     ------> ADC1_IN9
	    */
	    GPIO_InitStruct.Pin = GPIO_PIN_4;
	    GPIO_InitStruct.Mode = GPIO_MODE_ANALOG_ADC_CONTROL;
	    HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	  /* USER CODE BEGIN ADC1_MspInit 1 */
	
	  /* USER CODE END ADC1_MspInit 1 */
	  }
	}

主函数代码：  

	#include <rtthread.h>	
	#define DBG_TAG "main"
	#define DBG_LVL DBG_LOG
	#include <rtdbg.h>
	#include <board.h>
	#include <rtdevice.h>
	
	#define REFER_VOLTAGE       330         /* 参考电压 3.3V,数据精度乘以100保留2位小数*/
	#define CONVERT_BITS        (1 << 12)   /* 转换位数为12位 */
	
	rt_adc_device_t dev;
	rt_thread_t th1;
	void read_adc1_entry(void *parameter)
	{
	    rt_uint32_t val=0;
	    rt_uint32_t vol=0;
	    while(1){
	        val = rt_adc_read(dev, 9);
	        rt_kprintf("val: %d\n",val);		
	        /* 转换为对应电压值 */
	        vol = val * REFER_VOLTAGE / CONVERT_BITS;
	        rt_kprintf("the voltage is :%d.%02d V\n", vol / 100, vol % 100);
	
	        rt_thread_mdelay(5000);
	    }
	}
	
	int main(void)
	{
	    rt_err_t ret=0;
	    dev = (rt_adc_device_t)rt_device_find("adc1");//查找设备
	    if (dev == RT_NULL) {
	        LOG_E("rt_device_find(adc1) is failed...\n");
	        return -EINVAL;
	    }
	
	    ret = rt_adc_enable(dev, 9);//这个函数的类型需要仔细分析
	    if (ret < 0) {
	        LOG_E("rt_adc_enable(adc1) is failed...\n");
	        return ret;
	    }
	
	    th1 = rt_thread_create("th1", read_adc1_entry, NULL, 512, 10, 5);
	    if (th1 == RT_NULL) {
	        LOG_E("rt_thread_create(th1) is failed...\n");
	        return -ENOMEM;
	    }
	    rt_thread_startup(th1);
	
	    return RT_EOK;
	}

##时间：2023.03.22
###ADC驱动实例代码解读
	rt_adc_device_t dev;//这里的定义为什么不是rt_device_t dev;
	
	dev = (rt_adc_device_t)rt_device_find("adc1");//为什么这个地方非要一个强转类型！！
	ret = rt_adc_enable(dev, 9);//dev这个函数的类型需要仔细分析
	解释：rt_device_t rt_device_find(const char *name)
	rt_err_t rt_adc_enable(rt_adc_device_t dev, rt_uint32_t channel);
	仔细观察这两个函数，一般来说使用find();函数确实需要定义一个rt_device_t类型的结构体变量来接收返回值，但是下面我将接收到的dev  
    结构体变量传入rt_adc_enable这个函数，可以看到它的第一个参数是rt_adc_device_t dev类型的结构体变量，所以这里要定义
    rt_adc_device_t类型的结构体变量，同时将find（）函数的返回值强转成rt_adc_device_t 类型。


###重点：函数指针+链表：追本溯源理解代码
	
	int rt_device_pin_register(const char *name, const struct rt_pin_ops *ops, void *user_data)	①
	
	_hw_pin.ops                 = ops;   ⑥
	//从rt_device_pin_register()函数中的ops参数为入口；可以找到ops的参数类型是struct rt_pin_ops* 结构体指针类型
	//顺着rt_pin_ops结构体标签找到该结构体。里面的成员变量为pin_mode();pin_write();pin_read();等函数指针
	struct rt_pin_ops   ②
	{
	    void (*pin_mode)(struct rt_device *device, rt_base_t pin, rt_base_t mode);
	    void (*pin_write)(struct rt_device *device, rt_base_t pin, rt_base_t value);
	    int (*pin_read)(struct rt_device *device, rt_base_t pin);
	
	    /* TODO: add GPIO interrupt */
	    rt_err_t (*pin_attach_irq)(struct rt_device *device, rt_int32_t pin,
	                      rt_uint32_t mode, void (*hdr)(void *args), void *args);
	    rt_err_t (*pin_detach_irq)(struct rt_device *device, rt_int32_t pin);
	    rt_err_t (*pin_irq_enable)(struct rt_device *device, rt_base_t pin, rt_uint32_t enabled);
	};
	
	//现在去使用这个函数，可以看到传入的第二个参数是&_stm32_pin_ops，而这个结构体变量_stm32_pin_ops又是这个struct rt_pin_ops类型的。 
	return rt_device_pin_register("pin", &_stm32_pin_ops, RT_NULL);    ③
 
	const static struct rt_pin_ops _stm32_pin_ops =       ④
	{
	    stm32_pin_mode,
	    stm32_pin_write,
	    stm32_pin_read,
	    stm32_pin_attach_irq,
	    stm32_pin_dettach_irq,
	    stm32_pin_irq_enable,
	};

	//最后的使用方法：   ⑤
	return _hw_pin.ops->pin_read(&_hw_pin.parent, pin);//_hw_pin.ops = ops;-->ops=_stm32_pin_ops;-->_hw_pin=_stm32_pin_ops;
	_hw_pin.ops->pin_write(&_hw_pin.parent, pin, value);
	_hw_pin.ops->pin_mode(&_hw_pin.parent, pin, mode);
	return _hw_pin.ops->pin_irq_enable(&_hw_pin.parent, pin, enabled);
	
> 总结：  
>1.  ③处将& _stm32_pin_ops取地址传入 rt_device_pin_register函数中，那这个 _stm32_pin_ops结构体变量必然是和该函数第二个参数的定义类型是一样的，都是struct rt_pin_ops结构体类型。  
>2. rt_device_pin_register函数第二个参数是定义的为结构体指针类型，而 _stm32_pin_ops是结构体变量，所以这里需要取地址传入该函数中。  
>3. &_stm32_pin_ops就是ops，在⑥中定义_hw_pin.ops = ops；所以在⑤中_hw_pin.ops->pin_read(&_hw_pin.parent, pin)的含义_stm32_pin_ops->pin_read，这样转换就好理解多了，就是对这个结构体的成员变量进行相关操作了呗！！


###函数指针
	int(*p)(int, int);
这个语句就定义了一个指向函数的指针变量 p。首先它是一个指针变量，所以要有一个“*”，即（*p）；其次前面的 int 表示这个指针变量可以指向返回值类型为 int 型的函数；后面括号中的两个 int 表示这个指针变量可以指向有两个参数且都是 int 型的函数。所以合起来这个语句的意思就是：定义了一个指针变量 p，该指针变量可以指向返回值类型为 int 型，且有两个整型参数的函数。p 的类型为 int(*)(int，int)。  
所以函数指针的定义方式为： 
  
	函数返回值类型 (* 指针变量名) (函数参数列表);
	int (*pin_read)(struct rt_device *device, rt_base_t pin);
	static int stm32_pin_read(rt_device_t dev, rt_base_t pin)//stm32_pin_read：就是一个函数名
“函数返回值类型”表示该指针变量可以指向具有什么返回值类型的函数；“函数参数列表”表示该指针变量可以指向具有什么参数列表的函数。这个参数列表中只需要写函数的参数类型即可。  
注：函数指针的定义就是将“函数声明”中的“函数名”改成“（* 指针变量名）”。但是这里需要注意的是：“<font color=Blue>（* 指针变量名）”</font>两端的括号不能省略，括号改变了运算符的优先级。如果省略了括号，就不是定义函数指针而是一个函数声明了，即声明了一个返回值类型为指针型的函数。

##RT-Thread源码简析之GPIO
应用程序通过 RT-Thread 提供的 PIN 设备管理接口来访问 GPIO，相关接口如下所示：

	  函数               	  描述
	  rt_pin_get()	        获取引脚编号  
	  rt_pin_mode()	 		设置引脚模式  
	  rt_pin_write()		设置引脚电平  
	  rt_pin_read()			读取引脚电平  
	  rt_pin_attach_irq()	绑定引脚中断回调函数  
	  rt_pin_irq_enable()	使能引脚中断  
	  rt_pin_detach_irq()	脱离引脚中断回调函数 

###Env中遇到的问题 
SCons Error: no such option: --exec-path  
还没解决！！

##时间2023.03.23
###PIN的API接口函数分析如何实现的
####1.rt_pin_get()	        获取引脚编号 
 
	rt_base_t rt_pin_get(const char *name)
	{
	    RT_ASSERT(_hw_pin.ops != RT_NULL);
	    RT_ASSERT(name[0] == 'P');
	
	    if(_hw_pin.ops->pin_get == RT_NULL)
	    {
	        return -RT_ENOSYS;
	    }
	
	    return _hw_pin.ops->pin_get(name);
	}
	FINSH_FUNCTION_EXPORT_ALIAS(rt_pin_get, pinGet, get pin number from hardware pin);
#####分析代码实现过程：

	_hw_pin.ops->pin_get(name)//从这个函数入手，这个ops前面分析过的！！简单说，前面的是4.0.2版本的，这个_stm32_pin_ops结构体变量中还没有pin_get成员。  
	_hw_pin.ops = ops;//_hw_pin.ops = ops;-->ops=_stm32_pin_ops;-->_hw_pin=_stm32_pin_ops;  
	rt_device_pin_register("pin", &_stm32_pin_ops, RT_NULL);//传参，往上推导

	rt_base_t (*pin_get)(const char *name);//新添结构体成员
	stm32_pin_get//新添结构体成员--->又引入到底层硬件驱动

	static rt_base_t stm32_pin_get(const char *name)
	{
	    rt_base_t pin = 0;int hw_port_num, hw_pin_num = 0;int i, name_len;
	
	    name_len = rt_strlen(name);
	
	    if ((name_len < 4) || (name_len >= 6))//这两句就是传入参数字符的个人规定PF.13，必须在4-6个字符之间
	        return -RT_EINVAL;
	    if ((name[0] != 'P') || (name[2] != '.'))//且第一个字符必须是P，第三个字符必须是.；满足之后在继续往下判断
	        return -RT_EINVAL;
	
	    if ((name[1] >= 'A') && (name[1] <= 'Z'))
	        hw_port_num = (int)(name[1] - 'A');//这也是他自己的规定，A是0，B是2，类推F是5
	    else
	        return -RT_EINVAL;
	
	    for (i = 3; i < name_len; i++)
	    {
	        hw_pin_num *= 10;//假如为9，一开始这还没赋值，hw_pin_num=0；下面赋值hw_pin_num=9；就是9
	        hw_pin_num += name[i] - '0';
	    }
		//假如是11，一开始这还没赋值，hw_pin_num=0；下面赋值hw_pin_num=1；就是1，在循环一次，hw_pin_num=1*10=10；hw_pin_num=10+1=11；引脚就为11；
	    pin = PIN_NUM(hw_port_num, hw_pin_num);	//这个关键，F和9共同判断，返回引脚号。
	    return pin;
		//#define PIN_NUM(port, no) (((((port) & 0xFu) << 4) | ((no) & 0xFu)))
	}
	####解释：pin_number = rt_pin_get("PF.9");
	检查用户的字符串是否符合长度范围。  
	检查用户的字符串是否符合规定格式。  
	检查用户的字符串是否符合规定格式，然后计算出hw_port_num。比如"PF.9"，我们通过'F'可以推算出hw_port_num为5。  
	计算出hw_pin_num，还是"PF.9"。我们通过'9'可以推算出hw_pin_num为9。  
	最后调⽤PIN_NUM得出最终的引脚编号将其返回。#define PIN_NUM(port, no) (((((port) & 0xFu) << 4) | ((no) & 0xFu)))   
	这里面看着很复杂，其实就是 port*16+no。比如：PA.5=0*16+5=5;PF.9=5*16+9;引脚编号
	里面的0xFu就是⼀个普普通通的掩码，port和no不能超过15。  
	通过这样，新版本的gpio驱动，少了宏定义，节省了代码量，也变得很整洁了。   

使用宏定义  
如果使用 rt-thread/bsp/stm32 目录下的 BSP 则可以使用下面的宏获取引脚编号：
GET_PIN(port, pin)   
获取引脚号为 PF9 的 LED0 对应的引脚编号的示例代码如下所示：  

	#define LED0_PIN        GET_PIN(F,  9)
这样我们通过rt_pin_get函数或者GET_PIN获取到了gpio的引脚编号。当我们拿到引脚号过后就可以对其进行各种操作了。


####2.rt_pin_mode()	 		设置引脚模式

rt_pin_mode(BEEP_PIN_NUM, PIN_MODE_OUTPUT);