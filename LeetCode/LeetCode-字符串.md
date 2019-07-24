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



#### 最长特殊子序列-521

给定两个字符串，你需要从这两个字符串中找出最长的特殊序列。最长特殊序列定义如下：该序列为某字符串独有的最长子序列（即不能是其他字符串的子序列）。

子序列可以通过删去字符串中的某些字符实现，但不能改变剩余字符的相对顺序。空序列为所有字符串的子序列，任何字符串为其自身的子序列。

输入为两个字符串，输出最长特殊序列的长度。如果不存在，则返回 -1。

```
输入: "aba", "cdc"
输出: 3
解析: 最长特殊序列可为 "aba" (或 "cdc")
```

**思路：**注意题目中的独有两个字，
s1 = 'ab', s2 = 'a'，因为 ab 是 s1 独有，所以最长子序列为ab，
s1 = 'ab', s2 = 'ab'， 因为 ab 是两个串都有，ab排除，a也是两个串都有，排除，b也是两个串都有，排除。所以最长特殊序列不存在，返回-1
通过以上分析，我们可以得出结论，如果：两个串相等（不仅长度相等，内容也相等），那么他们的最长特殊序列不存在。返回-1
如果两个串长度不一样，那么长的串 永远也不可能是 短串的子序列，即 len(s1) > len(s2)，则最长特殊序列为s1，返回长度大的数

```java
class Solution {
    public int findLUSlength(String a, String b) {
        if(a.equals(b))return -1;
        return  a.length() > b.length() ? a.length() : b.length() ;
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



#### 验证回文串-125

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

**说明：**本题中，我们将空字符串定义为有效的回文串。

```
输入: "A man, a plan, a canal: Panama"
输出: true

输入: "race a car"
输出: false
```

**思路：**双指针

```java
class Solution {
    public boolean isPalindrome(String s) {
        char[] c = s.toCharArray();
        int left = 0;
        int right = c.length - 1;
        while (left < right) {
            while (left < right && !Character.isLetterOrDigit(c[left])) left++;
            while (left < right && !Character.isLetterOrDigit(c[right])) right--;
            if (Character.toLowerCase(c[left]) != Character.toLowerCase(c[right])) 
                return false;
            left++;
            right--;
        }
        return true;
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



#### 找到字符串中所有字母异位词-438

给定一个字符串 s 和一个非空字符串 p，找到 s 中所有是 p 的字母异位词的子串，返回这些子串的起始索引。字符串只包含小写英文字母，并且字符串 s 和 p 的长度都不超过 20100。

```
输入:
s: "cbaebabacd" p: "abc"
输出:
[0, 6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的字母异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的字母异位词。
```

```
输入:
s: "abab" p: "ab"
输出:
[0, 1, 2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的字母异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的字母异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的字母异位词。
```

**思路1：**使用暴力循环，比较每个 p 长度的字串和 p 是否是异位词。

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> res = new ArrayList();
        if(s.length() < p.length()) return res;
        int len = p.length();
        for(int i = 0; i < s.length() -len +1; i++){
            String str = s.substring(i, i+len);
            if(match(str, p)) res.add(i);
        }
        return res;
    }
    
    public boolean match(String s1, String s2){
        int[] arr = new int[26];
        for(int i = 0; i < s1.length(); i++){
            arr[s1.charAt(i)-'a']++;
            arr[s2.charAt(i)-'a']--;
        }
        for(int x : arr){
            if(x != 0) return false;
        }
        return true;
    }   
}
```

**思路2：**滑动窗口

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> res = new ArrayList();
        if(s.length() < p.length()) return res;
        int[] map = new int[26];
        for(int i = 0; i < p.length(); i++){
            map[p.charAt(i)-'a']++;
        }
        int start = 0, end = 0;
        while(end < s.length()){
            if(map[s.charAt(end)-'a'] > 0){
                map[s.charAt(end)-'a']--;
                end++;
                if(end - start == p.length())
                    res.add(start);
            }else{
                map[s.charAt(start)-'a']++;
                start++;
            }
        }
        return res;
    }
}
```



#### Z 字形变换-6

将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。比如输入字符串为 `"LEETCODEISHIRING"` 行数为 3 时，排列如下：

```html
L   C   I   R
E T O E S I I G
E   D   H   N

输入: s = "LEETCODEISHIRING", numRows = 3
输出: "LCIRETOESIIGEDHN"

输入: s = "LEETCODEISHIRING", numRows = 4
输出: "LDREOEIIECIHNTSG"
解释:

L     D     R
E   O E   I I
E C   I H   N
T     S     G
```

**思路1：**按行排序

我们可以使用 `min(rowNums, s.length())` 个列表来表示 Z 字形图案中的非空行。从左到右迭代 s，将每个字符添加到合适的行。可以使用当前行和当前方向这两个变量对合适的行进行跟踪。

只有当我们向上移动到最上面的行或向下移动到最下面的行时，当前方向才会发生改变。

- 时间复杂度：O(n)，其中 n = len(*s*)
- 空间复杂度：O(n)

```java
class Solution {
    public String convert(String s, int numRows) {

        if (numRows == 1) return s;

        List<StringBuilder> rows = new ArrayList<>();
        for (int i = 0; i < Math.min(numRows, s.length()); i++)
            rows.add(new StringBuilder());

        int curRow = 0;
        boolean goingDown = false;

        for (char c : s.toCharArray()) {
            rows.get(curRow).append(c);
            if (curRow == 0 || curRow == numRows - 1) goingDown = !goingDown;
            curRow += goingDown ? 1 : -1;
        }

        StringBuilder ret = new StringBuilder();
        for (StringBuilder row : rows) ret.append(row);
        return ret.toString();
    }
}
```

**思路2：**找到规律

2.每一行字母的所有下标其实是有规则的
我们先假定有 numRows=4 行来推导下，其中 2*numRows-2 = 6 , 我们可以假定为 step=2*numRows-2 ，我们先来推导下规则：

第0行： 0 - 6 - 12 - 18

==> 下标间距 `6 - 6 - 6 `==> 下标间距  `step - step - step`

第1行： `1 - 5 - 7 - 11- 13`

==> 下标间距` 4 - 2 - 4 - 2` ==> 下标间距`step-2*1(行)-2*1(行)-step-2*1(行)-2*1(行)`

第2行： `2 - 4 - 8 - 10 - 14`
==> 下标间距` 2 - 4 - 2 - 4` ==> 下标间距`step-2*2(行)-2*2(行)-step-2*2(行)-2*2(行)`

第3行：`3 - 9 - 15 - 21`

==> 下标间距间距 `6 - 6 - 6` ==> 下标间距`step - step - step`

可以得出以下结论：

1. 起始下标都是行号；
2. 第 0 层和第 numRows-1 层的下标间距总是 step ；
3. 中间层的下标间距总是step-2行数，2行数交替；
4. 下标不能超过 len(s) - 1。

```java
class Solution {
    public String convert(String s, int numRows) {

        if (numRows == 1) return s;

        StringBuilder ret = new StringBuilder();
        int n = s.length();
        int cycleLen = 2 * numRows - 2;

        for (int i = 0; i < numRows; i++) {
            for (int j = 0; j + i < n; j += cycleLen) {
                ret.append(s.charAt(j + i));
                if (i != 0 && i != numRows - 1 && j + cycleLen - i < n)
                    ret.append(s.charAt(j + cycleLen - i));
            }
        }
        return ret.toString();
    }
}
```



#### 根据二叉树创建字符串-606

你需要采用前序遍历的方式，将一个二叉树转换成一个由括号和整数组成的字符串。

空节点则用一对空括号 "()" 表示。而且你需要省略所有不影响字符串与原始二叉树之间的一对一映射关系的空括号对。

```
输入: 二叉树: [1,2,3,4]
       1
     /   \
    2     3
   /    
  4     

输出: "1(2(4))(3)"

解释: 原本将是“1(2(4)())(3())”，
在你省略所有不必要的空括号对之后，
它将是“1(2(4))(3)”。

输入: 二叉树: [1,2,3,null,4]
       1
     /   \
    2     3
     \  
      4 

输出: "1(2()(4))(3)"

解释: 和第一个示例相似，
除了我们不能省略第一个对括号来中断输入和输出之间的一对一映射关系。
```

**思路：**递归

```java
class Solution {
    public String tree2str(TreeNode t) {
        if(t == null) return "";
        
        String res = t.val + "";
        
        String left = tree2str(t.left);
        String right = tree2str(t.right);
        
        if(left == "" && right == "") return res;
        if(left == "") return res + "()" + "(" + right + ")";
        if(right == "") return res + "(" + left + ")";
        
        return res + "(" + left + ")" + "(" + right + ")";
    }
}
```

```java
class Solution {
    public String tree2str(TreeNode t) {
        if(t == null) return "";
        StringBuilder sb = new StringBuilder();
        helper(t, sb);
        return sb.toString();
    }
    
    public void helper(TreeNode root, StringBuilder sb){
        sb.append(root.val);
        if(root == null) return;
        if(root.left != null) {
            sb.append("(");
            helper(root.left, sb);
            sb.append(")");
        }
        if(root.right != null){
            if(root.left == null) sb.append("()");
            sb.append("(");
            helper(root.right, sb);
            sb.append(")");
        }
    }
}
```



#### 旋转数字-788

我们称一个数 X 为好数, 如果它的每位数字逐个地被旋转 180 度后，我们仍可以得到一个有效的，且和 X 不同的数。要求每位数字都要被旋转。

如果一个数的每位数字被旋转以后仍然还是一个数字， 则这个数是有效的。0, 1, 和 8 被旋转后仍然是它们自己；2 和 5 可以互相旋转成对方；6 和 9 同理，除了这些以外其他的数字旋转以后都不再是有效的数字。

现在我们有一个正整数 N, 计算从 1 到 N 中有多少个数 X 是好数？

```
示例:
输入: 10
输出: 4
解释: 
在[1, 10]中有四个好数： 2, 5, 6, 9。
注意 1 和 10 不是好数, 因为他们在旋转之后不变。
```

**思路：**判断数的每一位，如果出现 3，4，7 肯定不是好数，0，1，8 要和别的数搭配起来。

```java
class Solution {
    public int rotatedDigits(int N) {
        int res = 0;
        for(int i = 2; i <= N; i++){
            boolean flag = false;
            int num = i;
            while(num != 0){
                int t = num % 10;
                if(t == 2 || t == 5 || t == 6 || t == 9) flag = true;
                if(t == 3 || t == 4 || t == 7) break;
                num /= 10;
            }
            if(num == 0 && flag == true) res++;
        }
        return res;
    }
}
```

**使用位运算：**

```java
class Solution {
    //0 - 9: 0,1,8 必须和其他好数配合，2，5，6，9 是好数，其余一出现就不是好数
    private int[] sMap = {0, 0, 1, -1, -1, 1, 1, -1, 0, 1};

    public int rotatedDigits(int N) {
        int count = 0;
        for (int i = 1; i <= N; i++) {
            if (isGood(i)) {
                count++;
            }
        }
        return count;
    }
    // 0 和任何数或运算都是那个数，-1 和任何数或运算都是 -1
    private boolean isGood(int i) {
        int flag = 0;
        while (i > 0) {
            int mod = i % 10;
            flag |= sMap[mod];
            i /= 10;
        }
        return flag > 0;
    }
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



#### 学生出勤记录-551

给定一个字符串来代表一个学生的出勤记录，这个记录仅包含以下三个字符：

1. 'A' : Absent，缺勤
2. 'L' : Late，迟到
3. 'P' : Present，到场
   如果一个学生的出勤记录中不超过一个'A'(缺勤)并且不超过两个连续的'L'(迟到),那么这个学生会被奖赏。

你需要根据这个学生的出勤记录判断他是否会被奖赏。

```
输入: "PPALLP"
输出: True

输入: "PPALLL"
输出: False
```

**思路：**统计出现 'A' 和连续 'L' 的次数，超过要求就返回 false

```java
class Solution {
    public boolean checkRecord(String s) {
        int a = 0, l = 0;   
        for(int i = 0; i < s.length(); i++){
            if(a > 1 || l > 2) return false;
            char c = s.charAt(i);
            if(c == 'A') a++;
            if(c == 'L') l++;
            else l = 0;		// 下一个字符不是'L'的话就清零
        }
        return a < 2 && l < 3;
    }
}
```

**使用API**

```java
class Solution {
    public boolean checkRecord(String s) {
        return s.indexOf("LLL") < 0 && s.indexOf("A") == s.lastIndexOf("A");
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
        for(int i = 1; i < strs.length; i++) {
            int j = 0;
            for(; j < ans.length() && j < strs[i].length(); j++) {
                if(ans.charAt(j) != strs[i].charAt(j))
                    break;
            }
            ans = ans.substring(0, j);
            if(ans.equals("")) return ans;
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



#### 自定义字符串排序-791

字符串S和 T 只包含小写字符。在S中，所有字符只会出现一次。

S 已经根据某种规则进行了排序。我们要根据S中的字符顺序对T进行排序。更具体地说，如果S中x在y之前出现，那么返回的字符串中x也应出现在y之前。

返回任意一种符合条件的字符串T。

```html
示例:
输入:
S = "cba"
T = "abcd"
输出: "cbad"
解释: 
S中出现了字符 "a", "b", "c", 所以 "a", "b", "c" 的顺序应该是 "c", "b", "a". 
由于 "d" 没有在S中出现, 它可以放在T的任意位置. "dcba", "cdba", "cbda" 都是合法的输出。
```

**思路：**先处理 S 串中不含有的字符，则直接加上，然后处理 S 中有的字符，按照 S 中的顺序遍历加，出现多次的字符则多次添加。

```java
class Solution {
    public String customSortString(String S, String T) {
        StringBuilder sb = new StringBuilder();
        int[] map = new int[S.length()];
        for(char c : T.toCharArray()){
            int i = S.indexOf(c);
            if(i < 0) sb.append(c);
            else map[i]++;
        }
        for(int i = 0; i < S.length(); i++){
            char c = S.charAt(i); 
            while(map[i] != 0){
                sb.append(c);
                map[i]--;
            }
        }
        return sb.toString();
    }
}
```



#### 二进制求和-67

给定两个二进制字符串，返回他们的和（用二进制表示）。输入为**非空**字符串且只包含数字 `1` 和 `0`。

```
输入: a = "11", b = "1"
输出: "100"

输入: a = "1010", b = "1011"
输出: "10101"
```

**思路：**从尾到头遍历相加，用 carry 表示进位，使用 StringBuilder 进行字符串拼接，最后反转。

```java
class Solution{
    public String addBinary(String a, String b){
        int i = a.length()-1, j = b.length()-1;
        StringBuilder sb = new StringBuilder();
        int carry = 0;
        while(i >= 0 || j >= 0){
            if(i >= 0 && s.charAt(i--) == '1') temp++;
            if(j >= 0 && s.charAt(j--) == '1') temp++;
            sb.append(carry % 2);
            carry /= 2;
        }
        if(carry == 1) sb.append(carry);
        return sb.reverse().toString();
    }  
}
```



#### 反转字符串II-541

给定一个字符串和一个整数 k，你需要对从字符串开头算起的每个 2k 个字符的前k个字符进行反转。如果剩余少于 k 个字符，则将剩余的所有全部反转。如果有小于 2k 但大于或等于 k 个字符，则反转前 k 个字符，并将剩余的字符保持原样。

```
输入: s = "abcdefg", k = 2
输出: "bacdfeg"
```

**思路：**按照题目的要求，分部分进行反转。

```java
class Solution {
    public String reverseStr(String s, int k) {
        char[] c = s.toCharArray();
        int len = s.length();
        for(int i = 0; i < len; i += 2*k){
            if(i + 2*k < len || i + k < len){
                reverse(c, i, i+k-1);
            }else{
                reverse(c, i, len-1);
            }
        }
        return new String(c);
    }
    
    public void reverse(char[] c, int i, int j){
        while(i < j){
            char temp = c[i];
            c[i] = c[j];
            c[j] = temp;
            i++;
            j--;
        }
    }
}
```

