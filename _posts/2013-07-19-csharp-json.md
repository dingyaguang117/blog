---
layout: post
title:  "C#对JSON的支持"
date:   2013-04-11 15:04:02 +0800
---

.net 本身对JSON的对象映射做的很好，如果JSON数据的格式确定，并不太需要第三方库。
下面通过一个例子来简单介绍下.net中JSON的使用。  

需要引用的namespace：  
`using System.Runtime.Serialization.Json;`

然后下面是利用泛型 实现的一个通用的解析/转储 JSON 类：

```
public static class JSON
{
 
    public static T parse<T>(string jsonString)
    {
        using (var ms = new MemoryStream(Encoding.UTF8.GetBytes(jsonString)))
        {
            return (T)new DataContractJsonSerializer(typeof(T)).ReadObject(ms);
        }
    }
 
    public static string stringify(object jsonObject)
    {
        using (var ms = new MemoryStream())
        {
            new DataContractJsonSerializer(jsonObject.GetType()).WriteObject(ms, jsonObject);
            return Encoding.UTF8.GetString(ms.ToArray());
        }
    }
}
 
```


只需要定义好数据对象，就可以进行Object自动映射解析了，假如需要解析的json格式为：

```
{
   "name":"ding",
   "age":23
}
 
```

那么我们应该如下定义数据对象（注意需要引用命名空间using System.Runtime.Serialization;）：

```
[DataContract]
class User
{
    [DataMember]
    public String name{ set; get; }
 
    [DataMember(Name = "age")] //如果需要改变解析/转储时候的json key的名字的映射关系，这里的可以设置Name参数
    public int _age { set; get; }
}
```


转换过程如下：

```
//解析：
User user = JSON.parse<User>(strData);
//转储：
String strData = JSON.stringify(user)
 
```


**内嵌类型**

如果有嵌套，如何定义模型呢？ 比如：

```
{
   "name":"ding",
   "age":23,
   "girlfriends":
   [
     {
       "name":"",
       "nickName":""
     },
     ...
   ]
}
 
```

只需定义List类型就可以了

```
[DataContract]
class User
{
    [DataMember]
    public String name{ set; get; }
 
    [DataMember(Name = "age")]
    public int _age { set; get; }
 
    [DataMember]
    public List<GirlFriend> girlFriends {set; get;}
}

[DataContract]
class GirlFriend
{
    [DataMember]
    public String name{ set; get; }
 
    [DataMember]
    public String nickName { set; get; }
}
```