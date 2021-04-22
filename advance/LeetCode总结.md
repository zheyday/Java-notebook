# 数组

## 双指针

### 从头开始



### 首尾开始

11、盛水的容器

![image-20200210092718950](LeetCode总结/image-20200210092718950.png)

两线段之间形成的区域总是会受到其中较短那条长度的限制。此外，两线段距离越远，得到的面积就越大。所以先从首尾两端开始查找，移动较小的那个

## 三指针





## 二分查找

**解决问题**：在数组中找到目标值的索引

有两种写法，记住一种即可

```java
public int searchInsert(int[] nums, int target) {
    int left = 0, right = nums.length;//注意right
    while (left < right) {			//注意是<
        int mid = left + (right-left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;			//注意是mid
        } else
            return mid;
    }
    return left;
}
```



## 摩尔投票法



## 分治法

**解决问题**：最大（小）和的连续子数组

## 贪心



## 动态规划

问题要符合最优子结构，也就是子问题必须互相独立

- 明确初始条件
- 明确状态
- 明确选择
- 定义dp的含义



## 回溯



## 位图法

[https://github.com/MisterBooo/LeetCodeAnimation/blob/master/notes/LeetCode%E7%AC%AC268%E5%8F%B7%E9%97%AE%E9%A2%98%EF%BC%9A%E7%BC%BA%E5%A4%B1%E6%95%B0%E5%AD%97.md](https://github.com/MisterBooo/LeetCodeAnimation/blob/master/notes/LeetCode第268号问题：缺失数字.md)

给定一个包含 `0, 1, 2, ..., n` 中 *n* 个数的序列，找出 0 .. *n* 中没有出现在序列中的那个数

```
输入: [3,0,1]
输出: 2
```

**解法**：

补充一个完整数组，问题就变成了找出两个数组中只出现一次的数。

两个相同的数异或操作就变成了0，而且异或操作满足交换律，所以将两个数组全部异或，结果即为所求

(9^0)^(6^1)...(1^8)^9
=(0^0)^(1^1)...(9^9)^8
=0^8
=8

![image-20200209172926774](LeetCode总结/image-20200209172926774.png)



# 二叉树

### 前序、中序、后序

![image-20200215212806148](LeetCode总结/image-20200215212806148.png)

前序：根结点--左子树--右子树  A-B-D-E-C-F

中序：左子树--根结点--右子树  D-B-E-A-F-C

后序：左子树--右子树--根结点  D-E-B-F-C-A  

# 排序

| 排序方法 | 平均时间复杂度 | 最坏时间复杂度 | 最好时间复杂度 | 空间复杂度 | 稳定性 |
| -------- | -------------- | -------------- | -------------- | ---------- | ------ |
| 冒泡     | $O(n^2)$       | $O(n^2)$       | $O(n)$         |            | 稳定   |
| 选择     | $O(n^2)$       | $O(n^2)$       | $O(n^2)$       |            | 不稳定 |
| 插入     | $O(n^2)$       | $O(n^2)$       | $O(n)$         |            | 稳定   |
| 希尔     | O(n^1.3^)      | $O(n^2)$       | $O(n)$         |            | 不稳定 |
| 快速排序 | $O(nlogn)$     | $O(n^2)$       | $O(nlogn)$     |            | 不稳定 |
| 堆排序   | $O(nlogn)$     | $O(nlogn)$     | $O(nlogn)$     |            | 不稳定 |
| 归并排序 | $O(nlogn)$     | $O(nlogn)$     | $O(nlogn)$     |            | 稳定   |



![image-20200414152909405](LeetCode总结/image-20200414152909405.png)

## 希尔排序

```java
public int[] shellSort(int[] nums) {
    for (int gap = nums.length / 2; gap >= 1; gap /= 2) {
        for (int j = gap; j < nums.length; j++) {
            shellInsert(nums, gap, j);
        }
    }
    return nums;
}

private void shellInsert(int[] nums, int gap, int start) {
    if (nums[start - gap] > nums[start]) {
        int j = start - gap;
        int tmp = nums[start];
        while (j >= 0 && nums[j] > tmp) {
            nums[j + gap] = nums[j];
            j -= gap;
        }
        nums[j + gap] = tmp;
    }
}
```

## 归并排序



## 快速排序

https://wiki.jikexueyuan.com/project/easy-learn-algorithm/fast-sort.html

挑选一个元素(如最左)，称为基准。从两边向中间遍历，将小于基准的放在左边，大于的放在右边，遍历完之后i==j，交换i和基准值。再继续对左边和右边进行递归

![image-20200305153157066](LeetCode总结/image-20200305153157066.png)

## 堆排序

https://www.cnblogs.com/chengxiao/p/6129630.html

最好、最坏、平均复杂度均为O(nlogn)，是不稳定排序

### 堆

每个结点的值都大于等于左右子节点叫大顶堆，否则叫小顶堆

![image-20200303214512330](LeetCode总结/image-20200303214512330.png)

### 基本思想

将数组构建成一个大顶堆（升序，降序用小顶堆），此时堆顶就是最大值，将其与末尾元素交换，然后剩下的n-1个重新构建大顶堆





## 桶排序





# 链表

常用操作：

1. 设置dummy
2. 设置pre=null

# 不会的题

#### [面试题11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

#### 53. 最大子数组

**题目：**

给定一个数组 **nums**，找出一个最大和的连续子数组（至少包含一个数字）并返回

<blockquote>
Input: [-2,1,-3,4,-1,2,1,-5,4],<br>
Output: 6<br>
Explanation: [4,-1,2,1] has the largest sum = 6.<br>

**思路：**

找最大值的题目，先定义 **max**。题目要求是连续子数组，定义一个sum，然后遍历数组。如果sum<0，就舍弃之前的和，令sum=nums[i]，否则就加上nums[i]，然后max取最大的一个值。

**代码：**

```java
public int maxSubArray(int[] nums) {
        if (nums == null || nums.length==0)
            return 0;
        
        int sum=nums[0];
        int max=sum;
         
        for (int i=1;i<nums.length;i++){
            sum=sum<0?nums[i]:sum+nums[i];//如果前面的sum<0了，那么就舍弃，从当前开始重新计算
            //或者sum=Math.max(sum+nums[i],nums[i]);
            max=Math.max(max,sum);
        }
        return max;
  }
```

#### 121. 买卖股票的最好时机

问题：

<blockquote>
    给定一个数组模拟股票每天的价格，每天只允许买或卖一次，要求找到最大利润
</blockquote>

例子：

<blockquote>
    Input: [7,1,5,3,6,4]<br>
Output: 5<br>
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.<br>
             Not 7-1 = 6, as selling price needs to be larger than buying price.
</blockquote>

思路：

要找到最大利润，肯定是低买高卖，所以要找到最低价，然后遍历操作取差最大值。

代码：


```java
public int maxProfit(int[] prices) {
	if (prices==null || prices.length<2)
        return 0;   
	int minPrice=prices[0];
    int max=0;
    
    for (int i=1;i<prices.length;i++){
        minPrice=Math.min(minPrice,prices[i]);
        max=Math.max(max,prices[i]-minPrice);
    }
    return max;
}
```

#### [91. 解码方法](https://leetcode-cn.com/problems/decode-ways/)

