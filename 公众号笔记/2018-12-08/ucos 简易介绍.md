---
title: ucos 简易介绍
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
==制作时间：2018-12-8==


----------

上周没有写啥笔记，心情不太好，就给自己放个假，这周咱们继续写一些东西，在任何时候，学习还有有所必要的，无论学啥。本来想写写关于编程的设计模式，但是这东西比较抽象，换句话，我也没能很好的掌握，咱们还是找个时间在说设计模式。那么这周我们聊聊ucos的一些实现过程吧，可能很多人都知道这东西，但是不太了解他们的实现，咱们还是说说这个简单的操作系统的简易实现。
ucos源码可能也就5000来行的样子，但是他的功能确实强大的，是一个实时多任务的操作系统内核。那什么是多任务呢？
# 多任务

操作系统就不多说了，我们只要知道他是在硬件之上，应用软件之下的软件平台，windows就是例子。咱么说说多任务，我们用电脑可以听歌，上网，看电影。可以同时提执行多个并行任务，并且各个任务之间相互独立互不影响，在宏观上像并发运行。
其实我们知道CPU只有一颗，不可能做到他并发运行，只有不同的任务轮流使用CPU资源，本质上还是单任务。只是CPU运行的比较快让我们感觉是多任务在跑而已。
我们使用多任务还有一个好处，很好的使用CPU资源。因为我们在delay的时，CPU其实是在空跑，这样显得有点资源浪费，在多任务是延时时候会自动调用其他任务，这样就充分的利用CPU资源，提高使用效率。
任务有三态，运行态，就绪态和挂起态。下面分析任务之间的转化。

# 任务调度

假设我们定义一个就绪列表为一个32位的数据，例如：

``` c
unsigned int os_ready_tab; //就绪列表
```

在32位变量中，每一位都表示一个任务，0表示挂起状态，1表示就绪状态，它记录了各个任务是否就绪，所以就是一个就绪表。所以他最多支持32个任务。所以，若想创建（就绪）一个任务或者删除一个任务只需要置位或者清除就绪表对应的任务。代码可以简化如下：

``` c
/* 在就绪表中登记就绪任务 */
#define OSSetPrioRdy(prio) \
{			         	  \
	OSRdyTbl |= 0x01<<prio; \
}
/* 从就绪表中删除任务 */
#define OSDelPrioRdy(prio) \
{ 							\
	OSRdyTbl &= ~(0x01<<prio); \
}
```

我们知道如何在登记表中表示任务的状态，这不够，我们还得知道任务应该选择合适的任务调度，就是系统如何知道我们要调度哪个任务?这个时候就引入优先级的概念，为每个任务安排一个唯一的优先级别，当同时有多个任务就绪时，优先运行优先级较高的任务就可以解决这个问题。
同时的，任务的优先级也是作为任务的唯一标识，这就表示在ucos中是不允许两个任务处于同一个优先级的。
所以我们在使用ucos时候会常定义这样的宏，一般来说，数字越低表示优先级越高。

``` c
/**************定义任务优先级*************/
#define		 PrioTask0 		0
#define		 PrioTask1 		1
#define	 	 PrioTask2 		2
#define    	 PrioTask3 		3
```

这中还有一个概念：**抢占式调度**，一旦就绪状态中出现优先权更高的任务，便立即剥夺当前任务的运行权，把 CPU 分配给更高优先级的任务。这样 CPU 总是执行处于就绪条件下优先级最高的任务。

要实现运行高优先级任务，我们得知道那个优先级的任务比较高，所以就得找到任务优先级高的任务，代码可以如下：

``` c
/* 在就绪表中查找更高级的就绪任务 */
#define OSGetHighRdy() \
{										 \
for(OSNextTaskPrio = 0; \
     (OSNextTaskPrio < OS_TASKS) && (!(OSRdyTbl & (0x01<<OSNextTaskPrio))); \
      OSNextTaskPrio ++ ); 		\
OSPrioHighRdy = OSNextTaskPrio; \
}
```
代码中判断优先级从高到低查找，找到数据中某位数据位为1为止，这就表示找到优先级高的任务了，直接返回或者赋值给变量。代码中这样写不是特别的好，原因有二。其一：这个函数是不可重入的函数；其二，实时操作系统有一个特征就是延时可预测，然而这种算法的延时任务的数量的变化而不同，但是这种算法是最简单的。

# 多任务的时间管理

在多任务做，也是需要一个心跳来维持系统的正常运行，和晶振一样。在系统中一般利用一个固定周期的中断来充当。所以一般在任务中有一个任务级别的延时。这个函数比较简单。就是就是先挂起当起当前任务，设定其延时节拍数，然后进行任务切换，在指定的时钟节拍数到来之后，将当前任务恢复为就绪状态。任务必须通过 任务延时 或 任务挂起的方式 让出 CPU的使用权，使更低优先级任务有机会运行。代码实现可以如下：

``` c
void OSTimeDly(INT32U ticks)
{
	if( ticks > 0 ) 		/* 当延时有效 */
	{
		OS_ENTER_CRITICAL();			/* 进入临界区 */
		OSDelPrioRdy(OSPrioCur);  /* 把当前任务从就绪表中删去 */
		TCB[OSPrioCur].OSTCBDly = ticks;  /* 设置任务延时节拍数 */
		OS_EXIT_CRITICAL();		   	 /* 推出临界区 */
		OSSched();						 /* 重新调度 */
	}
}
```

代码中有一个临界区，这个临界区的实现就是关闭中断的方式，就不多讲了。在延时函数中， 定时器的中断服务函数依次对各个延时任务的延时节拍数减 1。若发现某个任务的延时节拍数变为 0，则把它从挂起态置为就绪态。中断服务函数可以如下：

``` c
void TickInterrupt(void)
{
	static INT8U i;
	OSTime++;
	for(i = 0; i < OS_TASKS; i++) /* 刷新各任务时钟 */
	{
		// 任务块时钟更新
		if(TCB[i].OSTCBDly )
		{
			TCB[i].OSTCBDly --;
			
			/* 当任务时钟到时,必须是由定时器减时的才行*/
			if(TCB[i].OSTCBDly == 0) 
			{
				OSSetPrioRdy(i); /* 使任务可以重新运行 */
			}
		}
	} 
	rSRCPND |= BIT_TIMER0;
	rINTPND |= BIT_TIMER0;
}
```
实际上任务心跳（定时器中断）做的是任务的刷新工作，因为任务要轮流的使用CPU的使用权限。实际上系统会自动的创建一个空闲任务，为了不让系统空跑，闲置资源。他的任务优先级是最低的，做一些无关紧要的工作，类似CPU的使用率之类的。所以像这种空闲任务是一直处于就绪状态。

# 任务的切换









![CYoung](./images/二维码.jpg)
