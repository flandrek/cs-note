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

