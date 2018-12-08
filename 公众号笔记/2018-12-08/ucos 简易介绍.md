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