#### generate-parentheses

给定N对括号，给出所有正确的括号组合。

For example, given *n* = 3, a solution set is: "((()))", "(()())", "(())()", "()(())", "()()()"

```java
import java.util.ArrayList;
public class Solution {
    ArrayList<String> r = new ArrayList<String>();
    public ArrayList<String> generateParenthesis(int n) {
        //保证左边‘（’的数量始终大于等于右边的‘）’数量，可以考虑回溯法
        gp("",0,0,n);
        return r;
    }
    private void gp(String s,int left,int right,int n){
        if(right == n){
            r.add(s);
        }
        if(left < n){
            gp(s+"(",left+1,right,n);
        }
        // 递归过程中 左括号x的个数必须大于等于右括号个数
        if(left > right){
            gp(s+")",left,right+1,n);
        }
    }
}

```



#### 前K个高频元素

给定一个非空的整数数组，返回其中出现频率前 **k** 高的元素。

```
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

1. 先用 Hash Map 记录每个元素出现的次数，然后放入一个桶数组-**数组中存放链表**-中，然后将存放在桶数组的元素反序取出：

```java
class Solution {
    public List<Integer> topKFrequent(int[] nums, int k) {              
        Map<Integer,Integer> map = new HashMap<Integer, Integer>();
        for(int i = 0;i < nums.length; i++){
            map.put(nums[i], map.getOrDefault(nums[i],0) + 1);
        }
        ArrayList<Integer>[] bucket = new ArrayList[nums.length + 1];
        for(Integer n : map.keySet()){
            Integer value = map.get(n);
            if(bucket[value] == null){
                ArrayList<Integer> list = new ArrayList();
                bucket[value] = list;
            }
            bucket[value].add(n);
        }
        List<Integer> res = new ArrayList<>();
        for(int i = nums.length; i >= 0 && res.size() < k ; i--){
            if(bucket[i] != null){
                res.addAll(bucket[i]);
            }
        }
        return res;     
    }
}
```

2. 先用 Hash Map 记录每个元素出现的次数，然后自定义优先队列，找出出现频率前 K 大的数：

```java
import java.util.*;
class Solution {
    public List<Integer> topKFrequent(int[] nums, int k) {
        List<Integer> res = new ArrayList<>();
        if(nums == null || nums.length == 0 || k < 1){
            return res;
        }
        Map<Integer, Integer> map = new HashMap<>();
        for(int num : nums){
            map.put(num, map.getOrDefault(num, 0) + 1);
        }
        PriorityQueue<Integer> pq = new PriorityQueue(k, (a,b) -> map.get(a)-map.get(b));
        for(int key : map.keySet()){
            if(pq.size() < k)
                pq.add(key);
            else if(map.get(key) > map.get(pq.peek())){
                pq.remove();
                pq.add(key);
            }
        }
        while(!pq.isEmpty())
            res.add(pq.remove());
        return res;
    }
}
```



#### 寻找重复数-287

给定一个包含 n + 1 个整数的数组 nums，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。

1. **不能**更改原数组（假设数组是只读的）。
2. 只能使用额外的 *O*(1) 的空间。
3. 时间复杂度小于 *O*(*n*2) 。
4. 数组中只有一个重复的数字，但它可能不止重复出现一次。

```
输入: [1,3,4,2,2]
输出: 2
```

**方法一：二分查找**

数组长度为n+1，而数字只从1到n，说明必定有重复数字。可以由二分查找法拓展：把1~n的数字从中间数字m分成两部分，若前一半1~m的数字数目超过m个，说明重复数字在前一半区间，否则，在后半区间m+1~n。每次在区间中都一分为二，知道找到重复数字，函数 countRange() 将被调用 O(logn) 次，每次需要 O(n) 的时间，时间复杂度为 O（nlogn）。

```java
class Solution {
    public int findDuplicate(int[] nums) {
        if(nums == null || nums.length == 0) return -1;
        int start = 1;
        int end = nums.length -1;
        int mid, count;
        while(start <= end){
            if(start == end){
                count = countRange(nums, start, end);
                if(count > 1) return start;
                else break;
            }
            mid = start + ((end - start) >> 1);
            count = countRange(nums, start, mid);
            if(count > mid - start + 1) end = mid;
            else start = mid + 1;
        }
        return -1;
    }
    public int countRange(int[] arr, int start, int end){
        int count = 0;
        for(int i = 0; i < arr.length; i++){
            if(arr[i] >= start && arr[i] <= end)
                count++;
        }
        return count;
    }    
}
```

**方法二：快慢指针**

假设数组中没有重复，那我们可以做到这么一点，就是将数组的下标和1到n每一个数一对一的映射起来。比如数组是213,则映射关系为0->2, 1->1, 2->3。假设这个一对一映射关系是一个函数f(n)，其中n是下标，f(n)是映射到的数。如果我们从下标为0出发，根据这个函数计算出一个值，以这个值为新的下标，再用这个函数计算，以此类推，直到下标超界。实际上可以产生一个类似链表一样的序列。比如在这个例子中有两个下标的序列，0->2->3。

但如果有重复的话，这中间就会产生多对一的映射，比如数组2131,则映射关系为0->2, {1，3}->1, 2->3。这样，我们推演的序列就一定会有环路了，这里下标的序列是0->2->3->1->1->1->1->...，而环的起点就是重复的数。

所以该题实际上就是找环路起点的题。过程是这样的：我们先用快慢两个下标都从0开始，快下标每轮映射两次，慢下标每轮映射一次，直到两个下标再次相同。这时候保持慢下标位置不变，再用一个新的下标从0开始，这两个下标都继续每轮映射一次，当这两个下标相遇时，就是环的起点，也就是重复的数。

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int length = nums.length;
        if (length > 1) {
            // 找到快慢指针相遇的地方
            int slow = nums[0];
            int fast = nums[nums[0]];
            while (fast != slow) {
                slow = nums[slow];
                fast = nums[nums[fast]];
            }
            // 用一个新指针从头开始，直到和慢指针相遇
            fast = 0;
            while (fast != slow) {
                slow = nums[slow];
                fast = nums[fast];
            }
            return slow;
        }
        return -1;
    }
}
```



#### 数组中第k个最大元素-215

在未排序的数组中找到第 **k** 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

```
输入: [3,2,1,5,6,4] 和 k = 2
输出: 5
```

**基于快排的思想：**

时间复杂度 O(n)

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        int low = 0;
        int n = nums.length - k;
        int high = nums.length -1;
        while(low < high){
            int index = partition(nums, low, high);
            if(index > n) high = index -1;
            else low = index +1;
        }
        return nums[n];
    }
    public int partition(int[] nums, int low, int high){
        int temp = nums[low];
        while(low < high){
            while(low < high && nums[high] >= temp) high--;
            if(low < high) nums[low] = nums[high];
            while(low < high && nums[low] <= temp) low++;
            if(low < high) nums[high] = nums[low];
        }
        nums[low] = temp;
        return low;
    }
}
```

**基于桶排序**:

时间复杂度 O(n)

```java
public int findKthLargest(int[] nums, int k) {
    int max = Integer.MIN_VALUE,min = Integer.MAX_VALUE;
    for(int i : nums){
        max = Math.max(max,i);
        min = Math.min(min,i);
    }
    int n = max - min;
    int[] bucket = new int[n + 1];
    for(int i = 0;i < nums.length;i++){
        int tmp = nums[i] - min;
        bucket[tmp]++;
    }
    for(int i = n;i >= 0;i--){
        if(bucket[i] > 0)
            k -= bucket[i];
        if(k <= 0)
            return i + min;
    }
    return 0;
}
```



#### 查找所有数组中消失的数字-448

给定一个范围在  1 ≤ a[i] ≤ n ( n = 数组大小 ) 的 整型数组，数组中的元素一些出现了两次，另一些只出现一次。

找到所有在 [1, n] 范围之间没有出现在数组中的数字。

您能在不使用额外空间且时间复杂度为O(n)的情况下完成这个任务吗? 你可以假定返回的数组不算在额外空间内。

```
输入:
[4,3,2,7,8,2,3,1]
输出:
[5,6]
```

基本思想是使用 **nums[nums[i]-1] = -nums[nums[i]-1]** 迭代输入数组和标记元素为负数。这样，我们看到的所有数字都会被标记为负数。在第二次迭代中，如果一个值没有被标记为负值，就意味着我们以前从未见过这个索引，所以只需将它添加到返回列表中即可。

```java
import java.util.*;
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> res = new ArrayList<>();
        for(int i = 0; i < nums.length; i++){
            int val = Math.abs(nums[i]) -1;
            if(nums[val] > 0) nums[val] = -nums[val];
        }
        for(int i = 0; i < nums.length; i++){
            if(nums[i] > 0) res.add(i+1);
        }
        return res;
    }
}
```

更快的方法：

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> list = new ArrayList<>();
        boolean[] flag = new boolean[nums.length];
        for (int i = 0; i < nums.length; i++){
            flag[nums[i]-1] = true;
        }
        for (int i = 0; i < flag.length; i++){
            if (!flag[i]) list.add(i+1);            
        }
        return list;
    }
}
```

