#### **跳跃游戏**

给定一个非负整数数组，你最初位于数组的第一个位置。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个位置。

解：我们记录一个的坐标代表当前可达的最后节点，这个坐标初始等于数组长度 -1， 然后我们每判断完是否可达，都向前移动这个坐标，直到遍历结束。

```java
public class Solution {
    public boolean canJump(int[] nums) {
        int lastPos = nums.length - 1;
        for (int i = nums.length - 1; i >= 0; i--) {
            if (i + nums[i] >= lastPos) {
                lastPos = i;
            }
        }
        return lastPos == 0;
    }
}
```



#### 任务调度器-621

给定一个用字符数组表示的 CPU 需要执行的任务列表。其中包含使用大写的 A - Z 字母表示的26 种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。CPU 在任何一个单位时间内都可以执行一个任务，或者在待命状态。

然而，两个相同种类的任务之间必须有长度为 n 的冷却时间，因此至少有连续 n 个单位时间内 CPU 在执行不同的任务，或者在待命状态。

你需要计算完成所有任务所需要的最短时间。

```
输入: tasks = ["A","A","A","B","B","B"], n = 2
输出: 8
执行顺序: A -> B -> (待命) -> A -> B -> (待命) -> A -> B.
```

思路：

- 假设数组 ["A","A","A","B","B","C"]，n = 2，A的频率最高，记为count = 3，所以两个A之间必须间隔2个任务，才能满足题意并且是最短时间（两个A的间隔大于2的总时间必然不是最短），因此执行顺序为： A->X->X->A->X->X->A，这里的X表示除了A以外其他字母，或者是待命，不用关心具体是什么，反正用来填充两个A的间隔的。上面执行顺序的规律是： 有count - 1个A，其中每个A需要搭配n个X，再加上最后一个A，所以总时间为 **(count - 1) * (n + 1) + 1**；
- 要注意可能会出现多个频率相同且都是最高的任务，比如 ["A","A","A","B","B","B","C","C"]，所以最后会剩下一个A和一个B，因此最后要加上频率最高的不同任务的个数 maxCount；
- 公式算出的值可能会比数组的长度小，如["A","A","B","B"]，n = 0，此时要取数组的长度；

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        int[] count = new int[26];
        for (int i = 0; i < tasks.length; i++) {
            count[tasks[i]-'A']++;
        }//统计词频
        Arrays.sort(count);//词频排序，升序排序，count[25]是频率最高的
        int maxCount = 0;
        //统计有多少个频率最高的字母
        for (int i = 25; i >= 0; i--) {
            if(count[i] != count[25]){s
                break;
            }
            maxCount++;
        }
        //公式算出的值可能会比数组的长度小，取两者中最大的那个
        return Math.max((count[25] - 1) * (n + 1) + maxCount , tasks.length);
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



#### 3 的幂-326

给定一个整数，写一个函数来判断它是否是 3 的幂次方。

**进阶：**你能不使用循环或者递归来完成本题吗？

**迭代：**

```java
class Solution {
    public boolean isPowerOfThree(int n) {
        if(n < 1) return false;
        while(n != 1){
            if(n % 3 != 0) 
                return false;
            n /= 3;
        }
        return true;
    }
}
```

**贪心：**3的幂次的质因子只有3，而所给出的n如果也是3的幂次，故而题目中所给整数范围内最大的3的幂次的因子只能是3的幂次，1162261467是3的19次幂，是整数范围内最大的3的幂次

```java
class Solution {
    public boolean isPowerOfThree(int n) {
        return n > 0 && 1162261467 % n == 0;
    }
}
```



#### 除数博弈-1025

爱丽丝和鲍勃一起玩游戏，他们轮流行动。爱丽丝先手开局。

最初，黑板上有一个数字 N 。在每个玩家的回合，玩家需要执行以下操作：

1. 选出任一 x，满足 0 < x < N 且 N % x == 0 。

2. 用 N - x 替换黑板上的数字 N 。

   如果玩家无法执行这些操作，就会输掉游戏。

只有在爱丽丝在游戏中取得胜利时才返回 True，否则返回 false。假设两个玩家都以最佳状态参与游戏。

```
输入：2
输出：true
解释：爱丽丝选择 1，鲍勃无法进行操作。
```

```
输入：3
输出：false
解释：爱丽丝选择 1，鲍勃也选择 1，然后爱丽丝无法进行操作。
```

**思路：**

1. 数字N如果是奇数，它的约数必然都是奇数；若为偶数，则其约数可奇可偶。
2. 无论N初始为多大的值，游戏最终只会进行到N=2时结束，那么谁轮到N=2时谁就会赢。
3. 因为爱丽丝先手，N初始若为偶数，爱丽丝则只需一直选1，使鲍勃一直面临N为奇数的情况，这样爱丽丝
4. 赢； N初始若为奇数，那么爱丽丝第一次选完之后N必为偶数，那么鲍勃只需一直选1就会稳赢。
   综述，判断N是奇数还是偶数，即可得出最终结果！

```java
class Solution {
    public boolean divisorGame(int N) {
        if(N % 2 == 0) return true;
        return false;
    }
}
```



#### 整数拆分-343

给定一个正整数 *n*，将其拆分为**至少**两个正整数的和，并使这些整数的乘积最大化。 返回你可以获得的最大乘积。

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1。

输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36。
```

**思路1：**动态规划

使用动态规划的思想：

1、定义函数f(n)表示为把长度为n的绳子剪成若干段后各段长度乘积的最大值。 

2、对于第一刀，我们有n-1种可能的选择，可推导出 f(n) = max{ f(i) * f(n-i) }; 

3、很明显这是一个从上至下的递归，但是这个递归存在很多重复的计算，所以使用至下而上的动态规划，将子问题的最优解保存。 

4、注意绳子剪成 i x (n-i) 和 (n-i) x i是相同的； 

5、注意不符合切割条件的输入n，以及输入为2、3长度时的结果，因为题中规定 m > 1。

```java
class Solution {
    public int integerBreak(int n) {
        if(n < 2) return 0;
        if(n == 2) return 1;
        if(n == 3) return 2;      
        int[] dp = new int[n+1];
        dp[0] = 0;
        dp[1] = 1;
        dp[2] = 2;
        dp[3] = 3;       
        for(int i = 4; i <= n; i++){
            for(int j = 1; j <= i/2; j++){
                dp[i] = Math.max(dp[i], dp[j]*dp[i-j]);
            }
        }
        return dp[n];
    }
}
```

**思路2：**贪心

当 n >= 5 时，尽可能多地剪长度为3的绳子；当剩下的绳子长度为4时，把绳子剪成两段长度为2的绳子。

```java
class Solution {
    public int integerBreak(int n) {
        if(n == 1) return 1;
        if(n == 2) return 1;
        if(n == 3) return 2;
        if(n == 4) return 4;
        int res = 1;
        while(n > 4){
            res *= 3;
            n -= 3;
        }
        return res * n;
    }
}
```

