---
layout: post
title:  "Python3 新特性简略"
date:   2019-12-12 15:04:02 +0800

---



*Python3 新特性
----

**1. typing**
---

> typing模块并不会在运行时强制检查类型，不符合的类型也可以继续工作

示例
    
```
from typing import Dict, List
    
def numbers(n: int) -> List[int]:
   return [1] * n
```

有啥用？
    
1. 于代码含义的解释和补充
2. 有助于 IDE 推导类型，完善自动补全 (比如：db.session)
3. 帮助代码检查工具检查代码
   

[typing官方文档](https://docs.python.org/zh-cn/3/library/typing.html)
    
**2.f-string**
---

引用上下文变量、函数，简化 format 写法
    
```
>> name = "James"
>> def foo():
....    return 20
>> f"Hello, {name}, {foo()}!"'
"Hello, James, 20!"
```


**3.dict 保证插入有序**
---

```
>> {str(i):i for i in range(5)}
{'0': 0, '1': 1, '2': 2, '3': 3, '4': 4}
```

**4.解包**
---

a) 列表解包  
    
```
>>a, b, *others = [1, 2, 3, 4]  
a -> 1  
b -> 2  
others = [3, 4]
```

b) 字典解包  
    
```
>>x = dict(a=1, b=2)
>>y = dict(b=3, d=4)
# Python 3.5+
>>z = {**x, **y}
```

c）列表、元组、set解包
    
```
>>[*a, *b, *c] 
>>(*a, *b, *c) 
>>{*a, *b, *c} 
```

**5.更简单的super**
---

```
# Python 2
class MyClass(Base):
    def __init__():
        super(MyClass, self).__init__()
    
# Python 3
class MyClass(Base):
    def __init__(self:
        super().__init__()
```

**6.原生的协程库 asyncio**
---

```
import asyncio
    
# 3.5以前
@asyncio.coroutine
def py34_coro():
    yield from stuff()
    
# 3.5+ 支持 async 和 await 关键字
async def py35_coro():
    await stuff()
```


tornado 6.0以后，默认的事件循环也使用的asyncio. 并且支持原生的语法。


​    
**7. 更好用的路径库**
---

```
from pathlib import Path
```

假设有 `filename = 'a/b/c/1.jpg'`
    
| 功能 |Python2|Python3|
|---|---|---|
| 父目录 |os.path.dirname(os.path.dirname(filename))| Path(filename).parent.parent |
| | | Path(filename).parents[1]  |
| 后缀 |os.path.basename(filename).split('.')[-1] | Path(filename).suffix |
| join | os.path.join('/roo', filename)| Path('/root', filename)  |
| | |  Path('/root') / filename  |
| 换文件名 | os.path.join(os.path.dirname(filename), '2.jpg') | Path(filename)..with_name('2.jpg') |
| 换后缀 | filename.split('.', 2)[0] + '.png'   | Path(filename).with_suffix('.png') |


**8. 新的标准库**
---

- asyncio（异步IO）
- faulthandler
- ipaddress （IP地址操作、计算）
- functools.lru_cache （LRU Cache装饰器）
- enum （枚举类）
- pathlib (更便捷的 路径库)


**9. 海象表达式 （Python3.8） :=**
---

赋值和使用放在一起

```
if (n := len(a)) > 10:
    print(f"List is too long ({n} elements, expected <= 10)")
```

```
while (block := f.read(256)) != '':
    process(block)
```