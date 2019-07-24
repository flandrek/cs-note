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

在编写代码时，由于 [0..n]恰好是这个数组的下标加上 n，因此可以用一次循环完成所有的异或运算

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





#### 重塑矩阵-566

给出一个由二维数组表示的矩阵，以及两个正整数r和c，分别表示想要的重构的矩阵的行数和列数。重构后的矩阵需要将原始矩阵的所有元素以相同的行遍历顺序填充。如果具有给定参数的reshape操作是可行且合理的，则输出新的重塑矩阵；否则，输出原始矩阵。

```
输入: 
nums = 
[[1,2],
 [3,4]]
r = 1, c = 4
输出: 
[[1,2,3,4]]
解释:
行遍历nums的结果是 [1,2,3,4]。新的矩阵是 1 * 4 矩阵, 用之前的元素值一行一行填充新矩阵。
```

```
输入: 
nums = 
[[1,2],
 [3,4]]
r = 2, c = 4
输出: 
[[1,2],
 [3,4]]
解释:
没有办法将 2 * 2 矩阵转化为 2 * 4 矩阵。 所以输出原矩阵。
```

**思路：**先判断是否是合法的输入，然后用 res [count / c] [count % c] 表示要存入的数。

```java
class Solution {
    public int[][] matrixReshape(int[][] nums, int r, int c) {
        int row = nums.length, col = nums[0].length;
        if( r * c != row * col) return nums;
        int[][] res = new int[r][c];
        int count = 0;
        for(int i = 0; i < row; i++){
            for(int j = 0; j < col; j++){
                res[count/c][count%c] = nums[i][j];
                count++;
            }
        }
        return res;
    }
}
```



#### 查找和替换模式-890

你有一个单词列表 words 和一个模式  pattern，你想知道 words 中的哪些单词与模式匹配。

如果存在字母的排列 p ，使得将模式中的每个字母 x 替换为 p(x) 之后，我们就得到了所需的单词，那么单词与模式是匹配的。（回想一下，字母的排列是从字母到字母的双射：每个字母映射到另一个字母，没有两个字母映射到同一个字母。）

返回 words 中与给定模式匹配的单词列表，你可以按任何顺序返回答案。

```html
输入：words = ["abc","deq","mee","aqq","dkd","ccc"], pattern = "abb"
输出：["mee","aqq"]
解释：
"mee" 与模式匹配，因为存在排列 {a -> m, b -> e, ...}。
"ccc" 与模式不匹配，因为 {a -> c, b -> c, ...} 不是排列。
因为 a 和 b 映射到同一个字母。
```

**思路1：**如果可以模式替换的话，则模式和字符串中的字符是一一映射的。使用两个 Map 进行验证是否一一对应。

```java
class Solution {
    public List<String> findAndReplacePattern(String[] words, String pattern) {
        List<String> ans = new ArrayList();
        for (String word: words)
            if (match(word, pattern))
                ans.add(word);
        return ans;
    }

    public boolean match(String word, String pattern) {
        Map<Character, Character> m1 = new HashMap();
        Map<Character, Character> m2 = new HashMap();

        for (int i = 0; i < word.length(); ++i) {
            char w = word.charAt(i);
            char p = pattern.charAt(i);
            if (!m1.containsKey(w)) m1.put(w, p);
            if (!m2.containsKey(p)) m2.put(p, w);
            if (m1.get(w) != p || m2.get(p) != w)
                return false;
        }
        return true;
    }
}
```

**思路2：**只使用一个 Map

```java
class Solution {
    public List<String> findAndReplacePattern(String[] words, String pattern) {
        List<String> res = new ArrayList();   
        for(String word : words){
            if(match(word, pattern))
                res.add(word);
        }
        return res;
        
    }
    public boolean match(String word, String pattern){
        if(word.length() != pattern.length()) return false;
        Map<Character, Character> map = new HashMap();
        for(int i = 0; i < word.length(); i++){
            char s = word.charAt(i);
            char p = pattern.charAt(i);
            if(!map.containsKey(p)){
                if(map.containsValue(s)) return false; // val已经有了映射关系，直接返回 false
                map.put(p, s);
            }else if(map.get(p) != s)
              // key-val已经对应，且key对应的val不等于当前的字母，说明不是一一映射所以直接返回false
                return false;
        }
        return true;
    }
}
```

**思路3：**使用 indexOf ( ) 方法，判断当前出现的字符，在模式和字符串中下一次出现的位置是否相同。

```java
class Solution {
    public List<String> findAndReplacePattern(String[] words, String pattern) {
        List<String> res = new ArrayList();   
        for(String word : words){
            if(match(word, pattern))
                res.add(word);
        }
        return res;        
    }
    public boolean match(String word, String pattern){
        for(int i = 0; i < word.length(); i++){
            char s = word.charAt(i);
            char p = pattern.charAt(i);
            if(word.indexOf(s, i+1) != pattern.indexOf(p, i+1))
                return false;
        }
        return true;
    }
}
```





#### 反转字符串中的单词-557

给定一个字符串，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

```
输入: "Let's take LeetCode contest"
输出: "s'teL ekat edoCteeL tsetnoc" 
```

**思路1：**使用 StringBulider 的 reverse ( ) 方法。

```java
class Solution {
    public String reverseWords(String s) {
        String res = "";
        // StringBuilder sb = new StringBuilder();
        String[] strs = s.split(" ");
        for(int i = 0; i < strs.length; i++){
            if(i != strs.length - 1){
                res += new StringBuilder(strs[i]).reverse().append(" ").toString();
            }else{
                res += new StringBuilder(strs[i]).reverse().toString();
            }
        }
        return res;
    }
}
```

**思路2：**从头到尾分别交换单词中的每一个字符。

```java
class Solution {
    public String reverseWords(String s) {
        char[] chars = s.toCharArray();
        int start = 0;
        while(start < s.length()){
            int end = s.indexOf(" ", start + 1); // indexOf 方法返回从index开始查找指定串
            if(end == -1) end = s.length();
            reverse(chars, start, end);
            start = end + 1;
        }
        return String.valueOf(chars);
    }
    
    private void reverse(char[] chars, int start, int end){
        for(int i = start, j = end - 1; i < j; i++, j--){
            char temp = chars[i];
            chars[i] = chars[j];
            chars[j] = temp;
        }
    }   
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

另一种写法：

```java
class Solution {
    public List<String> commonChars(String[] A) {
        if(null == A || A.length == 0){
            return null;
        }
        int[] arr = toArray(A[0]);
        for(int i = 1;i < A.length;i++){
            arr = reduce(arr,toArray(A[i]));
        }
        ArrayList<String> str = new ArrayList<>();
        for(int i = 0;i < arr.length; i++){
            while(arr[i]-- > 0){
                str.add(String.valueOf((char)(i+'a')));
            }
        }
        return str;
    }
    public int[] toArray(String str){
        int[] curr = new int[26];
        for(int i = 0; i < str.length();i++){
            curr[str.charAt(i)-'a']++;
        }
        return curr;
    }
    public int[] reduce(int before[],int after[]){
        for(int i = 0; i < after.length;i++){
            before[i] = Math.min(before[i],after[i]);
        }
        return before;
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



#### 唯一摩尔斯密码值-804

国际摩尔斯密码定义一种标准编码方式，将每个字母对应于一个由一系列点和短线组成的字符串， 比如: "a" 对应 ".-", "b" 对应 "-...", "c" 对应 "-.-.", 等等。

为了方便，所有26个英文字母对应摩尔斯密码表如下：

```html
[".-","-...","-.-.","-..",".","..-.","--.","....","..",".---","-.-",".-..","--","-.","---",".--.","--.-",".-.","...","-","..-","...-",".--","-..-","-.--","--.."]
```

给定一个单词列表，每个单词可以写成每个字母对应摩尔斯密码的组合。例如，"cab" 可以写成 "-.-..--..."，(即 "-.-." + "-..." + ".-"字符串的结合)。我们将这样一个连接过程称作单词翻译。

返回我们可以获得所有词不同单词翻译的数量。

```html
例如:
输入: words = ["gin", "zen", "gig", "msg"]
输出: 2
解释: 
各单词翻译如下:
"gin" -> "--...-."
"zen" -> "--...-."
"gig" -> "--...--."
"msg" -> "--...--."
```

共有 2 种不同翻译, "--...-." 和 "--...--.".

**思路：**使用哈希表

```java
class Solution {
    private static String[] MORSE_CODE_TABLE = new String[]
            {".-","-...","-.-.","-..",".","..-.",
            "--.","....","..",".---","-.-",".-..",
            "--","-.","---",".--.","--.-",".-.",
            "...","-","..-","...-",".--","-..-",
            "-.--","--.."};
    public int uniqueMorseRepresentations(String[] words) {
        if (words == null || words.length == 0) {
            return 0;
        }

        Set<String> uniqueMorseCodes = new HashSet<>();

        for (String word : words) {
            if (word == null) {
                continue;
            }
            char[] chars = word.toCharArray();
            StringBuilder sb = new StringBuilder();
            for (char ch : chars) {
                sb.append(MORSE_CODE_TABLE[ch - 'a']);
            }
            uniqueMorseCodes.add(sb.toString());
        }
        return uniqueMorseCodes.size();
    }
}
```



#### 键盘行-500

给定一个单词列表，只返回可以使用在键盘同一行的字母打印出来的单词。键盘如下图所示。

![American keyboard](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/keyboard.png)

**思路：**使用哈希表将每一行的字母与行号对应存储，然后取字符串的第一个字符，得出其行号，然后依次与剩下的部分对比，如果全部都是同一行，则是要找的字符串。

```java
class Solution {
    public String[] findWords(String[] words) {
        String[] strs = {"QWERTYUIOP", "ASDFGHJKL", "ZXCVBNM"};
        Map<Character, Integer> map = new HashMap();
        for(int i = 0; i < strs.length; i++){
            for(char c : strs[i].toCharArray()){
                map.put(c, i);//put <char, rowIndex> pair into the map
            }
        }
        List<String> res = new ArrayList();
        for(String w : words){
            if(w.equals("")) continue;
            int index = map.get(w.toUpperCase().charAt(0));
            for(char c : w.toUpperCase().toCharArray()){
                if(map.get(c) != index){
                    index = -1;
                    break;
                }
            }
            if(index != -1) res.add(w);
        }
        return res.toArray(new String[res.size()]);
    }
}
```

另一种做法：

```java
class Solution {
    public String[] findWords(String[] words) {
        if (words == null) {
            return null;
        }
        List<String> ans = new ArrayList<>();
        // 字典行
        String lines[] = new String[] {
                "qwertyuiop",
                "asdfghjkl",
                "zxcvbnm"
        };

        // 挨个单词匹配是否满足条件
        for (String word : words) {
            //要想忽略大小写，统一toLowerCase或者toUpperCase都可以
            if(judge(word.toLowerCase(),lines)) {
                ans.add(word);
            }
        }
        // 刚看到的时候有点疑惑返回值为什么不是List<String>而是String[]
        // list可直接利用API转换为String数组即可------list转array和array转list都很重要！
        return ans.toArray(new String[ans.size()]);//分配一个数组空间，再去转换
    }

    private boolean judge(String word,String[] lines) {
        String find = null;
        // 先用word首字符确定属于哪一行
        for (String line : lines) {
            if (line.indexOf(word.charAt(0)) > -1) {//在line中看一下word的首字符存不存在！
                find = line;
                break;
            }
        }
        if (find == null) {//如果没找到
            return false;
        }
        // 判断word字符串所有字符是否都属于同一行
        for (int i = 1;i < word.length();i++) {
            //只要word中有字符在find中找不到，就返回false
            if (find.indexOf(word.charAt(i)) < 0) {
                return false;
            }
        }
        return true;
    }
}
```

