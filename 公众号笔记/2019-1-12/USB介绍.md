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
那什么是type类型呢？我们生活中是不是经常听到听到type-C接口呀,就是那个苹果最先用的那种接口。其实还有type-A与type-B。type-A接口就是我们经常看到的那种，我们PC机器经常用到，99%的U盘用的就是type-A公头，自然我们大部分PC机就是那种母头接口。type-B接口可能生活中不常见，但是在嵌入式领域是肯定见过的，看下图就知道了












