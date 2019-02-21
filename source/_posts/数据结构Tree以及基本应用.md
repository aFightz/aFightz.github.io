---
数据结构Tree的基本应用
date: 2019-02-05 17:55:43
tags: [数据结构]
categories :
- 个人分享
- 数据结构
- 树
---



树的基本知识
-树的定义
从大体上来说，树可以分为二叉树与N叉树。这两者的区别可以从字面上就能理解出来
二叉树的所有孩子节点小于等于2，而N叉树的所有孩子节点中至少有一个是大于2的。

![](数据结构Tree以及基本应用\二叉树.png)

-树的实现

-二叉树的实现
public class Tree {
    //根节点
    private Node head;
}

public class Node {
    private int element;
    HuffManNode left;
    HuffManNode right;
}



-相关专业术语
叶子节点
高度



树的遍历
先序/中序/后序遍历

哈夫曼树


二叉搜索树

平衡二叉树(AVL Tree)

B树

B+树
