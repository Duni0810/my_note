---
title: 数据结构周边---hash表实现
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
==制作时间：2018-11-18==


----------

这篇文章就结束数据结构的篇章吧，下周开启新的篇章，所以呢，这篇文章可能也有点长，如果能耐心看完的话，应该会有所收获的。那这篇文章我们讲二叉排序数和基于二叉排序树的hash的实现。

# 二叉排序树
顾名思义，二叉排序树肯定也是一种特殊的二叉树，并且是可以排序的。其实二叉排序树是改进二分查找的办法，因为二分查找是一种静态查找的办法，所以二叉排序树就是动态查找的一种办法。