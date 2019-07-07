#### 最长上升子序列-300

给定一个无序的整数数组，找到其中最长上升子序列的长度。

```
输入: [10,9,2,5,3,7,101,18]
输出: 4 
解释: 最长的上升子序列是 [2,3,7,101]，它的长度是 4。
```

**二分查找：**

- 时间复杂度：O*(*N*log*N*)。
- 空间复杂度：O*(*N*)。

定义一个 tails 数组，其中 tails[i] 存储长度为 i + 1 的最长递增子序列的最后一个元素。对于一个元素 x，

- 如果它大于 tails 数组所有的值，那么把它添加到 tails 后面，表示最长递增子序列长度加 1；
- 如果 tails[i-1] < x <= tails[i]，那么更新 tails[i] = x。

例如对于数组 [4,3,6,5]，有：

```
tails      len      num
[]         0        4
[4]        1        3
[3]        1        6
[3,6]      2        5
[3,5]      2        null
```

可以看出 tails 数组保持有序，因此在查找 S<sub>i</sub> 位于 tails 数组的位置时就可以使用二分查找

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] tails = new int[n];
    int len = 0;
    for (int num : nums) {
        int index = binarySearch(tails, len, num);
        tails[index] = num;
        if (index == len) {
            len++;
        }
    }
    return len;
}

private int binarySearch(int[] tails, int len, int key) {
    int l = 0, h = len;
    while (l < h) {
        int mid = l + (h - l) / 2;
        if (tails[mid] == key) {
            return mid;
        } else if (tails[mid] > key) {
            h = mid;
        } else {
            l = mid + 1;
        }
    }
    return l;
}
```



#### 搜索旋转排序数组-33

假设按照升序排序的数组在预先未知的某个点上进行了旋转。例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] 

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。你可以假设数组中**不存在重复的元素。**你的算法时间复杂度必须是 O (log n) 级别。

```
输入: nums = [4,5,6,7,0,1,2], target = 0
输出: 4
输入: nums = [4,5,6,7,0,1,2], target = 3
输出: -1
```

**思路一：**

整体思路：先用二分法找出最小值，也是那个分割点,例如 [4,5,6,7,0,1,2]，我们找出数字 0；

接下来判断 target 是在分割点的左边还是右边；最后再使用一次二分法找出 target 的位置. 所以时间复杂度为：O(logn)

```java
class Solution {
    public int search(int[] nums, int target) {
        if (nums == null || nums.length == 0) return -1;
        int left = 0;
        int right = nums.length - 1;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] > nums[right]) left = mid + 1;
            else right = mid;
        }
        //System.out.println(left);
        int split_t = left;
        left = 0;
        right = nums.length - 1;
        if (nums[split_t] <= target && target <= nums[right]) left = split_t;
        else right = split_t;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) return mid;
            else if (nums[mid] > target) right = mid - 1;
            else left = mid + 1;
        }
        return -1;       
    }
}
```

**思路二：**

直接使用二分法，判断那个二分点，有几种可能性

1.直接等于target

2.在左半边的递增区域

   a. target 在 left 和 mid 之间

   b. 不在之间

3.在右半边的递增区域

   a. target 在 mid 和 right 之间

   b. 不在之间

```java
class Solution {
    public int search(int[] nums, int target) {
        if (nums == null || nums.length == 0) return -1;
        int n = nums.length;
        int left = 0;
        int right = n - 1;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) return mid;
            else if (nums[left] <= nums[mid]) {
                if (nums[left] <= target && target < nums[mid]) right = mid - 1;
                else left = mid + 1;

            } else if (nums[mid] < nums[right]) {
                if (nums[mid] < target && target <= nums[right]) left = mid + 1;
                else right = mid - 1;
            } 
        }
        return nums[left] == target ? left : -1;        
    }
}
```



#### 搜索旋转排序数组II-81

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 [0,0,1,2,2,5,6] 可能变为 [2,5,6,0,0,1,2] )。

编写一个函数来判断给定的目标值是否存在于数组中。若存在返回 true，否则返回 false。

```
输入: nums = [2,5,6,0,0,1,2], target = 0
输出: true
输入: nums = [2,5,6,0,0,1,2], target = 3
输出: false
```

**进阶:**

- 这是 [搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/description/) 的延伸题目，本题中的 `nums`  可能包含重复元素。
- 这会影响到程序的时间复杂度吗？会有怎样的影响，为什么？

**思路：**二分法，判断二分点，几种可能性

1. 直接nums[mid] == target
2. 当数组为[1,2,1,1,1]，nums[mid] == nums[left] == nums[right]，需要 left++，right --；
3. 当nums[left]<= nums[mid],说明是在左半边的递增区域

	a. nums[left] <=target < nums[mid],说明target在left和mid之间.我们令right = mid - 1;
	
	b. 不在之间, 我们令 left = mid + 1;

4. 当nums[mid] < nums[right],说明是在右半边的递增区域

	a. nums[mid] < target <= nums[right],说明target在mid 和right之间,我们令left = mid + 1
	
	b. 不在之间,我们令right = mid - 1;

```java
class Solution {
    public boolean search(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) return true;
            if (nums[left] == nums[mid] && nums[mid] == nums[right]) {
                left++;
                right--;
            } else if (nums[left] <= nums[mid]) {
                if (nums[left] <= target && target < nums[mid]) right = mid - 1;
                else left = mid + 1;
            } else {
                if (nums[mid] < target && target <= nums[right]) left = mid + 1;
                else right = mid - 1;
            }
        }
        return false;  
    }
}
```





#### 在排序数组中查找元素的第一个和最后一个值-34

给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。你的算法时间复杂度必须是 O(log n) 级别。如果数组中不存在目标值，返回 `[-1, -1]`。

```
输入: nums = [5,7,7,8,8,10], target = 8
输出: [3,4]
输入: nums = [5,7,7,8,8,10], target = 6
输出: [-1,-1]
```

**思路：**先用二分查找找出目标值，然后向两边扩散，直到找到两边的边界。

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int[] res = {-1, -1};
        int n = nums.length;
        if (n == 0) return res;
        int left = 0;
        int right = n-1;
        while (left <= right){
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) {
                left = mid;
                right = mid;
                while (left > 0 && nums[left] == nums[left-1]) left--;
                while (right < n - 1 && nums[right] == nums[right+1]) right++;
                res[0] = left;
                res[1] = right;
                return res;
            }
            else if (nums[mid] > target) right = mid - 1;
            else left = mid + 1;
        }
        return res;      
    }
}
```

**思路二：**

二分查找查左右边界。

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int[] res = new int[]{-1, -1};
        if(nums == null || nums.length == 0) return res;
        // find the left-end
        int left = 0;
        int right = nums.length - 1;
        while (left < right) {
            int mid = left + ((right - left) >> 1);
            if (nums[mid] < target) left = mid + 1; 
            else  right = mid;
        }
        res[0] = nums[left] == target ? left : -1;        
        // find right-end
        if (res[0] != -1) {
            if (left == nums.length - 1 || nums[left + 1] != target) {
                res[1] = left;
            } else {
                right = nums.length - 1;
                while (left < right) {
                    int mid = left + ((right - left) >> 1) + 1;
                    if (nums[mid] > target)  right = mid - 1;
                    else left = mid;
                }
                res[1] = right;
            }
        }
        return res;
    }
}
```



#### 寻找峰值-162

峰值元素是指其值大于左右相邻值的元素。给定一个输入数组 nums，其中 nums[i] ≠ nums[i+1]，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回任何一个峰值所在位置即可。

你可以假设 nums[-1] = nums[n] = -∞。

```
输入: nums = [1,2,3,1]
输出: 2
解释: 3 是峰值元素，你的函数应该返回其索引 

输入: nums = [1,2,1,3,5,6,4]
输出: 1 或 5 
解释: 你的函数可以返回索引 1，其峰值元素为 2；
     或者返回索引 5， 其峰值元素为 6。
```

**思路：**二分查找找左边界

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int left = 0, right = nums.length -1;
        while(left < right){
            int mid = left + ((right - left) >> 1);
            if(nums[mid] > nums[mid+1]) 
                right = mid;
            else left = mid + 1;
        }
        return left;
    }
}
```



#### x 的平方根-69

实现 int sqrt(int x) 函数。计算并返回 x 的平方根，其中 x 是非负整数。由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。

```
输入: 8
输出: 2
说明: 8 的平方根是 2.82842..., 
     由于返回类型是整数，小数部分将被舍去。
```

**思路：**二分法

```java
class Solution {  
     public int mySqrt1(int x) {
        int left = 1;
        int right = (x / 2) + 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (mid == x / mid) return mid;
            else if (mid < x / mid) left = mid + 1;
            else right = mid - 1;
        }
        return right;   
    }
}
```

