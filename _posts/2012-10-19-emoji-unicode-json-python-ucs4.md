---
layout: post
title:  "emoji,unicode,json,python,ucs-4"
date:   2015-06-10 15:04:02 +0800
---

今天有一项工作是将微博文本中的表情 [大笑]之类的替换成emoji表情
于是在网上找到一个[emoji表情的unicode对照表](http://web.archive.org/web/20161114023427/http://www.unicode.org/~scherer/emoji4unicode/snapshot/full.html)，发现很多unicode的表情是超过2字节的，因为UCS2规范只用2个字节进行映射对照，所以需要UCS4规范了。
那么怎么在pyhton中使用四字节的unicode字符呢？

先补充一下unicode的基本知识吧：

注意过"u1234"这种转义吗？其实这就是表示这个字符是编号为"0x1234"的unicode字符，注意与任何编码无关！因为unicode只是一种规范，仅仅是给了字符到unicode编号的映射关系而已。

例如：假如“丁”这个字在unicode中编号为10000，这个10000就是“丁”的unicode序号。但是10000写在文件里面怎么写呢？必须得转换成二进制字符串吧？你是用几个字节表示呢？这才涉及到编码问题。

好了，python中，直接在代码中怎么写unicode呢？只需要在字符串前面加 u 就好了，例如：

u"丁亚光" 这样解释器会根据文件头部设置的编码(就是#coding=utf-8这一行，没有就是系统默认编码)给"丁亚光"在 py文件中的二进制串进行decode。

如果直接知道字符的unicode序号也可以，直接u"u1234"就行了。但是这里我们需要的是4字节的,怎么写呢？要用U进行转义 u"U0001F604",后面是hexlify的8位串，其实是4个字节。

当然上面是直接在代码中写的硬代码，如果在文件中读取了"U0001F604",怎么办呢？回想python有ord和chr用来将字符变成ascii码，和将ascii码转成字符。python提供了unichr完成这个工作~所以需要这么写:

```
s = "0001F604"
ch = unichr(int(s,16))
```

很不幸，python报错了：

`ValueError: unichr() arg not in range(0x10000) (narrow Python build)`

原来是python编译的时候没有选择ucs-4的支持,不能超过0x10000啊，怎么办？重新安装python？太麻烦

想想代码中还是可以用硬编码来使用ucs-4的字符的，python这种脚本语言肯定能用eval函数啦~~

```
eval('u'\U001F604'')
```

恩恩 果然可以~~

好了这回可以把替换之后的文本发送给ios客户端啦~不过悲剧发生了~ios客户端居然显示方块

三观顿时粉碎，难道对emoji 的理解是错的吗？于是开始试验：

```
s = {"name":u"U0001F494"}
#python里面的U表示4字节的unicode u表示2个字节
jsonStr = json.dumps(s)
#这时，jsonSTr的值为 {"name": "ud83dudc94"}
#我了解了一下，这个UCS4的字符，被UTF-16编码成"ud83dudc94"了
s = json.loads(jsonStr)
#这时s = {u'name': u'U0001f494'}，还原，完全没有问题
```

看了看，ud83dudc94 其实是utf-16的编码。查了下，如果不希望json把no-ascii的字符格式转换，只需要：

```jsonStr = json.dumps(s,ensure_ascii=False)```

看python正反解析都是没问题的，于是怀疑ios客户端要求的emoji格式是不是并不是unicode的？去前端围观了下怎么打emoji，发现确实是只要是unicode的就行。难道是ios端json解析器有问题？于是让前端把unicode字符串用sbjson这个Object-C的json库dumps出来，发现是没有进行no-ascii转换的，直接就是utf-8编码的二进制串.猜想sbjson应该对"ud83dudc94"这种格式支持不太好，于是打出sbjson加载的"ud83dudc94"，发现只有三个字节，其实这个的utf-8是4个字节的。似乎确认了问题的原因，于是让前端换了 jsonkit这个Object-C的json库，问题解决!

附一篇写的不错的unicode文章，不太清楚的可以参考下
[http://tech.idv2.com/2008/02/21/unicode-intro/](http://tech.idv2.com/2008/02/21/unicode-intro/)