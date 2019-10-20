================================================== ==============================
                            README-Jetson.GPIO
                             适用于Tegra的Linux
          使用Jetson GPIO Python库和示例应用程序
================================================== ==============================

Jetson TX1，TX2和AGX Xavier开发板包含一个40引脚的GPIO接头，
类似于Raspberry Pi中的40针头。可以控制这些GPIO
使用Jetson中提供的Python库进行数字输入和输出
GPIO库包。该库与RPi.GPIO库具有相同的API
Raspberry Pi为了提供一种简单的方法来移动运行的应用程序
Raspberry Pi到Jetson板。

本文档介绍了Jetson GPIO库中包含的内容
包，如何配置系统并运行提供的示例应用程序，
和库API。

-------------------------------------------------- ------------------------------
                            包装组件
-------------------------------------------------- ------------------------------

除了本文档之外，Jetson GPIO库包中还包含
以下：

1. lib / python /子目录包含实现所有的Python模块
   库功能。gpio.py模块将是主要组件
   导入到应用程序并提供所需的API。gpio_event.py
   和gpio.pin_data.py模块由gpio.py模块使用，但不得使用
   直接导入到应用程序中。

2. samples /子目录包含有助于获取的示例应用程序
   熟悉库API并开始使用应用程序。该
   simple_input.py和simple_output.py应用程序显示如何执行读取
   并分别写入GPIO引脚，而button_led.py，
   button_event.py和button_interrupt.py显示如何使用按钮按下
   使用忙等待，阻塞等待和中断回调来使LED闪烁
   分别。

-------------------------------------------------- ------------------------------
              配置系统并运行示例脚本
-------------------------------------------------- ------------------------------

要使用Jetson GPIO库，必须使用正确的用户权限/组
先设定。

创建一个新的gpio用户组。然后将您的用户添加到新创建的组中。
        sudo groupadd -f -r gpio
        sudo usermod -a -G gpio your_user_name

通过将99-gpio.rules文件复制到rules.d中来安装自定义udev规则
目录：
        sudo cp /opt/nvidia/jetson-gpio/etc/99-gpio.rules/etc/udev/rules.d/

请注意，要执行新规则，您可能需要重新启动
或者通过发出以下命令重新加载udev规则：
        sudo udevadm control --reload-rules && sudo udevadm trigger

根据需要设置权限，提供示例应用程序
可以使用samples /目录。以下描述每个的操作
应用：

1. simple_input.py：该应用程序使用BCM引脚编号模式并读取
   40针头的引脚12处的值，并将值打印到
   屏幕。

2. simple_out.py：此应用程序使用BCM引脚编号模式
   Raspberry Pi并在BCM引脚18输出交替的高值和低值（或
   每2秒钟在插头上的插针12。

3. button_led.py：此应用程序使用BOARD引脚编号。它需要一个
   按钮连接到引脚18和GND，连接引脚18的上拉电阻
   到3V3和一个LED和限流电阻连接到引脚12
   应用程序读取按钮状态并保持LED每次亮1秒
   按下按钮的时间。

4. button_event.py：此应用程序使用BOARD引脚编号。它需要一个
   按钮连接到引脚18和GND，一个连接按钮的上拉电阻
   到3V3和一个LED和限流电阻连接到引脚12
   应用程序执行与button_led.py相同的功能，但执行
   阻止等待按钮按下事件而不是连续检查
   引脚的值，以减少CPU使用率。

5. button_interrupt.py：此应用程序使用BOARD引脚编号。它
   需要一个连接到引脚18和GND的按钮，一个连接的上拉电阻
   按钮3V3，LED和限流电阻连接到引脚12
   第二个LED和限流电阻连接到引脚13
   应用程序缓慢地连续闪烁第一个LED并快速闪烁
   仅在按下按钮时，第二个LED指示灯五次。

为了为Python设置正确的环境变量，
run_sample.sh脚本可用于运行这些示例应用程序。这可以
在samples /目录中使用以下命令完成：
        ./run_sample.sh <name_of_application_to_run>

也可以使用以下方法查看脚本的用法：
        ./run_sample.sh -h
        ./run_sample.sh --help

-------------------------------------------------- ------------------------------
                           完整的库API
-------------------------------------------------- ------------------------------

Jetson GPIO库提供RPi.GPIO提供的所有公共API
库，软件PWM API除外。以下讨论
使用每个API：

1.导入图书馆

要导入Jetson.GPIO模块，请使用：

    将Jetson.GPIO导入为GPIO

这样，您可以在其余部分中将模块称为GPIO
应用。也可以使用名称RPi.GPIO而不是导入模块
使用RPi库的现有代码的Jetson.GPIO。

2.引脚编号

Jetson GPIO库提供了四种编号I / O引脚的方法。首先
两个对应于RPi.GPIO库提供的模式，即BOARD和BCM
它指的是40引脚GPIO接头和Broadcom SoC的引脚号
GPIO号分别为。其余两种模式，CVM和TEGRA_SOC使用
字符串而不是与CVM / CVB上的信号名称对应的数字
连接器和Tegra SoC分别。

要指定您使用的模式（必需），请使用以下功能
呼叫：

    GPIO.setmode（GPIO.BOARD）
    ＃ 要么
    GPIO.setmode（GPIO.BCM）
    ＃ 要么
    GPIO.setmode（GPIO.CVM）
    ＃ 要么
    GPIO.setmode（GPIO.TEGRA_SOC）

要检查已设置的模式，您可以致电：

    mode = GPIO.getmode（）

该模式必须是GPIO.BOARD，GPIO.BCM，GPIO.CVM，GPIO.TEGRA_SOC或
没有。

3.警告

您正在尝试使用的GPIO可能已被使用
当前应用程序的外部。在这种情况下，Jetson GPIO
如果正在使用的GPIO被配置为除了之外的任何东西，库将警告您
默认方向（输入）。如果您之前尝试清理，它也会警告您
设置模式和频道。要禁用警告，请致电：

    GPIO.setwarnings（假）

4.设置频道

在用作输入或输出之前，必须先设置GPIO通道。配置
通道作为输入，调用：

    ＃（其中通道基于上面讨论的引脚编号模式）
    GPIO.setup（channel，GPIO.IN）

要将频道设置为输出，请致电：

    GPIO.setup（channel，GPIO.OUT）

也可以为输出通道指定初始值：

    GPIO.setup（channel，GPIO.OUT，initial = GPIO.HIGH）

将通道设置为输出时，也可以设置多个通道
频道立刻：

    ＃根据需要添加尽可能多的频道。你也可以使用元组：（18,12,13）
    渠道= [18,12,13]
    GPIO.setup（通道，GPIO.OUT）

5.输入

要读取频道的值，请使用：

    GPIO.input（通道）

这将返回GPIO.LOW或GPIO.HIGH。

6.输出

要设置配置为输出的引脚的值，请使用：

    GPIO.output（频道，州）

其中state可以是GPIO.LOW或GPIO.HIGH。

您还可以输出到频道的列表或元组：

    channels = [18,12,13]＃或使用元组
    GPIO.output（channels，GPIO.HIGH）＃或GPIO.LOW
    ＃将第一个通道设置为HIGH并将其设置为LOW
    GPIO.output（channel，（GPIO.LOW，GPIO.HIGH，GPIO.HIGH））

7.清理

在程序结束时，最好清理所有引脚的通道
被设置为默认状态。要清理所有使用的频道，请致电：

    GPIO.cleanup（）

如果您不想清洁所有通道，也可以清理
个人频道或频道列表或元组：

    GPIO.cleanup（chan1）＃cleanup only chan1
    GPIO.cleanup（[chan1，chan2]）＃cleanup只有chan1和chan2
    GPIO.cleanup（（chan1，chan2）＃执行与前一个语句相同的操作

8. Jetson董事会信息和图书馆版本

要获取有关Jetson模块的信息，请使用/ read：

    GPIO.JETSON_INFO

这为Python字典提供了以下键：P1_REVISION，RAM，
修订，类型，制造商和处理器。字典中的所有值都是
P1_REVISION除外的字符串，它是一个整数。

要获取有关库版本的信息，请使用/ read：

    GPIO.VERSION

这提供了具有XYZ版本格式的字符串。

9.中断

除了繁忙的民意调查，图书馆还提供了三种额外的方法
监视输入事件：

* wait_for_edge（）函数

  此函数阻止调用线程，直到提供的边缘为止
  检测。该函数可以调用如下：

    GPIO.wait_for_edge（channel，GPIO.RISING）

  第二个参数指定要检测的边缘，可以是
  GPIO.RISING，GPIO.FALLING或GPIO.BOTH。如果你只想限制等待
  到指定的时间，可以选择设置超时：

    #timeout以毫秒为单位
    GPIO.wait_for_edge（channel，GPIO.RISING，timeout = 500）

  该函数返回检测到边缘的通道，如果是，则返回None
  超时发生。

* event_detected（）函数

  此函数可用于定期检查自发生以来是否发生了事件
  最后一次通话。该功能可以设置和调用如下：

    ＃设置通道上的上升沿检测
    GPIO.add_event_detect（channel，GPIO.RISING）
    run_other_code（）
    如果GPIO.event_detected（频道）：
        做一点事（）

和以前一样，您可以检测GPIO.RISING，GPIO.FALLING或GPIO.BOTH的事件。

*检测到边缘时运行回调函数

  此功能可用于为回调函数运行第二个线程。因此，
  回调函数可以作为响应并发运行到主程序
  到了边缘。此功能可以使用如下：

    #define callback function
    def callback_fn（频道）：
        print（“从频道％s调用的回调”％channel）

    #add上升沿检测
    GPIO.add_event_detect（channel，GPIO.RISING，callback = callback_fn）

  如果需要，还可以添加多个回调，如下所示：

    def callback_one（频道）：
        打印（“第一回调”）

    def callback_two（频道）：
        打印（“第二回调”）

    GPIO.add_event_detect（channel，GPIO.RISING）
    GPIO.add_event_callback（channel，callback_one）
    GPIO.add_event_callback（channel，callback_two）

  在这种情况下，两个回调是顺序运行，而不是同时运行
  只有线程运行所有回调函数。

  为了防止通过折叠多次调用回调函数
  多个事件到一个，可以选择设置去抖时间：

    #bouncetime以毫秒为单位
    GPIO.add_event_detect（channel，GPIO.RISING，callback = callback_fn，
                          bouncetime = 200）

如果不再需要边缘检测，可以按如下方式删除它：

    GPIO.remove_event_detect（通道）

10.检查GPIO通道的功能

此功能允许您检查提供的GPIO通道的功能：

    GPIO.gpio_function（通道）

该函数返回GPIO.IN或GPIO.OUT。
