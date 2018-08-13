+++
date = "2017-12-01T20:11:31+08:00"
title = "搜索二叉树"
showonlyimage = false
image = "/img/blog/loop-tree-by-stack-queue/tree.png"
topImage = "/img/blog/loop-tree-by-stack-queue/tree.gif"
draft = false
weight = 401
tags = ["tree"]
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

> 为了更为清晰的表示函数的输入参数和返回值的类型，这里代码加入了类型标注，而 Dropbox 出品的第三方工具 [mypy](http://mypy-lang.org/) 则可以利用这些标注实现类型检测。

{{< highlight console >}}
pipenv install mypy
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
def DFS_binary(root: BinaryTree, fcn) -> bool:
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


### 广度搜索

1. 从跟节点开始
2. 逐层逐个搜寻

实现代码的区别仅仅在于当前节点不是目标节点时:

- 从 List 的首部取得下一个查找节点
- 记录待查找节点时，插入位置为队列的尾部
> 这里是借用 List 模拟 Queue 的 FIFO 特性。

{{< highlight python >}}
def BFS_binary(root: BinaryTree, fcn) -> bool:
    queue: List[BFSBinary] = [root]
    while len(queue) > 0:
        print('at node {}'.format(queue[0].get_value()))
        if fcn(queue[0]):
            return True
        else:
            temp = queue.pop(0)
            if temp.get_left_branch():
                queue.append(temp.get_left_branch())
            if temp.get_right_branch():
                queue.append(temp.get_right_branch())
    return False

{{< /highlight >}}

### 获取路经

如果不仅仅是知晓节点是否找到，还要获得达到节点的路徑，只需添加一个辅助函数回溯父节点即可:

{{< highlight python >}}
def trace_path(node: BinaryTree) -> List[BinaryTree]:
    if not node.get_parent():
        return [node]
    else:
        return [node] + trace_path(node.get_parent())


def BFS_binary_path(root: BinaryTree, fcn) -> List[BinaryTree]:
    queue: Deque[BinaryTree] = deque([root])
    while len(queue) > 0:
        if fcn(queue[0]):
            return trace_path(queue[0])
        else:
            temp = queue.popleft()
            if temp.get_left_branch():
                queue.append(temp.get_left_branch())
            if temp.get_right_branch():
                queue.append(temp.get_right_branch())
    return []
{{< /highlight >}}

### 有序树搜索

特别的，如果树的任意节点都遵从比它小的数据存放于其左侧子节点上，而比它大都处于右侧分支。一个简单的修改我们就可以获得巨大的搜索效率提升。

{{< highlight python >}}
def DFS_ordbinary(root: BinaryTree, fcn, ltfcn) -> bool:
    stack: List[BinaryTree] = [root]
    while len(stack) > 0:
        print('at node {}'.format(stack[0].get_value()))
        if fcn(stack[0]) > 0:
            return True
        elif ltfcn(stack[0]):
            temp = stack.pop(0)
            if temp.get_left_branch():
                stack.insert(0, temp.get_left_branch())
        else:
            temp = stack.pop(0)
            if temp.get_right_branch():
                stack.insert(0, temp.get_right_branch())
    return False


DFS_ordbinary(n5,
              lambda n: n == n6,
              lambda n: n.get_value() > 6)
# at node 5
# at node 8
# at node 6

{{< /highlight >}}

### 总结

- 不变更程序逻辑，仅通过数据结构 stack/queue ，就能在深度和广度的搜索方式间切换
- 若需要获得节点路径，添加一个辅助函数就能做到
- 若树上节点是有序存放的，微小改变就能大大提升搜索效率

参考文档

> - Paulo Scardine answer [How do I specify that the return type of a method is the same as the class itself in python?](https://stackoverflow.com/a/33533514/4393386)
> - Dan Poirier (2017-02-22) [Python type annotations](https://www.caktusgroup.com/blog/2017/02/22/python-type-annotations/)

封面图片来自 [Heraldry 4](https://dribbble.com/shots/3454908-Heraldry-4) <a href="https://dribbble.com/federicafragapane"><i class="fa fa-dribbble" aria-hidden="true"></i> Forbender</a>
