#### 阶乘后的零-172

给定一个整数 *n*，返回 *n*! 结果尾数中零的数量。

```
输入: 3
输出: 0
解释: 3! = 6, 尾数中没有零。

输入: 5
输出: 1
解释: 5! = 120, 尾数中有 1 个零.
```

**思路：**

不断除以 5，是因为每间隔 5 个数有一个数可以被 5 整除， 然后在这些可被 5 整除的数中， 每间隔 5 个数又有一个可以被 25 整除，故要再除一次， 直到结果为 0，表示没有能继续被 5 整除的数了。

其实末尾有多少个 0，取决于到该阶乘的最大直为止，一共有多少"5的整数次幂"； 比如对于50，有50/5 =10个 5的一次方；有 50 / 25 = 2 个 5 的二次方，所以总计有 10 + 2 = 12 个0。

```java
class Solution {
    public int trailingZeroes(int n) {
        int count = 0;
        while(n >= 5) {
            count += n / 5;
            n /= 5;
        }
        return count;
    }
}
```



#### 计算质数-204

统计所有小于非负整数 *n* 的质数的数量。

```
输入: 10
输出: 4
解释: 小于 10 的质数一共有 4 个, 它们是 2, 3, 5, 7 。
```

**思路：**如求10之内的质数，首先列出2~N-1的所有数，如果当前数为质数，则其倍数就是质数，如

第一个质数为2，在2上画圈，其倍数4/6/8不是质数，划掉4/6/8，继续遍历
下一个质数为3，在3上画圈，其倍数6/9不是质数，划掉6/9，继续遍历
下一个质数为5，在5上画圈，没有倍数，继续遍历
下一个质数为7，在7上画圈，没有倍数，继续遍历。
最后再次遍历整个数组，画圈的数字就是质数，即2,3,5,7

转换为代码就是如果需要求 < n 的所有质数个数，则创建一个长度为n的整数数组，所有元素值变为 false，false表示对应的索引值为质数，true 表示对应的索引值为非质数。从2开始遍历，如果当前值为 false，则获取其所有倍数，将元素值变为 true（标记为非质数）。

![img](E:\markdown笔记\图片\Sieve_of_Eratosthenes_animation.gif)

```java
public class Solution {
    public int countPrimes(int n) {
        boolean[] notPrime = new boolean[n];
        int count = 0;
        for (int i = 2; i < n; i++) {
            if (notPrime[i] == false) {
                count++;
                for (int j = 2; i*j < n; j++) {
                    notPrime[i*j] = true;
                }
            }
        }        
        return count;
    }
}
```

```java
class Solution {
    public int countPrimes(int n) {
        boolean[] noPrime = new boolean[n];
        int count = 0;
        for(int i = 2; i < n; i++){
            if(!noPrime[i]){
                for(int j = 2 * i; j < n; j += i){
                    noPrime[j] = true; 
                }
            }
        }
        for(int i = 2; i < n; i++){
            if(!noPrime[i]) count++;
        }
        return count;
    }
}
```



