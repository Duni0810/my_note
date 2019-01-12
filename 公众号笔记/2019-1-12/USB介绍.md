---
title: USB介绍
tags: young
grammar_abbr: true
grammar_table: true
grammar_defList: true
grammar_emoji: true
grammar_footnote: true
grammar_ins: true
grammar_mark: true
grammar_sub: true
grammar_sup: true
grammar_checkbox: true
grammar_mathjax: true
grammar_flow: true
grammar_sequence: true
grammar_plot: true
grammar_code: true
grammar_highlight: true
grammar_html: true
grammar_linkify: true
grammar_typographer: true
grammar_video: true
grammar_audio: true
grammar_attachment: true
grammar_mermaid: true
grammar_classy: true
grammar_cjkEmphasis: true
grammar_cjkRuby: true
grammar_center: true
grammar_align: true
grammar_tableExtra: true
---
==文档制作工具：小书匠（markdown工具）==
==制作人     ：young==
==制作时间：2019-1-12==


----------

已经有几周没写东西了，上周本想写点东西的，但有几个朋友远道而来，到成都玩耍，就带他们耍了下。有段时间没见他们了，姑娘还是那样优秀，小伙依然健硕。不说了，就从这周开始咱们继续开始呗。去年写操作系统是还没讲完，我到时候看看啥时候继续接着讲，那么这周我想讲解下USB，可能有那么几篇，教你实现一个USB设备打印机类，能够实现PC端的文档数据打印（我用串口模拟数据打印过程）。
从去年的十二月中旬开始接触USB，经理要求完成USB设备类打印机，这东西上上下下还是花了我快一个月时间，毕竟从0开始接触USB，蹭我还记忆犹新，先记录下。

# USB是什么
USB是什么呢？当初我上大学那会儿和朋友调侃，USB不就是You SB嘛？哈哈哈，事实上，当然不是这样子的啦，USB是==Universal Serial Bus==的缩写，中文译为通用串行总线，也是一种通讯总线协议。实际上我们知道的BUS(总线）,例如：网口，SPI，I2C，RS-485等等。所以简单的说，USB就是一种接口，一种总线。
我们知道I2C的好处是通讯要两根线,大大的减小CPU的引脚资源，那USB有什么特殊的地方呢？其实在USB出现之前，计算机领域中的接口太多太繁杂，可以用下面这张，关于PC机箱背后的接口的图片来说明：

![机箱后图](./images/1547280650951.png)

是不是发现接口特别的多，好多你都不认识是干嘛的，所以嘛，在USB出现之前，各种接口太多，而且都不太容易使用，互相之间的兼容性也差，这时候才诞生了USB。
所以我们生活中看到的接口大部分都是USB接口，什么硬盘，打印机，手机接口，键鼠等等。总的来说，USB的出现，是希望通过此单个的USB接口，同时支持多种不同的应用，而且用户用起来也很方便，直接插上就能用了，也方便不同的设备的之间的互联。说白了，就相当于在之前众多的接口之上，设计出一个USB这么个万能的接口，以后各种外设，都可以用这一种接口即可。

# USB相关基础

## 控制器类型

USB设备，从物理上的逻辑结构来说，包含了主机Host和设备Device。其中，主机Host端，有对应的硬件的USB的主机控制器Host Controller，而设备端，连接的是对应的USB设备。
USB控制器类型贼多，什么OHCI，UHCI,EHCI啥的，我们这次只讲USB device，不讲控制器，所以就简单了解下他们的关系就好。
==OHCI(Open Host Controller Interface)== 是美国国家半导体公司，微软等推出的,OHCI更多地把要做的事情，用硬件来实现，因此，实现OHCI的USB控制器的软件驱动的开发工作，相对要容易些，软件要做的事情，相对较少。对应地，OHCI更多地应用在扩展卡，尤其是嵌入式领域中，常见的很多开发板中的USB的控制器，很多都是OHCI的。

==UHCI(Universal Host Controller Interface)==，创立者是Intel，而UHCI把更多的功能，留给了软件，相对来说，软件做的事情，即负担要重些。但是实现对应的UHCI的硬件的USB控制器，价格上，就相对便宜些。对应地，UHCI更多地应用在PC机中的主板上的USB控制器。


# USB硬件属性

一般情况下哦， USB1.x/USB2.0使用的是4线制,而USB3.0使用9线，别慌，他们是通用的,他们引脚对应的含义一般如下 ：

表1 USB 1.x/2.0 定义
| 引脚 | 名称 | 颜色  | 描述   |
| ---- | ---- | ----- | ------ |
| 1    | VBUS | Red   | 电源   |
| 2    | D-   | White | 数据线 |
| 3    | D+   | Green | 数据线 |
| 4    | GND  | Black | 地     |


表2 USB 3.0
| 引脚  | 颜色   | type A | type B |
| ----- | ------ | ------ | ------ |
| 1     | red    |     VBUS   |
| 2     | white  | D-     |
| 3     | green  | D+         |
| 4     | black  | GND     GND    |
| 5     | blue   | SSRX-  | SSTX-  |
| 6     | yellow | SSRX+  | SSTX+  |
| 7     | shield | GND        |
| 8     | purple | SSTX-  | SSRX-  |
| 9     | orange | SSTX+  | SSRX+  |
| shell | shell  | shield |

## USB分类
以上USB引脚定义,接下来介绍USB接口长什么样子的。关于分类USB应该可以分成：type，mini和micro。
那什么是type类型呢？我们生活中是不是经常听到听到type-C接口呀,就是那个苹果最先用的那种接口。其实还有type-A与type-B。type-A接口就是我们经常看到的那种，我们PC机器经常用到，99%的U盘用的就是type-A公头，自然我们大部分PC机就是那种母头接口。type-B接口可能生活中不常见，但是在嵌入式领域是肯定见过的，看下图就知道了。

![type-B接口](./images/1547284059868.png)

对，就是J-link调试器那种接口。这里说上一句,关于type-C是基于USB3.1协议的.

那什么是USB mini呢?这个大家应该也见过,可能只是不知道叫啥，那种老人机的充电器，中间是正方形，外框是梯形的那种,看图。

![USB mini-B](./images/1547284431612.png)

上图那种叫做USB mini-B,自然肯定有USB mini-A，这个我没有图，就是我们用数码相机那接口,和这个好像，只是中间是梯形的，上图中的中间是长方形的。但是他们的接口(母口)都是一样的,所以通用。

最后还有一种USB接口,就是micro，这个大家肯定都见过，就是我们常看到安卓的接口,比mini更扁。我就不拍照了,大伙都懂。

看到这儿，下回看到USB知道那是属于什么类型的USB了，作为技术人员，咱们专业点。

# USB 协议基础

介绍完USB硬件基础，下面介绍USB协议基础，如果想详细了解USB协议的动易可以去USB官网看(https://www.usb.org/)，里面啥都有。
我们常说的USB版本大概有USB1.1，USB2.0和USB3.0。其实这三个版本是针对有限设备的。实际上,在USB2.0到3.0阶段，还发布过一个USB2.5(USB Wireless)版本，这个版本是针对USB无线设备的，例如我们用的无线鼠标，无线wifi啥的。

其中，USB 1.1中所支持的速度是低速（Low Speed）的1.5Mbits/s，全速（Full Speed）的12Mbits/s，而USB 2.0提高了速度至高速（High Speed）的480Mbits/s，而最新的USB 3.0，支持超高速（Super Speed）的5Gbits/s。
这时候，是不是在想，同样是USB，为啥他们的速度差距这么大呢？这个主要是历史原因，当初用USB1.1时,主要做的是键盘，鼠标这样的设备，那个速度完全满足用户要求;速度低，对电磁辐射的抗干扰能力较强；最主要的原因是降低成本。

为了满足人民群众日益增长的对于高速速度传输方面的需求，USB2.0就出来了，据说这个和USB1.1推出时间之差不到2年的时间。举个例子：从MP3里面拷贝歌曲出来，如果是USB 1.1，那么实际效果最快也就1MB左右，而如果是USB 2.0，平均效果大概有3MB/s,5MB/s，性能好的可达10MB/s，20MB/s，所以，如果拷贝个1G的东西，相当于USB 1.1要1小时左右，而USB 2.0只要1分钟左右。因为如果没有USB 2.0的出现的话，那么现在的人们，早就放弃了USB了，因为谁也忍受不了这个太慢的速度。所以为了满足大家的需求，才有了USB 2.0的出现。
现在的USB3.0也是一个道理，为了满足之后广大人们的"贪婪"，例如你传几个G的视频，USB2.0怎么的都要几min吧，现在只要几秒或者几十秒，哔的一下,就好了，你说刺不刺激。当然，理论上是5G，毕竟这个是和硬件和软件挂钩的,具体的还是看你设备咯。

其实USB发展到2.0阶段，还有一个概念，就是OTG，那什么是OTG呢？说白了就是模式的互转，就是一种主机协商协议，允许两个设备之间互相协商谁去当Host。不过，即使在OTG中，也只是同一时刻，只存在单个的Host，而不允许存在多个Host的。
我们生活上就有这种现象，比如我们经常用手机连接U盘，实现文件互传。数码相机直接链接打印机，打印图片等等，这种就是OTG技术.



