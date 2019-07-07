#### 有效的数独-36

判断一个 9 x 9 的数独是否有效。只需要根据以下规则，验证已经填入的数字是否有效即可。

1. 数字 1-9 在每一行只能出现一次。
2. 数字 1-9 在每一列只能出现一次。
3. 数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。

![250px-Sudoku-by-L2G-20050714.svg](E:\markdown笔记\图片\250px-Sudoku-by-L2G-20050714.svg.png)

```html
输入:
[
  ["8","3",".",".","7",".",".",".","."],
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  [".",".",".",".","8",".",".","7","9"]
]
输出: false
解释: 除了第一行的第一个数字从 5 改为 8 以外，空格内其他数字均与 示例1 相同。
     但由于位于左上角的 3x3 宫内有两个 8 存在, 因此这个数独是无效的。
```

**思路：**时间复杂度和空间复杂度均为 O (1)

一个简单的解决方案是遍历该 9 x 9 数独 三 次，以确保：

1. 行中没有重复的数字。
2. 列中没有重复的数字。
3. 3 x 3 子数独内没有重复的数字。

枚举子类：可以使用 `box_index = (row / 3) * 3 + columns / 3`，其中 `/` 是整数除法。

![1561796327(1)](E:\markdown笔记\图片\1561796327(1).png)

遍历数独。检查看到每个单元格值是否已经在当前的行 / 列 / 子数独中出现过：

1. 如果出现重复，返回 false。
2. 如果没有，则保留此值以进行进一步跟踪。
3. 返回 true。

```java
class Solution {
      public boolean isValidSudoku(char[][] board) {
            // init data
            HashMap<Integer, Integer> [] rows = new HashMap[9];
            HashMap<Integer, Integer> [] columns = new HashMap[9];
            HashMap<Integer, Integer> [] boxes = new HashMap[9];
            for (int i = 0; i < 9; i++) {
                  rows[i] = new HashMap<Integer, Integer>();
                  columns[i] = new HashMap<Integer, Integer>();
                  boxes[i] = new HashMap<Integer, Integer>();
            }
            // validate a board
            for (int i = 0; i < 9; i++) {
              	for (int j = 0; j < 9; j++) {
                	char num = board[i][j];
                    if (num != '.') {
                      int n = (int)num;
                      int box_index = (i / 3 ) * 3 + j / 3;

                      // keep the current cell value
                      rows[i].put(n, rows[i].getOrDefault(n, 0) + 1);
                      columns[j].put(n, columns[j].getOrDefault(n, 0) + 1);
                      boxes[box_index].put(n, boxes[box_index].getOrDefault(n, 0) + 1);

                      // check if this value has been already seen before
                      if (rows[i].get(n) > 1 || columns[j].get(n) > 1 || 
                                           		 boxes[box_index].get(n) > 1)
                        return false;
                	}
              	}
            }
            return true;
      }
}
```

利用 Set 特性，Set.add( ) 时如果已经存在则返回 false，否则添加成功且返回 true；

```java
public boolean isValidSudoku(char[][] board) {
    Set seen = new HashSet();
    for (int i = 0; i < 9; ++i) {
        for (int j=0; j<9; ++j) {
            char number = board[i][j];
            if (number != '.')
                if (!seen.add(number + " in row " + i) ||
                    !seen.add(number + " in column " + j) ||
                    !seen.add(number + " in block " + i/3 + "-" + j/3))
                    return false;
        }
    }
    return true;
}
```

用数组代替哈希表：

```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        boolean[][] row = new boolean[9][9];	//记录某行
        boolean[][] col = new boolean[9][9];	//记录某列
        boolean[][] block = new boolean[9][9];	//记录某3*3
        for(int i = 0; i < 9; i++)
            for(int j = 0; j < 9; j++){
                if(board[i][j] != '.'){
                    int num = board[i][j] - '1';
                    int blockIndex = i/3*3 + j/3;
                    if(row[i][num] || col[num][j] || block[blockIndex][num]){
                        return false;
                    }else{
                        row[i][num] = true;
                        col[num][j] = true;
                        block[blockIndex][num] = true;
                    }
                }
            }
        return true;
    }
}
```



#### 快乐数-202

编写一个算法来判断一个数是不是“快乐数”。

一个“快乐数”定义为：对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和，然后重复这个过程直到这个数变为 1，也可能是无限循环但始终变不到 1。如果可以变为 1，那么这个数就是快乐数。

```html
输入: 19
输出: true
解释: 
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

**思路：**

使用一个 Set 或者 List 判断是否陷入循环中；

```java
import java.util.*;
class Solution {
    public boolean isHappy(int n) {
        List<Integer> list = new ArrayList<>();
        list.add(n);
        int temp = n;
        while(temp != 1){
            int res = 0;
            while(temp != 0){
                res += (temp % 10) * (temp % 10);
                temp /= 10;
            }
            if(res == 1) return true;
            if(list.contains(res)) return false;
            else list.add(res);
            temp = res;
        }
        return true;
    }
}
```

**贪心：**

不是快乐数的数称为不快乐数（unhappy number），所有不快乐数的数位平方和计算，最後都会进入 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 的循环中。

```java
class Solution {
    public boolean isHappy(int n) {
        int x = n;
        int sum = 0;
        if(n == 1) return true;
        while(sum != 1) {
            sum = 0;
            while(x != 0) {
                int y = x % 10;
                sum += y * y;
                x = x / 10;
            }
            if(sum == n) return false;
            if(sum == 4) return false;
            x = sum;
        }
        return true;
    }
}
```



#### 有效的字母异位词-242

给定两个字符串 *s* 和 *t* ，编写一个函数来判断 *t* 是否是 *s* 的字母异位词。

```
输入: s = "anagram", t = "nagaram"
输出: true
输入: s = "rat", t = "car"
输出: false
```

**思路：**使用 hash 表，时间复杂度 O (n)，空间复杂度为 O (1)；

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) {
        return false;
    }
    int[] counter = new int[26];
    for (int i = 0; i < s.length(); i++) {
        counter[s.charAt(i) - 'a']++;
        counter[t.charAt(i) - 'a']--;
    }
    for (int count : counter) {
        if (count != 0) {
            return false;
        }
    }
    return true;
}
```

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) {
        return false;
    }
    int[] table = new int[26];
    for (int i = 0; i < s.length(); i++) {
        table[s.charAt(i) - 'a']++;
    }
    for (int i = 0; i < t.length(); i++) {
        table[t.charAt(i) - 'a']--;
        if (table[t.charAt(i) - 'a'] < 0) {
            return false;
        }
    }
    return true;
}
```



#### 四数相加II-454

给定四个包含整数的数组列表 A , B , C , D ,计算有多少个元组 (i, j, k, l) ，使得 A[i] + B[j] + C[k] + D[l] = 0。

```html
输入:
A = [ 1, 2]
B = [-2,-1]
C = [-1, 2]
D = [ 0, 2]
输出:
2
解释:
两个元组如下:
1. (0, 0, 0, 1) -> A[0] + B[0] + C[0] + D[1] = 1 + (-2) + (-1) + 2 = 0
2. (1, 1, 0, 0) -> A[1] + B[1] + C[0] + D[0] = 2 + (-1) + (-1) + 0 = 0
```

**思路：**使用哈希表

```java
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        Map<Integer,Integer> map = new HashMap<>();
        int count = 0;
        for(int i = 0; i < A.length; i++){
            for(int j = 0; j < B.length; j++){
                int sum = A[i] + B[j];
                map.put(sum, map.getOrDefault(sum,0) + 1);
            }
        }
        for(int i = 0; i < C.length; i++){
            for(int j = 0; j < D.length; j++){
                int sum = C[i] + D[j];
                count += map.getOrDefault(-sum,0);
            }
        }
        return count;
    }
}
```



#### 缺失的数字-268

给定一个包含 `0, 1, 2, ..., n` 中 *n* 个数的序列，找出 0 .. *n* 中没有出现在序列中的那个数。

```
输入: [3,0,1]
输出: 2
输入: [9,6,4,2,3,5,7,0,1]
输出: 8
```

**思路：**

我们知道数组中有 nn 个数，并且缺失的数在 [0..n]中。因此我们可以先得到 [0..n] 的异或值，再将结果对数组中的每一个数进行一次异或运算。未缺失的数在 [0..n] 和数组中各出现一次，因此异或后得到 0。而缺失的数字只在 [0..n] 中出现了一次，在数组中没有出现，因此最终的异或结果即为这个缺失的数字。

在编写代码时，由于 [0..n]恰好是这个数组的下标加上 nn，因此可以用一次循环完成所有的异或运算

```java
class Solution {
    public int missingNumber(int[] nums) {
        int missing = nums.length;
        for (int i = 0; i < nums.length; i++) {
            missing ^= i ^ nums[i];
        }
        return missing;
    }
}
```

**哈希表：**

我们可以直接查询每个数是否在数组中出现过来找出缺失的数字。如果使用哈希表，那么每一次查询操作都是常数时间的。

```java
class Solution {
    public int missingNumber(int[] nums) {
        Set<Integer> numSet = new HashSet<Integer>();
        for (int num : nums) numSet.add(num);

        int expectedNumCount = nums.length + 1;
        for (int number = 0; number < expectedNumCount; number++) {
            if (!numSet.contains(number)) {
                return number;
            }
        }
        return -1;
    }
}
```



#### 常数时间插入删除和获取随机元素-380

设计一个支持在平均 时间复杂度 O(1) 下，执行以下操作的数据结构。

1. insert(val)：当元素 val 不存在时，向集合中插入该项。
2. remove(val)：元素 val 存在时，从集合中移除该项。
3. getRandom：随机返回现有集合中的一项。每个元素应该有相同的概率被返回。

```java
// 初始化一个空的集合。
RandomizedSet randomSet = new RandomizedSet();

// 向集合中插入 1 。返回 true 表示 1 被成功地插入。
randomSet.insert(1);

// 返回 false ，表示集合中不存在 2 。
randomSet.remove(2);

// 向集合中插入 2 。返回 true 。集合现在包含 [1,2] 。
randomSet.insert(2);

// getRandom 应随机返回 1 或 2 。
randomSet.getRandom();

// 从集合中移除 1 ，返回 true 。集合现在包含 [2] 。
randomSet.remove(1);

// 2 已在集合中，所以返回 false 。
randomSet.insert(2);

// 由于 2 是集合中唯一的数字，getRandom 总是返回 2 。
randomSet.getRandom();
```

**思路：**常数时间复杂度，考虑使用哈希表。

1. 使用 Set 和 数组 实现：

```java
class RandomizedSet {
    
    private Set<Integer> set;

    /** Initialize your data structure here. */
    public RandomizedSet() {
        set = new HashSet<>();
    }
    
    /** Inserts a value to the set. Returns true if the set did not already contain the 		specified element. */
    public boolean insert(int val) {
        if(set.contains(val)){
            return false;
        }
        else{
            set.add(val);
            return true;
        }
    }
    
    /** Removes a value from the set. Returns true if the set contained the specified 			element. */
    public boolean remove(int val) {
        if(set.contains(val)){
            set.remove(val);
            return true; 
        }
        else{
            return false;
        }
    }
    
    /** Get a random element from the set. */
    public int getRandom() {
        if(set.size() == 0){
            return 0;
        }
        Integer[] data = set.toArray(new Integer[set.size()]);
        int index = (int)(Math.random()*set.size());
        return data[index];
    }
}
```

2. 使用 HashMap 和 ArrayList 实现，ArrayList 删除最后一个元素的时间复杂度为 O (1)，因此用一个 HashMap 来存储元素对应的下标位置：

```java
class RandomizedSet {
    Random rand;
    Map<Integer, Integer> map;
    List<Integer> keys;
    int count;
    
    /** Initialize your data structure here. */
    public RandomizedSet() {
        rand = new Random();
        map = new HashMap<Integer, Integer>();// (key, list index)
        keys = new ArrayList<>();
        count = 0;
    }
    
    /** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
    public boolean insert(int val) {
        if(map.containsKey(val)){   return false;}
        keys.add(val);
        map.put(val, count++);
        return true;
    }
    
    /** Removes a value from the set. Returns true if the set contained the specified element. */
    public boolean remove(int val) {
        if(!map.containsKey(val)) {return false;}
        int index = map.get(val);
        int last = count-1;
        if(index != last){
            //update keys(list)   list at index==(lastkey)
            keys.set(index, keys.get(last));       
            //update map: (lastkey, index) lastkey's index in keys(list)    
            map.put(keys.get(last), index);        
        }
        keys.remove(last);
        map.remove(val);
        count--;
        return true;
    }
    
    /** Get a random element from the set. */
    public int getRandom() {
        return keys.get(rand.nextInt(count));
    }
}
```

