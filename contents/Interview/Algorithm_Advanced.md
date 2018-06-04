# 高级篇

## 斐波那契数列
又称兔子数列，或者黄金分割数列。指的是这样一个数列：
0、1、1、2、3、5、8、13、21……从第三项起，它的每一项都等于前两项的和。
即：F(0)=1，F(1)=1; F(n) = F(n-1) + F(n-2) (n>=2);

### 解法1
```
public class FibonacciDemo {

    public static void main(String[] args) {

        FibonacciDemo fibonacciDemo = new FibonacciDemo();
        System.out.println(fibonacciDemo.fib(9));
    }

    public int fib(int n) {
        int a=0, b=1, sum = 0;
        for (int i=1; i<n; i++) {
            sum = a + b;
            a = b;
            b = sum;
        }
        return b;
    }
}
```

### 解法2
数组递归求解
```
public class FibonacciDemo {

    public static void main(String[] args) {

        FibonacciDemo fibonacciDemo = new FibonacciDemo();
        System.out.println(fibonacciDemo.fib(9));
    }

    public int fib(int n) {
        int[] arr = new int[n+1];
        arr[0]=0;
        arr[1]=1;
        for(int i=2; i<=n; i++){
            arr[i] = arr[i-1]+arr[i-2];
        }
        return arr[n];
    }
}
```

### 解法3
递归求解
```
public class FibonacciDemo {

    public static void main(String[] args) {

        FibonacciDemo fibonacciDemo = new FibonacciDemo();
        System.out.println(fibonacciDemo.fib(8));
    }

    public int fib(int n) {
        if (n==0) {
            return 0;
        } else if (n==1) {
            return 1;
        }
        return fib(n-1) + fib(n-2);
    }
}
```


