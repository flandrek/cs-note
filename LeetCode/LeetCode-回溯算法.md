#### 子集

给定一组**不含重复元素**的整数数组 *nums*，返回该数组所有可能的子集（幂集），解集不能包含重复的子集。

```
输入: nums = [1,2,3]
输出:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

回溯算法：

``` java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        backtrack(0, nums, res, new ArrayList<Integer>());
        return res;
    }

	private void backtrack(int i, int[] nums, List<List<Integer>> res, 				 					ArrayList<Integer> tmp) {
        res.add(new ArrayList<>(tmp));
        for (int j = i; j < nums.length; j++) {
            tmp.add(nums[j]);
            backtrack(j + 1, nums, res, tmp);
            tmp.remove(tmp.size() - 1);
        }
    }
}
```



```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> ans = new ArrayList<>();
    ans.add(new ArrayList<>());
    if (nums == null || nums.length == 0)
        return ans;
    for (int i = 0; i < nums.length; i++) {
        int size = ans.size();
        for (int j = 0; j < size; j++) {
            List<Integer> subset = new ArrayList<>(ans.get(j));
            subset.add(nums[i]);
            ans.add(subset);
        }
    }
    return ans;
}
```

![1560502244448](E:\markdown笔记\图片\1560502244448.png)

#### 子集II-90

给定一个可能包含重复元素的整数数组 ***nums***，返回该数组所有可能的子集（幂集）。

**说明：**解集不能包含重复的子集。

```
输入: [1,2,2]
输出:
[
  [2],
  [1],
  [1,2,2],
  [2,2],
  [1,2],
  []
]
```

**回溯法：**因为不能有重复的子集，因此要先将数据进行排序，再递归的时候出现相同的元素就跳过。

```java
class Solution {    
    List<List<Integer>> res = new ArrayList();    
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        List<Integer> list = new ArrayList();
        backtrack(nums, 0, list);
        return res;        
    }
    
    public void backtrack(int[] nums, int start, List<Integer> list){        
        res.add(new ArrayList(list));
        for(int i = start; i < nums.length; i++){
            if(i > start && nums[i-1] == nums[i]) continue;
            list.add(nums[i]);
            backtrack(nums, i+1, list);
            list.remove(list.size()-1);
        }                
    }
}
```







#### 全排列-46

给定一个**没有重复**数字的序列，返回其所有可能的全排列。

```
输入: [1,2,3]
输出:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```

这里有一个回溯函数，使用第一个整数的索引作为参数 backtrack(first)。

1. 如果第一个整数有索引 n，意味着当前排列已完成；
2. 2遍历索引 first 到索引 n - 1 的所有整数。

3. 在排列中放置第 i 个整数， 即 swap(nums[first], nums[i]).

4. 继续生成从第 i 个整数开始的所有排列: backtrack(first + 1).

5. 现在回溯，即通过 swap(nums[first], nums[i]) 还原.

```java
class Solution {
      public void backtrack(int n, ArrayList<Integer> nums, 
                            List<List<Integer>> output, int first) {
         // if all integers are used up
         if (first == n)
          	 output.add(new ArrayList<Integer>(nums));
         for (int i = first; i < n; i++) {
              // place i-th integer first 
              // in the current permutation
              Collections.swap(nums, first, i);
              // use next integers to complete the permutations
              backtrack(n, nums, output, first + 1);
              // backtrack
              Collections.swap(nums, first, i);
         }
      }

      public List<List<Integer>> permute(int[] nums) {
            // init output list
            List<List<Integer>> output = new LinkedList();
            // convert nums into list since the output is a list of lists
            ArrayList<Integer> nums_lst = new ArrayList<Integer>();
            for (int num : nums)  nums_lst.add(num);
            int n = nums.length;
            backtrack(n, nums_lst, output, 0);
            return output;
     }
}
```

```java
public List<List<Integer>> permute(int[] nums) {
   	List<List<Integer>> list = new ArrayList<>();
   	// Arrays.sort(nums); // not necessary
   	backtrack(list, new ArrayList<>(), nums);
   	return list;
}

private void backtrack(List<List<Integer>> list, List<Integer> tempList, int [] nums){
   	if(tempList.size() == nums.length){
      	list.add(new ArrayList<>(tempList));
   	} else{
      	for(int i = 0; i < nums.length; i++){ 
         	if(tempList.contains(nums[i])) continue; // element already exists, skip
         	tempList.add(nums[i]);
         	backtrack(list, tempList, nums);
         	tempList.remove(tempList.size() - 1);
      	}
   	}
} 
```



#### 组合总合-39

给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合，candidates 中的数字可以无限制重复被选取。

```html
所有数字（包括 target）都是正整数。
解集不能包含重复的组合。 
输入: candidates = [2,3,5], target = 8,
所求解集为:
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```

```java
class Solution {
    
    List<List<Integer>> res = new ArrayList<>();
    
    public List<List<Integer>> combinationSum(int[] candidates, int target) {        
        Arrays.sort(candidates);
        backtrack(candidates, target, res, 0, new ArrayList<Integer>());
        return res;
    }

    private void backtrack(int[] candidates, int target, int i, 			                                            ArrayList<Integer> list) {
        if (target < 0) return;
        if (target == 0) {
            res.add(new ArrayList<>(list));
            return;
        }
        for (int start = i; start < candidates.length; start++) {
            if (target < candidates[start]) break;
            list.add(candidates[start]);
            backtrack(candidates, target - candidates[start], start, list);
            list.remove(list.size() - 1);
        }
    }
}
```



#### 组合总和II-40

给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中只能使用一次。

```
输入: candidates = [10,1,2,7,6,1,5], target = 8,
所求解集为:
[
  [1, 7],
  [1, 2, 5],
  [2, 6],
  [1, 1, 6]
]

输入: candidates = [2,5,2,1,2], target = 5,
所求解集为:
[
  [1,2,2],
  [5]
]
```

**思路：**由于每个元素都只能使用一次，因此要在上一题的基础上多加一个判断，即：

`if(i > start && candidates[i] == candidates[i-1]) continue;`

```java
class Solution {
    
    List<List<Integer>> res = new ArrayList();
    
    public List<List<Integer>> combinationSum2(int[] candidates, int target){
        Arrays.sort(candidates);
        List<Integer> list = new ArrayList();
        backtrack(candidates, target, 0, list);
        return res;
    }
    
    public void backtrack(int[] candidates, int target, int start, 																List<Integer> list){
        if(target < 0) return;
        if(target == 0) {
            res.add(new ArrayList(list));
            return;
        }
        for(int i = start; i < candidates.length; i++){
            if(target < candidates[i]) break;
            if(i > start && candidates[i] == candidates[i-1]) continue;
            list.add(candidates[i]);
            backtrack(candidates, target-candidates[i], i+1, list);
            list.remove(list.size()-1);
        }
    }
}
```







#### 组合总和III-216

找出所有相加之和为 n 的 k 个数的组合。组合中只允许含有 1 - 9 的正整数，并且每种组合中不存在重复的数字。

说明：1.所有数字都是正整数；2.解集不能包含重复的组合。 

```
输入: k = 3, n = 7
输出: [[1,2,4]]

输入: k = 3, n = 9
输出: [[1,2,6], [1,3,5], [2,3,4]]
```

**思路：**先将数组排序，然后使用回溯法。

```java
class Solution {
    List<List<Integer>> res = new ArrayList();
    public List<List<Integer>> combinationSum3(int k, int n) {
        List<Integer> list = new ArrayList();
        backtrack(k, n, 1, list);
        return res;
    }
    
    public void backtrack(int k, int n, int start, List<Integer> list){
        
        if(list.size() == k && n == 0) 
            res.add(new ArrayList(list));
        for(int i = start; i <= 9; i++){
            if(i > n) break;
            list.add(i);
            backtrack(k, n-i, i+1, list);
            list.remove(list.size()-1);
        }
    }
}
```



#### 电话号码的字母组合-17

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合，给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![17_telephone_keypad](E:\markdown笔记\图片\17_telephone_keypad.png)

```
输入："23"
输出：["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```

**递归过程中记录组合：**

```java
public class Solution {
    private static final String[] KEYS = { "", "", "abc", "def", "ghi", "jkl", "mno", 				"pqrs", "tuv", "wxyz" };

    public List<String> letterCombinations(String digits) {
        List<String> ret = new LinkedList<String>();
        combination("", digits, 0, ret);
        return ret;
    }

    private void combination(String s, String digits, int offset, List<String> ret){
        if (offset >= digits.length()) {
            ret.add(s);
            return;
        }
        String letters = KEYS[(digits.charAt(offset) - '0')];
        for (int i = 0; i < letters.length(); i++) {
            combination(s + letters.charAt(i), digits, offset + 1, ret);
        }
    }
}
```

**FIFO队列：**

```java
public List<String> letterCombinations(String digits) {
    LinkedList<String> ans = new LinkedList<String>();
    if(digits.isEmpty()) return ans;
    String[] mapping = new String[] {"0", "1", "abc", "def", "ghi", "jkl", "mno", "pqrs", 							"tuv", "wxyz"};
    ans.add("");
    for(int i = 0; i < digits.length(); i++){
        int x = Character.getNumericValue(digits.charAt(i));
        while(ans.peek().length() == i){
            String t = ans.remove();
            for(char s : mapping[x].toCharArray())
                ans.add(t + s);
        }
    }
    return ans;
}
```



#### 单词搜索-79

给定一个二维网格和一个单词，找出该单词是否存在于网格中。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

```
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

给定 word = "ABCCED", 返回 true.
给定 word = "SEE", 返回 true.
给定 word = "ABCB", 返回 false.
```

```java
class Solution {
     public boolean exist(char[][] board, String word) {
        boolean[][] visited = new boolean[board.length][board[0].length];
        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                if (backtrack(i, j, 0, word, visited, board)) return true;
            }
        }
        return false;

    }

    private boolean backtrack(int i, int j, int idx, String word, 
                              	boolean[][] visited, char[][] board) {
        if (idx == word.length()) return true;
        if (i >= board.length || i < 0 || j >= board[0].length || j < 0 || 
            		board[i][j] != word.charAt(idx) || visited[i][j])
            return false;
        visited[i][j] = true;
        if (backtrack(i + 1, j, idx + 1, word, visited, board) ||
            backtrack(i - 1, j, idx + 1, word, visited, board) || 
            backtrack(i, j + 1, idx + 1, word, visited, board) ||
            backtrack(i, j - 1, idx + 1, word, visited, board))
            return true;
        visited[i][j] = false; // 回溯
        return false;
    }
}
```



#### 正则表达式匹配-10

给你一个字符串 `s` 和一个字符规律 `p`，请你来实现一个支持 `'.'` 和 `'*'` 的正则表达式匹配。

```
'.' 匹配任意单个字符
'*' 匹配零个或多个前面的那一个元素
s 可能为空，且只包含从 a-z 的小写字母。
p 可能为空，且只包含从 a-z 的小写字母，以及字符 . 和 *。
```

```
输入:
s = "aa"
p = "a*"
输出: true
解释: 因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。
```

```
输入:
s = "aab"
p = "c*a*b"
输出: true
解释: 因为 '*' 表示零个或多个，这里 'c' 为 0 个, 'a' 被重复一次。因此可以匹配字符串 "aab"。
```

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if(p.isEmpty()) return s.isEmpty();
        boolean first_match = (!s.isEmpty() && 
                               (p.charAt(0) == s.charAt(0) || p.charAt(0) == '.'));
        // 如果第二个字母为‘*’的情况, 有3种情况
        // 1:s,p+2; 2:s+1,p+2; 3:s+1,p;
        if(p.length() >= 2 && p.charAt(1) == '*'){
            // 此处先判断 s 是否和 p + 2 匹配，若不匹配，则 s,p+2; s+1,p+2 不用再判断
            return (isMatch(s, p.substring(2)) || 
                    (first_match && isMatch(s.substring(1), p)));
        }else{
            return first_match && isMatch(s.substring(1), p.substring(1));
        }
    }
}
```



#### 路径总合-III-437

给定一个二叉树，它的每个结点都存放着一个整数值，找出路径和等于给定数值的路径总数。

路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。

```html
root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8

      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1

返回 3。和等于 8 的路径有:

1.  5 -> 3
2.  5 -> 2 -> 1
3.  -3 -> 11
```

**递归实现：**

```java
class Sloution{
/* 有了以上铺垫，详细注释一下代码 */
    int pathSum(TreeNode root, int sum) {
        if (root == null) return 0;
        int pathImLeading = count(root, sum); // 自己为开头的路径数
        int leftPathSum = pathSum(root.left, sum); // 左边路径总数（相信他能算出来）
        int rightPathSum = pathSum(root.right, sum); // 右边路径总数（相信他能算出来）
        return leftPathSum + rightPathSum + pathImLeading;
    }
    int count(TreeNode node, int sum) {
        if (node == null) return 0;
        // 我自己能不能独当一面，作为一条单独的路径呢？
        int count = (node.val == sum) ? 1 : 0;
        // 左边的小老弟，你那边能凑几个 sum - node.val 呀？
        int leftBrother = count(node.left, sum - node.val); 
        // 右边的小老弟，你那边能凑几个 sum - node.val 呀？
        int rightBrother = count(node.right, sum - node.val);
        return  count + leftBrother + rightBrother; // 我这能凑这么多个
    }
}
```

**回溯法：**

第一种做法很明显效率不够高，存在大量重复计算；
所以第二种做法，采取了类似于数组的前 n 项和的思路，比如sum[4] == sum[1]，那么1到4之间的和肯定为 0

对于树的话，采取DFS加回溯，每次访问到一个节点，把该节点加入到当前的pathSum中，然后判断是否存在一个之前的前 n 项和，其值等于 pathSum 与 sum 之差；
如果有，就说明现在的前n项和，减去之前的前n项和，等于sum，那么也就是说，这两个点之间的路径和，就是sum，最后要注意的是，记得回溯，把路径和弹出去。

```java
class Solution{
	public int pathSum(TreeNode root, int sum) {
        if (root == null) return 0;
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
        return findPathSum(root, 0, sum, map);  
    }
    
    private int findPathSum(TreeNode curr, int sum, int target,
                            Map<Integer, Integer> map) {
        if (curr == null) return 0;
        // 通过添加当前 val 来更新前缀和
        sum += curr.val;
        // 获取由当前节点结束的有效路径数
        int numPathToCurr = map.getOrDefault(sum - target, 0); 
        // 用当前和更新映射，以便将映射传递到下一个递归;
        map.put(sum, map.getOrDefault(sum, 0) + 1);
        // 将当前数量 + 左子树 + 右子树
        int res = numPathToCurr + findPathSum(curr.left, sum, target, map)
                                      + findPathSum(curr.right, sum, target, map);
       	// 还原映射-回溯
        map.put(sum, map.get(sum) - 1);
        return res;
    }
}
```



#### 分割回文串-131

给定一个字符串 *s*，将 *s* 分割成一些子串，使每个子串都是回文串，返回 *s* 所有可能的分割方案。

```
输入: "aab"
输出:
[
  ["aa","b"],
  ["a","a","b"]
]
```

**回溯：**

先求前面的回文字符串，然后加上剩余子字符串的回文字符串。

```java
import java.util.*;
class Solution {
    List<List<String>> res = new ArrayList();
    public List<List<String>> partition(String s) {
        List<String> list = new ArrayList();
        backtrack(list, s, 0);
        return res;
    }
    
    public void backtrack(List<String> list, String s, int start){
        if(start == s.length()) res.add(new ArrayList<String>(list));
        for(int i = start; i < s.length(); i++){
            if(isPal(s, start, i)){
                list.add(s.substring(start, i+1));
                backtrack(list, s, i+1);
                list.remove(list.size()-1);
            }
        }
    }
    
    public boolean isPal(String s, int left, int right){
        while(left < right){
            if(s.charAt(left++) != s.charAt(right--)) return false;
        }
        return true;
    }
}
```



#### 矩阵中的最长递增路径-329

给定一个整数矩阵，找出最长递增路径的长度。

对于每个单元格，你可以往上，下，左，右四个方向移动。 你不能在对角线方向上移动或移动到边界外（即不允许环绕）。

```
输入: nums = 
[
  [9,9,4],
  [6,6,8],
  [2,1,1]
] 
输出: 4 
解释: 最长递增路径为 [1, 2, 6, 9]。
```

```
输入: nums = 
[
  [3,4,5],
  [3,2,6],
  [2,2,1]
] 
输出: 4 
解释: 最长递增路径是 [3, 4, 5, 6]。注意不允许在对角线方向上移动。
```

**思路：**

1. 先深度优先搜索每一个单元格；
2. 将每一个单元格和它旁边的四个方向的内容进行比较，跳过越界的和小的格子；
3. 得到每个格子的最大长度；
4. 使用 matrix[x] [y] <= matrix[i] [j]，就不用使用 visited [m] [n] 了；
5. 把得到的最大值都存储起来，遍历得到全局最大值；

```java
public static final int[][] dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};

public int longestIncreasingPath(int[][] matrix) {
    if(matrix.length == 0) return 0;
    int m = matrix.length, n = matrix[0].length;
    int[][] cache = new int[m][n];
    int max = 1;
    for(int i = 0; i < m; i++) {
        for(int j = 0; j < n; j++) {
            int len = dfs(matrix, i, j, m, n, cache);
            max = Math.max(max, len);
        }
    }   
    return max;
}

public int dfs(int[][] matrix, int i, int j, int m, int n, int[][] cache) {
    if(cache[i][j] != 0) return cache[i][j];
    int max = 1;
    for(int[] dir: dirs) {
        int x = i + dir[0], y = j + dir[1];
        if(x < 0 || x >= m || y < 0 || y >= n || matrix[x][y] <= matrix[i][j]) continue;
        int len = 1 + dfs(matrix, x, y, m, n, cache);
        max = Math.max(max, len);
    }
    cache[i][j] = max;
    return max;
}
```



#### 字母大小写全排列-784

给定一个字符串`S`，通过将字符串`S`中的每个字母转变大小写，我们可以获得一个新的字符串。返回所有可能得到的字符串集合。

```java
示例:
输入: S = "a1b2"
输出: ["a1b2", "a1B2", "A1b2", "A1B2"]

输入: S = "3z4"
输出: ["3z4", "3Z4"]

输入: S = "12345"
输出: ["12345"]
```

**思路：**

大小写字母切换：s[i] ^= (1 << 5)；

大小写字母相差 32,又因为异或重要特性:不进位加法,所以大写字母和(1<<5)异或变成变成小写字母,小写字母和(1<<5) 异或变成大写字母。

```java
class Solution {
    public List<String> letterCasePermutation(String S) {
        List<String> ans = new ArrayList<String>();
        backtrack(S.toCharArray(), 0, ans);
        return ans;
    }
    public void backtrack(char[] s, int i, List<String> ans){
        if(i == s.length){
            ans.add(String.valueOf(s));
            return;
        }
        backtrack(s, i+1, ans);
        if(s[i] <'0' || s[i] > '9'){
            s[i] ^= (1 << 5);
            backtrack(s, i+1, ans);
        }     
    }
}
```

先进入小写进入递归，再变成大写进入递归。

```java
class Solution {
    private List<String> res;
    private List<String> resByCallFunc;
    
    public List<String> letterCasePermutationByCallFunc(String S) {
        resByCallFunc = new LinkedList<>();
        solve(S.toCharArray(),0);
        return resByCallFunc;
    }

    private void solveByCallFunc(char[] data, int index){
        if(index == data.length){
            resByCallFunc.add(new String(data));
            return;
        }
        if(Character.isDigit(data[index])){
            solve(data,index + 1);
        }
        else{
            data[index] = Character.toLowerCase(data[index]);
            solve(data, index + 1);
            data[index] = Character.toUpperCase(data[index]);
            solve(data, index + 1);
        }
    }
}
```



#### 二进制手表-401

二进制手表顶部有 4 个 LED 代表**小时（0-11）**，底部的 6 个 LED 代表**分钟（0-59）**。每个 LED 代表一个 0 或 1，最低位在右侧。

![1562849230(1)](E:\markdown笔记\图片\1562849230(1).png)

例如，上面的二进制手表读取 “3:25”。

给定一个非负整数 *n* 代表当前 LED 亮着的数量，返回所有可能的时间。

```
输入: n = 1
返回: ["1:00", "2:00", "4:00", "8:00", "0:01", "0:02", "0:04", "0:08", "0:16", "0:32"]

输出的顺序没有要求。
小时不会以零开头，比如 “01:00” 是不允许的，应为 “1:00”。
分钟必须由两位数组成，可能会以零开头，比如 “10:2” 是无效的，应为 “10:02”。
```

**思路1：**

统计总共亮灯的个数，统计小时和分钟数的二进制中 ‘ 1 ’ 的数量，数量等于 num 的都是符合要求的组合。

```java
class Solution {
    public List<String> readBinaryWatch(int num) {
        List<String> res = new ArrayList();
        for(int h = 0; h < 12; h++){
            for(int m = 0; m < 60; m++){
                if(bitCount(m) + bitCount(h) == num){
                    // res.add(String.format("%d:%02d", i, j));
                    res.add(h + (m < 10 ? ":0" : ":") + m);
                }
            }
        }
        return res;
    }
    // 自定义的 bitCount，也可以使用系统的 bitCount
    public int bitCount(int n){
        int res = 0;
        while(n != 0){
            res++;
            n = n & (n - 1);
        }
        return res;
    }
}
```

**思路2：**使用回溯法

```java
public class Solution {
    public List<String> readBinaryWatch(int num) {
        List<String> res = new ArrayList<>();
        int[] nums1 = new int[]{8, 4, 2, 1}, nums2 = new int[]{32, 16, 8, 4, 2, 1};
        for(int i = 0; i <= num; i++) {
            List<Integer> list1 = generateDigit(nums1, i);
            List<Integer> list2 = generateDigit(nums2, num - i);
            for(int num1: list1) {
                if(num1 >= 12) continue;
                for(int num2 : list2) {
                    if(num2 >= 60) continue;
                    res.add(num1 + ":" + (num2 < 10 ? "0" + num2 : num2));
                }
            }
        }
        return res;
    }

    private List<Integer> generateDigit(int[] nums, int count) {
        List<Integer> res = new ArrayList<>();
        generateDigitHelper(nums, count, 0, 0, res);
        return res;
    }

    private void generateDigitHelper(int[] nums, int count, int pos, int sum, 													List<Integer> res) {
        if(count == 0) {
            res.add(sum);
            return;
        }

        for(int i = pos; i < nums.length; i++) {
            generateDigitHelper(nums, count - 1, i + 1, sum + nums[i], res);    
        }
    }
}
```

