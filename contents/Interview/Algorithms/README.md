# 算法篇

## 1.归并排序
归并排序算法伪代码如下：
```
// 归并排序算法, A是数组，n表示数组大小
merge_sort(A, n) {
  merge_sort_c(A, 0, n-1)
}

// 递归调用函数
merge_sort_c(A, p, r) {
  // 递归终止条件
  if p >= r  then return

  // 取p到r之间的中间位置q
  q = (p+r) / 2
  // 分治递归
  merge_sort_c(A, p, q)
  merge_sort_c(A, q+1, r)
  // 将A[p...q]和A[q+1...r]合并为A[p...r]
  merge(A[p...r], A[p...q], A[q+1...r])
}
```

java实现代码如下：
```
/**
 * @author Ricky Fung
 */
public class MergeSort {

    // 归并排序算法, A是数组，n表示数组大小
    public void mergeSort(int[] arr, int n) {
        merge_sort_c(arr, 0, n-1);
    }

    // 递归调用函数
    private void merge_sort_c(int[] arr, int p, int r) {
        // 递归终止条件
        if (p >= r)
            return;

        // 取p到r之间的中间位置q
        int q = (p+r) / 2;
        // 分治递归
        merge_sort_c(arr, p, q);
        merge_sort_c(arr, q+1, r);

        // 将A[p...q]和A[q+1...r]合并为A[p...r]
        merge(arr, p, q, r);
    }

    /**
     * merge函数的作用就是，将已经有序的 A[p…q]和 A[q+1…r]合并成一个有序的数组，并且放入 A[p…r]。
     * @param arr
     * @param p
     * @param q
     * @param r
     */
    private void merge(int[] arr, int p, int q, int r) {
        int i = p, j = q+1, k = 0; // 初始化变量i, j, k
        int[] tmp = new int[r-p+1]; // 申请一个大小跟A[p...r]一样的临时数组
        while (i<=q && j<=r) {
            if (arr[i] <= arr[j]) {
                tmp[k++] = arr[i++]; // i++等于i = i+1
            } else {
                tmp[k++] = arr[j++];
            }
        }
        // 判断哪个子数组中有剩余的数据
        int start = i, end = q;
        if (j <= r ) { //A[q+1…r] 有剩余
            start = j;
            end = r;
        }

        // 将剩余的数据拷贝到临时数组tmp
        while (start <= end) {
            tmp[k++] = arr[start++];
        }

        // 将tmp中的数组拷贝回A[p...r]
        for (i=0; i<tmp.length; i++) {
            arr[p+i] = tmp[i];
        }
    }
}
```

## 2.快排
伪代码如下：
```
// 快速排序，A是数组，n表示数组的大小
quick_sort(A, n) {
  quick_sort_c(A, 0, n-1)
}
// 快速排序递归函数，p,r为下标
quick_sort_c(A, p, r) {
  if p >= r then return
  
  q = partition(A, p, r) // 获取分区点
  quick_sort_c(A, p, q-1)
  quick_sort_c(A, q+1, r)
}
```

1.递归实现
```
/**
 * @author Ricky Fung
 */
public class QuickSort {

    // 快速排序，A是数组，n表示数组的大小
    public void quickSort(int[] A, int n) {
        quick_sort_c(A, 0, n-1);
    }

    // 快速排序递归函数，p,r为下标
    private void quick_sort_c(int[] A, int p, int r) {
        if (p >= r)
            return;

        int q = partition(A, p, r); // 获取分区点
        quick_sort_c(A, p, q-1);
        quick_sort_c(A, q+1, r);
    }

    private int partition(int[] A, int p, int r) {
        //随机选择一个元素作为 pivot（一般情况下，可以选择 p 到 r 区间的最后一个元素）
        int pivot = A[r];
        //然后对 A[p…r]分区，函数返回 pivot 的下标。
        int i = p;
        for (int j=p; j<=r-1; j++) {
            if (A[j] < pivot) {
                ArrayUtils.swap(A, i, j);
                i = i+1;
            }
        }
        ArrayUtils.swap(A, i, r);
        return i;
    }
}
```

2.非递归
```

```

## 3.堆排序
```
/**
 * @author Ricky Fung
 */
public class HeapSort {

    public static void main(String[] args) {
        int[] arr = {6, 2, 9, 5, 4, 7, 12, 8, 10};

        //排序
        sort(arr, arr.length);

        PrintUtils.printArray(arr);
    }

    // n表示数据的个数，数组a中的数据从下标1到n的位置。
    public static void sort(int[] arr, int n) {
        int[] a = new int[n+1];
        System.arraycopy(arr, 0, a, 1, n);

        buildHeap(a, n);
        int k = n;
        while (k > 1) {
            ArrayUtils.swap(a, 1, k);
            --k;
            heapify(a, k, 1);
        }

        //
        System.arraycopy(a, 1, arr, 0, n);
    }

    //建堆
    private static void buildHeap(int[] a, int n) {
        for (int i = n/2; i >= 1; --i) {
            heapify(a, n, i);
        }
    }

    //堆化
    private static void heapify(int[] a, int n, int i) {
        while (true) {
            int maxPos = i;
            if (i*2 <= n && a[i] < a[i*2]) {
                maxPos = i * 2;
            }
            if (i*2+1 <= n && a[maxPos] < a[i*2+1]) {
                maxPos = i * 2 + 1;
            }
            if (maxPos == i) {
                break;
            }
            ArrayUtils.swap(a, i, maxPos);
            i = maxPos;
        }
    }
}
```
