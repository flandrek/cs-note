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
        if (numRows <= 0) return triangle;
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

**思路二：**

使用一个临时链表，每次通过链表已有的值，计算下一行的值，并覆盖现有的值

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> res = new LinkedList<>();
        List<Integer> tmp = new ArrayList<>();
        for (int i = 0; i < numRows; i++) {
            tmp.add(0, 1);	// 每一行开头插入 1，这样使得每层的长度 + 1；
            for (int j = 1; j < tmp.size() - 1; j++) {
                tmp.set(j, tmp.get(j) + tmp.get(j + 1));
            }
            res.add(new ArrayList<>(tmp));
        }
        return res;
    }
}
```



#### 杨辉三角-II-119

给定一个非负索引 *k*，其中 *k* ≤ 33，返回杨辉三角的第 *k* 行。你可以优化你的算法到 *O*(*k*) 空间复杂度吗？

![PascalTriangleAnimated2](E:\markdown笔记\图片\PascalTriangleAnimated2.gif)

**思路：**使用一个临时链表，每次通过链表已有的值，计算下一行的值，并覆盖现有的值；

```java
class Solution {
    public List<Integer> getRow(int rowIndex) {
        List<Integer> res = new LinkedList<>();
        for(int i = 0; i <= rowIndex; i++){
            res.add(0, 1);
            for(int j = 1; j < res.size() - 1; j++){
                res.set(j, res.get(j) + res.get(j+1));
            }
        }
        return res;
    }
}
```



#### 螺旋矩阵-54

给定一个包含 *m* x *n* 个元素的矩阵（*m* 行, *n* 列），请按照顺时针螺旋顺序，返回矩阵中的所有元素。

```
输入:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
输出: [1,2,3,6,9,8,7,4,5]
```

**思路：**模拟过程

```java
import java.util.*;
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> res = new ArrayList<>();
        if(matrix == null || matrix.length == 0) return res;
        int rows = matrix.length, cols = matrix[0].length;
        int up_row = 0, down_row = rows - 1, left_col = 0, right_col = cols - 1;
        while(up_row <= down_row && left_col <= right_col){
            for(int i = left_col; i <= right_col; i++) res.add(matrix[up_row][i]);
            up_row++;
            if(up_row > down_row) break;
            for(int i = up_row; i <= down_row; i++) res.add(matrix[i][right_col]);
            right_col--;
            if(right_col < left_col) break;
            for(int i = right_col; i >= left_col; i--) res.add(matrix[down_row][i]);
            down_row--;
            for(int i = down_row; i >= up_row; i--) res.add(matrix[i][left_col]);
            left_col++;
        }
        return res;
    }  
    // public List<Integer> spiralOrder(int[][] matrix) {
    //     if (matrix == null || matrix.length == 0) return new ArrayList<>();
    //     List<Integer> res = new ArrayList<>();
    //     int shang_row = 0;
    //     int xia_row = matrix.length - 1;
    //     int zou_col = 0;
    //     int you_col = matrix[0].length - 1;
    //     while (shang_row <= xia_row && zou_col <= you_col) {
    //         // 从左到右
    //         for (int i = zou_col; i <= you_col; i++) res.add(matrix[shang_row][i]);
    //         shang_row++;
    //         if (shang_row > xia_row) break;
    //         // 从上到下
    //         for (int i = shang_row; i <= xia_row; i++) res.add(matrix[i][you_col]);
    //         you_col--;
    //         if (zou_col > you_col) break;
    //         // 从右到左
    //         for (int i = you_col; i >= zou_col; i--) res.add(matrix[xia_row][i]);
    //         xia_row--;
    //         //从下到上
    //         for (int i = xia_row; i >= shang_row; i--) res.add(matrix[i][zou_col]);
    //         zou_col++;
    //     }
    //     return res;
    // }
}
```

