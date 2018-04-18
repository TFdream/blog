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

### 5.冒泡排序

### 6.快速排序

### 7.归并排序

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

    public void swap(int[] arr, int i, int j) {
        if (i == j) {
            return;
        }
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

### 参考
[一遍记住Java常用的八种排序算法与代码实现](https://www.jianshu.com/p/5e171281a387)
