#### 无重复字符的最长子串

给定一个字符串，请你找出其中不含有重复字符的**最长子串**的长度。

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**1.滑动窗口**

通过使用 Hash Set 作为滑动窗口，我们可以用 O(1)的时间来完成对字符是否在当前的子字符串中的检查。回到我们的问题，我们使用 Hash Set 将字符存储在当前窗口 [i, j)（最初 j = i）中。 然后我们向右侧滑动索引 j，如果它不在 Hash Set 中，我们会继续滑动 j。直到 s[j] 已经存在于 Hash Set 中。此时，我们找到的没有重复字符的最长子字符串将会以索引 i 开头。如果我们对所有的 ii 这样做，就可以得到答案。

```java
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length();
        Set<Character> set = new HashSet<>();
        int ans = 0, i = 0, j = 0;
        while (i < n && j < n) {
            // try to extend the range [i, j]
            if (!set.contains(s.charAt(j))){
                set.add(s.charAt(j++));
                ans = Math.max(ans, j - i);
            } else {
                set.remove(s.charAt(i++));
            }
        }
        return ans;
    }
}
```

**2.优化的滑动窗口**

上述的方法最多需要执行 2n 个步骤。事实上，它可以被进一步优化为仅需要 n 个步骤。我们可以定义字符到索引的映射，而不是使用集合来判断一个字符是否存在。 当我们找到重复的字符时，我们可以立即跳过该窗口。也就是说，如果 s[j] 在 [i, j) 范围内有与 j' 重复的字符，我们不需要逐渐增加 ii 。 我们可以直接跳过 [i，j']范围内的所有元素，并将 i 变为 j' + 1。

```java
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length(), ans = 0;
        Map<Character, Integer> map = new HashMap<>(); // current index of character
        // try to extend the range [i, j]
        for (int j = 0, i = 0; j < n; j++) {
            if (map.containsKey(s.charAt(j))) {
                i = Math.max(map.get(s.charAt(j)), i);
            }
            ans = Math.max(ans, j - i + 1);
            map.put(s.charAt(j), j + 1);
        }
        return ans;
    }
}
```

```java
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length(), ans = 0;
        int[] index = new int[128]; // current index of character
        // try to extend the range [i, j]
        for (int j = 0, i = 0; j < n; j++) {
            i = Math.max(index[s.charAt(j)], i);
            ans = Math.max(ans, j - i + 1);
            index[s.charAt(j)] = j + 1;
        }
        return ans;
    }
}
```



**2.二进制相加**

给定两个二进制字符串，返回他们的和（用二进制表示），输入为**非空**字符串且只包含数字 `1` 和 `0`。

```
输入: a = "11", b = "1"
输出: "100"
```

```java
public class Solution {
    public String addBinary(String a, String b) {
        int m = a.length() -1;
        int n = b.length() -1;
        int temp = 0;
        StringBuilder sb = new StringBuilder();
        while(temp != 0 || m >= 0 || n >= 0){
            if(m >= 0 && a.charAt(m--) == '1')
                temp++;
            if(n >= 0 && b.charAt(n--) == '1')
                temp++;
            sb.append(temp % 2);
            temp /= 2;
        }
        return sb.reverse().toString();
    }
}
```



**2.正则表达式匹配**

给你一个字符串 `s` 和一个字符规律 `p`，请你来实现一个支持 `'.'` 和 `'*'` 的正则表达式匹配，所谓匹配，是要涵盖 **整个** 字符串 `s`的，而不是部分字符串。

```
'.' 匹配任意单个字符
'*' 匹配零个或多个前面的那一个元素
```

```
输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
```

```
输入:
s = "ab"
p = ".*"
输出: true
解释: ".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。
```

**方法一：回溯**

如果模式串中有星号，它会出现在第二个位置，即pattern[1] 。这种情况下，我们可以直接忽略模式串中这一部分，或者删除匹配串的第一个字符，前提是它能够匹配模式串当前位置字符，即pattern[0] 。如果两种操作中有任何一种使得剩下的字符串能匹配，那么初始时，匹配串和模式串就可以被匹配。

```java
class Solution {
    public boolean isMatch(String text, String pattern) {
        if (pattern.isEmpty()) return text.isEmpty();
        boolean first_match = (!text.isEmpty() &&
                               (pattern.charAt(0) == text.charAt(0) || 
                                pattern.charAt(0) == '.'));
        if (pattern.length() >= 2 && pattern.charAt(1) == '*'){
            return (isMatch(text, pattern.substring(2)) ||
                    (first_match && isMatch(text.substring(1), pattern)));
        } else {
            return first_match && isMatch(text.substring(1), pattern.substring(1));
        }
    }
}
```

**方法二：动态规划**

我们用 [方法 1] 中同样的回溯方法，除此之外，因为函数 match(text[i:], pattern[j:]) 只会被调用一次，我们用dp(i, j) 来应对剩余相同参数的函数调用，这帮助我们节省了字符串建立操作所需要的时间，也让我们可以将中间结果进行保存。

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



#### 最后一个单词的长度

给定一个仅包含大小写字母和空格 ' ' 的字符串，返回其最后一个单词的长度，如果不存在最后一个单词，请返回 0 。说明：一个单词是指由字母组成，但不包含任何空格的字符串。

```
输入: "Hello World"
输出: 5
```

```java
public class Solution {
    public int lengthOfLastWord(String s) {
        if(s == null || s.length() == 0) return 0;
        int end = s.length() - 1;
        while (end >= 0 && s.charAt(end) == ' ') end--;
        if (end == -1) return 0;
        int start = end;
        while (start >= 0 && s.charAt(start) != ' ') start--;
        return end - start;
    }
}
```



#### 字母异位词分组-49

给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。

```
输入: ["eat", "tea", "tan", "ate", "nat", "bat"],
输出:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]
```

**排序数组分类：**

维护一个映射 ans : {String -> List}，其中每个键 K 是一个排序字符串，每个值是初始输入的字符串列表，排序后等于 K，在 Java 中，我们将键存储为字符串，例如，code。 在 Python 中，我们将键存储为散列化元组，例如，('c', 'o', 'd', 'e')。

![Anagrams](https://pic.leetcode-cn.com/Figures/49/49_groupanagrams1.png)

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        if (strs.length == 0) return new ArrayList();
        Map<String, List> ans = new HashMap<String, List>();
        for (String s : strs) {
            char[] ca = s.toCharArray();
            Arrays.sort(ca);
            String key = String.valueOf(ca);
            if (!ans.containsKey(key)) ans.put(key, new ArrayList());
            ans.get(key).add(s);
        }
        return new ArrayList(ans.values());
    }
}
```

```
时间复杂度：O(NKlogK)，其中 N 是 strs 的长度，而 K 是 strs 中字符串的最大长度。当我们遍历每个字符串时，外部循环具有的复杂度为 O(N)。然后，我们在O(KlogK) 的时间内对每个字符串排序。
空间复杂度：O(NK)，排序存储在 ans 中的全部信息内容。
```

**按计数分类：**

我们可以将每个字符串s 转换为字符数 count，由26个非负整数组成，表示 a，b，c 的数量等。我们使用这些计数作为哈希映射的基础。在 Java 中，我们的字符数 count 的散列化表示将是一个用 **＃** 字符分隔的字符串。 例如，abbccc 将表示为 ＃1＃2＃3＃0＃0＃0 ...＃0，其中总共有26个条目。 在 python 中，表示将是一个计数的元组。 例如，abbccc 将表示为 (1,2,3,0,0，...，0)，其中总共有 26 个条目。

![Anagrams](https://pic.leetcode-cn.com/Figures/49/49_groupanagrams2.png)

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        if (strs.length == 0) return new ArrayList();
        Map<String, List> ans = new HashMap<String, List>();
        int[] count = new int[26];
        for (String s : strs) {
            Arrays.fill(count, 0);
            for (char c : s.toCharArray()) count[c - 'a']++;

            StringBuilder sb = new StringBuilder("");
            for (int i = 0; i < 26; i++) {
                sb.append('#');
                sb.append(count[i]);
            }
            String key = sb.toString();
            if (!ans.containsKey(key)) ans.put(key, new ArrayList());
            ans.get(key).add(s);
        }
        return new ArrayList(ans.values());
    }
}
```

```
时间复杂度：O(NK)，其中 N 是 strs 的长度，而 K 是 strs 中字符串的最大长度。计算每个字符串的字符串大小是线性的，我们统计每个字符串。
空间复杂度：O(NK)，排序存储在 ans 中的全部信息内容。
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

**中心扩散法：**时间复杂度 O (n2)，空间复杂度 O (1)。

事实上，只需使用恒定的空间，我们就可以在O(n 2) 的时间内解决这个问题。

我们观察到回文中心的两侧互为镜像。因此，回文可以从它的中心展开，并且只有 2n - 1个这样的中心。

![LeetCode 第 5 题：中心扩散法](https://pic.leetcode-cn.com/1774184e1cfacbce61865edbd7343ad8c84792312d40bacb8170a2bd126b8f5c-image.png)

我们完全可以设计一个方法，兼容以上两种情况：

1. 如果传入重合的索引编码，进行中心扩散，此时得到的最长回文子串的长度是奇数；
2. 如果传入相邻的索引编码，进行中心扩散，此时得到的最长回文子串的长度是偶数。

```java
class Solution {
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 1) return "";
        int start = 0, end = 0;
        for (int i = 0; i < s.length(); i++) {
            int len1 = expandAroundCenter(s, i, i);
            int len2 = expandAroundCenter(s, i, i + 1);
            int len = Math.max(len1, len2);
            if (len > end - start) {
                start = i - (len - 1) / 2;
                end = i + len / 2;
            }
        }
        return s.substring(start, end + 1);
	}

    private int expandAroundCenter(String s, int left, int right) {
        int L = left, R = right;
        while (L >= 0 && R < s.length() && s.charAt(L) == s.charAt(R)) {
            L--;
            R++;
        }
        return R - L - 1;
    }
}
```

```java
public class Solution {
	private int lo, maxLen;
    public String longestPalindrome(String s) {
        int len = s.length();
        if (len < 2) return s;
        for (int i = 0; i < len-1; i++) {
            extendPalindrome(s, i, i); 
            extendPalindrome(s, i, i+1); 
        }
        return s.substring(lo, lo + maxLen);
    }

    private void extendPalindrome(String s, int j, int k) {
        while (j >= 0 && k < s.length() && s.charAt(j) == s.charAt(k)) {
            j--;
            k++;
        }
        if (maxLen < k - j - 1) {
            lo = j + 1;
            maxLen = k - j - 1;
        }
	}
}
```

**动态规划：**

1. 定义 “状态”，这里 “状态”数组是二维数组。dp[j] [i] 表示子串 s [j，i]（包区间左右端点）是否构成回文串。如果是回文串，那么 dp[j] [i] = true;

2. 状态转移方程，

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
           // [j + 1,i - 1] 一定要构成至少两个元素额区间（ 1 个元素的区间，								s.charAt(i) == s.charAt(j) 已经判断过了）
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



#### 回文数-9

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

```
输入: 121
输出: true
输入: -121
输出: false
解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
```

**方法：反转一半数字**

**思路：**

映入脑海的第一个想法是将数字转换为字符串，并检查字符串是否为回文。但是，这需要额外的非常量空间来创建问题描述中所不允许的字符串。

第二个想法是将数字本身反转，然后将反转后的数字与原始数字进行比较，如果它们是相同的，那么这个数字就是回文。 但是，如果反转后的数字大于 \text{int.MAX}int.MAX，我们将遇到整数溢出问题。

按照第二个想法，为了避免数字反转可能导致的溢出问题，为什么不考虑只反转 \text{int}int 数字的一半？毕竟，如果该数字是回文，其后半部分反转后应该与原始数字的前半部分相同。

例如，输入 1221，我们可以将数字 “1221” 的后半部分从 “21” 反转为 “12”，并将其与前半部分 “12” 进行比较，因为二者相同，我们得知数字 1221 是回文。

让我们看看如何将这个想法转化为一个算法。

**算法：**

首先，我们应该处理一些临界情况。所有负数都不可能是回文，例如：-123 不是回文，因为 - 不等于 3。所以我们可以对所有负数返回 false。

现在，让我们来考虑如何反转后半部分的数字。 对于数字 1221，如果执行 1221 % 10，我们将得到最后一位数字 1，要得到倒数第二位数字，我们可以先通过除以 10 把最后一位数字从 1221 中移除，1221 / 10 = 122，再求出上一步结果除以 10 的余数，122 % 10 = 2，就可以得到倒数第二位数字。如果我们把最后一位数字乘以 10，再加上倒数第二位数字，1 * 10 + 2 = 12，就得到了我们想要的反转后的数字。如果继续这个过程，我们将得到更多位数的反转数字。

现在的问题是，我们如何知道反转数字的位数已经达到原始数字位数的一半？

我们将原始数字除以 10，然后给反转后的数字乘上 10，所以，当原始数字小于反转后的数字时，就意味着我们已经处理了一半位数的数字。

```java
public class Solution {
    public bool IsPalindrome(int x) {
        // 特殊情况：
        // 如上所述，当 x < 0 时，x 不是回文数。
        // 同样地，如果数字的最后一位是 0，为了使该数字为回文，
        // 则其第一位数字也应该是 0
        // 只有 0 满足这一属性
        if(x < 0 || (x % 10 == 0 && x != 0)) {
            return false;
        }
        int revertedNumber = 0;
        while(x > revertedNumber) {
            revertedNumber = revertedNumber * 10 + x % 10;
            x /= 10;
        }

        // 当数字长度为奇数时，我们可以通过 revertedNumber/10 去除处于中位的数字。
        // 例如，当输入为 12321 时，在 while 循环的末尾我们可以得到 x = 12，
        	// revertedNumber = 123，
        // 由于处于中位的数字不影响回文（它总是与自己相等），所以我们可以简单地将其去除。
        return x == revertedNumber || x == revertedNumber/10;
    }
}
```



#### 实现 strStr( ) -28

实现 strStr() 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

```
输入: haystack = "hello", needle = "ll"
输出: 2

输入: haystack = "aaaaa", needle = "bba"
输出: -1
```

**思路 1：**暴力法，时间复杂度：O((m*−*n)n)，*m* 是主字符串，*n* 是模式字符串。

```java
class Solution {
    public int strStr(String S, String T) {
        int n1 = S.length();
        int n2 = T.length();
        if (n1 < n2) return -1;
        else if ( n2 == 0) return 0;
        for (int i = 0; i < n1 - n2 + 1; i++ ){
            if (S.substring(i, i+n2).equals(T)) return i;
        }
        return -1;
    }
}
```

**思路2：**

1. 设置两个指针i和j，分别用于指向主串(haystack)和模式串(needle)；
2. 从左到右开始一个个字符匹配；
3. 如果主串和模式串的两字符相等，则 i 和 j 同时后移比较下一个字符；
4. 如果主串和模式串的两字符不相等，就跳回去，即模式串向右移动，同时模式串指针归零 (i = 1; j = 0)；
5. 所有字符匹配结束后，如果模式串指针指到串尾 ( j = needle.length)，说明完全匹配，此时模式串出现的第一个第一个位置为：i - j

```java
class Solution {
    public int strStr(String haystack, String needle) {
        char[] hayArr = haystack.toCharArray();
        char[] needArr = needle.toCharArray();
        int i = 0; //主串(haystack)的位置
        int j = 0; //模式串(needle)的位置
        while (i < hayArr.length && j < needArr.length) {
            if (hayArr[i] == needArr[j]) { // 当两个字符相等则比较下一个
                i++;
                j++;
            } else {
                i = i - j + 1; // 一旦不匹配，i后退
                j = 0; // j归零
            }
        }
        if (j == needArr.length) { //说明完全匹配
            return i - j;
        } else {
            return -1;
        }
    }
}
```



#### 最长公共前缀-14

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

```
输入: ["flower","flow","flight"]
输出: "fl"

输入: ["dog","racecar","car"]
输出: ""
解释: 输入不存在公共前缀。
```

**思路：**

1. 当字符串数组长度为 0 时则公共前缀为空，直接返回；
2. 令最长公共前缀 ans 的值为第一个字符串，进行初始化；
3. 遍历后面的字符串，依次将其与 ans 进行比较，两两找出公共前缀，最终结果即为最长公共前缀；
4. 如果查找过程中出现了 ans 为空的情况，则公共前缀不存在直接返回；
5. 时间复杂度：O(s)，s 为所有字符串的长度之和

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs.length == 0) 
            return "";
        String ans = strs[0];
        for(int i =1;i<strs.length;i++) {
            int j=0;
            for(;j<ans.length() && j < strs[i].length();j++) {
                if(ans.charAt(j) != strs[i].charAt(j))
                    break;
            }
            ans = ans.substring(0, j);
            if(ans.equals(""))
                return ans;
        }
        return ans;
    }
}
```

**使用库函数：**

```java
public String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    String pre = strs[0];
    for (int i = 1; i < strs.length; i++)
        while(strs[i].indexOf(pre) != 0)
            pre = pre.substring(0,pre.length()-1);
    return pre;
}
```



