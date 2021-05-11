---
title: hexo支持markdown语法化flowchart流程图
date: 2021-05-11 10:20:17
categories:
  - markdown
tags:
  - markdown
---



hexo默认不支持流程图，flowchart 简便易食，可以瞬间使 hexo 流程图无中生有。本文记录在hexo中添加Markdown flowchart流程图的方法。

添加支持

```shell
npm install --save hexo-filter-flowchart
```

### Flow语法结构

flow 语法其实是直截了当的，分为节点定义和节点连接两部分

#### 节点定义

语法结构如下：X=>Y: Z
其中，X是变量名， Y是指操作模块名，冒号后面的Z是具体显示的文字内容。需要注意的是，冒号后要加空格才能识别，而X，Y与=>之间不允许有空格。
其中，变量名X和文字内容Z可以比较随意设置，但是Y是有固定的内容，主要有以下几种：

| 操作模块名 | 表示含义说明 |
| :----: | :----: |
| start | 开始 |
| end | 结束 |
| operation | 普通操作模块 |
| subroutine | 子任务块 |
| condition | 判断快 |
| inputoutput | 输入输出块 |

#### 节点连接

定义语法节点后，需要理顺节点之间的关系，才能建立正确的流程图；
在flow中使用->符号连接两个前后的变量；

```
如： a->b->c，表示节点a转到b又到c节点；
```


上述转接也可以写成：

```
a->b
b->c
```


condition是判断，可以取yes和no两种结果，对于不同结果可以有不同走向。如：

```
cond(yes)->out表示condition成立时转向out执行；
cond(no)->op表示condition不成立时转向op执行；
```

#### 连接方向

- 连接线有上下左右四个方向，如果需要指定连接线连接到某一特定方向，在连接线开始的元素后面添加方向即可，方向包括：

````
(top)
(bottom)
(left)
(right)
````

- 如果要设置条件框连接线方向，在括号中添加即可。条件框只有两个方向可供选择：

````
# yes向下，no向右（默认）
# yes向右，no向下。
cond(yes,right)
cond(no,bottom)
````

只需要指定其中一条分支的方向即可。

参考：https://blog.csdn.net/zywvvd/article/details/110150617