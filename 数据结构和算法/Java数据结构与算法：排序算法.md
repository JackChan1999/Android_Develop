

# **1. 冒泡排序**
冒泡排序(Bubble Sort)，又被称为气泡排序或泡沫排序。

它是一种较简单的排序算法。它会遍历若干次要排序的数列，每次遍历时，它都会从前往后依次的比较相邻两个数的大小；如果前者比后者大，则交换它们的位置。这样，一次遍历之后，最大的元素就在数列的末尾！ 采用相同的方法再次遍历时，第二大的元素就被排列在最大元素之前。重复此操作，直到整个数列都有序为止！

相邻元素两两比较，大的往后放，第一次完毕，最大值出现在了最大索引处

```java
/*
 * 冒泡排序基本概念是：
 * 依次比较相邻的两个数，将小数放在前面，大数放在后面。
 * 即在第一趟：首先比较第1个和第2个数，将小数放前，大数放后。
 * 然后比较第2个数和第3个数，将小数放前，大数放后，如此继续，
 * 直至比较最后两个数，将小数放前，大数放后。至此第一趟结束，
 * 将最大的数放到了最后。在第二趟：仍从第一对数开始比较
 * （因为可能由于第2个数和第3个数的交换，使得第1个数不再小于第2个数），
 * 将小数放前，大数放后，一直比较到倒数第二个数（倒数第一的位置上已经是最大的），
 * 第二趟结束，在倒数第二的位置上得到一个新的最大数
 * （其实在整个数列中是第二大的数）。如此下去，重复以上过程，直至最终完成排序。 
 */
public void bubbleSort(int[] arr){
   for (int i = 0; i < arr.length-1; i++) {
       for (int j = 0; j < arr.length-1-i; j++) {
           if (array[j] > array[j+1]){
               int temp = array[j];
               array[j] = array[j+1];
               array[j+1] = temp;
           }
       }
   }
}
```

下面以数列{20,40,30,10,60,50}为例，演示它的冒泡排序过程

![冒泡排序](http://images.cnitblog.com/i/497634/201403/121521153545888.jpg)

我们先分析第1趟排序
当i=5,j=0时，a[0]<a[1]。此时，不做任何处理！
当i=5,j=1时，a[1]>a[2]。此时，交换a[1]和a[2]的值；交换之后，a[1]=30，a[2]=40。
当i=5,j=2时，a[2]>a[3]。此时，交换a[2]和a[3]的值；交换之后，a[2]=10，a[3]=40。
当i=5,j=3时，a[3]<a[4]。此时，不做任何处理！
当i=5,j=4时，a[4]>a[5]。此时，交换a[4]和a[5]的值；交换之后，a[4]=50，a[3]=60。

于是，第1趟排序完之后，数列{20,40,30,10,60,50}变成了{20,30,10,40,50,60}。此时，数列末尾的值最大。

根据这种方法：
第2趟排序完之后，数列中a[5...6]是有序的。
第3趟排序完之后，数列中a[4...6]是有序的。
第4趟排序完之后，数列中a[3...6]是有序的。
第5趟排序完之后，数列中a[1...6]是有序的。

第5趟排序之后，整个数列也就是有序的了

## 1.1 冒泡排序的时间复杂度和稳定性

### 1.1.1 冒泡排序时间复杂度

冒泡排序的时间复杂度是O(N2)。
假设被排序的数列中有N个数。遍历一趟的时间复杂度是O(N)，需要遍历多少次呢？N-1次！因此，冒泡排序的时间复杂度是O(N2)。

 

### 1.1.2 冒泡排序稳定性

冒泡排序是稳定的算法，它满足稳定算法的定义。
算法稳定性 -- 假设在数列中存在a[i]=a[j]，若在排序之前，a[i]在a[j]前面；并且排序之后，a[i]仍然在a[j]前面。则这个排序算法是稳定的！

```java
public class BubbleSort {

    /*
     * 冒泡排序
     *
     * 参数说明：
     *     a -- 待排序的数组
     *     n -- 数组的长度
     */
    public static void bubbleSort1(int[] a, int n) {
        int i,j;

        for (i=n-1; i>0; i--) {
            // 将a[0...i]中最大的数据放在末尾
            for (j=0; j<i; j++) {

                if (a[j] > a[j+1]) {
                    // 交换a[j]和a[j+1]
                    int tmp = a[j];
                    a[j] = a[j+1];
                    a[j+1] = tmp;
                }
            }
        }
    }

    /*
     * 冒泡排序(改进版)
     *
     * 参数说明：
     *     a -- 待排序的数组
     *     n -- 数组的长度
     */
    public static void bubbleSort2(int[] a, int n) {
        int i,j;
        int flag;                 // 标记

        for (i=n-1; i>0; i--) {

            flag = 0;            // 初始化标记为0
            // 将a[0...i]中最大的数据放在末尾
            for (j=0; j<i; j++) {
                if (a[j] > a[j+1]) {
                    // 交换a[j]和a[j+1]
                    int tmp = a[j];
                    a[j] = a[j+1];
                    a[j+1] = tmp;

                    flag = 1;    // 若发生交换，则设标记为1
                }
            }

            if (flag==0)
                break;            // 若没发生交换，则说明数列已有序。
        }
    }

    public static void main(String[] args) {
        int i;
        int[] a = {20,40,30,10,60,50};

        System.out.printf("before sort:");
        for (i=0; i<a.length; i++)
            System.out.printf("%d ", a[i]);
        System.out.printf("\n");

        bubbleSort1(a, a.length);
        //bubbleSort2(a, a.length);

        System.out.printf("after  sort:");
        for (i=0; i<a.length; i++)
            System.out.printf("%d ", a[i]);
        System.out.printf("\n");
    }
}
```

# 2. 选择排序
从0索引开始，依次和后面元素比较，小的往前放，第一次完毕，最小值出现在了最小索引处

```java
/*
 * 选择排序基本思路：
 * 把第一个元素依次和后面的所有元素进行比较。
 * 第一次结束后，就会有最小值出现在最前面。
 * 依次类推
 */
public void selectSort(int[] arr){
    for (int i = 0; i < arr.length-1; i++) {
        for (int j = i+1; j < arr.length; j++) {
            if (arr[j] < arr[i]){
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    }
}
```

# 3. 插入排序
排序思想：第二个数据依次往前比较，如果发现比自己大，该数据往后瞬移，否则该数据插入找到数据的后面
```java
/*
 * 插入排序基本思想
 * 将n个元素的数列分为已有序和无序两个部分，如插入排序过程示例下所示： 　　
 * {{a1}，{a2，a3，a4，…，an}} 　　
 * {{a1⑴，a2⑴}，{a3⑴，a4⑴ …，an⑴}} 　
 * {{a1(n-1），a2(n-1) ，…},{an(n-1)}} 　　
 * 每次处理就是将无序数列的第一个元素与有序数列的元素从后往前逐个进行比较，
 * 找出插入位置，将该元素插入到有序数列的合适位置中。
 */
public class AlogrithmApp
{

	/**
	 * @param number
	 * @return 1 到nubmer的和
	 */
	public static int sum(int number) {
		if (number < 1) {
			throw new RuntimeException("number must > 0");
		}

		if (number == 1) {
			return 1;
		} else {
			return number + sum(number - 1);
		}

	}

	private static int	counts	= 0;

	public static void moveNumber(int panNum, char a, char b, char c) {
		counts++;
		if (panNum != 1) {
			moveNumber(panNum - 1, a, c, b);
			moveNumber(panNum - 1, b, a, c);
		}
	}

	public static void move(int panNum, char a, char b, char c) {
		if (panNum == 1) {
			System.out.println("盘：" + panNum + " 从 " + a + " 柱移动 " + c + " 柱");
		} else {
			move(panNum - 1, a, c, b);
			System.out.println("盘：" + panNum + " 从 " + a + " 柱移动 " + c + " 柱");
			move(panNum - 1, b, a, c);
		}
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		/*moveNumber(30, 'a', 'b', 'c');

		System.out.println(counts);*/
		
		  int datas[] = new int[100000]; //初始化数据 
		  initData(datas); //打印数据
		  
		  //printData(datas); //插入排序 
		  quickSort(datas, 0, datas.length-1);
		  
		  System.out.println(System.currentTimeMillis());
		  insertSort(datas); //排序后打印 
		  //printData(datas);
		  System.out.println(System.currentTimeMillis());
		 
	}

	public static void selectSort(int[] datas) {
		for (int i = 0; i < datas.length - 1; i++) {
			int minIndex = i;// 最小数据的下标
			for (int j = i + 1; j < datas.length; j++) {
				if (datas[minIndex] > datas[j]) {
					minIndex = j;
				}
			}
			int temp = datas[i];
			datas[i] = datas[minIndex];
			datas[minIndex] = temp;
		}
	}

	/**
	 * 初始化数据
	 * 
	 * @param datas
	 */
	public static void initData(int datas[]) {
		for (int i = 0; i < datas.length; i++) {
			// 随机1到100的值
			datas[i] = (int) (Math.random() * 100) + 1;
		}
	}

	/**
	 * 打印数据
	 * 
	 * @param datas
	 */
	public static void printData(int datas[]) {
		for (int i = 0; i < datas.length; i++) {
			System.out.print(datas[i] + ",");
		}
		System.out.println();
	}

	public static void bubbleSort(int datas[]) {
		int j = 0; // 冒泡的次数
		int i = 0; // 每次冒泡的比较
		for (j = datas.length - 1; j > 0; j--) {
			for (i = 0; i < j; i++) {
				if (datas[i] > datas[i + 1]) {
					int temp = datas[i];
					datas[i] = datas[i + 1];
					datas[i + 1] = temp;
				}
			}
		}
	}

	public static void insertSort(int datas[]) {
		int j = 0; // 第二个数据开始插入的下标
		int i = 0;// 插入的次数
		for (i = 1; i < datas.length; i++) {
			int temp = datas[i];
			for (j = i - 1; j >= 0; j--) {
				if (datas[j] > temp) {
					datas[j + 1] = datas[j];
				} else {
					break;
				}
			}
			// 判断 j == -1 或者 就是第一个小于等于temp数据的位置
			datas[j + 1] = temp;
		}
	}

	/**
	 * 快速排序
	 * 
	 * @param datas
	 */
	public static void quickSort(int datas[], int start, int end) {
		if (start >= end) {
			return;
		} else {
			int middle = findMiddle(datas, start, end);
			quickSort(datas, start, middle - 1);
			quickSort(datas, middle + 1, end);
		}
	}

	private static int findMiddle(int datas[], int start, int end) {
		int temp = datas[end];// 参照物

		int left = start - 1;
		int right = end;

		while (true) {
			// 1.从左边依次找数据，找到第一个比参照大的数据
			while (++left < end && datas[left] <= temp);
				
			if (left == end) {
				//参照物是最大值
				break;
			}
			// 2.从右边依次找数据，找到第一个比参数小的数据
			while (--right >= start && datas[right] >= temp);
				
			// 3. 比较是否出现交叉（left 和 right）
			if (left < right) {
				// 4. 如果没有交叉，交换左右指针位置的数据
				int d = datas[left];
				datas[left] = datas[right];
				datas[right] = d;
			} else {
				// 5. 如果出现交叉，交换左指针和参照物的值，结束
				int d = datas[left];
				datas[left] = datas[end];
				datas[end] = d;
				break;
			}
		}
		return left;
	}

}
```

# 4. 快速排序
快速排序(Quick Sort)使用分治法策略。它的基本思想是：选择一个基准数，通过一趟排序将要排序的数据分割成独立的两部分；其中一部分的所有数据都比另外一部分的所有数据都要小。然后，再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

快速排序流程：

- 从数列中挑出一个基准值。
- 将所有比基准值小的摆放在基准前面，所有比基准值大的摆在基准的后面(相同的数可以到任一边)；在这个分区退出之后，该基准就处于数列的中间位置。
- 递归地把"基准值前面的子数列"和"基准值后面的子数列"进行排序。

选择一个基准数，通过一趟排序将要排序的数据分割成独立的两部分；其中一部分的所有数据都比另外一部分的所有数据都要小。然后，再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

```java
package cn.itcast;

/*
 * 快速排序：
 * 一趟快速排序的算法是： 　　
 * 1）设置两个变量i、j，排序开始的时候：i=0，j=N-1； 　　
 * 2）以第一个数组元素作为关键数据，赋值给key，即 key=A[0]； 　　
 * 3）从j开始向前搜索，即由后开始向前搜索（j=j-1即j--），
 * 找到第一个小于key的值A[j]，A[i]与A[j]交换； 　　
 * 4）从i开始向后搜索，即由前开始向后搜索（i=i+1即i++），
 * 找到第一个大于key的A[i]，A[i]与A[j]交换； 　　
 * 5）重复第3、4、5步，直到 I=J； 
 * (3,4步是在程序中没找到时候j=j-1，i=i+1，直至找到为止。
 * 找到并交换的时候i， j指针位置不变。
 * 另外当i=j这过程一定正好是i+或j-完成的最后令循环结束。） 
 */
public class QuickSort {
	public static void sort(int[] data) {
		quickSort(data, 0, data.length - 1);
	}

	private static void quickSort(int[] data, int i, int j) {
		int pivotIndex = (i + j) / 2;
		// swap
		SortTest.swap(data, pivotIndex, j);

		int k = partition(data, i - 1, j, data[j]);
		SortTest.swap(data, k, j);
		if ((k - i) > 1)
			quickSort(data, i, k - 1);
		if ((j - k) > 1)
			quickSort(data, k + 1, j);

	}

	/**
	 * @param data
	 * @param i
	 * @param j
	 * @return
	 */
	private static int partition(int[] data, int l, int r, int pivot) {
		do {
			while (data[++l] < pivot)
				;
			while ((r != 0) && data[--r] > pivot)
				;
			SortTest.swap(data, l, r);
		} while (l < r);
		SortTest.swap(data, l, r);
		return l;
	}
}

```

```java
/**
 * 快速排序
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     l -- 数组的左边界(例如，从起始位置开始排序，则l=0)
 *     r -- 数组的右边界(例如，排序截至到数组末尾，则r=a.length-1)
 */
public static void quickSort(int[] a, int l, int r) {

    if (l < r) {
        int i, j, x;

        i = l;
        j = r;
        x = a[i];
        while (i < j) {
            while (i < j && a[j] > x)
                j--; // 从右向左找第一个小于x的数
            if (i < j)
                a[i++] = a[j];
            while (i < j && a[i] < x)
                i++; // 从左向右找第一个大于x的数
            if (i < j)
                a[j--] = a[i];
        }
        a[i] = x;
        quickSort(a, l, i - 1); /* 递归调用 */
        quickSort(a, i + 1, r); /* 递归调用 */
    }
}
```
下面以数列a={30,40,60,10,20,50}为例，演示它的快速排序过程

![快速排序](http://images.cnitblog.com/i/497634/201403/121659127078460.jpg)

上图只是给出了第1趟快速排序的流程。在第1趟中，设置x=a[i]，即x=30。
(01) 从"右 --> 左"查找小于x的数：找到满足条件的数a[j]=20，此时j=4；然后将a[j]赋值a[i]，此时i=0；接着从左往右遍历。
(02) 从"左 --> 右"查找大于x的数：找到满足条件的数a[i]=40，此时i=1；然后将a[i]赋值a[j]，此时j=4；接着从右往左遍历。
(03) 从"右 --> 左"查找小于x的数：找到满足条件的数a[j]=10，此时j=3；然后将a[j]赋值a[i]，此时i=1；接着从左往右遍历。
(04) 从"左 --> 右"查找大于x的数：找到满足条件的数a[i]=60，此时i=2；然后将a[i]赋值a[j]，此时j=3；接着从右往左遍历。
(05) 从"右 --> 左"查找小于x的数：没有找到满足条件的数。当i>=j时，停止查找；然后将x赋值给a[i]。此趟遍历结束！

按照同样的方法，对子数列进行递归遍历。最后得到有序数组！

## 4.1 快速排序的时间复杂度和稳定性

### 4.1.2 快速排序稳定性
快速排序是不稳定的算法，它不满足稳定算法的定义。
算法稳定性 -- 假设在数列中存在a[i]=a[j]，若在排序之前，a[i]在a[j]前面；并且排序之后，a[i]仍然在a[j]前面。则这个排序算法是稳定的！

### 4.1.3 快速排序时间复杂度
快速排序的时间复杂度在最坏情况下是O(N2)，平均的时间复杂度是O(N*lgN)。
这句话很好理解：假设被排序的数列中有N个数。遍历一次的时间复杂度是O(N)，需要遍历多少次呢？至少lg(N+1)次，最多N次。
(01) 为什么最少是lg(N+1)次？快速排序是采用的分治法进行遍历的，我们将它看作一棵二叉树，它需要遍历的次数就是二叉树的深度，而根据完全二叉树的定义，它的深度至少是lg(N+1)。因此，快速排序的遍历次数最少是lg(N+1)次。
(02) 为什么最多是N次？这个应该非常简单，还是将快速排序看作一棵二叉树，它的深度最大是N。因此，快读排序的遍历次数最多是N次。

# 5. 堆排序


 堆排序利用了大根堆（或小根堆）堆顶记录的关键字最大（或最小）这一特征， 使得在当前无序区中选取最大（或最小）关键字的记录变得简单

## 5.1 用大根堆排序的基本思想 　　 

- 先将初始文件R[1..n]建成一个大根堆，此堆为初始的无序区 　　
- 再将关键字最大的记录R[1]（即堆顶）和无序区的最后一个 记录R[n]交换，由此得到新的无序区R[1..n-1]和有序区R[n]，
   且满足R[1..n-1].keys≤R[n].key 　　 
- 由于交换后新的根R[1]可能违反堆性质，故应将当前无序区R[1..n-1]调整为堆。
   然后再次将R[1..n-1]中关键字最大的记录R[1]和该区间的最后一个记录R[n-1]交换，
   由此得到新的无序区R[1..n-2]和有序区R[n-1..n]，
   且仍满足关系R[1..n-2].keys≤R[n-1..n].keys，同样要将R[1..n-2]调整为堆。 　 
   直到无序区只有一个元素为止。 　　

## 5.2 大根堆排序算法的基本操作
- 初始化操作：将R[1..n]构造为初始堆
- 每一趟排序的基本操作：将当前无序区的堆顶记录R[1]和该区间的最后一个记录交换， 然后将新的无序区调整为堆（亦称重建堆）


```java
package cn.itcast;

public class HeapSort {
	public static void sort(int[] data) {
		MaxHeap h = new MaxHeap();
		h.init(data);
		for (int i = 0; i < data.length; i++)
			h.remove();
		System.arraycopy(h.queue, 1, data, 0, data.length);
	}

	private static class MaxHeap {

		void init(int[] data) {
			this.queue = new int[data.length + 1];
			for (int i = 0; i < data.length; i++) {
				queue[++size] = data[i];
				fixUp(size);
			}
		}

		private int size = 0;

		private int[] queue;

		public int get() {
			return queue[1];

		}

		public void remove() {
			SortTest.swap(queue, 1, size--);
			fixDown(1);
		}

		// fixdown
		private void fixDown(int k) {
			int j;
			while ((j = k << 1) <= size) {
				if (j < size && queue[j] < queue[j + 1])
					j++;
				if (queue[k] > queue[j]) // 不用交换

					break;
				SortTest.swap(queue, j, k);
				k = j;
			}
		}

		private void fixUp(int k) {
			while (k > 1) {
				int j = k >> 1;
				if (queue[j] > queue[k])
					break;
				SortTest.swap(queue, j, k);

				k = j;
			}
		}

	}
}
```

# 6. 归并排序

```java
package cn.itcast;

/*
 * 归并操作(merge)，也叫归并算法，指的是将两个已经排序的序列合并成一个序列的操作。 　　
 * 如设有数列{6，202，100，301，38，8，1} 　　
 * 初始状态： [6] [202] [100] [301] [38] [8] [1] 比较次数 　　
 * i=1 [6 202 ] [ 100 301] [ 8 38] [ 1 ]　3 　　
 * i=2 [ 6 100 202 301 ] [ 1 8 38 ]　4 　　
 * i=3　[ 1 6 8 38 100 202 301 ] 4 
 */
public class MergeSort {
	public static void sort(int[] data) {
		int[] temp = new int[data.length];
		mergeSort(data, temp, 0, data.length - 1);
	}

	private static void mergeSort(int[] data, int[] temp, int l, int r) {
		int mid = (l + r) / 2;
		if (l == r)
			return;
		mergeSort(data, temp, l, mid);
		mergeSort(data, temp, mid + 1, r);

		for (int i = l; i <= r; i++) {
			temp[i] = data[i];
		}
		int i1 = l;
		int i2 = mid + 1;
		for (int cur = l; cur <= r; cur++) {
			if (i1 == mid + 1)
				data[cur] = temp[i2++];
			else if (i2 > r)
				data[cur] = temp[i1++];
			else if (temp[i1] < temp[i2])
				data[cur] = temp[i1++];
			else

				data[cur] = temp[i2++];
		}
	}
}

```

# 7. 希尔排序

```java
package cn.itcast;

/*
 * 希尔排序：先取一个小于n的整数d1作为第一个增量，
 * 把文件的全部记录分成（n除以d1）个组。所有距离为d1的倍数的记录放在同一个组中。
 * 先在各组内进行直接插入排序；然后，取第二个增量d2<d1重复上述的分组和排序，
 * 直至所取的增量dt=1(dt<dt-l<…<d2<d1)，即所有记录放在同一组中进行直接插入排序为止。 
 */
public class ShellSort {
	public static void sort(int[] data) {
		for (int i = data.length / 2; i > 2; i /= 2) {
			for (int j = 0; j < i; j++) {
				insertSort(data, j, i);
			}
		}
		insertSort(data, 0, 1);
	}

	/**
	 * @param data
	 * @param j
	 * @param i
	 */
	private static void insertSort(int[] data, int start, int inc) {
		for (int i = start + inc; i < data.length; i += inc) {
			for (int j = i; (j >= inc) && (data[j] < data[j - inc]); j -= inc) {
				SortTest.swap(data, j, j - inc);
			}
		}
	}
}
/*
 * 属于插入类排序,是将整个无序列分割成若干小的子序列分别进行插入排序 　　 
 * 排序过程：先取一个正整数d1<n，把所有序号相隔d1的数组元素放一组，
 * 组内进行直接插入排序；然后取d2<d1，重复上述分组和排序操作；直至di=1， 即所有记录放进一个组中排序为止 　　 
 * 初始：d=5 　　49 38 65 97 76 13 27 49 55 04 　　 
 * 49 13 　　|-------------------| 　　 
 * 38 27     |-------------------| 　　 
 * 65 49 　　|-------------------| 　　 
 * 97 55     |-------------------| 　　 
 * 76 04 　　|-------------------| 　　 
 * 一趟结果 　　13 27 49 55 04 49 38 65 97 76 　　 
 * d=3 　　 13 27 49  55 04 49 38 65 97 76 　　 
 * 13 55 38 76 |------------|------------|------------| 　　 
 * 27 04 65 |------------|------------| 　　 
 * 49 49 97 |------------|------------| 　　
 * 二趟结果  13 04 49* 38 27 49 55 65 97 76 　　 
 * d=1 　　13 04 49 38 27 49 55 65 97 76
 * 　　 |----|----|----|----|----|----|----|----|----| 　　 三趟结果 　　
 * 04 13 27 38 49 49 55 65 76 97
 */
```

```java
package cn.itcast;

import java.util.Arrays;

public class SortTest {

	public static void main(String[] args) {
		int[] arr = { 2, 5, 3, 1, 4 };
		System.out.println("排序前：" + Arrays.toString(arr));
		// InsertSort.sort(arr);
		// BubbleSort.sort(arr);
		// SelectionSort.sort(arr);
		// ShellSort.sort(arr);
		// QuickSort.sort(arr);
		// MergeSort.sort(arr);
		// HeapSort.sort(arr);
		System.out.println("排序后：" + Arrays.toString(arr));
	}

	/*
	 * 交换数组中的两个元素
	 */
	public static void swap(int[] data, int i, int j) {
		int temp = data[i];
		data[i] = data[j];
		data[j] = temp;
	}
}
```

# [7. 面试中的排序算法总结](http://www.cnblogs.com/wxisme/p/5243631.html)