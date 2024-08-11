
> 参考：
> - 代码随想录 https://programmercarl.com/
> - leetcode 题解
> - 牛客面经收集题目

# 经验

- 字符串尽量转换为 char 数组操作，String 类的方法很容易弄错
- dp 五部曲寻找递归的最好方式就是模拟 dp 数组
- 数学公式：
	- 概率：组合：$C_{n}^{m}=\frac{n!}{m!(n-m)!}$ ，排列：$P_{n}^{m}=\frac{n!}{(n-m)!}$

# 大厂手撕收集

未知：
- 链表实现栈
- 因数分解 

未处理：
- [43. 字符串相乘 - 力扣（LeetCode）](https://leetcode.cn/problems/multiply-strings/description/)
- 有向无环图，找 A 到 B 的最短路径
- 力扣 440 [字典序的第K小数字](https://leetcode.cn/problems/k-th-smallest-in-lexicographical-order/)
- 比 n 大的最小回文数
- 实现一个 ArrayList
- 字符串数组的最大公共前缀并给出时空复杂度
- ipv4 地址编码、解码 #携程
	- 编码: 将 `ipv4` 使用一个 `int` 类型存储
	- 解码: `int` 类型解码为 `ipv4` 地址

树
- 给定数组，判断其是否可能是二叉搜索树的后序遍历序列
- 二叉树展开为链表 

回溯
- 八皇后问题


# 数组

## ---------- 归并

## 合并两个有序数组 #手撕2 #rep

[88. 合并两个有序数组 - 力扣（LeetCode）](https://leetcode.cn/problems/merge-sorted-array/)

思路：

- × num1 中的元素后移，nums1 和 nums2 做归并，选最小值
- √ nums1 和 nums2 做归并，选最大值

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int k = nums1.length - 1;
        m--;
        n--;
        while (m >= 0 && n >= 0) {
            if (nums1[m] > nums2[n]) {
                nums1[k--] = nums1[m--];
            } else {
                nums1[k--] = nums2[n--];
            }
        }
        while (m >= 0) {
            nums1[k--] = nums1[m--];
        }
        while (n >= 0) {
            nums1[k--] = nums2[n--];
        }
    }
}
```

## ---------- 二分

## 二分查找

> 给定一个 n 个元素**有序的**（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在**返回下标**，否则返回 -1。
> 
> 1. 你可以假设 `nums` 中的所有元素是不重复的。
> 2. `n` 将在 `[1, 10000]`之间。
> 3. `nums` 的每个元素都将在 `[-9999, 9999]`之间。
>    
>  https://leetcode.cn/problems/binary-search

**二分的前提条件：** 有序、无重复

```java
// 双向指针
public int search(int[] nums, int tg) {
	int len = nums.length;
	for (int l = 0, r = len - 1; l <= r; ){
		int mid = (l + r) / 2;
		if (nums[mid] == tg) {
			return mid;
		}
		if (nums[mid] > tg) {
			r = mid - 1;
		} else {
			l = mid + 1;
		}
	}      
	return -1;
}
```

> 时间：`O(log n)`
> 
> 空间：`O(1)`

## 有序数组中的单一元素 #手撕 #rep

[540. 有序数组中的单一元素 - 力扣（LeetCode）](https://leetcode.cn/problems/single-element-in-a-sorted-array/description/)

你设计的解决方案必须满足 `O(log n)` 时间复杂度和 `O(1)` 空间复杂度。

---

思路：二分，根据当前二分点 mid 的奇偶性进行分情况讨论

- mid 为偶数下标：
	- 正常情况下偶数下标的值会与下一值相同，可以确保 mid 之前并没有插入单一元素，此时应该更新 l=mid，否则让 r=mid−1，
	- 非正常情况，让 l=mid+1 和 r=mid；
- mid 为奇数下标：正常情况下奇数下标的值会与上一值相同，因此如果满足该条件，可以确保 mid 之前并没有插入单一元素，相应的更新 l 和 r

```java
class Solution {
    public int singleNonDuplicate(int[] nums) {
        int n = nums.length;
        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + r >> 1;
            if (mid % 2 == 0) {
                if (mid + 1 < n && nums[mid] == nums[mid + 1]) l = mid + 1;
                else r = mid;
            } else {
                if (mid - 1 >= 0 && nums[mid - 1] == nums[mid]) l = mid + 1;
                else r = mid;
            }
        }
        return nums[r];
    }
}
```

## 寻找峰值 #手撕 #rep

[162. 寻找峰值 - 力扣（LeetCode）](https://leetcode.cn/problems/find-peak-element/description/)

思路：

- 基于两个结论（证明省略）
	- 对于任意数组而言，一定存在峰值（一定有解）
	- 二分不会错过峰值

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int n = nums.length;
        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + r >> 1;
            if (nums[mid] > nums[mid + 1])
                r = mid; // mid可能是峰值
            else
                l = mid + 1; // mid不可能是峰值
        }
        return r;
    }
}
```


## 移除元素

> 给你一个数组 nums 和一个值 val，你需要 **原地** 移除所有数值等于 val 的元素，并**返回移除后数组的新长度**。
> - 不要使用额外的数组空间，你必须仅使用 **O(1) 额外空间**并原地修改输入数组。
> - 元素的顺序可以改变。你**不需要考虑数组中超出新长度后面的元素**。
> 
> 说明：
> - `0 <= nums.length <= 100` (包括空数组)
> - `0 <= nums[i] <= 50`
> - `0 <= val <= 100`
> 
> **示例 2：**
> 输入：`nums = [0,1,2,2,3,0,4,2]`, `val = 2`
> 输出：5, `nums = [0,1,3,0,4]`
> 解释：函数应该返回新的长度 `5`, 并且 nums 中的前五个元素
> `[0,1,3,0,4]`。注意这五个元素可为任意顺序。你不需要考虑数组中超出新长度后面的元素。
> 
> [27. 移除元素 - 力扣（LeetCode）](https://leetcode.cn/problems/remove-element/)

**特例：**`1,1,1...,1`  (一个数组里全是要删除的)

```java
public class Solution {
    public static void main(String[] args) {
        // 随机生成nums1
        int[] nums1 = new int[1000000];
        int max = 50, min = 0;
        for (int i = 0; i < nums1.length; i++) {
            // 生成随机整数，范围在[minValue, maxValue]
            nums1[i] = new Random().nextInt((max - min + 1)) + min;
        }
        for (int i : nums1) {
            System.out.print(i + " ");
        }
        System.out.println();
        // 硬拷贝nums1给nums2
        int[] nums2 = Arrays.copyOf(nums1, nums1.length);

        int val = 10;

        TestTimeConsumingUtils t1 = new TestTimeConsumingUtils();
        System.out.println("快慢双指针结果：" + removeElement(nums1, val));
        t1.printTime("快慢双指针");

        TestTimeConsumingUtils t2 = new TestTimeConsumingUtils();
        System.out.println("相向双指针结果：" + removeElement2(nums2, val));
        t2.printTime("相向双指针");
    }

    /**
     * 快慢双指针
     */
    public static int removeElement(int[] nums, int val) {
        int slowIndex = 0;
        for (int fastIndex = 0; fastIndex < nums.length; fastIndex++) {
            if (nums[fastIndex] != val) {
                nums[slowIndex] = nums[fastIndex];
                slowIndex++;
            }
        }
        return slowIndex;
    }

    /**
     * 相向双指针
     */
    public static int removeElement2(int[] nums, int val) {
        int len = nums.length;
        if (len == 0 || (len == 1 && nums[0] == val)) return 0;
        int l = 0;
        for (int r = len - 1; l <= r; ) {
            if (nums[l] != val) {
                l++;
                continue;
            }
            while (nums[r] == val && r > 0) {
                r--;
            }
            nums[l] = nums[r--];
        }
        return l;
    }
}

/**
 * 测试代码执行时间
 */
class TestTimeConsumingUtils {
    // 当前时间
    private long thisCurrentTimeMillis = System.currentTimeMillis();

    /**
     * @param desc 输入描述信息
     */
    public void printTime(String desc) {
        long result = System.currentTimeMillis() - thisCurrentTimeMillis;
        System.err.println(desc + "耗时：" + result);
    }
}
```

> 快慢指针：
> - 时间：`O(n)`
> - 空间：`O(1)`
> 
> 双向指针：
> -时间：`O(n)`
> -空间：`O(1)`

## 有序数组的平方

> 给你一个按 **非递减顺序** 排序的整数数组 nums，返回 **每个数字的平方** 组成的新数组，要求也按 **非递减顺序** 排序。
> 
> 示例：
> - 输入：`nums = [-7,-3,2,3,11]`
> - 输出：`[4,9,9,49,121]`
> 
> 说明：
> - `1 <= nums.length <= 10^4`
> - `-10^4 <= nums[i] <= 10^4`
> 
> [977. 有序数组的平方 - 力扣（LeetCode）](https://leetcode.cn/problems/squares-of-a-sorted-array/)

双向指针

有负数，如果都是正数直接平方就好了

一直纠结O(1)的复杂度没做出来

```java
public class Solution {
    public static void main(String[] args) {
        int[] nums = new int[]{-4, -1, 0, 3, 10};
        for (int i : sortedSquares(nums)) {
            System.out.print(i + " ");
        }
    }

    /**
     * 双向指针
     */
    public static int[] sortedSquares(int[] nums) {
        int len = nums.length;
        int[] res = new int[nums.length];
        int i = len - 1; // 结果数组索引
        for (int l = 0, r = len - 1; l <= r; ) {
            if (Math.pow(nums[l], 2) > Math.pow(nums[r], 2)) {
                res[i--] = (int) Math.pow(nums[l++], 2);
            } else {
                res[i--] = (int) Math.pow(nums[r--], 2);
            }
        }
        return res;
    }
}
```

- 时间：`O(n)`
- 空间：`O(n)`

## 搜索旋转排序数组 #中等 #手撕 #rep

[33. 搜索旋转排序数组 - 力扣（LeetCode）](https://leetcode.cn/problems/search-in-rotated-sorted-array/description/)

分析：
- `O(log n)` 的算法，肯定是二分，但是数组不是有序的，不是简单的二分
- mid 的两边总有一边是非递减的，所以判断 target 在哪半边先从非递减的那半边判断

> `nums[mid] == nums[l]` 和 `nums[mid] > nums[l]` 其实可以合并

```java
public int search(int[] nums, int target) {
	int len = nums.length;
	for (int l = 0, r = len - 1; l <= r;) {
		int mid = (l + r) / 2;
		if (nums[mid] == target) {
			return mid;
		}
		if (nums[mid] == nums[l]) { // 只有两个元素
			l = mid + 1;
		} else if (nums[mid] > nums[l]) { // 左半区间非递减
			if (target >= nums[l] && target < nums[mid]) {
				r = mid - 1;
			} else {
				l = mid + 1;
			}
		} else { // 右半区间非递减
			if (target > nums[mid] && target <= nums[r]) {
				l = mid + 1;
			} else {
				r = mid - 1;
			}
		}
	}
	return -1;
}
```


## ---------- 滑动窗口

## 长度最小的子数组

> 给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其**和 大于等于 s 的长度最小的 连续** 子数组，并**返回其长度**。如果不存在符合条件的子数组，返回 0。
> 
> 示例：
> - 输入：`target = 7`, `nums = [2,3,1,2,4,3]`
> - 输出：2
> - 解释：子数组 `[4,3]` 是该条件下的长度最小的子数组。
> 
> 说明：
> - `1 <= target <= 10^9`
> - `1 <= nums.length <= 10^5`
> - `1 <= nums[i] <= 10^5`

滑动窗口（双指针）

实现滑动窗口，主要确定如下三点：
1. 窗口内是什么？
2. 如何移动窗口的起始位置？
3. 如何移动窗口的结束位置？

```java
public class Solution {
    public static void main(String[] args) {
        int[] nums = new int[]{1, 2, 3, 4, 5};
        int tar = 11;
        System.out.println(minSubArrayLen(tar, nums));
    }

    public static int minSubArrayLen(int target, int[] nums) {
        int len = nums.length;
        int res = len + 1;
        int sum = 0;
        for (int l = 0, r = 0; r <= len - 1; r++) {
            sum += nums[r];
            while (sum >= target) {
                res = Math.min(res, r - l + 1);
                sum -= nums[l++];
            }
        }
        return res == len + 1 ? 0 : res;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## 最长不重复子串 #中等 #手撕 #rep

[LCR 016. 无重复字符的最长子串 - 力扣（LeetCode）](https://leetcode.cn/problems/wtcaE1/description/)

分析：维护窗口内的元素不重复

```java
public int lengthOfLongestSubstring(String s) {
	char[] chars = s.toCharArray();
	Set<Character> set = new HashSet<>();
	int res = 0;
	for (int l = 0, r = 0; r < chars.length; r++) {
		while (set.contains(chars[r])) {
			set.remove(chars[l]);
			l++;
		}
		set.add(chars[r]);
		res = Math.max(res, r - l + 1);
	}
	return res;
}
```

## 万万没想到之抓捕孔连顺 #笔试 #rep

[牛客网 - 找工作神器|笔试题库|面试经验|实习招聘内推，求职就业一站解决_牛客网 (nowcoder.com)](https://www.nowcoder.com/exam/test/81211484/detail?pid=16516564&examPageSource=Company&testCallback=https%3A%2F%2Fwww.nowcoder.com%2Fexam%2Fcompany%3FcurrentTab%3Dleatest%26jobId%3D100%26selectStatus%3D0%26tagIds%3D665&testclass=%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91)

我叫王大锤，是一名特工。我刚刚接到任务：在字节跳动大街进行埋伏，抓捕恐怖分子孔连顺。和我一起行动的还有另外两名特工，我提议

1. 我们在字节跳动大街的 N 个建筑中选定 3 个埋伏地点。
2. 为了相互照应，我们决定相距最远的两名特工间的距离不超过 D 。

请听题：给定 N（可选作为埋伏点的建筑物数）、 D（相距最远的两名特工间的距离的最大值）以及可选建筑的坐标，计算在这次行动中，大锤的小队有多少种埋伏选择。

注意：
1. 两个特工不能埋伏在同一地点
2. 三个特工是等价的：即同样的位置组合( A , B , C ) 只算一种埋伏方法，不能因“特工之间互换位置”而重复使用

```
数据范围： $0<n,d≤10^6$

时间限制：C/C++ 1秒，其他语言2秒

空间限制：C/C++ 128M，其他语言256M
```

输入描述：
```
第一行包含空格分隔的两个数字 N和D(1 ≤ N ≤ 1000000; 1 ≤ D ≤ 1000000) 

第二行包含N个建筑物的的位置，每个位置用一个整数（取值区间为`[0, 1000000]`）表示，从小到大排列（将字节跳动大街看做一条数轴）
```

输出描述：
```
一个数字，表示不同埋伏方案的数量。结果可能溢出，**请对 99997867 取模**
```

示例1
```
输入例子：
4 3
1 2 3 4

输出例子：
4

例子说明：
可选方案 (1, 2, 3), (1, 2, 4), (1, 3, 4), (2, 3, 4)   
```

示例2
```
输入例子：
5 19
1 10 20 30 50

输出例子：
1

例子说明：
可选方案 (1, 10, 20)   
```

示例3
```
输入例子：
2 100
1 102

输出例子：
0
```

---

分析：
- 本题等价于有序数组中找到满足条件（最大差值不超过 D）的三个数的组合数
- 大数题，可选方案 res 每次累加要取余，否则会溢出
- 滑动窗口，l 是第一个特工的位置，r 是第三个特工的位置
- 找到满足他们间距离小于等于最大距离 D 的最大窗口，固定 r，res 累加 l 到 r-1 所有位置取两个的组合数 $C_{r-l}^{2}$

1）long 记录结果，每次取余
```java
import java.math.BigInteger;
import java.util.Scanner;
public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int N = scanner.nextInt();
        int D = scanner.nextInt();
        int[] location = new int[N];
        for (int i = 0; i < N; i++) {
            location[i] = scanner.nextInt();
        }

        long res = 0;
        long mods = 99997867;
        for (int l = 0, r = 0; r < N; r++) {
            while (location[r] - location[l] > D) {
                // 最远的两名特工之间超过最大距离
                l++;
            }
            if (r - l >= 2) {
                long temp = r - l; // 要用long，下面的计算int会溢出
                temp = temp * (temp - 1) / 2; // 组合个数
                res = (res + temp) % mods;
            }
        }
        System.out.println(res);
    }
}
```

2）BigInteger 记录结果，最后取余
```java
import java.math.BigInteger;
import java.util.Scanner;
public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int N = scanner.nextInt();
        int D = scanner.nextInt();
        int[] location = new int[N];
        for (int i = 0; i < N; i++) {
            location[i] = scanner.nextInt();
        }

        BigInteger num = BigInteger.ZERO; // 代表数字零的常量
        for (int l = 0, r = 0; r < N; r++) {
            while (location[r] - location[l] >
                    D) { // 最远的两名特工之间超过最大距离
                l++;
            }
            if (r - l >= 2) {
                long temp = r - l; // 要用long，下面的计算int会溢出
                num = num.add(BigInteger.valueOf(temp * (temp - 1) / 2));
            }
        }
        long mods = 99997867;
        System.out.println(num.mod(BigInteger.valueOf(mods)));
    }
}
```

## ---------- 模拟过程

## 螺旋矩阵

> 给定一个正整数 n，生成一个包含 1 到 n^2 所有元素，且元素按**顺时针顺序螺旋排列**的正方形矩阵。
> 
> 示例：
> - 输入：`n = 3`
> - 输出：`[[1,2,3],[8,9,4],[7,6,5]]`
> 
> 说明：
>  - `1 <= n <= 20`
> 
> [59. 螺旋矩阵 II - 力扣（LeetCode）](https://leetcode.cn/problems/spiral-matrix-ii/)

模拟过程，每次循环画完一圈

坚持**循环不变量原则**，每一圈四条边，每条边保持左闭右开

---
By Boer

```java
class Solution {  
    public static void main(String[] args) {  
        generateMatrix(3);  
    }  
  
    public static int[][] generateMatrix(int n) {  
        int[][] res = new int[n][n];  
        // n为单数需要补中心点
        res[(n - 1) / 2][(n - 1) / 2] = n * n;  
        int y = 0; // 起始位置  
        int x = 1; // 1~n^2  
        for (int i = y, j = y; n >= 0; n--, y++, i = y, j = y) {  
            while (j < n - 1) {  
                res[i][j++] = x++;  
            }  
            while (i < n - 1) {  
                res[i++][j] = x++;  
            }  
            while (j > y) {  
                res[i][j--] = x++;  
            }  
            while (i > y) {  
                res[i--][j] = x++;  
            }  
        }  
        return res;  
    }  
}
```

--- 
参考代码

```java
class Solution {
    public int[][] generateMatrix(int n) {
        int loop = 0;  // 控制循环次数
        int[][] res = new int[n][n];
        int start = 0;  // 每次循环的开始点(start, start)
        int count = 1;  // 定义填充数字
        int i, j;

        while (loop++ < n / 2) { // 判断边界后，loop从1开始
            // 模拟上侧从左到右
            for (j = start; j < n - loop; j++) {
                res[start][j] = count++;
            }

            // 模拟右侧从上到下
            for (i = start; i < n - loop; i++) {
                res[i][j] = count++;
            }

            // 模拟下侧从右到左
            for (; j >= loop; j--) {
                res[i][j] = count++;
            }

            // 模拟左侧从下到上
            for (; i >= loop; i--) {
                res[i][j] = count++;
            }
            start++;
        }

        if (n % 2 == 1) {
            res[start][start] = count;
        }

        return res;
    }
}
```

> 本题模拟过程，时空复杂度不必在意 	
> 
> 时间：`O(n^2)`
> 
> 空间：`O(1)`

# 链表

## 移除链表元素

> 给你一个链表的头节点 `head` 和一个整数 `val` ，请你**删除**链表中所有满足 `Node.val == val` 的节点，并返回 **新的头节点** 。
> 
> 示例：
> - 输入：`head = [1,2,6,3,4,5,6], val = 6`
> - 输出：`[1,2,3,4,5]`
>  
> 说明：
> - 列表中的节点数目在范围 `[0, 10^4]` 内 （包含空链表）
> - `1 <= Node.val <= 50`
> - `0 <= val <= 50`
>    
> [203. 移除链表元素 - 力扣（LeetCode）](https://leetcode.cn/problems/remove-linked-list-elements/)

特例：`[val,...,val]` 一个及以上的val组成的链表

**虚拟头结点**
- ==统一==头结点和其他节点的删除操作
- 初始虚拟头结点需要存档一份，用于后续返回（特例情况下原先的头结点指向的是旧链表）

```java
class Solution {  
    public ListNode removeElements(ListNode head, int val) {  
        ListNode dummy = new ListNode(-1, head); // 前置指针  
        ListNode pre = dummy; // 前置指针存档  
        while (pre.next != null) {  
            if (pre.next.val == val) {  
                pre.next = pre.next.next;  
            } else {  
                pre = pre.next;  
            }  
        }  
        return dummy.next;  
    }  
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## 设计链表

[707. 设计链表 - 力扣（LeetCode）](https://leetcode.cn/problems/design-linked-list/)


## 反转链表 #手撕2 #rep2

> 很重要的基础题！其他链表题都可能包含本题的操作

[206. 反转链表 - 力扣（LeetCode）](https://leetcode.cn/problems/reverse-linked-list/description/)

√ 双指针 正向反转

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode pre = null; // 上一次调整的节点
        ListNode cur = head; // 本次调整的节点

        while (cur != null) {
            ListNode temp = cur.next;
            cur.next = pre;
            pre = cur;
            cur = temp;
        }
        return pre;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`  

--- 
递归：正向反转

- 需要两个指针实现
- 先翻转再递归调用

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        return reverse(null, head);
    }

    private ListNode reverse(ListNode prev, ListNode cur) {
	    // 终止条件
        if (cur == null) {
            return prev;
        }
        // 反转操作
        ListNode temp = null;
        temp = cur.next; // 先保存下一个节点
        cur.next = prev; 
        // 递归调用
        return reverse(cur, temp);
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)` n层栈

--- 

递归：反向翻转

- 单指针即可实现
- 先递归调用再翻转

```java
class Solution {
    public ListNode reverseList(ListNode head) {
	    // 终止条件
        if (head == null || head.next == null) return head; 
        // 递归调用，记录尾节点
        ListNode last = reverseList(head.next);
        // 反转操作
        head.next.next = head;
        head.next = null;
        // 每层递归都把尾节点返回
        return last;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)` n层栈

## 两两交换链表中的节点

> 给定一个链表，**两两交换**其中相邻的节点，并返回交换后的链表。
> 你必须在不修改节点内部的值的情况下完成本题（即，**只能进行节点交换**）
> 
> 示例：
> - 输入：`head = [1,2,3,4]`
> - 输出：`[2,1,4,3]`
>  
> 说明：
> - 链表中节点的数目在范围 `[0, 100]` 内
> - `0 <= Node.val <= 100`
>    
> [24. 两两交换链表中的节点 - 力扣（LeetCode）](https://leetcode.cn/problems/swap-nodes-in-pairs/)

递归

```java
class Solution {  
    public ListNode swapPairs(ListNode head) {  
        // 终止条件  
        if (head == null || head.next == null) return head;  
        // 递归调用  
        head.next.next = swapPairs(head.next.next);  
        // 两两交换  
        ListNode next = head.next;  
        head.next = next.next;  
        next.next = head;  
        return next;  
    }  
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)`


## 删除链表的倒数第N个节点 #中等 #手撕

> 给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。
> 
> 示例：
> - 输入：`head = [1,2,3,4,5], n = 2`
> - 输出：`[1,2,3,5]`
>  
> 说明：
> - 链表中结点的数目为 `sz`
> - `1 <= sz <= 30`
> - `0 <= Node.val <= 100`
> - `1 <= n <= sz`
>    
> [19. 删除链表的倒数第 N 个结点 - 力扣（LeetCode）](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

快慢指针

- dummy 节点解决删除的是 head 节点的情况

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(-1,head);
        ListNode s=dummy;
        ListNode f=dummy;
        for(int i=0;i<=n;i++){
            f=f.next;
        }
        while(f!=null){
            f=f.next;
            s=s.next;
        }
        s.next=s.next.next;
        return dummy.next;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## 链表相交

[面试题 02.07. 链表相交 - 力扣（LeetCode）](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/)

双指针

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode curA=headA;
        ListNode curB=headB;
        int lenA=0, lenB=0;
        int gap; //长度差 >=0
        while(curA!=null){
            lenA++;
            curA=curA.next;
        }
        while(curB!=null){
            lenB++;
            curB=curB.next;
        }
        if(lenA>lenB){
            curA=headA;
            curB=headB;
            gap=lenA-lenB;
        }else{
            curA=headB;
            curB=headA;
            gap=lenB-lenA;
        }
        // 长的先动
        while(gap-->0){
            curA=curA.next;
        }
        while(curA!=null){
            if(curA==curB){
                return curA;
            }
            curA=curA.next;
            curB=curB.next;
        }
        return null;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## 环形链表 II #中等 #手撕3 #rep

[142. 环形链表 II - 力扣（LeetCode）](https://leetcode.cn/problems/linked-list-cycle-ii/)

快慢指针。数学问题，记就完事了

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode f=head, s=head;
        while(f!=null && f.next!=null){
            f=f.next.next;
            s=s.next;
            if(f==s){
                s=head;
                while(f!=s){
                    f=f.next;
                    s=s.next;
                }
                return f;
            }
        }
        return null;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## K 个一组翻转链表 #困难 #手撕3 #rep

[25. K 个一组翻转链表 - 力扣（LeetCode）](https://leetcode.cn/problems/reverse-nodes-in-k-group/description/)

模拟过程：

链表分区为已翻转部分+待翻转部分+未翻转部分

![500](assets/Pasted%20image%2020240405122533.png)

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;

        ListNode pre = dummy; // 已翻转子链表的尾节点
        ListNode end = dummy; // 待翻转子链表的尾节点
        ListNode next = dummy; // 未翻转子链表的头节点

        while (true) {
            for (int i = 0; i < k && end != null; i++)
                end = end.next;
            if (end == null)
                break;

            ListNode start = pre.next;
            next = end.next;
            end.next = null; // reverse的终止条件
            pre.next = reverse(start);
            start.next = next; // start到尾部，end到头部了
            pre = start;
            end = start;
        }
        return dummy.next;
    }

    private ListNode reverse(ListNode start) {
        ListNode pre = null;
        ListNode cur = start;
        while (cur != null) {
            ListNode temp = cur.next;
            cur.next = pre;
            pre = cur;
            cur = temp;
        }
        return pre;
    }
}
```

## 重排链表 #中等 #手撕3 #rep 

[143. 重排链表 - 力扣（LeetCode）](https://leetcode.cn/problems/reorder-list/description/)

分析：单向链表，如果每次找第 n、n-1、n-2 个节点，时间复杂度高

1）线性表 #todo 

- 时间复杂度：$O(N)$，其中 N 是链表中的节点数。
- 空间复杂度：$O(N)$，其中 N 是链表中的节点数。主要为线性表的开销。

2）寻找链表中点 + 链表逆序 + 合并链表 ✔

分析：
- 快慢指针找到原链表的中点
- 将原链表的右半端反转
- 归并合并两个链表

> 本题分解开其实都可以作为三道题了

```java
class Solution {
    public void reorderList(ListNode head) {
        if (head == null || head.next == null) {
            return;
        }
        ListNode mid = getMiddle(head);
        ListNode head2 = reverse(mid.next);
        mid.next = null;
        merge(head, head2);
    }

    // 找到链表中点
    public ListNode getMiddle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }

    // 反转链表，返回反转后的头结点
    public ListNode reverse(ListNode head) {
        // 初始化
        ListNode pre = null;
        ListNode cur = head;
        while (cur != null) {
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }

    // 归并合并两个链表
    public void merge(ListNode head, ListNode head2) {
        while (head2 != null) {
            ListNode next = head.next;
            ListNode next2 = head2.next;
            head.next = head2;
            head2.next = next;
            head = next;
            head2 = next2;
        }
    }
}
```


# 哈希表

## 有效的字母异位词

> 给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。
> 
> 注意：若 s 和 t 中**每个字符出现的次数都相同**，则称 s 和  t 互为字母异位词
> 
> 示例：
> - 输入: `s = "anagram", t= "nagaram"`
> - 输出: `true`
>  
> 说明：
> - `1 <= s.length, t.length <= 5 * 10^4`
> - `s` 和 `t` 仅包含小写字母
>    
> [242. 有效的字母异位词 - 力扣（LeetCode）](https://leetcode.cn/problems/valid-anagram/)


```java
class Solution {
    public boolean isAnagram(String s, String t) {
        int[] arr=new int[26];
        for(int i=0;i<s.length();i++){
            arr[s.charAt(i)-'a']++;
        }
        for(int i=0;i<t.length();i++){
            arr[t.charAt(i)-'a']--;
        }
        for(int i:arr){
            if(i!=0) return false;
        }
        return true;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)

## 两个数组的交集

> 给定两个数组 `nums1` 和 `nums2` ，返回 它们的交集 。输出结果中的每个元素一定是 唯一 的。我们可以 **不考虑输出结果的顺序** 。
> 
> 示例：
> - 输入: `nums1 = [4,9,5], nums2 = [9,4,9,8,4]`
> - 输出: `[9,4] or [4,9]`
> 
> 说明：
> - `1 <= nums1.length, nums2.length <= 1000
> - `0 <= nums1[i], nums2[i] <= 1000` （都可能是空数组）
>    
> [349. 两个数组的交集 - 力扣（LeetCode）](https://leetcode.cn/problems/intersection-of-two-arrays/)

--- 
双Set

```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> set = new HashSet<>();
        Set<Integer> res = new HashSet<>();
        for (int i : nums1) {
            set.add(i);
        }
        for (int i : nums2) {
            if (set.contains(i)) res.add(i);
        }
        return res.stream().mapToInt(x -> x).toArray();
    }
}
```

> 时间：`O(m+n)`，其中 m 和 n 分别是两个数组的长度。
> 	- 使用两个集合分别存储两个数组中的元素需要 `O(m+n)` 的时间，
> 	- 遍历较小的集合并判断元素是否在另一个集合中需要 `O(min⁡(m,n))` 的时间，
> 空间：`O(m+n)`，两个集合存储

---
数组哈希

> 力扣改了 题目描述 和 后台测试数据，增添了 数值范围：`0<=nums1[i], nums2[i]<=1000`

使用set 不仅占用空间比数组大，而且速度要比数组慢（hash计算）。在数据量大的情况，差距是很明显的

```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        int[] hash1 = new int[1002]; // 下标 1-1001
        int[] hash2 = new int[1002];
        for (int i : nums1)
            hash1[i]++;
        for (int i : nums2)
            hash2[i]++;
        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < 1002; i++)
            if (hash1[i] > 0 && hash2[i] > 0) res.add(i);
        return res.stream().mapToInt(x -> x).toArray();
    }
}
```

> 时间：`O(m+n)`，其中 m 和 n 分别是两个数组的长度。
> 
> 空间：`O(n)`，短的数组长度

## 快乐数

> 编写一个算法来判断一个数 `n` 是不是快乐数。
> 
> **「快乐数」** 定义为：
> - 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
> - 然后重复这个过程直到这个数变为 1，也可能是 **无限循环** 但始终变不到 1。
> - 如果这个过程 **结果为** 1，那么这个数就是快乐数。
> 
> 如果 `n` 是 _快乐数_ 就返回 `true` ；不是，则返回 `false` 。
> 
> 示例：
> 输入：`n = 19`
> 输出：`true`
>  
> 说明：
> - `1 <= n <= 2^31 - 1`
> 
> [202. 快乐数 - 力扣（LeetCode）](https://leetcode.cn/problems/happy-number/description/)

```java
class Solution {
    public boolean isHappy(int n) {
        int tmp=0;
        Set rec=new HashSet();
        while(n!=1 && !rec.contains(n)){
            rec.add(n);
            // 求出快乐数 tmp
            while(n!=0){
                tmp+=(n%10)*(n%10);
                n=n/10;
            }
            n=tmp;
            tmp=0;
        }
        return n==1;
    }
}
```

> 时间：`O(log n)`
> 
> 空间：`O(log n)

## 两数之和

> 给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** _`target`_  的那 **两个** 整数，并返回它们的数组下标。
> 
> - 你可以**假设每种输入只会对应一个答案**。但是，数组中同一个元素在答案里不能重复出现。
> - 你可以按任意顺序返回答案。
> 
> 示例：
> - 输入：`nums = [2,7,11,15], target = 9`
> - 输出：`[0,1]`
> - 解释：因为 `nums[0] + nums[1] == 9` ，返回 `[0, 1]` 。
>  
> 说明：
> - `2 <= nums.length <= 10^4`
> - `10^9 <= nums[i] <= 10^9`
> - `10^9 <= target <= 10^9`
> 
> 进阶：你可以想出一个**时间复杂度小于 `O(n^2)`** 的算法吗？
> 
> [1. 两数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/two-sum/)

--- 
by Boer（待优化）
```java
class Solution {
   public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> rec = new HashMap();
        int temp;
        for (int i = 0; i < nums.length; i++) {
            rec.put(nums[i], i);
        }
        for (int i = 0; i < nums.length; i++) {
            temp = target - nums[i];
            if (rec.containsKey(temp) && rec.get(temp)!=i)
                return new int[]{i, rec.get(n)};
        }
        return null;
    }
}
```

> 不用先把所有数组的元素放入map中
> map中查找的时候会重复找到自己 `rec.containsKey(temp) && rec.get(temp)!=i`
> 额外判断会浪费时间

--- 
优化后

```java
class Solution {
   public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> rec = new HashMap();
        int temp;
        for (int i = 0; i < nums.length; i++) {
            temp = target - nums[i];
            if (rec.containsKey(temp))
                return new int[]{i, rec.get(temp)};
            rec.put(nums[i],i);
        }
        return null;
    }
}
```

> 时间：`O(n)` ， n 是数组中的元素数量。对于每一个元素 `x`，我们可以 `O(1)` 地寻找 `target - x`
> 
> 空间：`O(n)` 哈希表

## 四数相加

> 给你四个整数数组 `nums1`、`nums2`、`nums3` 和 `nums4` ，数组长度都是 `n` ，请你计算**有多少个元组 `(i, j, k, l)`** 能满足：
> 
> - `0 <= i, j, k, l < n`
> - `nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0`
> 
> 示例：
> - 输入：`nums1 = [1,2], nums2 = [-2,-1], nums3 = [-1,2], nums4 = [0,2]`
> - 输出：2
> - 解释：
> 	- `(0, 0, 0, 1) -> nums1[0] + nums2[0] + nums3[0] + nums4[1] = 1 + (-2) + (-1) + 2 = 0`
> 	- `(1, 1, 0, 0) -> nums1[1] + nums2[1] + nums3[0] + nums4[0] = 2 + (-1) + (-1) + 0 = 0`
>  
> 说明：
> - `n == nums1.length`
> - `n == nums2.length`
> - `n == nums3.length`
> - `n == nums4.length`
> - `1 <= n <= 200`
> - `-228 <= nums1[i], nums2[i], nums3[i], nums4[i] <= 228`
> 
> [454. 四数相加 II - 力扣（LeetCode）](https://leetcode.cn/problems/4sum-ii/)

思路和两数之和是一样的

```java
class Solution {
    public int fourSumCount(int[] nums1, int[] nums2, int[] nums3, int[] nums4) {
        int len=nums1.length;
        int temp=0, res=0;
        Map<Integer, Integer> map=new HashMap();
        for(int i:nums1){
            for(int j:nums2){
                temp=i+j;
                map.put(temp,map.getOrDefault(temp,0)+1);
            }
        }
        for(int i:nums3){
            for(int j:nums4){
                temp=i+j;
                res+=map.getOrDefault(0-temp,0);
            }
        }
        return res;
    }
}
```

> 时间：`O(n^2)`
> 
> 空间：`O(n^2)`，最坏情况下A和B的值各不相同，相加产生的数字个数为 n^2

## 赎金信

> 给你两个字符串：`ransomNote` 和 `magazine` ，判断 **`ransomNote` 能不能由 `magazine` 里面的字符构成。**
> - 如果可以，返回 `true` ；否则返回 `false` 。
> - `magazine` 中的**每个字符只能在 `ransomNote` 中使用一次。**
> 
> 示例：
> - 输入：`ransomNote = "a", magazine = "b"`
> - 输出：false
>  
> 说明：
> - `1 <= ransomNote.length, magazine.length <= 105`
> - `ransomNote` 和 `magazine` 由**小写英文字母**组成
> 
> [383. 赎金信 - 力扣（LeetCode）](https://leetcode.cn/problems/ransom-note/)

小写字母--->数组哈希。相较于map省空间和时间（直接索引，不用计算哈希值）

```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        // Map<Character,Integer> rec=new HashMap();
        int[] rec=new int[26];
        char temp;
        for(char c:magazine.toCharArray()){
            rec[c -'a']+=1;
        }
        for(char c:ransomNote.toCharArray()){
            if(rec[c -'a']==0) return false;
            rec[c -'a']-=1;
        }
        return true;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## 三数之和

> 给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且**不重复**的三元组。
> 
> 示例：
> - 输入：`nums = [-1,0,1,2,-1,-4]`
> - 输出：`[[-1,-1,2],[-1,0,1]]
> - 解释：`
> 	- `nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。`
> 	- `nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。`
> 	- `nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。`
> 	- 不同的三元组是 `[-1,0,1]` 和 `[-1,-1,2]` 。
> 	- 注意，输出的顺序和三元组的顺序并不重要。
>  
> 说明：
> - `3 <= nums.length <= 3000`
> - `-10^5 <= nums[i] <= 10^5`
> 
> [15. 三数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/3sum/)

双指针：将原本暴力O(n^3)的解法，降为O(n^2)的解法

```java
class Solution {
    public static void main(String[] args) {
        int[] nums = new int[]{-1, 0, 1, 2, -1, -4};
        int[] nums1 = new int[]{-2, 0, 0, 2, 2};
        int[] nums2 = new int[]{-2, 0, 3, -1, 4, 0, 3, 4, 1, 1, 1, -3, -5, 4, 0};
        System.out.println(threeSum(nums2));
    }

    public static List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(nums);
        int len = nums.length;
        int l, r;
        for (int i = 0; i < len - 2; i++) {
            // 剪枝：第一个元素已经大于0，没有结果
            if (nums[i] > 0) return res;
            // 元组第一个元素去重
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            l = i + 1;
            r = len - 1;
            while (l < r) {
                if (nums[l] + nums[r] + nums[i] == 0) {
                    List<Integer> rec = new ArrayList<>();
                    rec.add(nums[i]);
                    rec.add(nums[l]);
                    rec.add(nums[r]);
                    res.add(rec);
                    // 元组第二个元素去重
                    while (++l < r && nums[l] == nums[l - 1]) ;
                    // 元组第三个元素去重
                    while (--r > l && nums[r] == nums[r + 1]) ;
                } else if (nums[l] + nums[r] + nums[i] <= 0) {
                    l++;
                } else {
                    r--;
                }
            }
        }
        return res;
    }
}
```

> 时间：`O(n^2)`
> 
> 空间：`O(1)`

## 四数之和

> 给你一个由 `n` 个整数组成的数组 `nums` ，和**一个目标值 `target`** 。请你找出并返回满足下述全部条件且**不重复**的四元组 `[nums[a], nums[b], nums[c], nums[d]]` （若两个四元组元素一一对应，则认为两个四元组重复）：
> - `0 <= a, b, c, d < n`
> - `a`、`b`、`c` 和 `d` **互不相同**（两个元组之间）
> - `nums[a] + nums[b] + nums[c] + nums[d] == target`
>
> 你可以按 **任意顺序** 返回答案 。
> 
> 示例：
> - 输入：`nums = [1,0,-1,0,-2,2], target = 0`
> - 输出：`[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]`
>  
> 说明：
> - `1 <= nums.length <= 200`
> - `-10^9 <= nums[i] <= 10^9`
> - `-10^9 <= target <= 10^9`
> 
> [18. 四数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/4sum/)

双指针：将原本暴力O(n^4)的解法，降为O(n^3)的解法

```java
class Solution {
    public static void main(String[] args) {
        int[] nums = new int[]{1, 0, -1, 0, -2, 2};
        int target = 0;
        int[] nums1 = new int[]{2, 2, 2, 2, 2};
        int target1 = 8;
        int[] nums2 = new int[]{0, 0, 0, 0};
        int target2 = 0;
        System.out.println(fourSum(nums2, target2));
    }

    public static List<List<Integer>> fourSum(int[] nums, int target) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(nums);
        int len = nums.length;
        int l, r;
        for (int i = 0; i < len - 3; i++) {
            // 剪枝
            if (nums[i] > target && nums[i] >= 0) return res;
            // 元组第一个元素去重
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            for (int j = i + 1; j < len - 2; j++) {
                // 元组第二个元素去重
                if (j > i + 1 && nums[j] == nums[j - 1]) continue;
                l = j + 1;
                r = len - 1;
                while (l < r) {
                    int sum = nums[i] + nums[j] + nums[l] + nums[r];
                    if (sum == target) {
                        res.add(Arrays.asList(nums[i], nums[j], nums[l], nums[r]));
                        // 元组第二个元素去重
                        while (++l < r && nums[l] == nums[l - 1]) ;
                        // 元组第三个元素去重
                        while (--r > l && nums[r] == nums[r + 1]) ;
                    } else if (sum <= target) {
                        l++;
                    } else {
                        r--;
                    }
                }
            }
        }
        return res;
    }
}
```

> 时间：`O(n^3)`
> 
> 空间：`O(1)`

# 字符串

## 反转字符串

> 编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 `s` 的形式给出。
> 
> 不要给另外的数组分配额外的空间，你必须**原地**修改输入数组、使用 O(1) 的额外空间解决这一问题。
> 
> 示例：
> - 输入：`s = ["h","e","l","l","o"]`
> - 输出：`["o","l","l","e","h"]`
>  
> 说明：
> - `1 <= s.length <= 105`
> - `s[i]` 都是 [ASCII](https://baike.baidu.com/item/ASCII) 码表中的可打印字符
> 
> [344. 反转字符串 - 力扣（LeetCode）](https://leetcode.cn/problems/reverse-string/)

双指针

```java
class Solution {
    public void reverseString(char[] s) {
        int len=s.length;
        int l=0;
        int r=len-1;
        char temp;
        while(l<r){
            temp=s[l];
            s[l++]=s[r];
            s[r--]=temp;
        }
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## 反转字符串 II

>  给定一个字符串 `s` 和一个整数 `k`，从字符串开头算起，**每计数至 `2k` 个字符，就反转这 `2k` 字符中的前 `k` 个字符**。
>  - 如果剩余字符少于 `k` 个，则将剩余字符全部反转。
> - 如果剩余字符小于 `2k` 但大于或等于 `k` 个，则反转前 `k` 个字符，其余字符保持原样。
> 
> 示例：
> - 输入：`s = "abcdefg", k = 2`
> - 输出：`"bacdfeg"`
>  
> 说明：
> - `1 <= s.length <= 10^4`
> - `s` 仅由小写英文组成
> - `1 <= k <= 10^4`
> 
> [541. 反转字符串 II - 力扣（LeetCode）](https://leetcode.cn/problems/reverse-string-ii/)

双指针

```java
class Solution {
    public String reverseStr(String s, int k) {
        char[] chs = s.toCharArray();
        int len = s.length();
        int l, r;
        char temp;
        for (int i = 0; i < len; i += 2 * k) {
            l = i;
            r = Math.min(l + k - 1, len - 1);
            while (l < r) {
                temp = chs[l];
                chs[l++] = chs[r];
                chs[r--] = temp;
            }
        }
        return new String(chs);
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## 反转字符串中的单词

> 给你一个字符串 `s` ，请你反转字符串中 **单词** 的顺序。
> 
> 单词 是由非空格字符组成的字符串。`s` 中使用**至少一个空格**将字符串中的 单词 分隔开。
> 
> **返回**：单词 **顺序颠倒**且 单词 之间用**单个空格连接**的结果字符串。
> 
> 注意：输入字符串 `s`中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。
> 
> 示例：
> - 输入：s = "`  the sky is blue`"
> - 输出："`blue is sky the`"
>  
> 说明：
> - `1 <= s.length <= 104`
> - `s` 包含英文大小写字母、数字和空格 `' '`
> - `s` 中 **至少存在一个** 单词
>   
>   进阶：如果字符串在你使用的编程语言中是一种可变数据类型，请尝试使用 `O(1)` 额外空间复杂度的 **原地 解法**。
> 
> [151. 反转字符串中的单词 - 力扣（LeetCode）](https://leetcode.cn/problems/reverse-words-in-a-string/)

**可变字符串stringbuilder**

Java中String的底层是字符数组，多余的空格是无法原地去除的，必须要另起一个数组。

stringbuilder底层也是字符数组，但会自动控制容量

```java
class Solution {
    StringBuilder sb;
    int l, r;
    char temp;

    public String reverseWords(String s) {
        // System.out.println(s);
        sb = new StringBuilder();
        l = 0;
        r = s.length() - 1;
        // ----- 消除多余空格
        // 消除头尾
        while (s.charAt(l) == ' ') l++;
        while (s.charAt(r) == ' ') r--;
        int k = 0;
        while (l <= r) {
            temp = s.charAt(l);
            // 连续空格只保留一个
            if (temp != ' ' || sb.charAt(sb.length() - 1) != ' ') {
                sb.append(temp);
            }
            l++;
        }
        // ----- 整体换
        int len = sb.length();
        l = 0;
        r = len - 1;
        reverse(l, r);
        // ----- 局部换
        for (int i = 0; i <= len - 1; i++) {
            if (sb.charAt(i) == ' ') continue;
            l = r = i;
            while (++r <= len - 1 && sb.charAt(r) != ' ') ;
            i = r--;
            reverse(l, r);
        }
        return new String(sb);
    }

    public void reverse(int l, int r) {
        while (l < r) {
            temp = sb.charAt(l);
            sb.setCharAt(l++, sb.charAt(r));
            sb.setCharAt(r--, temp);
        }
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)`

---
数组

![](assets/Pasted%20image%2020240110192718.png)

## 找出字符串中第一个匹配项的下标(KMP) #rep

> 实现C语言的 strstr() 以及 Java的 indexOf() 

> 给你两个字符串 `haystack` 和 `needle` ，请你在 `haystack` 字符串中找出 `needle` 字符串的**第一个匹配项的下标**（下标从 0 开始）。如果 `needle` 不是 `haystack` 的一部分，则返回  `-1` 。
> 
> 示例：
> - 输入：`haystack = "sadbutsad", needle = "sad"`
> - 输出：0
> - 解释：`"sad"` 在下标 0 和 6 处匹配。
> - 第一个匹配项的下标是 0 ，所以返回 0 。
>  
> 说明：
> - `1 <= haystack.length, needle.length <= 104`
> - `haystack` 和 `needle` 仅由小写英文字符组成
> 
> [28. 找出字符串中第一个匹配项的下标 - 力扣（LeetCode）](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)
> 
>  [代码随想录 (programmercarl.com)](https://programmercarl.com/0028.%E5%AE%9E%E7%8E%B0strStr.html#%E6%80%9D%E8%B7%AF)

前缀表
- 作用：前缀表是用来回退的，它记录了模式串与主串(文本串)不匹配的时候，**模式串应该从哪里开始重新匹配**
- 约等于 next 数组
	- next数组既可以就是前缀表，
	- 也可以是前缀表统一减一（右移一位，初始位置为-1）
- 记录下标 i 之前（包括 i ）的字符串中，最长相等前后缀

最长**相等**前后缀：
- 前缀：不包含最后一个字符的所有以第一个字符开头的连续子串
- 后缀：不包含第一个字符的所有以最后一个字符结尾的连续子串。

![](assets/Pasted%20image%2020240102232241.png)
找到的不匹配的位置 f， 那么此时我们要看它的前一个字符的前缀表的数值是多少，因为要找前面字符串的最长相同的前缀和后缀，前缀和后缀既然相等就不用匹配了。

```java
class Solution {  
    /**  
     * 获取next数组  
     * - 双指针 (i,j)，i 永不回退  
     * - j指向前缀末尾位置，i指向后缀末尾位置  
     * - next[j]就是j位置的最长相等前后缀长度  
     *  
     * @param next 前缀表  
     * @param s    模式串  
     */  
    void getNext(int[] next, char[] s) {  
        // next[0] = 0;  
        for (int i = 1, j = 0; i <= s.length - 1; i++) {  
            // 不相等就回退。1）回退到next[j]的前一个位置 2）回退到j=0就从头开始匹配了，不用退了
            while (j > 0 && s[i] != s[j]) {  
                // j之前的子串已经匹配了，那么[0~j-1]区间的最长相等前后缀就不用再匹配了  
                j = next[j - 1];  
            }  
            if (s[i] == s[j]) {  
                // j+1就是当前的最长相等前后缀长度（j是索引，长度要+1）  
                next[i] = ++j;  
                continue;  
            }  
            // j已经回退到指向第一个元素，并且chs[i]!=chs[j]  
            next[i] = 0;  
        }  
    }  
  
    public int strStr(String pp, String ss) {  
        char[] p = pp.toCharArray();  
        char[] s = ss.toCharArray();  
        int[] next = new int[ss.length()];  
        getNext(next, s);  
        // i永不回退  
        for (int i = 0, j = 0; i < p.length; i++) {  
            while (j > 0 && p[i] != s[j]) {  
                // j之前的子串已经匹配了，那么[0~j-1]区间的最长相等前后缀就不用再匹配了  
                j = next[j - 1];  
            }  
            if (p[i] == s[j]) {  
                j++;  
            }  
            if (j == s.length) return i - j + 1;  
        }  
        return -1;  
    }  
  
    public static void main(String[] args) {  
        Solution run = new Solution();  
        int res = run.strStr("aabaabab", "abab");  
        System.out.println(res);  
    }  
}
```

> 时间：`O(m+n)` ，`O(m)`求ss， `O(n)`匹配（最多遍历一次pp）
> 
> 空间：`O(m)` next数组

## 重复的子字符串 #rep

[459. 重复的子字符串 - 力扣（LeetCode）](https://leetcode.cn/problems/repeated-substring-pattern/)

**移位匹配**

多次 “移位” 字符串，并使其与原始字符串匹配。例如：`abcabc`  移位一次：`cabcab`   移位两次：`bcabca`   移位三次：`abcabc`。

基于这个思想，可以每次移动k个字符，直到匹配移动 `length - 1` 次，但如果 s 很长的话，效率很低。

可以让 str=s+s=abcabcabcabc，str中包含了所有abcabc移动的可能，用一个大小为`s.length()`的滑动窗口遍历：bcabca，cabcab，abcabc。
- 注意窗口的位置，第一个和最后一个窗口要去掉。

```java
class Solution {
    public boolean repeatedSubstringPattern(String s) {
        String str = s + s;
        return str.substring(1, str.length() - 1).contains(s);
    }
}
```

> 使用了语言自带的字符串查找函数，因此这里不深入分析其时空复杂度

--- 

KMP 最小重复子串（没研究过推导）

```java
class Solution {
     public boolean repeatedSubstringPattern(String s) {
        char[] chs = s.toCharArray();
        int len = chs.length;
        int[] next = new int[len];
        for (int i = 1, j = 0; i < len; i++) {
            while (j > 0 && chs[i] != chs[j]) {
                j = next[j - 1];
            }
            if (chs[i] == chs[j]) {
                j++;
            }
            next[i] = j;
        }
        // 最小重复子串判断
        return next[len - 1] > 0 && len % (len - next[len - 1]) == 0;
    }
}
```

- 时间：`O(n)`
- 空间：`O(n)` next 数组开销

## 万万没想到之聪明的编辑 #笔试 #rep

我叫王大锤，是一家出版社的编辑。我负责校对投稿来的英文稿件，这份工作非常烦人，因为每天都要去修正无数的拼写错误。但是，优秀的人总能在平凡的工作中发现真理。我发现一个发现拼写错误的捷径：

1. 三个同样的字母连在一起，一定是拼写错误，去掉一个的就好啦：比如 helllo -> hello
2. 两对一样的字母（AABB 型）连在一起，一定是拼写错误，去掉第二对的一个字母就好啦：比如 helloo -> hello
3. 上面的规则优先“从左到右”匹配，即如果是 AABBCC，虽然 AABB 和 BBCC 都是错误拼写，应该优先考虑修复 AABB，结果为 AABCC

请听题：请实现大锤的自动校对程序

数据范围： $1≤𝑛≤50$，每个用例的字符串长度满足 $1≤l≤1000$

```
时间限制：C/C++ 1秒，其他语言2秒
空间限制：C/C++ 32M，其他语言64M
```

输入描述：
```
第一行包括一个数字N，表示本次用例包括多少个待校验的字符串。

后面跟随N行，每行为一个待校验的字符串。
```

输出描述：
```
N行，每行包括一个被修复后的字符串。
```

示例1
```
输入例子：
2
helloo
wooooooow

输出例子：
hello
woow
```

示例2
```
输入例子：
1
nowcoder

输出例子：
nowcoder
```

---

分析：
- 涉及字符串的移除操作，StringBuilder
- 每个字符串先按规则 1 剔除，再按规则 2 剔除

```java
import java.util.ArrayList;
import java.util.Scanner;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        ArrayList<StringBuilder> list = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            list.add(new StringBuilder(in.next()));
        }
        for (int i = 0; i < n; i++) {
            StringBuilder sb = list.get(i);
            for (int j = 0; j + 2 < sb.length();) {
                if (sb.charAt(j) == sb.charAt(j + 1) && sb.charAt(j + 1) == sb.charAt(j + 2)) {
                    sb.deleteCharAt(j);
                } else {
                    j++;
                }
            }
            for (int j = 0; j + 3 < sb.length();) {
                if (sb.charAt(j) == sb.charAt(j + 1) && sb.charAt(j + 2) == sb.charAt(j + 3)) {
                    sb.deleteCharAt(j + 3);
                } else {
                    j++;
                }
            }
        }
        for (int i = 0; i < n; i++) {
            System.out.println(list.get(i));
        }
    }
}
```

# 栈与队列

栈和队列相关的实现类：
![](assets/Pasted%20image%2020240406171506.png)

Java 中的 LinkedList 是采用双向循环列表实现的，利用 LinkedList 可以实现栈（stack）、队列（queue）、双向队列（double-ended queue）

统一使用带 First/Last 后缀的方法
- addFirst/Last
- removeFirst/Last
- peekFirst/Last
- offerFirst/Last
- pollFirst/Last
- isEmpty
- `get(int index)`

```ad-tip
`add()` 和 `offer()` 的区别在于，有容量限制的时候，add 会抛出异常，offer 可以根据返回值判断是否添加元素成功。remove 和 poll 同理

大多数情况用 Deque 类型变量接收即可，某些情况下使用 LinkedList 类型变量接收（例如根据索引获取元素，需要用到从List接口继承的方法）
```


## 用栈实现队列 #手撕

[232. 用栈实现队列 - 力扣（LeetCode）](https://leetcode.cn/problems/implement-queue-using-stacks/)

假设所有操作都是有效的！ （例如，一个空的队列不会调用 `pop` 或者 `peek` 操作）

```java
public class MyQueue {
    private Stack<Integer> inStack = new Stack<>();
    private Stack<Integer> outStack = new Stack<>();

    public MyQueue() {}

    public void push(int x) {
        inStack.push(x);
    }

    public int pop() {
        if (outStack.isEmpty()) {
            while (!inStack.isEmpty()) {
                outStack.push(inStack.pop());
            }
        }
        return outStack.pop();
    }

    public int peek() {
        if (outStack.isEmpty()) {
            while (!inStack.isEmpty()) {
                outStack.push(inStack.pop());
            }
        }
        return outStack.peek();
    }

    public boolean empty() {
        return inStack.isEmpty() && outStack.empty();
    }
}
```

## 用队列实现栈 #rep

[225. 用队列实现栈 - 力扣（LeetCode）](https://leetcode.cn/problems/implement-stack-using-queues/)

思想：每进来一个元素，就把之前进来都出队，再入队到自己后面

```java
class MyStack {
    Queue<Integer> q;

    public MyStack() {
        q = new LinkedList<>(); // 双向链表
    }

    public void push(int x) {
        q.offer(x);
        int size = q.size();
        // 一个元素以上
        while (--size > 0)
            q.offer(q.poll());
    }

    public int pop() {
        return q.poll();
    }

    public int top() {
        return q.peek();
    }

    public boolean empty() {
        return q.isEmpty();
    }
}
```

## 反转字符串中的单词

> 给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。
> 
> 有效字符串需满足：
> 
> 1. 左括号必须用相同类型的右括号闭合。
> 2. 左括号必须以正确的顺序闭合。
> 3. 每个右括号都有一个对应的相同类型的左括号。
> 
> 示例：
> - 输入：`s = "()[]{}"`
> - 输出：true
>  
> 说明：
> - `1 <= s.length <= 104`
> - `s` 仅由括号 `'()[]{}'` 组成
> 
> [20. 有效的括号 - 力扣（LeetCode）](https://leetcode.cn/problems/valid-parentheses/)

注意栈空（遍历时、遍历完）的操作

```java
class Solution {  
    public boolean isValid(String s) {  
        Stack<Character> stack = new Stack<>();  
        for (int i = 0; i < s.length(); i++) {  
            if (s.charAt(i) == '(' || s.charAt(i) == '{' || s.charAt(i) == '[') {  
                stack.push(s.charAt(i));  
            }  
            if (stack.isEmpty()) return false;  
            if (s.charAt(i) == ')' && stack.pop() != '(') {  
                return false;  
            }  
            if (s.charAt(i) == '}' && stack.pop() != '{') {  
                return false;  
            }  
            if (s.charAt(i) == ']' && stack.pop() != '[') {  
                return false;  
            }  
        }  
        return stack.isEmpty();  
    }  
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)` 维护栈

## 删除字符串中的所有相邻重复项

> 给出由小写字母组成的字符串 `S`，重复项删除操作会选择**两个相邻且相同**的字母，并删除它们。
> 
> 在 S 上反复执行重复项删除操作，直到无法继续删除。
> 
> 在完成所有重复项删除操作后返回最终的字符串。答案保证唯一。
> 
> 示例：
> - 输入：`"abbaca"`
> - 输出：`"ca"`
> - 解释：
> 	- 例如，在 "abbaca" 中，我们可以删除 "bb" 由于两字母相邻且相同，这是此时唯一可以执行删除操作的重复项。之后我们得到字符串 "aaca"，其中又只有 "aa" 可以执行重复项删除操作，所以最后的字符串为 "ca"。
>  
> 说明：
> 1. `1 <= S.length <= 20000`
> 2. `S` 仅由小写英文字母组成。
> 
> [1047. 删除字符串中的所有相邻重复项 - 力扣（LeetCode）](https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string/)

双端队列实现栈。（可以直接用StringBuilder作为栈）

```java
class Solution {  
    public String removeDuplicates(String s) {  
        Deque<Character> dq = new LinkedList<>();  
        dq.push(s.charAt(0));  
        for (int i = 1; i < s.length(); i++) {  
            if (dq.isEmpty()) {  
                dq.push(s.charAt(i));  
                continue;  
            }  
            if (dq.peek() != s.charAt(i)) {  
                dq.push(s.charAt(i));  
            } else {  
                dq.pop();  
            }  
        }  
        StringBuilder sb = new StringBuilder();  
        while (!dq.isEmpty()) sb.append(dq.removeLast());  
        return sb.toString();  
    }  
  
    public static void main(String[] args) {  
        Solution run = new Solution();  
        System.out.println(run.removeDuplicates("avvacgb"));  
    }  
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)` 维护栈

---
**快慢指针！(1)**

```java
class Solution {  
    public String removeDuplicates(String str) {  
        char[] chs = str.toCharArray();  
        int s = 0, f = 0; // s维护删除后的s  
        while (f < chs.length) {  
            chs[s] = chs[f];  
            if (s > 0 && chs[s] == chs[s - 1]) {  
                s--;  
            } else {  
                s++;  
            }  
            f++;  
        }  
        return new String(chs, 0, s);  
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(1)`

## 逆波兰表达式求值

> 给你一个字符串数组 `tokens` ，表示一个根据 [逆波兰表示法](https://baike.baidu.com/item/%E9%80%86%E6%B3%A2%E5%85%B0%E5%BC%8F/128437) 表示的算术表达式。
> 
> 请你计算该表达式。返回一个表示表达式值的整数。
> 
> **注意：**
> - 有效的算符为 `'+'`、`'-'`、`'*'` 和 `'/'` 。
> - 每个操作数（运算对象）都可以是一个整数或者另一个表达式。
> - 两个整数之间的除法总是 **向零截断** 。
> - 表达式中不含除零运算。
> - 输入是一个根据逆波兰表示法表示的算术表达式。
> - 答案及所有中间计算结果可以用 **32 位** 整数表示。
>  
> **示例 ：**
> 输入：tokens = `["2","1","+","3","*"]`
> 输出：6
> 解释：该算式转化为常见的中缀算术表达式为：`((2 + 1) * 3) = 9`
>  
> **说明：**
> - `1 <= tokens.length <= 10^4`
> - `tokens[i]` 是一个算符（`"+"`、`"-"`、`"*"` 或 `"/"`），或是在范围 `[-200, 200]` 内的一个整数
> 
> [150. 逆波兰表达式求值 - 力扣（LeetCode）](https://leetcode.cn/problems/evaluate-reverse-polish-notation/description/)

```java
public int evalRPN(String[] tokens) {  
    Stack<Integer> stack = new Stack<>();  
    for (String token : tokens) {  
        if (token.equals("+")) {  
            stack.push(stack.pop() + stack.pop());  
        } else if (token.equals("-")) {  
            stack.push(-stack.pop() + stack.pop());  
        } else if (token.equals("*")) {  
            stack.push(stack.pop() * stack.pop());  
        } else if (token.equals("/")) {  
            int temp1 = stack.pop();  
            int temp2 = stack.pop();  
            stack.push(temp2 / temp1);  
        } else {  
            stack.push(Integer.parseInt(token));  
        }  
    }  
    return stack.pop();  
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)` 维护栈

## 滑动窗口最大值 #困难 #笔试 #rep

[239. 滑动窗口最大值 - 力扣（LeetCode）](https://leetcode.cn/problems/sliding-window-maximum/description/)

分析：
- 窗口是滑动的，需要将窗口的元素维护成有序的，且不能排序

1）单调队列
- 维护一个队头到队尾单调递减的队列
- 因为要获取队头和队尾的元素，使用双端队列
```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int[] res = new int[nums.length - k + 1];
        int idx = 0;
        Deque<Integer> deque = new LinkedList<>();
        for (int l = 1 - k, r = 0; r < nums.length; l++, r++) {
            // 队尾出队
            if (l > 0 && deque.peekLast() == nums[l - 1]) {
                deque.removeLast();
            }
            // 保持队列单调，如果队头元素小于当前元素，则队头元素出队
            while (!deque.isEmpty() && deque.peekFirst() < nums[r]) {
                deque.removeFirst();
            }
            deque.addFirst(nums[r]);
            // 获取结果
            if (l >= 0) {
                res[idx++] = deque.peekLast();
            }
        }
        return res;
    }
}
```

- 时间复杂度：`O(n)`
- 空间复杂度：`O(k)` 

2）双指针
- 上一种方式代码难写一点，因为出队的情况需要额外判断，可以直接在原数组上维护单调
```java
public int[] maxSlidingWindow(int[] nums, int k) {
	int[] res = new int[nums.length - k + 1];
	int idx = 0;
	for (int l = 1 - k, r = 0; r < nums.length; l++, r++) {
		// 保持单调递减窗口
		if (r > 0) {
			for (int s = r - 1; s >= 0 && s >= l && nums[s] < nums[s + 1]; s--) {
				nums[s] = nums[s + 1];
			}
		}
		// 获取结果
		if (l >= 0) {
			res[idx++] = nums[l];
		}
	}
	return res;
}
```


## 补：前K个高频元素 #todo

[347. 前 K 个高频元素 - 力扣（LeetCode）](https://leetcode.cn/problems/top-k-frequent-elements/description/)

## 验证二叉树的前序序列化 #中等 #手撕 #rep

[331. 验证二叉树的前序序列化 - 力扣（LeetCode）](https://leetcode.cn/problems/verify-preorder-serialization-of-a-binary-tree/description/)

思路：

前序遍历」是按照「根节点-左子树-右子树」的顺序遍历的，只有当根节点的所有左子树遍历完成之后，才会遍历右子树。对于本题的输入，我们可以先判断「左子树」是否有效的，然后再判断「右子树」是否有效的，最后判断「根节点-左子树-右子树」是否为有效的

**把有效的叶子节点使用 `"#"` 代替。** 比如把 `4##` 替换成 `#`，最后栈中只剩一个 `#` 说明这个前序序列的输入格式是有效的

```java
class Solution {
    public boolean isValidSerialization(String preorder) {
        LinkedList<String> stack = new LinkedList<>();
        for (String s : preorder.split(",")) {
            stack.addFirst(s);
            while (stack.size() >= 3
                    && stack.get(0).equals("#")
                    && stack.get(1).equals("#")
                    && !stack.get(2).equals("#")) {
                stack.removeFirst();
                stack.removeFirst();
                stack.removeFirst();
                stack.addFirst("#");
            }
        }
        return stack.size() == 1 && stack.peekFirst().equals("#");
    }
}
```


# 树

递归模板!!！

```java
T traversal(TreeNode root, ...){
	// 出口
	if() return ;

	// 递归调用
	T left = traversal(root.left, ...);
	T right = traversal(root.right, ...);
	
	// 根据left、right，返回/出口
	return ;
}
```

## ---------- 层序遍历

## 二叉树的层序遍历 #rep

[102. 二叉树的层序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-level-order-traversal/description/)

广度优先搜索

```java
class Solution {
     public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) return res;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            List<Integer> item = new ArrayList<>(); // 记录每一层的元素
            int levelSize = queue.size(); // 当层节点个数
            while (len > 0) {
                TreeNode node = queue.poll();
                item.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
                levelSize--;
            }
            res.add(item);
        }
        return res;
    }
}
```

时空复杂度分析：
- 时间：`O(n)`
- 空间：`O(n)` 队列

#todo 参考 [删除二叉搜索树中的节点 (1)](#删除二叉搜索树中的节点%20(1))，用 list 解决

## 二叉树的层序遍历II

> 给你二叉树的根节点 `root` ，返回其节点值 **自底向上的层序遍历** 。（即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）
> 
> [107. 二叉树的层序遍历 II - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/description/)

思路：二叉树的层序遍历的基础上反转结果

```java
class Solution {
     public List<List<Integer>> levelOrderBottom(TreeNode root) {
        List<List<Integer>> list = new ArrayList<>();
        if (root == null) return list;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            List<Integer> item = new ArrayList<>(); // 记录每一层的元素
            int len = queue.size();
            while (len > 0) {
                TreeNode node = queue.poll();
                item.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
                len--;
            }
            list.add(item);
        }

        List<List<Integer>> res = new ArrayList<>();
        for (int i = list.size() - 1; i >= 0; i--) {
            res.add(list.get(i));
        }
        return res;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)` 队列

## 二叉树的右视图 #中等 #手撕 #rep

> 给定一个二叉树的 **根节点** `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。
> 
> 示例：
> - 输入：`[1,2,3,null,5,null,4]`
> - 输出：`[1,3,4]`
>  
> 说明：
> - 二叉树的节点个数的范围是 `[0,100]`
> - `100 <= Node.val <= 100
> 
> [107. 二叉树的层序遍历 II - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/description/)

```java
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) return res;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            int len = queue.size();
            while (len > 0) {
                TreeNode node = queue.poll();
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
                if (--len == 0) res.add(node.val);
            }
        }
        return res;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(n)` 队列

## ---------- 递归遍历

## 二叉树的递归遍历

> 二叉树的前序遍历
>  
> 输入：`root = [1,null,2,3]`
> 输出：`[1,2,3]`
> 
> [144. 二叉树的前序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-preorder-traversal/)

```java
class Solution {
    List<Integer> res=new ArrayList<>();

    public List<Integer> preorderTraversal(TreeNode root) {
        // 终止条件
        if(root == null) return res;
        // 本层操作
        res.add(root.val);
        // 递归调用
        preorderTraversal(root.left);
        preorderTraversal(root.right);
        return res;
    }
}
```

--- 
> 二叉树的后序遍历
>  
> 输入：`root = [1,null,2,3]`
> 输出：`[3,2,1]`
> 
> [145. 二叉树的后序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-postorder-traversal/description/)

```java
class Solution {
    List<Integer> res=new ArrayList<>();

    public List<Integer> postorderTraversal(TreeNode root) {
        // 终止条件
        if(root == null) return res;
        // 递归调用
        postorderTraversal(root.left);
        postorderTraversal(root.right);
        // 本层操作
        res.add(root.val);
        return res;
    }
}
```

---
> 二叉树的中序遍历
>  
> 输入：`root = [1,null,2,3]`
> 输出：`[1,3,2]`
> 
> [94. 二叉树的中序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-inorder-traversal/description/)

```java
class Solution {
    List<Integer> res=new ArrayList<>();

    public List<Integer> inorderTraversal(TreeNode root) {
        // 终止条件
        if(root == null) return res;
        // 递归调用
        inorderTraversal(root.left);
         // 本层操作
        res.add(root.val);
        inorderTraversal(root.right);
        return res;
    }
}
```

> 递归的时间复杂度
> - 时间：`O(n)`
>-  空间：`O(log_2 n)`  递归调用栈的深度（树的深度）

<br>

## 二叉树的迭代遍历 #rep

**思想：模拟递归，递归调用就是入栈**

--- 
> 二叉树的前序遍历
>  
> 输入：`root = [1,null,2,3]`
> 输出：`[1,2,3]`
> 
> [144. 二叉树的前序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-preorder-traversal/)

每次循环：
- `cur!=null` 
	- 访问cur
	- `cur.right`入栈
	- `cur=cur.left`
- `cur==null`
	- 弹栈，`cur=stack.pop()`

```java
class Solution {
     public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if(root == null) return res;
        Stack<TreeNode> stack = new Stack();
        TreeNode cur=root;
        while (!stack.isEmpty() || cur!=null) {
            if(cur!=null){
                res.add(cur.val);
                stack.push(cur.right);
                cur=cur.left;
            }else{
                cur=stack.pop();
            }
        }
        return res;
    }
}
```

---
> 二叉树的中序遍历
>  
> 输入：`root = [1,null,2,3]`
> 输出：`[1,3,2]`
> 
> [94. 二叉树的中序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-inorder-traversal/description/)

每次循环：
- `cur!=null`
	- `cur`入栈
	- `cur=cur.left`
- `cur==null`
	- 弹栈，`cur=stack.pop()` 
	- 访问cur
	- `cur.right` 入栈

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res=new ArrayList<>();
        if(root==null) return res;
        Stack<TreeNode> stack=new Stack<>();
        TreeNode cur=root;
        while(!stack.isEmpty() || cur!=null){
            if(cur!=null){
                stack.push(cur);
                cur=cur.left;
            }else{
                cur=stack.pop();
                res.add(cur.val);
                cur=cur.right;
            }
        }
        return res;
    }
}
```

--- 
> 二叉树的后序遍历
>  
> 输入：`root = [1,null,2,3]`
> 输出：`[3,2,1]`
> 
> [145. 二叉树的后序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-postorder-traversal/description/)

每次循环：
- `cur!=null`
	- `cur`入栈
	- `cur=cur.left`
- `cur==null`
	- 弹栈，`cur=stack.pop()`
	- **cur已经入过栈** 或者 `cur.right==null`
		- 访问cur
		- 以cur为节点的子树就遍历完了，continue
	- **cur没有入过栈**
		- cur重新入栈，**标识重新入栈**（避免进入`cur!=null`的死循环）
		- `cur=cur.right`

标识方法：
- `Set<TreeNode> rec = new HashSet()`。不能是`Set<Integer>`，因为val会重复
- prev指向前一个节点，如果prev是右节点，说明cur已经入过栈了

Set标识。By Boer (genius).
```java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res=new ArrayList<>();
        if(root==null) return res;
        Set<TreeNode> rec=new HashSet(); // 标识当前节点是否已经入过stack
        Stack<TreeNode> stack=new Stack<>();
        TreeNode cur=root;
        while(!stack.isEmpty() || cur!=null){
            if(cur!=null){
                stack.push(cur);
                cur=cur.left;
            }else{
                cur=stack.pop();
                if(rec.contains(cur) || cur.right==null){
                    res.add(cur.val); 
                    cur=null;
                }else{
                    stack.push(cur);
                    rec.add(cur); // 标识已经入过stack
                    cur=cur.right;
                }    
            }
        }
        return res;
    }
}
```

prev标识
```java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        if (root == null) {
            return res;
        }

        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        TreeNode prev = null;
        while (root != null || !stack.isEmpty()) {
            while (root != null) {
                stack.push(root);
                root = root.left;
            }
            root = stack.pop();
            if (root.right == null || root.right == prev) {
                res.add(root.val);
                prev = root;
                root = null;
            } else {
                stack.push(root);
                root = root.right;
            }
        }
        return res;
    }
}
```

> 迭代的时间复杂度
> - 时间：`O(n)`
>-  空间：`O(log_2 n)`  树的深度


## 翻转二叉树

>给你一棵二叉树的根节点 `root` ，翻转这棵二叉树，并返回其根节点。
> 
> 示例：
> - 输入：`root = [4,2,7,1,3,6,9]`
> - 输出：`[4,7,2,9,6,3,1]`
>  
> 说明：
> - 树中节点数目范围在 `[0, 100]` 内
> - `-100 <= Node.val <= 100`
> 
> [226. 翻转二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/invert-binary-tree/description/)

递归

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        // 终止条件
        if(root==null) return root;
        // 递归调用
        root.left=invertTree(root.left);
        root.right=invertTree(root.right);
        // 操作    
        TreeNode temp=root.left;
        root.left=root.right;
        root.right=temp;
        return root;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(log_2 n)` 递归调用栈

--- 
迭代 todo

## 对称二叉树 #rep

>给你一个二叉树的根节点 `root` ， 检查它是否轴对称。
> 
> 示例：
> - 输入：`root = [1,2,2,3,4,4,3]`
> - 输出：`true`
>  
> 说明：
> - 树中节点数目范围在 `[0, 1000]` 内
> - `-100 <= Node.val <= 100`
> 
> [101. 对称二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/symmetric-tree/description/)

递归

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if (root == null) return true;
        return compare(root.left, root.right);
    }

    // 同层级的两个对称节点判断返回值，需要两个参数
    // left是左半树，right是右半树
    public boolean compare(TreeNode left, TreeNode right) {
        // 终止条件
        // 1）都为null
        if (left == null && right == null) return true;
        // 2）只有一个为null
        else if (left == null || right == null) return false;
        // 3）都不为null
        else if (left.val != right.val) return false;
        
        // 当前left和right对称了。递归调用
        boolean outside = compare(left.left, right.right);
        boolean inside = compare(left.right, right.left);
        
        // 操作
        return outside && inside;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(log_2 n)` 递归调用栈

---
迭代 todo

## 二叉树的最大深度

>给定一个二叉树 `root` ，返回其最大深度。
> 
> 二叉树的 最大深度 是指从根节点到**最远叶子节点的最长路径**上的节点数。
> 
> 示例：
> - 输入：`root = [3,9,20,null,null,15,7]`
> - 输出：`3`
>  
> 说明：
> - 树中节点数目范围在 `[0, 10^4]` 内
> - `-100 <= Node.val <= 100`
> 
> [104. 二叉树的最大深度 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-depth-of-binary-tree/description/)

递归
```java
class Solution {
    public int maxDepth(TreeNode root) {
        if(root==null) return 0;
        return Math.max(maxDepth(root.left), maxDepth(root.right))+1;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(log_2 n)` 递归调用栈

---
迭代 todo

## 二叉树的最小深度

> 题目 [104. 二叉树的最大深度 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-depth-of-binary-tree/description/)

给定一个二叉树，找出其最小深度。

最小深度是从**根节点到最近叶子节点的最短路径**上的节点数量。

示例：
- 输入：`root = [3,9,20,null,null,15,7]`
- 输出：`2`
 
说明：
- 树中节点数目范围在 `[0, 10^4]` 内
- `-100 <= Node.val <= 100`

> 思路一：递归

本题不能简单地将[二叉树的最大深度](#二叉树的最大深度)中的`Math.max()`改成`Math.min()`，会受到叶子节点的干扰。

例：`1,null,3`。minDepth是2而不是1

1）By Boer
```java
class Solution {
    public int minDepth(TreeNode root) {
        if(root==null) return 0;
        else if(root.left==null && root.right==null) return 1;
        else if(root.left==null && root.right!=null) return minDepth(root.right)+1;
        else if(root.left!=null && root.right==null) return minDepth(root.left)+1;
        else return Math.min(minDepth(root.left), minDepth(root.right))+1;
    }
}
```

2）优化后
```java
class Solution {
    public int minDepth(TreeNode root) {
        if(root==null) return 0;
        else if(root.left==null ) return minDepth(root.right)+1;
        else if(root.right==null ) return minDepth(root.left)+1;
        else return Math.min(minDepth(root.left), minDepth(root.right))+1;
    }
}
```

> 时间：`O(n)`
> 
> 空间：`O(log_2 n)` 递归调用栈

---
> 思路二：迭代

todo

## 完全二叉树的节点个数

> 题目 [222. 完全二叉树的节点个数 - 力扣（LeetCode）](https://leetcode.cn/problems/count-complete-tree-nodes/description/)

给你一棵 **完全二叉树** 的根节点 `root` ，求出该树的节点个数。

[完全二叉树](https://baike.baidu.com/item/%E5%AE%8C%E5%85%A8%E4%BA%8C%E5%8F%89%E6%A0%91/7773232?fr=aladdin) 的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 `h` 层，则该层包含 `1~ 2^h` 个节点。

示例：
- 输入：`root = [1,2,3,4,5,6]`
- 输出：`6`
 
说明：
- 树中节点的数目范围是`[0, 5 * 10^4]`
- `0 <= Node.val <= 5 * 10^4`
- 题目数据保证输入的树是 **完全二叉树**

进阶：遍历树来统计节点是一种时间复杂度为 `O(n)` 的简单解决方案。你可以设计一个更快的算法吗？

> 思路一：递归

后序：当前树节点总数=左子树节点总数+右子树节点总数+1

```java
class Solution {
    public int countNodes(TreeNode root) {
        if(root==null) return 0;
        return countNodes(root.left)+countNodes(root.right)+1;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

层序 todo

## 平衡二叉树

[222. 完全二叉树的节点个数 - 力扣（LeetCode）](https://leetcode.cn/problems/count-complete-tree-nodes/description/)

给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：一个二叉树_每个节点_ 的左右两个子树的高度差的绝对值不超过 1 。

示例：
- 输入：`root = [3,9,20,null,null,15,7]`
- 输出：`true`
 
说明：
- 树中的节点数在范围 `[0, 5000]` 内
- `-104 <= Node.val <= 104`

> 思路一：递归

后序：求树的最大深度

```java
class Solution {
    boolean flag=true;

    public boolean isBalanced(TreeNode root) {
        maxDepth(root);
        return flag;
    }

    // max(左子树最大深度, 右子树最大深度)
    public int maxDepth(TreeNode root){
        if(root==null) return 0;
        if(flag==false) return 0;
        int leftDepth=maxDepth(root.left);
        int rightDepth=maxDepth(root.right);
        int gap=Math.abs(leftDepth-rightDepth);
        if(gap>1) flag=false;
        return Math.max(leftDepth, rightDepth)+1;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

todo

## 二叉树的所有路径

> 题目 [257. 二叉树的所有路径 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-paths/description/)

给你一个二叉树的根节点 `root` ，按 **任意顺序** ，返回所有从根节点到叶子节点的路径。

**叶子节点** 是指没有子节点的节点。

示例：
- 输入：`root = [1,2,3,null,5]`
- 输出：`["1->2->5","1->3"]`
 
说明：
- 树中节点的数目在范围 `[1, 100]` 内
- `-100 <= Node.val <= 100`**

> 思路一：递归

先序

```java
public class Solution {
    public List<String> res = new ArrayList();

    public List<String> binaryTreePaths(TreeNode root) {
        getPath(root, "");
        return res;
    }

    public void getPath(TreeNode node, String path) {
        if (node == null)
            return;
        // 叶子结点
        if (node.left == null && node.right == null) {
            res.add(path + node.val);
            return;
        }
        getPath(node.right, path + node.val + "->");
        getPath(node.left, path + node.val + "->");
        return;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

todo

## 左叶子之和

> 题目 [404. 左叶子之和 - 力扣（LeetCode）](https://leetcode.cn/problems/sum-of-left-leaves/)

给定二叉树的根节点 `root` ，返回所有左叶子之和。

示例：
- 输入: `root = [3,9,20,null,null,15,7] `
- 输出: 24 
- 解释：在这个二叉树中，有两个左叶子，分别是 9 和 15，所以返回 24
 
说明：
- 节点数在 `[1, 1000]` 范围内
- `-1000 <= Node.val <= 1000`

> 思路一：递归

先序。By Boer.
```java
public class Solution {
    public List<String> res = new ArrayList();

    public List<String> binaryTreePaths(TreeNode root) {
        getPath(root, "");
        return res;
    }

    public void getPath(TreeNode node, String path) {
        if (node == null)
            return;
        // 叶子结点
        if (node.left == null && node.right == null) {
            res.add(path + node.val);
            return;
        }
        getPath(node.right, path + node.val + "->");
        getPath(node.left, path + node.val + "->");
        return;
    }
}
```

后序参考代码
```java
class Solution {
    public int sumOfLeftLeaves(TreeNode root) {
        if (root == null) return 0;
        int leftValue = sumOfLeftLeaves(root.left);    // 左
        int rightValue = sumOfLeftLeaves(root.right);  // 右
                                                       
        int midValue = 0;
        if (root.left != null && root.left.left == null && root.left.right == null) { 
            midValue = root.left.val;
        }
        int sum = midValue + leftValue + rightValue;  // 中
        return sum;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

todo

## 找树左下角的值 #rep

[513. 找树左下角的值 - 力扣（LeetCode）](https://leetcode.cn/problems/find-bottom-left-tree-value/description/)

思路一：递归

先序。先递归左子树以及 `depth > maxDepth` 能够确保同一层获取的是最左边的节点值。
```java
class Solution {
    int res;
    int maxDepth;

    public int findBottomLeftValue(TreeNode root) {
        inorder(root, 1);
        return res;
    }

    public void inorder(TreeNode root, int depth) {
        if (root == null)
            return;
        if (root.left == null && root.right == null) {
            if (depth > maxDepth) {
                maxDepth = depth;
                res = root.val;
            }
        }
        inorder(root.left, depth + 1);
        inorder(root.right, depth + 1);
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

#todo

## 路径总和

[112. 路径总和 - 力扣（LeetCode）](https://leetcode.cn/problems/path-sum/description/)

先序简便写法。targetsum 每次减掉当前节点值，不用算当前 path 的总和

```java
class solution {
    public boolean haspathsum(treenode root, int targetsum) {
        if (root == null) return false; // 为空退出

        // 叶子节点判断是否符合
        if (root.left == null && root.right == null) return root.val == targetsum;

        // 求两侧分支的路径和
        return haspathsum(root.left, targetsum - root.val) || haspathsum(root.right, targetsum - root.val);
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

#todo

## 从中序与后序遍历序列构造二叉树 #rep

> 题目 [106. 从中序与后序遍历序列构造二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/description/)

给定两个整数数组 `inorder` 和 `postorder` ，其中 `inorder` 是二叉树的中序遍历， `postorder` 是同一棵树的后序遍历，请你构造并返回这颗 _二叉树_ 。

示例：
- 输入：`inorder = [9,3,15,20,7], postorder = [9,15,7,20,3]`
- 输出：`[3,9,20,null,null,15,7]`

说明：
- `1 <= inorder.length <= 3000`
- `postorder.length == inorder.length`
- `-3000 <= inorder[i], postorder[i] <= 3000`
- `inorder` 和 `postorder` 都由 **不同** 的值组成
- `postorder` 中每一个值都在 `inorder` 中
- `inorder` **保证**是树的中序遍历
- `postorder` **保证**是树的后序遍历

> 思路一：递归

先序
```java
public class Solution {
    Map<Integer, Integer> map;
    int[] myInorder;
    int[] myPostorder;
    
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        map = new HashMap<>();
        for (int i = 0; i < inorder.length; i++) {
            map.put(inorder[i], i);
        }
        myInorder = inorder;
        myPostorder = postorder;
        return findRoot(0, inorder.length - 1, 0, postorder.length - 1);
    }

    public TreeNode findRoot(int inBegin, int inEnd, int postBegin, int postEnd) {
        if (inEnd < inBegin || postEnd < postBegin)
            return null;
        int rootIndex = map.get(myPostorder[postEnd]);
        TreeNode root = new TreeNode(myInorder[rootIndex]);
        // 左子树序列
        root.left = findRoot(inBegin, rootIndex - 1, postBegin, postBegin + rootIndex - inBegin - 1);
        // 右子树序列
        root.right = findRoot(rootIndex + 1, inEnd, postBegin + rootIndex - inBegin, postEnd - 1);
        return root;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

## 最大二叉树

> 题目 [654. 最大二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-binary-tree/description/)

给定一个不重复的整数数组 `nums` 。 **最大二叉树** 可以用下面的算法从 `nums` 递归地构建:

1. 创建一个根节点，其值为 `nums` 中的最大值。
2. 递归地在最大值 **左边** 的 **子数组前缀上** 构建左子树。
3. 递归地在最大值 **右边** 的 **子数组后缀上** 构建右子树。

返回 _`nums` 构建的_ **_最大二叉树_** 。

示例：
- 输入：`nums = [3,2,1,6,0,5]`
- 输出：`[6,3,5,null,2,0,null,null,1]`
- 解释：递归调用如下所示：
	- `[3,2,1,6,0,5]` 中的最大值是 6 ，左边部分是 `[3,2,1] `，右边部分是 `[0,5]` 。
	    - `[3,2,1]` 中的最大值是 3 ，左边部分是 `[]` ，右边部分是 `[2,1]` 。
	        - 空数组，无子节点。
	        - `[2,1]` 中的最大值是 2 ，左边部分是 `[]` ，右边部分是 `[1]` 。
	            - 空数组，无子节点。
	            - 只有一个元素，所以子节点是一个值为 1 的节点。
	    - `[0,5]` 中的最大值是 5 ，左边部分是 `[0]` ，右边部分是 `[]` 。
	        - 只有一个元素，所以子节点是一个值为 0 的节点。
	        - 空数组，无子节点。

说明：
- `1 <= nums.length <= 1000`
- `0 <= nums[i] <= 1000`
- `nums` 中的所有整数 **互不相同**

> 思路一：递归

先序
```java
class Solution {
    int[] myNums;

    public TreeNode constructMaximumBinaryTree(int[] nums) {
        myNums = nums;
        return construct(0, nums.length - 1);
    }

    public TreeNode construct(int start, int end) {
        if (start > end)
            return null;
        if (start == end)
            return new TreeNode(myNums[start]);
        // 获取最大值
        int maxVal = myNums[start];
        int maxIdx = start;
        for (int i = start + 1; i <= end; i++) {
            if (myNums[i] > maxVal) {
                maxVal = myNums[i];
                maxIdx = i;
            }
        }
        // 新建节点
        TreeNode node = new TreeNode(maxVal);
        // 递归调用
        node.left = construct(start, maxIdx - 1);
        node.right = construct(maxIdx + 1, end);
        return node;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

## 合并二叉树 #rep

> 题目 [617. 合并二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/merge-two-binary-trees/description/)

给你两棵二叉树： `root1` 和 `root2` 。

想象一下，当你将其中一棵覆盖到另一棵之上时，两棵树上的一些节点将会重叠（而另一些不会）。你需要将这两棵树合并成一棵新二叉树。合并的规则是：如果两个节点重叠，那么将这两个节点的值相加作为合并后节点的新值；否则，**不为** null 的节点将直接作为新二叉树的节点。

返回合并后的二叉树。

**注意:** 合并过程必须从两个树的根节点开始。

示例：
- 输入：`root1 = [1,3,2,5], root2 = [2,1,3,null,4,null,7]`
- 输出：`[3,4,5,5,4,null,7]`
 
说明：
- 两棵树中的节点数目在范围 `[0, 2000]` 内
- `-10^4 <= Node.val <= 10^4`

> 思路一：递归

先序
```java
class Solution {
    public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
        if (root1 == null)
            return root2;
        if (root2 == null)
            return root1;
        TreeNode node = new TreeNode(root1.val + root2.val);
        node.left = mergeTrees(root1.left, root2.left);
        node.right = mergeTrees(root1.right, root2.right);
        return node;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

todo

## 二叉树中的最大路径和 #困难 #手撕 #rep

[124. 二叉树中的最大路径和 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-maximum-path-sum/description/)

思路：
- 每个子树中都有最大路径和，通过**后序遍历**获得所有子树的最大路径和，取最大值
- 如何返回当前子树能向父节点“**提供**”的最大路径和？以下三种情况取最大值
	- 从左子树网上 `root.val + dfs(root.left)`
	- 从右子树网上 `root.val + dfs(root.right)`
	- 子树提供不了收益，返回 0

```java
class Solution {
    int res = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        traversal(root);
        return res;
    }

    public int traversal(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int left = traversal(root.left);
        int right = traversal(root.right);
        res = Math.max(res, left + right + root.val);
        int max = Math.max(root.val + left, root.val + right);
        return max > 0 ? max : 0;
    }
}
```

## ---------- 二叉搜索树

## 二叉搜索树中的搜索

> 题目 [700. 二叉搜索树中的搜索 - 力扣（LeetCode）](https://leetcode.cn/problems/search-in-a-binary-search-tree/description/)

给定二叉搜索树（BST）的根节点 `root` 和一个整数值 `val`。

你需要在 BST 中找到节点值等于 `val` 的节点。 返回以该节点为根的子树。 如果节点不存在，则返回 `null` 。

示例：
- 输入：`root = [4,2,7,1,3], val = 2`
- 输出：`[2,1,3]`
 
说明：
- 树中节点数在 `[1, 5000]` 范围内
- `1 <= Node.val <= 107`
- `root` 是二叉搜索树
- `1 <= val <= 10^7`

> 思路一：递归

先序
```java
class Solution {
    public TreeNode searchBST(TreeNode root, int val) {
        if (root == null || root.val == val)
            return root;
        if (root.val < val)
            return searchBST(root.right, val);
        else
            return searchBST(root.left, val);
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

todo

## 验证二叉搜索树 #rep

> 题目 [98. 验证二叉搜索树 - 力扣（LeetCode）](https://leetcode.cn/problems/validate-binary-search-tree/description/)

给你一个二叉树的根节点 `root` ，判断其是否是一个有效的二叉搜索树。

**有效** 二叉搜索树定义如下：

- 节点的左子树只包含 **小于** 当前节点的数。
- 节点的右子树只包含 **大于** 当前节点的数。
- 所有左子树和右子树自身必须也是二叉搜索树。

示例：
- 输入：`root = [2,1,3]`
- 输出：`true`
 
说明：
- 树中节点数目范围在`[1, 104]` 内
- `-231 <= Node.val <= 231 - 1`

> 思路一：递归

【陷阱】不能单纯的比较左节点小于中间节点，右节点大于中间节点就完事了。我们要比较的是 左子树所有节点小于中间节点，右子树所有节点大于中间节点

中序遍历下，输出的二叉搜索树节点的数值是有序序列，可以递归中序遍历将二叉搜索树转变成一个数组，看这个数组是不是有序的就可以了

但其实不用转变成数组，可以在递归遍历的过程中直接判断是否有序。

中序
```java
class Solution {
    TreeNode max;

    public boolean isValidBST(TreeNode root) {
        if (root == null)
            return true;
        boolean left = isValidBST(root.left);
        if (!left) {
            return false; // 剪枝
        }
        if (max != null && root.val <= max.val)
            return false; // 剪枝
        max = root;
        boolean right = isValidBST(root.right);
        return right;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

todo

## 二叉搜索树的最小绝对差

> [530. 二叉搜索树的最小绝对差 - 力扣（LeetCode）](https://leetcode.cn/problems/minimum-absolute-difference-in-bst/description/)

给你一个二叉搜索树的根节点 `root` ，返回 **树中任意两不同节点值之间的最小差值** 。

差值是一个正数，其数值等于两值之差的绝对值。

> 示例：
> - 输入：`root = [4,2,6,1,3]`
> - 输出：1
>  
> 说明：
> - 树中节点的数目范围是 `[2, 104]`
> - `0 <= Node.val <= 105`

---
递归，中序遍历

```java
class Solution {
    int min=Integer.MAX_VALUE;
    TreeNode pre;

    public int getMinimumDifference(TreeNode root) {
        inOrder(root);
        return min;
    }

    public void inOrder(TreeNode root) {
        if (root == null)
            return;
        inOrder(root.left);
        if(pre!=null)
            min = Math.min(min, root.val - pre.val);
        pre = root;
        inOrder(root.right);
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

Todo

## 二叉搜索树中的众数  #rep

> [501. 二叉搜索树中的众数 - 力扣（LeetCode）](https://leetcode.cn/problems/find-mode-in-binary-search-tree/description/)

给你一个含重复值的二叉搜索树（BST）的根节点 `root` ，找出并返回 BST 中的所有 [众数](https://baike.baidu.com/item/%E4%BC%97%E6%95%B0/44796)（即，出现频率最高的元素）。

如果树中有不止一个众数，可以按 **任意顺序** 返回。

假定 BST 满足如下定义：

- 结点左子树中所含节点的值 **小于等于** 当前节点的值
- 结点右子树中所含节点的值 **大于等于** 当前节点的值
- 左子树和右子树都是二叉搜索树

> 示例：
> - 输入：`root = [1,null,2,2]`
> - 输出：`[2]`
>  
> 说明：
> - 树中节点的数目在范围 `[1, 10^4]` 内
> - `-10^5 <= Node.val <= 10^5`

---
递归，中序

- 难度在于如何一次遍历就得到众数的集合，并且不使用额外的空间（递归调用栈除外）

```java
class Solution {
    List<Integer> list = new ArrayList<>();
    int maxCount;
    int count;
    TreeNode pre;

    public int[] findMode(TreeNode root) {
        traversal(root);
        int[] res = new int[list.size()];
        for (int i = 0; i < res.length; i++) {
            res[i] = list.get(i);
        }
        return res;
    }

    public void traversal(TreeNode root) {
        if (root == null)
            return;
        traversal(root.left);
        if (pre == null || root.val != pre.val) {
            count = 1;
        } else {
            count++;
        }
        if (count > maxCount) {
            list.clear();
            list.add(root.val);
            maxCount = count;
        } else if (count == maxCount) {
            list.add(root.val);
        }
        pre = root;
        traversal(root.right);
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

> 思路二：迭代

Todo

## 二叉树的最近公共祖先

> 题目 [501. 二叉搜索树中的众数 - 力扣（LeetCode）](https://leetcode.cn/problems/find-mode-in-binary-search-tree/description/)

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

[百度百科](https://baike.baidu.com/item/%E6%9C%80%E8%BF%91%E5%85%AC%E5%85%B1%E7%A5%96%E5%85%88/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

示例：
- 输入：`root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1`
- 输出：`3`
- 解释：节点 `5` 和节点 `1` 的最近公共祖先是节点 `3 。`
 
说明：
- 树中节点数目在范围 `[2, 10^5]` 内。
- `-10^9 <= Node.val <= 10^9`
- 所有 `Node.val` `互不相同` 。
- `p != q`
- `p` 和 `q` 均存在于给定的二叉树中。

> 思路一：递归

后序 By Boer.
```java
class Solution {
    TreeNode res;

    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        traversal(root, p, q);
        return res;
    }

    public boolean traversal(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null) {
            return false;
        } else if (root == p) {
            res = p;
            return true;
        } else if (root == q) {
            res = q;
            return true;
        }
        boolean left = traversal(root.left, p, q);
        boolean right = traversal(root.right, p, q);
        if (left == true && right == true) {
            res = root;
            return true;
        }
        return left || right;
    }
}
```

优化写法
```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) { // 递归结束条件
            return root;
        }

        // 后序遍历
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        if(left == null && right == null) { // 若未找到节点 p 或 q
            return null;
        }else if(left == null && right != null) { // 若找到一个节点
            return right;
        }else if(left != null && right == null) { // 若找到一个节点
            return left;
        }else { // 若找到两个节点
            return root;
        }
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

## 二叉树搜索树的最近公共祖先  #rep

[501. 二叉搜索树中的众数 - 力扣（LeetCode）](https://leetcode.cn/problems/find-mode-in-binary-search-tree/description/)

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

最近公共祖先的定义为：“ 对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

> 示例：
> - 输入：`root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8`
> - 输出：`6`
>  
> 	说明：
> - 所有节点的值都是唯一的。
> - p、q 为不同节点且均存在于给定的二叉搜索树中。

---
【思路一：递归】

- 等同于：沿着一条边搜索一棵树（先序的思想）
- 一定能找到公共祖先，所以不需要判空了

> 沿着一条边搜索一棵树模板：
> 
> ```java
> if (递归函数(root->left)) return ;
> if (递归函数(root->right)) return ;
> ```
> 
> 搜索整个树模板：
> 
> ```java
> left = 递归函数(root->left);
> right = 递归函数(root->right);
> left与right的逻辑处理;
> ```

中序
```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root.val>p.val && root.val>q.val){
            return lowestCommonAncestor(root.left,p,q);
        }
        if(root.val<p.val && root.val<q.val){
            return lowestCommonAncestor(root.right,p,q);
        }
        // root.val在中间，root.val==p.val，root.val==q.val
        return root;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

---
【思路二：迭代】

Todo
```java

```

## 二叉树搜索树中的插入操作

[701. 二叉搜索树中的插入操作 - 力扣（LeetCode）](https://leetcode.cn/problems/insert-into-a-binary-search-tree/description/)

给定二叉搜索树（BST）的根节点 `root` 和要插入树中的值 `value` ，将值插入二叉搜索树。返回插入后二叉搜索树的根节点。输入数据 **保证** ，新值和原始二叉搜索树中的任意节点值都不同。

**注意**，可能存在多种有效的插入方式，只要树在插入后仍保持为二叉搜索树即可。 你可以返回 **任意有效的结果** 。

> 示例：
> - 输入：`root = [4,2,7,1,3], val = 5`
> - 输出：`[4,2,7,1,3,5]`
>  
> 说明：
> - 树中的节点数将在 `[0, 104]` 的范围内。
> - `-108 <= Node.val <= 108`
> - 所有值 `Node.val` 是 **独一无二** 的。
> - `-108 <= val <= 108`
> - **保证** `val` 在原始BST中不存在。

---
【思路一：递归】

中序
```java
class Solution {
    public TreeNode insertIntoBST(TreeNode root, int val) {
        if (root == null) {
            root = new TreeNode(val);
        } else if (root.val < val) {
            root.right = insertIntoBST(root.right, val);
        } else {
            root.left = insertIntoBST(root.left, val);
        }
        return root;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

---
【思路二：迭代】

Todo
```java

```

## 删除二叉搜索树中的节点  #rep

> 题目： [450. 删除二叉搜索树中的节点 - 力扣（LeetCode）](https://leetcode.cn/problems/delete-node-in-a-bst/description/)

给定一个二叉搜索树的根节点 **root** 和一个值 **key**，删除二叉搜索树中的 **key** 对应的节点，并保证二叉搜索树的性质不变。返回二叉搜索树（有可能被更新）的根节点的引用。

一般来说，删除节点可分为两个步骤：
1. 首先找到需要删除的节点；
2. 如果找到了，删除它。

> 示例：
> - 输入：`root = [5,3,6,2,4,null,7], key = 3`
> - 输出：`[5,4,6,2,null,null,7]`
>  
> 说明：
> - 节点数的范围 `[0, 104]`.
> - `-105 <= Node.val <= 105`
> - 节点值唯一
> - `root` 是合法的二叉搜索树
> - `-105 <= key <= 105`

---
【思路一：递归】
- 删除当前 root 不用真的删除节点，直接返回子节点或者 null 即可
- 前序

```java
class Solution {
    public TreeNode deleteNode(TreeNode root, int key) {
        // 没找到
        if (root == null) {
            return null;
        }
        if (root.val == key) {
            // 叶节点：直接删
            if (root.left == null && root.right == null) {
                return null;
            } else if (root.left == null) {
                // 右子树为空
                return root.right;
            } else if (root.right == null) {
                // 左子树为空
                return root.left;
            } else {
                // 左右子树都不为空
                TreeNode cur = root.right;
                while (cur.left != null) {
                    cur = cur.left;
                }
                cur.left = root.left;
                return root.right;
            }
        } else if (root.val > key) {
            root.left = deleteNode(root.left, key);
        } else {
            root.right = deleteNode(root.right, key);
        }
        return root;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

---
【思路二：迭代】

Todo
```java

```


## 修建二叉搜索树 #rep

> 题目： [669. 修剪二叉搜索树 - 力扣（LeetCode）](https://leetcode.cn/problems/trim-a-binary-search-tree/description/)

给你二叉搜索树的根节点 `root` ，同时给定最小边界 `low` 和最大边界 `high`。通过修剪二叉搜索树，使得所有节点的值在 `[low, high]` 中。修剪树 **不应该** 改变保留在树中的元素的相对结构 (即，如果没有被移除，原有的父代子代关系都应当保留)。可以证明，存在 **唯一的答案** 。

所以结果应当返回修剪好的二叉搜索树的新的根节点。注意，根节点可能会根据给定的边界发生改变。

> 示例：
> - 输入：`root = [1,0,2], low = 1, high = 2`
> - 输出：`[1,null,2]`
>  
> 说明：
> - 树中节点数在范围 `[1, 104]` 内
> - `0 <= Node.val <= 104`
> - 树中每个节点的值都是 **唯一** 的
> - 题目数据保证输入是一棵有效的二叉搜索树
> - `0 <= low <= high <= 104`

---
【思路一：递归】
- 不要收到 [修建二叉搜索树  #rep]( #修建二叉搜索树 %20 (1)) 的影响
- 利用二叉搜索树的特性
- 前序

```java
class Solution {
    public static TreeNode trimBST(TreeNode root, int low, int high) {
        // 退出条件
        if (root == null) {
            return null;
        }
        // 本层操作
        if (root.val < low) {
            return trimBST(root.right, low, high);
        }
        if (root.val > high) {
            return trimBST(root.left, low, high);
        }
        // 递归调用
        root.left = trimBST(root.left, low, high);
        root.right = trimBST(root.right, low, high);
        return root;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

---
【思路二：迭代】

Todo
```java

```

## 将有序数组转换为二叉搜索树

> 题目： [108. 将有序数组转换为二叉搜索树 - 力扣（LeetCode）](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/description/)

给你一个整数数组 `nums` ，其中元素已经按 **升序** 排列，请你将其转换为一棵 **高度平衡** 二叉搜索树。

**高度平衡** 二叉树是一棵满足「每个节点的左右两个子树的高度差的绝对值不超过 1 」的二叉树。

> 示例：
> - 输入：`nums = [-10,-3,0,5,9]`
> - 输出：`[0,-3,9,-10,null,5]`
>  
> 说明：
> - `1 <= nums.length <= 104`
> - `-104 <= nums[i] <= 104`
> - `nums` 按 **严格递增** 顺序排列

---
【思路一：递归】
- 前序
- 二分查找

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        // 二分查找
        return constructTree(nums, 0, nums.length - 1);
    }

    public TreeNode constructTree(int[] nums, int l, int r) {
        // 终止条件
        if (l > r)
            return null;
        // 本层操作
        int mid = (l + r) / 2;
        TreeNode node = new TreeNode(nums[mid]);
        // 递归调用
        node.left = constructTree(nums, l, mid - 1);
        node.right = constructTree(nums, mid + 1, r);
        // 返回值
        return node;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

---
【思路二：迭代】

Todo
```java

```

## 把二叉搜索树转换为累加树

> 题目： [538. 把二叉搜索树转换为累加树 - 力扣（LeetCode）](https://leetcode.cn/problems/convert-bst-to-greater-tree/description/)

给出二叉 **搜索** 树的根节点，该树的节点值各不相同，请你将其转换为累加树（Greater Sum Tree），使每个节点 `node` 的新值等于原树中大于或等于 `node.val` 的值之和。

> 示例：
> - 输入：`[4,1,6,0,2,5,7,null,null,null,3,null,null,null,8]`
> - 输出：`[30,36,21,36,35,26,15,null,null,null,33,null,null,null,8]`
>  
> 说明：
> - 树中的节点数介于 `0` 和 `10^4` 之间。
> - 每个节点的值介于 `-10^4` 和 `10^4` 之间。
> - 树中的所有值 **互不相同** 。
> - 给定的树为二叉搜索树。

---
【思路一：递归】
- 逆中序：右中左

```java
class Solution {
    int sum;

    public TreeNode convertBST(TreeNode root) {
        // 终止条件
        if (root == null) {
            return null;
        }
        convertBST(root.right);
        sum += root.val;
        root.val = sum;
        convertBST(root.left);
        return root;
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(log_2 n)` 递归调用栈

---
【思路二：迭代】

Todo
```java

```

## 补充：完全二叉树插入器  #rep

> 题目：[919. 完全二叉树插入器 - 力扣（LeetCode）](https://leetcode.cn/problems/complete-binary-tree-inserter/description/)

**完全二叉树** 是每一层（除最后一层外）都是完全填充（即，节点数达到最大）的，并且所有的节点都尽可能地集中在左侧。

设计一种算法，将一个新节点插入到一个完整的二叉树中，并在插入后保持其完整。

实现 `CBTInserter` 类:
- `CBTInserter(TreeNode root)` 使用头节点为 `root` 的给定树初始化该数据结构；
- `CBTInserter.insert(int v)`  向树中插入一个值为 `Node.val == val`的新节点 `TreeNode`。使树保持完全二叉树的状态，**并返回插入节点** `TreeNode` **的父节点的值**；
- `CBTInserter.get_root()` 将返回树的头节点。

> 示例：
> - 输入：
> 	- `["CBTInserter", "insert", "insert", "get_root"]`
> 	- `[[[1, 2]], [3], [4], []]`
> - 输出：`[null, 1, 2, [1, 2, 3, 4]]`
>  
> 说明：
> - `CBTInserter cBTInserter = new CBTInserter([1, 2]);`
> - `cBTInserter.insert(3);  // 返回 1`
> - `cBTInserter.insert(4);  // 返回 2`
> - `cBTInserter.get_root(); // 返回 [1, 2, 3, 4]`

思路：把层序的顺序保存在列表里，并且记录首个左右孩子不全的节点在 list 中的索引

```java
class CBTInserter {
    List<TreeNode> list = new ArrayList<>();
    // 记录首个左右孩子不全的节点在list中的索引
    int idx = 0;

    public CBTInserter(TreeNode root) {
        list.add(root);
        int cur = 0;
        // 把所有节点加入到list里
        while (cur < list.size()) {
            TreeNode node = list.get(cur);
            if (node.left != null) {
                list.add(node.left);
            }
            if (node.right != null) {
                list.add(node.right);
            }
            cur++;
        }
    }

    public int insert(int val) {
        TreeNode node = new TreeNode(val);
        // 左右子节点都不为空
        while (list.get(idx).left != null && list.get(idx).right != null) {
            idx++;
        }

        TreeNode fa = list.get(idx);
        if (fa.left == null) {
            fa.left = node;
        } else if (fa.right == null) {
            fa.right = node;
        }
        list.add(node);
        return fa.val;
    }

    public TreeNode get_root() {
        return list.get(0);
    }
}
```

复杂度分析：
- 时间：`O(n)`
- 空间：`O(n)`

# 回溯

回溯法也可以叫做“回溯搜索法”，它是一种搜索的方式。

- 回溯是**递归**的副产品，只要有递归就会有回溯。
- 基本思想类同于：图的深度优先搜索、二叉树的**后序遍历**
- 回溯的本质就是**暴力穷举**，可以通过剪枝优化
- 回溯的两个关键问题：
	- 本层遍历集合的确定（横向去重、startIndex）
	- 本层各节点的确定（纵向去重）

> #Boer 有的问题暴力是写不出来了的！比如 n 数之和就得写 n 个循环，所以只能通过回溯

回溯模板代码：

```java
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本层集合中元素) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
}
```

## ---------- 组合

**N 个数**里面按一定规则找出 **k 个数的集合**

## 组合 #rep

> 题目：[77. 组合 - 力扣（LeetCode）](https://leetcode.cn/problems/combinations/)

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

你可以按 **任何顺序** 返回答案。

> 示例：
> - 输入： `n = 4, k = 2`
> - 输出：`[[2,4],[3,4],[2,3],[1,2],[1,3],[1,4],]`
>  
> 说明：
> - `1 <= n <= 20`
> - `1 <= k <= n`

![600](assets/Pasted%20image%2020240201224507.png)

回溯法

- path 用于控制层数
- startIndex 用于控制下一层的节点个数
- 注意：res 保存的时候一定要复制 path 到一个新的 list，因为 path 后续会改动

```java
class Solution {
    List<List<Integer>> result= new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();
    public List<List<Integer>> combine(int n, int k) {
        backtracking(n,k,1);
        return result;
    }

    public void backtracking(int n,int k,int startIndex){
        if (path.size() == k){
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i =startIndex;i<=n;i++){
            path.add(i);
            backtracking(n,k,i+1);
            path.removeLast();
        }
    }
}
```

剪枝优化：如果 for 循环选择的起始位置之后的元素个数 已经不足 我们需要的元素个数了，那么就没有必要搜索，以 `n=4,k=4` 为例

![650](assets/Pasted%20image%2020240201224654.png)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();
    public List<List<Integer>> combine(int n, int k) {
        combineHelper(n, k, 1);
        return result;
    }

    /**
     * 每次从集合中选取元素，可选择的范围随着选择的进行而收缩，调整可选择的范围，就是要靠startIndex
     * @param startIndex 用来记录本层递归的中，集合从哪里开始遍历（集合就是[1,...,n] ）。
     */
    private void combineHelper(int n, int k, int startIndex){
        //终止条件
        if (path.size() == k){
            result.add(new ArrayList<>(path));
            return;
        }
        // 剪枝：path还需要的元素个数<=剩余的元素个数。k-pathSize<=n-i+1
        for (int i = startIndex; i <= n - (k - path.size()) + 1; i++){
            path.add(i);
            combineHelper(n, k, i + 1);
            path.removeLast();
        }
    }
}
```

复杂度分析：

- 时间： #todo
- 空间： #todo

---

#Bo 每次都 new 一个新 list，没有回溯那一步，很显然比较耗费内存。

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();

    public List<List<Integer>> combine(int n, int k) {
        backtracking(n,k,1,new ArrayList<Integer>());
        return res;
    }

    public void backtracking(int n, int k, int start, List<Integer> list) {
        if(list.size()==k){
            res.add(list);
            return;
        }
        for (int i = start; i <= n; i++) {
            List<Integer> list2 = new ArrayList<>();
            for(Integer item:list){
                list2.add(item);
            }
            list2.add(i);
            backtracking(n,k,i+1,list2);
        }
    }
}
```

## 电话号码的字母组合  #rep

[17. 电话号码的字母组合 - 力扣（LeetCode）](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/description/)

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按 **任意顺序** 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![](assets/Pasted%20image%2020240203215403.png)

> 示例：
> - 输入： `digits = "23"`
> - 输出：`["ad","ae","af","bd","be","bf","cd","ce","cf"]`
>  
> 说明：
> - `0 <= digits.length <= 4`
> - `digits[i]` 是范围 `['2', '9']` 的一个数字。

---

回溯法

- 存储数字和字母的印射关系，不用 map，`String[]` 即可
- 获取数字对应的字母串，基本功：`String str = numString[digits.charAt(cur) - '0'];`

![600](assets/Pasted%20image%2020240203215546.png)

```java
class Solution {
    List<String> res = new ArrayList<>();
    StringBuilder path = new StringBuilder();

    public List<String> letterCombinations(String digits) {
        if (digits.length() == 0) {
            return res;
        }
        String[] numString = { "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz" };
        backtracking(digits, numString, 0);
        return res;
    }

    // cur是当前path中的位置（索引），cur决定递归深度，str.length()决定每层的宽度
    public void backtracking(String digits, String[] numString, int cur) {
        // 出口
        if (cur == digits.length()) {
            res.add(path.toString());
            return;
        }
        String str = numString[digits.charAt(cur) - '0'];
        for (int i = 0; i < str.length(); i++) {
            path.append(str.charAt(i));
            backtracking(digits, numString, ++cur);
            // 回溯
            cur--;
            path.deleteCharAt(path.length() - 1);
        }
    }
}
```

## 组合总和 #rep

[17. 电话号码的字母组合 - 力扣（LeetCode）](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/description/)

给你一个 **无重复元素** 的整数数组 `candidates` 和一个目标整数 `target` ，找出 `candidates` 中可以使数字和为目标数 `target` 的 所有 **不同组合** ，并以列表形式返回。你可以按 **任意顺序** 返回这些组合。

`candidates` 中的 **同一个** 数字可以 **无限制重复被选取** 。如果至少一个数字的被选数量不同，则两种组合是不同的。 

对于给定的输入，保证和为 `target` 的不同组合数少于 `150` 个。

> 示例：
> - 输入： `candidates = [2,3,5]`, ` target = 8`
> - 输出：`[[2,2,2,2],[2,3,3],[3,5]]`
>  
> 说明：
> - `1 <= candidates.length <= 30`
> - `2 <= candidates[i] <= 40`
> - `candidates` 的所有元素 **互不相同**
> - `1 <= target <= 40`

解法：回溯法

- 同一个数字可以无限使用，没有 k 的限制

![](assets/Pasted%20image%2020240204205335.png)

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    int sum;

    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        backTracking(candidates, target, 0);
        return res;
    }

    public void backTracking(int[] candidates, int target, int start) {
        if (sum > target) {
            return;
        }
        if (sum == target) {
            res.add(new ArrayList(path));
            return;
        }
        for (int i = start; i < candidates.length; i++) {
            path.add(candidates[i]);
            sum += candidates[i];
            backTracking(candidates, target, i);
            path.remove(path.size() - 1);
            sum -= candidates[i];
        }
    }
}
```

如何剪枝？

- 对总集合排序之后，如果下一层的 sum（就是本层的 `sum + candidates[i]`）已经大于 target，就可以结束本轮 for 循环的遍历。
- 横向的剪枝要在 for 里面进行

![](assets/Pasted%20image%2020240204222152.png)

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    int sum;

    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        backTracking(candidates, target, 0);
        return res;
    }

    public void backTracking(int[] candidates, int target, int start) {
        if (sum == target) {
            res.add(new ArrayList(path));
            return;
        }
        for (int i = start; i < candidates.length; i++) {
            if (sum + candidates[i] > target) {
                return;
            }
            path.add(candidates[i]);
            sum += candidates[i];
            backTracking(candidates, target, i);
            // 回溯
            path.remove(path.size() - 1);
            sum -= candidates[i];
        }
    }
}
```

## 组合总和 II #rep 

[40. 组合总和 II - 力扣（LeetCode）](https://leetcode.cn/problems/combination-sum-ii/description/)

给定一个候选人编号的集合 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。

`candidates` 中的每个数字在每个组合中只能使用 **一次** 。

注意：**解集不能包含重复的组合。**

> 示例：
> - 输入: candidates = `[10,1,2,7,6,1,5]`, target = `8`,
> - 输出：`[[1,1,6],[1,2,5],[1,7],[2,6]]`
>  
> 说明：
> - `1 <= candidates.length <= 100`
> - `1 <= candidates[i] <= 50`
> - `1 <= target <= 30`

回溯法

- 在上一题的基础上增加条件：不能包含重复的元素
- 剪枝的时候要跳过同一树层使用过的元素
- 纵向使用过的元素不用删除的

![700](assets/Pasted%20image%2020240206001408.png)

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    int sum;

    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        backTracing(candidates, target, 0);
        return res;
    }

    public void backTracing(int[] candidates, int target, int start) {
        if (sum == target) {
            res.add(new ArrayList<Integer>(path));
        }
        for (int i = start; i < candidates.length; i++) {
            if (sum + i > target)
                break;
            // 跳过同一树层使用过的元素
            if (i > start && candidates[i] == candidates[i - 1]) {
                continue;
            }
            path.add(candidates[i]);
            sum += candidates[i];
            backTracing(candidates, target, i + 1);
            // 回溯
            path.remove(path.size() - 1);
            sum -= candidates[i];
        }
    }
}
```

## 组合总和 III

[216. 组合总和 III - 力扣（LeetCode）](https://leetcode.cn/problems/combination-sum-iii/description/)

找出所有相加之和为 `n` 的 `k` 个数的组合，且满足下列条件：

- 只使用数字 1 到 9
- 每个数字 **最多使用一次** 

返回 _所有可能的有效组合的列表_ 。该列表不能包含相同的组合两次，组合可以以任何顺序返回。

> 示例：
> - 输入： `k = 3, n = 7`
> - 输出：`[[1,2,4]]`
>  
> 说明：
> - `2 <= k <= 9`
> - `1 <= n <= 60`

---

回溯法

- 剪枝优化：path 还需要的元素个数 > 剩余的元素个数，剪
- 剪枝优化：path 中的总和 > target，剪

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    int sum;
    int target;

    public List<List<Integer>> combinationSum3(int k, int n) {
        target = n;
        // 只能用1~9的数字
        backtracking(k, Math.min(9, n), 1);
        return res;
    }

    public void backtracking(int k, int n, int start) {
        // 剪枝
        if (sum > target) {
            return;
        }
        // 结束条件
        if (path.size() == k) {
            if (sum == target) {
                res.add(new ArrayList<Integer>(path));
            }
            return;
        }
        // 剪枝：path还需要的元素个数<=剩余的元素个数。k-pathSize<=n-i+1
        for (int i = start; i <= n - k + path.size() + 1; i++) {
            path.add(i);
            sum += i;
            backtracking(k, n, i + 1);
            // 回溯
            path.remove(path.size() - 1);
            sum -= i;
        }
    }
}
```

## ---------- 分割

## 分割回文串 #rep

[40. 组合总和 II - 力扣（LeetCode）](https://leetcode.cn/problems/combination-sum-ii/description/)

给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 **回文串** 。返回 `s` 所有可能的分割方案。

**回文串** 是正着读和反着读都一样的字符串。

> 示例：
> - 输入: `s = "aab"`
> - 输出：`[["a","a","b"],["aa","b"]]`
>  
> 说明：
> - `1 <= s.length <= 16`
> - `s` 仅由小写英文字母组成

---
回溯法

![500](assets/Pasted%20image%2020240207222346.png)

```java
class Solution {
    List<List<String>> res = new ArrayList<>();
    List<String> path = new ArrayList<>();

    public List<List<String>> partition(String s) {
        backTracing(s, 0);
        return res;
    }

    public void backTracing(String s, int start) {
        if (start >= s.length()) {
            res.add(new ArrayList<>(path));
        }
        for (int i = start; i < s.length(); i++) {
            if (!isPalindrome(s, start, i)) {
                continue;
            }
            path.add(s.substring(start, i + 1));
            backTracing(s, i + 1);
            // 回溯
            path.remove(path.size() - 1);
        }
    }

    private boolean isPalindrome(String s, int left, int right) {
        while (left < right) {
            if (s.charAt(left) != s.charAt(right)) {
                return false;
            }
            left++;
            right--;
        }
        return true;
    }
}
```

## 复原 IP 地址

[40. 组合总和 II - 力扣（LeetCode）](https://leetcode.cn/problems/combination-sum-ii/description/)

**有效 IP 地址** 正好由四个整数（每个整数位于 `0` 到 `255` 之间组成，且不能含有前导 `0`），整数之间用 `'.'` 分隔。

- 例如：`"0.1.2.201"` 和 `"192.168.1.1"` 是 **有效** IP 地址，但是 `"0.011.255.245"`、`"192.168.1.312"` 和 `"192.168@1.1"` 是 **无效** IP 地址。

给定一个只包含数字的字符串 `s` ，用以表示一个 IP 地址，返回所有可能的**有效 IP 地址**，这些地址可以通过在 `s` 中插入 `'.'` 来形成。你 **不能** 重新排序或删除 `s` 中的任何数字。你可以按 **任何** 顺序返回答案。

> 示例：
> - 输入: `s = "25525511135"`
> - 输出：`["255.255.11.135","255.255.111.35"]`
>  
> 说明：
> - `1 <= s.length <= 16`
> - `s` 仅由小写英文字母组成

---

回溯法

- break代表当前递归结束了，剩余长度不符合的情况应该是跳过循环 continue，因为下一次循环cur变长以后可能满足剩余长度的要求

![](assets/Pasted%20image%2020240213213819.png)

```java
class Solution {
    List<String> res = new ArrayList<>();
    List<String> path = new ArrayList<>();
    int length;

    public List<String> restoreIpAddresses(String s) {
        if (s.length() > 12)
            return res;
        backTracing(s, 0);
        return res;
    }

    public void backTracing(String s, int start) {
        if (path.size() == 4) {
            StringBuilder str = new StringBuilder();
            for (int i = 0; i < 4; i++) {
                if (i == 3) {
                    str.append(path.get(i));
                } else {
                    str.append(path.get(i)).append(".");
                }
            }
            res.add(str.toString());
        }
        for (int i = start; i < s.length(); i++) {
            // 不能含有前导 0
            if (s.charAt(start) == '0' && start < i) {
                break;
            }
            String cur = s.substring(start, i + 1);
            // 剪枝：剩余长度不符合
            if ((4 - path.size() - 1) * 3 + cur.length() + length < s.length()) {
                continue;
            }
            // 数字范围不符合
            if (Integer.parseInt(cur) > 255) {
                break;
            }
            path.add(cur);
            length += cur.length();
            backTracing(s, i + 1);
            path.remove(path.size() - 1);
            length -= cur.length();
        }
    }
}
```

## ---------- 子集

子集问题：一个 N 个数的集合里找出符合条件的子集

## 子集

[40. 组合总和 II - 力扣（LeetCode）](https://leetcode.cn/problems/combination-sum-ii/description/)

给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。

解集 **不能 包含重复**的子集。你可以按 **任意顺序** 返回解集。

> 示例：
> - 输入: `nums = [1,2,3]`
> - 输出：`[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]`
>  
> 说明：
> - `1 <= nums.length <= 10`
> - `-10 <= nums[i] <= 10`
> - `nums` 中的所有元素 互不相同

---

回溯法

- 组合问题和分割问题都是收集树的叶子节点，而子集问题是找树的所有节点！

![](assets/Pasted%20image%2020240214155217.png)

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();

    public List<List<Integer>> subsets(int[] nums) {
        res.add(new ArrayList<Integer>());
        backTracing(nums, 0);
        return res;
    }

    public void backTracing(int[] nums, int start) {
        for (int i = start; i < nums.length; i++) {
            path.add(nums[i]);
            res.add(new ArrayList(path));
            backTracing(nums, i + 1);
            path.remove(path.size() - 1);
        }
    }
}
```

## 子集II

[90. 子集 II - 力扣（LeetCode）](https://leetcode.cn/problems/subsets-ii/description/)

给你一个整数数组 `nums` ，其中**可能包含重复元素**，请你返回该数组所有可能的子集（幂集）

解集**不能包含重复**的子集。返回的解集中，子集可以按 **任意顺序** 排列

> 示例：
> - 输入: `nums = [1,2,2]`
> - 输出：`[[],[1],[1,2],[1,2,2],[2],[2,2]]`
>  
> 说明：
> - `1 <= nums.length <= 10`
> - `-10 <= nums[i] <= 10`

---

回溯法

- 本题和组合的区别：nums 有重复元素，需要去重
- 去重的逻辑：`if (i > start && nums[i] == nums[i - 1]) `

![600](assets/Pasted%20image%2020240215161930.png)

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();

    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        res.add(new ArrayList<Integer>());
        backTracing(nums, 0);
        return res;
    }

    public void backTracing(int[] nums, int start) {
        for (int i = start; i < nums.length; i++) {
            if (i > start && nums[i] == nums[i - 1]) {
                continue;
            }
            path.add(nums[i]);
            res.add(new ArrayList(path));
            backTracing(nums, i + 1);
            path.remove(path.size() - 1);
        }
    }
}
```

## 非递减子序列  #rep

[491. 非递减子序列 - 力扣（LeetCode）](https://leetcode.cn/problems/non-decreasing-subsequences/description/)

给你一个整数数组 `nums` ，找出并返回所有该数组中不同的递增子序列，递增子序列中 **至少有两个元素** 。你可以按 **任意顺序** 返回答案。

数组中可能含有重复元素，如出现两个整数相等，也可以视作递增序列的一种特殊情况。

> 示例：
> - 输入: `nums = [4,6,7,7]`
> - 输出：`[[4,6],[4,6,7],[4,6,7,7],[4,7],[4,7,7],[6,7],[6,7,7],[7,7]]`
>  
> 说明：
> - `1 <= nums.length <= 15`
> - `-100 <= nums[i] <= 100`

---

回溯法

- 本题的原始序列不是有序的，所以不能用原先的去重逻辑
- 去重的逻辑：通过 Set 集合记录本层的元素是否使用过

![700](assets/Pasted%20image%2020240216222807.png)

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();

    public List<List<Integer>> findSubsequences(int[] nums) {
        backTracing(nums, 0);
        return res;
    }

    public void backTracing(int[] nums, int start) {
        HashSet<Integer> set = new HashSet<>();
        for (int i = start; i < nums.length; i++) {
            // 去重
            if (set.contains(nums[i])) {
                continue;
            }
            // 保证递增
            if (start > 0 && nums[i] < nums[start - 1]) {
                continue;
            }
            path.add(nums[i]);
            set.add(nums[i]);
            // 至少两个元素
            if (path.size() >= 2) {
                res.add(new ArrayList(path));
            }
            backTracing(nums, i + 1);
            path.remove(path.size() - 1);
        }
    }
}
```

## 重新安排行程

#todo 

## ---------- 排列

## 全排列

[46. 全排列 - 力扣（LeetCode）](https://leetcode.cn/problems/permutations/description/)

给定一个**不含重复**数字的数组 `nums` ，返回其 所有可能的全排列 。你可以 按**任意顺序** 返回答案。

> 示例：
> - 输入: `nums = [1,2,3]`
> - 输出：`[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]`
>  
> 说明：
> - `1 <= nums.length <= 6`
> - `-10 <= nums[i] <= 10`
> - `nums` 中的所有整数 互不相同

---

回溯法

- 本题是不需要 start 索引的，因为排列是有序的，也就是说 `[1,2]` 和 `[2,1]` 是两个集合，这和之前分析的子集以及组合所不同的地方。所以每次 for 循环，i 都要从 0 开始
- 同一个 path 上不能有重复的元素，去重的逻辑：
	- Set 集合复杂了，不推荐
	- `boolean[] used` 查询最快
	- List 接口本身的 `contains()` 方法

![](assets/Pasted%20image%2020240219124408.png)

used 数组去重：

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    boolean[] used;

    public List<List<Integer>> permute(int[] nums) {
        used = new boolean[nums.length];
        backTracing(nums);
        return res;
    }

    public void backTracing(int[] nums) {
        if (path.size() == nums.length) {
            res.add(new ArrayList(path));
        }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) {
                continue;
            }
            path.add(nums[i]);
            used[i] = true;
            backTracing(nums);
            path.remove(path.size() - 1);
            used[i] = false;
        }
    }
}
```

## 全排列II

> 题目：[47. 全排列 II - 力扣（LeetCode）](https://leetcode.cn/problems/permutations-ii/solutions/417937/quan-pai-lie-ii-by-leetcode-solution/)

给定一个可包含重复数字的序列 `nums` ，按**任意顺序** 返回所有不重复的全排列。

> 示例：
> - 输入: `nums = [1,1,2]`
> - 输出：`[[1,1,2],[1,2,1],[2,1,1]]`
>  
> 说明：
> - `1 <= nums.length <= 8`
> - `-10 <= nums[i] <= 10`

---

回溯法

- 去重版全排列
- 不排序去重的逻辑：
	- Set 集合同层去重（for）
	- `boolean[] used` 纵向去重
- 排序去重的逻辑：（效率更高）
	- 相邻节点判断

不排序去重 #Boer 

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    boolean[] used;

    public List<List<Integer>> permuteUnique(int[] nums) {
        used = new boolean[nums.length];
        backTracing(nums);
        return res;
    }

    public void backTracing(int[] nums) {
        if (path.size() == nums.length) {
            res.add(new ArrayList(path));
        }
        Set<Integer> set = new HashSet<>();
        for (int i = 0; i < nums.length; i++) {
            if (set.contains(nums[i])) {
                continue;
            }
            if (used[i]) {
                continue;
            }
            path.add(nums[i]);
            used[i] = true;
            set.add(nums[i]);
            backTracing(nums);
            path.remove(path.size() - 1);
            used[i] = false;
        }
    }
}
```

排序去重

![](assets/Pasted%20image%2020240221215032.png)

```java
class Solution {
    //存放结果
    List<List<Integer>> result = new ArrayList<>();
    //暂存结果
    List<Integer> path = new ArrayList<>();

    public List<List<Integer>> permuteUnique(int[] nums) {
        boolean[] used = new boolean[nums.length];
        Arrays.fill(used, false);
        Arrays.sort(nums);
        backTrack(nums, used);
        return result;
    }

    private void backTrack(int[] nums, boolean[] used) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            // used[i - 1] == true，说明同⼀树⽀nums[i - 1]使⽤过
            // used[i - 1] == false，说明同⼀树层nums[i - 1]使⽤过
            // 如果同⼀树层nums[i - 1]使⽤过则直接跳过
            if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == false) {
                continue;
            }
            //如果同⼀树⽀nums[i]没使⽤过开始处理
            if (used[i] == false) {
                used[i] = true;//标记同⼀树⽀nums[i]使⽤过，防止同一树枝重复使用
                path.add(nums[i]);
                backTrack(nums, used);
                path.remove(path.size() - 1);//回溯，说明同⼀树层nums[i]使⽤过，防止下一树层重复
                used[i] = false;//回溯
            }
        }
    }
}
```

## ---------- 棋盘问题

## N皇后

#todo 

## 解数独

#todo 

# 贪心算法

贪心算法一般分为如下四步：

- 将问题分解为若干个子问题
- 找出适合的贪心策略
- 求解每一个子问题的最优解
- 将局部最优解堆叠成全局最优解

## 分发饼干

> [455. 分发饼干 - 力扣（LeetCode）](https://leetcode.cn/problems/assign-cookies/description/)

假设你是一位很棒的家长，想要给你的孩子们一些小饼干。但是，每个孩子最多只能给一块饼干。

对每个孩子 `i`，都有一个胃口值 `g[i]`，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干 `j`，都有一个尺寸 `s[j]` 。如果 `s[j] >= g[i]`，我们可以将这个饼干 `j` 分配给孩子 `i` ，这个孩子会得到满足。你的目标是尽可能满足越多数量的孩子，并输出这个最大数值。

> 示例：
> - 输入: `g = [1,2,3], s = [1,1]`
> - 输出：`1`
> - 解释: 你有三个孩子和两块小饼干，3个孩子的胃口值分别是：1,2,3。虽然你有两块小饼干，由于他们的尺寸都是1，你只能让胃口值是1的孩子满足。所以你应该输出1。
>  
> 说明：
> - `1 <= g.length <= 3 * 104`
> - `0 <= s.length <= 3 * 104`
> - `1 <= g[i], s[j] <= 231 - 1`

---
> 2024年2月26日20:36:14

解法：贪心，
- `s[j] >= g[i]` 就能喂饱，一块饼干喂得胃口越大越好
- 1）大饼干大胃口
- 2）小饼干小胃口

大饼干大胃口

```java
class Solution {
    public int findContentChildren(int[] g, int[] s) {
        // 大饼干优先给大胃口
        Arrays.sort(g);
        Arrays.sort(s);
        int max = 0;
        for (int i = g.length - 1, j = s.length - 1; i >= 0 && j >= 0;) {
            if (s[j] >= g[i]) {
                max++;
                j--;
            }
            i--;
        }
        return max;
    }
}
```

## 摆动序列 #rep

[376. 摆动序列 - 力扣（LeetCode）](https://leetcode.cn/problems/wiggle-subsequence/description/)

如果连续数字之间的差严格地在正数和负数之间交替，则数字序列称为 **摆动序列 。**第一个差（如果存在的话）可能是正数或负数。仅有一个元素或者含两个不等元素的序列也视作摆动序列。
- 例如， `[1, 7, 4, 9, 2, 5]` 是一个 **摆动序列** ，因为差值 `(6, -3, 5, -7, 3)` 是正负交替出现的。
- 相反，`[1, 4, 7, 2, 5]` 和 `[1, 7, 4, 5, 5]` 不是摆动序列，第一个序列是因为它的前两个差值都是正数，第二个序列是因为它的最后一个差值为零。

**子序列** 可以通过从原始序列中删除一些（也可以不删除）元素来获得，剩下的元素保持其原始顺序。

给你一个整数数组 `nums` ，返回 `nums` 中作为 **摆动序列** 的 **最长子序列的长度** 。

> 示例：
> - 输入: `[1,17,5,10,13,15,10,5,16,8]`
> - 输出：`7`
> - 解释: 这个序列包含几个长度为 7 摆动序列。其中一个是 `[1, 17, 10, 13, 10, 16, 8]` ，各元素之间的差值为 `(16, -7, 3, -3, 6, -8)` 。
>  
> 说明：
> - `1 <= nums.length <= 1000`
> - `0 <= nums[i] <= 1000`

---
> 2024年2月27日13:31:02

贪心算法：
- 局部最优：删除单调坡度上的节点（不包括单调坡度两端的节点），那么这个坡度就可以有两个局部峰值。
- 整体最优：整个序列有最多的局部峰值，从而达到最长摆动序列。
- 注意：
	- `nums.length == 1`，最长子序列的长度=1
	- `[x, x]`，最长子序列的长度=1

![](assets/Pasted%20image%2020240227130419.png)

```java
class Solution {
    public int wiggleMaxLength(int[] nums) {
        if (nums.length == 1) {
            return 1;
        }
        // 两个元素相等，最长子序列的长度也是1
        int res = 1;
        int preDiff = 0;
        int curDiff = 0;
        for (int i = 1; i < nums.length; i++) {
            curDiff = nums[i] - nums[i - 1];
            // preDiff只有初值才会是0
            if (curDiff > 0 && preDiff <= 0 || curDiff < 0 && preDiff >= 0) {
                preDiff = curDiff;
                res++;
            }
        }
        return res;
    }
}
```

---
动态规划 #todo


## 跳跃游戏 #rep

[55. 跳跃游戏 - 力扣（LeetCode）](https://leetcode.cn/problems/jump-game/description/)

给你一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标，如果可以，返回 `true` ；否则，返回 `false` 。

> 示例：
> - 输入: `nums = [2,3,1,1,4]`
> - 输出：`true`
> - 解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。
>  
> 说明：
> - `1 <= nums.length <= 10^4`
> - `0 <= nums[i] <= 10^5`

---
贪心
- 局部最优解：每次取最大跳跃步数（取最大覆盖范围），
- 整体最优解：最后得到整体最大覆盖范围，看是否能到终点。

![](assets/Pasted%20image%2020240301203847.png)

```java
class Solution {
    public boolean canJump(int[] nums) {
        if (nums.length == 1) {
            return true;
        }
        //覆盖范围, 初始覆盖范围应该是0，因为下面的迭代是从下标0开始的
        int coverRange = 0;
        //在覆盖范围内更新最大的覆盖范围
        for (int i = 0; i <= coverRange; i++) {
            coverRange = Math.max(coverRange, i + nums[i]);
            if (coverRange >= nums.length - 1) {
                return true;
            }
        }
        return false;
    }
}
```

## 跳跃游戏II #rep

[45. 跳跃游戏 II - 力扣（LeetCode）](https://leetcode.cn/problems/jump-game-ii/description/)

给定一个长度为 `n` 的 **0 索引**整数数组 `nums`。初始位置为 `nums[0]`。

每个元素 `nums[i]` 表示从索引 `i` 向前跳转的最大长度。换句话说，如果你在 `nums[i]` 处，你可以跳转到任意 `nums[i + j]` 处:

- `0 <= j <= nums[i]` 
- `i + j < n`

返回到达 `nums[n - 1]` 的最小跳跃次数。生成的测试用例可以到达 `nums[n - 1]`。

> 示例：
> - 输入: `nums = [2,3,1,1,4]`
> - 输出：`2`
> - 解释：跳到最后一个位置的最小跳跃数是 2。从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
>  
> 说明：
> - `1 <= nums.length <= 10^4`
> - `0 <= nums[i] <= 1000`
> - 题目保证可以到达 `nums[n-1]`

---
> 2024年3月2日22:12:17

贪心：
- 要统计两个覆盖范围，**当前这一步的最大覆盖**和**下一步最大覆盖**。如果移动下标达到了当前这一步的最大覆盖最远距离了，还没有到终点的话，那么就必须再走一步来增加覆盖范围，直到覆盖范围覆盖了终点。
- max 表示当前一步的最大覆盖，temp 表示下一步最大覆盖
- 在 max 的范围内移动时，不断更新 temp，temp 越大需要的步数越少
- 终止条件是 `max < nums.length-1` ，因为当 `max = nums.length-1` 已经覆盖到终点了，不需要下一步了

```java
class Solution {
    public int jump(int[] nums) {
        int res = 0;
        int max = 0;
        int temp = 0;
        for (int i = 0; i <= max && max < nums.length-1; i++) {
            temp = Math.max(i + nums[i], temp);
            if (i == max) {
                res++;
                max = temp;
            }
        }
        return res;
    }
}
```

## K 次取反后最大化的数组和 #rep

给你一个整数数组 nums 和一个整数 k ，按以下方法修改该数组：

选择某个下标 i 并将 nums[i] 替换为 -nums[i] 。
重复这个过程恰好 k 次。可以多次选择同一个下标 i 。

以这种方式修改数组后，返回数组 可能的最大和 。

> 示例：
> - 输入: `nums = [2,-3,-1,5,-4], k = 2`
> - 输出：`13`
> - 解释：选择下标 `(1, 4)` ，nums 变为 `[2,3,-1,5,4]`
>  
> 说明：
> - `1 <= nums.length <= 10^4`
> - `-100 <= nums[i] <= 100`
> - `1 <= k <= 10^4`

---
> 2024年3月3日17:46:55

贪心

```java
class Solution {
    public int largestSumAfterKNegations(int[] nums, int k) {
        // 绝对值从大到小排序
        nums = IntStream.of(nums) // Arrays.stream() 也可以
                .boxed() // 装箱为Integer
                .sorted((o1, o2) -> Math.abs(o2) - Math.abs(o1))
                .mapToInt(Integer::intValue) // 拆箱为int
                .toArray();

        int len = nums.length;
        for (int i = 0; i < len && k > 0; i++) {
            // 将负数都变正
            if (nums[i] < 0) {
                nums[i] = -nums[i];
                k--;
            }
        }

        // k>0，不断反复转变数值最小的元素，将k用完
        if (k % 2 == 1) {
            nums[len - 1] = -nums[len - 1];
        }

        return Arrays.stream(nums).sum();
    }
}
```

## 加油站 #rep

> https://leetcode.cn/problems/gas-station/description/

在一条环路上有 `n` 个加油站，其中第 `i` 个加油站有汽油 `gas[i]` 升。

你有一辆油箱容量无限的的汽车，从第 `i` 个加油站开往第 `i+1` 个加油站需要消耗汽油 `cost[i]` 升。你从其中的一个加油站出发，开始时油箱为空。

给定两个整数数组 `gas` 和 `cost` ，如果你可以按顺序绕环路行驶一周，则返回出发时加油站的编号，否则返回 `-1` 。如果存在解，则 **保证** 它是 **唯一** 的。

---
> 2024年3月6日17:35:21

贪心：
- 每个加油站的剩余量 `rest[i]为 gas[i] - cost[i]`
- i 从 0 开始累加 `rest[i]`，和记为 `curSum`，一旦 `curSum<0` ，说明 `[0, i]` **区间都不能**作为起始位置

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int curSum = 0;
        int sum = 0;
        int res = 0;
        for (int i = 0; i < gas.length; i++) {
            int rest = gas[i] - cost[i];
            curSum += rest;
            sum += rest;
            if (curSum < 0) {
                res = (i + 1) % gas.length;
                curSum = 0;
            }
        }
        if (sum < 0)
            return -1;
        return res;
    }
}
```

## 分发糖果 #rep

> [135. 分发糖果 - 力扣（LeetCode）](https://leetcode.cn/problems/candy/description/)

`n` 个孩子站成一排。给你一个整数数组 `ratings` 表示每个孩子的评分。

你需要按照以下要求，给这些孩子分发糖果：

- 每个孩子至少分配到 `1` 个糖果。
- 相邻两个孩子评分更高的孩子会获得更多的糖果。

请你给每个孩子分发糖果，计算并返回需要准备的 **最少糖果数目** 。

> 示例：
> - 输入: `ratings = [1,0,2]`
> - 输出：`5`
> - 解释: 你可以分别给第一个、第二个、第三个孩子分发 2、1、2 颗糖果

---
> 2024年3月7日23:34:03

贪心：
- 从左到右遍历，只比较右边孩子评分比左边大的情况。
- 从右到左遍历，只比较左边孩子评分比右边大的情况。

```java
class Solution {
    public int candy(int[] ratings) {
        int len = ratings.length;
        int[] candys = new int[len];
        candys[0] = 1;
        for (int i = 1; i < len; i++) {
            candys[i] = (ratings[i - 1] < ratings[i]) ? candys[i - 1] + 1 : 1;
        }

        for (int i = len - 2; i >= 0; i--) {
            if (ratings[i] > ratings[i + 1]) {
                candys[i] = Math.max(candys[i], candys[i + 1] + 1);
            }
        }

        int res = 0;
        for (int i : candys) {
            res += i;
        }
        return res;
    }
}
```

## 柠檬水找零

> [860. 柠檬水找零 - 力扣（LeetCode）](https://leetcode.cn/problems/lemonade-change/description/)

在柠檬水摊上，每一杯柠檬水的售价为 `5` 美元。顾客排队购买你的产品，（按账单 `bills` 支付的顺序）一次购买一杯。

每位顾客只买一杯柠檬水，然后向你付 `5` 美元、`10` 美元或 `20` 美元。你必须给每个顾客正确找零，也就是说净交易是每位顾客向你支付 `5` 美元。

注意，一开始你手头没有任何零钱。

给你一个整数数组 `bills` ，其中 `bills[i]` 是第 `i` 位顾客付的账。如果你能给每位顾客正确找零，返回 `true` ，否则返回 `false` 。

> 示例：
> - 输入: `bills = [5,5,10,10,20]`
> - 输出：`false`
> - 解释: 你可以分别给第一个、第二个、第三个孩子分发 2、1、2 颗糖果
>   
> 提示：
> - `bills[i]` 不是 `5` 就是 `10` 或是 `20`

---
> 2024年3月8日19:22:27

贪心：
- 局部最优：遇到账单 20，优先消耗美元 10，完成本次找零。
- 全局最优：完成全部账单的找零

```java
class Solution {
    public boolean lemonadeChange(int[] bills) {
        int[] rec = new int[2];
        for (int i = 0; i < bills.length; i++) {
            if (bills[i] == 5) {
                rec[0]++;
            } else if (bills[i] == 10) {
                rec[0]--;
                rec[1]++;
            } else {
                // 先用10
                if (rec[1] > 0) {
                    rec[0]--;
                    rec[1]--;
                } else {
                    rec[0] -= 3;
                }
            }
            if (rec[0] < 0 || rec[1] < 0) {
                return false;
            }
        }
        return true;
    }
}
```

## 根据身高重建队列 #rep

> [406. 根据身高重建队列 - 力扣（LeetCode）](https://leetcode.cn/problems/queue-reconstruction-by-height/description/)

假设有打乱顺序的一群人站成一个队列，数组 `people` 表示队列中一些人的属性（不一定按顺序）。每个 `people[i] = [hi, ki]` 表示第 `i` 个人的身高为 `hi` ，前面 **正好** 有 `ki` 个身高大于或等于 `hi` 的人。

请你重新构造并返回输入数组 `people` 所表示的队列。返回的队列应该格式化为数组 `queue` ，其中 `queue[j] = [hj, kj]` 是队列中第 `j` 个人的属性（`queue[0]` 是排在队列前面的人）。

> 示例：
> - 输入: `people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]`
> - 输出：`[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]`
>   
> 提示：
> - 题目数据确保队列可以被重建

---

贪心：

- 在按照身高从大到小排序后
- 局部最优：优先按身高高的 people 的 k 来插入。插入操作过后的 people 满足队列属性
- 全局最优：最后都做完插入操作，整个队列满足题目队列属性

![](assets/Pasted%20image%2020240309222528.png)

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        // 第一维从大到小，第二维从小到大
        Arrays.sort(people, (a, b) -> {
            if (a[0] == b[0]){
                return a[1] - b[1];
            }
            return b[0] - a[0];
        });

        LinkedList<int[]> que = new LinkedList<>();

        for (int[] p : people) {
            // 将p插入p[1]位置，p[1]有元素了就集体后移
            que.add(p[1],p);   
        }
        return que.toArray(new int[people.length][]);
    }
}
```

## 用最少数量的箭引爆气球 #rep

> [452. 用最少数量的箭引爆气球 - 力扣（LeetCode）](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/description/)

贪心：

- 气球起始位置从小到大排序
- 至少要一根箭
- 如果气球重叠了，更新最小右边界
- 气球左边界>最小右边界，说明不重叠，加一根

![|500](assets/Pasted%20image%2020240311204459.png)

```java
public int findMinArrowShots(int[][] points) {
	// xstart从小到大排序
	Arrays.sort(points, (a, b) -> Integer.compare(a[0], b[0]));

	int count = 1;
	for (int i = 1; i < points.length; i++) {
		if (points[i - 1][1] < points[i][0]) {
			// 气球i和气球i-1无重叠
			count++;
		} else {
			// 重叠，更新重叠最小右边界
			points[i][1] = Math.min(points[i - 1][1], points[i][1]);
		}
	}
	return count;
}
```

## 无重叠区间 #rep

> [435. 无重叠区间 - 力扣（LeetCode）](https://leetcode.cn/problems/non-overlapping-intervals/description/)

贪心：
- 按照左边界、有边界排序都可以
- 两个区间重叠（包含、交叉）后，怎么**移除区间**很关键！
	- 物理移除，需要把数组转为 LinkedList
	- √ 后面一个区间的右边界更新为：两个区间的**最小右边界**

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        Arrays.sort(intervals, (a,b)-> {
            return Integer.compare(a[0],b[0]);
        });
        int count = 0;
        for(int i = 1;i < intervals.length;i++){
            if(intervals[i][0] < intervals[i-1][1]){
                intervals[i][1] = Math.min(intervals[i - 1][1], intervals[i][1]);
                count++;
            }
        }
        return count;
    }
}
```

## 划分字母区间 #rep

> [划分字母区间](https://leetcode.cn/problems/partition-labels/)

贪心：

- 统计每一个字符最后出现的位置
- 从头遍历字符，并更新字符的最远出现下标，如果找到字符最远出现位置下标和当前下标相等了，则找到了分割点

![](assets/Pasted%20image%2020240312195037.png)

```java
class Solution {
    public List<Integer> partitionLabels(String s) {
        List<Integer> res = new LinkedList<>();
        int[] rec = new int[26];
        char[] chars = s.toCharArray();
        // 统计每一个字符最后出现的位置
        for (int i = 0; i < chars.length; i++) {
            rec[chars[i] - 'a'] = i;
        }
        int idx = 0;
        int prevIdx = -1;
        for (int i = 0; i < chars.length; i++) {
            idx = Math.max(idx, rec[chars[i] - 'a']);
            if (i == idx) {
                res.add(i - prevIdx);
                prevIdx = i;
            }
        }
        return res;
    }
}
```

## 合并区间

[56. 合并区间 - 力扣（LeetCode）](https://leetcode.cn/problems/merge-intervals/description/)

贪心：

- 左边界从小到大排序
- 重叠以后更新 最大右边界
- 最后一组容易遗漏！循环条件是 `i <= intervals.length`，`i == intervals.length` 时已经遍历完所有区间，要把最后一组区间合并，加入到结果中

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        // 左边界从小到大排序
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

        // 暂存结果
        LinkedList<int[]> resList = new LinkedList<>();
        int startIdx = 0;
        for (int i = 1; i <= intervals.length; i++) {
            if (i == intervals.length) {
            } else if (intervals[i - 1][1] >= intervals[i][0]) {
                // 重叠
                intervals[i][1] = Math.max(intervals[i - 1][1], intervals[i][1]);
                continue;
            }
            int[] ints = { intervals[startIdx][0], intervals[i - 1][1] };
            startIdx = i;
            resList.add(ints);
        }

        int[][] res = new int[resList.size()][2];
        int i = 0;
        for (int[] ints : resList) {
            res[i++] = ints;
        }
        return res;
    }
}
```


## 单调递增的数字 #rep

[738. 单调递增的数字 - 力扣（LeetCode）](https://leetcode.cn/problems/monotone-increasing-digits/description/)

贪心：

- 一旦出现 `strNum[i - 1] > strNum[i]`，让 `strNum[i - 1]` 减一，`strNum[i]` 赋值 9，例如 `98-->89`
- 某一位变成9，后面的位数都得是9，例如 `1000->999`
	- 用 start 记录 9 开始的位置

```java
class Solution {
    public int monotoneIncreasingDigits(int n) {
        String str = String.valueOf(n);
        char[] chars = str.toCharArray();
        int start = chars.length;
        for (int i = chars.length - 1; i > 0; i--) {
            if (chars[i - 1] > chars[i]) {
                chars[i - 1]--;
                start = i;
            }
        }
        for (int i = start; i < chars.length; i++) {
            chars[i] = '9';
        }
        return Integer.parseInt(new String(chars));
    }
}
```

# 动态规划

动态规划，英文：Dynamic Programming，简称DP，如果某一问题有很多**重叠子问题**，使用动态规划是最有效的。

所以动态规划中==每一个状态一定是由上一个状态推导出来的==，而贪心没有状态推导，而是从局部直接选最优的，

dp五部曲：
1. 确定 dp **数组**（dp table）以及**下标**的含义
2. 确定**递推**公式
3. dp 数组如何**初始化**
4. 确定**遍历顺序**
5. 举例推导 dp 数组

dp如何 debug？
- 最好的方式就是打印 dp 数组

贪心和 dp 的区别：
- 贪心是 dp 的一种特殊情况，贪心只依赖于前一个状态的结果，所以不需要一个数组记录历史状态
- 例如 `dp[i]` 只依赖于 `dp[i-1]`，那直接用变量记录 `dp[i-1]` 就好了


## 斐波那契数列

[509. 斐波那契数 - 力扣（LeetCode）](https://leetcode.cn/problems/fibonacci-number/description/)

dp
1. `dp[i]` 定义：第 i 个数的斐波那契数值
2. 递推公式：题目已给
3. 初始化：题目已给
4. 遍历顺序：从前向后
5. 模拟

迭代
```java
class Solution {
    public int fib(int n) {
        if (n < 2)
            return n;
        int[] dp = new int[2];
        dp[0] = 0;
        dp[1] = 1;
        for (int i = 2; i <= n; i++) {
            int sum = dp[0] + dp[1];
            dp[0] = dp[1];
            dp[1] = sum;
        }
        return dp[1];
    }
}
```

- 时间复杂度：$O(n)$
- 空间复杂度：$O(n)$

递归
```java
class Solution {
    public int fib(int n) {
        if (n < 2) {
            return n;
        }
        return fib(n - 1) + fib(n - 2);
    }
}
```

- 时间复杂度：$O(2^n)$
- 空间复杂度：$O(n)$

## 爬楼梯

[70. 爬楼梯 - 力扣（LeetCode）](https://leetcode.cn/problems/climbing-stairs/description/)

dp 思路同斐波那契数列，初始化不同

迭代
```java
class Solution {
    public int climbStairs(int n) {
        if (n <= 2)
            return n;
        int[] dp = new int[2];
        dp[0] = 1;
        dp[1] = 2;
        for (int i = 3; i <= n; i++) {
            int sum = dp[0] + dp[1];
            dp[0] = dp[1];
            dp[1] = sum;
        }
        return dp[1];
    }
}
```

## 使用最小费用爬楼梯

[746. 使用最小花费爬楼梯 - 力扣（LeetCode）](https://leetcode.cn/problems/min-cost-climbing-stairs/description/)

dp：

1. `dp[i]` 定义：到达第 i 台阶所花费的最少体力为 `dp[i]`，目标是 `dp[cost.length]`
2. 递推公式：`dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2])`
3. 初始化：`dp[0] = 0，dp[1] = 0`
4. 遍历顺序：从前向后
5. 模拟

迭代

```java
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        int[] dp = new int[2];
        dp[0] = 0;
        dp[1] = 0;
        for (int i = 2; i <= cost.length; i++) {
            int min = Math.min(dp[0] + cost[i - 2], dp[1] + cost[i - 1]);
            dp[0] = dp[1];
            dp[1] = min;
        }
        return dp[1];
    }
}	
```

## 不同路径 #中等 #笔试

[62. 不同路径 - 力扣（LeetCode）](https://leetcode.cn/problems/unique-paths/description/)

dp：

1. `dp[i][j]` 定义：从 `(0,0)` 出发，到 `(i,j)` 有 `dp[i][j]` 条不同的路径
2. 递推公式：`dp[i][j] = dp[i - 1][j] + dp[i][j - 1]`
3. 初始化：`dp[i][0] = 1, dp[0][j] = 1`
4. 遍历顺序：从左到右一层层遍历
5. 模拟

迭代

```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];
        // dp[0][0] = 1;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (i == 0 || j == 0) {
                    dp[i][j] = 1;
                    continue;
                }
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m-1][n-1];
    }
}
```

## 不同路径 II #中等

[63. 不同路径 II - 力扣（LeetCode）](https://leetcode.cn/problems/unique-paths-ii/description/)

dp：
1. `dp[i][j]` 定义：从 `(0,0)` 出发，到 `(i,j)` 有 `dp[i][j]` 条不同的路径
2. 递推公式：
	1. `(i,j)` 有障碍，`dp[i][j] = 0`。
	2. 否则 `dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
3. 初始化：
	1. `(i,0)` 和 `(0,j)` 边有障碍，障碍之后：`dp[i][0] = 0, dp[0][j] = 0`。
	2. 否则 `dp[i][0] = 1, dp[0][j] = 1`
4. 遍历顺序：从左到右一层层遍历
5. 模拟

迭代
```java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int m = obstacleGrid.length;
        int n = obstacleGrid[0].length;
        int[][] dp = new int[m][n];
        // 初始化
        for (int i = 0; i < m && obstacleGrid[i][0] == 0; i++) {
            dp[i][0] = 1;
        }
        for (int j = 0; j < n && obstacleGrid[0][j] == 0; j++) {
            dp[0][j] = 1;
        }
        // 递推
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = (obstacleGrid[i][j] == 0) ? dp[i - 1][j] + dp[i][j - 1] : 0;
            }
        }
        return dp[m - 1][n - 1];
    }
}
```

## 整数拆分 #中等 #rep 

[343. 整数拆分 - 力扣（LeetCode）](https://leetcode.cn/problems/integer-break/)

dp：
1. `dp[i]` 定义：分拆数字 i，可以得到的最大乘积为 `dp[i]`
2. 递推公式：`dp[i] = max({dp[i], (i - j) * j, dp[i - j] * j})`
	- j 为从 1 遍历到 `i/2`（记不住的话遍历到 `i-1` 也行）
	- `j * (i - j)` 是单纯的把整数拆分为两个数相乘，而 `j * dp[i - j]` 是拆分成两个以及两个以上的个数相乘
3. 初始化：`dp[2] = 1`
4. 遍历顺序：从前向后遍历
5. 模拟

迭代：
```java
class Solution {
    public int integerBreak(int n) {
        int[] dp = new int[n + 1];
        dp[2] = 1;
        for (int i = 3; i <= n; i++) {
            for (int j = 1; j <= i/2 ; j++) {
                int max = Math.max(j * (i - j), j * dp[i - j]);
                dp[i] = Math.max(dp[i], max);
            }
        }
        return dp[n];
    }
}
```

## 不同的二叉搜索树 #中等 #rep

https://leetcode.cn/problems/unique-binary-search-trees/description/

分析：
- `dp[3]`，就是 元素 1 为头结点搜索树的数量 + 元素 2 为头结点搜索树的数量 + 元素 3 为头结点搜索树的数量
	- 元素 1 为头结点搜索树的数量 = 右子树有 2 个元素的搜索树数量 * 左子树有 0 个元素的搜索树数量
	- 元素 2 为头结点搜索树的数量 = 右子树有 1 个元素的搜索树数量 * 左子树有 1 个元素的搜索树数量
	- 元素 3 为头结点搜索树的数量 = 右子树有 0 个元素的搜索树数量 * 左子树有 2 个元素的搜索树数量
- 有 2 个元素的搜索树数量就是 `dp[2]`。有 1 个元素的搜索树数量就是 `dp[1]`。有0个元素的搜索树数量就是 `dp[0]`。
- 所以 `dp[3] = dp[2] * dp[0] + dp[1] * dp[1] + dp[0] * dp[2]`
![[Pasted image 20240527154118.png]]

dp：
1. `dp[i]` 定义：1 到 i 为节点组成的二叉搜索树的个数为 `dp[i]`
2. 递推公式：
	- `dp[i] += dp[以j为头结点左子树节点数量] * dp[以j为头结点右子树节点数量]`，j 相当于是头结点的元素，从 1 遍历到 i 为止。
	- 所以递推公式：`dp[i] += dp[j - 1] * dp[i - j]` ，`j-1` 是 j 为头结点左子树节点数量，`i-j` 是以 j 为头结点右子树节点数量
3. 初始化：`dp[0] = 1`
4. 遍历顺序：从前向后遍历
5. 模拟

迭代：
```java
class Solution {
    public int numTrees(int n) {
        int[] dp = new int[n + 1];
        dp[0] = 1;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= i; j++) {
                dp[i] += dp[j - 1] * dp[i - j];
            }
        }
        return dp[n];
    }
}
```

## ---------- 背包问题

背包问题分类：
![[assets/Pasted image 20240604113606.png]]

背包问题的 `dp[0]` 初始化：常见的是 0 和 1，根据递推公式的需要选择，不用太纠结它的意义

### ----- 01背包

### 01背包

https://kamacoder.com/problempage.php?pid=1046

有 n 件物品和一个最多能背重量为 w 的背包。第i件物品的重量是 `weight[i]`，得到的价值是 `value[i]` 。**每件物品只能用一次**，求解将哪些物品装入背包里物品价值总和最大。

分析：
- 暴力解法，每件物品只有选和不选两种状态，可以使用回溯，时间复杂度是指数级别的 $O(2^n)$。考虑采用动态规划

dp：
1. `dp[i][j]` 表示从下标为 `[0-i]` 的物品里任意取，放进容量为 j 的背包，价值总和最大是多少
2. 递推公式：可以有两个方向推出来 `dp[i][j]`
	- **不放物品 i**：物品 i 的重量大于背包j的重量，无法放进背包中，所以递推公式：`dp[i][j] = dp[i - 1][j]`
	- **放物品i**：`dp[i - 1][j - weight[i]]` 为背包容量为 `j - weight[i]` 的时候不放物品 i 的最大价值，那么 `dp[i - 1][j - weight[i]] + value[i]` 就是背包放物品 i 得到的最大价值。所以递归公式： `dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])`
		- 为什么要取 max 呢？因为 weight 大的物品 value 不一定大
1. 初始化：`
	- 背包容量 j 为 0 的话，`dp[i][0]=0`
	- 当 `j < weight[0]`的时候，`dp[0][j] = 0`。当`j >= weight[0]`时，`dp[0][j] = value[0]`
2. 遍历顺序：按 容量 j / 物品 i 遍历都可，最后求出 `dp[n-1][bagSize]` 就行
3. 模拟

初始化情况：
![[Pasted image 20240528143303.png]]
dp数组值：
![[Pasted image 20240528143444.png]]

```java
public static void main(String[] args) {
	Scanner sc = new Scanner(System.in);
	int num = sc.nextInt(); // 物品种类
	int bagSize = sc.nextInt(); // 背包容量
	int[] weights = new int[num]; // 物品所占空间
	int[] values = new int[num]; // 物品价值
	for (int i = 0; i < num; i++) {
		weights[i] = sc.nextInt();
	}
	for (int i = 0; i < num; i++) {
		values[i] = sc.nextInt();
	}

	int[][] dp = new int[num][bagSize + 1];
	// 初始化：容量为0时，最大价值为0
//        for (int i = 0; i < num; i++) {
//            dp[i][0] = 0;
//        }
	// 初始化：dp第一行
	for (int j = 0; j <= bagSize; j++) {
		if (weights[0] <= j) {
			dp[0][j] = values[0];
		}
	}
	// 递推
	for (int i = 1; i < num; i++) {
		for (int j = 1; j <= bagSize; j++) {
			if (j < weights[i]) { // j容量放不下物品i
				dp[i][j] = dp[i - 1][j];
			} else {
				dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weights[i]] + values[i]);
			}
		}
	}

	// 结果
	System.out.println(dp[num - 1][bagSize]);
}
```

### 分隔等和子集 #中等 #rep

https://leetcode.cn/problems/partition-equal-subset-sum/description/

分析：
- 问题可以转换为：集合里能否出现总和为 sum / 2 的子集
- 转换为 01背包问题：
	- 背包的体积为sum / 2
	- 背包里放的物品就是集合中的元素，价值和重量都是元素值
	- 背包如果正好装满，说明找到了总和为 sum / 2 的子集。
	- 背包中每一个元素是不可重复放入。

dp：
- `dp[j]` 表示： 容量为 j 的背包，所背的物品价值最大可以为 `dp[j]`
- 递推公式：`dp[j] = max(dp[j], dp[j - nums[i]] + nums[i])`
- 初始化：全0
- 遍历顺序：j 从大到小遍历

```java
public boolean canPartition(int[] nums) {
	int sum = 0;
	for (int num : nums) {
		sum += num;
	}
	if (sum % 2 != 0) return false;
	int bagSize = sum / 2;
	int[] dp = new int[bagSize + 1];
	// 遍历
	for (int i = 0; i < nums.length; i++) {
		for (int j = bagSize; j >= nums[i]; j--) {
			dp[j] = Math.max(dp[j], dp[j - nums[i]] + nums[i]);
		}
		if (dp[bagSize] == bagSize) return true;
	}
	return false;
}
```

### 最后一块石头的重量 ll #中等 #rep

https://leetcode.cn/problems/last-stone-weight-ii/description/

分析：
- 尽量让石头分成重量相同的两堆，相撞之后剩下的石头最小，**这样就化解成01背包问题了**
- 物品的重量为 `stones[i]`，物品的价值也为 `stones[i]`

dp：
- `dp[j]` 表示重量为 j 的背包，最多可以背最大重量为 `dp[j]`
- 递推公式：`dp[j] = max(dp[j], dp[j - stones[i]] + stones[i])`
- 初始化：全0
- 遍历顺序：i 从小到大，j 从大到小遍历
- 模拟：
	- 分成两堆石头，一堆石头的总重量是 `dp[target]`，另一堆就是 `sum - dp[target]`
	- 在计算 target 的时候，`target = sum / 2` 因为是向下取整，所以 `sum - dp[target]` 一定大于等于 `dp[target]`
	- 那么相撞之后剩下的最小石头重量就是 `(sum - dp[target]) - dp[target]`

```java
public int lastStoneWeightII(int[] stones) {
	int sum = 0;
	for (int stone : stones) {
		sum += stone;
	}
	int bagSize = sum / 2;
	int[] dp = new int[bagSize + 1];
	// 遍历dp
	for (int i = 0; i < stones.length; i++) {
		for (int j = bagSize; j >= stones[i]; j--) {
			dp[j] = Math.max(dp[j], dp[j - stones[i]] + stones[i]);
		}
	}
	return (sum - dp[bagSize]) - dp[bagSize];
}
```

### 目标和 #中等 #rep 

dp：
- `dp[j]` 表示：填满容量为 j 的背包，有 `dp[j]` 种方法
- 递推公式：

```java
public int findTargetSumWays(int[] nums, int target) {
	int sum = 0;
	for (int num : nums) {
		sum += num;
	}
	if (sum < Math.abs(target) || (sum + target) % 2 != 0) return 0;
	int bagSize = (sum + target) / 2;
	int[] dp = new int[bagSize + 1];
	// 初始化
	dp[0] = 1;
	// 遍历dp数组
	for (int i = 0; i < nums.length; i++) {
		for (int j = bagSize; j >= nums[i]; j--) {
			dp[j] += dp[j - nums[i]];
		}
	}
	return dp[bagSize];
}
```

### 一和零 #中等 #rep

[474. 一和零 - 力扣（LeetCode）](https://leetcode.cn/problems/ones-and-zeroes/submissions/536636963/)

分析：
- strs 数组里的元素就是物品，每个物品都是一个，字符串里的 0 和 1 的个数就是重量和价值
- m 和 n 相当于是一个两个维度的背包
- 01背包问题

dp：
- 1）`dp[i][j]`：最多有 i 个 0 和 j 个 1 的 strs 的**最大子集的大小**为 `dp[i][j]`
- 2）递推公式：当前放入物品（字符串）有 zeroNum 个 0，oneNum 个1，`dp[i][j]` 可以由前一个 strs 里的字符串推导出来，`dp[i][j] = max(dp[i][j], dp[i - zeroNum][j - oneNum] + 1);`
- 3）初始化：全 0
- 4）遍历顺序：没有设置多的维度来保存每次放入新物品后的 dp 数组，所以为了防止覆盖上次放入物品的结果，i 和 j 都从大到小遍历

```java
class Solution {
    public int findMaxForm(String[] strs, int m, int n) {
        int[][] dp = new int[m + 1][n + 1];
        for (String str : strs) {
            int zeroNum = 0;
            int oneNum = 0;
            for (int i = 0; i < str.length(); i++) {
                if (str.charAt(i) == '0') {
                    zeroNum++;
                } else {
                    oneNum++;
                }
            }
            for (int i = m; i >= zeroNum; i--) {
                for (int j = n; j >= oneNum; j--) {
                    dp[i][j] = Math.max(dp[i][j], dp[i - zeroNum][j - oneNum] + 1);
                }
            }
        }
        return dp[m][n];
    }
}
```

### ----- 完全背包

### 完全背包理论 

完全背包问题：有 N 件物品和一个最多能背重量为 W 的背包。第 i 件物品的重量是 `weight[i]`，得到的价值是 `value[i]` 。**每件物品都有无限个（也就是可以放入背包多次）**，求解将哪些物品装入背包里物品价值总和最大。

完全背包和 01 背包问题唯一不同的地方：**每种物品有无限件**。

dp五部曲唯一不同：遍历顺序，01背包内嵌的循环是从大到小遍历，为了保证每个物品仅被添加一次。而完全背包的物品是可以添加多次的，所以要**从小到大去遍历**

```java
public static void main(String[] args) {
	Scanner sc = new Scanner(System.in);
	int num = sc.nextInt();
	int bagSize = sc.nextInt();
	// 读取物品重量和价值
	int[] weights = new int[num];
	int[] values = new int[num];
	for (int i = 0; i < num; i++) {
		weights[i] = sc.nextInt();
		values[i] = sc.nextInt();
	}

	int[] dp = new int[bagSize + 1];
	for (int i = 0; i < num; i++) {
		for (int j = weights[i]; j <= bagSize; j++) {
			dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]);
		}
	}
	System.out.println(dp[bagSize]);
}
```

### 零钱兑换 ll #中等 #手撕 #rep

https://leetcode.cn/problems/coin-change-ii

分析：
- 货币数量无限，将 amount 看作背包容量，转换为完全背包问题。

dp：
- 1）`dp[j]`：凑成总金额 j 的货币**组合数**为 `dp[j]`
- 2）递推公式：`dp[j] += dp[j - coins[i]]`
- 3）`dp[0]=1` 还说明了一种情况：如果正好选了 `coins[i]` 后，也就是 `j-coins[i] == 0` 的情况表示这个硬币刚好能选，此时 `dp[0]` 为 1 表示只选 `coins[i]` 存在这样的一种选法
- 4）==**遍历顺序：外层遍历物品，内层遍历容量**==
- 5）模拟：以 `amount=5, coins=[1,5,2]` 为例，==注意这里模拟的是一维 dp 数组==

先遍历物品再遍历容量，`dp[j]` 求得的是**组合数**

| 物品/容量 | 0   | 1   | 2   | 3   | 4   | 5   |
| ----- | --- | --- | --- | --- | --- | --- |
| 1     | 1   | 1   | 1   | 1   | 1   | 1   |
| 5     | 1   | 1   | 1   | 1   | 1   | 2   |
| 2     | 1   | 1   | 2   | 2   | 3   | 4   |


先遍历容量再遍历物品，`dp[j]` 求得的是**排列数**

| 物品/容量 | 0   | 1   | 2              | 3                  | 4                          | 5   |
| ----- | --- | --- | -------------- | ------------------ | -------------------------- | --- |
| 1     | 1   | 1   | 1              | ==2(111, 12)==     | ==3==                      | 5   |
| 5     | 1   | 1   | 1              | 2(111, 12)         | 3                          | 6   |
| 2     | 1   | 1   | ==2(111, 12)== | ==3(111, 12, 21)== | 5(1111, 121, 112, 211, 22) | 9   |

```java
public int change(int amount, int[] coins) {
	int[] dp = new int[amount + 1];
	dp[0] = 1;
	for (int i = 0; i < coins.length; i++) {
		for (int j = coins[i]; j <= amount; j++) {
			dp[j] += dp[j - coins[i]];
		}
	}
	return dp[amount];
}
```

### 组合总和 Ⅳ #中等 

https://leetcode.cn/problems/combination-sum-iv/

分析：
- 题目描述说是求组合，但又说是可以元素相同顺序不同的组合算两个组合，**其实就是求排列！**，排列时强调顺序的！
- `nums[i]` 中的元素作为物品，target 作为容量，元素数量无限，转换为完全背包问题

dp：
- 1）`dp[j]` 表示 `target=j` 可能的组合数量
- 2）递推公式：`dp[j]+=dp[j-nums[i]]`
- 3）初始化：`dp[0]=1`
- 4）遍历顺序：外层遍历容量，内层遍历物品

```java
public int combinationSum4(int[] nums, int target) {
	int[] dp = new int[target + 1];
	dp[0] = 1;
	for (int j = 0; j <= target; j++) {
		for (int i = 0; i < nums.length; i++) {
			if (j >= nums[i])
				dp[j] += dp[j - nums[i]];
		}
	}
	return dp[target];
}
```

### 爬楼梯（进阶版）

https://kamacoder.com/problempage.php?pid=1067

分析：
- 1 阶，2 阶，.... m 阶就是物品，楼顶就是背包，每一阶可以重复使用，问题转换为完全背包问题

dp：
- `dp[j]`：到达 j 阶有多少种办法
- 递推公式为：`dp[i] += dp[i - j]`
- 初始化：`dp[0]=1`
- 遍历顺序：外层遍历容量，内层遍历物品

模拟：`m=3, n=2`

| 物品/容量 | 0   | 1   | 2   | 3   |
| ----- | --- | --- | --- | --- |
| 1     | 1   | 1   | 1   | 2   |
| 2     | 1   | 1   | 2   | 3   |

```java
public static void main(String[] args) {
	Scanner sc = new Scanner(System.in);
	int bagSize = sc.nextInt();
	int m = sc.nextInt();
	int[] dp = new int[bagSize + 1];
	dp[0] = 1;
	for (int j = 1; j <= bagSize; j++) {
		for (int i = 1; i <= m; i++) {
			if (j >= i) dp[j] += dp[j - i];
		}
	}
	System.out.println(dp[bagSize]);
}
```

### 零钱兑换 #中等 #rep

[322. 零钱兑换 - 力扣（LeetCode）](https://leetcode.cn/problems/coin-change/)

分析：
- 硬币无限，完全背包问题

dp：
- `dp[j]` 表示凑成 j 的**最少**硬币数量
- 递推公式：
	- `dp[j] = min(dp[j - coins[i]] + 1, dp[j])`
	- 如果 `dp[j - coins[i]]==Integer.MAX_VALUE`，说明 `j - coins[i]` 目前没法凑出的，这个时候 `dp[j]` 不用改
- 初始化：`dp[0] = 0`，`dp[j]`必须初始化为一个**最大的数**
- 遍历顺序：本题并**不强调集合是组合还是排列**，容量和物品的遍历顺序没有要求

模拟：`amount=5, coins=[2,5,1]`，模拟一维 dp 数组

| 物品/容量 | 0   | 1   | 2   | 3   | 4   | 5   |
| ----- | --- | --- | --- | --- | --- | --- |
| 2     | 0   | max | 1   | max | 2   | max |
| 5     | 0   | max | 1   | max | 2   | 1   |
| 1     | 0   | 1   | 1   | 2   | 2   | 1   |

```java
public int coinChange(int[] coins, int amount) {
	int[] dp = new int[amount + 1];
	// 初始化
	for (int j = 1; j <= amount; j++) {
		dp[j] = Integer.MAX_VALUE;
	}
	for (int i = 0; i < coins.length; i++) {
		for (int j = coins[i]; j <= amount; j++) {
			// 如果 dp[j - coins[i]] == Integer.MAX_VALUE，说明 j - coins[i] 暂时凑不出来
			if (dp[j - coins[i]] != Integer.MAX_VALUE) {
				dp[j] = Math.min(dp[j], dp[j - coins[i]] + 1);
			}
		}
	}
	return dp[amount] == Integer.MAX_VALUE ? -1 : dp[amount];
}
```

### 完全平方数 #中等

[279. 完全平方数 - 力扣（LeetCode）](https://leetcode.cn/problems/perfect-squares/description/)

分析：完全平方数就是物品（可以无限件使用），凑个正整数n就是背包，完全背包问题

dp：
- `dp[j]` 表示和为 j 的最小平方数个数
- 递推公式：`dp[j]=min(dp[j], dp[j-i^2]+1)`
- 初始化：
	- 第一种：`dp[0]=0`，非 0 下标初始为最大值
	- 第二种：`dp[i]=i`
- 遍历顺序：物品和容量哪个先遍历都可以

模拟：`n=13`

| 物品/容量 | 0   | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  | 11  | 12  | 13  |
| ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1     | 0   | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  | 11  | 12  | 13  |
| 4     | 0   | 1   | 2   | 3   | 1   | 2   | 3   | 4   | 2   | 3   | 4   | 5   | 3   | 4   |
| 9     | 0   | 1   | 2   | 3   | 1   | 2   | 3   | 4   | 2   | 1   | 2   | 3   | 4   | 2   |

```java
public int numSquares(int n) {
	int[] dp = new int[n + 1];
	for (int i = 0; i <= n; i++) {
		dp[i] = i;
	}
	for (int i = 2; i * i <= n; i++) {
		for (int j = i * i; j <= n; j++) {
			dp[j] = Math.min(dp[j], dp[j - i * i] + 1);
		}
	}
	return dp[n];
}
```

### 单词拆分 #中等 

[139. 单词拆分 - 力扣（LeetCode）](https://leetcode.cn/problems/word-break/description/)

dp：
- `dp[j]` 表示长度为 j 的单词是否能被拼接
- 递推公式：
	- `dp[j] = true`，不用遍历后续 word，肯定是 true
	- 否则，`dp[j] = dp[j - word长度)] && [(j-word长度)~j]的子串和word是否相等`
- 初始化：`dp[0]=true`，其他元素为 false
- 遍历顺序：排列问题，先遍历背包，再遍历物品

模拟：`s = apppenapp, list = [app, pen]`，dp 为一维数组

| 物品/容量 | 0   | a   | p   | p   | p   | e   | n   | a   | p   | p   |
| ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| app   | T   | F   | F   | T   | f   | f   | f   | f   | f   | f   |
| pen   | T   | F   | F   | T   | f   | f   | f   | f   | f   | f   |

```java
public boolean wordBreak(String s, List<String> wordDict) {
	boolean[] dp = new boolean[s.length() + 1];
	dp[0] = true;
	for (int j = 1; j <= s.length(); j++) {
		for (String word : wordDict) {
			if (j >= word.length()) {
				String subStr = s.substring(j - word.length(), j);
				boolean equals = word.equals(subStr);
				dp[j] = dp[j] || (dp[j - word.length()] && equals);
			}
		}
	}
	return dp[s.length()];
}
```

### ----- 多重背包

leetcode 暂无题目

## 打家劫舍 #中等

[198. 打家劫舍 - 力扣（LeetCode）](https://leetcode.cn/problems/house-robber/)

分析：当前房屋偷与不偷取决于 前一个房屋和前两个房屋是否被偷了。所以这里就更感觉到，当前状态和前面状态会有一种依赖关系，那么这种依赖关系都是动规的递推公式

dp：
- `dp[i]`：考虑下标i（包括i）以内的房屋，最多可以偷窃的金额为`dp[i]`
- 递推公式：`dp[i] = max(dp[i - 2] + nums[i], dp[i - 1])`
- 初始化：递推公式的基础就是 `dp[0]` 和 `dp[1]`，`dp[0]` 一定是 `nums[0]`，`dp[1]` 就是 `nums[0]` 和 `nums[1]` 的最大值
- 遍历顺序：从前到后遍历

```java
class Solution {
    public int rob(int[] nums) {
        if (nums.length == 1)
            return nums[0];
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);
        for (int i = 2; i < nums.length; i++) {
            dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
        }
        return dp[nums.length - 1];
    }
}
```

## 打家劫舍 ll #中等 #rep 

[213. 打家劫舍 II - 力扣（LeetCode）](https://leetcode.cn/problems/house-robber-ii/description/)

分析：
- 和打家劫舍的区别就在于第一个房间和最后一个房间不能都偷
- 考虑两种情况即可
	- 考虑包含首元素，不包含尾元素
	- 考虑包含尾元素，不包含首元素

```java
public int rob(int[] nums) {
	if (nums.length == 1)
		return nums[0];
	if (nums.length == 2)
		return Math.max(nums[0], nums[1]);

	// 不包含尾节点
	int[] dp = new int[nums.length];
	dp[0] = nums[0];
	dp[1] = Math.max(nums[0], nums[1]);
	for (int i = 2; i < nums.length - 1; i++) {
		dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
	}
	int temp = dp[nums.length - 2];

	// 不包含头节点
	dp[1] = nums[1];
	dp[2] = Math.max(nums[1], nums[2]);
	for (int i = 3; i < nums.length; i++) {
		dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
	}
	return Math.max(temp, dp[nums.length - 1]);
}
```

## 打家劫舍 lll #中等 #树形dp #rep

[337. 打家劫舍 III - 力扣（LeetCode）](https://leetcode.cn/problems/house-robber-iii/)

分析：对于树的话，首先就要想到遍历方式，前中后序（深度优先搜索）还是层序遍历（广度优先搜索）。**本题一定是要后序遍历，因为通过递归函数的返回值来做下一步计算**

1）暴力递归，超时

```java
class Solution {
    public int rob(TreeNode root) {
        if (root == null) {
            return 0;
        }
        // 不偷当前节点
        int max1 = rob(root.left) + rob(root.right);
        // 偷当前节点
        int max2 = root.val;
        if (root.left != null) {
            max2 += rob(root.left.left) + rob(root.left.right);
        }
        if (root.right != null) {
            max2 += rob(root.right.left) + rob(root.right.right);
        }
        return Math.max(max1, max2);
    }
}
```

- 时间复杂度：`O(n^2)`，这个时间复杂度不太标准，也不容易准确化，例如越往下的节点重复计算次数就越多
- 空间复杂度：`O(log n)`，算上递推系统栈的空间

> 我们计算了 root 的四个孙子（左右孩子的孩子）为头结点的子树的情况，又计算了 root 的左右孩子为头结点的子树的情况，计算左右孩子的时候其实又把孙子计算了一遍

2）记忆化递归

```java
class Solution {
    private Map<TreeNode, Integer> map = new HashMap<>();

    public int rob(TreeNode root) {
        if (root == null) {
            return 0;
        }
        if (map.get(root) != null) {
            return map.get(root);
        }
        // 不偷当前节点
        int max1 = rob(root.left) + rob(root.right);
        // 偷当前节点
        int max2 = root.val;
        if (root.left != null) {
            max2 += rob(root.left.left) + rob(root.left.right);
        }
        if (root.right != null) {
            max2 += rob(root.right.left) + rob(root.right.right);
        }
        map.put(root, Math.max(max1, max2));
        return Math.max(max1, max2);
    }
}
```

- 时间复杂度：`O(n)`
- 空间复杂度：`O(log n)`，算上递推系统栈的空间

3）动态规划

dp：
- `dp[i]`：长度为 2 的数组，下标为 0 记录不偷该节点所得到的的最大金钱，下标为 1 记录偷该节点所得到的的最大金钱。
- 确定终止条件：在遍历的过程中，如果遇到空节点的话，很明显，无论偷还是不偷都是 0，返回 `[0,0]`
- 递归顺序：首先明确的是使用后序遍历。 因为要通过递归函数的返回值来做下一步计算。通过递归左节点，得到左节点偷与不偷的金钱。通过递归右节点，得到右节点偷与不偷的金钱。
- 递推公式：
	- 如果是偷当前节点，那么左右孩子就不能偷，`val1 = cur->val + left[0] + right[0];` 
	- 如果不偷当前节点，那么左右孩子就可以偷，至于到底偷不偷一定是选一个最大的，所以：`val2 = max(left[0], left[1]) + max(right[0], right[1]);`

```java
class Solution {
    public int rob(TreeNode root) {
        int[] dp = robAction(root);
        return Math.max(dp[0], dp[1]);
    }

    public int[] robAction(TreeNode root) {
        int[] dp = new int[2];
        if (root == null) {
            return dp;
        }
        int[] left = robAction(root.left);
        int[] right = robAction(root.right);
        // 不偷当前节点
        dp[0] = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
        // 偷当前节点
        dp[1] = root.val + left[0] + right[0];
        return dp;
    }
}
```

- 时间复杂度：`O(n)`
- 空间复杂度：`O(log n)`，算上递推系统栈的空间

## 买卖股票的最佳时机

[121. 买卖股票的最佳时机 - 力扣（LeetCode）](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

1）贪心法 ✔

分析：
- 每天根据 前一天的最小买入价格 和 前一天能够获取的最大利润，决定今天是否卖出，并更新这两个值

> 先更新哪个都可以，如果先更新 min 会出现当日买进当日卖出的情况，res=0，对结果没影响，但意思不明确

```java
class Solution {
    public int maxProfit(int[] prices) {
        int len = prices.length;
        int[] dp = new int[len];
        dp[0] = 0; // 当前卖出的最大利润
        int min = prices[0]; //
        for (int i = 1; i < len; i++) {
            dp[i] = Math.max(dp[i - 1], prices[i] - min);
            min = Math.min(prices[i], min);
        }
        return dp[len - 1];
    }
}
```

2）动态规划

dp五部曲：
- dp数组定义
	- `dp[i][0]` 表示第 i 天持有股票所得最多现金
	- `dp[i][1]` 表示第 i 天不持有股票所得最多现金
- 递推公式：
	- 前 i-1 天买入 和 当天买入 选一个现金最多的 `dp[i][0] = max(dp[i - 1][0], -prices[i]);`
	- 前 i-1 天卖出 和 当天卖出 选一个现金最多的`dp[i][1] = max(dp[i - 1][1], prices[i] + dp[i - 1][0])`
- 初始化：`dp[0][0] -= prices[0]`，`dp[0][1] = 0`
- 遍历顺序：`dp[i]` 都是由 `dp[i - 1]` 推导出来的，那么一定是从前向后遍历
- 模拟：返回结果 `dp[prices.length-1][1]`

```java
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0) return 0;
        int length = prices.length;
        // dp[i][0]代表第i天持有股票的最大收益
        // dp[i][1]代表第i天不持有股票的最大收益
        int[][] dp = new int[length][2];
        int result = 0;
        dp[0][0] = -prices[0];
        dp[0][1] = 0;
        for (int i = 1; i < length; i++) {
            dp[i][0] = Math.max(dp[i - 1][0], -prices[i]);
            dp[i][1] = Math.max(dp[i - 1][0] + prices[i], dp[i - 1][1]);
        }
        return dp[length - 1][1];
    }
}
```

`dp[i]` 只是依赖于 `dp[i - 1]` 的状态，滚动数组优化空间

```java
class Solution {
    public int maxProfit(int[] prices) {
        int len = prices.length;
        int dp[][] = new int[2][2];
        
        dp[0][0] = - prices[0];
        dp[0][1] = 0;

        for (int i = 1; i < len; i++){
            dp[i % 2][0] = Math.max(dp[(i - 1) % 2][0], - prices[i]);
            dp[i % 2][1] = Math.max(dp[(i - 1) % 2][1], prices[i] + dp[(i - 1) % 2][0]);
        }
        return dp[(len - 1) % 2][1];
    }
}
```

## 买卖股票的最佳时机 II #中等 #rep

[122. 买卖股票的最佳时机 II - 力扣（LeetCode）](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/description/)

1）贪心 ✔️

分析：
- 股票可以当天买当天卖，且可以买卖多次
- **最终利润是可以分解的**，假如第 0 天买入，第 3 天卖出，那么利润为：`prices[3] - prices[0]`。相当于 `(prices[3] - prices[2]) + (prices[2] - prices[1]) + (prices[1] - prices[0])`
- 局部最优：收集**每天的正利润**
- 全局最优：求得最大利润

![600](assets/Pasted%20image%2020240229234241.png)

```java
class Solution {
    public int maxProfit(int[] prices) {
        int res = 0;
        for (int i = 0; i < prices.length - 1; i++) {
            int pro = prices[i + 1] - prices[i];
            if (pro > 0) {
                res += pro;
            }
        }
        return res;
    }
}
```

2）动态规划

dp：
- `dp[i][0]` 表示第 i 天持有股票所得现金。`dp[i][1]` 表示第 i 天不持有股票所得最多现金
- 递推公式：
	- `dp[i][0]` 可以由两个状态推出来
		- 第 i-1天就持有股票，保持现状，所得现金即：`dp[i - 1][0]`
		- 第 i 天买入股票，所得现金即：`dp[i - 1][1] - prices[i]`
	- `dp[i][1]` 可以由两个状态推出来
		- 第 i-1天就不持有股票，保持现状，所得现金即：`dp[i - 1][1]`
		- 第 i 天卖出股票，所得现金即：`prices[i] + dp[i - 1][0]`
- 初始化
- 遍历顺序

```java
class Solution {
    public int maxProfit(int[] prices) {
        int[][] dp = new int[prices.length][2];
        dp[0][0] = -prices[0];
        dp[0][1] = 0;
        for (int i = 1; i < prices.length; i++) {
            dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] - prices[i]);
            dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] + prices[i]);
        }
        return dp[prices.length - 1][1];
    }
}
```

## 买卖股票的最佳时机 lll #困难 #todo 

|          | 3   | 3   | 5   | 0   | 0   | 3   | 1   | 4   |
| -------- | --- | --- | --- | --- | --- | --- | --- | --- |
| 第一次持有股票  | -3  | -3  |     |     |     |     |     |     |
| 第一次不持有股票 | 0   |     |     |     |     |     |     |     |
| 第二次持有股票  | 0   |     |     |     |     |     |     |     |
| 第二次不持有股票 | 0   |     |     |     |     |     |     |     |

## 买卖股票的最佳时机 4 #困难 #todo 

## 买卖股票的最佳时机含冷冻期 #中等 #todo

https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown

## 买卖股票的最佳时机含手续费 #中等 #todo

https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/

## ---------- 子序列/子数组

序列不要求连续，数组要求连续

### 最长递增子序列 #中等 #手撕2 #rep

https://leetcode.cn/problems/longest-increasing-subsequence/description/

dp：
- `dp[i]` 表示 i 之前包括 i 的以 `nums[i]` 结尾的最长递增子序列的长度
- 递推公式：`dp[i]` 为 j 从 0 到 i-1 各个位置的最长升序子序列 + 1 的最大值。所以：`if (nums[i] > nums[j]) dp[i] = max(dp[i], dp[j] + 1);`
- 初始化：起始大小至少都是1
- 遍历顺序：i 从前向后，j 从前向后或者从后向前都可以
- 返回 dp 数组的最大值

|         | 10  | 9   | 2   | 5   | 3   | 7   | 101 | 18  |
| ------- | --- | --- | --- | --- | --- | --- | --- | --- |
| `dp[i]` | 1   | 1   | 1   | 2   | 2   | 3   | 4   | 4   |

> 本题 `dp[i]` 如果定义为0-i 的序列包含的最长递增子序列的长度，那么 `dp[i]` 和之前的状态没有直接关系，找不出递推公式

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] dp = new int[nums.length];
        Arrays.fill(dp, 1);
        int res = 1;
        for (int i = 1; i < nums.length; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            res = Math.max(res, dp[i]);
        }
        return res;
    }
}
```

- 时间复杂度：$O(N^2)$

### 最长连续递增序列 

[674. 最长连续递增序列 - 力扣（LeetCode）](https://leetcode.cn/problems/longest-continuous-increasing-subsequence/description/)

```java
class Solution {
    public int findLengthOfLCIS(int[] nums) {
        int[] dp = new int[nums.length];
        Arrays.fill(dp, 1);
        int res = 1;
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] > nums[i - 1]) {
                dp[i] = dp[i - 1] + 1;
            }
            res = Math.max(res, dp[i]);
        }
        return res;
    }
}
```


### 最长重复子数组 #中等 #rep 

https://leetcode.cn/problems/maximum-length-of-repeated-subarray/description/

分析：
- 如果是暴力的解法，需要先两层 for 循环确定两个数组起始位置，然后再来一个循环从两个起始位置开始比较
- 本题其实是动规解决的经典题目，我们只要想到 用二维数组可以记录两个字符串的所有比较情况

dp：
- `dp[i][j]` 为 以下标 i 为结尾的 A，和以下标 j 为结尾的 B，最长重复子数组长度。
	- **注意：一定是以 i 或 j 结尾的**
- 递推公式：`dp[i][j]` 的状态只能由 `dp[i - 1][j - 1]` 推导出来。即当 `A[i]` 和 `B[j]` 相等的时候，`dp[i][j] = dp[i - 1][j - 1] + 1`
- 初始化：如果 `nums1[0] == nums2[i]`，`dp[0][i] = 1`。如果 `nums2[0] == nums1[i]`，`dp[i][0] = 1`
- 遍历顺序：i 和 j 都从 0 开始，谁在外层都可以

> 遍历顺序，i 和 j 为啥不从 1 开始呢，因为需要记录 dp 数组的最大值，最长重复子数组（res）可能为 0 或 1，初始化的部分也要遍历用于获取 res

|     | 3     | 2     | 1     | 4   | 7   |
| --- | ----- | ----- | ----- | --- | --- |
| 1   | 0     | 0     | 1     | 0   | 0   |
| 2   | 0     | 1     | 0     | 0   | 0   |
| 3   | **1** | 0     | 0     | 0   | 0   |
| 2   | 0     | **2** | 0     | 0   | 0   |
| 1   | 0     | 0     | **3** | 0   | 0   |

```java
class Solution {
    public int findLength(int[] nums1, int[] nums2) {
        int[][] dp = new int[nums1.length][nums2.length];
        // 初始化
        for (int i = 0; i < nums2.length; i++) {
            if (nums1[0] == nums2[i]) {
                dp[0][i] = 1;
            }
        }
        for (int i = 0; i < nums1.length; i++) {
            if (nums2[0] == nums1[i]) {
                dp[i][0] = 1;
            }
        }
        // 递推
        int res = 0;
        for (int i = 0; i < nums1.length; i++) {
            for (int j = 0; j < nums2.length; j++) {
                if (nums1[i] == nums2[j] && i > 0 && j > 0) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                }
                res = Math.max(res, dp[i][j]);
            }
        }
        return res;
    }
}
```

### 最长公共子序列 #中等 #rep

[1143. 最长公共子序列 - 力扣（LeetCode）](https://leetcode.cn/problems/longest-common-subsequence/description/)

分析：
- 本题和 最长重复子数组 区别在于这里**不要求是连续**的了，但要有相对顺序

dp：
- `dp[i][j]`：长度为 `[0, i]` 的字符串 text1 与长度为 `[0, j - 1]` 的字符串 text2 的最长公共子序列为 `dp[i][j]`
- 递推公式：
	- 如果 `text1[i]` 与 `text2[j]` 相同，那么找到了一个公共元素，所以 `dp[i][j] = dp[i - 1][j - 1] + 1`;
	- 如果 `text1[i]` 与 `text2[j]` 不相同，那就看看 `text1[0, i - 1]` 与 `text2[0, j]` 的最长公共子序列 和 `text1[0, i]` 与 `text2[0, j - 1]` 的最长公共子序列，取最大的。即：`dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])`
- 初始化：`dp[0][i]` 和 `dp[i][0]`，看代码
- 遍历顺序：从递推公式，可以看出，有三个方向可以推出 `dp[i][j]`，i 和 j 从 1 开始从小到大遍历，谁在外层都可以

![300](assets/Pasted%20image%2020240622161956.png)

|     | a   | b   | c   | d   | e     |
| --- | --- | --- | --- | --- | ----- |
| a   | 1   | 1   | 1   | 1   | 1     |
| c   | 1   | 1   | 2   | 2   | 2     |
| e   | 1   | 1   | 2   | 2   | ==3== |

```java
public int longestCommonSubsequence(String text1, String text2) {
	char[] chars1 = text1.toCharArray();
	char[] chars2 = text2.toCharArray();
	int[][] dp = new int[chars1.length][chars2.length];
	// 初始化
	for (int i = 0, v = 0; i < chars2.length; i++) {
		if (chars1[0] == chars2[i]) {
			v = 1;
		}
		dp[0][i] = v;
	}
	for (int i = 0, v = 0; i < chars1.length; i++) {
		if (chars1[i] == chars2[0]) {
			v = 1;
		}
		dp[i][0] = v;
	}
	// 递推
	for (int i = 1; i < chars1.length; i++) {
		for (int j = 1; j < chars2.length; j++) {
			if (chars1[i] == chars2[j]) {
				dp[i][j] = dp[i - 1][j - 1] + 1;
			} else {
				dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
			}
		}
	}
	return dp[chars1.length - 1][chars2.length - 1];
}
```

### 不相交的线 #中等 #rep
 
[1035. 不相交的线 - 力扣（LeetCode）](https://leetcode.cn/problems/uncrossed-lines/description/)

分析：
- 本题说是求绘制的最大连线数，其实就是求两个字符串的最长公共子序列的长度！那么和 最长公共子序列 是一样的

```java
public int maxUncrossedLines(int[] nums1, int[] nums2) {
	int[][] dp = new int[nums1.length][nums2.length];
	// 初始化
	for (int i = 0; i < nums2.length; i++) {
		if (nums1[0] == nums2[i]) {
			dp[0][i] = 1;
			for (int j = i + 1; j < nums2.length; j++) {
				dp[0][j] = 1;
			}
			break;
		}
	}
	for (int i = 0; i < nums1.length; i++) {
		if (nums1[i] == nums2[0]) {
			dp[i][0] = 1;
			for (int j = i + 1; j < nums1.length; j++) {
				dp[j][0] = 1;
			}
			break;
		}
	}
	// 递推
	for (int i = 1; i < nums1.length; i++) {
		for (int j = 1; j < nums2.length; j++) {
			if (nums1[i] == nums2[j]) {
				dp[i][j] = dp[i - 1][j - 1] + 1;
			} else {
				dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
			}
		}
	}
	return dp[nums1.length - 1][nums2.length - 1];
}
```

### 最大子数组和 #中等 #手撕2

[53. 最大子数组和 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-subarray/description/)

1）动态规划 ✔

dp 五部曲：
- ` dp[i]`：包括下标 i（以 `nums[i]` 为结尾）的最大连续子序列和为 `dp[i]`
- 递推公式：`dp[i] = max(dp[i - 1] + nums[i], nums[i])`
- 初始化：`dp[0] = nums[0]`
- 遍历顺序：i从1开始从小到大遍历
- 模拟：返回 dp 数组最大值

```java
public int maxSubArray(int[] nums) {
	int[] dp = new int[nums.length];
	dp[0] = nums[0];
	int res = nums[0];
	for (int i = 1; i < nums.length; i++) {
		dp[i] = Math.max(nums[i], nums[i] + dp[i - 1]);
		res = Math.max(res, dp[i]);
	}
	return res;
}
```

2）贪心

分析：
- 局部最优：**当前“连续和”为负数的时候立刻放弃，从下一个元素重新计算“连续和”**，因为负数加上下一个元素 “连续和”只会越来越小。
- 全局最优：选取最大“连续和”

```java
public int maxSubArray(int[] nums) {
	// 不能排序
	int res = nums[0];
	int temp = 0;
	for (int i = 0; i < nums.length; i++) {
		temp = nums[i] + temp;
		res = Math.max(res, temp);
		if (temp < 0) {
			temp = 0;
			continue;
		}
	}
	return res;
}
```

### 判断子序列 #rep

[392. 判断子序列 - 力扣（LeetCode）](https://leetcode.cn/problems/is-subsequence/description/)

1）双指针 ✔

分析：为什么本题可以用双指针？因为求的不是最长、最多、最大，仅仅是判断

```java
public boolean isSubsequence(String s, String t) {
	if (s.length() == 0) {
		return true;
	}
	if (s.length() > t.length()) {
		return false;
	}
	for (int l = 0, r = 0; r < t.length(); r++) {
		if (s.charAt(l) == t.charAt(r) && ++l == s.length()) {
			return true;
		}
	}
	return false;
}
```

2）dp

分析：可以照搬最长公共子序列，返回的结果是 `dp[chars1.length - 1][chars2.length - 1] == s.length()`

```java
public boolean isSubsequence(String s, String t) {
	if (s.length() == 0) {
		return true;
	}
	if (s.length() > t.length()) {
		return false;
	}
	char[] chars1 = s.toCharArray();
	char[] chars2 = t.toCharArray();
	int[][] dp = new int[chars1.length][chars2.length];
	// 初始化
	for (int i = 0, j = 0; i < chars2.length; i++) {
		if (chars1[0] == chars2[i]) {
			j = 1;
		}
		dp[0][i] = j;
	}
	for (int i = 0, j = 0; i < chars1.length; i++) {
		if (chars1[i] == chars2[0]) {
			j = 1;
		}
		dp[i][0] = j;
	}
	// 递推
	for (int i = 1; i < chars1.length; i++) {
		for (int j = 1; j < chars2.length; j++) {
			if (chars1[i] == chars2[j]) {
				dp[i][j] = dp[i - 1][j - 1] + 1;
			} else {
				dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
			}
		}
	}
	return dp[chars1.length - 1][chars2.length - 1] == s.length();
}
```


### 不同的子序列 #困难 #rep

[115. 不同的子序列 - 力扣（LeetCode）](https://leetcode.cn/problems/distinct-subsequences/)

dp五部曲：
- 定义：以 j 为结尾的 s 子序列中出现以 i 为结尾的 t 的个数为 `dp[i][j]`
- 递推公式：
	- 当`s[j]` 与 `t[i]` 相等时，`dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j]`
		- 例如： `s：bagg` 和 `t：bag` ，`s[3]` 和 `t[2]` 是相同的，但是字符串 s 也可以不用 `s[3]` 来匹配，即用 `s[0]s[1]s[2]` 组成的 bag。
	- 当`s[i - 1]` 与 `t[j - 1]`不相等时，`dp[i][j] = dp[i - 1][j]`
- 初始化：初始化 `dp[0][j]`即可，见代码
- 遍历顺序：这里外层 i 遍历 t，内层 j 遍历 s，其实 j 从 i 开始遍历即可

|     | r   | a   | b   | b   | b   | i   | t   |
| --- | --- | --- | --- | --- | --- | --- | --- |
| r   | 1   | 1   | 1   | 1   | 1   | 1   | 1   |
| a   | 0   | 1   | 1   | 1   | 1   | 1   | 1   |
| b   | 0   | 0   | 1   | 2   | 3   | 3   | 3   |
| b   | 0   | 0   | 0   | 1   | 3   | 3   | 3   |
| i   | 0   | 0   | 0   | 0   | 0   | 3   | 3   |
| t   | 0   | 0   | 0   | 0   | 0   | 0   | 3   |

|     | b   | a   | b   | g   | b   | a   | g   |
| --- | --- | --- | --- | --- | --- | --- | --- |
| b   | 1   | 1   | 2   | 2   | 3   | 3   | 3   |
| a   | 0   | 1   | 1   | 1   | 1   | 4   | 4   |
| g   | 0   | 0   | 0   | 1   | 1   | 1   | 5   |

```java
public int numDistinct(String s, String t) {
	if (s.length() < t.length()) {
		return 0;
	}
	int[][] dp = new int[t.length()][s.length()];
	// 初始化
	for (int i = 0, v = 0; i < s.length(); i++) {
		if (t.charAt(0) == s.charAt(i)) {
			v++;
		}
		dp[0][i] = v;
	}
	// 递推
	for (int i = 1; i < t.length(); i++) {
		for (int j = i; j < s.length(); j++) {
			if (t.charAt(i) == s.charAt(j)) {
				dp[i][j] = dp[i - 1][j - 1] + dp[i][j - 1];
			} else {
				dp[i][j] = dp[i][j - 1];
			}
		}
	}
	return dp[t.length() - 1][s.length() - 1];
}
```

### 两个字符串的删除操作 #中等

[583. 两个字符串的删除操作 - 力扣（LeetCode）](https://leetcode.cn/problems/delete-operation-for-two-strings/description/)

分析：同最长公共子序列

```java
public int minDistance(String word1, String word2) {
	char[] chars1 = word1.toCharArray();
	char[] chars2 = word2.toCharArray();
	int[][] dp = new int[chars1.length][chars2.length];
	// 初始化
	for (int i = 0, v = 0; i < chars2.length; i++) {
		if (chars1[0] == chars2[i]) {
			v = 1;
		}
		dp[0][i] = v;
	}
	for (int i = 0, v = 0; i < chars1.length; i++) {
		if (chars1[i] == chars2[0]) {
			v = 1;
		}
		dp[i][0] = v;
	}
	// 递推
	for (int i = 1; i < chars1.length; i++) {
		for (int j = 1; j < chars2.length; j++) {
			if (chars1[i] == chars2[j]) {
				dp[i][j] = dp[i - 1][j - 1] + 1;
			} else {
				dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
			}
		}
	}
	return chars1.length + chars2.length - 2 * dp[chars1.length - 1][chars2.length - 1];
}
```

### 编辑距离 #中等 #rep 

[72. 编辑距离 - 力扣（LeetCode）](https://leetcode.cn/problems/edit-distance/description/)

1）版本一：

dp 五部曲
- 定义：`dp[i][j]` 表示以下标 i-1为结尾的字符串 word1，和以下标 j-1为结尾的字符串 word2，最近编辑距离为 `dp[i][j]`
	- 本题没用 i 和 j 作为结尾，因为**初始化很麻烦**
- 递推公式：
	- `if (word1[i - 1] == word2[j - 1])` 那么说明不用任何编辑，`dp[i][j]` 就应该是 `dp[i - 1][j - 1]`
	- `if (word1[i - 1] != word2[j - 1])`，取下面三种情况的最大值
		- 操作一：word1 删除一个元素，那么就是以下标 i - 2为结尾的 word1 与 j-1为结尾的 word2的最近编辑距离 再加上一个操作，即 `dp[i][j] = dp[i - 1][j] + 1;`
		- 操作二：word2 删除一个元素，那么就是以下标 i - 1 为结尾的 word1 与 j-2 为结尾的 word2 的最近编辑距离 再加上一个操作，即 `dp[i][j] = dp[i][j - 1] + 1;`
		- 操作三：替换元素，`word1` 替换 `word1[i - 1]`，使其与 `word2[j - 1]` 相同，所以 `dp[i][j] = dp[i - 1][j - 1] + 1;`
- 初始化：
- 遍历顺序：`dp[i][0] = i`，即下标 i-1 为结尾的字符串 word1，和空字符串 word2，最近编辑距离为 i，同理 `dp[0][j] = j;`

> 添加元素等价于删除元素，word1 添加一个元素相当于 word2 删除一个元素，word2 添加一个元素，相当于 word1 删除一个元素

|     |     | h   | o   | r   | s   | e   |
| --- | --- | --- | --- | --- | --- | --- |
|     | 0   | 1   | 2   | 3   | 4   | 5   |
| r   | 1   | 1   | 2   | 2   | 3   | 4   |
| o   | 2   | 2   | 1   | 2   | 3   | 4   |
| s   | 3   | 3   | 2   | 2   | 2   | 3   |

```java
public static int minDistance2(String word1, String word2) {
	int m = word1.length();
	int n = word2.length();
	int[][] dp = new int[m + 1][n + 1];
	// 初始化
	for (int i = 1; i <= m; i++) {
		dp[i][0] = i;
	}
	for (int j = 1; j <= n; j++) {
		dp[0][j] = j;
	}
	// 递推
	for (int i = 1; i <= m; i++) {
		for (int j = 1; j <= n; j++) {
			if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
				dp[i][j] = dp[i - 1][j - 1];
			} else {
				dp[i][j] = Math.min(Math.min(dp[i - 1][j - 1], dp[i][j - 1]), dp[i - 1][j]) + 1;
			}
		}
	}
	return dp[m][n];
}
```

2）`dp[i][j]` 表示以下标 i 为结尾的字符串 word1 和以下标 j 为结尾的字符串 word2 的最近编辑距离，**初始化很麻烦**

以 u，aauu 为例，第一次 u 相等时 `dp[0][j]=dp[0][j-1]`，第二次 u 相等时 `dp[0][j]=dp[0][j-1]+1`，

```java
public int minDistance(String word1, String word2) {
	if (word1.isEmpty()) {
		return word2.length();
	}
	if (word2.isEmpty()) {
		return word1.length();
	}
	char[] chars1 = word1.toCharArray();
	char[] chars2 = word2.toCharArray();
	int[][] dp = new int[chars1.length][chars2.length];
	// 初始化
	int count = 0;
	if (chars1[0] != chars2[0]) {
		dp[0][0] = 1;
	} else {
		count++;
	}
	for (int j = 1, count1 = count; j < chars2.length; j++) {
		if (chars1[0] == chars2[j]) {
			if (count1 == 0) {
				dp[0][j] = dp[0][j - 1];
				count1++;
				continue;
			}
		}
		dp[0][j] = dp[0][j - 1] + 1;
	}
	for (int i = 1, count2 = count; i < chars1.length; i++) {
		if (chars1[i] == chars2[0]) {
			if (count2 == 0) {
				dp[i][0] = dp[i - 1][0];
				count2++;
				continue;
			}
		}
		dp[i][0] = dp[i - 1][0] + 1;
	}
	// 递推
	for (int i = 1; i < chars1.length; i++) {
		for (int j = 1; j < chars2.length; j++) {
			if (chars1[i] == chars2[j]) {
				dp[i][j] = dp[i - 1][j - 1];
			} else {
				int temp = Math.min(dp[i - 1][j], dp[i][j - 1]); // 删除的最小操作数
				dp[i][j] = Math.min(dp[i - 1][j - 1], temp) + 1;
			}
		}
	}
	return dp[chars1.length - 1][chars2.length - 1];
}
```


## 回文子串 #中等 #rep

[647. 回文子串 - 力扣（LeetCode）](https://leetcode.cn/problems/palindromic-substrings/description/) 

分析：
- 暴力解法：两层 for 循环，遍历区间起始位置和终止位置，然后还需要一层遍历判断这个区间是不是回文。所以时间复杂度：`O(n^3)`

2）动态规划：

分析：
- 本题如果定义 `dp[i]` 为 下标 i 结尾的字符串有 `dp[i]` 个回文串的话，我们会发现很难找到递归关系。因为 `dp[i]` 和 `dp[i-1]`，`dp[i + 1]` 看上去都没啥关系
- 判断一个子字符串（字符串的下标范围 `[i,j]`）是否回文，依赖于，子字符串（下表范围 `[i + 1, j - 1]`） 是否是回文

![400](assets/Pasted%20image%2020240629123021.png)

dp：
- `dp[i][j]`：表示区间范围 `[i,j]` 的子串是否是回文子串，如果是 `dp[i][j]` 为 true，否则为 false。
- 递推公式：
	- 当 `s[i] != s[j]`，`dp[i][j]` 一定是 false
	- 当 `s[i] == s[j]`
		- 下标 i 与 j 相同或相差为 1：是回文子串，例如 a、aa
		- 下标 i 与 j 相差大于 1：和 `dp[i + 1][j - 1]` 相同
- 初始化：不需要特别初始化，默认都是 false
- 遍历顺序：需要由 `dp[i + 1][j - 1]` 推出 `dp[i][j]`，`dp[i + 1][j - 1]` 在 `dp[i][j]` 的左下角。从下到上，从左到右遍历。遍历区域为主对角线及其上半部分即可

|     | c   | b   | a   | b   | c   |
| --- | --- | --- | --- | --- | --- |
| c   | t   | f   | f   | f   | t   |
| b   |     | t   | f   | t   | f   |
| a   |     |     | t   | f   | f   |
| b   |     |     |     | t   | f   |
| c   |     |     |     |     | t   |

```java
public int countSubstrings(String s) {
	int res = 0;
	char[] chars = s.toCharArray();
	int len = chars.length;
	boolean[][] dp = new boolean[len][len];
	for (int i = len - 1; i >= 0; i--) {
		for (int j = i; j < len; j++) {
			if (chars[i] == chars[j]) {
				if (j - i <= 1) {
					dp[i][j] = true;
				} else {
					dp[i][j] = dp[i + 1][j - 1];
				}
			}
			if (dp[i][j]) {
				res++;
			}
		}
	}
	return res;
}
```

## 最长回文子串 #中等 #手撕 #rep

分析：
- 本题和回文子串的思路相似

dp 五部曲：
- `dp[i][j]`：字符串 s 在 `[i, j]` 范围内最长的回文子序列的长度
- 递推公式：
	- `s[i]` 与 `s[j]` 相同，`dp[i][j] = dp[i + 1][j - 1] + 2`
	- `s[i]` 与 `s[j]` 不相同，`dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])`
- 初始化：对角线为 1，其他为 0
- 遍历顺序：对角线以上，从下到上，从左到右
- **返回值**：`dp[0][len-1]`

|     | b   | b   | b   | a   | b   |
| --- | --- | --- | --- | --- | --- |
| b   | 1   | 2   | 3   | 3   | 4   |
| b   |     | 1   | 2   | 2   | 3   |
| b   |     |     | 1   | 1   | 3   |
| a   |     |     |     | 1   | 1   |
| b   |     |     |     |     | 1   |

```java
public int longestPalindromeSubseq(String s) {
	char[] chars = s.toCharArray();
	int[][] dp = new int[chars.length][chars.length];
	for (int i = chars.length - 1; i >= 0; i--) {
		for (int j = i; j < chars.length; j++) {
			if (j == i) {
				dp[i][j] = 1;
				continue;
			}
			if (chars[i] == chars[j]) {
				dp[i][j] = dp[i + 1][j - 1] + 2;
			} else {
				dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
			}
		}
	}
	return dp[0][chars.length - 1];
}
```

