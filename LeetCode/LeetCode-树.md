#### **和为指定值的路径**

判断给定二叉树中是否含有和为sum的路径，路径为根节点到叶子节点。

``` java
public class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        if(root == null)
            return false;
        sum -= root.val;
        if(root.left == null && root.right == null && sum == 0)
            return true;
        return hasPathSum(root.left, sum) || hasPathSum(root.right, sum);
    }
}
```

判断给定二叉树中是否含有和为sum的路径，路径为根节点到叶子节点，并将路径输出。

```java
class Solution{
	 public void findPath1(TreeNode root,int target) {
         if(root == null)
             return;
         ArrayList<Integer> list= new ArrayList<>();
         printPath(root, target, list);
     }
	     
     private void printPath(TreeNode node,int target,ArrayList<Integer> list) {
        if(node == null)
            return;
        list.add(node.val);
        target -= node.val;  //每个结点的target不会受到方法的影响而改变
        if(target == 0 && node.left == null && node.right == null) {  //叶子结点
            for (Integer integer : list)
                System.out.print(integer+" ");
            System.out.println();
        }else {     //中间结点
            printPath(node.left, target, list);
            printPath(node.right, target, list);
        }
        list.remove(list.size() -1);	//回溯
    }
}
```



#### **对称二叉树**

判断一个二叉树是不是对称二叉树。

递归：时间复杂度和空间复杂度都为 O ( n )

```java
class Solution{
    public boolean isSymmetric1(TreeNode root) {
        if(root == null)
            return true;
        return Core(root, root);
    }
    private boolean Core(TreeNode root1, TreeNode root2){
        if(root1 == null && root2 == null)
            return true;
        if(root1 == null || root2 == null)
            return false;
        return root1.val == root2.val && Core(root1.left, root2.right) && 		      					Core(root1.right, root2.left);
    }
}
```

非递归：

```java
class Solution{
	public boolean isSymmetric(TreeNode root) {
        Queue<TreeNode> q = new LinkedList<>();
        q.add(root);
        q.add(root);
        while (!q.isEmpty()) {
            TreeNode t1 = q.poll();
            TreeNode t2 = q.poll();
            if (t1 == null && t2 == null) continue;
            if (t1 == null || t2 == null) return false;
            if (t1.val != t2.val) return false;
            q.add(t1.left);
            q.add(t2.right);
            q.add(t1.right);
            q.add(t2.left);
        }
        return true;
    }
}
```



#### 合并二叉树-617

给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则不为 NULL 的节点将直接作为新二叉树的节点。

```
输入: 
	Tree 1                     Tree 2                  
          1                         2                             
         / \                       / \                            
        3   2                     1   3                        
       /                           \   \                      
      5                             4   7                  
输出: 
合并后的树:
	     3
	    / \
	   4   5
	  / \   \ 
	 5   4   7
```

```java
class Solution {
    public TreeNode mergeTrees_1(TreeNode t1, TreeNode t2) {
        if (t1 == null) return t2;
        if (t2 == null) return t1;       
        // 先合并根节点
        t1.val += t2.val;
        // 再递归合并左右子树
        t1.left = mergeTrees(t1.left, t2.left);
        t1.right = mergeTrees(t1.right, t2.right);
        return t1;
    }
    /**
     * 不修改原二叉树的解法
     */
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if (t1 == null && t2 == null) return null;
        // 先合并根节点
        TreeNode root = new TreeNode((t1 == null ? 0 : t1.val) +
                                     (t2 == null ? 0 : 	t2.val));
        // 再递归合并左右子树
        root.left = mergeTrees(t1 == null ? null : t1.left, t2 == 
                               null ? null : t2.left);
        root.right = mergeTrees(t1 == null ? null : t1.right, t2 == 
                                null ? null : t2.right);
        return root;
    }
}
```



#### 翻转二叉树-226

将二叉树进行镜像翻转。

**递归：**

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root == null || (root.left == null && root.right == null)) return root;
        TreeNode temp = root.left;
        root.left = root.right;
        root.right = temp;
        invertTree(root.left);
        invertTree(root.right);
        return root;
    }
    
    public TreeNode invertTree(TreeNode root) {
    	if (root == null)  return null;
    	TreeNode right = invertTree(root.right);
    	TreeNode left = invertTree(root.left);
    	root.left = right;
    	root.right = left;
    	return root;
 	}
}
```

**迭代：**

```java
public TreeNode invertTree(TreeNode root) {
    if (root == null) return null;
    // LinkedList实现了集合框架的Queue接口
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root); // 加入元素
    while (!queue.isEmpty()) {
      	// 获取并移出元素
      	TreeNode current = queue.poll();
      	//交换左右子树
      	TreeNode temp = current.left;
      	current.left = current.right;
      	current.right = temp;
      	//左子树不为空，将左子树入栈
      	if (current.left != null) queue.add(current.left);
     	//右子树不为空，将右子树入栈
     	if (current.right != null) queue.add(current.right);
    }
    return root;
}
```



#### 二叉树展开为链表-114

给定一个二叉树，**原地**将它展开为链表。

```html
    1					
   / \
  2   5
 / \   \
3   4   6
展开为：
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```



#### 重建二叉树-105

根据一棵树的前序遍历与中序遍历构造二叉树。

```
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
```

![1de4b6b0a5754a82d28475586296bcb5ea6a3b93d61a5bb01485b516ad5dc819-image](E:\markdown笔记\图片\1de4b6b0a5754a82d28475586296bcb5ea6a3b93d61a5bb01485b516ad5dc819-image.png)

```java
public class Solution {
    // 时间复杂度和空间复杂度都为 O（n）
    // 使用前序遍历和中序遍历构建二叉树
    // 举出具体例子来分析就会很清晰，这里一定要分析清楚索引的起始
    // 例如：
    // pre: A B D E H I C F K G
    // in : D B H E I A F K C G
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        int preLen = preorder.length;
        int inLen = inorder.length;
        return helper(preorder, 0, preLen - 1, inorder, 0, inLen - 1);
    }

    private TreeNode helper(int[] preorder,int preL, int preR,int[] inorder,
                            int inL, int inR) {
        if (preL > preR || inL > inR) {
            return null;
        }
        int rootVal = preorder[preL];
        int l = inL;
        while (l <= inR && inorder[l] != rootVal) {
            l++;
        }
        TreeNode root = new TreeNode(rootVal);
        root.left = helper(preorder, preL + 1, preL + l - inL, inorder, inL, l - 1);
        root.right = helper(preorder, preL + l - inL + 1, preR, inorder, l + 1, inR);
        return root;
    }
}
```

剑指 offer上的做法：

```java
public static BinaryTreeNode reconstructe(int[] preOrder,int[] inOrder) {
		
    if(preOrder == null || inOrder == null || preOrder.length == 0 
       || inOrder.length == 0)
        return null;		
    //二叉树的根节点
    BinaryTreeNode root = new BinaryTreeNode(preOrder[0]);	
    //左子树节点的个数
    int leftNum = 0;
    for(int i = 0; i < inOrder.length; i++) {
        if(root.value == inOrder[i])
            break;
        else  leftNum++;
    }
    //右子树节点的个数
    int rightNum = inOrder.length - leftNum - 1;		
    //递归重建左子树
    if(leftNum > 0) {
        int[] leftPreOrder = new int[leftNum];
        int[] leftInOrder = new int[leftNum];
        for(int i = 0; i < leftNum; i++) {
            leftPreOrder[i] = preOrder[i + 1];
            leftInOrder[i] = inOrder[i];
        }
        BinaryTreeNode leftRoot = reconstructe(leftPreOrder, leftInOrder);
        root.left = leftRoot;
    }
    //递归重建右子树
    if (rightNum > 0) {

        int[] rightPreOrder = new int[rightNum];
        int[] rightInOrder = new int[rightNum];
        for (int i = 0; i < rightNum; i++) {
            rightPreOrder[i] = preOrder[leftNum + 1 + i];
            rightInOrder[i] = inOrder[leftNum + 1 + i];
        }
        BinaryTreeNode rightRoot = reconstructe(rightPreOrder, rightInOrder);
        root.right = rightRoot;
    }
    return root;
}
```



#### 二叉树的层序遍历-102

给定一个二叉树，返回其按层次遍历的节点值，即逐层地，从左到右访问所有节点。

**递归：**

时间复杂度和空间复杂度都为 O (N) 

最简单的解法就是递归，首先确认树非空，然后调用递归函数 helper(node, level)，参数是当前节点和节点的层次。程序过程如下：

1. 输出列表称为 levels，当前最高层数就是列表的长度 len(levels)，比较访问节点所在的层次 level 和当前最高层次 len(levels) 的大小，如果前者更大就向 levels 添加一个空列表。
2. 将当前节点插入到对应层的列表 levels[level] 中。
3. 递归非空的孩子节点：helper(node.left / node.right, level + 1)。

```java
class Solution {
    List<List<Integer>> levels = new ArrayList<List<Integer>>();
    public void helper(TreeNode node, int level) {
        // start the current level
        if (levels.size() == level)
            levels.add(new ArrayList<Integer>());

         // fulfil the current level
         levels.get(level).add(node.val);

         // process child nodes for the next level
         if (node.left != null)
            helper(node.left, level + 1);
         if (node.right != null)
            helper(node.right, level + 1);
    }    
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) return levels;
        helper(root, 0);
        return levels;
    }
}
```

**非递归**

时间复杂度和空间复杂度都为 O (N) 

```java
import java.util.*;
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if(root == null) return res;
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        int level = 0;
        while(!q.isEmpty()){
            res.add(new ArrayList<Integer>());
            int size = q.size();
            for(int i = 0; i < size; i++){
                TreeNode node = q.poll();
                res.get(level).add(node.val);
                if(node.left != null) q.offer(node.left);
                if(node.right != null) q.offer(node.right);
            }
            level++;
        }
        return res;
    }
}
```



#### 二叉树的最低公共祖先-236

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

![binarytree](E:\markdown笔记\图片\binarytree.png)

```
输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出: 3
解释: 节点 5 和节点 1 的最近公共祖先是节点 3。
输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
输出: 5
解释: 节点 5 和节点 4 的最近公共祖先是节点 5。因为根据定义最近公共祖先节点可以为节点本身。
```

 /**
        注意p,q必然存在树内, 且所有节点的值唯一!!!
        递归思想, 对以root为根的(子)树进行查找p和q, 如果root == null || p || q 直接返回root
        表示对于当前树的查找已经完毕, 否则对左右子树进行查找, 根据左右子树的返回值判断:

           1. 左右子树的返回值都不为null, 由于值唯一左右子树的返回值就是p和q, 此时root为LCA；
           2. 果左右子树返回值只有一个不为null, 说明只有p或q存在于左或右子树中, 最先找到的那个节点为LCA；
           3. 左右子树返回值均为null, p和q均不在树中, 返回null；

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if(left == null && right == null) return null;
        else if(left != null && right != null) return root;
        else return left == null ? right : left;
    }
}
```



#### 将二叉树转换为累加树-538

给定一个二叉搜索树（Binary Search Tree），把它转换成为累加树（Greater Tree)，使得每个节点的值是原来的节点值加上所有大于它的节点值之和。

思路：以 右 -> 根 -> 左 的顺序遍历二叉树，将遍历顺序的前一个结点的累加值记录起来，和当前结点相加，得到当前结点的累加值。

```
输入: 二叉搜索树:
              5
            /   \
           2     13
输出: 转换为累加树:
             18
            /   \
          20     13
```

递归：

```java
class Solution {
    private int preNum = 0;
    public TreeNode convertBST(TreeNode root) {
        unPreorder(root);
        return root;
    }
    public void unPreorder(TreeNode root){
        if(root == null) return;
        unPreorder(root.right);
        root.val += preNum;
        preNum = root.val;
        unPreorder(root.left);
    }
}
```

非递归：

```java
private int preNum = 0;
public TreeNode convertBST(TreeNode root) {
     if(root == null) return null;
     Stack<TreeNode> stack = new Stack<>();
     TreeNode node = root;
     while(!stack.isEmpty() || node != null){
         while(node != null){
             stack.push(node);
             node = node.right;
         }
         node = stack.pop();
         node.val += preNum;
         preNum = node.val;
         node = node.left;
     }
     return root;
}
```



#### 二叉树的直径-543

给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过根结点。

```
    	  1
         / \
        2   3
       / \     
      4   5 		返回 3, 它的长度是路径 [4,2,1,3] 或者 [5,2,1,3]。
```

二叉树的直径为 -> 左子树的最大深度 + 右子树的最大深度。res 作为类内部的全局变量，时刻保留当前最大的二叉树直径。

```java
class Solution {  
    //全局变量保留最大的值
    private int res =0;
    public int diameterOfBinaryTree(TreeNode root) {
        maxDiameter(root);
        return res;
    }
    public int maxDiameter(TreeNode root) {
        if (root == null) return 0;
      	//递归左右子树
        int r = maxDiameter(root.right);
        int l = maxDiameter(root.left);
        //更新二叉树直径
        res = Math.max(res,l+r);
        //返回的是该节点的层数
        return Math.max(l,r) + 1;
    }
}
```



#### 验证二叉搜索树-98

给定一个二叉树，判断其是否是一个有效的二叉搜索树，假设一个二叉搜索树具有如下特征：

1. 节点的左子树只包含小于当前节点的数。
2. 节点的右子树只包含大于当前节点的数。
3. 所有左子树和右子树自身必须也是二叉搜索树。

```
输入:
    	5
   	   / \
  	  1   4
    	 / \
    	3   6
输出: false
解释: 输入为: [5,1,4,null,null,3,6]。
     根节点的值为 5 ，但是其右子节点值为 4 。
```

主要利用 二叉搜索树 中序遍历的有序性，只需在遍历的过程中判断当前节点是否大于前一个节点即可。

**递归：**

```java
class Solution{
	long pre = Long.MIN_VALUE;
    public boolean isValidBST(TreeNode root) {
        if(root == null) return true;       
        if(isValidBST(root.left)) {
            if(pre < root.val) {
                pre = root.val;
                return isValidBST(root.right);
            }
        }
        return false;
    }
}
```

**非递归：**

```java
class Solution {
  	public boolean isValidBST(TreeNode root) {
    	Stack<TreeNode> stack = new Stack();
    	double inorder = - Double.MAX_VALUE;
    	while (!stack.isEmpty() || root != null) {
         	 while (root != null) {
        		stack.push(root);
        		root = root.left;
      		 }
      	root = stack.pop();
      	if (root.val <= inorder) return false;
      		inorder = root.val;
      		root = root.right;
    	}
    	return true;
  	}
}
```



#### 二叉树展开为链表-114

给定一个二叉树，[原地](https://baike.baidu.com/item/原地算法/8010757)将它展开为链表。

```
    1					1		
   / \					 \ 
  2   5					  2
 / \   \				   \
3   4   6					3
							 \
							  4
  							   \
  								5
								 \ 
								  6                               
```

**递归：**

```java
class Solution {
    TreeNode pre = null;
    public void flatten(TreeNode root) {
        if(root == null) return;
        flatten(root.right);
        flatten(root.left);
        root.right = pre;
        root.left = null;
        pre = root;        
    }
}
```

**非递归：**

```java
class Solution{
	public void flatten(TreeNode root) {
        if (root == null) return;
        Stack<TreeNode> stk = new Stack<TreeNode>();
        stk.push(root);
        while (!stk.isEmpty()){
            TreeNode curr = stk.pop();
            if (curr.right!=null)  
                 stk.push(curr.right);
            if (curr.left!=null)  
                 stk.push(curr.left);
            if (!stk.isEmpty()) 
                 curr.right = stk.peek();
            curr.left = null;  // dont forget this!! 
        }
    }
}
```



#### 路径总合-III-437

给定一个二叉树，它的每个结点都存放着一个整数值，找出路径和等于给定数值的路径总数。

路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。

```html
root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8

      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1

返回 3。和等于 8 的路径有:

1.  5 -> 3
2.  5 -> 2 -> 1
3.  -3 -> 11
```

**递归实现：**

```java
class Sloution{
/* 有了以上铺垫，详细注释一下代码 */
    int pathSum(TreeNode root, int sum) {
        if (root == null) return 0;
        int pathImLeading = count(root, sum); // 自己为开头的路径数
        int leftPathSum = pathSum(root.left, sum); // 左边路径总数（相信他能算出来）
        int rightPathSum = pathSum(root.right, sum); // 右边路径总数（相信他能算出来）
        return leftPathSum + rightPathSum + pathImLeading;
    }
    int count(TreeNode node, int sum) {
        if (node == null) return 0;
        // 我自己能不能独当一面，作为一条单独的路径呢？
        int count = (node.val == sum) ? 1 : 0;
        // 左边的小老弟，你那边能凑几个 sum - node.val 呀？
        int leftBrother = count(node.left, sum - node.val); 
        // 右边的小老弟，你那边能凑几个 sum - node.val 呀？
        int rightBrother = count(node.right, sum - node.val);
        return  count + leftBrother + rightBrother; // 我这能凑这么多个
    }
}
```

**回溯法：**

第一种做法很明显效率不够高，存在大量重复计算；
所以第二种做法，采取了类似于数组的前 n 项和的思路，比如sum[4] == sum[1]，那么1到4之间的和肯定为 0

对于树的话，采取DFS加回溯，每次访问到一个节点，把该节点加入到当前的pathSum中，然后判断是否存在一个之前的前 n 项和，其值等于 pathSum 与 sum 之差；
如果有，就说明现在的前n项和，减去之前的前n项和，等于sum，那么也就是说，这两个点之间的路径和，就是sum，最后要注意的是，记得回溯，把路径和弹出去。

```java
class Solution{
	public int pathSum(TreeNode root, int sum) {
        if (root == null) return 0;
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
        return findPathSum(root, 0, sum, map);  
    }
    
    private int findPathSum(TreeNode curr, int sum, int target,
                            Map<Integer, Integer> map) {
        if (curr == null) return 0;
        // 通过添加当前 val 来更新前缀和
        sum += curr.val;
        // 获取由当前节点结束的有效路径数
        int numPathToCurr = map.getOrDefault(sum - target, 0); 
        // 用当前和更新映射，以便将映射传递到下一个递归;
        map.put(sum, map.getOrDefault(sum, 0) + 1);
        // 将当前数量 + 左子树 + 右子树
        int res = numPathToCurr + findPathSum(curr.left, sum, target, map)
                                      + findPathSum(curr.right, sum, target, map);
       	// 还原映射-回溯
        map.put(sum, map.get(sum) - 1);
        return res;
    }
}
```



#### 将有序数组转换为二叉搜索树-108

将一个按照升序排列的有序数组，转换为一棵高度平衡二叉搜索树。本题中，一个高度平衡二叉树是指一个二叉树*每个节点* 的左右两个子树的高度差的绝对值不超过 1。

```
给定有序数组: [-10,-3,0,5,9],

一个可能的答案是：[0,-3,9,-10,null,5]，它可以表示下面这个高度平衡二叉搜索树：

      0
     / \
   -3   9
   /   /
 -10  5
```

**思路：**

用二分法获得数组的中间节点作为根节点，树肯定是平衡的。

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        // 左右等分建立左右子树，中间节点作为子树根节点，递归该过程
        return nums == null ? null : buildTree(nums, 0, nums.length - 1);
    }

    private TreeNode buildTree(int[] nums, int l, int r) {
        if (l > r)  return null;
        int mid = l + (r - l) / 2;
        TreeNode root = new TreeNode(nums[mid]);
        root.left = buildTree(nums, l, mid - 1);
        root.right = buildTree(nums, mid + 1, r);
        return root;
    }
}
```



#### 二叉搜索树中第K小的元素-230

给定一个二叉搜索树，编写一个函数 `kthSmallest` 来查找其中第 **k** 个最小的元素。

```
输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 3
```

**思路：**要想找到第 K 小的元素，需要中序遍历，结果刚好是从小到大排列。

**递归：**

```java
class Solution {
	int count = 0;
    int result = Integer.MIN_VALUE;
    public int kthSmallest(TreeNode root, int k) {
        traverse(root, k);
        return result;
    }
    public void traverse(TreeNode root, int k) {
        if(root == null) return;
        traverse(root.left, k);
        count++;
        if(count == k) result = root.val;
        traverse(root.right, k);       
    }
}
```

**迭代：**

```java
public int kthSmallest(TreeNode root, int k) {
     Stack<TreeNode> stack = new Stack<TreeNode>();
     TreeNode p = root;
     int count = 0;     
     while(!stack.isEmpty() || p != null) {
         if(p != null) {
             stack.push(p);  // Just like recursion
             p = p.left;               
         } else {
            TreeNode node = stack.pop();
            if(++count == k) return node.val; 
            p = node.right;
         }
     }     
     return Integer.MIN_VALUE;
}
```



#### 二叉树的锯齿形层次遍历-103

给定一个二叉树，返回其节点值的锯齿形层次遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

```
例如：
给定二叉树 [3,9,20,null,null,15,7]
    3
   / \
  9  20
    /  \
   15   7
```

**使用队列：**

**迭代：**是偶数层用尾插，奇数层用头插，这样出队列的时候刚好是相反的；

```java
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) return res;
        Deque<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        int depth = 0;
        while (!queue.isEmpty()) {
            List<Integer> tmp = new LinkedList<>();
            int cnt = queue.size();
            for (int i = 0; i < cnt; i++) {
                TreeNode node = queue.poll();
                // System.out.println(node.val);
                if (depth % 2 == 0) tmp.add(node.val);
                else tmp.add(0, node.val);
                if (node.left != null) queue.add(node.left);
                if (node.right != null) queue.add(node.right);
            }
            res.add(tmp);
            depth++;
        }
        return res;
    }
}
```

**递归：**

```java
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        helper(res, root, 0);
        return res;

    }

    private void helper(List<List<Integer>> res, TreeNode root, int depth) {
        if (root == null) return;
        if (res.size() == depth) res.add(new LinkedList<>());
        if (depth % 2 == 0) res.get(depth).add(root.val);
        else res.get(depth).add(0, root.val);
        helper(res, root.left, depth + 1);
        helper(res, root.right, depth + 1);
    }
}
```

**双栈法：**

```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) return res;

        Stack<TreeNode> stack1 = new Stack<>();
        Stack<TreeNode> stack2 = new Stack<>();
        stack1.push(root);
        // 该变量用于记录层数，偶数层从左往右入栈，奇数层从优往左入栈
        int level = 0;
        while (!stack1.isEmpty() || !stack2.isEmpty()){
            List<Integer> temp = new ArrayList<>();
            if (level % 2 == 0){
                while (!stack1.isEmpty()){
                    TreeNode cur = stack1.pop();
                    temp.add(cur.val);
                    if (cur.left != null){ stack2.push(cur.left); }
                    if (cur.right != null){ stack2.push(cur.right); }
                }
                res.add(temp);
            }else {
                while (!stack2.isEmpty()){
                    TreeNode cur = stack2.pop();
                    temp.add(cur.val);
                    if (cur.right != null){ stack1.push(cur.right); }
                    if (cur.left != null){ stack1.push(cur.left); }
                }
                res.add(temp);
            }
            level++;
        }
        return res;
    }
}
```

