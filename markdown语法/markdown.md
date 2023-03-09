# 前言
markdown是一门轻量级的标记语言，常用于编写各大博客网站的文章，以及项目中以.md为后缀的文件内。因为打算记录下日常学习的内容方便日后重温，所以开始学习markdown语法。而markdown语法类比于html语法有许多同工异曲之处，如果有html的基础，想必学习markdown就是轻轻松松的事情。

1. [标题](#标题)
2. [段落](#段落)
3. [换行](#换行)
4. [强调](#强调)
5. [引用](#引用)
6. [列表](#列表)
7. [代码](#代码)
8. [分隔线](#分隔线)
9. [链接](#链接)
10. [图片](#图片)
11. [转义字符](#转义字符)
12. [内嵌HTML](#内嵌HTML)
13. [表格](#表格)
14. [删除线](#删除线)
15. [任务列表](#任务列表)

### <span id="标题">1. 标题</span>
若干‘#’与单个空格为前缀，标题名称为后缀，形成标题语法（n个#则代表n级标题）
```markdown
# 一级标题
## 二级标题
```
而对于一级与二级标题，可以使用以下可选语法
```markdown
一级标题
======
二级标题
-------
```


### <span id="段落">2. 段落</span>
段落为单独的一行
```markdown
abcd

efgh
```

### <span id="换行">3. 换行</span>
```markdown
<br>
```

### <span id="强调">4. 强调</span>
可实现斜体、粗体以及粗斜体，分为使用星号与下划线
```markdown
*abc*
**abc**
***abc***
```

```markdown
_abc_
__abc__
___abc___
```

### <span id="引用">5. 引用</span>
相当于创建块，块可嵌套，块中可正常使用其他元素
```markdown
> abc
> 
> def
```

```markdown
> abc
>> def
```

### <span id="列表">6. 列表</span>
列表分为无序表和有序表，列表可嵌套，列表中可正常使用其他元素
无序表可使用 * 、 - 、+
```markdown
- abc
- def
  -xyz
- ghi

+ abc
+ def
+ ghi

* abc
* def
* ghi
```

有序表
```markdown
1. abc
2. def
3. ghi
```

### <span id="代码">7. 代码</span>
代码块可使用\`\`、\`\`\`\`以及\`\`\`\`\`\`包裹内容
~~~markdown
`abc`

``abc``

```html
	<span id="代码">代码</span>
```
~~~

### <span id="分隔线">8. 分隔线</span>
要创建分隔线，请在单独一行上使用三个或多个星号 (`***`)、破折号 (`---`) 或下划线 (`___`) ，并且不能包含其他内容
```markdown
***
---
___
```

### <span id="链接">9. 链接</span>
链接文本放在中括号内，链接地址放在后面的括号中，链接title可选，也可以使用尖括号把超链接括起来
`[超链接显示名](超链接地址 "超链接title")`
`<链接地址>`

### <span id="图片">10. 图片</span>
使用感叹号 (`!`), 然后在方括号增加替代文本，图片链接放在圆括号里，括号里的链接后可以增加一个可选的图片标题文本
`![图片alt](图片链接 "图片title")`
若为链接图片则
`[![图片alt](图片链接 "图片title")](链接地址)`

### <span id="转义字符">11. 转义字符</span>
使用反斜杠转义
对于特殊字符，则需要特殊处理
### <span id="内嵌HTML">12. 内嵌HTML</span>
### <span id="表格">13. 表格</span>
使用三个或多个连字符（`---`）创建每列的标题，并使用管道（`|`）分隔每列。您可以选择在表的任一端添加管道
```markdown
| Syntax      | Description |
| ----------- | ----------- |
| Header      | Title       |
| Paragraph   | Text        |
```
通过在标题行中的连字符的左侧，右侧或两侧添加冒号（`:`），将列中的文本对齐到左侧，右侧或中心
~~~markdown
| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |
~~~
### <span id="删除线">14. 删除线</span>
```markdown
~~abc~~def
```
### <span id="任务列表">15. 任务列表</span>
在任务列表项之前添加破折号（`-`）和方括号，并`[ ]`在其前面加上空格
```markdown
- [x] Write the press release
- [ ] Update the website
- [ ] Contact the media
```