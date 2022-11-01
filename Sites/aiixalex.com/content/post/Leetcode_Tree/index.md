---
title: LeetCode - Tree - Hard
date: 2020-07-08
math: true
diagram: true
authors:
- admin
---

### 99. Recover Binary Search Tree

#### Problem

Two elements of a binary search tree (BST) are swapped by mistake.

Recover the tree without changing its structure.

**Example:**

```md
Input: [3,1,4,null,null,2]

  3
 / \
1   4
   /
  2

Output: [2,1,4,null,null,3]

  2
 / \
1   4
   /
  3
```

#### Algorithm

In case of binary search trees (BST), in-order traversal gives nodes in non-decreasing order. Since that exactly two elements of BST are swapped, exactly two elements in the array of in-order traversal is swapped mistakenly.

**Example:**

~~~md
[1, 5, 3, 4, 2, 6]
~~~

We compare each pair of elements.

- In the first abnormal pair of elements, **[5, 3]**, the previous element is the first swapped element;
- In the last abnormal pair of elements, **[4, 2]**,  the latter element is the second swapped element.

#### Space Complexity

This problem can have a constant space solution.

The process of in-order traversal does not need space and only the previous TreeNode is recorded.

#### Code

~~~cpp
class Solution {
public:
    TreeNode* first = NULL;
    TreeNode* second = NULL;
    TreeNode* pre = new TreeNode(INT_MIN);
    
    void InOrder(TreeNode* root){
        if(root == NULL){
            return;
        }
        
        InOrder(root -> left);
        
        if(first == NULL && root -> val < pre -> val){
            first = pre;
        }
        if(first != NULL && root -> val < pre -> val){
            second = root;
        }
        pre = root;
        
        InOrder(root -> right);
    }
    
    void recoverTree(TreeNode* root) {
        InOrder(root);
        swap(first -> val, second -> val);
    }
    
};
~~~



### Did you find this page helpful? Consider sharing it 🙌
