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
从大体上来说，树可以分为二叉树与N叉树。这两者的区别可以从字面上就能理解出来。
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
- 高度/深度：树的最大层次。



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
**1、定义**
若其左子树存在，则其左子树中每个节点的值都不大于该节点值；
若其右子树存在，则其右子树中每个节点的值都不小于该节点值。

**2、复杂度**

二叉搜索树的搜索复杂度与树的深度有关，范围是O(n) ~ O(log n)之间。

**3、遍历算法**
可使用中序遍历输出一组有序的值。

**4、插入算法（假设将元素a插入树B中）**

- 如果B为空树，则将元素a作为树的根节点。
- 否则，比较根节点与a的值，a若比较小则往左子树走，否则往右子树走，递归重复此节点比较过程，直至将a以叶子节点的形式插入到B（a与节点相等时可根据实际情况看作是小于或大于或者直接丢弃都可）

**5、删除算法（假设删除节点p）**

- 若p是子节点，则直接删除。
- 若p只有一个孩子节点，则删除p，让孩子节点顶替p的位置。
- 若p有两个孩子节点，那么删除p，并寻找p的后继顶替p。（p的后继是p的右子树的最左节点。）
  
  > 寻找p的前继应该也可以？p的前继是p的左节点，或是p的左节点的右节点。

**6、查找算法（假设根节点为p，需要查找的元素为a）**

- 若p = a，则直接返回p。
- 若p < a，则往左子树走，否则往右子树走，递归调用此流程。










<center> <h4><font color = "#36648B">✎✎✎</br>平衡二叉树(AVL Tree)</center>
**1、定义**

任意一节点的左子树与右子树高度相差不超过1。

**2、查找、删除、遍历算法**

平衡二叉树的查找、删除、遍历算法都与二叉搜索树的步骤一样，只是在树结构变化了之后（增加节点或删除节点），需要**递归判断**是否需要**再平衡**。

**3、再平衡**

当结构处于失衡状态时，就需要再均衡。判断是否处于失衡状态的逻辑是：判断**此节点的左右子树高度是否相差超过1**，如果是，则处于失衡状态，此节点为失衡点。之后需要判断是处于哪种失衡结构：

- **LL失衡**。需要对结构进行右旋操作。
  伪代码实现的实现的简单逻辑为：
  
  ```java
  /**
   * @param h    h为失衡点
   */
private Node rotateRight(Node h) {
      Node x = h.left;
      h.left = x.right;
      x.right = h;
      //省略更新高度、更新父节点等。
      return x;
  }
  ```
  
  如下图所示：
  ![](数据结构Tree以及基本应用\右旋.jpg)
  节点3为失衡点。
  
  
  
- **RR失衡**。需要进行左旋操作。与右旋操作类似，只是操作的“方向”是反的。

  伪代码实现的实现的简单逻辑为：

  ```java
  private Node rotateLeft(Node h) {
      Node x = h.right;
      h.right = x.left;
      x.left = h;
      //省略更新高度、更新父节点等。
      return x;
   }
  ```

- **LR失衡**。先执行左旋再执行右旋。

  伪代码实现的实现的简单逻辑为：

  ```java
  private Node rotateLeftRight(Node h) {
      h.left = rotateLeft(h.left);
      return rotateRight(h);
  }
  ```
  >对平衡点的左节点进行左旋，再对平衡点进行右旋。

  如下图示：

  ![](数据结构Tree以及基本应用\左旋右旋.jpg)



- **RL失衡**。先执行右旋再执行左旋。

> 第一个字母表示高度较高的子树位置，第二个字母表示锁挂载节点所处于的子树位置。







<center> <h4><font color = "#36648B">✎✎✎</br>2-3树</center>
1、定义

> 假设父节点为P，左节点为L，中间节点为M，右节点为R。

- 所有叶子节点都在同一层。
- 2节点：父节点存储一个值，最多有左右两个子树。且`L< P < R`。
- 3节点：父节点存储两个值，最多有左中右三个子树。且`L < P1 < M < P2 < R`。
- 子节点要么为0个，要么大于等于2个。



2、插入

永远都是在叶子节点处新增节点，插入过程如下图



![](数据结构Tree以及基本应用\2-3树插入过程.png)

<center> <h4><font color = "#36648B">✎✎✎</br>B树</center>

<center> <h4><font color = "#36648B">✎✎✎</br>B+树</center> 

<center> <h4><font color = "#36648B">✎✎✎</br>红黑树</center>