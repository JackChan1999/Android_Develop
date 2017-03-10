# **1. 堆的定义**

设有n个数据元素的关键字为（k0、k1、…、kn-1），如果它们满足以下的关系：k<sub>i</sub><= k<sub>2i+1</sub>且k<sub>i</sub><= k<sub>2i+2</sub>（或k<sub>i</sub>>= k<sub>2i+1</sub>且k<sub>i</sub>>= k<sub>2i+2</sub>）（i=0、1、…、(n-2)/2）则称之为堆(Heap)。

如果将此数据元素序列用一维数组存储，并将此数组对应一棵完全二叉树，则堆的含义可以理解为：在完全二叉树中任何非终端结点的关键字均不大于（或不小于）其左、右孩子结点的关键字。

下图(b)、(c)分别给出了最小堆和最大堆的例子，前者任一非终端结点的关键字均小于或等于它的左、右孩子的关键字，此时位于堆顶(即完全二叉树的根结点位置)的结点的关键字是整个序列中最小的，所以称它为最小堆；后者任一非终端结点的关键字均大于或等于它的左、右孩子的关键字，此时位于堆顶的结点的关键字是整个序列中最大的，所以称它为最大堆。

![堆](http://img.blog.csdn.net/20170104181148004?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

调整算法FilterDown要求将以分支结点i为根的子树调整为最小堆，其基本思想是：从结点i开始向下调整，先比较结点i左孩子结点和右孩子结点的关键字大小，如果结点i左孩子结点的关键字小于右孩子结点的关键字，则沿结点i的左分支进行调整；否则沿结点i的右分支进行调整，在算法中用j指示关键字值较小的孩子结点。然后结点i和结点j进行关键字比较，若结点i的关键字大于结点j的关键字，则两结点对调位置，相当于把关键字小的结点上浮。再令i＝j，j＝2*j十l，继续向下一层进行比较；若结点i的关键字不大于结点j的关键字或结点i没有孩子时调整结束。 

![堆](http://img.blog.csdn.net/20170104181334614?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注意：这里的“堆”是指一种的特殊的二叉树，不要和java和C++等编程语言里的
“堆”混淆，后者指的是程序员用new能得到的计算机内存的可用部分。

堆的介绍

- 堆是完全二叉树
- 常常用数组实现
- 每一个节点的关键字都大于（等于）这个节点的子节点的关键字

弱序，优先级队列

# **2. 在堆中插入元素**

在堆的类定义中成员函数Insert( )用于在堆中插入一个数据元素，在此规定数据元素总是插在已经建成的最小堆后面，如下图所示在堆中插入关键字为14的数据元素。显然在堆中插入元素后可能破坏堆的性质，所以还需要调用FilterUp( )函数，进行自下而上调整使之所在的子树成为堆。 

![堆](http://img.blog.csdn.net/20170104181430881?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在堆的类定义中成员函数DeleteTop( )用于删除堆顶数据元素。在从堆中删除堆顶元素后，一般把堆的最后一个元素移到堆顶，并将堆的当前元素个数heapCurrentSize减1，最后需要调用FilterDown()函数从堆顶向下进行调整。如图6-20所示给出了在堆中删除堆顶元素的过程。

![堆](http://img.blog.csdn.net/20170104181506740?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# **3. 二叉堆**

二叉堆就是通常我们所说的数据结构中"堆"中的一种。和以往一样，本文会先对二叉堆的理论知识进行简单介绍，然后给出C语言的实现。后续再分别给出C++和Java版本的实现；实现的语言虽不同，但是原理如出一辙，选择其中之一进行了解即可

# **4. 堆和二叉堆的介绍**

## **4.1 堆的定义**

堆(heap)，这里所说的堆是数据结构中的堆，而不是内存模型中的堆。堆通常是一个可以被看做一棵树，它满足下列性质：

- [性质一] 堆中任意节点的值总是不大于(不小于)其子节点的值；
- [性质二] 堆总是一棵完全树。

将任意节点不大于其子节点的堆叫做最小堆或小根堆，而将任意节点不小于其子节点的堆叫做最大堆或大根堆。常见的堆有二叉堆、左倾堆、斜堆、二项堆、斐波那契堆等等。

## **4.2 二叉堆的定义**

二叉堆是完全二元树或者是近似完全二元树，它分为两种：最大堆和最小堆。
最大堆：父结点的键值总是大于或等于任何一个子节点的键值；最小堆：父结点的键值总是小于或等于任何一个子节点的键值。示意图如下：

![](http://images.cnitblog.com/i/497634/201403/182339209436216.jpg)

二叉堆一般都通过"数组"来实现。数组实现的二叉堆，父节点和子节点的位置存在一定的关系。有时候，我们将"二叉堆的第一个元素"放在数组索引0的位置，有时候放在1的位置。当然，它们的本质一样(都是二叉堆)，只是实现上稍微有一丁点区别。
假设"第一个元素"在数组中的索引为 0 的话，则父节点和子节点的位置关系如下：

- 索引为i的左孩子的索引是 (2*i+1);
- 索引为i的左孩子的索引是 (2*i+2);
- 索引为i的父结点的索引是 floor((i-1)/2);

假设"第一个元素"在数组中的索引为 1 的话，则父节点和子节点的位置关系如下：

- 索引为i的左孩子的索引是 (2*i)
- 索引为i的左孩子的索引是 (2*i+1)
- 索引为i的父结点的索引是 floor(i/2)

![](http://images.cnitblog.com/i/497634/201403/182343402241540.jpg)

<font color="red">注意：本文二叉堆的实现统统都是采用"二叉堆第一个元素在数组索引为0"的方式！</font>

# 5. 二叉堆的图文解析

在前面，我们已经了解到："最大堆"和"最小堆"是对称关系。这也意味着，了解其中之一即可。本节的图文解析是以"最大堆"来进行介绍的。

二叉堆的核心是"添加节点"和"删除节点"，理解这两个算法，二叉堆也就基本掌握了。下面对它们进行介绍。

## 5.1 添加

假设在最大堆[90,80,70,60,40,30,20,10,50]种添加85，需要执行的步骤如下：

![](http://images.cnitblog.com/i/497634/201403/182345301461858.jpg)

如上图所示，当向最大堆中添加数据时：先将数据加入到最大堆的最后，然后尽可能把这个元素往上挪，直到挪不动为止！

将85添加到[90,80,70,60,40,30,20,10,50]中后，最大堆变成了[90,85,70,60,80,30,20,10,50,40]。

```java
/*
 * 最大堆的向上调整算法(从start开始向上直到0，调整堆)
 *
 * 注：数组实现的堆中，第N个节点的左孩子的索引值是(2N+1)，右孩子的索引是(2N+2)。
 *
 * 参数说明：
 *     start -- 被上调节点的起始位置(一般为数组中最后一个元素的索引)
 */
protected void filterup(int start) {
    int c = start;            // 当前节点(current)的位置
    int p = (c-1)/2;        // 父(parent)结点的位置 
    T tmp = mHeap.get(c);        // 当前节点(current)的大小

    while(c > 0) {
        int cmp = mHeap.get(p).compareTo(tmp);
        if(cmp >= 0)
            break;
        else {
            mHeap.set(c, mHeap.get(p));
            c = p;
            p = (p-1)/2;   
        }       
    }
    mHeap.set(c, tmp);
}
  
/* 
 * 将data插入到二叉堆中
 */
public void insert(T data) {
    int size = mHeap.size();

    mHeap.add(data);    // 将"数组"插在表尾
    filterup(size);        // 向上调整堆
}
```

insert(data)的作用：将数据data添加到最大堆中。mHeap是动态数组ArrayList对象。

当堆已满的时候，添加失败；否则data添加到最大堆的末尾。然后通过上调算法重新调整数组，使之重新成为最大堆。

## 5.2 删除

假设从最大堆[90,85,70,60,80,30,20,10,50,40]中删除90，需要执行的步骤如下：

![](http://images.cnitblog.com/i/497634/201403/182348387716132.jpg)

如上图所示，当从最大堆中删除数据时：先删除该数据，然后用最大堆中最后一个的元素插入这个空位；接着，把这个“空位”尽量往上挪，直到剩余的数据变成一个最大堆。
从[90,85,70,60,80,30,20,10,50,40]删除90之后，最大堆变成了[85,80,70,60,40,30,20,10,50]。


注意：考虑从最大堆[90,85,70,60,80,30,20,10,50,40]中删除60，执行的步骤不能单纯的用它的字节点来替换；而必须考虑到"替换后的树仍然要是最大堆"！

![](http://images.cnitblog.com/i/497634/201403/182350015371912.jpg)

```java
/* 
 * 最大堆的向下调整算法
 *
 * 注：数组实现的堆中，第N个节点的左孩子的索引值是(2N+1)，右孩子的索引是(2N+2)。
 *
 * 参数说明：
 *     start -- 被下调节点的起始位置(一般为0，表示从第1个开始)
 *     end   -- 截至范围(一般为数组中最后一个元素的索引)
 */
protected void filterdown(int start, int end) {
    int c = start;          // 当前(current)节点的位置
    int l = 2*c + 1;     // 左(left)孩子的位置
    T tmp = mHeap.get(c);    // 当前(current)节点的大小

    while(l <= end) {
        int cmp = mHeap.get(l).compareTo(mHeap.get(l+1));
        // "l"是左孩子，"l+1"是右孩子
        if(l < end && cmp<0)
            l++;        // 左右两孩子中选择较大者，即mHeap[l+1]
        cmp = tmp.compareTo(mHeap.get(l));
        if(cmp >= 0)
            break;        //调整结束
        else {
            mHeap.set(c, mHeap.get(l));
            c = l;
            l = 2*l + 1;   
        }       
    }   
    mHeap.set(c, tmp);
}

/*
 * 删除最大堆中的data
 *
 * 返回值：
 *      0，成功
 *     -1，失败
 */
public int remove(T data) {
    // 如果"堆"已空，则返回-1
    if(mHeap.isEmpty() == true)
        return -1;

    // 获取data在数组中的索引
    int index = mHeap.indexOf(data);
    if (index==-1)
        return -1;

    int size = mHeap.size();
    mHeap.set(index, mHeap.get(size-1));// 用最后元素填补
    mHeap.remove(size - 1);                // 删除最后的元素

    if (mHeap.size() > 1)
        filterdown(index, mHeap.size()-1);    // 从index号位置开始自上向下调整为最小堆

    return 0;
}
```

# 6. 二叉堆的Java实现

## 6.1 二叉堆(最大堆)
```java
/**
 * 二叉堆(最大堆)
 *
 * @author skywang
 * @date 2014/03/07
 */

import java.util.ArrayList;
import java.util.List;

public class MaxHeap<T extends Comparable<T>> {

    private List<T> mHeap;    // 队列(实际上是动态数组ArrayList的实例)

    public MaxHeap() {
        this.mHeap = new ArrayList<T>();
    }

    /* 
     * 最大堆的向下调整算法
     *
     * 注：数组实现的堆中，第N个节点的左孩子的索引值是(2N+1)，右孩子的索引是(2N+2)。
     *
     * 参数说明：
     *     start -- 被下调节点的起始位置(一般为0，表示从第1个开始)
     *     end   -- 截至范围(一般为数组中最后一个元素的索引)
     */
    protected void filterdown(int start, int end) {
        int c = start;          // 当前(current)节点的位置
        int l = 2*c + 1;     // 左(left)孩子的位置
        T tmp = mHeap.get(c);    // 当前(current)节点的大小

        while(l <= end) {
            int cmp = mHeap.get(l).compareTo(mHeap.get(l+1));
            // "l"是左孩子，"l+1"是右孩子
            if(l < end && cmp<0)
                l++;        // 左右两孩子中选择较大者，即mHeap[l+1]
            cmp = tmp.compareTo(mHeap.get(l));
            if(cmp >= 0)
                break;        //调整结束
            else {
                mHeap.set(c, mHeap.get(l));
                c = l;
                l = 2*l + 1;   
            }       
        }   
        mHeap.set(c, tmp);
    }

    /*
     * 删除最大堆中的data
     *
     * 返回值：
     *      0，成功
     *     -1，失败
     */
    public int remove(T data) {
        // 如果"堆"已空，则返回-1
        if(mHeap.isEmpty() == true)
            return -1;

        // 获取data在数组中的索引
        int index = mHeap.indexOf(data);
        if (index==-1)
            return -1;

        int size = mHeap.size();
        mHeap.set(index, mHeap.get(size-1));// 用最后元素填补
        mHeap.remove(size - 1);                // 删除最后的元素

        if (mHeap.size() > 1)
            filterdown(index, mHeap.size()-1);    // 从index号位置开始自上向下调整为最小堆

        return 0;
    }

    /*
     * 最大堆的向上调整算法(从start开始向上直到0，调整堆)
     *
     * 注：数组实现的堆中，第N个节点的左孩子的索引值是(2N+1)，右孩子的索引是(2N+2)。
     *
     * 参数说明：
     *     start -- 被上调节点的起始位置(一般为数组中最后一个元素的索引)
     */
    protected void filterup(int start) {
        int c = start;            // 当前节点(current)的位置
        int p = (c-1)/2;        // 父(parent)结点的位置 
        T tmp = mHeap.get(c);        // 当前节点(current)的大小

        while(c > 0) {
            int cmp = mHeap.get(p).compareTo(tmp);
            if(cmp >= 0)
                break;
            else {
                mHeap.set(c, mHeap.get(p));
                c = p;
                p = (p-1)/2;   
            }       
        }
        mHeap.set(c, tmp);
    }
      
    /* 
     * 将data插入到二叉堆中
     */
    public void insert(T data) {
        int size = mHeap.size();

        mHeap.add(data);    // 将"数组"插在表尾
        filterup(size);        // 向上调整堆
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        for (int i=0; i<mHeap.size(); i++)
            sb.append(mHeap.get(i) +" ");

        return sb.toString();
    }
 
    public static void main(String[] args) {
        int i;
        int a[] = {10, 40, 30, 60, 90, 70, 20, 50, 80};
        MaxHeap<Integer> tree=new MaxHeap<Integer>();

        System.out.printf("== 依次添加: ");
        for(i=0; i<a.length; i++) {
            System.out.printf("%d ", a[i]);
            tree.insert(a[i]);
        }

        System.out.printf("\n== 最 大 堆: %s", tree);

        i=85;
        tree.insert(i);
        System.out.printf("\n== 添加元素: %d", i);
        System.out.printf("\n== 最 大 堆: %s", tree);

        i=90;
        tree.remove(i);
        System.out.printf("\n== 删除元素: %d", i);
        System.out.printf("\n== 最 大 堆: %s", tree);
        System.out.printf("\n");
    }
}
```
## 6.2 二叉堆(最小堆)
```java
/**
 * 二叉堆(最小堆)
 *
 * @author skywang
 * @date 2014/03/07
 */

import java.util.ArrayList;
import java.util.List;

public class MinHeap<T extends Comparable<T>> {

    private List<T> mHeap;        // 存放堆的数组

    public MinHeap() {
        this.mHeap = new ArrayList<T>();
    }

    /* 
     * 最小堆的向下调整算法
     *
     * 注：数组实现的堆中，第N个节点的左孩子的索引值是(2N+1)，右孩子的索引是(2N+2)。
     *
     * 参数说明：
     *     start -- 被下调节点的起始位置(一般为0，表示从第1个开始)
     *     end   -- 截至范围(一般为数组中最后一个元素的索引)
     */
    protected void filterdown(int start, int end) {
        int c = start;          // 当前(current)节点的位置
        int l = 2*c + 1;     // 左(left)孩子的位置
        T tmp = mHeap.get(c);    // 当前(current)节点的大小

        while(l <= end) {
            int cmp = mHeap.get(l).compareTo(mHeap.get(l+1));
            // "l"是左孩子，"l+1"是右孩子
            if(l < end && cmp>0)
                l++;        // 左右两孩子中选择较小者，即mHeap[l+1]

            cmp = tmp.compareTo(mHeap.get(l));
            if(cmp <= 0)
                break;        //调整结束
            else {
                mHeap.set(c, mHeap.get(l));
                c = l;
                l = 2*l + 1;   
            }       
        }   
        mHeap.set(c, tmp);
    }
     
    /*
     * 最小堆的删除
     *
     * 返回值：
     *     成功，返回被删除的值
     *     失败，返回null
     */
    public int remove(T data) {
        // 如果"堆"已空，则返回-1
        if(mHeap.isEmpty() == true)
            return -1;

        // 获取data在数组中的索引
        int index = mHeap.indexOf(data);
        if (index==-1)
            return -1;

        int size = mHeap.size();
        mHeap.set(index, mHeap.get(size-1));// 用最后元素填补
        mHeap.remove(size - 1);                // 删除最后的元素

        if (mHeap.size() > 1)
            filterdown(index, mHeap.size()-1);    // 从index号位置开始自上向下调整为最小堆

        return 0;
    }

    /*
     * 最小堆的向上调整算法(从start开始向上直到0，调整堆)
     *
     * 注：数组实现的堆中，第N个节点的左孩子的索引值是(2N+1)，右孩子的索引是(2N+2)。
     *
     * 参数说明：
     *     start -- 被上调节点的起始位置(一般为数组中最后一个元素的索引)
     */
    protected void filterup(int start) {
        int c = start;            // 当前节点(current)的位置
        int p = (c-1)/2;        // 父(parent)结点的位置 
        T tmp = mHeap.get(c);        // 当前节点(current)的大小

        while(c > 0) {
            int cmp = mHeap.get(p).compareTo(tmp);
            if(cmp <= 0)
                break;
            else {
                mHeap.set(c, mHeap.get(p));
                c = p;
                p = (p-1)/2;   
            }       
        }
        mHeap.set(c, tmp);
    }
 
    /* 
     * 将data插入到二叉堆中
     */
    public void insert(T data) {
        int size = mHeap.size();

        mHeap.add(data);    // 将"数组"插在表尾
        filterup(size);        // 向上调整堆
    }
       
    public String toString() {
        StringBuilder sb = new StringBuilder();
        for (int i=0; i<mHeap.size(); i++)
            sb.append(mHeap.get(i) +" ");

        return sb.toString();
    }

    public static void main(String[] args) {
        int i;
        int a[] = {80, 40, 30, 60, 90, 70, 10, 50, 20};
        MinHeap<Integer> tree=new MinHeap<Integer>();

        System.out.printf("== 依次添加: ");
        for(i=0; i<a.length; i++) {
            System.out.printf("%d ", a[i]);
            tree.insert(a[i]);
        }

        System.out.printf("\n== 最 小 堆: %s", tree);

        i=15;
        tree.insert(i);
        System.out.printf("\n== 添加元素: %d", i);
        System.out.printf("\n== 最 小 堆: %s", tree);

        i=10;
        tree.remove(i);
        System.out.printf("\n== 删除元素: %d", i);
        System.out.printf("\n== 最 小 堆: %s", tree);
        System.out.printf("\n");
    }
}
```

## 6.3 二叉堆的Java测试程序

测试程序已经包含在相应的实现文件中了，这里只说明运行结果。

最大堆(MaxHeap.java)的运行结果：

```
== 依次添加: 10 40 30 60 90 70 20 50 80 
== 最 大 堆: 90 80 70 60 40 30 20 10 50 
== 添加元素: 85
== 最 大 堆: 90 85 70 60 80 30 20 10 50 40 
== 删除元素: 90
== 最 大 堆: 85 80 70 60 40 30 20 10 50 
```

最小堆(MinHeap.java)的运行结果：

```
== 最 小 堆: 10 20 30 50 90 70 40 80 60 
== 添加元素: 15
== 最 小 堆: 10 15 30 50 20 70 40 80 60 90 
== 删除元素: 10
== 最 小 堆: 15 20 30 50 90 70 40 80 60 
```

PS. 二叉堆是"堆排序"的理论基石。以后讲解算法时会讲解到"堆排序"，理解了"二叉堆"之后，"堆排序"就很简单了