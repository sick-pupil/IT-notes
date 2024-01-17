## 1. 起步
```
#!/usr/bin/python

# -*- coding:utf-8 -*-
```

1. `#!/usr/bin/python`：`Linux`中是根据文件开头首行标记来判断文件类型的，通过文件首行指定程序来运行，使用`#!/usr/bin/python`即告诉`Linux`调用`/usr/bin`下的`python`解释器来执行文件脚本；
	而在`Windows`中这句只相当于普通的注释，无任何含义
2. `# -*- coding:utf-8 -*-`：`python 2.x`的`py`文件默认为`ASCII`码，如果文件中有中文，运行则会出现乱码，该语句则把文件编码强制转换为`utf-8` 

## 2. 基础语法
### 1. 标识符
- 在`Python`里，标识符由字母、数字、下划线组成
- 在`Python`中，所有标识符可以包括英文、数字以及下划线`_`，但不能以数字开头
- `Python`中的标识符是区分大小写的
- 以下划线开头的标识符是有特殊意义的
	- 单下划线前缀：以单下划线为前缀的变量可以正常访问，但以单下划线开头的方法不能使用`from xxx_module import *`导入，可以使用`import xxx_module`导入
	- 单下划线后缀：当一个属性名与关键字重名，可以使用下划线后缀命名属性
	- 双下划线前缀：防止父类被拓展，存在其子类同名变量或者方法，使用双下划线前缀命名，`python`解释器会进行重新命名防止冲突
	- 双下划线前后缀：`python`中的关键方法

**`python`关键字：**

|   |   |   |
|---|---|---|
|and|exec|not|
|assert|finally|or|
|break|for|pass|
|class|from|print|
|continue|global|raise|
|def|if|return|
|del|import|try|
|elif|in|while|
|else|is|with|
|except|lambda|yield|

`python`不使用花括号`{}`标明范围，`python`使用缩进写模块
```python
if True:
	print("True")
else:
	print("False")
```

`python`一般以换行作为语句结束符，但是可以使用`\`将一行语句拆分成多行语句显示，但`[] {} ()`中就不需要
```python
total = item_one + \ 
		item_two + \
		item_three

days = ['Mon', 'Tues', 'Wed',
		'Thurs', 'Fri']
```

`python`可以使用单引号、双引号、三引号表示字符串
```python
word = 'word'
sentence = "this is a sentence"
paragraph = """this is 
				a paragraph"""
```

`python`使用`#`作为单行注释，使用`'''`或者`"""`作为多行注释
```python
# 123456

'''
123456
123456
'''

"""
123456
123456
"""
```

### 2. 数据类型
`python`变量：
- `Python`中的变量赋值不需要类型声明
- 每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建
- 等号 = 用来给变量赋值
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
counter = 100 # 赋值整型变量
miles = 1000.0 # 浮点型
name = "John" # 字符串
print counter
print miles
print name

a = b = c = 1
a, b, c = 1, 2, "john"
```

`python`存在五种标准数据类型
- `Numbers`：数字，为不可改变数据类型，改变值则意味着分配一个新的对象，可以使用`del var`删除对象引用，具体数字类型有：`int`整型、`long`长整型、`float`浮点、`complex`复数
- `String`：字符串，字符串中每个字符的索引存在正序和逆序的取值，正序`0 - n-1`，逆序`-(n-1) - -1`，`n`为字符串长度
- `List`：列表，列表索引与字符串索引一样，存在正序与逆序，索引号都一样，同一个列表中可以存在多种类型的元素
- `Tuple`：元组，与列表相似，但是元组不能二次赋值，为只读列表
- `Dictionary`：字典，通过键值对存储，以键值取值

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

str = 'Hello World!'
print str # 输出完整字符串
print str[0] # 输出字符串中的第一个字符
print str[2:5] # 输出字符串中第三个至第六个之间的字符串
print str[2:] # 输出从第三个字符开始的字符串
print str * 2 # 输出字符串两次
print str + "TEST" # 输出连接的字符串

list = [ 'runoob', 786 , 2.23, 'john', 70.2 ]
tinylist = [123, 'john']
print list # 输出完整列表
print list[0] # 输出列表的第一个元素
print list[1:3] # 输出第二个至第三个元素
print list[2:] # 输出从第三个开始至列表末尾的所有元素
print tinylist * 2 # 输出列表两次
print list + tinylist # 打印组合的列表

tuple = ( 'runoob', 786 , 2.23, 'john', 70.2 )
tinytuple = (123, 'john')
print tuple # 输出完整元组
print tuple[0] # 输出元组的第一个元素
print tuple[1:3] # 输出第二个至第四个（不包含）的元素
print tuple[2:] # 输出从第三个开始至列表末尾的所有元素
print tinytuple * 2 # 输出元组两次
print tuple + tinytuple # 打印组合的元组

dict = {}
dict['one'] = "This is one"
dict[2] = "This is two"
tinydict = {'name': 'runoob','code':6734, 'dept': 'sales'}
print dict['one'] # 输出键为'one' 的值
print dict[2] # 输出键为 2 的值
print tinydict # 输出完整的字典
print tinydict.keys() # 输出所有键
print tinydict.values() # 输出所有值
```

**`python`数据类型转换：**

|   |   |
|---|---|
|`int(x [,base])`|将x转换为一个整数|
|`long(x [,base] )`|将x转换为一个长整数|
|`float(x)`|将x转换到一个浮点数|
|`complex(real [,imag])`|创建一个复数|
|`str(x)`|将对象 x 转换为字符串|
|`repr(x)`|将对象 x 转换为表达式字符串|
|`eval(str)`|用来计算在字符串中的有效Python表达式,并返回一个对象|
|`tuple(s)`|将序列 s 转换为一个元组|
|`list(s)`|将序列 s 转换为一个列表|
|`set(s)`|转换为可变集合|
|`dict(d)`|创建一个字典。d 必须是一个序列 (key,value)元组。|
|`frozenset(s)`|转换为不可变集合|
|`chr(x)`|将一个整数转换为一个字符|
|`unichr(x)`|将一个整数转换为Unicode字符|
|`ord(x)`|将一个字符转换为它的整数值|
|`hex(x)`|将一个整数转换为一个十六进制字符串|
|`oct(x)`|将一个整数转换为一个八进制字符串|

### 3. 运算符

|运算符|描述|实例|
|---|---|---|
| + | 加 - 两个对象相加 | a + b 输出结果 30 |
| - | 减 - 得到负数或是一个数减去另一个数 | a - b 输出结果 -10 |
| * | 乘 - 两个数相乘或是返回一个被重复若干次的字符串 | a * b 输出结果 200 |
| / | 除 - x除以y | b / a 输出结果 2 |
| % | 取模 - 返回除法的余数 | b % a 输出结果 0 |
| \*\* | 幂 - 返回x的y次幂 | a\*\*b 为10的20次方， 输出结果 100000000000000000000 |
| \/\/ | 取整除 - 返回商的整数部分（向下取整）| >>> 9\/\/2 <br/> 4 <br/> >>> -9//2 <br/> -5|

```python
a = 21
b = 10
c = 0

c = a + b
print "1 - c 的值为：", c

c = a - b
print "2 - c 的值为：", c

c = a * b
print "3 - c 的值为：", c

c = a / b
print "4 - c 的值为：", c

c = a % b
print "5 - c 的值为：", c

# 修改变量 a 、b 、c
a = 2
b = 3
c = a**b
print "6 - c 的值为：", c

a = 10
b = 5
c = a//b
print "7 - c 的值为：", c
```

|运算符|描述|实例|
|---|---|---|
| == | 等于 - 比较对象是否相等 | (a == b) 返回 False |
| != | 不等于 - 比较两个对象是否不相等 | (a != b) 返回 True |
| <> | 不等于 - 比较两个对象是否不相等。python3 已废弃 | (a <> b) 返回 True。这个运算符类似 !=  |
| > | 大于 - 返回x是否大于y | (a > b) 返回 False |
| < | 小于 - 返回x是否小于y。所有比较运算符返回1表示真，返回0表示假。这分别与特殊的变量 True 和 False 等价 | (a < b) 返回 True |
| >= | 大于等于 - 返回x是否大于等于y | (a >= b) 返回 False |
| <= | 小于等于 - 返回x是否小于等于y | (a <= b) 返回 True |

```python
#!/usr/bin/python
#- * -coding: UTF - 8 - * -

a = 21
b = 10
c = 0

if a == b:
    print "1 - a 等于 b"
else :
    print "1 - a 不等于 b"

if a != b:
    print "2 - a 不等于 b"
else :
    print "2 - a 等于 b"

if a < > b:
    print "3 - a 不等于 b"
else :
    print "3 - a 等于 b"

if a < b:
    print "4 - a 小于 b"
else :
    print "4 - a 大于等于 b"

if a > b:
    print "5 - a 大于 b"
else :
    print "5 - a 小于等于 b"

#修改变量 a 和 b 的值
a = 5
b = 20
if a <= b:
    print "6 - a 小于等于 b"
else :
    print "6 - a 大于  b"

if b >= a:
    print "7 - b 大于等于 a"
else :
    print "7 - b 小于 a"
```

|运算符|描述|实例|
|---|---|---|
| = | 简单的赋值运算符 | c = a + b 将 a + b 的运算结果赋值为 c |
| += | 加法赋值运算符 | c += a 等效于 c = c + a |
| -= | 减法赋值运算符 | c -= a 等效于 c = c - a |
| \*= | 乘法赋值运算符 | c \*= a 等效于 c = c \* a |
| \/= | 除法赋值运算符 | c \/= a 等效于 c = c \/ a |
| %= | 取模赋值运算符 | c %= a 等效于 c = c % a |
| \*\*= | 幂赋值运算符 | c \*\*= a 等效于 c = c \*\* a |
| \/\/= | 取整除赋值运算符 | c \/\/= a 等效于 c = c \/\/ a |

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

a = 21
b = 10
c = 0

c = a + b
print "1 - c 的值为：", c

c += a
print "2 - c 的值为：", c

c *= a
print "3 - c 的值为：", c

c /= a
print "4 - c 的值为：", c

c = 2
c %= a
print "5 - c 的值为：", c

c **= a
print "6 - c 的值为：", c

c //= a
print "7 - c 的值为：", c
```

|运算符|逻辑表达式|描述|实例|
|---|---|---|---|
|and|x and y|布尔"与" - 如果 x 为 False，x and y 返回 False，否则它返回 y 的计算值。|(a and b) 返回 20。|
|or|x or y|布尔"或" - 如果 x 是非 0，它返回 x 的计算值，否则它返回 y 的计算值。|(a or b) 返回 10。|
|not|not x|布尔"非" - 如果 x 为 True，返回 False 。如果 x 为 False，它返回 True。|not(a and b) 返回 False|

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
a = 10
b = 20
if a and b :
	print "1 - 变量 a 和 b 都为 True"
else: 
	print "1 - 变量 a 和 b 有一个不为 True"

if a or b : 
	print "2 - 变量 a 和 b 都为 True，或其中一个变量为 True"
else: 
	print "2 - 变量 a 和 b 都不为 True" 
	
# 修改变量 a 的值 
a = 0
if a and b : 
	print "3 - 变量 a 和 b 都为 True" 
else: 
	print "3 - 变量 a 和 b 有一个不为 True" 
	
if a or b : 
	print "4 - 变量 a 和 b 都为 True，或其中一个变量为 True" 
else: 
	print "4 - 变量 a 和 b 都不为 True" 
	
if not( a and b ): 
	print "5 - 变量 a 和 b 都为 False，或其中一个变量为 False" 
else: 
	print "5 - 变量 a 和 b 都为 True"
```

|运算符|描述|实例|
|---|---|---|
|in|如果在指定的序列中找到值返回 True，否则返回 False。|x 在 y 序列中 , 如果 x 在 y 序列中返回 True。|
|not in|如果在指定的序列中没有找到值返回 True，否则返回 False。|x 不在 y 序列中|

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
a = 10
b = 20
list = [1, 2, 3, 4, 5 ];
if ( a in list ): 
	print "1 - 变量 a 在给定的列表中 list 中"
else: 
	print "1 - 变量 a 不在给定的列表中 list 中"

if ( b not in list ): 
	print "2 - 变量 b 不在给定的列表中 list 中"
else: 
	print "2 - 变量 b 在给定的列表中 list 中"

# 修改变量 a 的值
a = 2
if ( a in list ): 
	print "3 - 变量 a 在给定的列表中 list 中"
else: 
	print "3 - 变量 a 不在给定的列表中 list 中"
```

|运算符|描述|实例|
|---|---|---|
|is|is 是判断两个标识符是不是引用自一个对象|**x is y**, 类似 **id(x) == id(y)** , 如果引用的是同一个对象则返回 True，否则返回 False|
|is not|is not 是判断两个标识符是不是引用自不同对象|**x is not y** ， 类似 **id(a) != id(b)**。如果引用的不是同一个对象则返回结果 True，否则返回 False。|

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
a = 20
b = 20
if ( a is b ):
	print "1 - a 和 b 有相同的标识"
else: 
	print "1 - a 和 b 没有相同的标识"

if ( a is not b ): 
	print "2 - a 和 b 没有相同的标识" 
else: 
	print "2 - a 和 b 有相同的标识"

# 修改变量 b 的值
b = 30
if ( a is b ):
	print "3 - a 和 b 有相同的标识"
else: 
	print "3 - a 和 b 没有相同的标识"

if ( a is not b ):
	print "4 - a 和 b 没有相同的标识"
else:
	print "4 - a 和 b 有相同的标识"
```

### 4. 条件、循环
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# 例3：if语句多个条件
num = 9
if num >= 0 and num <= 10: # 判断值是否在0~10之间
	print 'hello'
# 输出结果: hello
num = 10 
if num < 0 or num > 10: # 判断值是否在小于0或大于10
	print 'hello'
else:
	print 'undefine'
# 输出结果: undefine

num = 8
# 判断值是否在0~5或者10~15之间
if (num >= 0 and num <= 5) or (num >= 10 and num <= 15):
	print 'hello'
else:
	print 'undefine'
# 输出结果: undefine
```

```python
#!/usr/bin/python
count = 0
while (count < 9):
	print 'The count is:', count
	count = count + 1
print "Good bye!"

i = 1
while i < 10:
	i += 1
	if i%2 > 0: # 非双数时跳过输出
		continue
	print i # 输出双数2、4、6、8、10
i = 1
while 1: # 循环条件为1必定成立
	print i # 输出1~10
	i += 1
	if i > 10: # 当i大于10时跳出循环
		break

count = 0
while count < 5:
	print count, " is less than 5"
	count = count + 1
else:
	print count, " is not less than 5"

for letter in 'Python': # 第一个实例
	print("当前字母: %s" % letter)
fruits = ['banana', 'apple', 'mango']
for fruit in fruits: # 第二个实例
	print ('当前水果: %s'% fruit)
print ("Good bye!")

fruits = ['banana', 'apple', 'mango']
for index in range(len(fruits)):
	print ('当前水果 : %s' % fruits[index])
print ("Good bye!")
```

### 5. 列表、元组、字典
```python
list = [] # 空列表
list.append('Google') # 使用 append() 添加元素
list.append('Runoob')
print list

list1 = ['physics', 'chemistry', 1997, 2000]
print list1
del list1[2]
print "After deleting value at index 2 : "
print list1

tup1 = ('physics', 'chemistry', 1997, 2000)
tup2 = (1, 2, 3, 4, 5, 6, 7 )
print "tup1[0]: ", tup1[0]
print "tup2[1:5]: ", tup2[1:5]

tup1 = (12, 34.56)
tup2 = ('abc', 'xyz')
# 以下修改元组元素操作是非法的。
# tup1[0] = 100
# 创建一个新的元组
tup3 = tup1 + tup2
print tup3
del tup3

tinydict = {'Name': 'Zara', 'Age': 7, 'Class': 'First'}
tinydict['Age'] = 8 # 更新
tinydict['School'] = "RUNOOB" # 添加
print "tinydict['Age']: ", tinydict['Age']
print "tinydict['School']: ", tinydict['School']

tinydict = {'Name': 'Zara', 'Age': 7, 'Class': 'First'}
del tinydict['Name'] # 删除键是'Name'的条目
tinydict.clear() # 清空字典所有条目
del tinydict # 删除字典
print "tinydict['Age']: ", tinydict['Age']
print "tinydict['School']: ", tinydict['School']
```

### 6. 函数
```python
def functionName( parameters )
	...

# 定义函数
def printme( str ): 
	"打印任何传入的字符串"
	print str
	return 
# 调用函数
printme("我要调用用户自定义函数!")
printme("再次调用同一函数")

#可写函数说明
def printinfo( name, age ): 
	"打印任何传入的字符串" 
	print "Name: ", name
	print "Age ", age
	return
#调用printinfo函数
printinfo( age=50, name="miki" )

#可写函数说明
def printinfo( name, age = 35 ): 
	"打印任何传入的字符串"
	print "Name: ", name
	print "Age ", age
	return #调用printinfo函数
printinfo( age=50, name="miki" )
printinfo( name="miki" )

def printinfo( arg1, *vartuple ):
	"打印任何传入的参数"
	print "输出: "
	print arg1
	for var in vartuple:
		print var
	return
# 调用printinfo 函数
printinfo( 10 )
printinfo( 70, 60, 50 )

sum = lambda arg1, arg2: arg1 + arg2
# 调用sum函数
print "相加后的值为 : ", sum( 10, 20 )
print "相加后的值为 : ", sum( 20, 20 )
```

### 7. 模块
```python
import support
support.print_func("...")

from fib import fibonacci
from math import *
```

**命名空间和作用域：**
- 变量是拥有匹配对象的名字（标识符）。命名空间是一个包含了变量名称们（键）和它们各自相应的对象们（值）的字典
- 一个`Python`表达式可以访问局部命名空间和全局命名空间里的变量。如果一个局部变量和一个全局变量重名，则局部变量会覆盖全局变量
- 每个函数都有自己的命名空间。类的方法的作用域规则和通常函数的一样
- `Python`会智能地猜测一个变量是局部的还是全局的，它假设任何在函数内赋值的变量都是局部的。因此，如果要给函数内的全局变量赋值，必须使用`global`语句

```python
Money = 2000
def AddMoney():
	# 想改正代码就取消以下注释:
	global Money
	Money = Money + 1
	
print Money
AddMoney()
print Money
```

`dir()`函数一个排好序的字符串列表，内容是一个模块里定义过的名字；返回的列表容纳了在一个模块里定义的所有模块，变量和函数
```python
# 导入内置math模块
import math
content = dir(math)
print content

['__doc__', '__file__', '__name__', 'acos', 'asin', 'atan', 
'atan2', 'ceil', 'cos', 'cosh', 'degrees', 'e', 'exp', 
'fabs', 'floor', 'fmod', 'frexp', 'hypot', 'ldexp', 'log',
'log10', 'modf', 'pi', 'pow', 'radians', 'sin', 'sinh', 
'sqrt', 'tan', 'tanh']
```

根据调用地方的不同，`globals()`和`locals()`函数可被用来返回全局和局部命名空间里的名字
- 如果在函数内部调用`locals()`，返回的是所有能在该函数里访问的命名
- 如果在函数内部调用`globals()`，返回的是所有在该函数里能访问的全局名字

当一个模块被导入到一个脚本，模块顶层部分的代码只会被执行一次
因此，如果你想重新执行模块里顶层部分的代码，可以用`reload()`函数。该函数会重新导入之前导入过的模块`reload(module_name)`

### 8. 文件IO
```python
# 文件名，绝对路径与相对路径
# 文件访问模式
	# r:只读模式，不能写入，默认的格式,必须是文件已经存在
	# w：只写模式，覆盖写入,如果文件不存在，创建文件并写入
	# a:追加写入，在原来文件内容的基础上，继续写入数据
	# r+:读写模式，需要文件已存在
	# w+:读写模式，文件可以不存在，它可以先创建、再读写
	# b：二进制读写，对非文本文件的读写
# 读取数据的编码格式
file object = open(file_name ,mode, encoding)

read(size) # 从文件读取指定的字节数，如果未给定或为负则读取所有
readline(size) # 用于从文件读取整行，包括 “\n” 字符,指定非负整数，则返回相应的字节数
readlines(size) # 读取所有行并返回列表，如size大于零，则一次性返回相应的字节数（减少压力）

write(str) # 将字符串写入文件，返回的是写入的字符长度。
writelines() # 向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符

# 使用open去打开一个文件进行读取，假设文件不存在的话，就会抛出一个IOError的错误
# 此时open后面的close方法将不能正常执行
# 所以，为了保证无论是否出错都能正确地关闭文件，我们可以使用try ... finally来实现
try:
    f = open('e:\xxx.txt', 'r')
    print(f.read())
finally:
    if f:
        f.close()
# 引入了with语句来自动帮我们调用close()方法
with open("e:\xxx.txt","r",encoding="utf-8") as f:
    print(f.read())

# 打开一个文件
fo = open("foo.txt", "r+")
str = fo.read(10)
print "读取的字符串是 : ", str
# 查找当前位置
position = fo.tell()
print "当前文件位置 : ", position
# 把指针再次重新定位到文件开头
position = fo.seek(0, 0)
str = fo.read(10)
print "重新读取字符串 : ", str
# 关闭打开的文件
fo.close()

import os
# 重命名文件test1.txt到test2.txt。
os.rename("test1.txt", "test2.txt")
os.remove("test2.txt")
os.mkdir("newdir")
os.chdir("newdir")
os.getcwd()
os.rmdir('dirname')
```

### 9. 异常
```python
#!/usr/bin/python  
# -*- coding: UTF-8 -*-  
  
try:  
	fh = open("testfile", "w")  
	fh.write("这是一个测试文件，用于测试异常!!")  
except IOError:  
	print "Error: 没有找到文件或读取文件失败"
else:  
	print "内容写入文件成功"  
	fh.close()

try:  
	fh = open("testfile", "w")  
	fh.write("这是一个测试文件，用于测试异常!!")  
finally:
	print "Error: 没有找到文件或读取文件失败"

# 定义函数  
def mye( level ):
	if level < 1:
		raise Exception,"Invalid level!"  
		# 触发异常后，后面的代码就不会再执行  
try:
	mye(0)            # 触发异常  
except Exception,err:
	print 1,err  
else:
	print 2
```
### 10. 进程
Python多进程依赖于标准库`mutiprocessing`

|序号|方法|含义|
|---|---|---|
|1|start()|创建一个Process子进程实例并执行该实例的run()方法|
|2|run()|子进程需要执行的目标任务|
|3|join()|主进程阻塞等待子进程直到子进程结束才继续执行，可以设置等待超时时间timeout|
|4|terminate()|终止子进程|
|5|is_alive()|判断子进程是否终止|
|6|daemon|设置子进程是否随主进程退出而退出|

| 序号 | 构造方法属性 | 含义 |
| ---- | ---- | ---- |
| 1 | group | 线程组 |
| 2 | target | 要执行的方法 |
| 3 | name | 进程名 |
| 4 | args/kwargs | 要传入方法的参数 |

| 序号 | 属性 | 含义 |
| ---- | ---- | ---- |
| 1 | daemon | 和线程的`setDeamon`功能一样 |
| 2 | name | 进程名字 |
| 3 | pid | 进程号 |
#### 1. 创建方式1
```python
import os, time
import multiprocessing

class myProcess(multiprocessing.Process):
    def __init__(self, *args, **kwargs) -> None:
        super().__init__()
        self.name = kwargs['name']

    def run(self):
        print("process name:", self.name)
        for i in range(10):
            print(multiprocessing.current_process(), "process pid:",
                  os.getpid(), "正在执行...")
            time.sleep(0.2)

if __name__ == "__main__":
    task = myProcess(name="testProcess")
    task.start()
    task.join()  
    print("----------------")
```
#### 2. 创建方式2
```python
from multiprocessing import  Process

def fun1(name):
    print('测试%s多进程' %name)

if __name__ == '__main__':
    process_list = []
    for i in range(5):  #开启5个子进程执行fun1函数
        p = Process(target=fun1, args=('Python',)) #实例化进程对象
        p.start()
        process_list.append(p)

    for i in process_list:
        p.join()

    print('结束测试')
```
#### 3. 创建方式3
```python
from multiprocessing import  Process

class MyProcess(Process): #继承Process类
    def __init__(self,name):
        super(MyProcess,self).__init__()
        self.name = name

    def run(self):
        print('测试%s多进程' % self.name)


if __name__ == '__main__':
    process_list = []
    for i in range(5):  #开启5个子进程执行fun1函数
        p = MyProcess('Python') #实例化进程对象
        p.start()
        process_list.append(p)

    for i in process_list:
        p.join()

    print('结束测试')
```
#### 4. 进程池
`from concurrent.futures import ProcessPoolExecutor`
- `apply(func, args=(), kwds={})`：该函数用于传递不定参数，主进程会被阻塞直到函数执行结束
- `apply_async(func, args=(), kwds={}, callback=None)`：与`apply`用法一致，但它是非阻塞的且支持结果返回后进行回调
- `map(func, iterable, chunksize=None)`：与内置的`map`函数用法基本一致，它会使进程阻塞直到结果返回
- `map_async(func, iterable, chunksize, callback)`：与`map`用法一致，但是它是非阻塞的

```python
from  multiprocessing import Process,Pool
import os, time, random

def fun1(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    pool = Pool(5) #创建一个5个进程的进程池

    for i in range(10):
        pool.apply_async(func=fun1, args=(i,))

    pool.close()
    pool.join()
    print('结束测试')
```

```python
import time

def func2(args):  # multiple parameters (arguments)
    # x, y = args
    x = args[0]  # write in this way, easier to locate errors
    y = args[1]  # write in this way, easier to locate errors

    time.sleep(1)  # pretend it is a time-consuming operation
    return x - y


def run__pool():  # main process
    from multiprocessing import Pool

    cpu_worker_num = 3
    process_args = [(1, 1), (9, 9), (4, 4), (3, 3), ]

    print(f'| inputs:  {process_args}')
    start_time = time.time()
    with Pool(cpu_worker_num) as p:
        outputs = p.map(func2, process_args)
    print(f'| outputs: {outputs}    TimeUsed: {time.time() - start_time:.1f}    \n')

    '''Another way (I don't recommend)
    Using 'functions.partial'. See https://stackoverflow.com/a/25553970/9293137
    from functools import partial
    # from functools import partial
    # pool.map(partial(f, a, b), iterable)
    '''

if __name__ =='__main__':
    run__pool()
```
#### 5. 进程通信
##### 1. Queue
```python
def func1(i):
    time.sleep(1)
    print(f'args {i}')

def run__queue():
    from multiprocessing import Process, Queue

    queue = Queue(maxsize=4)  # the following attribute can call in anywhere
    queue.put(True)
    queue.put([0, None, object])  # you can put deepcopy thing
    queue.qsize()  # the length of queue
    print(queue.get())  # First In First Out
    print(queue.get())  # First In First Out
    queue.qsize()  # the length of queue

    process = [Process(target=func1, args=(queue,)),
               Process(target=func1, args=(queue,)), ]
    [p.start() for p in process]
    [p.join() for p in process]

if __name__ =='__main__':
    run__queue()




def fun1(q,i):
    print('子进程%s 开始put数据' %i)
    q.put('我是%s 通过Queue通信' %i)

if __name__ == '__main__':
    q = Queue()

    process_list = []
    for i in range(3):
        p = Process(target=fun1,args=(q,i,))  #注意args里面要把q对象传给我们要执行的方法，这样子进程才能和主进程用Queue来通信
        p.start()
        process_list.append(p)

    for i in process_list:
        p.join()

    print('主进程获取Queue数据')
    print(q.get())
    print(q.get())
    print(q.get())
    print('结束测试')
```
##### 2. Pipe
```python
from multiprocessing import Process, Pipe
def fun1(conn):
    print('子进程发送消息：')
    conn.send('你好主进程')
    print('子进程接受消息：')
    print(conn.recv())
    conn.close()

if __name__ == '__main__':
    conn1, conn2 = Pipe() #关键点，pipe实例化生成一个双向管
    p = Process(target=fun1, args=(conn2,)) #conn2传给子进程
    p.start()
    print('主进程接受消息：')
    print(conn1.recv())
    print('主进程发送消息：')
    conn1.send("你好子进程")
    p.join()
    print('结束测试')




import time

def func_pipe1(conn, p_id):
    print(p_id)

    time.sleep(0.1)
    conn.send(f'{p_id}_send1')
    print(p_id, 'send1')

    time.sleep(0.1)
    conn.send(f'{p_id}_send2')
    print(p_id, 'send2')

    time.sleep(0.1)
    rec = conn.recv()
    print(p_id, 'recv', rec)

    time.sleep(0.1)
    rec = conn.recv()
    print(p_id, 'recv', rec)


def func_pipe2(conn, p_id):
    print(p_id)

    time.sleep(0.1)
    conn.send(p_id)
    print(p_id, 'send')
    time.sleep(0.1)
    rec = conn.recv()
    print(p_id, 'recv', rec)


def run__pipe():
    from multiprocessing import Process, Pipe

    conn1, conn2 = Pipe()

    process = [Process(target=func_pipe1, args=(conn1, 'I1')),
               Process(target=func_pipe2, args=(conn2, 'I2')),
               Process(target=func_pipe2, args=(conn2, 'I3')), ]

    [p.start() for p in process]
    print('| Main', 'send')
    conn1.send(None)
    print('| Main', conn2.recv())
    [p.join() for p in process]

if __name__ =='__main__':
    run__pipe()
```
##### 3. Manager
```python
import multiprocessing as mp

def worker(shared_list):
    shared_list.append(6)
    print(f\"Worker: {shared_list}\")

if __name__ == '__main__':
	manager = mp.Manager()
	shared_list = manager.list([1, 2, 3, 4, 5])
	
	processes = []
	for _ in range(2):
		p = mp.Process(target=worker, args=(shared_list,))
		p.start()
		processes.append(p)
	
	for p in processes:
		p.join()
	
	print(f\"Main: {shared_list}\")




import multiprocessing as mp

def worker(shared_dict):
	shared_dict['d'] = 4
    print(f\"Worker: {shared_dict}\")

if __name__ == '__main__':
    manager = mp.Manager()
    shared_dict = manager.dict({'a': 1, 'b': 2, 'c': 3})

    processes = []
    for _ in range(2):
        p = mp.Process(target=worker, args=(shared_dict,))
        p.start()
        processes.append(p)

    for p in processes:
        p.join()

    print(f\"Main: {shared_dict}\")
```
### 11. 线程
#### 1. threading与_thread
存在两个线程模块`_thread`与`threading`
其中，`_thread`可以使用`start_new_thread()`产生新线程：`_thread.start_new_thread ( function, args, kwargs )`
- `function`：线程函数
- `args`：传递给线程函数的参数，他必须是个`tuple`类型
- `kwargs`：可选参数
```python
#!/usr/bin/python3

import _thread
import time

# 为线程定义一个函数
def print_time( threadName, delay):
   count = 0
   while count < 5:
      time.sleep(delay)
      count += 1
      print ("%s: %s" % ( threadName, time.ctime(time.time()) ))

# 创建两个线程
try:
   _thread.start_new_thread( print_time, ("Thread-1", 2, ) )
   _thread.start_new_thread( print_time, ("Thread-2", 4, ) )
except:
   print ("Error: 无法启动线程")

while 1:
   pass
```

<img src="D:\Project\IT-notes\Python\img\Thread调用过程.png" style="width:700px;height:450px;" />

而`threading`不仅包含了`_thread`模块中的所有方法外，还提供以下其他方法：
- `threading.currentThread`：返回当前的线程变量
- `threading.enumerate`：返回一个包含正在运行的线程的`list`。正在运行指线程启动后、结束前，不包括启动前和终止后的线程
- `threading.activeCount`：返回正在运行的线程数量，与`threading.enumerate`返回的`len(list)`相同
`threading`还提供`Thread`类操作线程，并提供以下方法：
- `run()`：线程活动的方法
- `start()`：启动线程
- `join()`或者`join(time)`：外部调用线程等待该线程结束
- `isAlive()`：线程是否活动
- `getName()`：获取线程名称
- `setName()`：设置线程名称

```python
from threading import Thread
from time import sleep, ctime
def func(name, sec):
    print('---开始---', name, '时间', ctime())
    sleep(sec)
    print('***结束***', name, '时间', ctime())

# 创建 Thread 实例
t1 = Thread(target=func, args=('第一个线程', 1))
t2 = Thread(target=func, args=('第二个线程', 2))

# 启动线程运行
t1.start()
t2.start()

# 等待所有线程执行完毕
t1.join()  # join() 等待线程终止，要不然一直挂起
t2.join()



from threading import Thread
from time import sleep, ctime
# 创建 Thread 的子类
class MyThread(Thread):
    def __init__(self, func, args):
        '''
        :param func: 可调用的对象
        :param args: 可调用对象的参数
        '''
        Thread.__init__(self)   # 不要忘记调用Thread的初始化方法
        self.func = func
        self.args = args

    def run(self):
        self.func(*self.args)


def func(name, sec):
    print('---开始---', name, '时间', ctime())
    sleep(sec)
    print('***结束***', name, '时间', ctime())

def main():
    # 创建 Thread 实例
    t1 = MyThread(func, (1, 1))
    t2 = MyThread(func, (2, 2))
    # 启动线程运行
    t1.start()
    t2.start()
    # 等待所有线程执行完毕
    t1.join()
    t2.join()

if __name__ == '__main__':
    main()
```
#### 2. 线程同步(锁)
```python
#!/usr/bin/python3

import threading
import time

class myThread (threading.Thread):
	def __init__(self, threadID, name, delay):
		threading.Thread.__init__(self)
		self.threadID = threadID
		self.name = name
		self.delay = delay
	def run(self):
		print ("开启线程： " + self.name)
		# 获取锁，用于线程同步
		threadLock.acquire()
		print_time(self.name, self.delay, 3)
		# 释放锁，开启下一个线程
		threadLock.release()

def print_time(threadName, delay, counter):
	while counter:
		time.sleep(delay)
		print ("%s: %s" % (threadName, time.ctime(time.time())))
		counter -= 1

threadLock = threading.Lock()
threads = []

# 创建新线程
thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)

# 开启新线程
thread1.start()
thread2.start()

# 添加线程到线程列表
threads.append(thread1)
threads.append(thread2)

# 等待所有线程完成
for t in threads:
	t.join()
print ("退出主线程")
```
#### 3. 线程优先级队列
`Python`的`Queue`模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列`Queue`，`LIFO`（后入先出）队列`LifoQueue`，和优先级队列`PriorityQueue`
这些队列都实现了锁原语，能够在多线程中直接使用，可以使用队列来实现线程间的同步

`Queue`模块中的常用方法：
- `Queue.qsize()`返回队列的大小
- `Queue.empty()`如果队列为空，返回`True`，反之`False`
- `Queue.full()`如果队列满了，返回`True`，反之`False`
- `Queue.full`与`maxsize`大小对应
- `Queue.get([block[, timeout]])`获取队列，`timeout`等待时间
- `Queue.get_nowait()`相当`Queue.get(False)`
- `Queue.put(item)`写入队列，`timeout`等待时间
- `Queue.put_nowait(item)`相当`Queue.put(item, False)`
- `Queue.task_done()`在完成一项工作之后，`Queue.task_done()`函数向任务已经完成的队列发送一个信号
- `Queue.join()`实际上意味着等到队列为空，再执行别的操作

```python
#!/usr/bin/python3
import queue
import threading
import time

exitFlag = 0

class myThread (threading.Thread):
    def __init__(self, threadID, name, q):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.q = q
    def run(self):
        print ("开启线程：" + self.name)
        process_data(self.name, self.q)
        print ("退出线程：" + self.name)

def process_data(threadName, q):
    while not exitFlag:
        queueLock.acquire()
        if not workQueue.empty():
            data = q.get()
            queueLock.release()
            print ("%s processing %s" % (threadName, data))
        else:
            queueLock.release()
        time.sleep(1)

threadList = ["Thread-1", "Thread-2", "Thread-3"]
nameList = ["One", "Two", "Three", "Four", "Five"]
queueLock = threading.Lock()
workQueue = queue.Queue(10)
threads = []
threadID = 1

# 创建新线程
for tName in threadList:
    thread = myThread(threadID, tName, workQueue)
    thread.start()
    threads.append(thread)
    threadID += 1

# 填充队列
queueLock.acquire()
for word in nameList:
    workQueue.put(word)
queueLock.release()

# 等待队列清空
while not workQueue.empty():
    pass

# 通知线程是时候退出
exitFlag = 1

# 等待所有线程完成
for t in threads:
    t.join()
print ("退出主线程")
```
#### 4. 线程池
`from concurrent.futures import ThreadPoolExecutor`
```python
import concurrent.futures
import time

# 定义一个简单的任务函数
def task(name):
    print(f"{name} is running")
    time.sleep(2)
    return f"{name} is done"

# 使用线程池
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    # 提交任务给线程池
    future_to_name = {executor.submit(task, f"Thread-{i}"): f"Thread-{i}" for i in range(5)}

    # 获取任务结果
    for future in concurrent.futures.as_completed(future_to_name):
        name = future_to_name[future]
        try:
            result = future.result()
            print(f"{name}: {result}")
        except Exception as e:
            print(f"{name}: {e}")
```
### 12. 协程
协程的作用是在执行函数A时可以随时中断去执行函数B，然后中断函数B继续执行函数A（可以自由切换）。但这一过程并不是函数调用，这一整个过程看似像多线程，然而协程只有一个线程执行

即一个线程通过不停交叉调用不同的函数，让一个线程的工作过程看起来像是多个线程执行各自的函数一样，避免了线程切换的消耗

- 执行效率极高，因为子程序切换（函数）不是线程切换，由程序自身控制，没有切换线程的开销。所以与多线程相比，线程的数量越多，协程性能的优势越明显
- 不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在控制共享资源时也不需要加锁，因此执行效率高很多
- 发展历程从`python2.x`的`yield/send` -> `greenlet` -> `gevent`到`python3.x`的`asyncio`和`async/await`
#### 1. yield+send
```python
#-*- coding:utf8 -*-
def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER]Consuming %s...' % n)
        r = '200 OK'

def producer(c):
    # 启动生成器
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER]Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER]Consumer return: %s' % r)
    c.close()

if __name__ == '__main__':
    c = consumer()
    producer(c)
```
#### 2. greenlet
`generator`实现的协程在`yield value`时只能将value返回给调用者`(caller)`。而在`greenlet`中，`target.switch(value)`可以切换到指定的协程`(target)`， 然后`yield value`。`greenlet`用`switch`来表示协程的切换，从一个协程切换到另一个协程需要显式指定
```python
# 基本的greenlet使用
from greenlet import greenlet
def test1():
    print 12
    gr2.switch()
    print 34
def test2():
    print 56
    gr1.switch()
    print 78
gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch()



# greenlet被切换后，再切换回来函数会返回值
import greenlet
def test1(x, y):
    z = gr2.switch(x+y)
    print('test1 ', z)
def test2(u):
    print('test2 ', u)
    gr1.switch(10)
gr1 = greenlet.greenlet(test1)
gr2 = greenlet.greenlet(test2)
print gr1.switch("hello", " world")



# greenlet从调用greenlet的主线程，和在主线程产生的子孙greenlet，从上至下形成一棵greenlet树
import greenlet
def test1(x, y):
    print id(greenlet.getcurrent()), id(greenlet.getcurrent().parent) # 40240272 40239952
    z = gr2.switch(x+y)
    print 'back z', z

def test2(u):
    print id(greenlet.getcurrent()), id(greenlet.getcurrent().parent) # 40240352 40239952
    return 'hehe'
gr1 = greenlet.greenlet(test1)
gr2 = greenlet.greenlet(test2)
print id(greenlet.getcurrent()), id(gr1), id(gr2)     # 40239952, 40240272, 40240352
print gr1.switch("hello", " world"), 'back to main'    # hehe back to main



# 当协程对应的函数执行完毕，协程才会die
from greenlet import greenlet
def test1():
    gr2.switch(1)
    print 'test1 finished'

def test2(x):
    print 'test2 first', x
    z = gr1.switch()
    print 'test2 back', z
gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch()
print 'gr1 is dead?: %s, gr2 is dead?: %s' % (gr1.dead, gr2.dead)
gr2.switch()
print 'gr1 is dead?: %s, gr2 is dead?: %s' % (gr1.dead, gr2.dead)
print gr2.switch(10)
```
#### 3. gevent
```python
# 基本使用
g1=gevent.spawn(func,1,2,3,x=4,y=5)
# 创建一个协程对象g1，spawn括号内第一个参数是函数名，如eat，后面可以有多个参数，可以是位置实参或关键字实参，都是传给函数eat的
g2=gevent.spawn(func2)
g1.join() #等待g1结束
g2.join() #等待g2结束
#或者上述两步合作一步：gevent.joinall([g1,g2])
g1.value#拿到func1的返回值



# 遇到io立刻 切换到另外一个任务，这是使用gevent.sleep自己产生的io操作
# time.sleep或者其他阻塞，gevent是不能识别的
import gevent
import time

def eat(name):
    print("%s:eat 1" %name)
    gevent.sleep(3)
    print("%s:eat 2" %name)

def play(name):
    print("%s:play 1" % name)
    gevent.sleep(4)
    print("%s:play 2" % name)

g1 = gevent.spawn(eat,"mike")
g2 = gevent.spawn(play,"mike")

start_time = time.time()
g1.join()
g2.join()

end_time = time.time()
print(end_time-start_time)
'''
mike:eat 1
mike:eat 2
mike:play 1
mike:play 2
7.0004003047943115
'''



# 如果希望gevent识别到大部分的IO阻塞，则需要打补丁mokey
# monkey.pathch_all() 下面所有代码的涉及到io阻塞操作都打个标记，变成非阻塞操作，让gevent可以识别
from gevent import monkey;monkey.patch_all()
import gevent
import time

def eat(name):
    print("%s:eat 1" %name)
    time.sleep(3)
    print("%s:eat 2" %name)

def play(name):
    print("%s:play 1" % name)
    time.sleep(4)
    print("%s:play 2" % name)

g1 = gevent.spawn(eat,"mike")
g2 = gevent.spawn(play,"mike")

start_time = time.time()
g1.join()
g2.join()

end_time = time.time()
print(end_time-start_time)
'''
mike:eat 1
mike:play 1
mike:eat 2
mike:play 2
4.032230854034424
'''


# join()主线程等待任务运行完后才销毁
# joinall()等待多个任务 用列表存放任务
from gevent import monkey;monkey.patch_all()
import gevent
import time

def eat(name):
    print("%s:eat 1" %name)
    time.sleep(3)
    print("%s:eat 2" %name)

def play(name):
    print("%s:play 1" % name)
    time.sleep(4)
    print("%s:play 2" % name)


g1 = gevent.spawn(eat,"mike")
g2 = gevent.spawn(play,"mike")

gevent.joinall([g1,g2])
```
#### 4. asyncio
`asyncio`用于实现异步编程，基于事件循环模型，通过协程实现任务切换和调度
- `async`：定义协程函数
- `await`：挂起协程的执行
- `eventloop`：负责协调和调度协程的执行，以及处理IO操作和定时器等事件，会循环监听事件的发生，并根据事件的类型选择适当的协程进行调度
- `run_until_complete`：`run_until_complete()`方法接受一个可等待对象作为参数，可以是协程对象、任务对象或者`Future`对象。它会持续运行事件循环，直到可等待对象完成
```python
# async、await
import asyncio
async def hello():
    print("Hello")
    await asyncio.sleep(1)  # 模拟耗时操作，挂起协程执行
    print("World")
async def main():
    await asyncio.gather(
        hello(),
        hello()
    )

asyncio.run(main())



# eventloop
import asyncio
async def hello():
    print("Hello")
    await asyncio.sleep(1)  # 模拟耗时操作，挂起协程执行
    print("World")

loop = asyncio.get_event_loop()
loop.run_until_complete(hello())
loop.close()



# io
import asyncio
async def read_file(file_path):
    async with asyncio.open(file_path, 'r') as file:
        data = await file.read()
        print(data)
asyncio.run(read_file('example.txt'))

  
import asyncio
async def write_file(file_path, content):
    try:
        async with asyncio.open_file(file_path, 'w') as file:
            await file.write(content)
            print("文件写入成功")
    except OSError:
        print("写入文件失败")

async def main():
    await write_file("myfile.txt", "Hello, world!")

asyncio.run(main())



# 异步创建网络连接
import asyncio
import aiohttp
async def fetch_data(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            data = await response.text()
            print(data)
async def main():
    await fetch_data("https://www.example.com")
asyncio.run(main())



# 异步数据库操作
import asyncio
import cx_Oracle
class OracleHandler:
    def __init__(self, username, password, hostname, port, sid):
        self.username = username
        self.password = password
        self.hostname = hostname
        self.port = port
        self.sid = sid

    async def connect(self):
        """建立与Oracle数据库的连接"""
        connection = await cx_Oracle.connect(f"{self.username}/{self.password}@{self.hostname}:{self.port}/{self.sid}")
        return connection

    async def disconnect(self, connection):
        """断开与Oracle数据库的连接"""
        await connection.close()

    async def insert(self, table_name, data):
        """插入数据"""
        connection = await self.connect()
        try:
            cursor = await connection.cursor()
            # 构造插入语句
            query = f"INSERT INTO {table_name} (column1, column2, ...) VALUES (:value1, :value2, ...)"
            await cursor.execute(query, data)
            await connection.commit()

        finally:
            await self.disconnect(connection)

    async def delete(self, table_name, condition):
        """删除数据"""
        connection = await self.connect()
        try:
            cursor = await connection.cursor()
            # 构造删除语句
            query = f"DELETE FROM {table_name} WHERE {condition}"
            await cursor.execute(query)
            await connection.commit()

        finally:
            await self.disconnect(connection)

    async def update(self, table_name, data, condition):
        """更新数据"""
        connection = await self.connect()
        try:
            cursor = await connection.cursor()
            # 构造更新语句
            query = f"UPDATE {table_name} SET column1 = :value1, column2 = :value2, ... WHERE {condition}"
            await cursor.execute(query, data)
            await connection.commit()

        finally:
            await self.disconnect(connection)

    async def select(self, table_name, columns, condition):
        """查询数据"""
        connection = await self.connect()
        try:
            cursor = await connection.cursor()
            # 构造查询语句
            query = f"SELECT {columns} FROM {table_name} WHERE {condition}"
            await cursor.execute(query)
            result = await cursor.fetchall()
            return result

        finally:
            await self.disconnect(connection)

# 示例用法
async def main():
    handler = OracleHandler("username", "password", "hostname", "port", "sid")

    # 插入数据
    data = {"value1": "foo", "value2": "bar"}  # 替换为具体的数据
    await handler.insert("table_name", data)

    # 删除数据
    condition = "column1 = 'foo'"  # 替换为具体的条件
    await handler.delete("table_name", condition)

    # 更新数据
    data = {"value1": "new_value"}  # 替换为具体的数据
    condition = "column2 = 'bar'"  # 替换为具体的条件
    await handler.update("table_name", data, condition)

    # 查询数据
    columns = "column1, column2"  # 替换为具体的列名
    condition = "column1 = 'foo'"  # 替换为具体的条件
    result = await handler.select("table_name", columns, condition)
    print(result)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```
## 3. 面向对象
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
class Employee: 
	'所有员工的基类' 
	empCount = 0
	
	def __init__(self, name, salary):
		self.name = name
		self.salary = salary
		Employee.empCount += 1
		
	def displayCount(self):
		print "Total Employee %d" % Employee.empCount
	
	def displayEmployee(self):
		print "Name : ", self.name, ", Salary: ", self.salary
```
- `empCount`变量是一个类变量，它的值将在这个类的所有实例之间共享。你可以在内部类或外部类使用 `Employee.empCount`访问
- 第一种方法`__init__()`方法是一种特殊的方法，被称为类的构造函数或初始化方法，当创建了这个类的实例时就会调用该方法
- `self`代表类的实例，`self`在定义类的方法时是必须有的，虽然在调用时不必传入相应的参数

```python
hasattr(emp1, 'age') # 如果存在 'age' 属性返回 True。
getattr(emp1, 'age') # 返回 'age' 属性的值
setattr(emp1, 'age', 8) # 添加属性 'age' 值为 8
delattr(emp1, 'age') # 删除属性 'age'
```
- `getattr(obj, name)`：访问对象的属性。
- `hasattr(obj,name)`：检查是否存在一个属性。
- `setattr(obj,name,value)`：设置一个属性。如果属性不存在，会创建一个新属性。
- `delattr(obj, name)`：删除属性。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
class Employee: 
	'所有员工的基类'
	empCount = 0
	def __init__(self, name, salary):
		self.name = name
		self.salary = salary
		Employee.empCount += 1
		
	def displayCount(self):
		print "Total Employee %d" % Employee.empCount
	
	def displayEmployee(self):
		print "Name : ", self.name, ", Salary: ", self.salary

print "Employee.__doc__:", Employee.__doc__
print "Employee.__name__:", Employee.__name__
print "Employee.__module__:", Employee.__module__
print "Employee.__bases__:", Employee.__bases__
print "Employee.__dict__:", Employee.__dict__
```
- `__dict__`：类的属性（包含一个字典，由类的数据属性组成）
- `__doc__`：类的文档字符串
- `__name__`：类名
- `__module__`：类定义所在的模块（类的全名是`__main__.className`，如果类位于一个导入模块`mymod`中，那么`className.__module__`等于`mymod`）
- `__bases__`：类的所有父类构成元素（包含了一个由所有父类组成的元组）

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
class JustCounter: 
	__secretCount = 0 # 私有变量
	publicCount = 0 # 公开变量
	
	def count(self):
		self.__secretCount += 1 
		self.publicCount += 1 
		print self.__secretCount
		
	def __count1(self):
		self.__secretCount += 1 
		self.publicCount += 1 
		print self.__secretCount
	
counter = JustCounter()
counter.count()
counter.count()
print counter.publicCount
print counter.__secretCount # 报错，实例不能访问私有变量

# Python不允许实例化的类访问私有数据，但你可以使用 object._className__attrName（对象名._类名__私有属性名）访问属性
print counter._JustCounter__secretCount
```
