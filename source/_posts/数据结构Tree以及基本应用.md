---
title: 数据结构Tree的基本应用
date: 2019-02-05 17:55:43
tags: [数据结构]
categories :
- 个人分享
- 数据结构
- 树
---





<center> <h4><font color = "#36648B">✎</br>树的基本知识</center>
**1、树的定义**
从大体上来说，树可以分为二叉树与N叉树。这两者的区别可以从字面上就能理解出来
二叉树的所有孩子节点小于等于2，而N叉树的所有孩子节点中至少有一个是大于2的。



**2、树的实现**

```java
public class Tree {
    //根节点
    private Node head;
}

public class Node {
    public int element;
    public Node left;
    public Node right;
}
```


**3、相关专业术语**
- 叶子节点：没有子节点（孩子节点）的节点。
- 高度：树的最大层次。



**4、树的遍历：先序/中序/后序遍历**

后序遍历Demo
```java
public void orderTraverse(Node t){
    if(t == null){
        return ;
    }
    //System.out.println(t.element); 如果在这里（前面）访问元素则是先序遍历
    orderTraverse(t.left);
    //System.out.println(t.element); 如果在这里（中间）访问元素则是中序遍历
    orderTraverse(t.right);
    System.out.println(t.element);
}
```

<center> <h4><font color = "#36648B">✎✎</br>哈夫曼树</center>


<center> <h4><font color = "#36648B">✎✎✎</br>二叉搜索树</center>
1、定义
若其左子树存在，则其左子树中每个节点的值都不大于该节点值；
若其右子树存在，则其右子树中每个节点的值都不小于该节点值。

2、查询复杂度
查询每个节点需要的比较次数为节点深度加一。

使用中序遍历输出从小到大的值。

平衡二叉树(AVL Tree)


B树

B+树
