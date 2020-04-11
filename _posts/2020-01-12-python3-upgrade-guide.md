---
layout: post
title:  "Python3 新特性简略"
date:   2020-01-12 15:04:02 +0800


---



这本书全面详细的介绍了迁移到 Python3 所需要的知识：

[Supporting Python 3: An in-depth guide](http://python3porting.com/bookindex.html)

**1. print 成为 函数**
---

在Python3中print是个函数，这意味着在使用的时候必须带上小括号,并且它是带有参数的。**

```
old: print "The answer is", 2+2
new: print("The answer is", 2+2)  

old: print x,      # 末尾加上逗号阻止换行  
new: print(x, end="") # 使用空格来代替新的一行  

old: print >>sys.staerr, "fatal error"  
new: print ("fatal error", file=sys.stderr)  

old: print (x, y)   # 打印出元组(x, y) 
new: print((x, y))  # 同上,在python3中print(x, y)的结果是跟这不同的 
```



**2. import 默认使用绝对路径**
---

  Python3中import使用相对路径，必须加 `.` ， 比如有下列目录
	
  ```
	a/
	  text.py
	  processor.py
  ```

  从 processor.py 引用 text.py
	
	
  py2 写法，支持这种写法和下面py3的写法
	
	
  ```
  from text import *
  ```


  py3 写法，只支持这种写法
	
	
  ```
  from .text import *
  ```




**3. 字符串**
---

  字符串的变化可能是对Python2项目升级到Python3最大的挑战了，不仅是代码字面量默认类型的变化("a" 在Python3中是Unicode字符串，在Python2中是字节串)，关键词的含义也有相应的变化(str 在两个版本中代表的类型是不一样)。

  下表总结一些变化

|No.| | Python2 | Python3 | 备注|
|:---:|----|----|----|----|
|0.| basestring | str和unicode的父类|-| |
|1.| str | 字节串|unicode串| |
|2.| unicode|unicode串| - | |
|3.| bytes | 字节串(str别名) | 字节串 | |
|4. | b"" | 字节串 | 字节串 | |
|5. | "" | 字节串 | unicode串| |
|6. | u""|unicode串| unicode串| |
|7.|b"a" + u"b"| u"ab"| 异常 （TypeError: can't concat str to bytes）| |
|8.|u"{}".format(b"a") | u"a" | u"b'a'"  |format千万不要类型混在一起|
|9.|b"{}".format(u"a")| b"a" | 异常 （'bytes' object has no attribute 'format'）  | |
|10.|u"中文".decode('utf-8')| 异常 (UnicodeEncodeError: 'ascii' codec can't encode characters)  | 异常 (AttributeError: 'str' object has no attribute 'decode')   | Python2会先尝试使用默认编码encode再decode, Python3 直接没有decode方法|

总结：

a.  如果是文本类型，尽量在输入输出的时候就进行unicode转换，避免bytes类型传递到业务逻辑中做类型检查。  
b. 升级Python3后检查代码中所有的 encode("utf-8") 和 decode("utf-8")， 90% 会出问题。  
c. 字符串拼接和format，一定要注意上表格中 第8条

**4. 标准库改名**
---

ConfigParser 等库被改名


httplib, BaseHTTPServer, CGIHTTPServer, SimpleHTTPServer, Cookie, cookielib 被合并到 http 包内。

StringIO 模块现在被合并到新的 io 模组内。 

new, md5, gopherlib 等模块被删除

urllib 和 urllib2 合并



**不用担心 `2to3` 帮你搞定**



**7. "一切"皆迭代类型**
---

range函数、字典的 items/values/keys 方法、map函数、filter函数， 均返回迭代器类型；同时移除了显示的迭代器类型方法。

|表达式| Python2 | Python3 |
|:---:|----|----|
|xrange| 返回迭代器 | -（不存在） |
| range | 返回列表 | 返回迭代器 |
| {}.items() | [] | dict_items([])|
| {}.iteritems() | iterator | - (不存在) |
| {}.keys() | [] | dict_keys([]) |
| {}.iterkeys() | iterator | - (不存在) |
| {}.values()| [] | dict_values([])|
| {}.itervalues() | iterator | - (不存在) |
| | | |

**8. 类型比较更严格**
---

以下在 Python2 中合法，但是在Python3中不合法

|非法表达式|备注|
|---|---|
| 3 < '3' | 数字与字符串不能比较|
| 2 < None | None 不能与任何类型进行大小比较，但是可以 == 比较 |
| (3, 4) < (3, None) | 包含None |
| (4, 5) < [4, 5] | 元组和list不能比较 |
| sorted([2, '1', 3]) | 不同类型不能比较大小，所以也不能排序 |
| max(1, None) | 同上 |
| min(1, None)   | 同上 |


**9.只支持 Exception as**
---

```
# Python2 中可以使用下面两种方法捕捉异常
try:
    do_something()
except Exception, e:
    print e

# Python3 支持 as

try:
    do_something()
except Exception as e:
    print e
```

**10. 整型数**
---


* long重命名了int,因为在内置只有一个名为int的整型,但它基本跟之前的long一样。

* 像1/2这样的语句将返回float，即0.5。使用1//2来获取整型,这也是之前版本所谓的“地板除”。

* 移除了sys.maxint,因为整型数已经没了限制。sys.maxsize可以用来当做一个比任何列表和字符串下标都要大的整型数。

* repr()中比较大的整型数将不再带有L后缀。

* 八进制数的字面量使用0o720代替了0720。

* round(1, 2) 返回整型， 在Python2 中 round 总是返回 float类型


**11. 不等于表达式**
---

Python 2.x 中不等于有两种写法 != 和 <>

Python 3.x 中去掉了 <>，只有 != 一种写法。

**12.打开文件**
---

原先有两种打开方式：

```
file( ..... )  
```
```
open(.....) 
```

现在改成只能用

```
open(......)
```

**13. 从stdin输入**
--
以前

```
raw_input("提示信息")
```

现在

```
input("提示信息")
```

在 python2.x 中 raw_input() 和 input( )，两个函数都存在，其中区别为：

* raw_input()：将所有输入作为字符串看待，返回字符串类型  
* input()：只能接收"数字"的输入，在对待纯数字输入时具有自己的特性，它返回所输入的数字的类型（int, float ）

在 python3.x 中 raw_input() 和 input( ) 进行了整合，去除了 raw_input()，仅保留了 input() 函数，其接收任意任性输入，将所有输入默认为字符串处理，并返回字符串类型。


**14.对只读属性赋值**
---

下面这段代码在 Python3 下会报错

```
class A:
    @property
    def is_whitelist_only(self):
        return True

a = A()
a.is_whitelist_only = False
```


**补充**
---

上面的总结并不全面，强烈建议看一下： 2to3工具 [修改的范围](https://docs.python.org/3.6/library/2to3.html#fixers)，
