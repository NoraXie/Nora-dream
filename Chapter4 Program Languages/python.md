## Python3基础语法

### 数据类型

> 为什么所有的编程语言的变量名都规定不能以数字开头?

以数字开头的话,无法将变量名与数字字面量区分开.如:`x = 3x`这样将无法确定3x是数字还是变量名.

### 容器 类型 对象

`1` list tuple string dict 可以跨行定义最后一个元素可以使用逗号结束  
`2` 所有对象都有引用计数,当引用计算为0时,会被GC回收  
`3` list dict支持两种类型的复制操作: 浅复制(创建一个新的引用对象)和深度复制(创建新的引用对象和指向每一个元素对象import copy和copy.deepcopy())  
`4` 所有对象都是"第一类",使用同一标识符对象都具有相同状态,能够当数据进行处理  
`5` 所有序列都支持迭代,list tuple可以包含任意Python对象  
`6` 所有序列支持的操作和方法

```python
s[index]
s[i:j]
s[i:j:stride]
len(s)
min(s)
max(s)
sum(s)
all(s)
any(s)
s1 + s2: 连接
s1 * N: 重复
obj in s1: 成员关系判断
obj not s1: 成员关系判断
```

`7` 可变序列的操作

```python
s[index] = value: 元素赋值
s[1:j] = t: 切片赋值
del s[index]: 
del s[i:j]: 
del s[i:j:stride]: 

```

### 表达式与语句

三元选择表达式: `x if y else z`

匿名函数: lambda args: expression

生成器函数发送协议: yield x 

#### 运算优先级

`1` (),[],{}  
`2` s[i],s[i:j]  
`3` s.attribute  
`4` +x,-x,~x  
`5` x ** y 
`6` 四则运算  
`7` <<, >>  
`8` 逻辑运算  
`9` 关系运算  
`10` is, not is
`11` not
`12` and/or  
`13` lambda

#### 语句

`1` 赋值语句  
`2` 调用  
`3` 打印  
`4` if/elif/else 条件判断
`5` for/else 序列迭代  
`6` while/else 普通循环  
`7` pass 占位符  
`8` yield  
`9` global 全名空间   
`10` raise 触发异常  
`11` from 模块属性访问  
`12` class 类  
`13` try/catch/finally 捕捉异常  
`14` del 删除引用   
`15` assert 调试检查  
`16` with/as 环境管理器

> 赋值语句

未赋值的变量名被调用时会抛出异常NameError,表示引用了一个没有赋值的变量.如:

```python
In [265]: print(abc)
-------------------------------------------------------------------
NameError                         Traceback (most recent call last)
<ipython-input-265-fb8e540c9088> in <module>()
----> 1 print(abc)

NameError: name 'abc' is not defined
```
隐式赋值: import, from, def, class, for和函数参数.这些语句会触发隐式赋值.

分解赋值: tuple, list. 当赋值符号左值为tuple/list时,python会按照位置逐一对应进行匹配.个数不匹配时触发异常.

多重目标赋值:

增强赋值: +=, -=, /=, *=, %=, >>=, <<=

### 流程控制 

#### 条件测试

> if 条件测试express:

> x if bool express else z

x if x < y else y

当x<y时,表达式的值=x; 当x>=y时,表达式的值=y

#### 循环

> for循环

通用的序列迭代器,主要用于遍历.

> while循环

用于编写通用迭代结构,非遍历式的循环,因为遍历需要计数

```python
while boolean_expression:
	while-suite
else:
	else_suite
```
else分支仅在循环结束后执行一次.

`栗子1`

```python
In [283]: url = 'xpercent.tech'

In [284]: while url:
     ...:     print(url)
     ...:     url = url[1:]
     ...:
xpercent.tech
percent.tech
ercent.tech
rcent.tech
cent.tech
ent.tech
nt.tech
t.tech
.tech
tech
ech
ch  
h    
```
`栗子2`

```python
In [7]: url = 'xpercent.tech'

In [8]: while url:
   ...:     print(url)
   ...:     url = url[:-1]
   ...: else:
   ...:     print('game over')
   ...:
xpercent.tech
xpercent.tec
xpercent.te
xpercent.t
xpercent.
xpercent
xpercen
xperce
xperc
xper
xpe
xp
x
game over
```

`栗子3`

```python
In [11]: x = 0

In [12]: url = 'xpercent.tech'

In [13]: while url:
    ...:     print(url)
    ...:     url = url[:-1]
    ...:     x += 1
    ...:     if x  > 7:
    ...:         break
    ...: else:
    ...:     print('game over')
    ...:
xpercent.tech
xpercent.tec
xpercent.te
xpercent.t
xpercent.
xpercent
xpercen
xperce
```

栗子3说明,当循环由break终止执行,else语句块不会执行.

> 隐式迭代工具

in成员关系测试  
列表解析  
内置的map,reduce和filter函数  


> 循环 生成器 enumerate

`列表解析`

`生成器`

`迭代器`

`enumerate`


### 文件

> 文件打开模式

`r` 只读模式打开  
`w` 写模式打开  
`a` 追加模式打开  
`r+` 以读模式打开,但是同时支持文件读写  
`w+` 以写模式打开,同时支持读写  
`a+` 以追加模式打开,同时支持读写  


> os模块(与文件系统交互的模块)

支持跨平台的文件操作API.支持的方法和属性特别多,此外`os子模块os.path模块`实现文件路径字符串的管理.常用的

```python
目录相关
chdir()/fchdir() 改变工作目录
chroot() 	改变当前进程的根目录
listdir() 	列出指定目录下的所有文件名 包括目录
mkdir() 	创建指定目录 如果给的路径不存在会抛出异常
makedirs() 	如果父目录不存在的话 先创建你目录 再创建目录本身
getcwd()
rmdir()
removedirs() 

文件相关
mkfifo()	创建先进先出的管道文件
mknod()		创建设备文件
remove()	删除文件
unlink()	删除链接文件
rename()	文件重命名
renames()	
stat()		返回文件状态信息
symlink()	创建一个符号链接 
utime()		更新文件时间戳
tmpfile()	创建并打开(w+b)一个新的临时文件

访问权限相关
access()	检验用户对文件是否有访问权限 
chmod()		改变权限 
chown()		改变属主属组

文件描述符
open() 		low level IO, 调用系统调用实现的
read()		根据文件描述符读
write()		根据文件描述符写

设备文件
mkdev()		根据文件的主设备号 次设备号创建设备(mknod是创建设备文件的)
major()		获取主设备号
minor()		获取次设备号

os.path模块
basename()	路径基名
dirname()	路径父目录名
join()		将
split()		返回(filename,basename)tuple
splitext()	返回(filename,extension)tuple

getatime()	文件最近一次时间
getctime()	文件创建时间
getmtime()	文件最近一次修改时间
getsize()	返回文件大小

exists()	指定文件是否存在
isabs()		判断指定路径是否为绝对路径
isfile()	是否为文件
isdir()		是否为目录
islink()	是否为符号链接
ismount() 	是否为挂载点
samefile()	判断两个路径是否指向同一个文件
```

练习: 判断一个文件是否存在,存在则打开,打开后让用户通过键盘反复输入多行数据,追加保存至些文件中.

```python
#!/usr/bin/python36
#
import os
import os.path

filename = '/home/hbase/python/c'

if os.path.isfile(filename):
    f = open(filename,'a+')

while True:
    line = input("please enter sth>")
    if line == 'q' or line == 'quit':
        break;
    f.write(line + '\n')

f.flush()
f.close()
```

> 对象持久化

对象的流式化工具(相当于java的序列化),常用模块  
`pickle` 将Python的原生对象存储到文件中  
`marshal` 

第三方数据库接口DBM: 这种接口不支持扁平化.

### 函数

#### 函数类型

`全局函数`  直接定义在模块中  

`局部函数`	被嵌套于其它函数中,闭合,装饰器,

`lambda函数`	表达式

`方法`	与特定数据类型关联的函数, 并且只能与数据类型关联一起使用

> 语法

```python
def functionName(parameters):  
    suite
```
> 名称空间 namespace

namespace是用来管理python中的变量.python中创建\删除\修改变量都在namespace中查看到.ipython解释器通过`whos`命令查看namespace中的全局变量.

变量作用域即名称空间:  

`全局作用域`	在整个python代码文件(比如module)中能使用的变量

`函数作用域`	只能在函数中使用的变量
