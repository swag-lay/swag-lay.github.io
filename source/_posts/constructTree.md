---
title: constructTree
date: 2023-06-12 12:35:42
categories:
- leetcode
tags:
- construct
---

# 三种遍历方式构造二叉树的通解

## 不同之处一：寻找当前根节点

- 前+中

  当前根节点为pre[pre_start]，找出它在中序中的位置，然后把左右子树分开。

- 中+后

  当前根节点为post[post_end]，找出它在中序中的位置，然后把左右子树分开。

- 前+后

  当前根节点为pre[pre_start]，并在他在后序中的位置为post_end；左子树的根节点为pre[pre_start+1]，因此只要找到他在后序中的位置就可以分开左右子树

## 不同之处二：左右遍历范围

- 前+中
  中序遍历中，我们知道 左子树：[inorder_start,index-1], 右子树：[index+1, inorder_end]
  在前序遍历中，左子树起始位置为pre_start+1,左子树一共有(index-1 - inorder_start)个，因此左子树：[pre_start+1, pre_start+1 + (index-1 - inorder_start)]
  右子树起始位置为左子树终止位置+1，终止位置为pre_end，因此右子树：[ pre_start+1 + (index-1 - inorder_start) + 1, pre_end]
- 中+后
  中序遍历中，我们知道 左子树：[inorder_start,index-1], 右子树：[index+1, inorder_end]
  在后序遍历中，左子树起始位置为post_start，左子树一共有(index-1 - inorder_start)个，因此左子树：[post_start, post_start + (index-1 - inorder_start)]
  右子树的终止位置为post_end - 1,右子树一共有(inorder_end - (index+1))个,因此右子树:[post_end - 1 - (inorder_end - (index+1)), post_end - 1]
- 前+后
  后序遍历中，我们知道 左子树：[post_start,index], 右子树：[index+1, post_end-1]
  在前序遍历中，左子树起始位置为pre_start+1,左子树个数一共有(index - post_start)个，因此左子树：[pre_start+1, pre_start+1 + (index - post_start)]
  右子树起始位置为左子树终止位置+1，终止位置为pre_end，因此右子树：[ pre_start+1 + (index - post_start) + 1, pre_end]


## 代码：

```java
//从前序和中序遍历序列构造二叉树
class Solution {
    private Map<Integer,Integer> indexMap;
    int rootIndex=0;
    public TreeNode myBuildTree(int[] preorder,int[] inorder,int preorderLeft,int preorderRight){
        if(preorderLeft<=preorderRight){
            int rootVal=preorder[rootIndex++];
            TreeNode root=new TreeNode(rootVal);
            root.left=myBuildTree(preorder,inorder,preorderLeft,indexMap.get(rootVal)-1);
            root.right=myBuildTree(preorder,inorder,indexMap.get(rootVal)+1,preorderRight);
            return root;
        }else{
            return null;
        }
    }
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        int n=preorder.length;
        indexMap=new HashMap<>();
        for(int i=0;i<n;i++){
            indexMap.put(inorder[i],i);
        }
        return myBuildTree(preorder,inorder,0,n-1);
    }
}

//从中序和后序遍历序列构造二叉树
class Solution {
    Map<Integer,Integer> map=new HashMap<>();
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        int n=inorder.length;
        for(int i=0;i<n;i++){
            map.put(inorder[i],i);
        }
        return myBuildTree(inorder,postorder,0,n-1,0,n-1);
    }
    public TreeNode myBuildTree(int[] inorder,int[] postorder,int inorderLeft,int inorderRight,int postorderLeft,int postorderRight){
        if (postorderLeft > postorderRight) {
            return null;
        }

        int postorderRoot = postorderRight;
        // 在中序遍历中定位根节点
        int inorderRoot = map.get(postorder[postorderRoot]);
        
        // 先把根节点建立出来
        TreeNode root = new TreeNode(postorder[postorderRoot]);
        // 得到左子树中的节点数目
        int size_left_subtree = inorderRoot - inorderLeft;
        
        root.left = myBuildTree(inorder, postorder, inorderLeft,inorderRoot-1, postorderLeft, postorderLeft+size_left_subtree-1);
        
        root.right = myBuildTree(inorder, postorder, inorderRoot+1, inorderRight, postorderLeft+size_left_subtree, postorderRight-1);
        return root;
    }
}

//从前序和后序遍历构造二叉树
class Solution {
    Map<Integer,Integer> map=new HashMap<>();
    public TreeNode constructFromPrePost(int[] preorder, int[] postorder) {
        int n=postorder.length;
        for(int i=0;i<n;i++){
            map.put(postorder[i],i);
        }
        return myBuildTree(preorder,postorder,0,n-1,0,n-1);
    }
    public TreeNode myBuildTree(int[] preorder,int[] postorder,int preorderL,int preorderR,int postorderL,int postorderR){
        if(preorderL>postorderR){
            return null;
        }
        TreeNode root=new TreeNode(preorder[preorderL]);
        int leftSize=map.get(preorder[preorderL+1])-postorderL+1;
        root.left=myBuildTree(preorder,postorder,preorderL+1,preorderL+leftSize,postorderL,postorderL+leftSize-1);
        root.right=myBuildTree(preorder,postorder,preorderL+leftSize+1,preorderR,postorderL+leftSize,postorderR-1);
        return root;

    }
}
```

