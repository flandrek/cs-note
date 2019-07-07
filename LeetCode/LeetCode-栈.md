#### trapping-rain-water

给定n个非负整数，表示每个条的宽度为1的高程图，计算雨后它能捕获多少水。

![rainwatertrap](E:\markdown笔记\图片\rainwatertrap.png)

上面的高程图由数组[0,1,0,2,1,1,1,3,2,1,2,1,1]表示。在这种情况下，6个单位的雨水(蓝色部分)被捕获。

**双指针法：**

```java
public class Solution {
    public int trap(int[] A) {
        if(A == null || A.length == 0){
            return 0;
        }
        int left = 0;
        int right = 0;
        int sum = 0;
        int maxIndex = 0;
        //找到下标的最大值
        for(int i = 1; i < A.length; i++){
            if(A[i] > A[maxIndex]){
                maxIndex = i;
            }
        }
        //计算左边的存水量
        for(int i = 0; i < maxIndex; i++){
            if(A[i] < left){
                sum += (left - A[i]);
            }else{
                left = A[i];
            }
        }
        //计算右边的存水量
        for(int j = A.length -1; j > maxIndex; j--){
            if(A[j] < right){
                sum += (right - A[j]);
            }else{
                right = A[j];
            }
        }
        return sum;
    }
}
```

**栈：**

```java
class Solution {
    public int trap(int[] height) {
        if (height == null || height.length == 0) return 0;
        Deque<Integer> stack = new ArrayDeque<>();
        int res = 0;
        for (int i = 0; i < height.length; i++){
            while ( ! stack.isEmpty() && height[stack.peek()] < height[i]) {
                int tmp = stack.pop();
                if (stack.isEmpty()) break;
                res += (Math.min(height[i],height[stack.peek()]) - height[tmp]) * (i - 								stack.peek() - 1);
            }
            stack.push(i);
        }
        return res;
    }
}
```



#### 每日温度-739

根据每日 气温 列表，请重新生成一个列表，对应位置的输入是你需要再等待多久温度才会升高的天数。如果之后都不会升高，请输入 0 来代替。

```
例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。提示：气温 列表长度的范围是 [1, 30000]。每个气温的值的都是 [30, 100] 范围内的整数。
```

以某个元素为出发点，第一想法是将后续所有元素入栈，再遍历栈顶与当前元素比较。
遍历过程中我们要找的是首个更大值的下标，某个大值后的较小值对我们毫无意义，剔除降序片段，所以我们的目标是 维护一个从栈顶至栈底递增的栈 。根据栈后进先出的特性，要从后往前遍历数组。目的是获取下标差值，所以栈保存下标而不是值。

1. 如果当前元素大于等于栈顶元素，则重复pop，直到栈顶元素大于当前元素，二者下标差值即为所求。
2.  如果栈为空，说明栈中没有大于当前元素的值，保存0。
3. 将当前元素（新的最小值）入栈。

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        int[] result = new int[T.length];
        Stack<Integer> stack = new Stack<>();
        for (int i = T.length - 1; i >= 0; i--) {
            // 循环pop，直到栈顶大于当前元素
            while (!stack.isEmpty() && T[i] >= T[stack.peek()]) stack.pop();
            result[i] = stack.isEmpty() ? 0 : stack.peek() - i;
            stack.push(i);
        }
        return result;
    }
}
```



#### 字符串解码-394

给定一个经过编码的字符串，返回它解码后的字符串。

编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k  保证为正整数。

你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。

此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 3a 或 2[4] 的输入。

```
s = "3[a]2[bc]", 返回 "aaabcbc".
s = "3[a2[c]]", 返回 "accaccacc".
s = "2[abc]3[cd]ef", 返回 "abcabccdcdcdef".
```

![1560907598417](E:\markdown笔记\图片\1560907598417.png)

```java
public class Solution {
    public String decodeString(String s) {
        String res = "";
        Stack<Integer> countStack = new Stack<>();
        Stack<String> resStack = new Stack<>();
        int idx = 0;
        while (idx < s.length()) {
            if (Character.isDigit(s.charAt(idx))) {
                int count = 0;
                while (Character.isDigit(s.charAt(idx))) {
                    count = 10 * count + (s.charAt(idx) - '0');
                    idx++;
                }
                countStack.push(count);
            }
            else if (s.charAt(idx) == '[') {
                resStack.push(res);
                res = "";
                idx++;
            }
            else if (s.charAt(idx) == ']') {
                StringBuilder temp = new StringBuilder (resStack.pop());
                int repeatTimes = countStack.pop();
                for (int i = 0; i < repeatTimes; i++) {
                    temp.append(res);
                }
                res = temp.toString();
                idx++;
            }
            else {
                res += s.charAt(idx++);
            }
        }
        return res;
    }
}
```

简洁写法：

```java
public class Solution {
    public String decodeString(String s) {
        Stack<Integer> intStack = new Stack<>();
        Stack<StringBuilder> strStack = new Stack<>();
        StringBuilder cur = new StringBuilder();
        int k = 0;
        for (char ch : s.toCharArray()) {
            if (Character.isDigit(ch)) {
                k = k * 10 + ch - '0';
            } else if ( ch == '[') {
                intStack.push(k);
                strStack.push(cur);
                cur = new StringBuilder();
                k = 0;
            } else if (ch == ']') {
                StringBuilder tmp = cur;
                cur = strStack.pop();
                for (k = intStack.pop(); k > 0; --k) cur.append(tmp);
            } else cur.append(ch);
        }
        return cur.toString();
    }
}
```



#### 有效的括号-20

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。
3. 注意空字符串可被认为是有效字符串。

```
输入: "()[]{}"
输出: true
输入: "([)]"
输出: false
```

```java
class Solution {
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<>();
        char[] chars = s.toCharArray();
        for (char aChar : chars) {
            if (stack.size() == 0) {
                stack.push(aChar);
            } else if (isSym(stack.peek(), aChar)) {
                stack.pop();
            } else {
                stack.push(aChar);
            }
        }
        return stack.size() == 0;
    }
    
    private boolean isSym(char c1, char c2) {
        return (c1 == '(' && c2 == ')') || (c1 == '[' && c2 == ']') 
            || (c1 == '{' && c2 == '}');
    }
}
```

匹配问题，我们一般使用 **栈**

遍历字符串，我们把左括号压入栈中，当遇到右括号，和栈顶元素比较！

时间复杂度：O(n)；空间复杂度：O(n)

```java
class Solution {
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<Character>();
        for (char alp : s.toCharArray()) {
            if (alp == '(') stack.push(')');
            else if (alp == '[') stack.push(']');
            else if (alp == '{') stack.push('}');
            else if (stack.isEmpty() || stack.pop() != alp) return false;
        }
        return stack.isEmpty();      
    }
}
```

