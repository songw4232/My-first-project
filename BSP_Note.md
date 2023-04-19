

#RTT之ENV工具
BSP 工程所在的路径不要有中文或者空格存在。  

ENV工具，包，可以在RT-Thread官网找到相关的软件包，对于ENV软件，它的软件外设包都在里面的，使用menucongfig里面的软件包都能查到。

![](/figture/tu3.jpg)

![](/figture/tu4.jpg)
上面两个图，可以看到在官网找到需要的软件包的时候，RT-Thread online packages里面，可以看到分类，这个里面的内容是ENV软件自带的！！


1、构建工程，自动将源码添加到工程  
2、解决依赖，自动添加头文件到工程，这两个是通过scons脚本来实现  
3、系统裁减，自动生成宏定义到工程，这是通过Kconfig来完成。  

基本系统文件间的关系及作用如下图：
![](/figture/tu1.jpg)

1. 源码、头文件添加相关：SConsturct和SConscript 脚本。  
2. 图形配置相关：Kconfig 脚本。  
3. 项目工程生成相关：模板工程。  
4. 编译器配置、内核移植相关：rtconfig.py。  

这四部分构成了Env环境。
Kconfig和SConscript这两个文件和Env软件menuconfig图形化设置密切相关！！
###细分这几个文件具体作用 
####“menuconfig”  

第一个命令“menuconfig”，这个命令对于经常用Linux的肯定是很熟悉，这是个命令行内的图形化配置命令。实际就是根据Kconfig文件修改对应的.h文件内的宏定义（将宏定义用作开关一样，开关某个模块或软件包或MCU外设）。


menuconfig 是一种图形化配置工具，RT-Thread 使用其对整个系统进行配置、裁剪。

menuconfig 有多种类型的配置项，修改方法也有所不同，常见类型如下：  

开/关 型：使用空格键来选中或者关闭  
数值、字符串型：按下回车键后会出现对话框，在对话框中对配置项进行修改  

####保存配置  
选择好配置项之后按 ESC 键退出，选择保存修改即可自动生成 rtconfig.h 文件。此时再次使用 scons 命令就会根据新的 rtconfig.h 文件重新编译工程了。  

####3 个选项分别为：
使用 menuconfig -s 命令进入 Env 配置界面 

软件包自动更新功能：在退出 menuconfig 功能后，会自动使用pkgs --update命令来下载并安装软件包，同时删除旧的软件包。本功能在下载在线软件包时使用。  

自动创建 MDK 或 IAR 工程功能：当修改 menuconfig 配置后 ，必须输入 scons --target=xxx 来重新生成工程。开启此功能，就会在退出 menuconfig 时，自动重新生成工程，无需再手动输入 scons 命令来重新生成工程。  

使用镜像服务器下载软件包：由于大部分软件包目前均存放在 GitHub 上，所以在国内的特殊环境下，下载体验非常差。开启此功能，可以通过 国内镜像服务器 下载软件包，大幅提高软件包的下载速度和稳定性，减少更新软件包和 submodule 时的等待时间，提升下载体验。  


####[kconfig语法及实例](https://www.rt-thread.org/document/site/#/development-tools/build-config-system/Kconfig)重点！！重点！！重点！！一定要看！！！
![](/figture/tu2.jpg)

Kconfig在RT-Thread中的工作机制    
C语言项目的裁剪配置本质上通过条件编译和宏的展开来实现的，RT-Thread借助Kconfig这套机制更方便的实现了这一功能。当前以Windows下Env工具中的使用为例，简述Kconfig在RT-Thread的工作机制。  

Kconfig机制包括了Kconfig文件和配置UI界面（如menuconfig，pyconfig等）。Kconfig机制有如下特点：  
Kconfig文件中的配置项会映射至rtconfig.h中  
Kconfig文件可以随源码分散至各级子目录，便于灵活修改。  

Kconfig文件在源码中呈现树形结构，需要在工程的根目录下存在一份顶层Kconfig文件，顶层Kconfig文件在文件中通过source语句显示地调用各子目录下的Kconfig文件。Env在根目录下执行menuconfig命令后会递归解析各级Kconfig文件，然后提供如下配置界面，完成相应的配置后并保存，根目录下会存在一份.config文件保存当前选择的配置项，并将.config文件转为RT-Thread的系统配置文件rtconfig.h。  

##Kconfig语法：

####Kconfig 采用 # 作为注释标记符  

####config语句：

	config BSP_USING_GPIO #配置选项的名称，下面几行定义了该配置选项的属性
	    bool "Enable GPIO"  #类型，变量有5种类型，bool布尔类型变量的取值范围是 y/n 
	    select RT_USING_PIN  #select 是反向依赖关系的意思，即当前配置选项被选中，则 RT_USING_PIN 就会被选中。
	    default y   #默认值
	    help  #帮助信息
	    config gpio

config 表示一个配置选项的开始，紧跟着的 BSP_USING_GPIO 是配置选项的名称，config 下面几行定义了该配置选项的属性。属性可以是该配置选项的
![](/figture/tu7.jpg)
bool 表示配置选项的类型，每个 config 菜单项都要有类型定义，变量有5种类型
bool 布尔类型：布尔类型变量的取值范围是 y/n   
tristate 三态类型：三态类型变量的取值范围是 y、n 和 m。tristate 代表在内核中有三种状态，一种是不选中，一种是选中直接编译进内核，还有一种是 m 手动添加驱动，而布尔类型变量只有两种状态，即选中和不选中。其使用方法和布尔类型变量类似。    
string 字符串：字符串变量的默认值是一个字符串     
hex 十六进制：十六进制类型变量的取值范围是一个十六进制的数，其使用方法和整型变量用法一致，整型变量生成的是十进制的数，而十六进制生成的是十六进制的数。  
int 整型：整型变量的取值范围是一个整型的数   

通过 env 选中以上配置界面的选项后，最终可在 rtconfig.h 文件中生成如下两个宏

	#define RT_USING_PIN
	#define BSP_USING_GPIO

####menu/endmenu语句
	menu "Hardware Drivers Config"
	    config BSP_USING_COM2
	        bool "Enable COM2 (uart2 pin conflict with Ethernet and PWM)"
	        select BSP_USING_UART
	        select BSP_USING_UART2
	        default n
	    config BSP_USING_COM3
	        bool "Enable COM3 (uart3 pin conflict with Ethernet)"
	        select BSP_USING_UART3
	        default n
	endmenu
menu 之后的字符串是菜单名。menu 和 endmenu 间有很多 config 语句，以上代码对应的配置界面如下所示
![](/figture/tu5.jpg)

![](/figture/tu6.jpg)
通过 env 选中以上配置界面的所有选项后，最终可在 rtconfig.h 文件中生成如下五个宏

	#define BSP_USING_UART
	#define BSP_USING_UART2
	#define BSP_USING_UART3
	#define BSP_USING_COM2
	#define BSP_USING_COM3

####menuconfig语句
menuconfig 语句表示带菜单的配置项

	menu "Hardware Drivers Config"
	    menuconfig BSP_USING_UART
	        bool "Enable UART"
	        default y
	        select RT_USING_SERIAL
	        if BSP_USING_UART
	            config BSP_USING_UART1
	                bool "Enable UART1"
	                default y
	
	            config BSP_UART1_RX_USING_DMA
	                bool "Enable UART1 RX DMA"
	                depends on BSP_USING_UART1 && RT_SERIAL_USING_DMA
	                default n
	        endif
	endmenu

语句分析：  
menuconfig 这个语句和 config 语句很相似，但它在 config 的基础上要求所有的子选项作为独立的行显示。  
depends on 表示依赖某个配置选项，depends on BSP_USING_UART1 && RT_SERIAL_USING_DMA 表示只有当 BSP_USING_UART1 和 RT_SERIAL_USING_DMA 配置选项同时被选中时，当前配置选项的提示信息才会出现，才能设置当前配置选项  

####choice/endchoice语句
choice 语句将多个类似的配置选项组合在一起，供用户选择一组配置项  
	
	menu "Hardware Drivers Config"
	    menuconfig BSP_USING_ONCHIP_RTC
	        bool "Enable RTC"
	        select RT_USING_RTC
	        select RT_USING_LIBC
	        default n
	        if BSP_USING_ONCHIP_RTC
	            choice
	                prompt "Select clock source"
	                default BSP_RTC_USING_LSE
	
	                config BSP_RTC_USING_LSE
	                    bool "RTC USING LSE"
	
	                config BSP_RTC_USING_LSI
	                    bool "RTC USING LSI"
	            endchoice
	        endif
	endmenu
语句解析：  
choice/endchoice 给出选择项，中间可以定义多个配置项供选择，但是在配置界面只能选择一个配置项；
prompt 给出提示信息，光标选中后回车进入就可以看到多个 config 条目定义的配置选项。

####comment语句
comment 语句出现在界面的第一行，用于定义一些提示信息。  
comment 代码示例如下  

	menu "Hardware Drivers Config"
	    comment "uart2 pin conflict with Ethernet and PWM"
	    config BSP_USING_COM2
	        bool "Enable COM2"
	        select BSP_USING_UART
	        select BSP_USING_UART2
	        default n
	endmenu


####source语句
source 语句用于读取另一个文件中的 Kconfig 文件，如：

	source "../libraries/HAL_Drivers/Kconfig"

上述语句用于读取当前 Kconfig 文件所在路径的上一级文件夹 libraries/HAL_Drivers 下的 Kconfig 文件。



最外层的kconfig文件

	source "$RTT_DIR/Kconfig"
	source "$PKGS_DIR/Kconfig"
	source "../libraries/Kconfig"
	source "board/Kconfig"
那现在就知道了用于读取另一个文件中的 Kconfig 文件  

board文件夹板级支持外设的kconfig文件：供图形化选择的

	menu "Onboard Peripheral Drivers"#大条件，板载外设，如：Enable LCD (spi3)                                                                   Enable LVGL for LCD；Enable SDCARD (spi1)； Enable icm20608 (i2c3)
	
	    config BSP_USING_STLINK_TO_USART
	        bool "Enable STLINK TO USART (uart1)"
	        select BSP_USING_UART
	        select BSP_USING_UART1
	        default y	
		config BSP_USING_KEY
	        bool "Enable onboard keys"
	        select RT_USING_PIN
	        select RT_USING_TIMER_SOFT
	        select PKG_USING_MULTIBUTTON
	        default n	

	menu "On-chip Peripheral Drivers"#大条件，里面主要是片上外设，比如IIC、SPI、PWN、TIMER等，最开始最基础的！！
	
	    config BSP_USING_GPIO
	        bool "Enable GPIO"
	        select RT_USING_PIN
	        default y
	
	    menuconfig BSP_USING_UART
	        bool "Enable UART"
	        default y
	        select RT_USING_SERIAL
	        if BSP_USING_UART

	menu "Board extended module Drivers"#大条件，外部扩展
	
	    menuconfig BSP_USING_AT_ESP8266
	        bool "Enable ESP8266(AT Command, COM2)"
	        default n
	        select BSP_USING_COM2
	        select PKG_USING_AT_DEVICE
	        select AT_DEVICE_USING_ESP8266
	        select AT_DEVICE_ESP8266_SAMPLE
	        select AT_DEVICE_ESP8266_SAMPLE_BSP_TAKEOVER

注：我现在移植BSP这个部分menu "On-chip Peripheral Drivers"#大条件，里面主要是片内外设，比如IIC、SPI、PWN、TIMER等，最开始最基础的！！实现这些功能！！

再来看看这段Kconfig代码对应的图形配置界⾯！！

####SConscript依赖实现：
__SConstruct与SConscript是SCons的脚本文件，它们共同配合，一起构建工程。一般一个工程中只有唯一的一个SConctruct是，而SConscript可以有很多个。SConctruct有点类似于一个总的枝干包含所有的SConscript，而SConscript是枝杈，一般一个SConscript就是负责把一个独立的模块添加到工程中（比如外设库）。__

####功能：自己写的驱动，加入工程中

在SConscript文件中，添加如下代码！！注意：文件夹名称和驱动文件名称必须要在SConscript文件中添加正确！！不然搜索不到路径，不报错，但加不到工程中！！

	if GetDepend(['BSP_USING_KEY']):
    	src += Glob('ports/drv_key.c')


##KEY按键软件包的使用（这些有文档更详细）

首先我们要使用按键功能，先去查找⼀下，有哪些适合我们使用的软件包。
button软件包本身包含哪些文件？button软件包，怎么添加到工程中去的？
flexible_button.c和.h文件加一个demo.c文件，一个是按键驱动，一个是按键使用demo
####如何使用flexible_button软件包   

#####第一步：
在ENV软件中，先打开软件包的Kconfig文件，看一下他的图形化界面大概是个什么样子的，需要定义哪些变量才能使用配置相关函数。比如：在flexible_button软件包Kconfig文件里

	menuconfig PKG_USING_FLEXIBLE_BUTTON
	    bool "FlexibleButton: Small and flexible button driver"
	    default n
	
	if PKG_USING_FLEXIBLE_BUTTON
	
	    config PKG_FLEXIBLE_BUTTON_PATH
	        string
	        default "/packages/misc/FlexibleButton"
	
	    config PKG_USING_FLEXIBLE_BUTTON_DEMO
	        bool "Enable flexible button demo"
	        default n
在这里，大家可以看到有两个⽐较重要的宏
  
	PKG_USING_FLEXIBLE_BUTTON ： 使能该软件包    
	PKG_USING_FLEXIBLE_BUTTON_DEMO ：使能该软件包的示例代码  
#####第二步：
所以当我们要使⽤这个软件包的时候，想要把它加进我们⾃⼰的BSP，注意这两个宏就行了。  

	正点原子是这样写的！！
	menu "Onboard Peripheral Drivers"#板载外设
			  
			config BSP_USING_KEY  #定义#define BSP_USING_KEY
		        bool "Enable onboard keys"
		        select RT_USING_PIN  #依赖项么，跟着定义
		        select RT_USING_TIMER_SOFT
		        select PKG_USING_MULTIBUTTON
		        default n
	#define BSP_USING_KEY，因为很多代码都是这样写的#ifdef  BSP_USING_KEY ...endif;只有你定义了这个才能进入条件使用
	PKG_USING_MULTIBUTTON，正点原子使用的是另一个BUTTON包！！不一样，自己会看需要那个定义函数变量。
	
在Kconfig文件中仿真写
	menu "Onboard Peripheral Drivers"

	config BSP_USING_KEY
			bool "Enable onboard keys"
	        select RT_USING_PIN
	        select RT_USING_TIMER_SOFT
	        select PKG_USING_FLEXIBLE_BUTTON
			select PKG_USING_FLEXIBLE_BUTTON_DEMO
	        default n

scons工具重新编译，在MDK5中编译，有错误，毕竟这个只是demo⽂件，有错误很正常，也不⼀定能跑起来，我们后⾯会更改为适合我们并且能跑起来的BSP驱动⽂件。修改相关变量名就可以使用了！！

#####这个软件包毕竟是别人的，我们下次要使用的话，还的添加，修改里面的代码，不方便。所以，下面我们要把它添加成自己的驱动文件  
#####第一步：同上
#####第二步：
自己的驱动文件写道board里面，建立文件夹ports/drv_key.c文件
#####第三步：
还有一个重要的文件不可少！！SConscript文件正点原子的文件ports/drv_key.c同一级下的SConscript文件，和scons有关，构建用的！！
PKG_USING_FLEXIBLE_BUTTON_DEMO文件内容复制到ports/drv_key.c中（注意上面的select PKG_USING_FLEXIBLE_BUTTON_DEMO删除）
#####第四步：
然后将ports/drv_key.c添加到工程中：  
board文件夹下SConscript文件，它是Scons脚本文件，就是通过编辑SConscript文件，使用scons编译，把button软件包相关文件添加到工程中，（至于头文件还要不要添加不知道，还没实干过）
在这个sconscript这个⽂件⾥⾯，我们可以通过配置的宏定义往⼯程⾥⾯添加⽂件。   

	if GetDepend(['BSP_USING_KEY']):
	    src += Glob('ports/drv_key.c')
这个SConscript文件语法还不太明白！！
这里动用了两个SConscript文件！！注意！！
​
最后我们再使⽤menuconfig配置工程。	  
完成！！不难！！  

##UART驱动的使⽤
前⾯我们所讲的按键，是基于PIN框架的一个实际应用，并且前⾯我们是默认配置了USART1的，由于板
子上的USART不具体驱动某一个外设，这⾥我们仅仅添加到我们的片上外设中。 
###第一步：
	menuconfig BSP_USING_UART#菜单
	        bool "Enable UART"#类型，也是显示的
	        default y
	        select RT_USING_SERIAL#依赖项
	        if BSP_USING_UART#条件
	            config BSP_USING_UART1#配置选项的名称
	                bool "Enable UART1"#类型
	                default y#默认值
	
	            config BSP_UART1_RX_USING_DMA
	                bool "Enable UART1 RX DMA"
	                depends on BSP_USING_UART1 && RT_SERIAL_USING_DMA
	                default n

这个就只是配置了USART1。我们这⾥添加USART2，仿照写！！  
我这样直接配置能不能⽤？？？？

###第二步：
大家都知道，像串口外设，⼀般引脚都是可以复用的，不管是我们前面框架分析，还是这一节，都没有
看到引脚配置的身影，相必，如果没有引脚配置，我们这个外设肯定驱动不起来，而且，⼤家配置裸机
都知道，我们还需要配置外设的时钟，⼤家来看⼀下那个Studio软件。   

重点：配置这个MspInit这个函数的！！！通过Cube MX配置！！USART2，直接使能串口，引脚看一下是否合适，其他不需要配置！！  
然后生成工程，这里不需要设置单独.c文件！！
在rt-thread-v4.1.0\bsp\stm32\stm32l475-rtt-pandora\board\CubeMX_Config\Src\stm32l4xx_hal_msp.c中！！生成相关文件，是一些初始化文件！！使用Cube MX软件本来就是初始化的！！！

	/* Peripheral clock enable */
    __HAL_RCC_USART2_CLK_ENABLE();

    __HAL_RCC_GPIOA_CLK_ENABLE();
    /**USART2 GPIO Configuration
    PA2     ------> USART2_TX
    PA3     ------> USART2_RX
    */
    GPIO_InitStruct.Pin = GPIO_PIN_2|GPIO_PIN_3;
    GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_VERY_HIGH;
    GPIO_InitStruct.Alternate = GPIO_AF7_USART2;
    HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

###第三步：
这样子，我们就可以使⽤USART2了。我们还是准备⼀个驱动代码吧。
（驱动代码！！驱动！！有点理解了什么叫驱动了，上面的按键DEMO就是一个驱动文件）上面的操作叫初始化！！

找一个：程序清单：这是⼀个串口设备使用例程
在board\ports\drv_usart2_test.c文件，drv_usart2_test.c作为驱动文件！！使用例程拷贝进去
作为驱动文件
![](/figture/tu8.jpg)

###第四步：
然后将ports/drv_usart2_test.c添加到工程中：  
board文件夹下SConscript文件，它是Scons脚本文件，就是通过编辑SConscript文件，使用scons编译，把button软件包相关文件添加到工程中，（至于头文件还要不要添加不知道，还没实干过）
在这个sconscript这个⽂件⾥⾯，我们可以通过配置的宏定义往⼯程⾥⾯添加⽂件。   

	if GetDepend(['BSP_USING_KEY']):
	    src += Glob('ports/drv_key.c')
	
	if GetDepend(['BSP_USING_UART2']):
	    src += Glob('ports/drv_usart2_test.c')  

这样我们的USART2外设就可以⽤了。  
后⾯基本所有的外设都是这么配置，千万不要忘记配置MspInit这个函数哟！！！！！


##LCD驱动
先总结：四个步骤  
Kconfig配置  
添加相应外设的MspInit函数  
编写驱动文件  
添加到BSP的borad级驱动  

现在才后知后觉，这个STM32L475的Pandora开发板上面的TFT_LCD的显示屏不是什么8080并口的！！看引脚不就知道了！！以为啥LCD屏都是同一个接口通信方式呢！！看了原理图了，就应该知道了，没仔细去看原理图！！这里是SPI通信方式！！之前还说为什么只配SPI和PWM呢，为什么不去配置像8080并口传输数据的8个引脚端口呢，仔细去看一下LCD的原理图，哪来的传输数据的8个引脚！！
看一下原理图   

	spi通信（spi3）  
	pwm调背光（pwm4 ch2）  
所以我们需要实现这两个外设驱动。  
####第一步：Kconfig配置
在Kconfig里面的片上外设（不是板载外设，LCD在板载外设）中配置SPI和PWM

	menuconfig BSP_USING_SPI#使用了SPI3，后面要用1，仿照一样的配置
	        bool "Enable SPI BUS"
	        default n
	        select RT_USING_SPI
	        if BSP_USING_SPI      
	            config BSP_USING_SPI3
	                bool "Enable SPI3 BUS"
	                default n
	
	            config BSP_SPI3_TX_USING_DMA
	                bool "Enable SPI3 TX DMA"
	                depends on BSP_USING_SPI3
	                default n
	                
	            config BSP_SPI3_RX_USING_DMA
	                bool "Enable SPI3 RX DMA"
	                depends on BSP_USING_SPI3
	                select BSP_SPI3_TX_USING_DMA
	                default n
	        endif

	menuconfig BSP_USING_PWM#（pwm4 ch2）自己查原理图，需要哪个PWM，哪个通道
	        bool "Enable PWM"
	        default n
	        select RT_USING_PWM
	        if BSP_USING_PWM
	        menuconfig BSP_USING_PWM4
	            bool "Enable timer4 output PWM"
	            default n
	            if BSP_USING_PWM4
	                config BSP_USING_PWM4_CH1
	                    bool "Enable PWM4 channel1"
	                    default n
	
	                config BSP_USING_PWM4_CH2
	                    bool "Enable PWM4 channel2"
	                    default n	            
	            endif
	        endif

	menuconfig BSP_USING_TIM#PWM会涉及到定时器！！
	        bool "Enable timer"
	        default n
	        select RT_USING_HWTIMER
	        if BSP_USING_TIM
	            config BSP_USING_TIM15
	                bool "Enable TIM15"
	                default n
	        endif

###第二步：添加相应外设的MspInit函数 

就是使用Cube MX来生成相关初始化 
spi,定时器设置PWM通道等，

上面的还有没配置完成，在Kconfig里面添加LCD相关的配置，刚刚只是片上外设的配置，LCD所需要的一些外设配置，LCD自己的还没有配置
在板载外设里面添加SPI和PWM等！！

	config BSP_USING_SPI_LCD
	        bool "Enable LCD (spi3)"
	        select BSP_USING_SPI
	        select BSP_USING_SPI3
	        select BSP_SPI3_TX_USING_DMA
	        select BSP_USING_PWM
	        select BSP_USING_PWM4
	        select BSP_USING_PWM4_CH2
	        default n
	
	    config BSP_USING_LCD_SAMPLE
	        bool "Enable LCD raw driver sample"
	        depends on BSP_USING_SPI_LCD
	        default n
	
	    config BSP_USING_LCD_QRCODE
	        bool "Enable LCD to show QRCode"
	        depends on BSP_USING_SPI_LCD
	        select BSP_USING_LCD_SAMPLE
	        select PKG_USING_QRCODE
	        default n

在这⾥，我们仅仅只是配置了外设，还是需要⼀个外设驱动⽂件，原⼦已经写好了，搞过来。


####第三步：编写驱动文件 
复制原子的。


####第四步：添加到BSP的borad级驱动 

在总的SConstruct总的文件夹里添加
	objs.extend(SConscript(os.path.join(os.getcwd(), 'board', 'ports', 'SConscript')))
	 'board', 'ports', 文件夹自己新添加的!!
