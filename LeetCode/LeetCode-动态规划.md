#### **最小和路径-64**

给定一个充满非负数的 m x n 网格，找到一条从左上角到右下角的路径，找出和最小的路径。

注意:你只能在任何时间点向下或向右移动。

```java
public class Solution {
    public int minPathSum(int[][] grid) {
        if(grid == null)
            return 0;
        int rows = grid.length;
        int cols = grid[0].length;
        int[][] dp = new int[rows][cols];
        dp[0][0] = grid[0][0];
        //第一行
        for(int i = 1; i < cols; i++){
            dp[0][i] = dp[0][i-1] + grid[0][i];
        }
        //第一列
        for(int j = 1; j < rows; j++){
            dp[j][0] = dp[j-1][0] + grid[j][0];
        }
        for(int i = 1; i < rows; i++){
            for(int j = 1; j < cols; j++){
                dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }
        return dp[rows-1][cols-1];
    }
}
```

**优化空间：**？？？

```java
public class Solution {
    public int minPathSum(int[][] grid) {
        if (grid == null) {
            return 0;
        }
        int rows = grid.length;
        int cols = grid[0].length;
        int[] res = new int[cols + 1];
        Arrays.fill(res, Integer.MAX_VALUE);
        res[1] = 0;
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                //当前点的最小路径和为 : 从左边和上边选择最小的路径和再加上当前点的值
                //res[j]没更新之前就表示i-1行到第j个元素的最小路径和
                //因为是从左往右更新,res[j-1]表示i行第j-1个元素的最小路径和
                res[j] = Math.min(res[j], res[j - 1]) + grid[i - 1][j - 1];
            }
        }
        return res[cols];
    }
}
```

**不需要额外空间：**

```java
public int minPathSum(int[][] grid) {
    if(grid.length == 0 || grid[0].length == 0)
        return 0;
    for(int i = 0; i < grid.length; i++){
        for(int j = 0; j < grid[0].length; j++){
            if(i == 0 && j == 0) continue;
            if(i == 0) grid[i][j] += grid[i][j - 1];
            else if(j == 0) grid[i][j] += grid[i - 1][j];
            else grid[i][j] += Math.min(grid[i - 1][j], grid[i][j - 1]);
        }
    }
    return grid[grid.length - 1][grid[0].length - 1];    
}
```



#### **unique-paths-ii**

现在考虑是否在网格中添加了一些障碍。有多少条独特的路径？在网格中，障碍物和空格分别标记为1和0。

在3x3网格的中间有一个障碍，如下图所示。

```html
[
  [0,0,0],
  [0,1,0],
  [0,0,0]
]
```

```java
public class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        if(obstacleGrid == null)
            return 0;
        int rows = obstacleGrid.length;
        int cols = obstacleGrid[0].length;
        int[][] dp = new int[rows][cols];
        
        for(int i = 0; i < cols; i++){
            if(obstacleGrid[0][i] == 1){
                break;
            }
            dp[0][i] = 1;
        }
        for(int j = 0; j < rows; j++){
            if(obstacleGrid[j][0] == 1){
                break;
            }
            dp[j][0] = 1;
        }
        for(int i = 1; i < rows; i++){
            for(int j = 1; j < cols; j++){
                if(obstacleGrid[i][j] == 1){
                    dp[i][j] = 0;
                }else{
                    dp[i][j] = dp[i-1][j] + dp[i][j-1];
                }
            }
        }
        return dp[rows-1][cols-1];
    }
}
```

```java
public class Solution {
	public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        if(obstacleGrid == null)
            return 0;
        int rows = obstacleGrid.length;
        int cols = obstacleGrid[0].length;
        int[][] dp = new int[rows][cols];
        // 第一个格点的值与障碍数相反
        dp[0][0] = 1 - obstacleGrid[0][0];
        // 依次计算
        for(int i = 0; i < rows; i++) {
            for(int j = 0; j < cols; j++) {
                // 只有没有障碍才有通路
                if(obstacleGrid[i][j] == 0) {
                    if(i == 0 && j != 0) dp[0][j] = dp[0][j - 1]; // 左 第一行
                    else if(i != 0 && j == 0) dp[i][0] = dp[i - 1][0]; // 上 第一列
                    else if(i != 0 && j != 0) dp[i][j] += dp[i - 1][j] + dp[i][j - 1]; 
                }
            }
        }
        return dp[rows - 1][cols - 1];
    }
}
```



#### 跳跃游戏-II

给定一个非负整数数组，你最初位于数组的第一个位置，数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用最少的跳跃次数到达数组的最后一个位置。

动态规划：

```java
public int jump(int[] A) {
	int[] dp = new int[A.length]; // dp存放都到各点的最小步数
    for (int i = 0; i < dp.length; i ++) {
    	int maxPosition = Math.min(i + A[i], A.length -1); // 从i点出发能走的最远距离
        for (int j = i +1; j <= maxPosition; j++) {
        	if(dp[j] == 0) dp[j] = dp[i] + 1; // 如果位置没被走过,则到达j点的步数为dp[i]+1
        }
        if(dp[A.length - 1] != 0) break; // 当第一次到达终点时,肯定是到达终点最短的步数
    }
    return dp[A.length - 1];
}
```

```java
public int jump(int[] A) {
 	int jumps = 0, curEnd = 0, curFarthest = 0;
 	for (int i = 0; i < A.length - 1; i++) {
 		curFarthest = Math.max(curFarthest, i + A[i]);
 		if (i == curEnd) {
 			jumps++;
 			curEnd = curFarthest;
 		}
	}
	return jumps;
}
```



#### triangle

给定一个三角形，求从上到下的最小路径和,每一步都可以移动到下面一行的相邻数。例如，给定以下三角形：

The minimum path sum from top to bottom is 11 (i.e., 2 + 3 + 5 + 1 = 11).

```html
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
```

```java
import java.util.*;
public class Solution {
    public int minimumTotal(ArrayList<ArrayList<Integer>> triangle) {
        for (int i = triangle.size() - 2; i >= 0; i --) {
            for (int j = 0; j < triangle.get(i + 1).size() - 1; j ++) {
                int min = Math.min(triangle.get(i + 1).get(j), 
                                   triangle.get(i + 1).get(j + 1));
                triangle.get(i).set(j, triangle.get(i).get(j) + min);
            }
        }
        return triangle.get(0).get(0);
    }
}
```



#### 不同的二叉搜索树-96

给定一个整数 *n*，求以 1 ... *n* 为节点组成的二叉搜索树有多少种？

```
输入: 3
输出: 5
解释:
给定 n = 3, 一共有 5 种不同结构的二叉搜索树:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

标签：动态规划
假设n个节点存在二叉排序树的个数是G(n)，令f(i)为以i为根的二叉搜索树的个数，则
G(n) = f(1) + f(2) + f(3) + f(4) + ... + f(n)；当i为根节点时，其左子树节点个数为i-1个，右子树节点为n-i，则
f(i) = G(i-1) *G(n-i)；综合两个公式可以得到 卡特兰数 公式：G(n) = G(0)*G(n-1) + G(1)*(n-2) +...+ G(n-1)*G(0)

```java
class Solution {
    public int numTrees(int n) {
        int[] dp = new int[n+1];
        dp[0] = 1;
        dp[1] = 1;       
        for(int i = 2; i <= n; i++)
            for(int j = 1; j <= i; j++) 
                dp[i] += dp[j-1] * dp[i-j];        
        return dp[n];
    }
}
```



#### 每日温度-739

根据每日 气温 列表，请重新生成一个列表，对应位置的输入是你需要再等待多久温度才会升高的天数。如果之后都不会升高，请输入 0 来代替。

```
例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。提示：气温 列表长度的范围是 [1, 30000]。每个气温的值的都是 [30, 100] 范围内的整数。
```

```java
/**
 * 根据题意，从最后一天推到第一天，这样会简单很多。因为最后一天显然不会再有升高的可能，结果直接为0。
 * 再看倒数第二天的温度，如果比倒数第一天低，那么答案显然为1，如果比倒数第一天高，又因为倒数第一天
 * 对应的结果为0，即表示之后不会再升高，所以倒数第二天的结果也应该为0。
 * 自此我们容易观察出规律，要求出第i天对应的结果，只需要知道第i+1天对应的结果就可以：
 * - 若T[i] < T[i+1]，那么res[i]=1；
 * - 若T[i] > T[i+1]
 *   - res[i+1]=0，那么res[i]=0;
 *   - res[i+1]!=0，那就比较T[i]和T[i+1+res[i+1]]（即将第i天的温度与比第i+1天大的那天的温度进行比较）
 */
public int[] dailyTemperatures(int[] T) {
    int[] res = new int[T.length];
    res[T.length - 1] = 0;
    for (int i = T.length - 2; i >= 0; i--) {
        for (int j = i + 1; j < T.length; j += res[j]) {
            if (T[i] < T[j]) {
                res[i] = j - i;
                break;
            } else if (res[j] == 0) {
                res[i] = 0;
                break;
            }
        }
    }
    return res;
}
```



#### 编辑距离-72

给定两个单词 word1 和 word2，计算出将 word1 转换成 word2 所使用的最少操作数 。

你可以对一个单词进行如下三种操作：1.插入一个字符；2.删除一个字符；3.替换一个字符

```
输入: word1 = "horse", word2 = "ros"
输出: 3
解释: 
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

```
输入: word1 = "intention", word2 = "execution"
输出: 5
解释: 
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```

```html
dp[i] [j] 代表 word1 到 i 位置转换成 word2 到 j 位置需要最少步数

当 word1[i] == word2[j]，dp[i] [j] = dp[i-1] [j-1];

当 word1[i] != word2[j]，dp[i] [j] = min(dp[i-1] [j-1], dp[i-1] [j], dp[i] [j-1]) + 1

其中，dp[i-1] [j-1] 表示替换操作，dp[i-1] [j] 表示删除操作，dp[i] [j-1] 表示插入操作。
```

在这里，我们考虑从 word1 到 word2 的任何操作。它的意思是，当我们说插入操作时，我们在word2的第 j 个字符之后插入一个新字符。因此，现在必须将 word1 的 i 字符匹配到 word2 的 j - 1 字符。其他两个操作也是如此。

注意问题是对称的。在一个方向上的插入操作(即从word1到word2)与在另一个方向上的删除操作相同。所以，我们可以选择任何方向。

注意，针对第一行，第一列要单独考虑，我们引入 '' 下图所示：

![76574ab7ff2877d63b80a2d4f8496fab3c441065552edc562f62d5809e75e97e-Snipaste_2019-05-29_15-28-02](E:\markdown笔记\图片\76574ab7ff2877d63b80a2d4f8496fab3c441065552edc562f62d5809e75e97e-Snipaste_2019-05-29_15-28-02.png)

第一行，是 `word1` 为空变成 `word2` 最少步数，就是插入操作

第一列，是 `word2` 为空，需要的最少步数，就是删除操作

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int n1 = word1.length();
        int n2 = word2.length();
        int[][] dp = new int[n1 + 1][n2 + 1];
        // 第一行
        for (int j = 1; j <= n2; j++) dp[0][j] = dp[0][j - 1] + 1;
        // 第一列
        for (int i = 1; i <= n1; i++) dp[i][0] = dp[i - 1][0] + 1;

        for (int i = 1; i <= n1; i++) {
            for (int j = 1; j <= n2; j++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) dp[i][j] = 
                    	dp[i - 1][j - 1];
                else dp[i][j] = Math.min(Math.min(dp[i - 1][j - 1], dp[i][j - 1]), 
                                         dp[i - 1][j]) + 1;
            }
        }
        return dp[n1][n2];  
    }
}
```



#### 完全平方数-279

给定正整数 *n*，找到若干个完全平方数（比如 `1, 4, 9, 16, ...`）使得它们的和等于 *n*。你需要让组成和的完全平方数的个数最少。

```
输入: n = 12
输出: 3 
解释: 12 = 4 + 4 + 4
输入: n = 13
输出: 2
解释: 13 = 4 + 9
```

直接思路：找出N最接近的平方数，再循环找出剩余最接近的平方数集合（结果可能不是最优）
比如：12->9+1+1+1，最优的是12->4+4+4；
所以，上面的思路还得把所有的情况都求出来，再选出最少的，性能较差；
**优化思路**：利用之前计算的步数，转换为动态规划方程；
dp[i]代表第i需要的最少步骤，遍历所有的情况，从而找出最优解；

```java
 public int numSquares(int n) {
     //利用动态规划 定义长度为n+1的数组 对应索引所对应的数装最少的步数
     int[] dp = new int[n + 1];
     dp[0] = 0;
     for (int i = 1; i <= n; i++) {
         dp[i] = i; //先假设到这一步的最大的步数为每次+1,也可以将dp[]中全放入Integer.MAXVALUE
         for (int j = 1; i - j * j >= 0; j++) { 
             //i-j*j>=0 找到最大的j j*j就是i里面最大的完全平方数
             //dp[i-j*j]+1 表示d[i-j*j]的步数＋1 1即j*j这个完全平方数只需要一步
             dp[i] = Math.min(dp[i], dp[i - j * j] + 1);
         }
     }
     return dp[n];
 }
```

**贪心算法：**

四平方定理： 任何一个正整数都可以表示成不超过四个整数的平方之和。 推论：满足四数平方和定理的数n（四个整数的情况），必定满足 n=4^a(8b+7)

```java
int numSquares(int n) {
    //先根据上面提到的公式来缩小n
    while(n % 4 == 0) {
        n /= 4;
    }
    //如果满足公式 则返回4
    if(n % 8 == 7) {
        return 4;
    }
    //在判断缩小后的数是否可以由一个数的平方或者两个数平方的和组成
    int a = 0;
    while ((a * a) <= n) {
        int b = sqrt((n - a * a));
        if(a * a + b * b == n) {
            //如果可以 在这里返回
            if(a != 0 && b != 0) return 2;
       		else return 1;
        }
        a++;
    }
    //如果不行 返回3
    return 3;
}
```



#### 最佳买卖股票时机含冷冻期-309

给定一个整数数组，其中第 i 个元素代表了第 i 天的股票价格 。

设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:

你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。

```
输入: [1,2,3,0,2]
输出: 3 
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```

```java
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0) {
            return 0;
        }
        // 由于可以无限次交易，所以只定义两个维度，第一个维度是天数，第二个维度表示是否持有股票
        // 0表示不持有，1表示持有，2表示过渡期
        int[][] dp = new int[prices.length][3];
        dp[0][0] = 0;
        dp[0][1] = -prices[0];
        dp[0][2] = 0;
        for (int i = 1; i < prices.length; i++) {
            //第i天不持有股票的情况有两种
            // a.第i-1天也不持有股票
            // b.第i-1天是过渡期
            dp[i][0] = Math.max(dp[i-1][0], dp[i-1][2]);
            //第i天持有股票有两种情况
            // a.第i-1天也持有股票，第i天不操作，
            // b.第i-1天不持有股票，在第i天买入
            dp[i][1] = Math.max(dp[i-1][1], dp[i-1][0] - prices[i]);
            //第i天是冷冻期只有一种情况，第i-1天持有股票且卖出
            dp[i][2] = dp[i-1][1] + prices[i];
        }
        //最后最大利润为最后一天，不持有股票或者进入冷冻期的情况
        return Math.max(dp[prices.length-1][0], dp[prices.length-1][2]);
    }
}
```

```java
  /**
     * buy[i]表示第i天买入时的最大利润（最后一个操作是买）
     * sell[i]表示第i天卖出时的最大利润（最后一个操作是卖）
     */
    public int maxProfit(int[] prices) {
        int n = prices.length;
        if (n == 0)
            return 0;
        int[] buy = new int[n];
        int[] sell = new int[n];
        buy[0] = -prices[0];
        sell[0] = 0;
        if (n > 1) {
            // 因为冷冻期，所以前一天只能为买，所以比较的是buy[0]=-prices[0],而大前天无买卖，
            // 所以利润为0，则得到0-prices[1]
            buy[1] = Math.max(buy[0], 0 - prices[1]);           
            sell[1] = Math.max(sell[0], buy[0] + prices[1]);
        }
        for (int i = 2; i < n; ++i) {
            //因为冷冻期，所以前一天只能为买，所以比较的是buy[i - 1]
            buy[i] = Math.max(buy[i - 1], sell[i - 2] - prices[i]);
            //比较的是前一天卖出的最大利润和前一天买入的最大利润+今天卖出的价格
            sell[i] = Math.max(sell[i - 1], buy[i - 1] + prices[i]);
        }
        return sell[n - 1];
    }
```





#### 最长上升子序列-300

给定一个无序的整数数组，找到其中最长上升子序列的长度。

```
输入: [10,9,2,5,3,7,101,18]
输出: 4 
解释: 最长的上升子序列是 [2,3,7,101]，它的长度是 4。
```

**动态规划：**时间复杂度 O (n2)，空间复杂度 O (n) 。

定义状态：dp[i] 表示以第 i 个数字为结尾的最长上升子序列的长度。即在 [0, ..., i] 的范围内，选择 以数字 nums[i] 结尾，可以获得的最长上升子序列的长度。注意：以第 i 个数字为结尾，即 要求 nums[i] 必须被选取。反正一个子序列一定会以一个数字结尾，那我就将状态这么定义，这一点是常见的。

状态转移方程：遍历到索引是 i 的数的时候，我们应该把索引是 [0, ... ,i - 1] 的 dp 都看一遍，如果当前的数 nums[i] 严格大于之前的某个数，那么 nums[i] 就可以接在这个数后面形成一个更长的上升子序列。把前面的 i 个数都看了，dp[i] 的值就是它们的最大值加 1。即比当前数要小的那些里头，找最大的，然后加 1。

于是状态转移方程是：dp[i] = max{1 + dp[j]  if j < i and nums[i] > nums[j]}。

最后不要忘了，扫描一遍这个 dp 数组，其中最大值的就是题目要求的最长上升子序列的长度。

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    for (int i = 0; i < n; i++) {
        int max = 1;	//如果只有 1 个元素，那么这个元素自己就构成了最长上升子序列，所以设置为 1 
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                max = Math.max(max, dp[j] + 1);
            }
        }
        dp[i] = max;
    }
    int ret = 0;
    // 最后要全部走一遍，看最大值
	for (int i = 0; i < n; i++) ret = Math.max(ret, dp[i]);
	return ret;
}
```



#### 单词拆分-139

给定一个非空字符串 s 和一个包含非空单词列表的字典 wordDict，判定 s 是否可以被空格拆分为一个或多个在字典中出现的单词。

说明：

1. 拆分时可以重复使用字典中的单词
2. 你可以假设字典中没有重复的单词

```html
输入: s = "applepenapple", wordDict = ["apple", "pen"]
输出: true
解释: 返回 true 因为 "applepenapple" 可以被拆分成 "apple pen apple"。
     注意你可以重复使用字典中的单词。
输入: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
输出: false
```

现在，我们考虑 dp 数组求解的过程。我们使用 n+1大小数组的 dp ，其中 n 是给定字符串的长度。我们也使用 2 个下标指针 i 和 j ，其中 i 是当前字符串从头开始的子字符串（s'）的长度， j 是当前子字符串（s'）的拆分位置，拆分成 s'(0,j) 和 s'(j+1,i) ；

为了求出 dp 数组，我们初始化dp[0] 为 true ，这是因为空字符串总是字典的一部分。dp 数组剩余的元素都初始化为 false 。

我们用下标 i 来考虑所有从当前字符串开始的可能的子字符串。对于每一个子字符串，我们通过下标 j 将它拆分成 s1' 和 s2'（注意 i 现在指向 s2' 的结尾）。为了将 dp[i] 数组求出来，我们依次检查每个 dp[j] 是否为 true ，也就是子字符串 s1' 是否满足题目要求。如果满足，我们接下来检查 s2' 是否在字典中。如果包含，我们接下来检查 s2' 是否在字典中，如果两个字符串都满足要求，我们让 dp[i] 为 true ，否则令其为 false 。

时间复杂度：O(n2)，空间复杂度：O(n)

```java
public class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> wordDictSet=new HashSet(wordDict);
        boolean[] dp = new boolean[s.length() + 1];
        dp[0] = true;
        for (int i = 1; i <= s.length(); i++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && wordDictSet.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.length()];
    }
}
```



#### 回文子串-647

给定一个字符串，你的任务是计算这个字符串中有多少个回文子串，具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被计为是不同的子串。

```
输入: "abc"
输出: 3
解释: 三个回文子串: "a", "b", "c".
输入: "aaa"
输出: 6
说明: 6个回文子串: "a", "a", "a", "aa", "aa", "aaa".
```

**思路：**从每一个 Index 开始，尝试去扩展回文串，包含奇数情况和偶数情况

```java
public class Solution {
    private int count = 0;    
    public int countSubstrings(String s) {
        if (s == null || s.length() == 0) return 0;        
        for (int i = 0; i < s.length(); i++) { // i is the mid point
            extendPalindrome(s, i, i); // odd length;
            extendPalindrome(s, i, i + 1); // even length
        }        
        return count;
    }
    
    private void extendPalindrome(String s, int left, int right) {
        while (left >=0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            count++; left--; right++;
        }
    }
}
```



#### 最长回文子串-5

给定一个字符串 `s`，找到 `s` 中最长的回文子串。你可以假设 `s` 的最大长度为 1000。

```
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```

**动态规划：**

定义 “状态”，这里 “状态”数组是二维数组。

dp[j] [i] 表示子串 s [j，i]（包区间左右端点）是否构成回文串。如果是回文串，那么 dp[j] [i] = true ;

状态转移方程：

```
如果 s[j, i] 是一个回文串，例如 “abccba”，那么这个回文串两边各往里面收缩一个字符（如果可以的话）的子串 s[j + 1, i - 1] 也一定是回文串，即：如果 dp[j][i] == true 成立，一定有 dp[j + 1][i - 1] = true 成立。
注意：
1、此时我们要保证 [j + 1, i - 1] 能够形成区间，因此有 j + 1 < i - 1，整理得 j - i < -2，或者 
i - j > 2。
2、如果 [j + 1, i - 1] 不能形成区间，即 i - j <= 2 ，只需要判断 s[j] == s[i] 即可，因此考虑
“回文串两边各往里面收缩一个字符”的时候，二者之一成立即可。
```

于是整理成“状态转移方程：

```
dp[j, i] = (s[j] == s[i] and (i - j <= 2 or dp[j + 1, i - 1]))
因为要构成子串 j 一定小于等于 i ，我们只关心 “状态”数组“下三角”的那部分取值。理解上面的“状态转移方程”中的 (j >= i - 2 or dp[j + 1, i - 1]) 这部分是关键。
```

```java
public class Solution {

    public String longestPalindrome(String s) {
        int len = s.length();
        if(len < 2) return s;
        int maxLen = 1;
        String maxStr = s.substring(0, 1);
        boolean[][] dp = new boolean[len][len];
        // 如果 dp[j,i] = true 那么 dp[j+1,i-1] 也一定为 true
        // [j + 1,i - 1] 一定要构成至少两个元素额区间（ 1 个元素的区间，								s.charAt(i)==s.charAt(j) 已经判断过了）
        // 即 j + 1 < i - 1，即 i > j + 2 (不能取等号，取到等号，就退化成 1 个元素的情况了)
        // 应该反过来写
        for (int i = 0; i < len; i++) {
            for (int j = 0; j <= i; j++) {
                // 区间应该慢慢放大
                if (s.charAt(i) == s.charAt(j) && (i - j <= 2 || dp[j + 1][i - 1])) {
                    dp[j][i] = true;
                    if (i - j + 1 > maxLen) {
                        maxLen = i - j + 1;
                        maxStr = s.substring(j, i + 1);
                    }
                }
            }
        }
        return maxStr;
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
1, If p.charAt(j) == s.charAt(i) :  dp[i][j] = dp[i-1][j-1];
2, If p.charAt(j) == '.' : dp[i][j] = dp[i-1][j-1];
3, If p.charAt(j) == '*': 
   here are two sub conditions:
               1   if p.charAt(j-1) != s.charAt(i) : dp[i][j] = dp[i][j-2]  //in this case, a* only counts as empty
               2   if p.charAt(i-1) == s.charAt(i) or p.charAt(i-1) == '.':
                              dp[i][j] = dp[i-1][j]    
                              //in this case, a* counts as multiple a 
                           or dp[i][j] = dp[i][j-1]   
                           // in this case, a* counts as single a
                           or dp[i][j] = dp[i][j-2]   
                           // in this case, a* counts as empty
```

```java
public boolean isMatch(String s, String p) {
    if (s == null || p == null) {
        return false;
    }
    boolean[][] dp = new boolean[s.length()+1][p.length()+1];
    dp[0][0] = true;
    for (int i = 0; i < p.length(); i++) {
        if (p.charAt(i) == '*' && dp[0][i-1]) {
            dp[0][i+1] = true;
        }
    }
    for (int i = 0 ; i < s.length(); i++) {
        for (int j = 0; j < p.length(); j++) {
            if (p.charAt(j) == '.') {
                dp[i+1][j+1] = dp[i][j];
            }
            if (p.charAt(j) == s.charAt(i)) {
                dp[i+1][j+1] = dp[i][j];
            }
            if (p.charAt(j) == '*') {
                if (p.charAt(j-1) != s.charAt(i) && p.charAt(j-1) != '.') {
                    dp[i+1][j+1] = dp[i+1][j-1];
                } else {
                    dp[i+1][j+1] = (dp[i+1][j] || dp[i][j+1] || dp[i+1][j-1]);
                }
            }
        }
    }
    return dp[s.length()][p.length()];
}
```

**复杂度分析:**

**时间复杂度：**用 T 和 P 分别表示匹配串和模式串的长度。对于i = 0, ... , T 和 j = 0, ... , P 每一个 dp(i, j)只会被计算一次，所以后面每次调用都是 O(1)的时间。因此，总时间复杂度为 O(TP)。

**空间复杂度：**我们用到的空间仅有 O(TP) 个 boolean 类型的缓存变量。所以，空间复杂度为 O(TP)。

```java
class Solution {
    public boolean isMatch(String text, String pattern) {
        boolean[][] dp = new boolean[text.length() + 1][pattern.length() + 1];
        dp[text.length()][pattern.length()] = true;

        for (int i = text.length(); i >= 0; i--){
            for (int j = pattern.length() - 1; j >= 0; j--){
                boolean first_match = (i < text.length() &&
                                       (pattern.charAt(j) == text.charAt(i) ||
                                        pattern.charAt(j) == '.'));
                if (j + 1 < pattern.length() && pattern.charAt(j+1) == '*'){
                    dp[i][j] = dp[i][j+2] || first_match && dp[i+1][j];
                } else {
                    dp[i][j] = first_match && dp[i+1][j+1];
                }
            }
        }
        return dp[0][0];
    }
}
```



#### 杨辉三角-118

给定一个非负整数 *numRows，*生成杨辉三角的前 *numRows* 行。

![PascalTriangleAnimated2](E:\markdown笔记\图片\PascalTriangleAnimated2.gif)

```
输入: 5
输出:
[
     [1],
    [1,1],
   [1,2,1],
  [1,3,3,1],
 [1,4,6,4,1]
]
```

虽然这一算法非常简单，但用于构造杨辉三角的迭代方法可以归类为动态规划，因为我们需要基于前一行来构造每一行。

首先，我们会生成整个 triangle 列表，三角形的每一行都以子列表的形式存储。然后，我们会检查行数为 0 的特殊情况，否则我们会返回 [1][1]。如果 numRows > 0，那么我们用 [1][1] 作为第一行来初始化 triangle with [1][1]。

时间复杂度为：O (numRows^2)；

```java
public class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> triangle = new ArrayList<List<Integer>>();
        if (numRows <= 0)
            return triangle;
        for (int i=0; i<numRows; i++){
            List<Integer> row =  new ArrayList<Integer>();
            for (int j=0; j<i+1; j++){
                if (j==0 || j==i) row.add(1);
 				else {
                    row.add(triangle.get(i-1).get(j-1) + triangle.get(i-1).get(j));
                }
            }
            triangle.add(row);
        }
        return triangle;
    }
}
```

