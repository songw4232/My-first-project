

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

注：只是再重新scons一下并不能添加到工程中，必须再进入menuconfig设置一下！！也可能我的编译方式不对！！



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

注：自己实操的时候，定时器PWM的时钟没有开启，导致屏幕不亮，查错查了半天，这个Cube MX的配置一步都不能错！！细心！！

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


##STM32U575系列移植
####遇到的问题Cube MX重新生成core文件夹，替换掉。会出现Error_Handler()函数在*_msp.c文件下报错的问题
	.\build\keil\Obj\rtthread.axf: Error: L6218E: Undefined symbol Error_Handler (referred from stm32u5xx_hal_msp.o).
	解决办法：（在官方仓库中看见的！！这仓库的东西够我学一年）
	[stm32] 解决Error_Handler()函数在*_msp.c文件下报错的问题
	在用户头文件区增加#include <drv_common.h>即可

####添加PWM外设遇到的问题
在BSP移植STM32U5系列的，因为这个板子可能不常见，官方的库不全面，没有包含SOC_SERIES_STM32F4像这个一样的宏定义，单片机时会出现各种各样的错误！！！这里肯定需要的是SOC_SERIES_STM32U5！！上面没有，不知道进入哪个条件，也可能进错条件了！！

	#if defined(SOC_SERIES_STM32F2) || defined(SOC_SERIES_STM32F4) || defined(SOC_SERIES_STM32F7)
	    if (htim->Instance == TIM9 || htim->Instance == TIM10 || htim->Instance == TIM11)
	#elif defined(SOC_SERIES_STM32L4) || defined(SOC_SERIES_STM32H7)|| defined(SOC_SERIES_STM32F3)
	    if (htim->Instance == TIM15 || htim->Instance == TIM16 || htim->Instance == TIM17)
	#elif defined(SOC_SERIES_STM32MP1)
	    if (htim->Instance == TIM4)
	#elif defined(SOC_SERIES_STM32F1) || defined(SOC_SERIES_STM32F0) || defined(SOC_SERIES_STM32G0)
	    if (0)
	#endif
	    {
	#if !defined(SOC_SERIES_STM32F0) && !defined(SOC_SERIES_STM32G0)
	        tim_clock = (rt_uint32_t)(HAL_RCC_GetPCLK2Freq() * pclk2_doubler);
	#endif
	    }
	    else
	    {
	        tim_clock = (rt_uint32_t)(HAL_RCC_GetPCLK1Freq() * pclk1_doubler);
	    }
解决方法：

	#if defined(SOC_SERIES_STM32F2) || defined(SOC_SERIES_STM32F4) || defined(SOC_SERIES_STM32F7)
	    if (htim->Instance == TIM9 || htim->Instance == TIM10 || htim->Instance == TIM11)
	#elif defined(SOC_SERIES_STM32L4) || defined(SOC_SERIES_STM32H7)|| defined(SOC_SERIES_STM32F3) ||defined(SOC_SERIES_STM32U5)
	    if (htim->Instance == TIM15 || htim->Instance == TIM16 || htim->Instance == TIM17)
	#elif defined(SOC_SERIES_STM32MP1)
	    if (htim->Instance == TIM4)
	#elif defined(SOC_SERIES_STM32F1) || defined(SOC_SERIES_STM32F0) || defined(SOC_SERIES_STM32G0)
	    if (0)

####STM32U575系列移植ADC

       .Instance                   = ADC1,                          \
     /*  .Init.ClockPrescaler        = ADC_CLOCK_SYNC_PCLK_DIV4,    */ \
       .Init.Resolution            = ADC_RESOLUTION_12B,            \

\这个是啥，一行还没结束！！然后今天被坑了！！keil中出现绿色的，我以为是屏蔽了！！结果是语法错误！！这个确实想当然了！！

	#if defined(SOC_SERIES_STM32L4) || defined(SOC_SERIES_STM32G0) || defined (SOC_SERIES_STM32MP1) || defined(SOC_SERIES_STM32H7) || defined (SOC_SERIES_STM32WB) || defined(SOC_SERIES_STM32U5)
	        ADC_Enable(stm32_adc_handler);
	#else
	        __HAL_ADC_ENABLE(stm32_adc_handler);
	#endif
	    }
	    else
	    {
	#if defined(SOC_SERIES_STM32L4) || defined(SOC_SERIES_STM32G0) || defined (SOC_SERIES_STM32MP1) || defined(SOC_SERIES_STM32H7) || defined (SOC_SERIES_STM32WB)|| defined(SOC_SERIES_STM32U5)
	        ADC_Disable(stm32_adc_handler);
	#else
	        __HAL_ADC_DISABLE(stm32_adc_handler);
	#endif

不是有点明显啊！！ 第一步：一开始是都有这个defined(SOC_SERIES_STM32U5)，然后我就想着应该是少了这个吧！！第二步：然后这个代码他报错！__HAL_ADC_ENABLE(stm32_adc_handler);，那我就想着用上面一个呗！！然后就好了！应该对吧！！


####STM32U575系列移植SPI，测试使用007
bug是别人帮忙改的，学着加强改bug能力！！

	#if defined(SOC_SERIES_STM32F2) || defined(SOC_SERIES_STM32F4) || defined(SOC_SERIES_STM32F7)
	            spi_bus_obj[i].dma.handle_rx.Init.Channel = spi_config[i].dma_rx->channel;
	#elif defined(SOC_SERIES_STM32L4) || defined(SOC_SERIES_STM32G0) || defined(SOC_SERIES_STM32MP1) || defined(SOC_SERIES_STM32WB) || defined(SOC_SERIES_STM32H7)
	            spi_bus_obj[i].dma.handle_rx.Init.Request = spi_config[i].dma_rx->request;
	#endif
						#if 0
	            spi_bus_obj[i].dma.handle_rx.Init.Direction           = DMA_PERIPH_TO_MEMORY;
	            spi_bus_obj[i].dma.handle_rx.Init.PeriphInc           = DMA_PINC_DISABLE;//这一部分代码报错，原因没有定义
	            spi_bus_obj[i].dma.handle_rx.Init.MemInc              = DMA_MINC_ENABLE;
	            spi_bus_obj[i].dma.handle_rx.Init.PeriphDataAlignment = DMA_PDATAALIGN_BYTE;
	            spi_bus_obj[i].dma.handle_rx.Init.MemDataAlignment    = DMA_MDATAALIGN_BYTE;
	            spi_bus_obj[i].dma.handle_rx.Init.Mode                = DMA_NORMAL;
	            spi_bus_obj[i].dma.handle_rx.Init.Priority            = DMA_PRIORITY_HIGH
						#endif

别人帮我改好了，才发觉。现在看来，好像一开始的思路有点问题！！代码报错的部分，原因没有定义！我的思路是应该和上面修改PWM和ADC一样，在适当的地方加一个defined(SOC_SERIES_STM32U5)，但是你仔细看这里的报错和defined(SOC_SERIES_STM32U5)if的宏定义没有关系的，#endif已经结束了！！
那现在最简单的改法，这是DMA现在用不到，那就屏蔽掉！！

	ArmClang: error: no such file or directory: '../libraries/STM32U5xx_HAL/STM32U5xx_HAL_Driver/Src/stm32u5xx_hal_qspi.c'
没有添加这个qspi，但是现在不需要这个，就去屏蔽掉，去哪找，去哪屏蔽！！关键点
用VScode查找stm32u5xx_hal_qspi这个在哪！然后你要确定是哪个文件的包含stm32u5xx_hal_qspi才把这个加入到工程里面的，但是这里面并没有这个文件而报错！
![](/figture/tu9.jpg)
这个文件在Sconcript脚本文件！！

	warning: redefinition of typedef 'socklen_t' is a C11 feature [-Wtypedef-redefinition]
	警告：typedef 'socklen_t' 的重定义是 C11 功能[-Wtypedef-redefinition]

####查找快捷键ctrl+f
####多任务切换 win+tab

####5.0版本RW007软件包移植

遇到了之前的一个问题！！

	.\build\keil\Obj\rtthread.axf: Error: L6218E: Undefined symbol Error_Handler (referred from stm32u5xx_hal_msp.o).
	[stm32] 解决Error_Handler()函数在*_msp.c文件下报错的问题
	在用户头文件区增加#include <drv_common.h>即可
	Error_Handler在drv_common.h文件里，所以加这个头文件！！
	
	我的也是Undefined symbol rt_hw_spi_device_attach，我就找在那个文件里面，#include <drv_spi.h>这里，但是已经加过了！！又
	是啥原因呢！！然后我发现drv_spi.c里面的rt_hw_spi_device_attach是灰色的！就是没使用的意思！！我才意识到是宏定义没定义，没使
	用这个rt_hw_spi_device_attach，然后意识到没初始化这个SPI1，menuconfig里面没有配置！！

##STM32U5系列BSP移植总结（重点重点超重点！！！）

具体说一下，我这两周通过BSP添加了STM32U575板子的外设！u575是新板子么，RT-Thread的BSP库里面之前也没人使用过，所以需要人移植，添加一些驱动，供人使用。  
好！那么没有人用过，添加BSP时，就不可能添加的移植代码百分百的适配这个板子！这一点能理解吧！！所以，遇到什么问题，就要去解决它，使它适配u575的开发板。  
上面也说了一些添加按键软件包、串口2、ADC1、PWM、IIC、SPI等外设时，所遇到的问题，及其简单的处理bug,但上面的bug虽然在这个问题下解决了，但是却影响了整个RT-Thread代码里面BSP/stm32里面的代码中其他工程！！！如果是这样解决的问题，肯定是不能这样处理的，打个比方来说：
STM32U575系列移植SPI，测试使用007

	#if defined(SOC_SERIES_STM32F2) || defined(SOC_SERIES_STM32F4) || defined(SOC_SERIES_STM32F7)
	            spi_bus_obj[i].dma.handle_rx.Init.Channel = spi_config[i].dma_rx->channel;
	#elif defined(SOC_SERIES_STM32L4) || defined(SOC_SERIES_STM32G0) || defined(SOC_SERIES_STM32MP1) || defined(SOC_SERIES_STM32WB) || defined(SOC_SERIES_STM32H7)
	            spi_bus_obj[i].dma.handle_rx.Init.Request = spi_config[i].dma_rx->request;
	#endif
						#if 0
	            spi_bus_obj[i].dma.handle_rx.Init.Direction           = DMA_PERIPH_TO_MEMORY;
	            spi_bus_obj[i].dma.handle_rx.Init.PeriphInc           = DMA_PINC_DISABLE;//这一部分代码报错，原因没有定
						#endif
那现在最简单的改法，这是DMA现在用不到，那就屏蔽掉！！那这样处理确实不报错了！但是这个drv_spi.c文件并不是stm32u575的专有文件，这边直接屏蔽了，那么其他文件想要使用就使用不了了！！这一点需要了解！如何解决是重点！！一开始，听别人的，这是一些DMA有关的文件，找一个宏定义一下就行，我找的是有关DMA的，如果打开了，还是不对！！那就是最大的宏，应该使用了U５７５就会报错，那么，就是如果没有定义它，就把它（这部分代码）打开，如果定义了就屏蔽掉，这样处理就不会影响到其他代码了！！是不是很神奇！！以后修改代码就是这个思路！！　　

	#ifndef SOC_SERIES_STM32U5
	            spi_bus_obj[i].dma.handle_rx.Init.Direction           = DMA_PERIPH_TO_MEMORY;
	            spi_bus_obj[i].dma.handle_rx.Init.PeriphInc           = DMA_PINC_DISABLE;
	            spi_bus_obj[i].dma.handle_rx.Init.MemInc              = DMA_MINC_ENABLE;
	#endif

###所以下面是STM32U5系列BSP移植时，遇到的全部报错情况及大佬的修改

###1.使用ADC时出现的报错情况分析
最起码的知道是什么错误，而不是一味的去屏蔽这个地方的错误，这样即使把错误解决了，代码也可能功能上出错了！！  
####错误一：  

	#define ADC1_CONFIG                                                 \
	    {                                                               \
	       .Instance                   = ADC1,                          \
	       .Init.ClockPrescaler        = ADC_CLOCK_SYNC_PCLK_DIV4,      \把这个换成下面的代码，因为u5里面没有这个宏定义
	       .Init.ClockPrescaler        = ADC_CLOCK_ASYNC_DIV4,          \
	       .Init.Resolution            = ADC_RESOLUTION_12B,            \
	       .Init.DataAlign             = ADC_DATAALIGN_RIGHT,           \
	       .Init.ScanConvMode          = ADC_SCAN_DISABLE, 

（1）ADC1_CONFIG一开始是这个报错，原因是`.Init.ClockPrescaler        = ADC_CLOCK_SYNC_PCLK_DIV4,`这一行的代码没有定义！！一开始的想法是直接把这一行的代码删掉，但是总觉得不妥！！咋可能无缘无故的去删除源库里面的代码，不妥不妥！！  
（2）但是，这个问题去交给我解决的话，我估计是搞个好几周都不一定解决的了的！所以看大佬改BUG真的可以学到东西的，因为这行代码没有定义，先看这行代码是什么，为什么找不到定义，这是一个ADC四分频的宏定义！难道是说这个stm32u575的ADC不用分频的？这是最开始的猜测。所以，把这行代码删掉就合理！！但是，可能性不大，这个单片机不太可能没有ADC分频的。  
（3）现在是大佬的表演时间！现在就去VScode里面去找一些`ADC_CLOCK_SYNC_PCLK_DIV4`这个相似的定义，看看这个单片机是不是真的没有ADC分频的相关定义，所以，找`_PCLK_DIV4`一半，找相似的定义，这个凭运气嘛，因为库没人能看完，看完也记不住这些的，就是去搜索，去看看相似的定义，从而找到解决问题的方法！！从上帝视角看，这个搜索，也没有什么发现，大佬有搜索这个`SYNC`，我的话，肯定想不到这个，都不知道这个的含义，就发现了这个宏`ADC_CLOCK_ASYNC_DIV4`，那找到它的时候大佬就已经明白了，对比源代码的宏，多一个A，基本功：异步，异的意思，就能想到这个stm32u575单片机没有同步分频的功能，没有这个宏定义么，所以只有异步的功能，把它修改成这个宏就行！！  
（4）我一开始还担心，如果把这个代码修改了，会不会和上面的问题一样，会影响到其他文件啊，大佬看了一下文件所在的具体位置，是u5专用的！！不会影响的。问题解决  

####错误二：
 	vref_mv = __LL_ADC_CALC_VREFANALOG_VOLTAGE(vref_value, stm32_adc_handler->Init.Resolution);
报错！！一开始我的解决问题是屏蔽了就好了！！不能这么修改代码，简单的屏蔽了，ADC的功能就可能实现不了了，怪不得我的验证ADC测量电压一直为零！！不就能说明问题。  
我想，我稍微的去看一下这个代码的函数定义就能发现它错在哪了！！能本就不用去看报错原因！！可是我没有看！可能是个态度问题吧，这个需要改！！是函数参数不对！！
（1）大佬的解决问题，这个我肯定一下子学不来！看函数定义缺少哪个或者那几个参数，添加进来就行，我现在的功力达不到

	#ifdef SOC_SERIES_STM32U5
	    vref_mv = __LL_ADC_CALC_VREFANALOG_VOLTAGE(stm32_adc_handler->Instance, vref_value, stm32_adc_handler->Init.Resolution);
	#else
	    vref_mv = __LL_ADC_CALC_VREFANALOG_VOLTAGE(vref_value, stm32_adc_handler->Init.Resolution);
	#endif

这里就有一个非常大非常大的坑！！我感觉一般人搞不定的，源代码现在没打开，条件不允许啊！不知道为啥，这个文件并不是像上面的文件一样，只是u5专用，这个文件时drv_adc.c文件，是所有stm32芯片公用的！这里如果直接把`vref_mv = __LL_ADC_CALC_VREFANALOG_VOLTAGE(vref_value, stm32_adc_handler->Init.Resolution);`修改成`vref_mv = __LL_ADC_CALC_VREFANALOG_VOLTAGE(stm32_adc_handler->Instance, vref_value, stm32_adc_handler->Init.Resolution);`会不会影响其他文件，难道还是说这个代码是源代码里面的BUG，代码更新，u5里面的代码没有及时更新，导致参数输入错误，很有可能！！  
（2）资深大佬可不是仅靠猜测来修改代码的，他去看看别的stm32其他系列单片机里面的这个函数是否也是两个参数，查了一下，是，那就把U5单独提出来做修改，修改如上，经典思路，谨记谨记  
（3）我觉得其他stm32文件的`__LL_ADC_CALC_VREFANALOG_VOLTAGE(stm32_adc_handler->Instance, vref_value, stm32_adc_handler->Init.Resolution);`这个函数，会不会也是没更新，使用时也会报错，这个因为没使用，现在条件不允许，会验证！！这个以后再说  

###2使用SPI1驱动flash中遇到的错误

####错误一：
	#ifndef SOC_SERIES_STM32U5
	            spi_bus_obj[i].dma.handle_tx.Init.Direction           = DMA_MEMORY_TO_PERIPH;
	            spi_bus_obj[i].dma.handle_tx.Init.PeriphInc           = DMA_PINC_DISABLE;
	            spi_bus_obj[i].dma.handle_tx.Init.MemInc              = DMA_MINC_ENABLE;
	            spi_bus_obj[i].dma.handle_tx.Init.PeriphDataAlignment = DMA_PDATAALIGN_BYTE;
	            spi_bus_obj[i].dma.handle_tx.Init.MemDataAlignment    = DMA_MDATAALIGN_BYTE;
	            spi_bus_obj[i].dma.handle_tx.Init.Mode                = DMA_NORMAL;
	            spi_bus_obj[i].dma.handle_tx.Init.Priority            = DMA_PRIORITY_LOW;
	#endif
（1）有两处dma的地方，没有定义报错，我上面的修改，是最简单的一种，但是，我现在忽然想到，这里的该法是因为我并没有使用DMA，这里DMA报错，那我直接屏蔽掉就行，如果使用u5就把它屏蔽，如果没有定义u5就可以使用，可取。
（2）但是现在，那如果我要在U5里面使用DMA呢，把屏蔽了，算咋回事，这出的修改BUG有待改良！！
有时间可以自己去解决！！和`.Init.ClockPrescaler        = ADC_CLOCK_ASYNC_DIV4`解决方法一样！！

###错误二：
	src += ['STM32U5xx_HAL_Driver/Src/stm32u5xx_hal_qspi.c']没有找到这个qspi文件！
（1）这个BUG另一个大佬帮解决的！！它是如何找到，并屏蔽它的，前面说的有，也是使用VScode在整个文件里面去找的，然后靠自己的积累，能力的体现的，去找到报错产生的可能的位置，并把它屏蔽！！必须有点技术啊，这个文件是scons编译加载进来的，然后没有这个文件报错，这也是u5的专属文件，可以修改！！  
（2）上面的是初级版的BUG修改，还可以优化！看大佬的修改，他发现确实没有这个文件，但是有ospi.c文件，就猜测写源代码的人，出现写了这个BUG，就把修改了：

    src += ['STM32U5xx_HAL_Driver/Src/stm32u5xx_hal_qspi.c']
    src += ['STM32U5xx_HAL_Driver/Src/stm32u5xx_hal_ospi.c']

###3其他添加外设并无报错问题！！

