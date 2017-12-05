+++
date = "2017-12-01T20:11:31+08:00"
title = "搜索二叉树"
showonlyimage = false
image = "/img/blog/loop-tree-by-stack-queue/tree.png"
topImage = "/img/blog/loop-tree-by-stack-queue/tree.gif"
draft = false
weight = 1000
+++

学习栈和队列在树搜索中的使用
<!--more-->

### 数据结构 Tree

Tree 可用于存放有层级的数据，使我们很自然地采用决策方式思考问题。定义如下

- 一个树由一个或以上的节点组成，每个节点可用于存储数据
- 各个节点通过分支相连接
- 除了叶子，其他节点都会有一个或以上子节点，有子节点的节点叫父节点
- 在简单树中，每个子节点只有一个父节点，否则就叫图

Binary Tree 作为一个特例，其约束为每个节点至多能有两个子节点，它在存放有序数据时特别有用。

数据结构可以定义如下

{{< highlight python >}}
class BinaryTree():
    def __init__(self, value):
        self.value = value
        self.left_branch = None
        self.right_branch = None
        self.parent = None

    def set_left_branch(self, node: 'BinaryTree') -> None:
        self.left_branch = node

    def get_left_branch(self) -> 'BinaryTree':
        return self.left_branch

    def set_right_branch(self, node: 'BinaryTree') -> None:
        self.right_branch = node

    def get_right_branch(self) -> 'BinaryTree':
        return self.right_branch

    def set_parent(self, parent: 'BinaryTree') -> None:
        self.parent = parent

    def get_parent(self) -> 'BinaryTree':
        return self.parent

    def get_value(self):
        return self.value

    def __str__(self):
        return self.value

{{< /highlight >}}

### 二叉树

使用上面的定义，创建一个二叉树。

{{< highlight python >}}
...
n2 = BinaryTree(2)
n5 = BinaryTree(5)
n8 = BinaryTree(8)


n5.set_left_branch(n2)
n2.set_parent(n5)
n5.set_right_branch(n8)
n8.set_parent(n5)
...

{{< /highlight >}}

### 深度搜索

1. 从跟节点开始
2. 任何节点，若非目标，先向左搜索
3. 当达到叶子节点后，回溯到最近一个决策点向右搜索

实现算法:

- 借助 stack 保存尚未检查的节点——可用父节点代指所有未检查的子节点
- DFSBinary 设计为高阶函数，可接受一个自定义函数来判定当前节点是否为目标节点
- 若不是，则将其子节点全压入 stack ，并向左继续查找

{{< highlight python >}}
def DFSBinary(root: BinaryTree, fcn) -> bool:
    stack: List[BinaryTree] = [root]
    while len(stack) > 0:
        print('at node {}'.format(stack[0].get_value()))
        if fcn(stack[0]):
            return True
        else:
            temp = stack.pop(0)
            if temp.get_right_branch():
                stack.insert(0, temp.get_right_branch())
            if temp.get_left_branch():
                stack.insert(0, temp.get_left_branch())
    return False

{{< /highlight >}}

{{< highlight console >}}
pipenv install mypy
{{< /highlight >}}

### 广度搜索

1. 从跟节点开始
2. 逐层逐个搜寻

参考文档

> - Paulo Scardine answer [How do I specify that the return type of a method is the same as the class itself in python?](https://stackoverflow.com/a/33533514/4393386)

封面图片来自 [Heraldry 4](https://dribbble.com/shots/3454908-Heraldry-4) <a href="https://dribbble.com/federicafragapane"><i class="fa fa-dribbble" aria-hidden="true"></i> Forbender</a>
