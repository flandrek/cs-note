#### 两数之和-1

给定一个整数数组，找到两个数字，使它们加起来成为一个特定的目标数字。

函数 two-Sum 应该返回两个数字的索引，使它们加到目标，其中索引1必须小于索引2。请注意，您返回的答案(索引1和索引2)不是基于零的。

您可以假设每个输入都有一个解决方案。

输入: number = {2,7,11,15}，target=9

输出: index 1 = 1, index 2 = 2

```java
import java.util.HashMap;
public class Solution {
    public int[] twoSum(int[] numbers, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < numbers.length; i++){
            if(map.containsKey(target - numbers[i]))
                return new int[]{map.get(target - numbers[i]) + 1, i + 1};
            else
                map.put(numbers[i], i);
        }
        return null;
    }
}
```



#### **3-sum**

给定一个n整数的数组S, S中是否有元素a, b, c使得a + b + c = 0 ？ 找出数组中所有唯一3个元素组合，使他们的和为 0。注意:三元组中的元素(a,b,c)必须是非降序的。(即 a<=b<=c)，解决方案集不能包含重复的组合。

**采用排序 + 双指针，时间复杂度O(n^2)**

``` java
import java.util.*;
public class Solution {
    public ArrayList<ArrayList<Integer>> threeSum(int[] num) {
        Arrays.sort(num);
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();      
        for (int i = 0; i < num.length - 2; i++){
            if (i > 0 && num[i] == num[i-1]) //nums[i] != nums[i-1] 去重
                continue;
            int l = i +1, r = num.length -1;
            while(l < r){
                int sum = num[i] + num[l] + num[r];
                if (sum == 0){
                    ArrayList<Integer> tmp = new ArrayList<>();
                    tmp.add(num[i]);
                    tmp.add(num[l]);
                    tmp.add(num[r]);
                    res.add(tmp);
                    while(l < r && num[l] == num[l+1]) l++;
                    while(l < r && num[r] == num[r-1]) r--; //去重
                    l++;
                    r--;
                }else if (sum > 0){
                    while(l < r && num[r] == num[r-1]) r--; //去重
                    r--;
                }
                else {
                    while(l < r && num[l] == num[l+1]) l++; //去重
                    l++;
                }
            }
        }
        return res;
    }
}
```



#### **3-sum-closest**

找出3数之和最接近目标值，返回最接近的值。

```java
import java.util.*;
public class Solution {
    public int threeSumClosest(int[] num, int target) {
        Arrays.sort(num);
        int ans = num[0] + num[1] + num[2];
        for(int i = 0; i < num.length; i++) {
            int start = i +1, end = num.length - 1;
            while(start < end) {
                int sum = num[start] + num[end] + num[i];
                if(Math.abs(target - sum) < Math.abs(target - ans))
                    ans = sum;
                if(sum > target)
                    end--;
                else if(sum < target)
                    start++;
                else
                    return ans;
            }
        }
        return ans;
    }
}
```



#### **4-sum**

给定一个n整数的数组S, S中是否有元素a, b, c, d使得a + b + c + d = target ？ 找出数组中所有唯一4个元素组合，使他们的和为 target 。注意:三元组中的元素 (a, b, c, d) 必须是非降序的。( 即 a<=b<=c<=d )，解决方案集不能包含重复的组合。

```html
给定数组 nums = [1, 0, -1, 0, -2, 2]，和 target = 0。
满足要求的四元组集合为：
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
```

固定两个数，找另外两个数：

```java
import java.util.*;
public class Solution {
    public ArrayList<ArrayList<Integer>> fourSum(int[] num, int target) {
        ArrayList<ArrayList<Integer>> res = new ArrayList<>(); 
        if(num == null || num.length == 0)
            return res;
        Arrays.sort(num);
        for(int i = 0; i < num.length -3; i++){
            if(i != 0 && num[i] == num[i-1]) continue;
            for(int j = i +1; j < num.length -2; j++){
                if(j != i +1 && num[j] == num[j-1]) continue;
                int l = j +1;
                int r = num.length -1;
                while(l < r){
                    int sum = num[i] + num[j] + num[l] + num[r];
                    if(sum == target){
                        ArrayList<Integer> list = new ArrayList<>();
                        list.add(num[i]);
                        list.add(num[j]);
                        list.add(num[l]);
                        list.add(num[r]);
                        res.add(list);
                        while(l < r && num[l+1] == num[l]) l++;
                        while(l < r && num[r-1] == num[r]) r--;
                        l++;
                        r--;
                    }else if(sum < target){
                        while(l < r && num[l+1] == num[l]) l++;
                        l++;
                    }else{
                        while(l < r && num[r-1] == num[r]) r--;
                        r--;
                    }
                }
            }
        }
        return res;
    }
}
```

~~回溯法~~（不知道是否正确）：

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> fourSum(int[] nums, int target) {
      if(nums == null || nums.length < 1)
          return res;
      Arrays.sort(nums);
      List<Integer> list = new ArrayList<>();
        process(nums, -1, target, list);
        return res;
    }
  
   void process(int[] nums,int index,int target,List<Integer> list){
        if(list.size() == 4){
            if(target == 0 && !res.contains(list))
                res.add(new ArrayList<Integer>(list));
            return;
        }
        if(index >= nums.length)
               return;
        for(int i = index +1; i < nums.length; i++){
            list.add(nums[i]);     
            process(nums, i, target -nums[i], list);
            list.remove(list.size() -1);
        }
    }
}
```



#### 求众数-169

给定一个大小为 *n* 的数组，找到其中的众数。众数是指在数组中出现次数**大于** `⌊ n/2 ⌋` 的元素，你可以假设数组是非空的，并且给定的数组总是存在众数。

```
输入: [2,2,1,1,1,2,2]
输出: 2
```

**思路1：**基于快排的 partition 方法。数字次数超过一半，则说明：排序之后数组中间的数字一定就是所求的数字。

　　利用partition()函数获得某一随机数字，其余数字按大小排在该数字的左右。若该数字下标刚好为n/2，则该数字即为所求数字；若小于n/2，则在右边部分继续查找；反之，左边部分查找。

```java
class Solution {
    public int majorityElement(int[] nums) {
        int left = 0, right = nums.length - 1;
        int index = partition(nums, left, right);
        int mid = left + (right - left) / 2;
        while(index != mid){
            if(index < mid){
                left = index + 1;
                index = partition(nums, left, right);
            } 
                
            if(index > mid) {
                right = index - 1;
                index = partition(nums, left, right);
            }
        }
        return nums[index];
    }
    private int partition(int[] nums, int left, int right){
        int temp = nums[left];
        while(left < right){
            while(left < right && nums[right] >= temp) right--;
            nums[left] = nums[right];
            while(left < right && nums[left] <= temp) left++;
            nums[right] = nums[left];
        }
        nums[left] = temp;
        return left;
    }
}
```



**思路2：**数字次数超过一半，则说明：该数字出现的次数比其他数字之和还多

　　遍历数组过程中保存两个值：一个是数组中某一数字，另一个是次数。遍历到下一个数字时，若与保存数字相同，则次数加1，反之减1。若次数=0，则保存下一个数字，次数重新设置为1。由于要找的数字出现的次数比其他数字之和还多，那么要找的数字肯定是最后一次把次数设置为1的数字。

```java
class Solution {
    public int majorityElement(int[] nums) {
        int res = nums[0];
        int count = 1;
        for(int i = 1; i < nums.length; i++){
            if(count == 0){
                res = nums[i];
                count = 1;
            }
            else if(nums[i] == res) count++;
            else count --;
        }
        return res;
    }
}
```



#### 找到所有数组中消失的数字-448

给定一个范围在  1 ≤ a[i] ≤ n ( n = 数组大小 ) 的 整型数组，数组中的元素一些出现了两次，另一些只出现一次。

找到所有在 [1, n] 范围之间没有出现在数组中的数字。

您能在不使用额外空间且时间复杂度为O(n)的情况下完成这个任务吗? 你可以假定返回的数组不算在额外空间内。

```
输入:
[4,3,2,7,8,2,3,1]

输出:
[5,6]
```

位图法：使用额外的空间

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> res = new ArrayList();
        boolean[] map = new boolean[nums.length];
        for(int num : nums) map[num-1] = true;
        for(int i = 0; i < nums.length; i++){
            if(!map[i]) res.add(i+1);
        }
        for(int i = 0; i < nums.length; i++){
            if(nums[i] > 0) res.add(i+1);
        }
        return res;
    }
}
```

不使用额外空间，遍历时将访问到的数值代表的下标位置为负值，然后再遍历，找到大于 0 的值。

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> res = new ArrayList();
        for(int i = 0; i < nums.length; i++){
            int x = Math.abs(nums[i]) - 1;
            nums[x] = -Math.abs(nums[x]);
        }
        for(int i = 0; i < nums.length; i++){
            if(nums[i] > 0) res.add(i+1);
        }  
        return res;
    }
}
```

不使用额外空间，遍历时将数放到他对应的下标上，再次遍历如果下标和数的值不对应，则就为要找的值。

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> res = new ArrayList<>();
        for(int i=0;i<nums.length;i++){
             while(nums[i] != i+1 && nums[nums[i]-1] != nums[i]){
             	swap(nums, i, nums[i]-1);
          	 }
        }
        for(int i = 0; i < nums.length; i++){
            if(nums[i] != i+1) res.add(i + 1);
        }
        return res;
    }
    public void swap(int[] num, int i, int j){
        int tmp = num[i];
         num[i] = num[j];
         num[j] = tmp;
    }
}
```



#### 搜索二维矩阵-74

编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

1. 每行中的整数从左到右按升序排列。
2. 每行的第一个整数大于前一行的最后一个整数。

```html
输入:
matrix = [
  [1,   3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
]
target = 3
输出: true

输入:
matrix = [
  [1,   3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
]
target = 13
输出: false
```

**思路：**

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        if(matrix == null || matrix.length == 0) return false;
        int row = matrix.length, col = matrix[0].length;
        int i = 0, j = col - 1;
        while(i < row && j >= 0){
            if(matrix[i][j] == target) return true;
            else if(matrix[i][j] > target) j--;
            else i++;
        }
        return false;
    }
}
```



#### 旋转图像

给定一个 n × n 的二维矩阵表示一个图像，将图像顺时针旋转 90 度。说明：你必须在原地旋转图像，这意味着你需要直接修改输入的二维矩阵，请不要使用另一个矩阵来旋转图像。

```java
给定 matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],
原地旋转输入矩阵，使其变为:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```

最直接的想法是先转置矩阵，然后翻转每一行。这个简单的方法已经能达到最优的时间复杂度O(N^2) 。

```java
class Solution {
  	public void rotate(int[][] matrix) {
    	int n = matrix.length;
    	// transpose matrix 矩阵转置 i,j 互换
    	for (int i = 0; i < n; i++) {
      		for (int j = i; j < n; j++) {
        		int tmp = matrix[j][i];
        		matrix[j][i] = matrix[i][j];
        		matrix[i][j] = tmp;
      		}
   		}
        // reverse each row 每一行互换
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n / 2; j++) {
                int tmp = matrix[i][j];
                matrix[i][j] = matrix[i][n - j - 1];
                matrix[i][n - j - 1] = tmp;
            }
        }
  	}
}
```



#### 翻转图像-832

给定一个二进制矩阵 A，我们想先水平翻转图像，然后反转图像并返回结果。

水平翻转图片就是将图片的每一行都进行翻转，即逆序。例如，水平翻转 [1, 1, 0] 的结果是 [0, 1, 1]。

反转图片的意思是图片中的 0 全部被 1 替换， 1 全部被 0 替换。例如，反转 [0, 1, 1] 的结果是 [1, 0, 0]。

```html
输入: [[1,1,0],[1,0,1],[0,0,0]]
输出: [[1,0,0],[0,1,0],[1,1,1]]
解释: 首先翻转每一行: [[0,1,1],[1,0,1],[0,0,0]]；
     然后反转图片: [[1,0,0],[0,1,0],[1,1,1]]
```

**思路：**先依次翻转每一行，再取反。

```java
class Solution {
    public int[][] flipAndInvertImage(int[][] A) {
        int row = A.length, col = A[0].length;
        
        for(int i = 0; i < row; i++){
            for(int j = 0; j < col/2; j++){
                int temp = A[i][j];
                A[i][j] = A[i][col-j-1];
                A[i][col-j-1] = temp;
            }
        }       
        for(int i = 0; i < row; i++){
            for(int j = 0; j < col; j++){
                A[i][j] = 1-A[i][j];
            }
        }
        return A;
    }
}
```



```java
class Solution {
    public int[][] flipAndInvertImage(int[][] A) {
        for(int i = 0; i < A.length; i++){
            int index = A[i].length;
            int[] temp = new int[index];
            for(int j = 0; j<A[i].length; j++){
                temp[j] = A[i][--index] ^ 1 ;
            }
            A[i] = temp;
        }
        return A;
    }
}
```



```java
public int[][] flipAndInvertImage(int[][] A) {
    int m = A.length, n = A[0].length;
    for(int i = 0; i < m; i++){
        for(int j = 0 ; j < n/2; j++){
            int temp = A[i][j];
            A[i][j] = A[i][n -j -1] ^ 1;  
            A[i][n-j-1] = temp ^ 1;
        }
        if(n % 2 != 0){
            A[i][n/2] = A[i][n/2] ^ 1;
        }
    }
    return A;
}
```





#### 颜色分类-75

给定一个包含红色、白色和蓝色，一共 n 个元素的数组，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。

```
输入: [2,0,2,1,1,0]
输出: [0,0,1,1,2,2]
```

我们用三个指针（p0,  p2 和 curr）来分别追踪0的最右边界，2的最左边界和当前考虑的元素。

![5b3d372e0bfb293ca3aac12e90421d7612c9e75b78b579f954c42ebfe74705d4-image](E:\markdown笔记\图片\5b3d372e0bfb293ca3aac12e90421d7612c9e75b78b579f954c42ebfe74705d4-image.png)

本解法的思路是沿着数组移动 curr 指针，若nums[curr] = 0，则将其与 nums[p0]互换；若 nums[curr] = 2 ，则与 nums[p2]互换。

初始化0的最右边界：p0 = 0。在整个算法执行过程中 nums[idx < p0] = 0.

初始化2的最左边界 ：p2 = n - 1。在整个算法执行过程中 nums[idx > p2] = 2.

初始化当前考虑的元素序号 ：curr = 0.

While curr  <=  p2 :

1. 若 nums[curr] = 0 ：交换第 curr个 和 第p0个 元素，并将指针都向右移。
2. 若 nums[curr] = 2 ：交换第 curr个和第 p2个元素，并将 p2指针左移 。
3. 若 nums[curr] = 1 ：将指针 curr 右移。

```java
class Solution {
    public void sortColors(int[] nums) {
        int p0 = 0, cur = 0, p2 = nums.length -1;
        while(cur <= p2){
            if(nums[cur] == 0){
                int temp = nums[cur];
                nums[cur++] = nums[p0];
                nums[p0++] = temp;
            }else if(nums[cur] == 2){
                int temp = nums[cur];
                nums[cur] = nums[p2];
                nums[p2--] = temp;
            }else{
                cur++;
            }
        }
    }
}
```



#### 最长连续序列-128

给定一个未排序的整数数组，找出最长连续序列的长度，要求算法的时间复杂度为 *O(n)*。

```
输入: [100, 4, 200, 1, 3, 2]
输出: 4
解释: 最长连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

**排序**：时间复杂度 O ( nlogn )，空间复杂度 O ( 1 )

![48009532da24bc193697651f29266dee2c09f9fb4d9404846525ca2d4ff33cd8-image](E:\markdown笔记\图片\48009532da24bc193697651f29266dee2c09f9fb4d9404846525ca2d4ff33cd8-image.png)

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        if (nums.length == 0) return 0;
        Arrays.sort(nums);        
        int longestStreak = 1;
        int currentStreak = 1;
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] != nums[i-1]) {
                if (nums[i] == nums[i-1]+1)
                    currentStreak += 1;
                else {
                    longestStreak = Math.max(longestStreak, currentStreak);
                    currentStreak = 1;
                }
            }
        }
        return Math.max(longestStreak, currentStreak);
    }
}
```

**空间换时间：**

使用 HashSet 当遇到一个序列的开始，才开始计算序列的长度。可以达到 O ( 1 ) 的查询复杂度。

while 循环在整个运行过程中只会被迭代 nn 次。这意味着尽管看起来时间复杂度为 O (n⋅n) ，实际这个嵌套循环只会运行 O (n + n) = O (n)次。所有的计算都是线性时间的，所以总的时间复杂度是 O (n) 的。

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> num_set = new HashSet<Integer>();
        for (int num : nums) {
            num_set.add(num);
        }
        int longestStreak = 0;
        for (int num : num_set) {
            if (!num_set.contains(num-1)) { // 遇到一个序列的开始，才开始计算序列的长度。
                int currentNum = num;
                int currentStreak = 1;
                while (num_set.contains(currentNum+1)) {
                    currentNum += 1;
                    currentStreak += 1;
                }
                longestStreak = Math.max(longestStreak, currentStreak);
            }
        }
        return longestStreak;
    }
}
```

另一种空间换时间：

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> numsSet = new HashSet<>();
        for (Integer num : nums) {
            numsSet.add(num);
        }
        int longest = 0;
        for (Integer num : nums) {
            if (numsSet.remove(num)) {
                // 向当前元素的左边搜索,eg: 当前为100, 搜索：99，98，97,...
                int currentLongest = 1;
                int current = num;
                while (numsSet.remove(current - 1)) current--;
                currentLongest += (num - current);
				// 向当前元素的右边搜索,eg: 当前为100, 搜索：101，102，103,...
                current = num;
                while(numsSet.remove(current + 1)) current++;
                currentLongest += (current - num);
        		// 搜索完后更新longest.
                longest = Math.max(longest, currentLongest);
            }
        }
        return longest;
    }
}
```



#### 和为K的子数组-560

给定一个整数数组和一个整数 **k，**你需要找到该数组中和为 **k** 的连续的子数组的个数。

```html
输入:nums = [1,1,1], k = 2
输出: 2 , [1,1] 与 [1,1] 为两种不同的情况。
```

**暴力法：**时间复杂度 O(n2)，空间复杂度为 O(1)

```java
public class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;
        for (int start = 0; start < nums.length; start++) {
            int sum=0;
            for (int end = start; end < nums.length; end++) {
                sum += nums[end];
                if (sum == k) count++;
            }
        }
        return count;
    }
}
```

**HashMap:** 时间和空间复杂度均为 O(n)

一次遍历，会得到连续的每一个 num 的累加和，然后把累加和作为 key，记录在hashmap中，value域 为累加和出现的次数，则把 任意两个key做差等于K的value相加，则为最后结果

```java
public class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0, sum = 0;
        HashMap < Integer, Integer > map = new HashMap < > ();
        map.put(0, 1);
        for (int i = 0; i < nums.length; i++) {
            sum += nums[i];
            // 如果存在 sum-k 则从开始累加到当前的下标值与 累加到 sum-k的下标值 的差
            // 就是 sum-k 到 sum 下标之间的元素累加和为 K
            if (map.containsKey(sum - k))
                count += map.get(sum - k);
            map.put(sum, map.getOrDefault(sum, 0) + 1);
        }
        return count;
    }
}
```



#### 合并区间-56

给出一个区间的集合，请合并所有重叠的区间。

```
输入: [[1,3],[2,6],[8,10],[15,18]]
输出: [[1,6],[8,10],[15,18]]
解释: 区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
输入: [[1,4],[4,5]]
输出: [[1,5]]
解释: 区间 [1,4] 和 [4,5] 可被视为重叠区间。
```

先按首位置进行排序;

接下来，如何判断两个区间是否重叠呢？比如 a = [1,4]，b = [2,3]；当 a[1] >= b[0] 说明两个区间有重叠。但是如何把这个区间找出来呢？左边位置一定是确定，就是a[0]，而右边位置是 max (a[1]，b[1])，所以，我们就能找出整个区间为：[1,4] 。

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        List<int[]> res = new ArrayList<>();
        if (intervals.length == 0 || intervals == null)
            return res.toArray(new int[0][]);
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
        int i = 0;
        while (i < intervals.length) {
            int left = intervals[i][0];
            int right = intervals[i][1];
            while (i < intervals.length - 1 && intervals[i + 1][0] <= right) {
                i++;
                right = Math.max(right, intervals[i][1]);
            }
            res.add(new int[]{left, right});
            i++;
        }
        return res.toArray(new int[0][]);
    }
}
```



#### 根据身高重建队列-406

假设有打乱顺序的一群人站成一个队列。 每个人由一个整数对(h, k)表示，其中h是这个人的身高，k是排在这个人前面且身高大于或等于h的人数。 编写一个算法来重建这个队列。

```
输入:
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]

输出:
[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]
```

**思路：**

按照身高降序，K升序排列。K定义为排在 h 前面且身高大于等于 h 的人数，因为从身高降序开始插入，此时所有人身高都大于等于 h ，因此 K 值即为需要插入的位置。

```java
public int[][] reconstructQueue(int[][] people) {
    if (0 == people.length || 0 == people[0].length)
        return new int[0][0];
    //按照身高降序 K升序排序 
    Arrays.sort(people, new Comparator<int[]>() {
        @Override
        public int compare(int[] o1, int[] o2) {
            return o1[0] == o2[0] ? o1[1] - o2[1] : o2[0] - o1[0];
        }
    });
    List<int[]> list = new ArrayList<>();
    //K值定义为 排在h前面且身高大于或等于h的人数 
    //因为从身高降序开始插入，此时所有人身高都大于等于h
    //因此K值即为需要插入的位置
    for (int[] i : people) {
        list.add(i[1], i);
    }
    return list.toArray(new int[list.size()][]);
}
```



#### 查找常用字符-1002

给定仅有小写字母组成的字符串数组 A，返回列表中的每个字符串中都显示的全部字符（包括重复字符）组成的列表。例如，如果一个字符在每个字符串中出现 3 次，但不是 4 次，则需要在最终答案中包含该字符 3 次。你可以按任意顺序返回答案。

```
输入：["bella","label","roller"]
输出：["e","l","l"]

输入：["cool","lock","cook"]
输出：["c","o"]
```

**思路：**用哈希表存储每个字符串中字母出现的次数，然后遍历字符串数组，保存出现次数的最小值。最后保留的最小值即为每个字符出现的次数。

```java
class Solution {
    public List<String> commonChars(String[] A) {
        if(A == null || A.length == 0)
            return null;
        List<String> res = new ArrayList<>();
        int[][] countMap = new int[A.length][26];
        for(int i = 0; i < A.length; i++) 
            map(A[i], countMap[i]);
        
        for(int i = 0; i < 26; i++){
            int min = Integer.MAX_VALUE;
            for(int j = 0; j < A.length; j++){
                min = Math.min(min, countMap[j][i]);
            }
            while(min-- > 0){
                res.add(String.valueOf((char)(i+'a')));
            }
        }
        return res;
    }
     
    private void map(String s, int[] arr){
        for (int i = 0;i < s.length();i++) {
            arr[s.charAt(i) - 'a']++;
        }
    }
}
```



#### 除自身以外数组的乘积-238

给定长度为 n 的整数数组 nums，其中 n > 1，返回输出数组 output ，其中 output[i] 等于 nums 中除 nums[i] 之外其余各元素的乘积。

**说明:** 请**不要使用除法，**且在 O(*n*) 时间复杂度内完成此题。

```
输入: [1,2,3,4]
输出: [24,12,8,6]
```

**思路：**

第一遍从左到右扫描，将左边的数累乘；第二遍从右到左扫面，将右边的数累乘。

时间复杂度为 O (n)，空间复杂度为 O (1) 。

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int[] res = new int[nums.length];
        int k = 1;
        for(int i = 0; i < res.length; i++){
            res[i] = k;
            k = k * nums[i]; // 此时数组存储的是除去当前元素左边的元素乘积
        }
        k = 1;
        for(int i = res.length - 1; i >= 0; i--){
            res[i] *= k; // k为该数右边的乘积。
            k *= nums[i]; // 此时数组等于左边的 * 该数右边的。
        }
        return res;
    }
}
```



#### 生命游戏-289

根据百度百科，生命游戏，简称为生命，是英国数学家约翰·何顿·康威在1970年发明的细胞自动机。

给定一个包含 m × n 个格子的面板，每一个格子都可以看成是一个细胞。每个细胞具有一个初始状态 live（1）即为活细胞， 或 dead（0）即为死细胞。每个细胞与其八个相邻位置（水平，垂直，对角线）的细胞都遵循以下四条生存定律：

1. 如果活细胞周围八个位置的活细胞数少于两个，则该位置活细胞死亡；
2. 如果活细胞周围八个位置有两个或三个活细胞，则该位置活细胞仍然存活；
3. 如果活细胞周围八个位置有超过三个活细胞，则该位置活细胞死亡；
4. 如果死细胞周围正好有三个活细胞，则该位置死细胞复活；

根据当前状态，写一个函数来计算面板上细胞的下一个（一次更新后的）状态。下一个状态是通过将上述规则同时应用于当前状态下的每个细胞所形成的，其中细胞的出生和死亡是同时发生的。

```html
输入: 
[
  [0,1,0],
  [0,0,1],
  [1,1,1],
  [0,0,0]
]
输出: 
[
  [0,0,0],
  [1,0,1],
  [0,1,1],
  [0,1,0]
]
```

**解题思路：**

时间复杂度 O (m * n)，空间复杂度 O (1)；

1. 遍历二维数组，然后改变每个点的状态； 遍历取当前位置周围8个细胞，同时记录活细胞数量。如果x坐标或y坐标超出边界，则不记录； 
2. 把即将死亡的 活细胞 置为 2， 把即将复活的 死细胞 置为 -1； 则在判断 当前细胞 >0 时当前为活细胞，<= 0 当前为死细胞；
3. 根据题目要求，通过活细胞数量来改变该点的状态，最后，再遍历数组，将2 还原为 0，将 -1 还原为 1；

```java
public void gameOfLife(int[][] board) {
    for(int i = 0; i < board.length; i++){
        for(int j = 0; j < board[i].length; j++){
            changeBorder(board,i,j);
        }
    }

    for(int i = 0; i < board.length; i++){
        for(int j = 0; j < board[i].length; j++){
            if(board[i][j] == -1) board[i][j] = 1;
            if(board[i][j] == 2) board[i][j] = 0;
        }
    }
}

public void changeBorder(int[][] board,int x, int y){
    int row = board.length;
    int col = board[0].length;

    int total = 0; // 这是记录该细胞周围活细胞的数量

    for(int i= x - 1; i <= x + 1 ; i++){
        for(int j= y - 1; j <= y + 1; j++){
            if(i == x && j == y) continue;
            if(i >= 0 && i < row && j >= 0 && j < col){
                if(board[i][j] > 0) total ++;
            }
        }
    }
    if(board[x][y] > 0){
        if(total < 2 || total > 3) board[x][y] = 2;
    }
    if(board[x][y] <= 0 && total == 3) board[x][y] = -1;
}

```



#### 矩阵置零-73

给定一个 *m* x *n* 的矩阵，如果一个元素为 0，则将其所在行和列的所有元素都设为 0。请使用**原地**算法**。**

```
输入: 
[
  [0,1,2,0],
  [3,4,5,2],
  [1,3,1,5]
]
输出: 
[
  [0,0,0,0],
  [0,4,5,0],
  [0,3,1,0]
]
```

**思路一: 用 O(m+n) 额外空间**

两遍扫`matrix`,第一遍用集合记录哪些行,哪些列有`0`;第二遍置`0`

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        Set<Integer> row_zero = new HashSet<>();
        Set<Integer> col_zero = new HashSet<>();
        int row = matrix.length;
        int col = matrix[0].length;
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (matrix[i][j] == 0) {
                    row_zero.add(i);
                    col_zero.add(j);
                }
            }
        }
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (row_zero.contains(i) || col_zero.contains(j)) matrix[i][j] = 0;
            }
        }  
    }
}
```

**思路二：不使用额外的空间**

1. 遍历整个矩阵，如果 cell [i] [j] == 0 就将第 i 行和第 j 列的第一个元素标记。
2. 第一行和第一列的标记是相同的，都是 cell [0] [0]，所以需要一个额外的变量告知第一列是否被标记，同时用 cell [0] [0] 继续表示第一行的标记。
3. 然后，从第二行第二列的元素开始遍历，如果第 r 行或者第 c 列被标记了，那么就将 cell [r] [c] 设为 0。这里第一行和第一列的作用就相当于方法一中的 row_set 和 column_set 。
4. 然后我们检查是否 cell [0] [0] == 0 ，如果是则赋值第一行的元素为零。然后检查第一列是否被标记，如果是则赋值第一列的元素为零。

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        int row = matrix.length;
        int col = matrix[0].length;
        boolean row0_flag = false;
        boolean col0_flag = false;
        // 第一行是否有零
        for (int j = 0; j < col; j++) {
            if (matrix[0][j] == 0) {
                row0_flag = true;
                break;
            }
        }
        // 第一列是否有零
        for (int i = 0; i < row; i++) {
            if (matrix[i][0] == 0) {
                col0_flag = true;
                break;
            }
        }
        // 把第一行第一列作为标志位
        for (int i = 1; i < row; i++) {
            for (int j = 1; j < col; j++) {
                if (matrix[i][j] == 0) {
                    matrix[i][0] = matrix[0][j] = 0;
                }
            }
        }
        // 置0
        for (int i = 1; i < row; i++) {
            for (int j = 1; j < col; j++) {
                if (matrix[i][0] == 0 || matrix[0][j] == 0) {
                    matrix[i][j] = 0;
                }
            }
        }
        if (row0_flag) {
            for (int j = 0; j < col; j++) {
                matrix[0][j] = 0;
            }
        }
        if (col0_flag) {
            for (int i = 0; i < row; i++) {
                matrix[i][0] = 0;
            }
        } 
    }
}
```

**简化版：**

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        boolean col0_flag = false;
        int row = matrix.length;
        int col = matrix[0].length;
        for (int i = 0; i < row; i++) {
            if (matrix[i][0] == 0) col0_flag = true;
            for (int j = 1; j < col; j++) {
                if (matrix[i][j] == 0) {
                    matrix[i][0] = matrix[0][j] = 0;
                }
            }
        }
        for (int i = row - 1; i >= 0; i--) {
            for (int j = col - 1; j >= 1; j--) {
                if (matrix[i][0] == 0 || matrix[0][j] == 0) {
                    matrix[i][j] = 0;
                }
            }
            if (col0_flag) matrix[i][0] = 0;
        }
    }
}
```



#### 加油站-134

在一条环路上有 N 个加油站，其中第 i 个加油站有汽油 gas[i] 升。

你有一辆油箱容量无限的的汽车，从第 i 个加油站开往第 i+1 个加油站需要消耗汽油 cost[i] 升。你从其中的一个加油站出发，开始时油箱为空。

如果你可以绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1。

说明: 

1. 如果题目有解，该答案即为唯一答案。
2. 输入数组均为非空数组，且长度相同。
3. 输入数组中的元素均为非负数。

```
输入: 
gas  = [1,2,3,4,5]
cost = [3,4,5,1,2]
输出: 3
解释:
从 3 号加油站(索引为 3 处)出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
开往 3 号加油站，你需要消耗 5 升汽油，正好足够你返回到 3 号加油站。
因此，3 可为起始索引。
```

**思路：**

如果 `sum(gas) < sum(cost)` ，那么不可能环行一圈，这种情况下答案是 `-1` 。我们可以用这个式子计算环行过程中邮箱里剩下的油：total_tank = sum(gas) - sum(cost) ，如果 total_tank < 0 则返回 -1 。

对于加油站 `i` ，如果 `gas[i] - cost[i] < 0` ，则不可能从这个加油站出发，因为在前往 `i + 1` 的过程中，汽油就不够了。

我们引入变量 `curr_tank` ，记录当前油箱里剩余的总油量。如果在某一个加油站 `curr_tank`比 `0` 小，意味着我们无法到达这个加油站。

1. 初始化 total_tank 和 curr_tank 为 0 ，并且选择 0 号加油站为起点，遍历所有的加油站。
2. 每一步中，都通过加上 gas[i] 和减去 cost[i] 来更新 total_tank 和 curr_tank 。
3. 如果在 i + 1 号加油站， curr_tank < 0 ，将 i + 1 号加油站作为新的起点，同时重置 curr_tank = 0 ，让油箱也清空。
4. 如果 total_tank < 0 ，返回 -1 ，否则返回 starting station。

```java
class Solution {
      public int canCompleteCircuit(int[] gas, int[] cost) {
            int n = gas.length;
            int total_tank = 0;
            int curr_tank = 0;
            int starting_station = 0;
            for (int i = 0; i < n; ++i) {
                  total_tank += gas[i] - cost[i];
                  curr_tank += gas[i] - cost[i];
                  // If one couldn't get here,
                  if (curr_tank < 0) {
                        // Pick up the next station as the starting one.
                        starting_station = i + 1;
                        // Start with an empty tank.
                        curr_tank = 0;
                  }
            }
            return total_tank >= 0 ? starting_station : -1;
      }
}
```



#### 打乱数组-384

打乱一个没有重复元素的数组。

```html
// 以数字集合 1, 2 和 3 初始化数组。
int[] nums = {1,2,3};
Solution solution = new Solution(nums);

// 打乱数组 [1,2,3] 并返回结果。任何 [1,2,3]的排列返回的概率应该相同。
solution.shuffle();

// 重设数组到它的初始状态[1,2,3]。
solution.reset();

// 随机返回数组[1,2,3]打乱后的结果。
solution.shuffle();
```

**洗牌算法：**

此类算法都是靠随机选取元素交换来获取随机性，直接看代码（伪码），该算法有 4 种形式，都是正确的：

```java
// 得到一个在闭区间 [min, max] 内的随机整数
int randInt(int min, int max);

// 第一种写法
void shuffle(int[] arr) {
    int n = arr.length();
    /******** 区别只有这两行 ********/
    for (int i = 0 ; i < n; i++) {
        // 从 i 到最后随机选一个元素
        int rand = randInt(i, n - 1);
        /*************************/
        swap(arr[i], arr[rand]);
    }
}

// 第二种写法
    for (int i = 0 ; i < n - 1; i++)
        int rand = randInt(i, n - 1);

// 第三种写法
    for (int i = n - 1 ; i >= 0; i--)
        int rand = randInt(0, i);

// 第四种写法
    for (int i = n - 1 ; i > 0; i--)
        int rand = randInt(0, i);
```

**分析洗牌算法正确性的准则：产生的结果必须有 n! 种可能，否则就是错误的。**这个很好解释，因为一个长度为 n 的数组的全排列就有 n! 种，也就是说打乱结果总共有 n! 种。算法必须能够反映这个事实，才是正确的。

```java
class Solution {
    private int[]start;
    public Solution(int[] nums) {
        start = nums;
    }
    
    /** Resets the array to its original configuration and return it. */
    public int[] reset() {
        return start;
    }
    
    /** Returns a random shuffling of the array. */
    public int[] shuffle() {
        Random r = new Random();
        int[] copy = start.clone();
        int len = copy.length;
        int i = 0;
        while(len > 1){
            len--;
            int k = r.nextInt(len+1);
            int v = copy[k];
            copy[k] = copy[len];
            copy[len] = v;
        }
        return copy;
    }
}
```

```java
class Solution {
    int[] start;
    int[] exch;
    int len;
    public Solution(int[] nums) {
        start = nums;
        len = nums.length;
        exch = start.clone();
    }
    
    /** Resets the array to its original configuration and return it. */
    public int[] reset() {
        return start;
    }
    
    /** Returns a random shuffling of the array. */
    public int[] shuffle() {
        Random random = new Random();
        int n = len;
        for(int i = n - 1; i > 0; i--){
            int x = random.nextInt(i+1);
            swap(exch, x, i);
        }
        return exch;
    }
    
    private void swap(int[] arr, int x, int y){
        int temp = arr[x];
        arr[x] = arr[y];
        arr[y] = temp;
    }
}
```



#### 单调数列-896

如果数组是单调递增或单调递减的，那么它是单调的。如果对于所有 i <= j，A[i] <= A[j]，那么数组 A 是单调递增的。 如果对于所有 i <= j，A[i]> = A[j]，那么数组 A 是单调递减的。当给定的数组 A 是单调数组时返回 true，否则返回 false。

```
输入：[1,2,2,3]
输出：true

输入：[6,5,4,4]
输出：true
```

**思路：**用两个变量记录递增和递减数列的长度，如果长度等于数组长度，则是单调数列。

```java
class Solution {
    public boolean isMonotonic(int[] A) {
        if(A.length <= 2) return true;
        int count1 = 1, count2 = 1;
        for(int i = 1; i < A.length; i++){
            if(A[i] > A[i-1]) count1++;
            else if(A[i] < A[i-1]) count2++;
            else{
                count1++;
                count2++;
            }
        }
        return count1 == A.length || count2 == A.length;
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
    int i = 1;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i-1]) {
            nums[i++] = nums[j];
        }
    }
    return i;
}
```



#### 删除排序数组中的重复项-II-80

给定一个排序数组，你需要在原地删除重复出现的元素，使得每个元素最多出现两次，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

```html
给定 nums = [1,1,1,2,2,3],
函数应返回新长度 length = 5, 并且原数组的前五个元素被修改为 1, 1, 2, 2, 3 。


给定 nums = [0,0,1,1,1,1,2,3,3],
函数应返回新长度 length = 7, 并且原数组的前五个元素被修改为 0, 0, 1, 1, 2, 3, 3 。

你不需要考虑数组中超出新长度后面的元素。
```

**思路：**和移除重复元素相同，使用双指针。扩展性较强，如果将 occur < 2 改为   occur<3  ，就变成了允许重复最多三次。

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if(nums.length < 2) return 2;
        int i = 2;
        for(int j = 2; j < nums.length; j++){
            if(nums[j] != nums[i-2])
                nums[i++] = nums[j];
        }
        return i;
    }
}
```



#### 数组中重复的数据-442

给定一个整数数组 a，其中1 ≤ a[i] ≤ n （n为数组长度）, 其中有些元素出现两次而其他元素出现一次。找到所有出现两次的元素。

你可以不用到任何额外空间并在O(n)时间复杂度内解决这个问题吗？

```
输入:
[4,3,2,7,8,2,3,1]

输出:
[2,3]
```

**思路1：**使用额外的空间

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> res = new ArrayList();
        int[] count = new int[nums.length+1];
        for(int x : nums){
            count[x]++;
            if(count[x] == 2) res.add(x);
            
        }
        return res;
    }
}
```

**思路2：**因为数组元素范围为[1,n]，所以选取数组的索引作为基准，遍历数组，将当前元素对应索引位置的元素取负数，这样如果元素出现了两次，第二个元素访问对应索引位置元素会是负数。

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> res = new ArrayList();
        for(int num : nums){
            int index = Math.abs(num);
            if(nums[index-1] < 0) res.add(index);
            else nums[index-1] *= -1;
        }
        return res;
    }
}
```

```java
class Solution {
    public int findDuplicates(int[] nums) {
        for(int num : nums){
            int index = Math.abs(num);
            if(nums[index-1] < 0) return index;
            else nums[index-1] *= -1;
        }
        return -1;
    }
}
```



#### 查询后的偶数和-985

给出一个整数数组 A 和一个查询数组 queries。

对于第 i 次查询，有 `val = queries [i] [0]，index = queries [i] [1]，我们会把 val 加到 A[index] 上。然后，第 i 次查询的答案是 A 中偶数值的和。

（此处给定的 index = queries[i][1] 是从 0 开始的索引，每次查询都会永久修改数组 A。）

返回所有查询的答案。你的答案应当以数组 answer 给出，answer[i] 为第 i 次查询的答案。

```html
输入：A = [1,2,3,4], queries = [[1,0],[-3,1],[-4,0],[2,3]]
输出：[8,6,2,4]
解释：
开始时，数组为 [1,2,3,4]。
将 1 加到 A[0] 上之后，数组为 [2,2,3,4]，偶数值之和为 2 + 2 + 4 = 8。
将 -3 加到 A[1] 上之后，数组为 [2,-1,3,4]，偶数值之和为 2 + 4 = 6。
将 -4 加到 A[0] 上之后，数组为 [-2,-1,3,4]，偶数值之和为 -2 + 4 = 2。
将 2 加到 A[3] 上之后，数组为 [-2,-1,3,6]，偶数值之和为 -2 + 6 = 4。
```

**思路：**

让我们尝试不断调整 S，即每一步操作之后整个数组的偶数和。

我们操作数组中的某一个元素 A[index] 的时候，数组 A 其他位置的元素都应保持不变。如果 A[index] 是偶数，我们就从 S 中减去它，然后计算 A[index] + val 对 S 的影响（如果是偶数则在 S 中加上它），

是奇数则保持不变。

```java
class Solution {
    public int[] sumEvenAfterQueries(int[] A, int[][] queries) {
        int s = 0;
        for(int i : A){
            if(i % 2 == 0)
                s += i;
        }
        int[] res = new int[A.length];
        int j = 0;
        for(int[] q : queries){
            int index = q[1];
            int val = q[0];
            if(A[index] % 2 == 0) s -= A[index];
            A[index] += val;
            if(A[index] % 2 == 0) s += A[index];
            res[j++] = s;
        }
        return res;
    }
}
```



#### 复写零-1089

给你一个长度固定的整数数组 arr，请你将该数组中出现的每个零都复写一遍，并将其余的元素向右平移。

注意：请不要在超过该数组长度的位置写入元素。

要求：请对输入的数组 就地 进行上述修改，不要从函数返回任何东西。

```html
输入：[1,0,2,3,0,4,5,0]
输出：null
解释：调用函数后，输入的数组将被修改为：[1,0,0,2,3,0,0,4]

输入：[1,0,2,3,0,4,5,0]
输出：null
解释：调用函数后，输入的数组将被修改为：[1,0,0,2,3,0,0,4]
```

**思路1：**使用额外的空间

```java
class Solution {
    public void duplicateZeros(int[] arr) {
       	int copy = arr.clone();
        int len = arr.length;
        int index = 0;
        for(int i : copy){
            if(index >= len) break;
            if(i == 0 && index + 1 < len){
                arr[index++] = 0;
                arr[index++] = 0;
            }else{
                arr[index++] = copy[i];
            }
        }
    }
}
```

**思路2：**原地修改

```java
class Solution {
    public void duplicateZeros(int[] arr) {
        for(int i = 0; i < arr.length; i++){
            if(arr[i] == 0){
                for(int j = arr.length - 1; j > i; j--){
                	arr[j] = arr[j-1];
            	}
                i++;
            }
        }
    }
}
```



#### 最长递增序列-674

给定一个未经排序的整数数组，找到最长且**连续**的的递增序列。

```
输入: [1,3,5,4,7]
输出: 3
解释: 最长连续递增序列是 [1,3,5], 长度为3。
尽管 [1,3,5,7] 也是升序的子序列, 但它不是连续的，因为5和7在原数组里被4隔开。 
```

遍历：

```java
class Solution {
    public int findLengthOfLCIS(int[] nums) {
        if(nums.length <= 1) return nums.length;
        int res = 1, count = 1;
        for(int i = 1; i < nums.length; i++){
            if(nums[i] > nums[i-1]) count++;
            else count = 1;
            res = Math.max(res, count);
        }
        return res;
    }
}
```



#### 可被5整除的二进制前缀-1018

给定由若干 0 和 1 组成的数组 A。我们定义 N_i：从 A[0] 到 A[i] 的第 i 个子数组被解释为一个二进制数（从最高有效位到最低有效位）。

返回布尔值列表 answer，只有当 N_i 可以被 5 整除时，答案 answer[i] 为 true，否则为 false。

```
输入：[0,1,1]
输出：[true,false,false]
解释：
输入数字为 0, 01, 011；也就是十进制中的 0, 1, 3 。只有第一个数可以被 5 整除，因此 answer[0] 为真。

输入：[0,1,1,1,1,1]
输出：[true,false,false,false,true,false]

输入：[1,1,1,0,1]
输出：[false,false,false,false,false]
```

**思路：**采用十进制对5取模会超出位数，如果上一个数就能被 5 整除，那么再乘以 2 还是能被 5 整除，因此每次判断之后，将这个数对 5 取模。

```java
class Solution {
    public List<Boolean> prefixesDivBy5(int[] A) {
        List<Boolean> res = new ArrayList();
        if(A == null || A.length == 0) return res;
        int sum = 0;
        for(int i = 0; i < A.length; i++){
            sum = sum * 2 + A[i];
            res.add(sum % 5 == 0);
            sum = sum % 5;
        }
        return res;     
    }
}
```

使用位运算优化：

```java
class Solution {
    public List<Boolean> prefixesDivBy5(int[] A) {
        List<Boolean> res = new ArrayList();
        if(A == null || A.length == 0) return res;
        int sum = 0;
        for(int i = 0; i < A.length; i++){
            sum = (sum << 1 | A[i]) % 5;
            res.add(sum == 0);
        }
        return res;     
    }
}
```







#### 图片平滑器-661

包含整数的二维矩阵 M 表示一个图片的灰度。你需要设计一个平滑器来让每一个单元的灰度成为平均灰度 (向下舍入) ，平均灰度的计算是周围的8个单元和它本身的值求平均，如果周围的单元格不足八个，则尽可能多的利用它们。

```html
输入:
[[1,1,1],
 [1,0,1],
 [1,1,1]]
输出:
[[0, 0, 0],
 [0, 0, 0],
 [0, 0, 0]]
解释:
对于点 (0,0), (0,2), (2,0), (2,2): 平均(3/4) = 平均(0.75) = 0
对于点 (0,1), (1,0), (1,2), (2,1): 平均(5/6) = 平均(0.83333333) = 0
对于点 (1,1): 平均(8/9) = 平均(0.88888889) = 0
```

**思路：**遍历，直接计算周围的8个单元和它本身的值求平均即可。 设置偏移标志数组。

```java
class Solution {
    public int[][] imageSmoother(int[][] M) {
        if(M == null || M.length == 0) return null;
        int row = M.length, col = M[0].length;
        int[][] res = new int[row][col];
        for(int i = 0; i < row; i++){
            for(int j = 0; j < col; j++){
                res[i][j] = cacul(M, row, col, i, j);
            }
        }
        return res;
    }
    
    int[] R = new int[]{-1,1,0,0,-1,1,-1,1};
    int[] C = new int[]{0,0,-1,1,-1,-1,1,1};
    
    public int cacul(int[][] arr, int row, int col, int i, int j){
        
        int count = 1;
        int sum = arr[i][j];
        
        for(int k = 0; k < R.length; k++){
            int nextR = i + R[k];
            int nextC = j + C[k];
            if(nextR < row && nextR >= 0 && nextC < col && nextC >= 0){
                count++;
                sum += arr[nextR][nextC];
            }
        }
        return sum / count;
    }
}
```



#### 1比特和2比特字符-717

有两种特殊字符。第一种字符可以用一比特0来表示。第二种字符可以用两比特(10 或 11)来表示。现给一个由若干比特组成的字符串。问最后一个字符是否必定为一个一比特字符。给定的字符串总是由0结束。

```
输入: 
bits = [1, 0, 0]
输出: True
解释: 
唯一的编码方式是一个两比特字符和一个一比特字符。所以最后一个字符是一比特字符。

输入: 
bits = [1, 1, 1, 0]
输出: False
解释: 
唯一的编码方式是两比特字符和两比特字符。所以最后一个字符不是一比特字符
```

**思路1：**我们可以对 bits 数组从左到右扫描来判断最后一位是否为一比特字符。当扫描到第 i 位时，如果 bits[i]=1，那么说明这是一个两比特字符，将 i 的值增加 2。如果 bits[i]=0，那么说明这是一个一比特字符，将 i 的值增加 1。

如果 i 最终落在了bits.length−1 的位置，那么说明最后一位一定是一比特字符。

```java
class Solution {
    public boolean isOneBitCharacter(int[] bits) {
        int i = 0;
        while (i < bits.length - 1) {
            i += bits[i] + 1;
        }
        return i == bits.length - 1;
    }
}
```

**思路2：**贪心，三种字符分别为 0，10 和 11，那么 bits 数组中出现的所有 0 都表示一个字符的结束位置（无论其为一比特还是两比特）。因此最后一位是否为一比特字符，只和他左侧出现的连续的 1 的个数（即它与倒数第二个 0 出现的位置之间的 1 的个数，如果 bits 数组中只有 1 个 0，那么就是整个数组的长度减一）有关。如果 1 的个数为偶数个，那么最后一位是一比特字符，如果 1 的个数为奇数个，那么最后一位不是一比特字符。

```java
class Solution {
    public boolean isOneBitCharacter(int[] bits) {
        int i = bits.length - 2;
        while (i >= 0 && bits[i] > 0) i--;
        return (bits.length - i) % 2 == 0;
    }
}
```





#### 公平的糖果交换-888

爱丽丝和鲍勃有不同大小的糖果棒：A[i] 是爱丽丝拥有的第 i 块糖的大小，B[j] 是鲍勃拥有的第 j 块糖的大小。

因为他们是朋友，所以他们想交换一个糖果棒，这样交换后，他们都有相同的糖果总量。（一个人拥有的糖果总量是他们拥有的糖果棒大小的总和。）

返回一个整数数组 ans，其中 ans[0] 是爱丽丝必须交换的糖果棒的大小，ans[1] 是 Bob 必须交换的糖果棒的大小。

如果有多个答案，你可以返回其中任何一个。保证答案存在。

```
输入：A = [1,1], B = [2,2]
输出：[1,2]

输入：A = [2], B = [1,3]
输出：[2,3]
```

**思路：**

设爱丽丝和鲍勃分别总计有 sumA，sumB 的糖果。如果爱丽丝给了糖果 x，并且收到了糖果 y，那么鲍勃收到糖果 x 并给出糖果 y。那么，我们一定有 sumA - x + y = sumB - y + x，对于公平的糖果交换。这意味着：

y = x + (sumB - sumA) / 2，对于爱丽丝拥有的每个糖果 x，如果鲍勃有糖果 y = x + (sumB - sumA) / 2
我们就返回 x，y。我们在常量时间内使用集 Set 结构来检查 Bob 是否拥有所需的糖果 y。

时间复杂度为：O (A.length + B.length)

空间复杂度为：O (B.length)

```java
class Solution {
    public int[] fairCandySwap(int[] A, int[] B) {
        int sumA = 0, sumB = 0;
        for(int x : A) sumA += x;
        for(int x : B) sumB += x;
        int diff = (sumB - sumA) >> 1;
        Set<Integer> set = new HashSet();
        for(int x : B) set.add(x);
        for(int x : A){
            if(set.contains(x + diff)){
                return new int[]{x, x + diff};
            }
        }
        return null;
    }
}
```

也可以先排序，然后进行二分查找：

```java
public int[] fairCandySwap(int[] A, int[] B) {
    int Asum = 0, Bsum = 0;
    for(int i : A) Asum += i;
	for(int i : B) Bsum += i;
    Arrays.sort(A);
    Arrays.sort(B);
    int temp = (Asum - Bsum) / 2;
    int[] ans = new int[2];
    int i = 0, j = 0;
    while(i < A.length && j < B.length) {
        if(A[i] - B[j] == temp) {
            ans[0] = A[i];
            ans[1] = B[j];
            break;
        } else if(A[i] - B[j] > temp) {
            j++;
        } else {
            i++;
        }
    }  
    return ans;
}
```





#### 合并两个有序数组-88

给定两个有序整数数组 nums1 和 nums2，将 nums2 合并到 nums1 中，使得 num1 成为一个有序数组。

**说明:**

1. 初始化 nums1 和 nums2 的元素数量分别为 m 和 n。
2. 你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。

```
输入:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3
输出: [1,2,2,3,5,6]
```

**双指针：**从后向前

- 时间复杂度 : O(n + m)
- 空间复杂度 : O(1)

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int k = m + n -1;
        int i = m-1, j = n-1; 
        
        while(i >= 0 && j >= 0){
            if(nums1[i] > nums2[j]) nums1[k--] = nums1[i--];
            else nums1[k--] = nums2[j--];
        }
        while(i >= 0) nums1[k--] = nums1[i--];
        while(j >= 0) nums1[k--] = nums2[j--];
    }
}
```



#### 按递增顺序显示卡牌-950

牌组中的每张卡牌都对应有一个唯一的整数。你可以按你想要的顺序对这套卡片进行排序。

最初，这些卡牌在牌组里是正面朝下的（即，未显示状态）。

现在，重复执行以下步骤，直到显示所有卡牌为止：

1. 从牌组顶部抽一张牌，显示它，然后将其从牌组中移出。
2. 如果牌组中仍有牌，则将下一张处于牌组顶部的牌放在牌组的底部。
3. 如果仍有未显示的牌，那么返回步骤 1。否则，停止行动。
4. 返回能以递增顺序显示卡牌的牌组顺序。

答案中的第一张牌被认为处于牌堆顶部。

```
输入：[17,13,11,2,3,5,7]
输出：[2,13,3,11,5,17,7]
解释：
我们得到的牌组顺序为 [17,13,11,2,3,5,7]（这个顺序不重要），然后将其重新排序。
重新排序后，牌组以 [2,13,3,11,5,17,7] 开始，其中 2 位于牌组的顶部。
我们显示 2，然后将 13 移到底部。牌组现在是 [3,11,5,17,7,13]。
我们显示 3，并将 11 移到底部。牌组现在是 [5,17,7,13,11]。
我们显示 5，然后将 17 移到底部。牌组现在是 [7,13,11,17]。
我们显示 7，并将 13 移到底部。牌组现在是 [11,17,13]。
我们显示 11，然后将 17 移到底部。牌组现在是 [13,17]。
我们展示 13，然后将 17 移到底部。牌组现在是 [17]。
我们显示 17。
由于所有卡片都是按递增顺序排列显示的，所以答案是正确的。
```

**思路：**此题，乍一眼看上去，一点规律没有，甚至示例看起来都觉得莫名其妙，相信很多人看了这道题都会像我这么觉得，所以此题归属于明显的阅读理解加脑筋急转弯题型。

但是，如果我们从另一个角度去看此题，就会发现此题的意思所在了。于是我们尝试着倒着看示例，我们有一个数组，数组中元素不重复，且按照降序排列，我们称之为卡牌，初始值为[17,13,11,7,5,3,2]，我们有另一个队列，初始为空，我们每次将数组中未放入队列中的最大的数插入到队列中，然后我们把队首元素移到队尾，重复这个过程，直到数组中所有元素都被放入队列中为止。
第一次，我们选17，[17]->[17]
第二次，我们选13，[17, 13]->[13, 17]
第三次，我们选11，[13, 17, 11]->[17, 11, 13]
第四次，我们选7，[17, 11, 13, 7]->[11, 13, 7, 17]
第五次，我们选5，[11, 13, 7, 17, 5]->[13, 7, 17, 5, 11]
第六次，我们选3，[13, 7, 17, 5, 11, 3]->[7, 17, 5, 11, 3, 13]
第七次，我们选2，[7, 17, 5, 11, 3, 13, 2]，此时，数组中所有元素都在队列中，过程结束。我们需要将这个队列倒过来，于是我们便称这样的队列，是按递增顺序显示卡牌。

于是解题代码如下：

```java
class Solution {
    public int[] deckRevealedIncreasing(int[] deck) {
        if (deck == null || deck.length < 1) {
            return deck;
        }        
        Arrays.sort(deck);// 得到升序排列的数组      
        Queue<Integer> queue = new LinkedList<>();
        for (int i = deck.length - 1; i >= 0; i--) {// 倒着遍历，便是按降序访问
            queue.add(deck[i]);// 选最大值插入队列
            if (i == 0) {// 数组中所有元素均在队列中，退出过程
                break;
            }          
            queue.add(queue.poll());// 将队头元素插入到队尾中
        }
        for (int i = deck.length - 1;i >= 0;i--) {
            deck[i] = queue.poll();// 倒回去，得到answer
        }
        
        return deck;
    }
}
```



#### 最短无序连续子数组-581

给定一个整数数组，你需要寻找一个**连续的子数组**，如果对这个子数组进行升序排序，那么整个数组都会变为升序排序。你找到的子数组应是**最短**的，请输出它的长度。

```
输入: [2, 6, 4, 8, 10, 9, 15]
输出: 5
解释: 你只需要对 [6, 4, 8, 10, 9] 进行升序排序，那么整个表都会变为升序排序。
```

**思路：**

如果最右端的一部分已经排好序，这部分的每个数都比它左边的最大值要大，同理，如果最左端的一部分排好序，这每个数都比它右边的最小值小。

所以我们从左往右遍历，如果 i 位置上的数比它左边部分最大值小，则这个数肯定要排序， 就这样找到右端不用排序的部分，同理找到左端不用排序的部分，它们之间就是需要排序的部分

```java
class Solution {
    public int findUnsortedSubarray(int[] arr) {
        if(arr == null || arr.length < 2){
			return 0;
		}
		int max = Integer.MIN_VALUE;
		int min = Integer.MAX_VALUE;
		int R = 0;
		int L = 0;
		for (int i = 0; i < arr.length; i++) {
			if(max > arr[i]) {
				R = i;
			}
			max = Math.max(max, arr[i]);
		}
		for (int i = arr.length - 1; i >= 0; i--) {
			if(min < arr[i]) {
				L = i;
			}
			min = Math.min(min, arr[i]);
		}
		return R == L ? 0 : R - L + 1;	
    }
}
```



#### 两个数组的交集II-350

给定两个数组，编写一个函数来计算它们的交集。

```
输入: nums1 = [1,2,2,1], nums2 = [2,2]
输出: [2,2]
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [4,9]
```

**思路：**使用 HashMap 求交集

```java
import java.util.*;
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        Map<Integer,Integer> map = new HashMap<>();
        for(int i = 0; i < nums1.length; i++){
            map.put(nums1[i],map.getOrDefault(nums1[i],0)+1);
        }
        List<Integer> list = new ArrayList<>();
        for(int i = 0; i < nums2.length; i++){
            if(map.containsKey(nums2[i]) && map.get(nums2[i]) > 0){
                list.add(nums2[i]);
                map.put(nums2[i], map.get(nums2[i])-1);
            }
        }
        int[] res = new int[list.size()];
        for(int i = 0; i < res.length; i++){
            res[i] = list.get(i);
        }
        return res;
    }
}
```

2：使用数组代替 HashMap：

```java
public int[] intersect(int[] nums1, int[] nums2) {
    
    int max = Integer.MIN_VALUE, min = Integer.MAX_VALUE;
    for (int i = 0; i < nums1.length; i++) {          
        min = Math.min(min, nums1[i]);
        max = Math.max(max, nums1[i]);
    }
    for (int i = 0; i < nums2.length; i++) {
        min = Math.min(min, nums2[i]);
        max = Math.max(max, nums2[i]);
    }
    int[] count = new int[max-min+1];
    for (int i = 0; i < nums1.length; i++) {
        count[nums1[i]-min]++;
    }
     List<Integer> list = new ArrayList<Integer>();
     for (int i = 0; i < nums2.length; i++) {
         if(count[nums2[i]-min] > 0) {
             count[nums2[i]-min]--;
             list.add(nums2[i]);
         }
     }    
    int[] res = new int[list.size()];
    for(int i = 0; i < res.length; i++) res[i] = list.get(i);
    return res;   
}
```

3：将数组转换为集合：

```java
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        List<Integer> list1 = new ArrayList<>();
        for (int a : nums1) {
            list1.add(a);
        }
        int a[] = new int[list1.size()];
        int temp = -1;
        for (int i = 0; i < nums2.length; i++) {
            if (list1.contains(nums2[i])) {
                temp++;
                a[temp] = nums2[i];
                Integer x = nums2[i];
                list1.remove(x);
            }
        }
        return Arrays.copyOf(a, temp + 1);
    }
}
```

4：归一化处理，先排序，在求重复

```java
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        List<Integer>temp = new ArrayList<>();
        Arrays.sort(nums1);
        Arrays.sort(nums2);
        int i = 0, j = 0;
        
        while (i < nums1.length && j < nums2.length) {
            if(nums1[i] < nums2[j]) i++;
            else if(nums1[i] > nums2[j]) j++;
            else {
                temp.add(nums1[i]);
                i++;
                j++;
            }
        }
        int[] res = new int[temp.size()];
        for(int m = 0; m < res.length; m++) res[m] = temp.get(m);
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



#### 加一-66

给定一个由整数组成的非空数组所表示的非负整数，在该数的基础上加一。最高位数字存放在数组的首位， 数组中每个元素只存储一个数字。你可以假设除了整数 0 之外，这个整数不会以零开头。

```
输入: [1,2,3]
输出: [1,2,4]
解释: 输入数组表示数字 123。
输入: [4,3,2,1]
输出: [4,3,2,2]
解释: 输入数组表示数字 4321。
```

**思路：**从后向前每次加一，如果没有出现 0 说明没有进位，可以直接返回，否则就把第一位置 0 且数组长度加一。

```java
class Solution {
    public int[] plusOne(int[] digits) {
        if(digits == null || digits.length == 0) return null;
        for(int i = digits.length -1; i >= 0; i--){
            digits[i]++;
            digits[i] %= 10;
            if(digits[i] != 0) return digits;
        }
        digits = new int[digits.length+1];
        digits[0] = 1;
        return digits;
    }
}
```



#### 数组形式的整数加法-989

对于非负整数 X 而言，X 的数组形式是每位数字按从左到右的顺序形成的数组。例如，如果 X = 1231，那么其数组形式为 [1,2,3,1]。给定非负整数 X 的数组形式 A，返回整数 X+K 的数组形式。

```
输入：A = [1,2,0,0], K = 34
输出：[1,2,3,4]
解释：1200 + 34 = 1234

输入：A = [2,7,4], K = 181
输出：[4,5,5]
解释：274 + 181 = 455

输入：A = [2,1,5], K = 806
输出：[1,0,2,1]
解释：215 + 806 = 1021
```

**思路：**将整个加数加入数组表示的数的最低位，然后依次计算

时间复杂度：O(max(N,logK))，其中 N 是数组 A 的长度。

空间复杂度：O(max(N,logK))。

```java
class Solution {
    public List<Integer> addToArrayForm(int[] A, int K) {
        // 这里写成 List<Integer> 就会报错，不知道为什么？？？
        LinkedList<Integer> res = new LinkedList<>();
        int cur = K;
        int i = A.length-1;
        while(i >= 0 || cur > 0){
            if(i >= 0) cur += A[i];
            res.addFirst(cur % 10);
            cur /= 10;
            i--;
        }
        return res;
    }
}
```



#### 到最近的人的最大距离-849

在一排座位（ seats）中，1 代表有人坐在座位上，0 代表座位上是空的。

至少有一个空座位，且至少有一人坐在座位上。

亚历克斯希望坐在一个能够使他与离他最近的人之间的距离达到最大化的座位上。

返回他到离他最近的人的最大距离。

```html
输入：[1,0,0,0,1,0,1]
输出：2
解释：
如果亚历克斯坐在第二个空位（seats[2]）上，他到离他最近的人的距离为 2 。
如果亚历克斯坐在其它任何一个空位上，他到离他最近的人的距离为 1 。
因此，他到离他最近的人的最大距离是 2 。 

输入：[1,0,0,0]
输出：3
解释： 
如果亚历克斯坐在最后一个座位上，他离最近的人有 3 个座位远。
这是可能的最大距离，所以答案是 3 。
```

**思路：**由三种情况 1.统计左端点开始全是 0 的情况；2.统计右端点开始全是 0 的情况；3.统计中间 0 的最大长度，按长度为奇数和偶数分别考虑；

然后求出，1，2，3 的最大值，即为最大距离

```java
class Solution {
    public int maxDistToClosest(int[] seats) {
        int p = 0, q = seats.length - 1;
        while(seats[p] == 0) p++;
        int max = p;
        while(seats[q] == 0) q--;
        max = Math.max(max, seats.length - q -1);
        int count = 0, temp = 0;
        for(int i = p; i <= q; i++){
            if(seats[i] == 0) temp++;
            else temp = 0;
            count = Math.max(count, temp);
        }
        if(count % 2 == 0) max = Math.max(max, count / 2);
        else max = Math.max(max, count / 2 + 1);
        return max;
    }
}
```

另一种写法：

```java
class Solution {
    public int maxDistToClosest(int[] seats) {
        if (seats == null || seats.length == 0)
            return 0;
        int p = 0;
        int q = seats.length - 1;
        while (seats[p] != 1) {
            p++;
        }
        int max = p;
        while (seats[q] != 1) {
            q--;
        }
        max = Math.max(max, seats.length - 1 - q);
        int count = 0;
        for (int i = p; i <= q; i++) {
            count++;
            if (seats[i] == 1) {
                max = Math.max(max, (count - 1) / 2);
                count = 1;
            }
        }
        return max;
    }
}
```



#### 寻找数组的中心索引-724

给定一个整数类型的数组 nums，请编写一个能够返回数组“中心索引”的方法。

我们是这样定义数组中心索引的：数组中心索引的左侧所有元素相加的和等于右侧所有元素相加的和。

如果数组不存在中心索引，那么我们应该返回 -1。如果数组有多个中心索引，那么我们应该返回最靠近左边的那一个。

```
输入: 
nums = [1, 7, 3, 6, 5, 6]
输出: 3
解释: 
索引3 (nums[3] = 6) 的左侧数之和(1 + 7 + 3 = 11)，与右侧数之和(5 + 6 = 11)相等。
同时, 3 也是第一个符合要求的中心索引。

输入: 
nums = [1, 2, 3]
输出: -1
解释: 
数组中不存在满足此条件的中心索引。
```

**思路：**先求出数组全部元素的和，然后从左向右遍历，遍历的同时看当前元素的左右子数组的元素和是否相等，相等则返回当前索引。

```java
class Solution {
    public int pivotIndex(int[] nums) {
        if(nums.length < 3) return -1;
        int sum = 0;
        for(int num : nums) sum += num;
        int temp = 0;
        for(int i = 0; i < nums.length; i++){
            if(i != 0){
                temp += nums[i-1];
            }
            if( (sum - nums[i]) % 2 == 0){
                if( temp == (sum - nums[i]) / 2) 
                    return i;
            }
        }
        return -1;
    }
}
```

另一种写法：

```java
class Solution {
    public int pivotIndex(int[] nums) {
        if(nums == null || nums.length == 0) return -1;
        int sum = 0, leftsum = 0;
        for(int num : nums) sum += num;
        for(int i = 0; i < nums.length; i++){
            if(leftsum == sum - nums[i] - leftsum)
                return i;
            leftsum += nums[i];
        }
        return -1;
    }
}
```





#### 第三大的数-414

给定一个非空数组，返回此数组中第三大的数。如果不存在，则返回数组中最大的数。要求算法时间复杂度必须是O(n)。

```
输入: [3, 2, 1]
输出: 1
解释: 第三大的数是 1.

输入: [2, 2, 3, 1]
输出: 1
解释: 注意，要求返回第三大的数，是指第三大且唯一出现的数。
存在两个值为2的数，它们都排第二。
```

**思路：**使用三个变量存储前三大的值，然后再判断特殊情况

```java
class Solution {
    public int thirdMax(int[] nums) {
        long max1 = Long.MIN_VALUE;
        long max2 = Long.MIN_VALUE;
        long max3 = Long.MIN_VALUE;
        for(int num : nums){
            if(num > max1){
                max3 = max2;
                max2 = max1;
                max1 = num;
            }else if(num > max2 && num < max1){ //不能等于，是为了排除重复元素的影响
                max3 = max2;
                max2 = num;
            }else if(num > max3 && num < max2){
                max3 = num;
            }
        }
        // if(max3 == Long.MIN_VALUE) return (int)max1;
        // return (int)max3 ;
        return max3 == Long.MIN_VALUE ? (int)max1 : (int)max3;
    }
}
```



#### 递增的三元子序列-334

给定一个未排序的数组，判断这个数组中是否存在长度为 3 的递增子序列。

数学表达式如下:

如果存在这样的 i, j, k,  且满足 0 ≤ i < j < k ≤ n-1，使得 arr[i] < arr[j] < arr[k] ，返回 true ; 否则返回 false 。

```
输入: [1,2,3,4,5]
输出: true

输入: [5,4,3,2,1]
输出: false
```

**思路1：**用两个变量保存最小和次小的值，当出现比这两个大的数时，就返回 true

```java
class Solution {
    public boolean increasingTriplet(int[] nums) {
        int a = Integer.MAX_VALUE;
        int b = Integer.MAX_VALUE;
        for(int num : nums){
            if(num <= a) a = num;
            else if(num <= b) b = num;
            else return true;
        }
        return false;
    }
}
```

**思路2：**动态规划， dp[i] 表示以 i 结尾的最长的递增子序列，则判断 dp[1] - dp[i]，如果有比 dp[i]

小的数，则 dp[i] = Math.max(dp[i], dp[j]+1)；

```java
class Solution {
    public boolean increasingTriplet(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        for(int i = 0; i < n; i++) dp[i] = 1;
        
        for(int i = 1; i < n; i++){
            for(int j = i-1; j >= 0; j--){
                if(nums[j] < nums[i])
                    dp[i] = Math.max(dp[i], dp[j]+1);
                if(dp[i] == 3) return true;
            }
        }
        return false;
    }
}
```





#### 滑动窗口的最大值-239

给定一个数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口 k 内的数字。滑动窗口每次只向右移动一位。返回滑动窗口最大值。

```
输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7] 
解释: 

  滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**思路：**

遍历数组，将数存放在双向队列中，并用L,R来标记窗口的左边界和右边界。队列中保存的并不是真的数，而是该数值对应的数组下标位置，并且数组中的数要从大到小排序。

如果当前遍历的数比队尾的值大，则需要弹出队尾值，直到队列重新满足从大到小的要求。

1. 如果遇到的数字比队列中所有的数字都大，那么它就是最大值，其它数字不可能是最大值了，将队列中的所有数字清空，放入该数字，该数字位于队列头部；
2. 如果遇到的数字比队列中的所有数字都小，那么它还有可能成为之后滑动窗口的最大值，放入队列的末尾；
3. 如果遇到的数字比队列中最大值小，最小值大，那么将比它小数字不可能成为最大值了，删除较小的数字，放入该数字。

刚开始遍历时，L和R都为0，有一个形成窗口的过程，此过程没有最大值，L不动，R向右移。当窗口大小形成时，L和R一起向右移，每次移动时，判断队首的值的数组下标是否在[L,R]中，如果不在则需要弹出队首的值，当前窗口的最大值即为队首的数。

```html
输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7]

解释过程中队列中都是具体的值，方便理解，具体见代码。
初始状态：L=R=0,队列:{}
i=0,nums[0]=1。队列为空,直接加入。队列：{1}
i=1,nums[1]=3。队尾值为1，3>1，弹出队尾值，加入3。队列：{3}
i=2,nums[2]=-1。队尾值为3，-1<3，直接加入。队列：{3,-1}。此时窗口已经形成，L=0,R=2，result=[3]
i=3,nums[3]=-3。队尾值为-1，-3<-1，直接加入。队列：{3,-1,-3}。队首3对应的下标为1，L=1,R=3，有效。result=[3,3]
i=4,nums[4]=5。队尾值为-3，5>-3，依次弹出后加入。队列：{5}。此时L=2,R=4，有效。result=[3,3,5]
i=5,nums[5]=3。队尾值为5，3<5，直接加入。队列：{5,3}。此时L=3,R=5，有效。result=[3,3,5,5]
i=6,nums[6]=6。队尾值为3，6>3，依次弹出后加入。队列：{6}。此时L=4,R=6，有效。result=[3,3,5,5,6]
i=7,nums[7]=7。队尾值为6，7>6，弹出队尾值后加入。队列：{7}。此时L=5,R=7，有效。result=[3,3,5,5,6,7]
```

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums == null || nums.length < 2) return nums;
        // 双向队列 保存当前窗口最大值的数组位置 保证队列中数组位置的数值按从大到小排序
        LinkedList<Integer> queue = new LinkedList();
        // 结果数组
        int[] result = new int[nums.length-k+1];
        // 遍历nums数组
        for(int i = 0;i < nums.length;i++){
            // 保证从大到小 如果前面数小则需要依次弹出，直至满足要求
            while(!queue.isEmpty() && nums[queue.peekLast()] <= nums[i]){
                queue.pollLast();
            }
            // 添加当前值对应的数组下标
            queue.addLast(i);
            // 判断当前队列中队首的值是否有效
            if(queue.peek() <= i-k){
                queue.poll();   
            } 
            // 当窗口长度为k时 保存当前窗口中最大值
            if(i+1 >= k){
                result[i+1-k] = nums[queue.peek()];
            }
        }
        return result;
    }
}
```



#### 数组的度-697

给定一个非空且只包含非负数的整数数组 nums, 数组的度的定义是指数组里任一元素出现频数的最大值。你的任务是找到与 nums 拥有相同大小的度的最短连续子数组，返回其长度。

```
输入: [1, 2, 2, 3, 1]
输出: 2
解释: 
输入数组的度是2，因为元素1和2的出现频数最大，均为2.
连续子数组里面拥有相同度的有如下所示:
[1, 2, 2, 3, 1], [1, 2, 2, 3], [2, 2, 3, 1], [1, 2, 2], [2, 2, 3], [2, 2]
最短连续子数组[2, 2]的长度为2，所以返回2.

输入: [1,2,2,3,1,4,2]
输出: 6
```

**思路1：**用三个 HashMap 分别存储每个数字出现的次数，第一次出现的位置，最后一次出现的位置，当出现次数与最大出现次数相等时，通过第一次和最后一次出现的位置找到最短长度。

时间和空间复杂度均为 O (n)

```java
class Solution {
    public int findShortestSubArray(int[] nums) {
        Map<Integer, Integer> left = new HashMap(),
            right = new HashMap(), count = new HashMap();

        for (int i = 0; i < nums.length; i++) {
            int x = nums[i];
            if (left.get(x) == null) left.put(x, i);
            right.put(x, i);
            count.put(x, count.getOrDefault(x, 0) + 1);
        }

        int ans = nums.length;
        int degree = Collections.max(count.values());
        for (int x: count.keySet()) {
            if (count.get(x) == degree) {
                ans = Math.min(ans, right.get(x) - left.get(x) + 1);
            }
        }
        return ans;
    }
}
```

**思路2：**用数组存储

```java
class Solution {
    public int findShortestSubArray(int[] nums) {
        int max = 0;
        for(int num : nums) max = Math.max(max, num);
        int[] count = new int[max+1];
        int[] left = new int[max+1];
        int[] right = new int[max+1];
        int degree = 1;
        for(int i = 0; i < nums.length; i++){
            int num = nums[i];
            if(count[num] == 0){
                left[num] = i;
                right[num] = i;
            }else{
                right[num] = i;
            }
            degree = Math.max(degree, ++count[num]);
        }
        int res = nums.length;
        for(int x : nums){
            if(count[x] == degree){
                res = Math.min(res, right[x] - left[x] + 1);
            }
        }
        return res;
    }
}
```



#### 旋转数组-189

给定一个数组，将数组中的元素向右移动 *k* 个位置，其中 *k* 是非负数。

```
输入: [1,2,3,4,5,6,7] 和 k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右旋转 1 步: [7,1,2,3,4,5,6]
向右旋转 2 步: [6,7,1,2,3,4,5]
向右旋转 3 步: [5,6,7,1,2,3,4]

输入: [-1,-100,3,99] 和 k = 2
输出: [3,99,-1,-100]
解释: 
向右旋转 1 步: [99,-1,-100,3]
向右旋转 2 步: [3,99,-1,-100]
```

**方法一：**使用额外的数组

```java
public class Solution {
    public void rotate(int[] nums, int k) {
        int[] a = new int[nums.length];
        for (int i = 0; i < nums.length; i++) {
            a[(i + k) % nums.length] = nums[i];
        }
        for (int i = 0; i < nums.length; i++) {
            nums[i] = a[i];
        }
    }
}
```

**方法二：**使用反转

这个方法基于这个事实：当我们旋转数组 k 次，k % n 个尾部元素会被移动到头部，剩下的元素会被向后移动。

在这个方法中，我们首先将所有元素反转。然后反转前 k 个元素，再反转后面 n−k 个元素，就能得到想要的结果。

```
原始数组                  : 1 2 3 4 5 6 7
反转所有数字后             : 7 6 5 4 3 2 1
反转前 k 个数字后          : 5 6 7 4 3 2 1
反转后 n-k 个数字后        : 5 6 7 1 2 3 4 --> 结果
```

```java
public class Solution {
    public void rotate(int[] nums, int k) {
        k %= nums.length;
        reverse(nums, 0, nums.length - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, nums.length - 1);
    }
    public void reverse(int[] nums, int start, int end) {
        while (start < end) {
            int temp = nums[start];
            nums[start] = nums[end];
            nums[end] = temp;
            start++;
            end--;
        }
    }
}
```



#### 螺旋矩阵-54

给定一个包含 *m* x *n* 个元素的矩阵（*m* 行, *n* 列），请按照顺时针螺旋顺序，返回矩阵中的所有元素。

```
输入:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
输出: [1,2,3,6,9,8,7,4,5]
```

**思路：**模拟过程

```java
import java.util.*;
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> res = new ArrayList<>();
        if(matrix == null || matrix.length == 0) return res;
        int rows = matrix.length, cols = matrix[0].length;
        int up_row = 0, down_row = rows - 1, left_col = 0, right_col = cols - 1;
        while(up_row <= down_row && left_col <= right_col){
            for(int i = left_col; i <= right_col; i++) res.add(matrix[up_row][i]);
            up_row++;
            if(up_row > down_row) break;
            for(int i = up_row; i <= down_row; i++) res.add(matrix[i][right_col]);
            right_col--;
            if(right_col < left_col) break;
            for(int i = right_col; i >= left_col; i--) res.add(matrix[down_row][i]);
            down_row--;
            for(int i = down_row; i >= up_row; i--) res.add(matrix[i][left_col]);
            left_col++;
        }
        return res;
    }  
}
```



#### 螺旋矩阵II-59

给定一个正整数 *n*，生成一个包含 1 到 *n*2 所有元素，且元素按顺时针顺序螺旋排列的正方形矩阵。

```
输入: 3
输出:
[
 [ 1, 2, 3 ],
 [ 8, 9, 4 ],
 [ 7, 6, 5 ]
]
```

**模拟过程：**

```java
class Solution {
    public int[][] generateMatrix(int n) {
        int l = 0, r = n - 1, t = 0, b = n - 1;
        int[][] mat = new int[n][n];
        int num = 1, tar = n * n;
        while(num <= tar){
            for(int i = l; i <= r; i++) mat[t][i] = num++; // left to right.
            t++;
            for(int i = t; i <= b; i++) mat[i][r] = num++; // top to bottom.
            r--;
            for(int i = r; i >= l; i--) mat[b][i] = num++; // right to left.
            b--;
            for(int i = b; i >= t; i--) mat[i][l] = num++; // bottom to top.
            l++;
        }
        return mat;
    }
}
```





#### 有序数组的平方-977

给定一个按非递减顺序排序的整数数组 `A`，返回每个数字的平方组成的新数组，要求也按非递减顺序排序。

```html
输入：[-4,-1,0,3,10]
输出：[0,1,9,16,100]

输入：[-7,-3,2,3,11]
输出：[4,9,9,49,121]
```

**思路：**使用类似与归并排序的双指针。因为数组 `A` 已经排好序了， 所以可以说数组中的负数已经按照平方值降序排好了，数组中的非负数已经按照平方值升序排好了。

举一个例子，若给定数组为 [-3, -2, -1, 4, 5, 6]，数组中负数部分 [-3, -2, -1] 的平方为 [9, 4, 1]，数组中非负部分 [4, 5, 6] 的平方为 [16, 25, 36]。我们的策略就是从前向后遍历数组中的非负数部分，并且反向遍历数组中的负数部分。

```java
class Solution {
    public int[] sortedSquares(int[] A) {
        int n = A.length;
        int j = 0;
        while(j < n && A[j] < 0) j++;
        int i = j - 1, t = 0;
        int[] res = new int[n];
        while(i >= 0 && j < n){
            if(A[i]*A[i] < A[j]*A[j]){
                res[t++] = A[i]*A[i];
                i--;
            }else{
                res[t++] = A[j]*A[j];
                j++;
            }
        }
        while(i >= 0) {
            res[t++] = A[i]*A[i];
            i--;
        }
        while(j < n){
            res[t++] = A[j]*A[j];
            j++;
        }
        return res;
    }
}
```



#### 高度检查器-1051

学校在拍年度纪念照时，一般要求学生按照 非递减 的高度顺序排列。

请你返回至少有多少个学生没有站在正确位置数量。该人数指的是：能让所有学生以 非递减 高度排列的必要移动人数。

```
输入：[1,1,4,2,1,3]
输出：3
解释：
高度为 4、3 和最后一个 1 的学生，没有站在正确的位置。
```

**思路：**数组拷贝后排序，与原数组一一对比，记录不同的元素的个数。

时间复杂度为 O (nlogn)，空间复杂度为 O (n)；

```java
public int heightChecker(int[] heights) {
    int[] sortHeights = new int[heights.length];
    System.arraycopy(heights, 0, sortHeights, 0, heights.length);
    Arrays.sort(sortHeights);
    int diffCount = 0;
    for (int i = 0; i < heights.length; i++) {
        if (heights[i] != sortHeights[i]) {
            diffCount++;
        }
    }
    return diffCount;
}
```

**思路二：**

首先我们其实并不关心排序后得到的结果，我们想知道的只是在该位置上，与最小的值是否一致 题目中已经明确了值的范围 1 <= heights[i] <= 100；这是一个在固定范围内的输入，比如输入： [1,1,4,2,1,3]
输入中有3个1，1个2，1个3和1个4，3个1肯定会在前面，依次类推，所以，我们需要的仅仅只是计数而已。

时间复杂度和空间复杂度均为 O (n)；

```java
class Solution{
    public int heightChecker(int[] heights) {
        int[] arr = new int[101];
        for (int height : heights) {
            arr[height]++;
        }
        int count = 0;
        for (int i = 1, j = 0; i < arr.length; i++) {
            while (arr[i]-- > 0) {
                if (heights[j++] != i) count++;
            }
        }
        return count;
    }
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



#### 转置矩阵-867

给定一个矩阵 `A`， 返回 `A` 的转置矩阵。矩阵的转置是指将矩阵的主对角线翻转，交换矩阵的行索引与列索引。

```
输入：[[1,2,3],[4,5,6],[7,8,9]]
输出：[[1,4,7],[2,5,8],[3,6,9]]

输入：[[1,2,3],[4,5,6]]
输出：[[1,4],[2,5],[3,6]]
```

**思路：**尺寸为 R x C 的矩阵 A 转置后会得到尺寸为 C x R 的矩阵 ans，对此有 ans[c] [r] = A[r] [c]。让我们初始化一个新的矩阵 ans 来表示答案。然后，我们将酌情复制矩阵的每个条目。

```java
class Solution {
    public int[][] transpose(int[][] A) {
        int R = A.length, C = A[0].length;
        int[][] ans = new int[C][R];
        for (int r = 0; r < R; r++)
            for (int c = 0; c < C; c++) {
                ans[c][r] = A[r][c];
            }
        return ans;
    }
}
```



#### 两个数组的交集-349

给定两个数组，编写一个函数来计算它们的交集。

```
输入: nums1 = [1,2,2,1], nums2 = [2,2]
输出: [2]
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [9,4]
```

**思路：**使用哈希表

```java
public class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> set = new HashSet<>();
        Set<Integer> intersect = new HashSet<>();
        for (int i = 0; i < nums1.length; i++) {
            set.add(nums1[i]);
        }
        for (int i = 0; i < nums2.length; i++) {
            if (set.contains(nums2[i])) {
                intersect.add(nums2[i]);
            }
        }
        int[] result = new int[intersect.size()];
        int i = 0;
        for (Integer num : intersect) {
            result[i++] = num;
        }
        return result;
    }
}
```

使用数组：

```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        for(int x : nums1){
            max = Math.max(max, x);
            min = Math.min(min, x);
        }
        boolean[] map = new boolean[max-min+1];
        for(int x : nums1){
            map[x-min] = true;
        }
        int[] arr = new int[max-min+1];
        int index = 0;
        for(int x : nums2){
            if(min <= x && x <= max && map[x-min]){
                arr[index++] = x;
                map[x-min] = false;
            }
        }
        int[] res = new int[index];
        for(int i = 0; i < index; i++){
            res[i] = arr[i];
        }
        return res;
    }
}
```



#### 分割等和子集-416

给定一个**只包含正整数**的**非空**数组。是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

```html
输入: [1, 5, 11, 5]
输出: true
解释: 数组可以分割成 [1, 5, 5] 和 [11].

输入: [1, 2, 3, 5]
输出: false
解释: 数组不能分割成两个元素和相等的子集.
```

**回溯法：**

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for(int num : nums) sum += num;
        if(sum % 2 != 0) return false;	// 不能被2整除直接返回 false
        sum /= 2;
        return helper(nums, 0, sum);
    }
    
    public boolean helper(int[] nums, int index, int target){
        if(target == 0) return true;
        if(index == nums.length || target < 0) return false;
        if(helper(nums, index+1, target-nums[index])) return true;
        int j = index + 1;	// 找到下一个起点，跳过相等的值
        while(j < nums.length && nums[j] == nums[index]) j++; // 新的起点不用减 nums[index]
        return helper(nums, j, target);
    }
}
```

**动态规划：**

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for(int num : nums) sum += num;
        if(sum % 2 != 0) return false;
        sum /= 2;
        boolean[] dp = new boolean[sum+1];
        dp[0] = true;
        for(int num : nums){
            for(int j = sum; j >= 0; j--){
                if(j >= num) dp[j] = dp[j] || dp[j-num];
            }
        }
        return dp[sum];
    }
}
```





#### 将数组分成和相等的三个部分-1013

给定一个整数数组 A，只有我们可以将其划分为三个和相等的非空部分时才返回 true，否则返回 false。

形式上，如果我们可以找出索引 i+1 < j 且满足 (A[0] + A[1] + ... + A[i] == A[i+1] + A[i+2] + ... + A[j-1] == A[j] + A[j-1] + ... + A[A.length - 1]) 就可以将数组三等分。

```
输出：[0,2,1,-6,6,-7,9,1,2,0,1]
输出：true
解释：0 + 2 + 1 = -6 + 6 - 7 + 9 + 1 = 2 + 0 + 1

输入：[0,2,1,-6,6,7,9,-1,2,0,1]
输出：false

输入：[3,3,6,5,-2,2,5,1,-9,4]
输出：true
解释：3 + 3 = 6 = 5 - 2 + 2 + 5 + 1 - 9 + 4
```

**思路：**先判断是否能整除以3，计算和等于 sum / 3 的子数组个数，如果等于 3 则返回 true

```java
class Solution {
    public boolean canThreePartsEqualSum(int[] A) {
        int sum = 0;
        for(int x : A) sum += x;
        if(sum % 3 != 0) return false;
        int count = 0, part = 0;
        for(int i = 0; i < A.length; i++){
            part += A[i];
            if(part == sum / 3) {
                count++;
                part = 0;
            }
            if(count == 3) return true;
        }
        return false;        
    }
}
```

