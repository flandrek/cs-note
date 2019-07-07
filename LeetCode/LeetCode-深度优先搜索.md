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

