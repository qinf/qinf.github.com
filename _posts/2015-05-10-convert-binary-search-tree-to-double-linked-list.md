---
layout: post
title: 面试题27：二叉搜索树与双向链表
date: 2015-05-10
categories: 剑指Offer
tags: [binary tree]
---

题目：输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。比如输入下图中左边儿茶搜索树，则输出转换后的排序双向链表。
<!-- more -->
```
// 结构体定义
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
```
解题思路：
1. 二叉排序树的中序遍历是有序的。
2. 二叉树的节点中左右孩子指针可以相当于双向链表的指针。
3. 根节点的左指针直向左子树的中序排序的最右节点，根节点的右指针指向右子树中序遍历的最左节点。如图。
4. 递归。
![二叉搜索树与双向链表][1]
代码如下：
```C++
class Solution {
public:
    TreeNode* Convert(TreeNode* pRootOfTree)
    {
        TreeNode* last = nullptr;   // 指向双向链表的最后一个指针
        convert(pRootOfTree, last);
        TreeNode* head = last;
        // 寻找双向链表的头指针
        while (last != nullptr && last->left != nullptr) {
            last = last->left;
        }
        head = last;
        return head;
    }
private:
    void convert(TreeNode* root, TreeNode* &last) {
        if (root == nullptr)
            return;
        TreeNode* cur = root;           // 根节点
        if (cur->left != nullptr)       // 将左子树链表化
            convert(cur->left, last);
        cur->left = last;               // 根的左指针指向链表最后一个指针，也就是说指向左子树中节点值最大的节点
        if (last != nullptr)
            last->right = cur;
        last = cur;                     // last指针指向根节点
        if (cur->right != nullptr)      // 处理右子树
            convert(root->right, last);
    }
};
```

  [1]: http://7xj0l3.com1.z0.glb.clouddn.com/20150510160443.jpg
