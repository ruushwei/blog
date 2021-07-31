---
title: 二叉树算法
date: 2019-09-09 11:14:51
---

<!-- toc -->


### - 二叉树的打印，前中后序遍历, 递归&非递归

#### 非递归先序遍历
```java
public List<Integer> preOrder(BinaryTree node) {

    List<Integer> list=new ArrayList<Integer>();
    if(node==null) return list;
    Stack<BinaryTree> stack= new Stack<BinaryTree>();
    while(node!=null || !stack.empty()) {
        if(node!=null){
            list.add(node.val);
            stack.push(node);
            node=node.left;
        } else {
            node=stack.pop();
            node=node.right;
        }
   } 
   return list;
}
```

#### 非递归中序遍历

```java
public List<Integer> inOrder(BinaryTree node) {

    List<Integer> list=new ArrayList<Integer>();
    if(node==null) return list;
    Stack<BinaryTree> stack= new Stack<BinaryTree>();
    while(node!=null || !stack.empty()) {
        if(node!=null){
            stack.push(node);
            node=node.left;
        } else {
            node=stack.pop();
            list.add(node.val);
            node=node.right;
        }
   } 
   return list;
}

```

#### 非递归后序遍历
```java
public List<Integer> postOrder(BinaryTree node) {

    List<Integer> list=new ArrayList<Integer>();
    if(node==null) return list;
    Stack<BinaryTree> stack= new Stack<BinaryTree>();
    while(node!=null || !stack.empty()) {
        if(node!=null){
            list.add(0, node.val);
            stack.push(node);
            node=node.right;
        } else {
            node=stack.pop();
            node=node.left;
        }
   } 
   return list;
}
```

#### 022-从上往下打印二叉树 * 

```java
public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        ArrayList<Integer> result = new ArrayList<Integer>();
        if(root == null) return result;
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        while(!queue.isEmpty()) {
            TreeNode temp = queue.poll();
            result.add(temp.val);
            if(temp.left != null) queue.offer(temp.left);
            if(temp.right != null) queue.offer(temp.right);
        }
        return result;
    }
}
```

#### 060-把二叉树打印成多行 

```java
ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
    ArrayList<Integer> list = new ArrayList<Integer>();
    if(pRoot == null){
        return list;
    }
    Queue<TreeNode> queue = new LinkedList();
    //根节点入队列
    queue.offer(pRoot);
    //当队列不是空
    while(!queue.isEmpty()){
        //放里面，记录当前节点个数
        int size  = queue.size();
        ArrayList<Integer> temp = new ArrayList();
        //遍历一次是一层
        for(int i=0;i<size;i++){
            TreeNode node = queue.poll();
            if(node.left!=null){
                queue.offer(node.left);
            }
            if(node.right!=null){
                queue.offer(node.right);
            }
            temp.add(node.val);
        }
        list.add(temp);
    }
    return list;
}
```

#### 059-按之字形顺序打印二叉树 * 

按行打印的部分，按奇偶来改变头插尾插


#### 023-二叉搜索树的后序遍历序列 * 

同非递归二叉树的后序遍历， 思路: 先序是 abc, acb, 的反转 bca即需要的后序遍历。使用栈， 与先序写法差不多，只是先push right, 再left, list.add用头插. 

#### 024-二叉树中和为某一值的路径

回溯
```java
public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int target) {
    ArrayList<ArrayList<Integer>> result = new ArrayList<>();
    if (root == null) {
        return result;
    }
    find(root, target, 0, new ArrayList<Integer>(), result);
    return result;
}

private void find(TreeNode node, int target, int sum, ArrayList<Integer> path, ArrayList<ArrayList<Integer>> result) {

    sum += node.val;
    path.add(node.val);

    if (node.left == null && node.right == null && sum == target) {
        result.add(new ArrayList<>(path));
        path.remove(path.size() - 1);
        return;
    }
    
    if (node.left != null) {
        find(node.left, target, sum, path, result);
    }
    if (node.right != null) {
        find(node.right, target, sum, path, result);
    }
    path.remove(path.size() - 1);
}
```

#### 038-二叉树的深度

递归
```java
public int TreeDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    
    if (root.left == null && root.right == null) {
        return 1;
    }
    
    int leftDepth = TreeDepth(root.left);
    int rightDepth = TreeDepth(root.right);
    
    return 1 + Integer.max(leftDepth, rightDepth);
}
```


#### 二叉树层次遍历

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> levels = new ArrayList<List<Integer>>();
    if (root == null) return levels;
    Queue<TreeNode> queue = new LinkedList<TreeNode>();
    queue.add(root);
    int level = 0;
    while ( !queue.isEmpty() ) {
      // start the current level
      levels.add(new ArrayList<Integer>());

      // number of elements in the current level
      int level_length = queue.size();
      for(int i = 0; i < level_length; ++i) {
        TreeNode node = queue.remove();

        // fulfill the current level
        levels.get(level).add(node.val);

        // add child nodes of the current level
        // in the queue for the next level
        if (node.left != null) queue.add(node.left);
        if (node.right != null) queue.add(node.right);
      }
      // go to next level
      level++;
    }
    return levels;
}
```

#### 114. 二叉树展开为链表

直接展开
```java
public void flatten(TreeNode root) {
    TreeNode node = root;
    while (node != null) {
        
        // 
        if (node.left == null) {
            node = node.right;
            continue;
        }

        // 找到左分支的最右节点
        TreeNode pre = node.left;
        while (pre.right != null) {
            pre = pre.right;
        }        

        // pre 替换node 右分支的位置
        pre.right = node.right;
        node.right = node.left;
        node.left = null;
        node = node.right;
    }
}
```

先序遍历
```java
public static void preOrderStack(TreeNode root) {
    if (root == null) { 
        return;
    }
    Stack<TreeNode> s = new Stack<TreeNode>();
    while (root != null || !s.isEmpty()) {
        while (root != null) {
            System.out.println(root.val);
            s.push(root);
            root = root.left;
        }
        root = s.pop();
        root = root.right;
    }
}
```


#### 236. 二叉树的最近公共祖先

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    Stack<TreeNode> stack = new Stack();
    Map<TreeNode, TreeNode> son2ParentMap = new HashMap();
    stack.push(root);
    son2ParentMap.put(root, null);
    while (!son2ParentMap.containsKey(p) || !son2ParentMap.containsKey(q)) {
        TreeNode node = stack.pop();
        if (node.left != null) {
            son2ParentMap.put(node.left, node);
            stack.push(node.left);
        }
        if (node.right != null) {
            son2ParentMap.put(node.right, node);
            stack.push(node.right);
        }
    }
    Set<TreeNode> sets = new HashSet();
    while (p != null) {
        sets.add(p);
        p = son2ParentMap.get(p);
    }
    while (!sets.contains(q)) {
        q = son2ParentMap.get(q);
    }
    return q;
}
```