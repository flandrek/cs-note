#### 接雨水-42

给定 *n* 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![rainwatertrap](E:\markdown笔记\图片\rainwatertrap.png)

```
输入: [0,1,0,2,1,0,1,3,2,1,2,1]
输出: 6
```

在一个位置能容下的雨水量等于它左右两边柱子最大高度的最小值减去它的高度，所以，问题就变成了: 如何找所有位置的左右两边的柱子的最大值？

```java
class Solution {
    public int trap(int[] height) {
        if (height == null || height.length == 0) return 0;
        int left = 0;
        int right = height.length - 1;
        int left_max = 0;
        int right_max = 0;
        int res = 0;
        while (left < right) {
            if (height[left] < height[right]) {
                if (height[left] < left_max) res += left_max - height[left];
                else left_max = height[left];
                left++;
            } else {
                if (height[right] < right_max) res += right_max - height[right];
                else right_max = height[right];
                right--;
            }
        }
        return res; 
    }
}
```



#### 删除排序数组中的重复项-26

给定一个排序数组，你需要在原地删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

```
给定 nums = [0,0,1,1,1,2,2,3,3,4],

函数应该返回新的长度 5, 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4。

你不需要考虑数组中超出新长度后面的元素。
```

**双指针法：**

使用两个指针 i 和 j，当 nums[i] = nums[j]，就增加 j 跳过重复项；当 nums[i] != nums[j] ，把 nums[j] 复制到 nums[i+1]，然后递增 i ，直到循环结束。

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i]) {
            i++;
            nums[i] = nums[j];
        }
    }
    return i + 1;
}
```



#### 按奇偶排序数组-905

给定一个非负整数数组 `A`，返回一个数组，在该数组中， `A` 的所有偶数元素之后跟着所有奇数元素。你可以返回满足此条件的任何数组作为答案。

```
输入：[3,1,2,4]
输出：[2,4,3,1]
输出 [4,2,3,1]，[2,4,1,3] 和 [4,2,1,3] 也会被接受。
```

**思路：**双指针

```java
class Solution {
    public int[] sortArrayByParity(int[] A) {
        int l = 0, r = A.length-1;
        while(l < r){
            while( l < A.length && A[l] % 2 == 0) l++;
            while( r >= 0 && A[r] % 2 == 1) r--;
            if(l < r){
                int temp = A[l];
                A[l] = A[r];
                A[r] = temp;
            }            
        }
        return A;
    }
}
```



#### 按奇偶排序数组-II-922

给定一个非负整数数组 A， A 中一半整数是奇数，一半整数是偶数。对数组进行排序，以便当 A[i] 为奇数时，i 也是奇数；当 A[i] 为偶数时， i 也是偶数。你可以返回任何满足上述条件的数组作为答案。

**思路：**双指针

```java
class Solution {
    public int[] sortArrayByParityII(int[] A) {
        if(A.length < 2) return A;
        int odd = 1, even = 0;
        while(odd < A.length && even < A.length){
            while(odd < A.length && A[odd] % 2 == 1) odd += 2;
            while(even < A.length && A[even] % 2 == 0) even += 2;
            if(odd < A.length && even < A.length){
                int temp = A[odd];
                A[odd] = A[even];
                A[even] = temp;
            }
        }
        return A;
    }
}
```

使用额外空间：

```java
class Solution {
    public int[] sortArrayByParityII(int[] A) {
        int[] B = new int[A.length];
        int index1 = 0, index2 = 1;
        for(int i = 0; i < A.length; i++){
            if(A[i] % 2 == 0){
                B[index1] = A[i];
                index1 += 2;
            }else{
                B[index2] = A[i];
                index2 += 2;
            }
        }
        return B;
    }
}
```

