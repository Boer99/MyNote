### 2.4.4 动态规划与贪婪算法
#### 面试题14：剪绳子

![](assets/Pasted%20image%2020231121113522.png)

> 同leetcode[343. 整数拆分](https://leetcode.cn/problems/integer-break/)

**一、动态规划**

首先定义函数f(n)为把长度为n的绳子剪成若干段后各段长度乘积的最大值。在剪第一刀的时候，我们有 n-1 种可能的选择，也就是剪出来的第
段绳子的可能长度分别为`1,2,...,n-1`。因此`f(n)=max(f(i))xf(n-i))`，其中`0<i<n`。

这是一个从上至下的递归公式。由于递归会有很多重复的子问题，从而有大量不必要的重复计算。一个更好的办法是按照从下而上的顺序计算，也就是说我们==先得到f(2)、f(3)，再得到 f(4)、f(5)，直到得到f(n)==

为什么i要从4开始？从4开始 `f(n)>=n`
```java
int maxProductAfterCutting_solution1(int n) { // n>=2
    if (n < 2) return 0;  
    if (n == 2) return 1;  
    if (n == 3) return 2;  
  
    int[] products = new int[n + 1]; // 多一个位置方便对齐  
    products[0] = 0;  
    products[1] = 1;  
    products[2] = 2;  
    products[3] = 3;  
  
    int max = 0;  
    for (int i = 4; i < n + 1; i++) {  
        max = 0;  
        for (int j = 1; j <= i / 2; j++) {  
            int product = products[j] * products[i - j];  
            max = Math.max(product, max);  
        }  
        products[i] = max;  
    }  
    return products[n];  
}
```

复杂度分析：
- 时间：O(n^2)，每个整数都要计算`products[i]`，`n*(n/2)`
- 空间：O(n)，

也可以不考虑数学问题
```java
public int integerBreak(int n) {
        int[] dp = new int[n + 1];
        for (int i = 2; i <= n; i++) {
            int curMax = 0;
            for (int j = 1; j < i; j++) {
                curMax = Math.max(curMax, Math.max(j * (i - j), j * dp[i - j]));
            }
            dp[i] = curMax;
        }
        return dp[n];
    }

作者：力扣官方题解
```


**二、贪婪算法**

当n>5时，我们尽可能多地剪长度为3的绳子；当剩下的绳子长度为4时，把绳子剪成两段长度为2的绳子。
```java
static int maxProductAfterCutting_solution2(int n) {  
//        if (n < 2) return 0;  
	if (n == 2) return 1;  
	if (n == 3) return 2;  

	int timesOf3 = n / 3;  
	int rem = n % 3;  
	if (rem == 1) {  
		timesOf3 -= 1;  
	}  
	int timesOf2 = (n - timesOf3 * 3) / 2;  
	return (int) Math.pow(3, timesOf3) * (int) Math.pow(2, timesOf2);  
}
```
复杂度分析：
- 时间：O(1)
- 空间：O(1)

```java
class Solution {
    public int integerBreak(int n) {
        if (n <= 3) {
            return n - 1;
        }
        int quotient = n / 3;
        int remainder = n % 3;
        if (remainder == 0) {
            return (int) Math.pow(3, quotient);
        } else if (remainder == 1) {
            return (int) Math.pow(3, quotient - 1) * 4;
        } else {
            return (int) Math.pow(3, quotient) * 2;
        }
    }
}

作者：力扣官方题解
```
