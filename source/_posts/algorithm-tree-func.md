---
title: 二叉树
date: 2020-06-03 08:08:49
tags: algorithm
---

> 二叉树

<!-- more -->


## 一. 定义


![](/img/2020/a_tree_1.png)


二叉树：每个节点最多有两个分支（分支的度小于2）的树结构，可为空树。
根节点：一棵树最上面的节点称为根节点。
左右子树：某个节点的左分支叫做左子树，右分支叫做右子树。
左右孩子：某个节点的左、右分支的根节点叫做该节点的左、右孩子。
兄弟节点：具有相同父节点的节点互为兄弟节点。
节点的度：节点拥有的子树数。
叶子节点：没有任何子节点的节点称为叶子节点。
内部节点：非叶子节点称为内部节点。
根的层次：从根节点开始定义，根为第一层，根的孩子为第二层，如此计数，直到该结点为止。
树的深度和高度：二叉树中节点的最大层次称为二叉树的深度或高度。


## 一. 分类


##### 1. 完满二叉树（Full Binary Tree）
树中除了叶子节点，每个节点都有两个子节点
![](/img/2020/a_tree_2.png)


##### 2. 完全二叉树（Complete Binary Tree）
在满足满二叉树的性质后，最后一层的叶子节点均需在最左边


![](/img/2020/a_tree_3.png)

##### 3. 完美二叉树（Perfect Binary Tree）
满足完全二叉树性质，树的叶子节点均在最后一层，也就是形成了一个完美的三角形


![](/img/2020/a_tree_4.png)


##### 4. 二叉搜索树(Binary Search Tree)

- 若任意节点的左子树不空，则左子树上所有节点的值均小于它的根节点的值；
- 若任意节点的右子树不空，则右子树上所有节点的值均大于或等于它的根节点的值；
- 任意节点的左、右子树也分别为二叉查找树；


![](/img/2020/a_tree_5.png)



##### 5. 平衡二叉搜索树（Self-balancing binary search tree）又被称为AVL树
 - 具有二叉查找树的全部特性。
 - 每个节点的左子树和右子树的高度差至多等于1，即平衡因子 = | 左子树高度 - 右子树高度 | <= 1
 - 通过左旋和右旋的方式，将二叉查找树维持在平衡状态


![](/img/2020/a_tree_6.png)


##### 6. 红黑树
因为 AVL 树为了维持平衡，需要多次旋转，即数据在插入、删除很频繁的场景中，非常影响性能，所以又有了红黑树的结构，它不会像 AVL 树一样频繁调整，红黑树在最坏情况下也能保持查找时间复杂度在 O(logn)，空间复杂度也是O(logn)

![](/img/2020/a_tree_7.png)


## 二. 遍历

##### 1. 广度优先
优先遍历当前层，之后再往下一层
遍历时类似队列，节点先进先出

- 层次遍历

![](/img/2020/tree_1.png)

```python

class Node:
    def __init__(self,value=None,left=None,right=None):
         self.value=value
         self.left=left    #左子树
         self.right=right  #右子树
def levelTraverse(root):
    '''
    层次遍历，节点先进先出
    '''
    if root==None:
        return
    queue=[]
    queue.insert(0,root)
    while queue:
        node=queue.pop()
        print(node.value)
        if node.left:
            queue.insert(0,node.left)
        if node.right:
            queue.insert(0,node.right)
if __name__=='__main__':
    root=Node('1',Node('2',Node('4'),Node('5')),Node('3',Node('6'),Node('7')))
    print('层次遍历：')
    levelTraverse(root)
'''

层次遍历：
1
2
3
4
5
6
7
'''

```


##### 2. 深度优先
优先遍历下一层，再遍历当前层
遍历时类似栈，节点后进先出

- 前序遍历：根节点 -> 左节点 -> 右节点


![](/img/2020/a_tree_8.png)


```python
def preTraverse(root):  
    '''
    前序遍历
    '''
    if root==None:  
        return  
    print(root.value)  
    preTraverse(root.left)  
    preTraverse(root.right)  
if __name__=='__main__':
    print('前序遍历：')
    root=Node('1',Node('2',Node('3'),Node('4')),Node('5',Node('6'),Node('7')))
    preTraverse(root)

'''
前序遍历：
1
2
3
4
5
6
7
'''
```


- 中序遍历：左节点 -> 根节点 -> 右节点


![](/img/2020/a_tree_9.png)


```python

def midTraverse(root): 
    '''
    中序遍历
    '''
    if root==None:  
        return  
    midTraverse(root.left)  
    print(root.value)  
    midTraverse(root.right)  
  
if __name__=='__main__':
    print('中序遍历：')
    root=Node('4',Node('2',Node('1'),Node('3')),Node('6',Node('5'),Node('7')))
    midTraverse(root)

'''
中序遍历：
1
2
3
4
5
6
7
'''
```

- 后序遍历：左节点 -> 右节点 -> 根节点


![](/img/2020/a_tree_10.png)


```python

def afterTraverse(root):  
    '''
    后序遍历
    '''
    if root==None:  
        return  
    afterTraverse(root.left)  
    afterTraverse(root.right)  
    print(root.value)

if __name__=='__main__':
    print('后序遍历：')
    root=Node('7',Node('3',Node('1'),Node('2')),Node('6',Node('4'),Node('5')))
    afterTraverse(root)


'''

后序遍历：
1
2
3
4
5
6
7

'''
```