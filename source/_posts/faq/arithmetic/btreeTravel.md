---
title: 二叉树遍历
date: 2019-07-01T12:54:24+02:00
tags: 
- 面试
- 算法
categories: 算法
---

<!-- toc -->

#### 先序遍历递归实现
```java
public List<Integer> preOrder(TreeNode root) {
    List<Integer> list=new ArrayList<Integer>();
    if(root!=null){
    	list.add(root.val);
    	list.addAll(preOrder(root.left));
    	list.addAll(preOrder(root.right));
    }	
    return list;
}
```

#### 先序遍历的非递归实现
```java
public List<Integer> preOrder2(BinaryTree node)
{
    List<Integer> list=new ArrayList<Integer>();
    if(node==null) {
        return list;
    }
    Stack<BinaryTree> stack = new  Stack<BinaryTree>();
    while(node!=null || !stack.empty())
    {
        if(node!=null)
        {
            list.add(node.val);
            stack.push(node);
            node=node.left;
        }
        else
        {
            node=stack.pop();
            node=node.right;
        }
   } 
  return list;
}
```

#### 中序遍历递归实现
```java
public List<Integer> preOrder(TreeNode root) {
    List<Integer> list=new ArrayList<Integer>();
    if(root!=null){
    	list.addAll(preOrder(root.left));
        list.add(root.val);
    	list.addAll(preOrder(root.right));
    }	
    return list;
}
```

#### 中序遍历的非递归实现
```java
public List<Integer>  inOrder2(BinaryTree node)
{
    List<Integer> list=new ArrayList<Integer>();
    if(node==null)
    return list;
    Stack<BinaryTree> stack = new  Stack<BinaryTree>();
    while(node!=null || !stack.empty())
    {
        if(node!=null)
        {
        stack.push(node);
        node=node.left;
        }
        else
        {
            node=stack.pop();
            list.add(node.val);
            node=node.right;
        }
    } 
  return list;
}
```

#### 后序遍历的递归实现
```java
public List<Integer> preOrder(TreeNode root) {
    List<Integer> list=new ArrayList<Integer>();
    if(root!=null){
    	list.addAll(preOrder(root.left));
    	list.addAll(preOrder(root.right));
        list.add(root.val);
    }	
    return list;
}
```

#### 后序遍历的非递归实现  

后序遍历的非递归实现比前两种遍历的非递归实现较为复杂，因为后序遍历是只有左右两个子树都被遍历过之后，才能遍历根节点。  
后序遍历的顺序是左子树-右子树-根节点，先序遍历的顺序是根节点-左子树-右子树，因此如果我们控制先序遍历的顺序为根节点-右子树-左子树，那么通过翻转先序遍历的序列，就可以得到遍历顺序为左子树-右子树-根节点的遍历序列，即后序遍历的序列。  
```java
public List<Integer> postOrder2(BinaryTree node)
{
  List<Integer> list=new ArrayList<Integer>();
  if(node==null)
   return list;
   Stack<BinaryTree> stack= 
  new Stack<BinaryTree>();
  while(node!=null || !stack.empty())
   {
            if(node!=null)
            {
               list.add(0,node.val);
               stack.push(node);
               node=node.right;
            }
           else
          {
              node=stack.pop();
              node=node.left;
          }
   } 
  return list;
}
```