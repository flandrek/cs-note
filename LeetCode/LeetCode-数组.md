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

1. 遍历整个矩阵，如果 cell[i] [j] == 0 就将第 i 行和第 j 列的第一个元素标记。
2. 第一行和第一列的标记是相同的，都是 cell[0] [0]，所以需要一个额外的变量告知第一列是否被标记，同时用 cell[0] [0] 继续表示第一行的标记。
3. 然后，从第二行第二列的元素开始遍历，如果第 r 行或者第 c 列被标记了，那么就将 cell[r] [c] 设为 0。这里第一行和第一列的作用就相当于方法一中的 row_set 和 column_set 。
4. 然后我们检查是否 cell[0] [0] == 0 ，如果是则赋值第一行的元素为零。然后检查第一列是否被标记，如果是则赋值第一列的元素为零。

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

