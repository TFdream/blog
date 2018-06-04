
## 八大排序算法

- 插入排序
  - 直接插入排序
  - 希尔排序
- 选择排序
  - 简单选择排序
  - 堆排序
- 交换排序
  - 冒泡排序
  - 快速排序
- 归并排序
- 基数排序

### 1.直接插入排序
将一个记录插入到已排序好的有序表中，从而构成一个新的有序序列。即：先将序列的第1个记录看成是一个有序的子序列，然后从第2个记录逐个进行插入，直至整个序列有序为止。
#### 要点：设立哨兵，作为临时存储和判断数组边界之用。

#### 代码实现

```
    public void insertSort(int[] arr){
        int length = arr.length;//数组长度
        for(int i=1; i<length; i++){    //因为第一次不用，所以从1开始
            int temp = arr[i]; //待插入元素
            int j = i-1;    //有序队列元素个数
            while(j>=0 && arr[j]>temp){//从后往前循环，将大于insertNum的数向后移动
                arr[j+1] = arr[j];//元素向后移动
                j--;
            }
            arr[j+1] = temp;//找到位置，插入当前元素
        }
    }
```

或者：
```
    public void insertSort(int[] arr){
        int length = arr.length;//数组长度
        for(int i=1; i<length; i++){    //因为第一次不用，所以从1开始
            int j;
            int temp = arr[i];  //temp为待插入元素
            for(j=i; j>0 && temp<arr[j-1]; j--){//通过循环，逐个后移一位找到要插入的位置。
                arr[j] = arr[j-1];
            }
            arr[j]=temp;//插入
        }
    }
```

时间复杂度：O（n^2）.

如果碰见一个和插入元素相等的，那么插入元素把想插入的元素放在相等元素的后面。所以，相等元素的前后顺序没有改变，从原无序序列出去的顺序就是排好序后的顺序，所以插入排序是稳定的。

### 2.希尔排序
先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录“基本有序”时，再对全体记录进行依次直接插入排序。

即：先将要排序的一组记录按某个增量d（n/2,n为要排序数的个数）分成若干组子序列，每组中记录的下标相差d.对每组中全部元素进行直接插入排序，然后再用一个较小的增量（d/2）对它进行分组，在每组中再进行直接插入排序。继续不断缩小增量直至为1，最后使用直接插入排序完成排序。

#### 代码实现
```

    public  void shellSort(int[] arr) {
        int d  = arr.length;
        while (d!=0) {
            d=d/2;
            for (int x = 0; x < d; x++) {   //分的组数
                for (int i = x + d; i < arr.length; i += d) {   //组中的元素，从第二个数开始
                    int j = i - d;  //j为有序序列最后一位的位数
                    int temp = arr[i];  //要插入的元素
                    for (; j >= 0 && temp < arr[j]; j -= d) {   //从后往前遍历。
                        arr[j + d] = arr[j];    //向后移动d位
                    }
                    arr[j + d] = temp;
                }
            }
        }
    }
```

### 3.简单选择排序
在要排序的一组数中，选出最小（或者最大）的一个数与第1个位置的数交换；然后在剩下的数当中再找最小（或者最大）的与第2个位置的数交换，依次类推，直到第n-1个元素（倒数第二个数）和第n个元素（最后一个数）比较为止。

步骤：
* 遍历整个序列，将最小的数放在最前面；
* 遍历剩下的序列，将最小的数放在最前面；
* 重复第二步，直到只剩下一个数。

#### 代码实现
```
/**
 * @author Ricky Fung
 */
public class SelectSort {

    public static void main(String[] args) {

        int[] arr = {9, 3, 5, 1, 6, 7, 10};
        ArrayUtils.printArray(arr);

        SelectSort selectSort = new SelectSort();
        selectSort.selectSort(arr);

        ArrayUtils.printArray(arr);

    }

    public void selectSort(int[] arr) {
        int length = arr.length;
        for (int i=0; i < length; i++) {//循环次数
            int key = arr[i];
            int position = i;
            for (int j = i + 1; j < length; j++) {  //选出最小的值和位置
                if (arr[j] < key) {
                    key = arr[j];
                    position = j;
                }
            }
            arr[position] = arr[i];//交换位置
            arr[i]=key;
        }
    }

}
```

### 4.堆排序
对简单选择排序的优化。
步骤：
* 将序列构建成大顶堆。
* 将根节点与最后一个节点交换，然后断开最后一个节点。
* 重复第一、二步，直到所有节点断开。

#### 代码实现
```
/**
 * @author Ricky Fung
 */
public class HeapSort {

    public static void main(String[] args) {

        int[] arr = {9, 3, 5, 1, 6, 7, 10, 30, 4, 2, 11, 40};
        ArrayUtils.printArray(arr);

        HeapSort heapSort = new HeapSort();
        heapSort.heapSort(arr);

        ArrayUtils.printArray(arr);
    }

    public void heapSort(int[] arr) {
        int length = arr.length;
        //循环建堆
        for(int i=0;i<length-1; i++){
            //建堆
            buildMaxHeap(arr, length-1-i);

            //交换堆顶和最后一个元素
            ArrayUtils.swap(arr,0, length-1-i);
        }
    }

    //对data数组从0到lastIndex建大顶堆
    private void buildMaxHeap(int[] data, int lastIndex) {
        //从lastIndex处节点（最后一个节点）的父节点开始
        for(int i=(lastIndex-1)/2;i>=0;i--){
            //k保存正在判断的节点
            int k=i;
            //如果当前k节点的子节点存在
            while(k*2+1<=lastIndex){
                //k节点的左子节点的索引
                int biggerIndex=2*k+1;
                //如果biggerIndex小于lastIndex，即biggerIndex+1代表的k节点的右子节点存在
                if(biggerIndex<lastIndex){
                    //若果右子节点的值较大
                    if(data[biggerIndex]<data[biggerIndex+1]){
                        //biggerIndex总是记录较大子节点的索引
                        biggerIndex++;
                    }
                }
                //如果k节点的值小于其较大的子节点的值
                if(data[k]<data[biggerIndex]){
                    ArrayUtils.swap(data,k,biggerIndex);
                    //将biggerIndex赋予k，开始while循环的下一次循环，重新保证k节点的值大于其左右子节点的值
                    k=biggerIndex;
                }else{
                    break;
                }
            }
        }
    }
}
```

### 5.冒泡排序
#### 基本思想

在要排序的一组数中，对当前还未排好序的范围内的全部数，自上而下对相邻的两个数依次进行比较和调整，让较大的数往下沉，较小的往上冒。即：每当两相邻的数比较后发现它们的排序与排序要求相反时，就将它们互换。

步骤：
* 将序列中所有元素两两比较，将最大的放在最后面。
* 将剩余序列中所有元素两两比较，将最大的放在最后面。
* 重复第二步，直到只剩下一个数。

#### 代码实现
```
/**
 * @author Ricky Fung
 */
public class BubbleSort {

    public static void main(String[] args) {
        int[] arr = {9, 3, 5, 1, 6, 7, 10, 30, 4, 2, 11, 40};
        ArrayUtils.printArray(arr);

        BubbleSort bubbleSort = new BubbleSort();
        bubbleSort.bubbleSort(arr);

        ArrayUtils.printArray(arr);
    }

    public void bubbleSort(int[] arr){
        int length = arr.length;
        for(int i=0; i<length; i++){
            for(int j=0; j<length-i-1; j++){
                if(arr[j]>arr[j+1]){
                    //交换两个元素位置
                    ArrayUtils.swap(arr, j, j+1);
                }
            }
        }
    }

}
```

#### 冒泡排序算法的改进

对冒泡排序常见的改进方法是加入一标志性变量exchange，用于标志某一趟排序过程中是否有数据交换，如果进行某一趟排序时并没有进行数据交换，则说明数据已经按要求排列好，可立即结束排序，避免不必要的比较过程。

设置一标志性变量pos,用于记录每趟排序中最后一次进行交换的位置。由于pos位置之后的记录均已交换到位,故在进行下一趟排序时只要扫描到pos位置即可。

改进后算法如下:
```
/**
 * @author Ricky Fung
 */
public class BubbleSort {

    public static void main(String[] args) {
        int[] arr = {9, 3, 5, 1, 6, 7, 10, 30, 4, 2, 11, 40};
        ArrayUtils.printArray(arr);

        BubbleSort bubbleSort = new BubbleSort();
        bubbleSort.bubbleSort(arr);

        ArrayUtils.printArray(arr);
    }

    public void bubbleSort(int[] arr){
        int length = arr.length;
        int i= length -1;  //初始时,最后位置保持不变
        while ( i> 0) {
            int pos= 0; //每趟开始时,无记录交换
            for (int j= 0; j< i; j++)
                if (arr[j]> arr[j+1]) {
                    pos= j; //记录交换的位置
                    ArrayUtils.swap(arr, j, j+1);
                }
            i= pos; //为下一趟排序作准备
        }
    }

}
```

### 6.快速排序
#### 基本思想
* 选择一个基准元素p（通常选择第一个元素或者最后一个元素），小于p的数放在左边，大于p的数放在右边；
* 递归的将p左边和右边的数都按照第一步进行，直到不能递归。

#### 时间复杂度
快速排序是通常被认为在同数量级（O(nlog2n)）的排序方法中平均性能最好的。但若初始序列按关键码有序或基本有序时，快排序反而蜕化为冒泡排序。

#### 代码实现
1. 递归版本
```
package com.mindflow;

/**
 * @author Ricky Fung
 */
public class QuickSort {

    public static void main(String[] args) {

        int[] arr = {9, 3, 5, 1, 6, 7, 10, 30, 4, 2, 11, 40};
        ArrayUtils.printArray(arr);

        QuickSort quickSort = new QuickSort();
        quickSort.quickSort(arr, 0, arr.length-1);

        ArrayUtils.printArray(arr);
    }

    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivot = arr[low]; // 选定的基准值（第一个数值作为基准值）
            int pos = low;
            for (int i = low + 1; i <= high; i++) {
                if (arr[i] < pivot) {
                    pos++;
                    ArrayUtils.swap(arr, i, pos);
                }
            }
            ArrayUtils.swap(arr, low, pos);
            // 分而治之
            quickSort(arr, low, pos - 1);// 排序左半部分
            quickSort(arr, pos + 1, high);// 排序右半部分
        }
    }

}

```

2. 非递归版本
```

```

### 7.归并排序
速度仅次于快排，内存少的时候使用，可以进行并行计算的时候使用。

#### 基本思想
归并（Merge）排序法是将两个（或两个以上）有序表合并成一个新的有序表，即把待排序序列分为若干个子序列，每个子序列是有序的。然后再把有序子序列合并为整体有序序列。

#### 代码实现
```

```

### ArrayUtils
```

/**
 * @author Ricky Fung
 */
public class ArrayUtils {

    public static void printArray(int[] arr) {
        for (int i=0; i<arr.length; i++) {
            if(i==arr.length-1) {
                System.out.println(" "+ arr[i]);
            } else {
                System.out.print(" " + arr[i]);
            }
        }
    }

    public static void swap(int[] arr, int i, int j) {
        if (i == j) {
            return;
        }
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

## 参考
[一遍记住Java常用的八种排序算法与代码实现](https://www.jianshu.com/p/5e171281a387)


