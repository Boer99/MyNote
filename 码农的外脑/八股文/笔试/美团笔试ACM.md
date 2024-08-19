
经验：
- 记录常用的导包
- 可以适当sout看看每一部分的结果是否正确
- 

# 美团2024年春招第一场笔试【技术】

## 1. 小美的平衡矩阵

## 2. 小美的数组询问

小美拿到了一个由正整数组成的数组，但其中有一些元素是未知的（用 0 来表示）。  
现在小美想知道，如果那些未知的元素在区间 `[l,r]` 范围内随机取值的话，数组所有元素之和的最小值和最大值分别是多少？  

共有 q 次询问。

![550](assets/Pasted%20image%2020240405214618.png)

```java
import java.util.Scanner;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int q = in.nextInt();
        // System.out.println(n);
        // System.out.println(q);
        int[] nums = new int[n];
        long sum = 0;
        long count0 = 0;
        for (int i = 0; i < n; i++) {
            nums[i] = in.nextInt();
            if (nums[i] == 0) {
                count0++;
                continue;
            }
            sum += nums[i];
            // System.out.print(nums[i]);
        }
        while (q-- > 0) {
            int n1 = in.nextInt();
            int n2 = in.nextInt();
            System.out.println((sum + n1 * count0) + " " + (sum + n2 * count0));
        }
    }
}
```

注意：

- sum 和 count0 要用 long 类型，输入用例的数字很大，int 会溢出
- 时间把控很严格，不要有多余的变量和步骤。例如 `while (q-- > 0)` 写成 `for (int i = 0; i < q; i++)` 就差一个测试用例运行不完

## 3. 小美的 MT

![600](assets/Pasted%20image%2020240405215652.png)

#Bo

```java
import java.util.Scanner;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int k = in.nextInt();
        String s = in.next();
        int count1 = 0;
        int count2 = 0;
        for (int i = 0; i < n; i++) {
            if (s.charAt(i) == 'M' || s.charAt(i) == 'T') {
                count1++;
                continue;
            }
            if (k > 0) {
                count2++;
                k--;
            }
        }
        System.out.println(count1 + count2);
    }
}
```

参考网上答案如下，省去了一些判断，但是运行时间没区别

```java
import java.util.Scanner;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int k = in.nextInt();
        String s = in.next();
        int countNotMT = 0;
        for (int i = 0; i < n; i++) {
            if (s.charAt(i) != 'M' && s.charAt(i) != 'T') {
                countNotMT++;
            }
        }
        System.out.println(n - countNotMT + Math.min(countNotMT, k));
    }
}
```

## 4. 小美的朋友关系

放

## 5. 小美的区间删除


