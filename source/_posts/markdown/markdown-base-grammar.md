---
title: markdown基础语法
date: 2021-05-10 19:41:23
categories:
  - markdown
tags:
  - markdown
---

Markdown是一种纯文本格式的标记语言。

通过简单的标记语法，它可以使普通文本内容具有一定的格式。

相比WYSIWYG编辑器

**优点：**

1. 因为是纯文本，所以只要支持Markdown的地方都能获得一样的编辑效果，可以让作者摆脱排版的困扰，专心写作。
2. 操作简单。比如:WYSIWYG编辑时标记个标题，先选中内容，再点击导航栏的标题按钮，选择几级标题。要三个步骤。而Markdown只需要在标题内容前加#即可

**缺点：**

1. 需要记一些语法（当然，是很简单。五分钟学会）。
2. 有些平台不支持Markdown编辑模式。

# 一、标题

在想要设置为标题的文字前面加#来表示

一个#是一级标题，

二个#是二级标题，以此类推。支持六级标题。

**注：标准语法一般在#后跟个空格再写文字。**

```
示例：
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题
```

效果如下：

# 这是一级标题

## 这是二级标题

### 这是三级标题

#### 这是四级标题

##### 这是五级标题

###### 这是六级标题

---

# 二、字体

- 加粗
  - 要加粗的文字左右分别用两个*号包起来

- 斜体
  - 要倾斜的文字左右分别用一个*号包起来
  
- 加粗斜体
  - 要倾斜和加粗的文字左右分别用三个*号包起来
  
- 删除线
  - 要加删除线的文字左右分别用两个~~号包起来
  
  ```
  示例：
  **这是加粗的文字**
  *这是倾斜的文字*`
  ***这是斜体加粗的文字***
  ~~这是加删除线的文字~~
  ```
  
  效果如下：
  
  **这是加粗的文字**
  
  *这是倾斜的文字*
  
  ***这是斜体加粗的文字***
  
  ~~这是加删除线的文字~~

# 三、引用

在引用的文字前加>即可。引用也可以嵌套，如加两个>>三个>>>

```
示例：
>这是引用的内容
>>这是引用的内容
>>>>>>>>>>这是引用的内容
```

效果如下

> 这是引用的内容

> > 这是引用的内容

> > > > > > > > > > 这是引用的内容

# 四、分割线

三个或者三个以上的 - 或者 * 都可以。

```
示例
---
----
***
*****
```

效果如下

---

-----

***

*****

# 五、图片

```
语法：![图片alt](图片地址 "图片title")

图片alt就是显示在图片下面的文字，相当于对图片内容的解释。
图片title是图片的标题，当鼠标移到图片上时显示的内容。title可加可不加
```

# 六、超链接

```
[超链接名](超链接地址 "超链接title")
title可加可不加
```

```
示例：
[简书](http://jianshu.com)
[百度](http://baidu.com)
```

效果如下

[简书](http://jianshu.com)

[百度](http://baidu.com)

# 七、列表

1. 无序列表

```
语法：无序列表用 - + * 任何一种都可以
- 列表内容
+ 列表内容
* 列表内容

注意：- + * 跟内容之间都要有一个空格
```

效果如下：

- 列表内容

+ 列表内容

* 列表内容

2. 有序列表

```
语法：数字加点
1. 列表内容
2. 列表内容
3. 列表内容
注意：序号跟内容之间要有空格
```

效果如下：

1. 列表内容
2. 列表内容
3. 列表内容

列表嵌套

**上一级和下一级之间敲三个空格即可**

- 一级无序列表内容
   - 二级无序列表内容
   - 二级无序列表内容
   - 二级无序列表内容
- 一级无序列表内容
   1. 二级有序列表内容
   2. 二级有序列表内容
   3. 二级有序列表内容
1. 一级有序列表内容
   - 二级无序列表内容
   - 二级无序列表内容
   - 二级无序列表内容
2. 一级有序列表内容
   1. 二级有序列表内容
   2. 二级有序列表内容
   3. 二级有序列表内容

# 八、表格

```
语法：
表头|表头|表头
---|:--:|---:
内容|内容|内容
内容|内容|内容

第二行分割表头和内容。
- 有一个就行，为了对齐，多加了几个
文字默认居左
-两边加：表示文字居中
-右边加：表示文字居右
注：原生的语法两边都要用 | 包起来。此处省略
```

```
示例：
姓名|技能|排行
--|:--:|--:
刘备|哭|大哥
关羽|打|二哥
张飞|骂|三弟
```

效果如下

| 姓名 | 技能 | 排行 |
| ---- | :--: | ---: |
| 刘备 |  哭  | 大哥 |
| 关羽 |  打  | 二哥 |
| 张飞 |  骂  | 三弟 |



# 九、代码

```
语法：单行代码：代码之间分别用一个反引号包起来
`代码内容`
代码块：代码之间分别用三个反引号包起来，且两边的反引号单独占一行
​```
代码...
代码...
代码...
​```
注：为了防止转译，前后三个反引号处加了小括号，实际是没有的。这里只是用来演示，实际中去掉两边小括号即可。
示例：
单行代码
`create database hero;`
代码块
(```)
function fun(){
echo "这是一句非常牛逼的代码";
}
fun();
(```)
效果如下：
单行代码
create database hero;
代码块
function fun(){
echo "这是一句非常牛逼的代码";
}
fun();
```

# 十、流程图

```
元素定义语法：
tag=>type: content:>url
tag就是元素名字，
type是这个元素的类型，有6中类型，分别为：
  1. start # 开始
  2. end # 结束
  3. operation # 操作
  4. subroutine # 子程序
  5. condition # 条件
  6. inputoutput # 输入或产出
content就是在框框中要写的内容，注意type后的冒号与文本之间一定要有个空格。
url是一个连接，与框框中的文本相绑定
->: 连接元素，
condition有两个分支，如下所示：
c2(yes)->io->e 
c2(no)->op2->e
```

示例：

flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: get_hotel_ids|past
op2=>operation: get_proxy|current
sub1=>subroutine: get_proxy|current
op3=>operation: save_comment|current
op4=>operation: set_sentiment|current
op5=>operation: set_record|current

cond1=>condition: ids_remain空?
cond2=>condition: proxy_list空?
cond3=>condition: ids_got空?
cond4=>condition: 爬取成功??
cond5=>condition: ids_remain空?

io1=>inputoutput: ids-remain
io2=>inputoutput: proxy_list
io3=>inputoutput: ids-got

st->op1(right)->io1->cond1
cond1(yes)->sub1->io2->cond2
cond2(no)->op3
cond2(yes)->sub1
cond1(no)->op3->cond4
cond4(yes)->io3->cond3
cond4(no)->io1
cond3(no)->op4
cond3(yes, right)->cond5
cond5(yes)->op5
cond5(no)->cond3
op5->e

效果：

```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: get_hotel_ids|past
op2=>operation: get_proxy|current
sub1=>subroutine: get_proxy|current
op3=>operation: save_comment|current
op4=>operation: set_sentiment|current
op5=>operation: set_record|current

cond1=>condition: ids_remain空?
cond2=>condition: proxy_list空?
cond3=>condition: ids_got空?
cond4=>condition: 爬取成功??
cond5=>condition: ids_remain空?

io1=>inputoutput: ids-remain
io2=>inputoutput: proxy_list
io3=>inputoutput: ids-got

st->op1(right)->io1->cond1
cond1(yes)->sub1->io2->cond2
cond2(no)->op3
cond2(yes)->sub1
cond1(no)->op3->cond4
cond4(yes)->io3->cond3
cond4(no)->io1
cond3(no)->op4
cond3(yes, right)->cond5
cond5(yes)->op5
cond5(no)->cond3
op5->e
```

参考：

流程图：https://www.jianshu.com/p/02a5a1bf1096

其他：https://www.jianshu.com/p/191d1e21f7ed