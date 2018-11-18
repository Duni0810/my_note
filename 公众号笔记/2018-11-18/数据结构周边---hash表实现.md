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
我们先看一张图来说明下什么是二叉排序树呗，如下：

![二叉排序树](./images/二叉排序树.png)

有没有发现，二分查找的过程实际上就是一棵二叉树。我们看看图二叉排序树有什么特性：（１）任意一个结点的值都　**==大于==**　其　**==左子树的所有结点值==**　；（２）任意一个结点的值都　**==小于==**　其　**==右子树的所有结点值==**　。

这个好理解吧，因为二叉排序树也是二叉树，所以他的操作也和二叉树是一样一样的，我就不介绍了，可以看看之前的文章有具体的介绍，这里我就介绍他们不同的地方咯。
因为二叉排序树是特殊的二叉树，所以大部分的操作是一样的，只有插入，删除和查找不一样，所以我就不重新写一个二叉树了，就那拿之前实现的代码来改改就好。

## 结构变化

因为是特殊的二叉树，所以结构肯定有所变化,我们新增一个关键字，其实就越是数据。

``` c
typedef struct _tag_BSTreeNode BSTreeNode;

struct _tag_BSTreeNode
{
    BSKey* key;				// 查找关键字 
    BSTreeNode* left;	  	// 左子树 
    BSTreeNode* right;		// 右子树 
};
```

不仅如此，我们除了定义一个打印函数的类型，还得定义一个比较函数的数据类型。如果我们想实现可以复用的二叉排序树的话，我们就得让用户传入这个函数，因为程序是不知道我们要比较的函数的数据类型，所以得用户提供一个比较函数。

``` c
typedef void (BSTree_Printf)(BSTreeNode*);		// 打印函数 
typedef int (BSTree_Compare)(BSKey*, BSKey*);	// 比较函数 
```

## 二叉排序树的添加


