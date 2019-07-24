#### 位1的个数-191

编写一个函数，输入是一个无符号整数，返回其二进制表达式中数字位数为 ‘1’ 的个数（也被称为[汉明重量](https://baike.baidu.com/item/汉明重量)）。

```
输入：00000000000000000000000000001011
输出：3
解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。
```

**思路：**

一个数和它减去1做 & 操作，会去掉最后一个 1；

```java
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight1(int n) {
        int flag = 1;
        int count = 0;
        while(flag != 0){
            if((flag & n) != 0) count++;
            flag = flag << 1;
        }
        return count;
    }
    
    public int hammingWeight(int n) {
        int count = 0;
        while(n != 0){
            count++;
            n = n & (n-1);
        }
        return count;
    }
}
```



#### 只出现一次的数字-136

给定一个**非空**整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

```
输入: [2,2,1]
输出: 1

输入: [4,1,2,1,2]
输出: 4
```

**思路：**相同的数字异或结果为 0，因此可以将数组从头到尾异或一遍，最后剩下的就是只出现一次的数字。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        for(int i = 0; i < nums.length; i++)
            res ^= nums[i];
        return res;
    }
}
```



#### 只出现一次的数字II-137

给定一个**非空**整数数组，除了某个元素只出现一次以外，其余每个元素均出现了三次。找出那个只出现了一次的元素。**说明：**你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

```
输入: [2,2,3,2]
输出: 3

输入: [0,1,0,1,0,1,99]
输出: 99
```

**思路：**将所有数字的二进制表示的对应位都加起来，如果某一位能被三整除，那么只出现一次的数字在该位为0，反之，为1。

除了一个数字出现 M次外,其他数字都出现 K次( 0 < M < K)这个问题,其实此题的解法也是有效的，即

 bitSum % k != 0；

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        int[] bits = new int[32];
        for(int num : nums){
             for(int j = 0; j < 32; j++) {
                bits[j] += ((num >> j) & 1);
            }
        }
        for(int i = 0; i < 32; i++){
            if(bits[i] % 3 != 0)
                res += (1 << i);
        }
        return res;
    }
}
```

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        for(int i = 0; i < 32; i++){
            int bitSum = 0;
            int bit = 1 << i;
            for(int num : nums){
                if((num & bit) != 0)
                    bitSum++;
            }
            if(bitSum % 3 != 0)
                res |= bit;
        }
        return res;
    }
}
```

**这个看不懂！**

```java
class Solution {
    public  static int singleNumber(int[] nums) {
    	 int a = 0, b = 0;
         for (int num : nums){
             a = (a ^ num) & ~b;
             b = (b ^ num) & ~a;
         }
         return a;
     }
}
```



#### 只出现一次的数字III-260

给定一个整数数组 `nums`，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。

```
输入: [1,2,1,3,2,5]
输出: [3,5]
```

　**思路：**我们依旧从头到尾异或每个数字，那么最终的结果就是这两个只出现一次的数字的异或结果，由于两个数不同，因此这个结果数字中一定有一位为1，把结果中第一个1的位置记为第n位。因为是两个只出现一次的数字的异或结果，所以这两个数字在第n位上的数字一定是1和0。

　　 接下来我们根据数组中每个数字的第 n 位上的数字是否为1来进行分组，恰好能将数组分为两个都只有一个数字只出现一次的数组，对两个数组从头到尾异或，就可以得到这两个数了。

```java
class Solution {
    public int[] singleNumber(int[] nums) {
        int res = 0;
        for(int num : nums) res ^= num;
        int index = 0;
        while((res & index) == 0) index >>= 1;
        int[] ans = new int[2];
        for(int num : nums){
            if((num & index) != 0)
                ans[0] ^= num;
            else ans[1] ^= num;
        }
        return ans;
    }
}
```

```java
class Solution{
	public int[] singleNumber(int[] nums) {
        int res = 0;
        for(int num : nums) res ^= num;
        int index = 0;
        while((res & 1) == 0 && index < 32){
            res >>= 1;
            index++;
        }
        int[] ans = {0, 0};
        for(int i = 0; i < nums.length; i++){
            if((nums[i] >> index & 1) == 1)
                ans[0] ^= nums[i];
            else ans[1] ^= nums[i];
        }
        return ans;
    }
}
```





#### 二进制中1个个数-比特位计算

给定一个非负整数 **num**。对于 **0 ≤ i ≤ num** 范围中的每个数字 **i** ，计算其二进制数中的 1 的数目并将它们作为数组返回。

```
输入: 5
输出: [0,1,1,2,1,2]
```

方法一：i & (i - 1)去掉 i 最右边的一个1；因  i & ( i - 1 )< i，故 result [i & (i - 1)] 已计算，所以 i 中 1 的个数为

result [i & (i - 1)] + 1

```java
public int[] countBits2(int num) {
	int[] result = new int[num + 1];
	for (int i = 1; i <= num; ++i) {
		result[i] = result[i & (i - 1)] + 1;
	}
	return result;
}
```

方法二：i >> 1去掉i的最低位；因(i >> 1) < i，故 result [i >> 1]已计算，因此i中1的个数为 i >> 1 中 1 的个数加最后一位1的个数，即为 result [i >> 1] + (i & 1)。

```java
public int[] countBits(int num) {
 	int[] result = new int[num + 1];
 	for (int i = 1; i <= num; ++i) {
 		result[i] = result[i >> 1] + (i & 1);
 	}
 	return result;
}
```



#### 两整数之和-371

**不使用**运算符 `+` 和 `-` ，计算两整数 `a` 、`b` 之和。

```
输入: a = 1, b = 2
输出: 3
输入: a = -2, b = 3
输出: 1
```

**思路：**

1. a + b 的问题拆分为 (a 和 b 的无进位结果) + (a 和 b 的进位结果)；
2. 无进位加法使用异或运算计算得出；
3. 进位结果使用与运算和移位运算计算得出，左移一位；
4. 循环此过程，直到进位为 0；

```java
//迭代
public int getSum(int a, int b) {
	if (a == 0) return b;
	while (b != 0) {
		int carry = a & b; // 进位值
		a = a ^ b;	// 无进位和
		b = carry << 1;
	}
	return a;
}
//递归
public int getSum(int a, int b) {
    if(b == 0) return a;
	return getSum(a ^ b, (a & b) << 1);
}
```



#### 可被5整除的二进制前缀-1018

给定由若干 0 和 1 组成的数组 A。我们定义 N_i：从 A[0] 到 A[i] 的第 i 个子数组被解释为一个二进制数（从最高有效位到最低有效位）。

返回布尔值列表 answer，只有当 N_i 可以被 5 整除时，答案 answer[i] 为 true，否则为 false。

```
输入：[0,1,1]
输出：[true,false,false]
解释：
输入数字为 0, 01, 011；也就是十进制中的 0, 1, 3 。只有第一个数可以被 5 整除，因此 answer[0] 为真。

输入：[0,1,1,1,1,1]
输出：[true,false,false,false,true,false]

输入：[1,1,1,0,1]
输出：[false,false,false,false,false]
```

**思路：**采用十进制对5取模会超出位数，如果上一个数就能被 5 整除，那么再乘以 2 还是能被 5 整除，因此每次判断之后，将这个数对 5 取模。

```java
class Solution {
    public List<Boolean> prefixesDivBy5(int[] A) {
        List<Boolean> res = new ArrayList();
        if(A == null || A.length == 0) return res;
        int sum = 0;
        for(int i = 0; i < A.length; i++){
            sum = sum * 2 + A[i];
            res.add(sum % 5 == 0);
            sum = sum % 5;
        }
        return res;     
    }
}
```

使用位运算优化：

```java
class Solution {
    public List<Boolean> prefixesDivBy5(int[] A) {
        List<Boolean> res = new ArrayList();
        if(A == null || A.length == 0) return res;
        int sum = 0;
        for(int i = 0; i < A.length; i++){
            sum = (sum << 1 | A[i]) % 5;
            res.add(sum == 0);
        }
        return res;     
    }
}
```



#### 缺失数字-268

给定一个包含 `0, 1, 2, ..., n` 中 *n* 个数的序列，找出 0 .. *n* 中没有出现在序列中的那个数。

```
输入: [3,0,1]
输出: 2
输入: [9,6,4,2,3,5,7,0,1]
输出: 8
```

**思路：**

我们知道数组中有 n 个数，并且缺失的数在 [0..n] 中。因此我们可以先得到 [0..n] 的异或值，再将结果对数组中的每一个数进行一次异或运算。未缺失的数在 [0..n] 和数组中各出现一次，因此异或后得到 0。而缺失的数字只在 [0..n] 中出现了一次，在数组中没有出现，因此最终的异或结果即为这个缺失的数字。

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



#### 颠倒二进制位-190

颠倒给定的 32 位无符号整数的二进制位。

```
输入: 00000010100101000001111010011100
输出: 00111001011110000010100101000000
解释: 输入的二进制串 00000010100101000001111010011100 表示无符号整数 43261596，
      因此返回 964176192，其二进制表示形式为 00111001011110000010100101000000。
```

```java
public int reverseBits(int n) {
    if (n == 0) return 0;
    
    int result = 0;
    for (int i = 0; i < 32; i++) {
        result <<= 1;
        if ((n & 1) == 1) result++;
        n >>= 1;
    }
    return result;
}
```

```java
 public int reverseBits1(int n) {
     int result = 0;
     for (int i = 0; i <= 32; i++) {
         // 1. 将给定的二进制数,由低到高位逐个取出
         // 1.1 右移 i 位,
         int tmp = n >> i;
         // 1.2  取有效位
         tmp = tmp & 1;
         // 2. 然后通过位运算将其放置到反转后的位置.
         tmp = tmp << (31 - i);
         // 3. 将上述结果再次通过运算结合到一起
         result |= tmp;
     }
     return result;
 }
```

