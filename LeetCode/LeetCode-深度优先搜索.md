#### 岛屿数量-200

给定一个由 '1'（陆地）和 '0'（水）组成的的二维网格，计算岛屿的数量。一个岛被水包围，并且它是通过水平方向或垂直方向上相邻的陆地连接而成的。你可以假设网格的四个边均被水包围。

```
输入:					输入:
11110				 11000
11010 				 11000
11000				 00100
00000				 00011
输出: 1				输出: 3
```

**思路：**采用深度优先搜索，每次访问一个节点，就将他标记，如果是陆地，则检查临近地点。

如果给定坐标（i，j）为一个岛，就从给定的网格中消灭这个岛，使其不再被计算。

```java
public class Solution {

private int n;
private int m;

public int numIslands(char[][] grid) {    
    n = grid.length;
    if (n == 0) return 0;
    m = grid[0].length;
    int count = 0;
    for (int i = 0; i < n; i++){
        for (int j = 0; j < m; j++){
            if (grid[i][j] == '1') {
                DFS(grid, i, j);
                count++;
            }
        }
    }    
    return count;
}

private void DFS(char[][] grid, int i, int j) {
     // Check for invalid indices and for sites that aren't land
    if (i < 0 || j < 0 || i >= n || j >= m || grid[i][j] != '1') return;
    grid[i][j] = '0';	// Mark the site as visited
    // Check all adjacent sites
    DFS(grid, i + 1, j);
    DFS(grid, i - 1, j);
    DFS(grid, i, j + 1);
    DFS(grid, i, j - 1);
}
```

**BFS 感染：**

```java
class Solution {
    public int numIslands(char[][] grid) {
        if(grid==null||grid.length<1||grid[0].length<1)
            return 0;
        int res=0;
        //进行一个一个校验  
        for(int line = 0;line < grid.length; line++){
            for(int col = 0;col < grid[0].length; col++){
                if(grid[line][col] == '1' && bfs(grid,line,col))
                    res++;
            }
        }
        return res;       
    }
    boolean bfs(char[][] grid,int line,int col){
     //发现新的1 会将1周围的所有1全部感染为2,也就是说这片一已经全部为2了
        if(inmap(line,col,grid)&&grid[line][col]=='1'){
                grid[line][col]=2;
                bfs(grid,line+1,col);
                bfs(grid,line-1,col);
                bfs(grid,line,col+1);
                bfs(grid,line,col-1);
                return true;
        }
        return false;       
    }
    boolean inmap(int line,int col,char[][] matrix){
        if(line>matrix.length-1||line<0||col>matrix[0].length-1||col<0)
            return false;
        return true;
    }
}
```



#### 矩阵中的最长递增路径-329

给定一个整数矩阵，找出最长递增路径的长度。

对于每个单元格，你可以往上，下，左，右四个方向移动。 你不能在对角线方向上移动或移动到边界外（即不允许环绕）。

```
输入: nums = 
[
  [9,9,4],
  [6,6,8],
  [2,1,1]
] 
输出: 4 
解释: 最长递增路径为 [1, 2, 6, 9]。
```

```
输入: nums = 
[
  [3,4,5],
  [3,2,6],
  [2,2,1]
] 
输出: 4 
解释: 最长递增路径是 [3, 4, 5, 6]。注意不允许在对角线方向上移动。
```

**思路：**

1. 先深度优先搜索每一个单元格；
2. 将每一个单元格和它旁边的四个方向的内容进行比较，跳过越界的和小的格子；
3. 得到每个格子的最大长度；
4. 使用 matrix[x] [y] <= matrix[i] [j]，就不用使用 visited [m] [n] 了；
5. 把得到的最大值都存储起来，遍历得到全局最大值；

```java
public static final int[][] dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};

public int longestIncreasingPath(int[][] matrix) {
    if(matrix.length == 0) return 0;
    int m = matrix.length, n = matrix[0].length;
    int[][] cache = new int[m][n];
    int max = 1;
    for(int i = 0; i < m; i++) {
        for(int j = 0; j < n; j++) {
            int len = dfs(matrix, i, j, m, n, cache);
            max = Math.max(max, len);
        }
    }   
    return max;
}

public int dfs(int[][] matrix, int i, int j, int m, int n, int[][] cache) {
    if(cache[i][j] != 0) return cache[i][j];
    int max = 1;
    for(int[] dir: dirs) {
        int x = i + dir[0], y = j + dir[1];
        if(x < 0 || x >= m || y < 0 || y >= n || matrix[x][y] <= matrix[i][j]) continue;
        int len = 1 + dfs(matrix, x, y, m, n, cache);
        max = Math.max(max, len);
    }
    cache[i][j] = max;
    return max;
}
```



#### N叉树的最大深度-559

给定一个 N 叉树，找到其最大深度。最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。例如，给定一个 `3叉树` :

![img](../../markdown%E7%AC%94%E8%AE%B0/%E5%9B%BE%E7%89%87/narytreeexample.png)

我们应返回其最大深度，3。

**思路：**和二叉树的最大深度类似，有递归和迭代两种解法；

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public List<Node> children;

    public Node() {}

    public Node(int _val,List<Node> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
    public int maxDepth(Node root) {
        return maxDepth1(root);
    }
    public int maxDepth1(Node root){
        if(root == null) return 0;
        int max = 0;
        for(Node node : root.children){
            int depth = maxDepth1(node);
            max = Math.max(max, depth);
        }
        return max + 1;
    }   
}
```

**迭代：**

```java
class Solution {
    public int maxDepth(Node root) {
        if(root == null) return 0;
        Queue<Node> q = new LinkedList();
        int depth = 0;
        q.add(root);
        while(!q.isEmpty()){
            int size = q.size();
            depth++;
            while(size > 0){
                size--;
                Node cur = q.poll();
                for(Node n : cur.children){
                    if(n != null) q.add(n);   
                }
            }
        }
        return depth;
    }
}
```



#### 被围绕的区域-130

给定一个二维的矩阵，包含 `'X'` 和 `'O'`（**字母 O**）。

找到所有被 `'X'` 围绕的区域，并将这些区域里所有的 `'O'` 用 `'X'` 填充。

解释:

被围绕的区间不会存在于边界上，换句话说，任何边界上的 'O' 都不会被填充为 'X'。 任何不在边界上，或不与边界上的 'O' 相连的 'O' 最终都会被填充为 'X'。如果两个元素在水平或垂直方向相邻，则称它们是“相连”的。

```
输入：
X X X X
X O O X
X X O X
X O X X
输出：
X X X X
X X X X
X X X X
X O X X
```

**思路：**

被围绕的区间不会存在于边界上，换句话说，任何边界上的 'O' 都不会被填充为 'X'。 任何不在边界上，或不与边界上的 'O' 相连的 'O' 最终都会被填充为 'X'。如果两个元素在水平或垂直方向相邻，则称它们是“相连”的。

该题目需要将所有被X包围的O找出来，那剩下的O就是连接起来与边界连通的O。直接找出来所有被X包围的O并不好找，但是我们可以用排除法：先找到所有连接起来与边界连通的O，将这些O标记一下，然后遍历数组，所有没有被标记的O就是我们要找的O。

首先对矩阵边界上所有的O做深度优先搜索，将相连的O更改为-，然后编辑数组，将数组中O更改为X，将数组中-更改为O。

```java
class Solution {
    int row, col;
    public void solve(char[][] board) {
        if(board == null || board.length == 0) return;
        row = board.length;
        col = board[0].length;
        for(int i = 0; i < row; i++){    //对第一列和最后一列的所有O进行深度优先搜索
                dfs(board, i, 0);
                dfs(board, i, col-1);
        }
        for(int j = 0; j < col; j++){    //对第一行和最后一行的所有O进行深度优先搜索
                dfs(board, 0, j);
                dfs(board, row-1, j);
        }
        for(int i = 0; i < row; i++){    //遍历矩阵，将O变为X，将-变为O
            for(int j = 0; j < col; j++){
                if(board[i][j] == 'O') board[i][j]='X';
                if(board[i][j]=='-') board[i][j]='O';
            }
        }
        return;
    }
  /**
  * 使用递归进行深度优先搜索
  */
    public void dfs(char[][] board, int i, int j){
        if(i < 0 || j < 0 || i >= row || j >= col || board[i][j] != 'O') //递归终止条件判断
            return;
        board[i][j] = '-';    //将当前O更改为-
        dfs(board, i-1, j);   //递归该点上方的点
        dfs(board, i+1, j);   //递归该点下方的点
        dfs(board, i, j-1);   //递归该点左边的点
        dfs(board, i, j+1);   //递归该点右边的点
        return;
    }
}
```

