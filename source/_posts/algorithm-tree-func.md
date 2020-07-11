---
title: 二叉树遍历
date: 2020-06-03 08:08:49
tags: algorithm
---

> 二叉树遍历

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


## 二. 遍历

##### 1. 广度优先(Breadth First Search)
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


##### 2. 深度优先(Depth First Search)
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