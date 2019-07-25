+++
date = "2019-03-25T12:52:49+08:00"
title = "ARTS 2019w13"
showonlyimage = false
image = "/img/blog/arts-40-per-week/matrix.webp"
draft = false
weight = 1913
tags = ["ARTS"]
+++

保持硬件感知力才能写出高性能程序
<!--more-->

## Algorithm

[leetcode #990 Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/) 通过给定地等式和不等式判断其中地逻辑谬误。

此题可用 Union Find，先将所有的的相等的变量链接起来，形成 UnionFind 的数据结构，之后若有任何一个不等式和上面的数据结构相悖，返回 False

{{< highlight python "linenos=inline, hl_lines=6">}}
def equations_possible_using_union_find(equations: List[str]) -> bool:
    uf = UnionFind()
    for leftvar, op, _, rightvar in equations:
        if op == '=':
            uf.union(leftvar, rightvar)
    return not any(op == '!' and uf.is_connect(l, r) 
                   for l, op, _, r in equations)
{{< /highlight >}}

## Review

Richard Jones Amazon Web Services 课程 Identity and Access Management

{{< figure src="/img/blog/arts-40-per-week/aws-IAM.jpg" title="AWS IAM" >}}

## Tip

利用 python 调用外部命令行，解析屏幕输出。

对于 python2/3 都可以的模块应该是 `subprocess.check_output`

> 推荐调用是 `subprocess.run(["ls", "-l", "/dev/null"], capture_output=True)` 但 python2 的模块中没有这个方法。

{{< highlight python >}}
import subprocess
import sys
from collections import OrderedDict
import pprint
import traceback

def parse_to_ordereddict(output):
    lines = output.split("\n")

    status = OrderedDict()
    header = lines[0].split()

    for objline in lines[1:]:
        attr = objline.split()
        if len(attr):
            status[attr[0]] = OrderedDict(zip(header, attr))

    return status

def parse_to_dict(output):
    lines = output.split("\n")

    status = dict()
    header = lines[0].split()

    for objline in lines[1:]:
        attr = objline.split()
        if len(attr):
            status[attr[0]] = dict(zip(header, attr))

    return status

def parse_to_arr(output):
    lines = output.split("\n")
    header = lines[0].split()
    return [ dict(zip(header, value_line.split())) for value_line in lines[1:] if value_line ]

def main():
    try:
        # python3 output is as byte and needs to be decoded
        output = subprocess.check_output(sys.argv[1:]).decode('utf-8')
        print(output)
        pprint.pprint(parse_to_ordereddict(output))
        print("\n")
        pprint.pprint(parse_to_dict(output))
        print("\n")
        pprint.pprint(parse_to_arr(output))
        print("\n")
    except:
        print("traceback:%s" % traceback.format_exc())

main()

{{< /highlight >}}

{{< highlight txt >}}
OrderedDict([('po-ngx-5ptss',
              OrderedDict([('NAME', 'po-ngx-5ptss'),
                           ('READY', '1/1'),
                           ('STATUS', 'Running'),
                           ('RESTARTS', '2'),
                           ('AGE', '26d')])),
             ('po-ngx-8kd62',
              OrderedDict([('NAME', 'po-ngx-8kd62'),
                           ('READY', '1/1'),
                           ('STATUS', 'Running'),
                           ('RESTARTS', '2'),
                           ('AGE', '26d')])),
             ('po-ngx-g4gfw',
              OrderedDict([('NAME', 'po-ngx-g4gfw'),
                           ('READY', '1/1'),
                           ('STATUS', 'Running'),
                           ('RESTARTS', '2'),
                           ('AGE', '26d')]))])


{'po-ngx-5ptss': {'AGE': '26d',
                  'NAME': 'po-ngx-5ptss',
                  'READY': '1/1',
                  'RESTARTS': '2',
                  'STATUS': 'Running'},
 'po-ngx-8kd62': {'AGE': '26d',
                  'NAME': 'po-ngx-8kd62',
                  'READY': '1/1',
                  'RESTARTS': '2',
                  'STATUS': 'Running'},
 'po-ngx-g4gfw': {'AGE': '26d',
                  'NAME': 'po-ngx-g4gfw',
                  'READY': '1/1',
                  'RESTARTS': '2',
                  'STATUS': 'Running'}}


[{'AGE': '26d',
  'NAME': 'po-ngx-5ptss',
  'READY': '1/1',
  'RESTARTS': '2',
  'STATUS': 'Running'},
 {'AGE': '26d',
  'NAME': 'po-ngx-8kd62',
  'READY': '1/1',
  'RESTARTS': '2',
  'STATUS': 'Running'},
 {'AGE': '26d',
  'NAME': 'po-ngx-g4gfw',
  'READY': '1/1',
  'RESTARTS': '2',
  'STATUS': 'Running'}]
{{< /highlight>}}
## Share

William Kennedy [Ultimate Go Live Lessons](http://www.informit.com/store/ultimate-go-programming-livelessons-9780134757483) 

课程中不断提到 Mechanical Sympathy 是写出高性能程序的关键。

数据结构章节开始讲array之前[提到](https://github.com/ardanlabs/gotraining/blob/cac7e4a12c9a6b423694dbdf6c83bc0fc2d431a8/topics/go/language/arrays/README.md)：为什么 golang 中只提供/推荐使用简单数据结构，如 slice (类似 C++ 中 vector）

- 可预测的数据访问模式，使得 CPU 中的 prefetcher 能够在数据被使用前，放入 CPU L1 Cache；
- 操作系统为了提供更快的内存访问，将活跃访问的虚拟内存地址和物理内存地址的映射放入 TLB (页表缓存)；
- 最慢的访问是每次需要用到的数据在不同的 page 上，需要访问 Page Table 查找地址——纯粹的随机访问；
- 除非有特别理由，比如用到了利用 stack、queue 等其他数据结构的高效算法，或是使得程序特别易于维护和理解，最好使用 golang slice、array、map 等基础数据机构；

{{< highlight go >}}
package caching

import "fmt"

const = {
        rows = 2 * 1024
        cols = 2 * 1024
}

var matrix [rows][cols]byte
type node struct {
        v byte
        p *node
}

var list *node

func init() {
        var last *node
        for row := 0, row < rows, row++ {
                for col := 0, col < cols, cols++ {
                        var newNode node
                        if list == nil {
                                list = &newNode
                        }
                        // if previous last exist, 
                        //   1. make it points to new created node
                        //   2. reset the last pointer
                        if last != nil {
                                last.p = &newNode
                        }
                        last = &newNode

                        if row%2 == 0 {
                            matrix[row][col] = 0xFF
                            d.v = 0xFF
                        }
                }
        }

        var ctr int
        d := list
        for d != nil {
            ctr++
            d = d.p
        }

        fmt.Println("Elements in the link list", ctr)
        fmt.Println("Elements in the matrix", rows*cols)
}
{{< /highlight >}}

{{< figure src="/img/blog/arts-40-per-week/cacheline_TLB_RAM.jpg" title="CacheLine TLB and Memory Random Access" >}}


{{< highlight go >}}
func LinkedListTraverse() int {
        var ctr int

        d := list
        for d != nil {
                if d.v == 0xFF {
                        ctr++
                }
                d = d.p
        }

        return ctr
}
{{< /highlight >}}

{{< highlight go >}}
func RowTraverse() int {
        var ctr int
        for row := 0, row < rows, row++ {
                for col := 0, col < cols, col++ {
                        if matrix[row][col] == 0xFF {
                                ctr++
                        }
                }
        }
        return ctr
}
{{< /highlight >}}

{{< highlight go >}}
func ColTraverse() int {
        var ctr int
        for col := 0, col < cols, col++ {
                for row := 0, row < rows, row++ {
                        if matrix[row][col] == 0xFF {
                                ctr++
                        }
                }
        }
}
{{< /highlight >}}

封面图片来自 [Wake up, Neo...](https://dribbble.com/shots/1017520-Wake-up-Neo) <a href="https://dribbble.com/louisbullock"><i class="fa fa-dribbble" aria-hidden="true"></i> Louis Bullock</a>